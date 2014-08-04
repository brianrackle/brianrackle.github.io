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
clock_t begin, end;
begin = clock();
// task goes here...
duration = (double)(clock - begin) / CLOCKS_PER_SEC;
{% endhighlight %}

The code isn't hard but can get repetitious. Thankfully, C++11 introduces a better way to manage date and time with the, `<chrono>` header. Useful for timing, STL introduces the following:

* `std::chrono::high_resolution_clock`
  * a clock with the smallest tick period available.
  * tick rate may change (depends on the value of `is_steady`)
* `std::chrono::system_clock`
  * a clock that represents the system real time wall clock
  * tick rate may change (depends on the value of `is_steady`)
* `std::chrono::steady_clock`
  * a monotonic clock, meaning that time can only increment
  * tick rate doesn't change

Only `std::chrono::steady_clock` is guaranteed to give a stable tick rate. This is often important when timing because accuracy of the result compared to wall time is not as important as accuracy of the result when comparing to other things that have been timed. Let's say you want to compare the performance of two algorithms. Here are your results

#### Unstable Tick Rate
| Algorithm | Timed Duration | Real Duration |
|:--- |:---:|:---:|
| algo_1 | 3000 milliseconds | 3000 milliseconds |
| algo_2 | 3400 milliseconds | 2000 milliseconds |

Wow, something went seriously wrong here. We timed "algo_2" as being slower than "algo_1", but in reality it was 1/3 faster. How did this happen? Well, in the middle of processing "algo_2" the system changed the clock of the cpu, perhaps for thermal reasons, and with that change the tick rate changed as well. So now each, millisecond was happening faster than when we ran "algo_1". Over the long run the system will be able to adjust the tick rate with the changes in clock speed and can provide an accurate wall clock time. That changing tick rate, just doesn't help us when comparing two durations.

#### Stable Tick Rate
| Algorithm | Timed Duration | Real Duration |
|:--- |:---:|:---:|:---:\
| algo_1 | 3050 milliseconds | 3000 milliseconds |
| algo_2 | 2033 milliseconds | 2000 milliseconds |

What happened this time? We correctly timed "algo_2" as being 1/3 faster than "algo_1" however, the timed duration no longer matches the real duration. This time, the tick rate never changes, however it is off by a very small amount so the timed duration slowly drifts away from the real duration. When comparing times, the real duration isn't that important, we care more about the accuracy of times when compared to each other.