---
layout: post
title:  "Easy Timing"
date:   2014-08-03 22:02:44
categories: update c++ stl
tags: c++ 
author: brian
---

Timing in C++ is annoying. There are so many different ways to accomplish timing, all of which involve C functions. Also, timing tends to clutter code with blocks like:

{% highlight c++ %}
clock_t begin = clock();
// task goes here...
double duration = (double)(clock - begin) / CLOCKS_PER_SEC;
{% endhighlight %}

The code isn't hard but can get repetitious. Thankfully, C++11 introduces a better way to manage date and time with the, `<chrono>` header. Useful for timing, STL introduces the following:

* `std::chrono::high_resolution_clock`
  * a clock with the smallest tick period available.
  * tick rate may change (depends on the value of `is_steady`)
* `std::chrono::system_clock`
  * a clock that represents the system real time wall clock
  * tick rate may change (depends on the value of `is_steady`)
* `std::chrono::steady_clock`
  * a monotonic clock, meaning that time can only increase
  * tick rate doesn't change

Only `std::chrono::steady_clock` is guaranteed to give a stable tick rate. When comparing times, a stable tick rate is exactly what is needed to get a reliable assessment of how various durations relate. 

## Timing Things
Let's say you want to compare the performance of two algorithms. Here are the results

#### Unstable Tick Rate

| Algorithm | Timed Duration | Real Duration |
|:--- |:---:|:---:|
| algo_1 | 3000 milliseconds | 3000 milliseconds |
| algo_2 | 3400 milliseconds | 2000 milliseconds |

Wow, something went seriously wrong here. We timed "algo_2" as being slower than "algo_1", but in reality it was 1/3 faster. How did this happen? Well, in the middle of processing "algo_2", the system changed the clock of the cpu, perhaps for thermal reasons, and with that change the tick rate changed as well. Each tick started happening faster than when we ran "algo_1". Over the long run, the system will be able to adjust the tick rate with the changes in clock speed, and can provide an accurate wall clock time. But over small intervals, when comparing two durations, a changing tick rate will not give us a stable result.

#### Stable Tick Rate

| Algorithm | Timed Duration | Real Duration |
|:--- |:---:|:---:|
| algo_1 | 3050 milliseconds | 3000 milliseconds |
| algo_2 | 2033 milliseconds | 2000 milliseconds |

What happened this time? We correctly timed "algo_2" as being 1/3 faster than "algo_1" however, the timed duration no longer matches the real duration. This time, the tick rate never changes, however it is off by a very small amount so the timed duration slowly drifts away from the real duration. When comparing times, the real duration isn't that important, we care more about the accuracy of times when compared to each other. A stable tick rate gives us accuracy when comparing durations.

### Durations

C++11 also introduces `std::chrono::duration`. Duration is simply a value (count) and ratio that represents fractions of a second. You can create any duration you want.

{% highlight c++ %}
using namespace std::chrono;
duration<int,std::ratio(60 * 60 * 24, 1)> day(1);
cout << "The number of seconds in a day: " 
	<< duration_cast<duration::seconds>(day).count(); 
{% endhighlight %}

There are also the following built-in [durations](http://en.cppreference.com/w/cpp/chrono/duration):

* std::chrono::nanoseconds	
  * duration</*signed integer type of at least 64 bits*/, std::nano>
* std::chrono::microseconds	
  * duration</*signed integer type of at least 55 bits*/, std::micro>
* std::chrono::milliseconds	
  * duration</*signed integer type of at least 45 bits*/, std::milli>
* std::chrono::seconds	
  * duration</*signed integer type of at least 35 bits*/ >
* std::chrono::minutes	
  * duration</*signed integer type of at least 29 bits*/,std::ratio<60>>
* std::chrono::hours	
  * duration</*signed integer type of at least 23 bits*/, std::ratio<3600>>

## The Code

[The Full Solution](https://github.com/brianrackle/brainstem_breakfast/blob/master/BrainstemBreakfast/BrainstemBreakfast/time_it.hpp)

Even with the new `<chrono>` header in C++11, similar to C-style time measuring, we still need clunky code blocks to wrap the things we want to time. 

{% highlight c++ %}
using namespace std::chrono;
auto begin = steady_clock::now();
// task goes here...
auto duration = begin - steady_clock::now();
{% endhighlight %}

The C++ version is a little bit simpler than in C, but there is an solution that better fits our needs.

{% highlight c++ %}
template<class D = std::chrono::nanoseconds, class F>
inline D time_it(F && f)
{
	using namespace std::chrono;

	auto begin = steady_clock::now();
	f();
	auto end = steady_clock::now();
	return std::chrono::duration_cast<D>(end - begin);
}
{% endhighlight %}

Leveraging templates, I have built the time_it function which takes a lambda expression and wraps it in a timing block. Additionally, time_it will cast the result to the passed in duration, defaulting to `std::chono::nanoseconds`. This solution gives us two advantages over the previously presented solutions. First, we have removed tedious timing code from our function. Second, clock choice and duration interval are now independent of each other. So even if a clock defaults to nanoseconds, it is possible to get results in seconds.

### The Output

Here is a simple test of the new time_it function.

{% highlight c++ %}
using namespace std::chrono;
auto result = time_it < milliseconds >([]{ 
	for (size_t i = 0; i < 100000 * 1000; ++i) 
		std::pow(i, 2); });
std::ofstream outfile("outfile.txt");
outfile << "milliseconds: " + std::to_string(result.count());
{% endhighlight %}

milliseconds: 7939

Success! I hope you have learned how to more effectively and accurately time things in C++11. And that I have helped you understand how you can engineer better solutions to common timing problems.