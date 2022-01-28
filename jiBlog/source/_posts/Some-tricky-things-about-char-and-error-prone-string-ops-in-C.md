---
title: Some detailed things in C (1)
date: 2022-01-28 20:03:38
tags: detail
categories: C 
---

# detailed error-prone things in C (1st)
---

As we all know, in C programming language, functions in *string.h* is very error-prone! In this post, I show something that might be in your way when you are manipulating string functions.

---

## static keyword

prologue about initialized value:
* global or static
  * allocated in static field
    * live with process and is 0
* local and not static
  * on stackframe -> random

#### static on local variable
the static local variable declared in a function is **initialized when the process started, and always initialized to 0**, not **when function is called**.

the static local variable resides at the beginning of process's address space (static field), and **only destructed when process exists**

```C
void function()
{
    static int foo; // static variable are initialized to zero
    printf("call %d\n", foo++);
}

int
main()
{
    int i;
    for (i = 0; i < 3; ++i)
        function();
    return 0;
}

outputs: 
call 0
call 1
call 2
```

#### static on global variable
global variable itself is saved in static field, so *static* keyword does not change its position in memory.

However, it restricts its link-attribute. The global variable declared with *static* **can only be accessed by the files which included the file where the variable is declared**

```C
// a.h
# pragma once
static int a = 1;

// b.h
#include "a.h"
void func()
{
    printf("%d\n", a);
}

// a.c
#include "a.h"
#include "b.h"

int
main()
{
    func();
    printf("%d\n", a);
    return 0;
}

outputs:
2
2
```

#### static on function
function declared with *static*, **can only be accessed by the file which defines it**

That is, you have to declare and implement the static function within one file.


---

## string literals (very error-prone!!!)

[StackoverFlow-String literal problems](https://stackoverflow.com/questions/5717176/bus-error-while-running-a-simple-string-c-program)

Check out the code here (error code)
```C
#include <stdio.h>
#include <string.h>

int
main()
{
    char *s = "this is ";
    char *s1 = "me";  
    strcat(s,s1); 
    printf("%s",s);
    return 0;
}
output: bus error in MACOS
```

Why there is an error, the code seems to be fine!

This is because string in this form `"this is "`, is a **string literal**.

The string literal is allocated at the program startup and held until the program exits. The **key problem** is, the places where string literal resides **may or may not be writable**, so modify a string literal is a **undefined behavior**.

This part of code has two problems.
* Buffer overflow
  * s has length 9, s1 has length 3. And the code is trying to write 11 character in a 9 character buffer.
  
* The string literal is not writable
  
The right way to do this is to **copy the literal onto the stackframe**
```C
char *s = "this is ";
char *s1 = "me";
char target[strlen(s) + strlen(s1) + 1];

strcpy(target, s);
strcat(target, s1);
// or sprintf(target, "%s%s", s, s1);

// The recommended way (on heap)
char *target = malloc(strlen(s) + strlen(s1) + 1);
...
free(target);
```

---

## sizeof()

sizeof is a very error-prone function! The value of sizeof is determined when **compiling, or runtime**.

One of the most problematic things about *sizeof()* is `sizeof(<some_ptr>)`, here are some examples.
```C
char* p = "Hello world"
printf("%lu", sizeof(p))
```
The output will be 8 in 64-bit word machines and 4 in 32-bit word machines.
You probably already notice why, because the **pointer is actually a address!**. And in a 64-bit word machines, addresses are 8-byte long.

### sizeof(ptr) and sizeof(array)
Please notice the difference between `sizeof(<some_array>)` and `sizeof(<some_ptr>)`

The code below will show clearly the difference
```C
#include <stdio.h>

void bar(char f[3]) {
	printf("%lu\n", sizeof(f));
}

int
main()
{
	char foo[3];
	printf("%lu\n", sizeof(foo)); // 3
	bar(foo);                     // 8
	return 0;
}
```
In the first `sizeof(foo)`, the compiler knows the array is three-elem long, so the value is 3.

In the second sizeof(f), the compiler treats the `f` as a pointer. This is because when we pass an array to a function, we are actually passing the first-elem's address of the array (i.e. a pointer).

### sizeof(struct)
when using sizeof() on a struct, it returns the actual space occupied by the structure.

In C, the struct are **aligned**.

For example, assume this is on a **32bit word machine**
```C
struct mystruct {
    char c1;
    int  i1;
    char c2;
} s1;

printf("%lu", sizeof(s1));
```

This will print `12`, because `c1` itself occupies a word, i1 occupies the next and c2 occupies the next. So in total three words (12 bytes) are occupies by s1.

---
