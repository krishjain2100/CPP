Handling $X$ simultaneous submissions is where pure system design takes over.

To handle $X$ submissions, you don't actually run them all at the exact same nanosecond. You build an **Asynchronous Job Pipeline** that processes as many concurrently as your hardware allows, and queues the rest.

Here is the architectural blueprint for how you scale this to handle any number of submissions without crashing.

### The 4 Pillars of Your Distributed Platform

#### 1. The API Gateway (The Receptionist)

When a contestant submits their code, it hits a lightweight web server (written in Node.js, Python FastAPI, or Go).

- **Job:** It does **zero** heavy lifting. It takes the `.cpp` file, saves it to a cheap storage bucket (like AWS S3 or a local folder), and immediately generates a `submission_id`.

- **The Magic:** It replies to the user instantly: _"Submission received! You are #14 in the queue."_


#### 2. The Message Queue (The Waiting Room)

The API Server takes that `submission_id` and drops it into a Message Broker.

- **The Tools:** Redis (using Celery or BullMQ), RabbitMQ, or AWS SQS.
    
- **Why it’s required:** If your system can only handle 5 tests at a time, but 50 come in, the Queue safely holds the other 45 in memory. As soon as a test finishes, the next one in line pops out.

#### 3. The Worker Fleet (The Arenas)

This is the muscle of your platform. You spin up multiple distinct virtual machines (VMs) or cloud servers. These are your "Workers."

When a Worker is idle, it pulls the next `submission_id` from the Queue and executes the pipeline:

1. **Build:** It pulls the code and runs the Multi-Stage Docker build we discussed.
    
2. **Deploy:** It starts the Contestant's sterile Docker container.
    
3. **Attack:** It boots up your C++ Load Generator (`load_generator.cpp`) and points it at the container.
    
4. **Score:** It measures the Throughput (TPS) and Latency.
    
5. **Clean:** It destroys the Docker container and tells the Queue: _"I'm ready for the next one."_
    

#### 4. The Telemetry Database (The Scoreboard)

While the Load Generator is attacking, it needs somewhere to put the results. It streams the latencies and TPS numbers to a fast database (like PostgreSQL or TimescaleDB).

Your frontend website then constantly reads this database to update the Live Leaderboard.

### The "Noisy Neighbor" Problem (Critical for Fairness)

There is one massive trap you must avoid when testing performance: **Resource Starvation.**

If you run the Contestant's Docker Container and your C++ Load Generator on the exact same CPU cores, your Load Generator will steal CPU cycles away from their code. Their code will look slow, but it's only slow because your bot swarm is hogging the processor.

**The Solution:**

When your Worker Node runs a test, you must use Docker's resource limitation flags to pin them to different hardware.

- Give the Contestant's container exactly 2 CPU cores (`--cpus=2.0`).
    
- Pin your Load Generator to the other 2 CPU cores.
    
- This ensures a perfectly fair, isolated test environment (a "Sandbox").
    

### How to Build This for the Hackathon

You don't need to rent a massive AWS server farm to prove this works. You can simulate the entire distributed system on your Mac:

- Terminal 1 runs the **API Server**.
    
- Terminal 2 runs a local **Redis** instance (the Queue).
    
- Terminal 3 and 4 run **Worker Scripts** that listen to Redis and execute Docker commands.
    

This proves to the judges that your architecture _could_ scale to 10,000 users, even if you are only running it locally.

Which of these pillars feels like the biggest mystery right now—setting up the Message Queue, or writing the script that automates the Docker build/run commands?


---

redis-server --port 6379
to start up a redis server at 6379

---


Why use JavaScript (Node.js) for the Worker when you are already writing the high-speed Load Generator in C++?


The Worker script doesn't actually do any heavy computing. Its entire existence consists of waiting around:

- Waiting for Redis to hand it a job.
- Waiting for Docker to finish compiling the code.
- Waiting for the C++ Load Generator to finish its attack.

Node.js is famous for its Event Loop, which is mathematically optimized for waiting on network and file system I/O. C++ is optimized for raw CPU crunching. Using C++ to wait for a Docker command to finish is like using a Formula 1 car to wait in a drive-thru line.

### 2. Spawning Subprocesses is Painful in C++

The core job of this Worker is to run terminal commands (e.g., `docker build`, `./load_generator`).

- **In Node.js:** It is literally a built-in one-liner: `await exec('docker run ...')`.
    
- **In C++:** You have to mess with `std::system` (which is highly insecure and blocks the thread) or manually manage POSIX `fork()` and `execvp()` calls, wire up the standard output pipes to catch compiler errors, and manually reap the zombie processes so you don't leak memory.
    

### 3. Native JSON Handling

Redis passes the job tickets as JSON strings.

- **In Node.js:** JavaScript _is_ JSON. You just type `JSON.parse(string)` and you instantly have an object you can read.
    
- **In C++:** The standard library doesn't parse JSON. You would have to download and link a third-party library (like `nlohmann/json`) and write boilerplate parsing logic just to read the `job_id`.
    

### The Architecture Summary

- **The API Gateway (Node.js):** Handles slow web clients and JSON gracefully.
    
- **The Orchestrator Worker (Node.js):** Acts as the manager, effortlessly parsing JSON from Redis and commanding the OS to launch Docker containers.
    
- **The Load Generator (C++):** The raw muscle. Built to bypass OS limits and fire 100,000 TCP packets a second directly into the Docker sandbox.
    

We use the right tool for the right job so you can build the architecture in hours instead of weeks.

That being said, it is your hackathon! We can absolutely scrap the Node.js worker and write a C++ orchestrator using `libhiredis` if you want to keep the entire backend stack in C++. Do you want to stick with Node.js to wire up the actual Docker execution commands next, or would you rather switch gears and build the worker in C++?

i dont get it, then why not have js file for load generation too

This is the absolute perfect question to ask right now. If Node.js is the king of asynchronous networking, why don't we just write the Load Generator in JavaScript too and keep the whole project in one language?

It comes down to the difference between **I/O-Bound** tasks and **CPU-Bound** tasks.

Here is why JavaScript fails at load generation, and why C++ is required.

### 1. Fast at Waiting vs. Fast at Doing

Node.js is fast at _waiting_. When the Worker script tells Docker to compile a container, Node isn't actually doing the compiling. It just sends the command to the OS and says, _"Wake me up when you're done."_ It takes almost 0% of your CPU to do this.

A Load Generator, however, is not waiting. It is actively creating, formatting, and firing hundreds of thousands of TCP packets per second. This requires massive mathematical and memory-allocation work. This is a **CPU-Bound** task.

If you ask Node.js to mathematically construct 100,000 packets a second in a `while` loop, its single-threaded JavaScript engine will hit 100% CPU capacity and choke.

### 2. The Garbage Collector Ruins Telemetry

To judge a hackathon accurately, you need to calculate **p99 latency** with microsecond precision. You need to know exactly how fast the contestant's code replied.

- **In C++:** You have manual memory management. The memory is exactly where you put it, and it never moves. The clock is perfectly accurate.
    
- **In JavaScript:** Node.js uses a Garbage Collector. Every few milliseconds, the engine randomly pauses your code for a fraction of a second to clean up unused memory. If that pause happens while you are timing a contestant's server, your Load Generator will incorrectly report a slow response time. **JavaScript will literally falsify your hackathon scores.**
    

### 3. Bare Metal vs. Abstraction Layers

When you blast packets at a server, you are fighting the Operating System's kernel limits.

- **C++ (with Boost.Asio):** Talks directly to your Mac's kernel (using `kqueue` or `epoll`). It says, _"Give me raw access to the network card and get out of my way."_
    
- **JavaScript:** Runs inside the V8 JavaScript Engine, which runs inside the Node.js runtime, which talks to the `libuv` library, which finally talks to the OS. Every layer adds milliseconds of delay.
    

### The Analogy

- **Node.js is an Air Traffic Controller:** It is brilliant at sitting in a tower, looking at a radar screen, and managing 1,000 airplanes at the same time without breaking a sweat. (This is your Worker/API).
    
- **C++ is an F-22 Fighter Jet:** It only does one thing, but it does it with explosive, bare-metal speed that nothing else can catch. (This is your Load Generator).
    

You _could_ technically write a basic load generator in Node.js (tools like `autocannon` do this), but it would cap out at maybe 10,000 requests per second. A single C++ Boost thread will easily crush 100,000+ requests per second, completely exposing the flaws in the contestants' code.


---
### API
Every message must end with a newline `\n`.
**From Load Generator to User Container (The Attack):**
- **Add Order:** `[Order_ID] ADD [Side] [Ticker] [Price] [Quantity]`
    - _Example:_ `1001 ADD BUY AAPL 150 10\n`
- **Cancel Order:** `[Order_ID] CANCEL`
    - _Example:_ `1001 CANCEL\n`

**From User Container to Load Generator (The Feedback):**
- **Acknowledge (ACK):** `[Order_ID] ACK`
    - _Meaning:_ The order was successfully parsed and placed in the order book.
- **Reject:** `[Order_ID] REJ [Reason]`
    - _Meaning:_ Malformed order (e.g., negative price).
- **Fill (Trade):** `[Order_ID] FILL [Match_ID] [Matched_Price] [Matched_Quantity]`
    - _Meaning:_ An execution occurred. If an order is partially filled, this message is sent for the partial amount, and the rest remains in the book.



Why mirror in Codeforces ? 
If there is a wrapper then what is a point of bots ?
Time measurement 


