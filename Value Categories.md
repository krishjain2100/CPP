### **Values**
lvalue: something with a memory location.
rvalue: does not point anywhere.

```cpp
int a = 10;
int& ref = a;     // lvalue reference
int&& ref2 = 10;  // rvalue reference
int &ref3 = 10;   // Obviously this ain't allowed
```
  
References also exist for functions.

---
