---
layout: post
title:  "Hello World!"
date:   2014-07-13 22:02:44
categories: update c++
tags: c++ c history basics return int
author: brian
---

Hello world! Rather cliché isn't it? Is there anything interesting about Hello World? Most would say no, but I am sure we can discover a few interesting bits of information about this mundane code. Let's analyze the most ubiquitous program ever written.

{% highlight c++ %}
#include <iostream>
#include <cstdlib>
 
int main()
{
  std::cout << "Hello World!" << std::endl;
  return EXIT_SUCCESS;
}
{% endhighlight %}

Boring right? Maybe not! 

## Int or Void

Let's start with the basics, the return value. Returning an `int` allows the program to let the operating system know if the program executed successfully. And while many compilers support a `void` return value but it is not spec.  The only valid definitions for `main` are:

**3.6.1.2**

> int main() { /* ... */ }
> and
> int main(int argc, char* argv[]) { /* ... */ }

## What to Return

Any integer value can be returned from `main`. For success, return `EXIT_SUCCESS` or `0`. For failure, return `EXIT_FAILURE`. If you need multiple failure states, any non-zero integer can be used. Just remember to note what each failure state means.

It is also possible to exit the program at any point by calling `std::exit(EXIT_SUCCESS)` or `std::exit(EXIT_FAILURE)`. However, using `std::exit` should be done with caution because objects of local scope, thread local scope, or static scope will not have their destructors executed, as the spec explains:

**18.5.3**

> The program is terminated without executing destructors for objects of automatic, thread, or static storage duration and without calling functions passed to atexit()

___

It is also important to know that `main` is the __*only*__ function that does not require a return statement. When no return statement is defined, `0` is automatically returned by `main`. Some compilers  allow omitting the return statement for any function. To understand why this might be a problem let's look at how one cpu architecture handles return values. On *x86*, the return value if stored in the *eax* register. The *eax* register is also used for temporary storage because registers are fast and there are a limited number of them. This means that if the return value has not been set, then the value in *eax* remains undefined. So the function will also return an undefined value. This is not good if you want other people to use your functions. 

## The History of Return Values

So why is returning zero so common? Originally, *C* did not have a `void` type. Not having `void` meant that every function needed to define some return value. A common practice was to use an `int` return value when none was needed. And as we learned before, if we have a return value, we need to return a value, or noone will want to use our code. This lead to programmers adopting the convention of explicitly using `return 0;` on any function that did not require a return value. As time went on eventually `0` introduced the `void` type but it is still very common to see functions `return 0;`.

## Summation

If you made it here, I applaud you. Understanding specs is not the most exciting thing but hopefully you learned something and can use the information to become a better programmer.

See you next time!