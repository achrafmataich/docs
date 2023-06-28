**Chapter 1 : Use The power of C++ in Python**

## Table of content

- [Table of content](#table-of-content)
- [Introduction](#introduction)
- [Using __ctypes__ library for python](#using-ctypes-library-for-python)
  - [Creating C++ file](#creating-c-file)
  - [Compiling C++ file to be a shared C++ file](#compiling-c-file-to-be-a-shared-c-file)
  - [Use it inside python](#use-it-inside-python)
  - [Execution](#execution)

## Introduction

Python is known to be a slow language.  
When it came to fast-processing data, the best way to do so is to use a compiled programming language like **C/C++**.  
>_But what if we use  both the performance of C/C++ and the simplicity of Python?_

## Using __ctypes__ library for python 

__ctypes__ is a python library that handles the import and the use of C/C++ functions and classes inside python code.

### Creating C++ file

```cpp
#include <iostream>

using namespace std;

extern "C++"
{
    void sayHello()
    {
        cout << "Hello" << endl;
    }
}
```

Here's the breakdown of this file:

```cpp
#include <iostream>
```

this is the preprocessing directives (here we imported headers of input/output methods).

```cpp
using namespace std;
```

Just so we don't use std prefix of the standard namespace.

```cpp
extern "C++"
{
    // definitions here
}
```

Everything between the brackets of this code is declared to be used outside the file we defined inside.  
It is often used when mixed with assembly code but in our case it gonna be with python.

```cpp
void sayHello()
{
    cout << "Hello" << endl;
}
```

This function say hello by printing it to the standard output.

### Compiling C++ file to be a shared C++ file

```ps
g++ -fPIC -shared -o mylib.so mylib.cpp
```

### Use it inside python

```python
import ctypes

c_lib_handler = ctypes.CDLL("mylib.so")

c_lib_handler.sayHello()
```

### Execution

When the python file is executed it shows this output

```
Hello
```