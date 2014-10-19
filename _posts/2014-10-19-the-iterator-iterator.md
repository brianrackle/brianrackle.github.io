---
layout: post
title:  "The Iterator Iterator"
date:   2014-10-19 22:02:44
categories: update c++
tags: c++ iterator type traits meta programming stl
author: brian
---

So I was recently thinking about the code of an iterating for loop and why there is no functionality to retrieve an iterator with a range-based for loop. The range-based for loop was a hugely popular addition to C++, however it is limited to only returning dereferenced iterators. I don't think I need to convince anyone of why one would need an iterator rather than a dereferenced iterator, there are plenty of use cases to warrant such a loop. The issue here is that to iterate over a range using a classic for loop takes quite a lot of characters, even with the wonderful `auto it` that C++11 provides.

## The Problem

Range-based for loops work like this:

{% highlight c++ %}
//get a copy of each value in the container
for(auto value : container) { }

//get a reference of each value in the container
for(auto & v : container) { }
{% endhighlight %} 

But sometimes you need this:

{% highlight c++ %}
//get an iterator for each value in the container
for(auto it = container.begin(); it != container.end(); ++it) { }
{% endhighlight %} 

Ideally, it would work like this:

{% highlight c++ %}
//ideal syntax for getting an iterator for each value in the container
for_it(auto it : container) { }
{% endhighlight %}

I haven't mastered meta-programming, so adding a `for_it` definition is still on the horizon. So I propose this:

{% highlight c++ %}
//syntax I have settled for to get an iterator for each value in the container
for(auto it : iit(container)) { }
{% endhighlight %}  

## The Code

The full solution can be found [here](https://github.com/brianrackle/iit/){:target="_blank"}


### The Iterator

The code here is small but powerful. `itr` (the iterator iterator) is simply a container that holds a standard forward iterator. All operations work the same as most other forward iterators except that the iterator iterator dereference operator returns an iterator, not a dereferenced iterator.

{% highlight c++ %}
template <typename T>
class itr : public T
{
  T _obj;
 public:
    itr(T&& obj) : _obj(std::move(obj)) {}
    bool operator!=(const itr& other) const { return _obj != other._obj; }
    T operator*() const { return _obj; }
    const itr& operator++() { ++_obj; return *this; }
};
{% endhighlight %} 

This is a minimal implementation of only what is needed for range-based for loops. A proper implementation would implement [`std::iterator`](http://en.cppreference.com/w/cpp/iterator/iterator/){:target='_blank"} and also follow the rules of [`InputIterator'](http://en.cppreference.com/w/cpp/concept/InputIterator/){:target='_blank"}

In addition to the iterator class, a `make` function has been implemented. The idea here is to save the user from having to define the type of `itr` themselves, a similar idea to `make_tuple` and `make_shared`. My `make` function has been simply defined as `iit` to save some characters.

{% highlight c++ %}
template <typename T>
auto iit(T&& t) -> typename std::conditional<std::is_lvalue_reference<decltype(t)>::value,
                                 ref<T>,
                                 val<T>>::type
{
    return std::forward<T>(t);
}
{% endhighlight %} 

### Ref and Val

`iit` is defined around a universal reference (l-value or r-value) and a return type conditionally tied to if the type is l-value or r-value. This allows `iit` to accept the value by reference using the class `ref`, or move construct the type `val`. Primarily, the `ref` and `val` encapsulating containers provide the `begin` and `end` functions which return `itr`. Additionally, having a `ref` and `val` class allows `iit` to encapsulate the passed in container and keep an r-value alive through the lifetime of `ref`. Here are the definitions of `ref` and `val`:

{% highlight c++ %}
template <typename T>
class val
{
  T _obj;
 public:
 val(T&& obj) : _obj(std::move(obj)) {}

  auto begin() const -> itr<decltype(std::begin(this->_obj))> { return std::begin(_obj); }
  auto end() const -> itr<decltype(std::end(this->_obj))> { return std::end(_obj); }
};

template <typename T>
class ref
{
  T& _obj;
public:
 ref(T& obj) : _obj(obj) {}
  
  auto begin() const -> itr<decltype(std::begin(this->_obj))> { return std::begin(_obj); }
  auto end() const -> itr<decltype(std::end(this->_obj))> { return std::end(_obj); }
};
{% endhighlight %} 

#### Usage

{% highlight c++ %}
for(auto it : iit(vector<int>{1,2,3,4})) 
  std::cout << *it << std::endl;
{% endhighlight %}  

Without the `val` container to capture the r-value and store it, the vector would go out of scope.

{% highlight c++ %}
vector<int> v = {1,2,3,4};
for(auto it : iit(v)) 
  if(it == v.begin())
    std::cout << *it << std::endl;
{% endhighlight %}  

Here, v is sent to `ref` which only holds a reference to vector, no copies needed.

Thanks for reading!

Brian Rackle
