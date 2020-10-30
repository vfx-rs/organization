The C wrappers should look like the C++ code as much as possible. But, there are going to be differences between the wrappers and the C++ code for many reasons.

- The projects we are wrapping usually have a style guide. Follow that, or make a best guess based on the code already written.
- Usually the projects use clang-format. It's highly recommended to also use clang-format, and run it whenever the file is saved.
- In the case of overrides, check if the override is a convenience override or not. If it is not, then ... (TODO: what guideline should be followed?)

Here are some examples of how to style the C code.

Functions
---------

### C++

```cpp
namespace mything {
std::string do_thing(int thing);
}
```

### C

```c
char* mything_do_thing(int thing);
```

Classes
-------

### C++

```cpp
namespace mything {
class MyClass {
    int thing;
};
}
```

### C

```c
typedef struct {
    int thing;
} mything_MyClass;
```

Class Methods
-------------

### C++

```cpp
namespace mything {
class MyClass {
    std::string do_thing(int thing);
};
}
```

```c
char* mything_MyClass_do_thing(MyClass *self, int thing);
```
