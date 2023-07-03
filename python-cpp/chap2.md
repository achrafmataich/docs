**Chapter 2 : Use C++ classes and map them in python class**

# Table of content

- [Table of content](#table-of-content)
- [Introduction](#introduction)
- [Creating a C-Compatible Interface](#creating-a-c-compatible-interface)
  - [Definition for the class](#definition-for-the-class)
  - [Create the wrapper](#create-the-wrapper)
- [Create a mapping of the C++ class in Python](#create-a-mapping-of-the-c-class-in-python)

# Introduction

This chapter provides a comprehensive guide on how to utilize C++ classes in Python by leveraging the `ctypes` library. By combining the power of C++ with the flexibility of Python, you can harness the benefits of both languages in your projects.

When working with C++ code, it's common to have existing classes that encapsulate complex functionality. By using `ctypes`, you can create a bridge between C++ and Python, allowing you to interact with C++ classes and their methods from within Python code.

In this chapter, we'll explore the step-by-step process of integrating C++ classes into Python. We'll cover the following topics:

1. **Creating a C-Compatible Interface:** We'll start by creating a C-compatible wrapper for the C++ class. This involves defining C functions that can be accessed from Python and act as intermediaries to the corresponding C++ class methods.

2. **Compiling the C++ Code:** We'll compile the C++ code along with the wrapper functions into a shared library, which can be loaded by Python using `ctypes`.

3. **Mapping C++ Classes to Python Classes:** We'll demonstrate how to map C++ classes to Python classes using `ctypes`. By defining Python classes that mirror the structure and behavior of the C++ classes, we can provide a seamless integration between the two languages.

By following the steps outlined in this chapter, you'll gain the ability to leverage existing C++ classes within your Python codebase. This integration opens up a world of possibilities, enabling you to utilize high-performance C++ functionality while enjoying the simplicity and flexibility of Python.

Let's dive in and discover how to unlock the power of C++ classes in Python using `ctypes`!

# Creating a C-Compatible Interface

`ctypes` is known to use only C functions and types, that means that we cannot create a C++ class and use it directly in python using `ctypes`.

To resolve this issue, we will create a wrapper.

we will try to create a c++ class named __*Friend*__

## Definition for the class

let's create a `friend.hpp` file.

```cpp
class Friend
{
    private:
        const char* f_name;
        int f_age;

    public:
        Friend(const char*, int);
        void sayHello();
};
```

This is a basic class declaration.

Now let's define the methods of this class in `friend.cpp` file.

```cpp
#include <iostream>
#include "friend.hpp"

using namespace std;

Friend::Friend(const char* name, int age): f_name(name), f_age(age) {}

void Friend::sayHello()
{
    cout << "Hello friend! My name is " << this->f_name << ", my age is " << this->f_age << endl;
}
```

## Create the wrapper

The wrapper is a C++ to C mapping so the ctypes use only pointers and C functions.

Let's begin with the `wrapper.h` file.

```cpp
#ifdef __cplusplus
extern "C" {
#endif

    typedef void* VP; // void pointer

    VP create_friend_instance(const char*, int);
    void destroy_friend_instance(VP);
    void call_friend_sayHello(VP);

#ifdef __cplusplus
}
#endif
```

As you can see, we refer to the class type by a void pointer so we can always have access to the instance by its address in the memory, the address will be stored in a python variable, we will see how we can do that later on this doc.

Now let's define these functions in a `wrapper.cpp` file.

```cpp
#include "wrapper.h"
#include "friend.hpp"

VP create_friend_instance(const char* name, int age)
{
    return new Friend(name, age);
}

void destroy_friend_instance(VP handler)
{
    delete static_cast<Friend*>(handler);
}

void call_friend_sayHello(VP handler)
{
    Friend* friendInstance = static_cast<Friend*>(handler);
    friendInstance->sayHello();
}
```

Now we have a well-structured C-compatible interface that handles all the logic of our C++ class.

Here are our files created until this moment:

- friend.cpp
- friend.hpp
- wrapper.cpp
- wrapper.h

Now let's wrap all this in a shared library.

```bash
g++ -shared -o friend.so -fPIC wrapper.cpp friend.cpp
```

# Create a mapping of the C++ class in Python

Mapping the C++ class in a Python class let the coding process be more user-friendly, because the `.so` files read by the `ctypes.CDLL` method don't let the intellisense of the IDE detects the functions wrapped inside.

It is our duty, as the library makers, to define all the methods created in C++ to be known by a python user of the library.

To do so, let's first define all the methods created in `wrapper.cpp` using `ctypes` in our Python file that we will call `friend_c_loader.py`.

```python
import ctypes

# reference to the shared library
friend_lib = ctypes.CDLL("friend.so")

# reference to the functions of the shared library
cr_friend = friend_lib.create_friend_instance
de_friend = friend_lib.destroy_friend_instance
ca_friend_say_hello = friend_lib.call_friend_sayHello

# define args data types and return data type
cr_friend.argtypes = [ctypes.c_char_p, ctypes.c_int]
cr_friend.restype = ctypes.c_void_p

de_friend.argtypes = [ctypes.c_void_p]
de_friend.restype = None

ca_friend_say_hello.argtypes = [ctypes.c_void_p]
ca_friend_say_hello.restype = None
```

Now let's create a class named __*Friend*__ in `friend.py`.

```python
from friend_c_loader import *

class Friend:

    def __init__(self, name: str, age: int) -> None:
        self.__name_b = ctypes.c_char_p(name.encode("utf-8"))
        self.__name = name
        self.__age = age
        self.__this_c_instance = cr_friend(self.__name_b, age)
    
    def say_hello(self):
        ca_friend_say_hello(self.__this_c_instance)

    def __del__(self):
        de_friend(self.__this_c_instance)
```

We have everything set up, all we have to do is test this.

We will create a `__main__.py` file.

```python
from friend import Friend

f1 = Friend("Achraf", 24)
f2 = Friend("Mataich", 23)

f1.say_hello()
f2.say_hello()
```

When we run this code we got this result:

```
Hello friend! My name is Achraf, my age is 24
Hello friend! My name is Mataich, my age is 23
```