### 1. The Static Variable

**Behaviour:** It is created only **once** (the first time the function runs). It **never dies** until the program closes. It keeps its value between function calls.

```cpp
void counter() {
	static int count = 0; // Created ONCE. Skipped on future calls.
	count++;
	cout << count << endl;
} // 'count' is NOT destroyed. It sits in a special memory area.

int main() {
	counter(); // Prints 1
	counter(); // Prints 2
	counter(); // Prints 3
}
```

Its memory location is **static** (stationary/fixed). It sits in one spot in memory for the entire life of the program.

### 2. Static Inside a Class (The "Shared" Data)

When you put `static` on a member variable, it is no longer owned by the object. It is owned by the **Class itself**.

- **Normal Variable:** Every object gets its own copy.
- **Static Variable:** All objects share **ONE** copy.

**Common Use Case:** Counting how many objects exist.

```cpp
class Robot {
public:
	// 1. Declaration (Inside class)
	static int population;
	Robot() {
		population++; // Every time a robot is born, increase the SHARED counter
	}
};

// 2. Definition (OUTSIDE class - Required!)
int Robot::population = 0;

int main() {

	Robot r1;
	Robot r2;
	Robot r3;
	
	// They all share the same variable!
	cout << r1.population; // Prints 3
	cout << Robot::population; // Prints 3 (Better way to access it)
}
```

### 3. Static at File Scope (The "Hidden" Global)

If you create a global variable in `file1.cpp`, it is technically visible to `file2.cpp` if they try to look for it. This causes "Naming Collisions" (linker errors).

Adding `static` to a global variable makes it **invisible** to other files.

```cpp
// File1.cpp
static int secret_code = 1234; // Only File1 can see this.
```

---