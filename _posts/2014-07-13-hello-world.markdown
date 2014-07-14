---
layout: post
title:  "Hello World!"
date:   2014-07-13 22:02:44
categories: update basics history
tags: c++ c history basics return int
---

Hello world! Rather cliché isn't it? Is there anything interesting about `HelloWorld`? Most would say no, but I am sure we can discover a few interesting bits of information about this mundane code. Let's analyze the most ubiquitous program ever written.

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

Int or Void
---
Many compilers support a `void` return value but it is not spec. Returning an `int` allows the program to let the operating system know if the program executed successfully. The only valid definitions for `main` are:

**3.6.1.2**

> int main() { /* ... */ }
> and
> int main(int argc, char* argv[]) { /* ... */ }

What to Return From Main
---
Any value can be returned from `main`. However, the only <i class="icon-share"></i>[spec defined value][exit_status] for returning success and failure is `EXIT_SUCCESS`/`zero` and `EXIT_FAILURE`. `zero` will be return automatically by the function is no return statement is defined.

It is also possible to exit the program at any point by calling `std::exit(EXIT_SUCCESS)` or `std::exit(EXIT_FAILURE)`. However, using `std::exit` should be done with caution because <i class="icon-share"></i>[destructors of variables with automatic storage durations are not called][exit], as the spec explains:

**18.5.3**

> The program is terminated without executing destructors for objects of automatic, thread, or static storage duration and without calling functions passed to atexit()

___

Also note that `main` is the only function that does not require a return statement despite deing defined with an `int` return value.

**3.6.2**

> If control reaches the end of main without encountering a return statement, the eﬀect is that of executing
> &nbsp;&nbsp;&nbsp;&nbsp;return 0;

Some compilers  allow omitting the `return` statement for any function. On `x86`, the value stored in the `eax` register is used to store the function `return` value. If the `return` value has not been set, then the value in `eax` remains undefined. 

The History of Return Values
---
Originally, `C` did not have a `void` type. Not having `void` meant that every function needed to define some `return` value. This lead to programmers adopting the convention of explicitly using `return 0;` on any function that did not require a return value.

Summation
---
If you made it here, I applaud you. Understanding specs is not the most exciting thing but hopefully you learned something and can use the information to become a better programmer.

See you next time!


[exit_status]: http://en.cppreference.com/w/cpp/utility/program/EXIT_status
[exit]: http://en.cppreference.com/w/cpp/utility/program/exit
