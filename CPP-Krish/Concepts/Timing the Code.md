One way  to time your code to see how long it takes to run. C++11 comes with some functionality in the chrono library for this. However, using the chrono library is a bit arcane (understood only, ny a few people). But  we can encapsulate all the timing functionality we need into a class that we can then use in our own programs.

Here’s the class:

```cpp
#include <chrono> // for std::chrono functions
class Timer {
private:
	using Clock = std::chrono::steady_clock;
	using Second = std::chrono::duration<double, std::ratio<1> >;
	std::chrono::time_point<Clock> m_beg { Clock::now() };

public:
	void reset() { m_beg = Clock::now(); }
	double elapsed() const {
		return std::chrono::duration_cast<Second>(Clock::now() - m_beg).count();
	}
};
```

Example: 

```cpp
int main() {
    Timer t;
    std::cout << "Time elapsed: " << t.elapsed() << " seconds\n";
}
```

---
### Things that can impact the performance of your program

First, make sure you’re using a release build target, not a debug build target. Debug build targets typically turn optimization off, and that optimization can have a significant impact on the results. 

Second, your timing results may be influenced by other things your system may be doing in the background. Make sure your system isn’t doing anything CPU, memory, or hard drive intensive. Seemingly innocent things, like idle web browsers, can temporarily spike your CPU to 100% utilization when the active tab rotates in a new ad banner and has to parse a bunch of javascript. The more apps you can shut down before measuring, the less variance in your results you are likely to have.

Third, if your program uses a random number generator, the particular sequence of random numbers generated may impact timing. For example, if you’re sorting an array filled with random numbers, the results will likely vary from run to run because the number of swaps required to sort the array will vary from run to run. To get more consistent results across multiple runs of your program, you can temporarily seed your random number generator with a literal value (rather than std::random_device or the system clock), so that it generates the same sequence of numbers with each run. However, if your program’s performance is highly dependent on the particular random sequence generated, this can also lead to misleading results overall.

---
### Measuring performance

When measuring the performance of your program, gather at least 3 results. If the results are all similar, these likely represent the actual performance of your program on that machine. Otherwise, continue to take measurements until you have a cluster of similar results. 

If your results have a lot of variance (and aren’t clustering well), your program is likely either being significantly affected by other things happening on the system, or by the effects of randomization within your application.

Because performance measurements are impacted by so many things (particularly hardware speed, but also OS, apps running, etc…), absolute performance measurements are not useful. On a different machine, that same program may run in significantly different time.  It’s hard to know without actually measuring across a spectrum of different hardware.

However, on a single machine, relative performance measurements can be useful. We can gather performance results from several different variants of a program to determine which variant is the most performant.

After measuring the second variant, a good sanity check is to measure the first variant again. If the results of the first variant are consistent with your initial measurements for that variant, then the result of both variants should be reasonably fair. However, if the results of the first variant are no longer consistent with your initial measurements for that variant, then something has happened on the machine that is now affecting performance, and it will be hard to tell whether differences in measurement are due to the variant or due to the machine itself. In this case, it’s best to discard the existing results and re-measure.

---
