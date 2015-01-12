---
layout: post
title:  "Mutating Functions"
date:   2014-05-01 22:02:44
categories: update c++
tags: c++ 
author: brian
---

Recently, I got tired of dealing with out parameters. I am not trying to convince anyone not to use them, but at times it is just nice to have values returned directly from the function.

As seen below, output parameters can just be cumbersome.

{% highlight c++ %}
double x0, x1;
obj.get_x0(x0);
obj.get_x1(x1);

double midpoint;
get_midpoint(x0, x1, midpoint);
{% endhighlight %}

The above code can be greatly simplified using return values, as seen here:

{% highlight c++ %}
double midpoint = get_midpoint(obj.get_x0(), obj.get_x1());
{% endhighlight %}

So I set out on a mission, I must defeat the output parameter and make it submit to my will. My goal was simple, create a way to turn output parameters into return values. Not only could I hopefully end up with a useful tool, but I could improve my understanding of templates.

As usual, the full code can be found [here](https://github.com/brianrackle/to_out){:target="_blank"}

The plan is to use templating to hide the few lines of code necessary for using output parameters. The interface for the template will expose all parameters of an out function, and making the final parameter, the output parameter the return value of the templated function. Finally, I wanted to focus on converting methods rather than stand alone function since I personally encounter output parameters in methods often. Also, I am limiting the scope to handle only output parameters defined using referencing as so `get_x(double & x);`.

The first pass at the problem was simple and produced a clumsy and inflexible solution. Consisting of two template functions and a macro already this code is less than ideal:

{% highlight c++ %}
//O object type, F function pointer type, output parameter type
template<class O, class F, class P>
P to_out(O & obj, F func)
{
  P out;
  (obj.*func)(out);
  return std::move(out);
};

#define t_o(O, F, P) to_out<decltype(O), decltype(F), P>(O, F)
{% endhighlight %}

The usage of this code is as follows:

{% highlight c++ %}
t_o(obj,&decltype(obj)::get_x0, double);
{% endhighlight %}

Let's go through the problems with this implementation.

- Template type deduction is not possible because the `P` template parameter is explicit.
- A macro is needed to hide the specification of the template parameters.
- Support for only functions with a single parameters.

To fix these problems, it is necessary to allow for template type deduction of the method's output parameter. To accomplish this we need a traits class that works on methods. The basic ideas for a method traits are covered [here](https://functionalcpp.wordpress.com/2013/08/05/function-traits/){:target="_blank"}. The basic implementation that works with methods that have a single parameter is as follows:

{% highlight c++ %}
template<class T> struct member_function_traits;
template<class T, class R, class A>
struct member_function_traits<R (T::*)(A)>
{
using class_type = T;
using return_type = R;
using arg_type = A;
};
{% endhighlight %}

Types `T`, `R`, and `A` are deduced through the `member_function_traits` type specialization `<R (T::*)(A)>`. These types are then exposed publically with type aliasing, giving `class_type`, `return_type` and `arg_type` aliases.

Now it is possible to remove the ugly macro and rely on template type deduction to give us the same information that was previously required to be explicitly defined. Doing something like this:

So on to the next implementation!

{% highlight c++ %}
//O object type, F function pointer type
template<class O, class F>
P to_out(O && obj, F func)
{
  std::remove_reference<member_function_traits<F>::arg_type>>::type out;
  (obj.*func)(out);
  return std::move(out);
};
{% endhighlight %}

This implementation saves us several problems. First, thanks to [universal references](http://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers) we only need to define a single `to_out` function that can handle l-value and r-value references. And universal references are available because template type deduction is being used. And template type deduction is being used because the method's output parameter type is no longer defined through the template, but through examining the `F func` parameter using our `member_function_traits` class.

The final piece to the puzzle is allowing multiple parameter functions. In some cases, functions with output parameters will require input; for example, `get_name(int ssn, std::string & name);`. So `member_function_traits` needs to be elaborated on to allow the deduction of all parameters of a method.


{% highlight c++ %}
  template<class T> struct mft;
  template<class T, class R, class... A>
  struct mft<R (T::*)(A...)>
  {
    using class_type = T;
    using return_type = R;

    static constexpr std::size_t arity = sizeof...(A);

    template <std::size_t N>
    struct arg
    {
      using type = typename std::tuple_element<N, std::tuple<A...>>::type;
    };
  };
{% endhighlight %}

Here, a few techniques come into play. First, variadic templates allow our member function traits class `mft`to accept a function with any number of arguments. There are a few notable syntaxes here that are similar.  The syntax for declaring a variadic template parameter is `class... A`, seen in `template<class T, class R, class... A>`. The next syntax, for declaring a parameter pack, which is necessary because we are specializing our variadic template is, `A...`, seen in `struct mft<R (T::*)(A...)>`. Going down the code line by line the next related syntax is `sizeof...(A)`; which expands the templat parameter pack and gives the count. Finally, the last related syntax is `std::tuple<A...>, which expands the parameter pack `A`, for the template `tuple`. I will expand on the topic of variadic templates in another blog post, but the above code block highlights most of the syntaxes needed for parameter packs.

Beyond explaining the various syntaxes for understanding variadics, one should also understand the over purpose of the code. From the previous `member_function_traits` implementation, `A` has been replaced by a variadic, `arity` has been added to keep a count of the number of parameters the method takes, and `arg` is now to be used in place of the type alias `arg_type`. And `arg` parameter types are looked up using the `N` template parameter.

The individual parameter type is looked up by constructing a tuple type using parameter pack expansion and then looking up the type of the `N`th tuple element with the handy `std::tuple_element`.

Now that the method traits class has been fully realised, the `to_out` function can be improved to take functions with a variable number of arguments:

{% highlight c++ %}
  template <class T, std::size_t N> using mft_arg_t = 
    typename mft<T>::template arg<N>::type;

  template <class T> using rr = typename std::remove_reference<T>;

  template<class O, class F, class... A >
  auto to_out(O && obj, F func, A... args)
  {
    constexpr auto arity = mft<F>::arity;

    typename std::remove_reference<mft_arg_t<F, arity - 1>>::type out_param;

    (obj.*func)(args..., out_param);
    return std::move(out_param);
  };
{% endhighlight %}

`to_out` is a template function that uses that amazing template type deduction that we want so dearly. The final function parameter `A... args` defines a function parameter pack, and as noted earl before, `class... A` defines the template paramter pack. However, there is one gotcha in this code on this line:

{% highlight c++ %}
  template <class T, std::size_t N> using mft_arg_t = 
    typename mft<T>::template arg<N>::type;
{% endhighlight %}

You might noticde the keyword `template` appears twice. Having a look at ISO C++03 14.2/4 might clear things up, or make things more confusing:

> When the name of a member template specialization appears after . or -> in a postfix-expression, or after nested-name-specifier in a qualified-id, and the postfix-expression or qualified-id explicitly depends on a template-parameter (14.6.2), the member template name must be prefixed by the keyword template. Otherwise the name is assumed to name a non-template.

In this case we are using a alias template to access `arg<N>` is part of a "nested-name-specifier" and relies on the template parameter N, so we need to use the template keyword.

## Conclusion

So that wasn't so much work but did we get anything useful out of that project? I am not so sure, but I am leaning towards no. The syntax for converting output parameters to return values just plain sucks.

{% highlight c++ %}
get_midpoint(to_out(obj, &Type::get_x0), to_out(obj, &Type::get_x1));
{% endhighlight %}

Nesting calls results in an extremely difficult to read mess. Getting single values is a little more understandable and maybe with a better name than `to_out` it might even be understandable, but it is still an excessive amount of code. The main problem is that the function arguement needs to be defined in terms of the arguement for the object. I tried various trickery but couldn't figure out a way, when dealing with methods, to mkae it work without `&Type::`. In all, an interesting experiment, however the results leave me wanting more. 

Ultimately, this project leads me to question the plausability and usefulness of modifying functions through templates. Of course the practice of wrapping functions is common and useful, but is there a case for a generic wrapper class and if so what would it look like?