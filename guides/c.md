Introduction
============

The C wrappers should look like the C++ code as much as possible. But, there are going to be differences between the wrappers and the C++ code for many reasons. When in doubt, refer to to the wrapped library's style guides, or make a best guess based on code already written in either the wrapped library, or other wrappers.

Formatting
==========

Use autoformatting where possible. Generally the wrapped libraries use clang-format, so it's best to use that. Also, it is suggested to do formatting on save.

Implementation
==============

- If there are function overrides, then prefer to wrap the core implementation. If there's multiple implementations, then try to pick a suffix that is clear and understandable.

Error Handling
==============

If a project provides mechanisms to return C compatible errors, then those should be used. However, there may be a case where a project's error handling is incompatible with C (such as exceptions), so we will need to create our own error handler. The following pattern is suggested for the case of those cases.

### C++

```cpp
std::thread_local std::string error_string;

int some_function_that_throws() {
    try {
        CPP::some_function_that_throws();
    } catch (const Exception1& e) {
        error_string = e.what();
        return 1;
    } catch (const Exception2& e) {
        error_string = e.what();
        return 2;
    }

    return 0;
}

const char* get_error() {
      return error_string.c_str();
}
```

Exceptions
==========

C does not support exceptions, and it's generally considered bad to allow exceptions to pass across the C++ to C boundary because it can lead to undefined behaviour. So, if wrapped code can cause an exception, then it should be wrapped in code like the example in [Error Handling](#Error-Handling).

TODO: This may not catch everything.

Static Asserts
==============

It is critical to make sure that the types passed through the C++ to C interface are always exactly the same if the type is non-opaque. So, when writing the bindings, there must be a static assert to validate that the C types are the same as the C++ types.

```cpp
// C++ class being wrapped
namespace mything {
class Thing {
    int a;
    int b;
    float c;
};
}

extern "C" {
// C wrapper
typedef struct {
    int a;
    int b;
    float c;
} mything_Thing;

static_assert (sizeof(mything_Thing) == sizeof(mything::Thing));
static_assert (alignof(mything_Thing) == alignof(mything::Thing));
static_assert (offsetof(mything_Thing, a) == offsetof(mything::Thing, a));
static_assert (offsetof(mything_Thing, b) == offsetof(mything::Thing, b));
static_assert (offsetof(mything_Thing, c) == offsetof(mything::Thing, c));
}
```

Examples
========

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

Classes with Public C Compatible Attributes
-------------------------------------------

### C++

```cpp
namespace mything {
class MyClass {
    public:
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

Classes with Private C Compatible Attributes
--------------------------------------------

### C++

```cpp
namespace mything {
class MyClass {
    private:
    int thing;
};
}
```

### C

```c
typedef unsigned char mything_MyClass[2];
```

Classes with C Incompatible Attributes
--------------------------------------

### C++

```cpp
namespace mything {
class MyClass {
    std::string thing;
};
}
```

### C

```c
struct mything_MyClass;
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

Function Overrides
==================

### C++

```cpp
namespace mything {
int do_thing(int thing);
std::string do_thing(std::string thing);
}
```

### C

```c
int mything_do_thing_int(int thing);
char* mything_do_thing_string(char* thing);
```
