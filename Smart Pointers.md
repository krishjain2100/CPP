(in the memory library)

i) **Exclusive ownership:** exactly one owner at a time (e.g. std::unique_ptr). When that owner is destroyed, the resource is destroyed.

ii) **Shared ownership:** many owners may exist and the resource dies when the last owner goes away (e.g. std::shared_ptr).

iii) **Non-owning/observer:** pointers that refer to an object but do not participate in lifetime management (raw pointers, references, std::weak_ptr, observer_ptr).

```cpp
class Person {
public:
    Person() { cout << "Created" << endl; }
    ~Person() {cout << "Destroyed" << endl; }
};
```

---
### **std::unique_ptr()**

Anything allocated to the heap using 'new' has to be manually deleted, which is def not convenient 
  
``` cpp
int main() {
	Person p; 
} // Constructor and Destructor both are called
  
int main() {
	Person *p = new Person;
} // Only constructor is called

int main() {
	// Both are same
	unique_ptr <Person> p = unique_ptr<Person> (new Person);
	unique_ptr <Person> p2 = make_unique <Person> ();
}  // Constructor and Destructor both are called in both cases
```

Memory allocated using unique pointers is automatically freed when they go out of scope. Deleting or copying them is not allowed, but move is allowed

```cpp
unique <Person> mike = move(p); // valid
```

---
### **std::shared_ptr()**

```cpp
// Creating our shared pointer
shared_ptr <Person> ptr1 = make_shared <Person> ();
{
	// resource shared
	shared_ptr <Person> ptr2 = ptr1; // This is allowed now
	cout << ptr2.use_count() << endl; // 2
} // ptr2 is freed now
cout << ptr1.use_count() << endl; // 1
```

They do have a copy constructor and the destructor is called when the last shared pointer goes out of scope (or equivalently, the reference count becomes zero). A reference count for the resource created using this method is maintained and the destructor is called when the reference count becomes zero.

Memory leak is still possible:
```cpp
struct B; //forward declaraion

struct A {
    shared_ptr<B> b;
    ~A() { cout << "A destroyed" << endl; }
};

struct B {
    shared_ptr<A> a;
    ~B() { cout << "B destroyed" << endl; }
};
  
int main() {
    auto a = make_shared<A>(); 
    auto b = make_shared<B>();
    a->b = b;  
    b->a = a;  
    cout << "Leaving scope" << endl;
}
```

A points to B,  B points to A, no one's reference count becomes zero, so they never get freed.

---
  
### **std::weak_ptr()**
Non-owning pointer and does not increase reference count

	- ptr.expired(): checks whether the referenced object was already deleted
	
It is used to solve the cyclic dependency problem since it does not contribute to the reference count

---

