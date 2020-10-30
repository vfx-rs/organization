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

TODO

SUGGESTION: Implement something like Rust's `Result<T, E>`, unless the wrapped library already implements error handling that we can inherit from in C.

```c
enum StatusCode {
    OK;
    ERROR;
};
typedef struct {
    StatusCode status;
    union value {
        MyType value;
        Error err;
    }
} MyTypeStatus;
```

Exceptions
==========

C does not support exceptions, and it's generally considered bad to allow exceptions to pass across the C++ to C boundary (TODO: Why?). So, if wrapped code can cause an exception, then it should be wrapped in code like below.

TODO: This may not catch everything.

```cpp
ThingStatus mything_do_thing(int thing) {
    ThingStatus status;

    try {
        Thing thing = mything::do_thing(int thing);

        status.status = StatusCode.OK;
        status.value = thing;

        return status
    } catch (const std::exception& err) {
        status.status = StatusCode.ERROR;
        status.err = err;
        return status;
    }
};
```

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
