---
layout: post
title:  "Range Mapping and Projecting Values"
date:   2014-08-18 22:10:00
categories: update c++
tags: c++ 
author: brian
---

Mapping values from one range to another is a common need in certain problem sets. Simply getting a percent of a total is a basic form of range mapping. You have values in a given range, and you need to translate those values into a range from 0 to 1. Let's see if we can generalize the operation to map any range to any other range.

## The Basics

To start, it is important to understand the easiest form of range mapping, finding a percent. Finding a percent in a range starting from zero is actually the easiest starting point. This is easy because both `range_to` and `value` represent their distance from `range_from`. Meaning, if we have a range from 0 to 255 then 255 is `range_to`, and it is also the distance from `range_from`. Distance from the origin is important, because it allows us to deduce the total percentage within the range. 

Given a value of 32, and a range of 0 to 255. The following equations will give 32 as a percent of 255:

$$ percent = \frac{value}{range1} $$

$$ percent = \frac{32}{255} $$

$$ percent = 0.1254901960 $$

It is now possible to use `percent` to project into a new range. Given a second range, 0 to 511 use the following equations to project the previously calculated `percent` into that range:

$$ projected = percent \times range2 $$

$$ projected = 0.1254901960 \times 511 $$

$$ projected = 64.125490156 $$

The previous equations can be combined into the following equation:

$$ projected = \frac{value}{range1} \times range2 $$

## Generalizing

The math is very easy. All that is needed is to generalize it to work with any pair of ranges. First, it is necessary to allow ranges that start at any number, not just 0. This is done by first shifting the range over to start at 0 and then shifting back after projecting using the following modification to our previous equation:

$$ projected = \frac{value - range1from}{range1to - range1from} \times (range2to - range2from) + range2from $$

$$ projected = \frac{(value - range1from) \times (range2to - range2from)}{range1to - range1from} + range2from $$

$$ projected = \frac{(32 - 0) \times (511 - 0)}{255 - 0} + 0 $$

$$ projected = 64.125490196 $$

## The Code

[The Full Solution](https://github.com/brianrackle/brainstem_breakfast/blob/master/BrainstemBreakfast/BrainstemBreakfast/range_map.hpp)

Using the above generalized equation it is possible to write the following generalized algorithm:

{% highlight c++ %}
template <class _FT, class _TT>
_TT scale_value(_FT value, _FT lowestFrom, _FT highestFrom, _TT lowestTo, _TT highestTo)
{
  return ((highestTo - lowestTo) * 
    (value - lowestFrom) / (highestFrom - lowestFrom)) + lowestTo;
}
{% endhighlight %}

### Results

Running the above code with some sample data, this is the output:

| Value |lowestFrom | highestFrom | lowestTo | highestTo | Return Value|
|:---:|:---:|:---:|:---:|:---:|:---:|
|-100 | -100 | 0 | 100 | 200 |  100 |
| 2147483647 | -2147483648 | 2147483647 | -1.79769e+308 | 1.79769e+308 | <span style="color:#ff0000;">1.#INF0e+000</span> |

Well something obviously went wrong. Breaking apart the equation makes it possible to understand where the calculation went wrong. The debugged values are commented underneath their lines.

{% highlight c++ %}
template <class _FT, class _TT>
_TT scale_value(_FT value, _FT lowestFrom, _FT highestFrom, _TT lowestTo, _TT highestTo)
{
  _FT v_delta = value - lowestFrom; 
  //1.#INF000000000000 = 1.7976931348623157e+308 - -1.7976931348623157e+308

  _TT to_range = highestTo - lowestTo; 
  // -1 = 2147483647 - -2147483648

  _FT from_range = highestFrom - lowestFrom; 
  // 1.#INF000000000000 = 1.7976931348623157e+308 - -1.7976931348623157e+308
}
{% endhighlight %}

Now it is obvious what happened. When we subtract lowest from highest we end up with a value larger than the data type can hold. 

### The Fix

The solution here is to scale the from-range and to-range so that computing the total range will not overflow the data type. It would be possible to simply use an unsigned data type when only dealing with integral types, however floating-point types do not have unsigned versions. It is necessary to scale the values down so that `std::numeric_limits<T>::max() - std::numeric_limits<T>::lowest()` will not overflow `T`. This can be done by simply reducing all input values by half, performing the calculations and then expanding the result by double.

{% highlight c++ %}
template <class _FT, class _TT>
_TT scale_value(_FT value, _FT lowestFrom, _FT highestFrom, _TT lowestTo, _TT highestTo)
{
  //scale by half to account for negative and positive range being too large to represent
  const auto && fHLF = [](_FT v){ return v / 2; };
  const auto && tHLF = [](_TT v){ return v / 2; };

  auto scaledOffsetResult =
    ((tHLF(highestTo) - tHLF(lowestTo)) * 
    ((fHLF(value) - fHLF(lowestFrom)) / (fHLF(highestFrom) - fHLF(lowestFrom)))) * 2;

  return (_TT)(scaledOffsetResult + lowestTo); //seperated to prevent overflow
}
{% endhighlight %}

### The Final Code

Ok, I run the previous code and once again arrive at failure; the result is `1.#INF0e+000`. This time the problem occurs when expanding the projected value, `scaledOffsetResult`, by multiplying by 2. If we expand the value before adding `lowestTo` then we can overflow the data type. The solution is the expand after adding `lowestTo`. The full solution thus becomes:

{% highlight c++ %}
//scales 'value' from the range 'lowestFrom'/'highestFrom' to the range 'lowestTo'/'highestTo'
template <class _FT, class _TT,
class = typename std::enable_if<std::is_arithmetic<_FT>::value>::type,
class = typename std::enable_if<std::is_arithmetic<_TT>::value>::type>
_TT scale_value(_FT value, _FT lowestFrom, _FT highestFrom, _TT lowestTo, _TT highestTo)
{
  //scale by half to account for negative and positive range being too large to represent
  const auto && fHLF = [](_FT v){ return v / 2; };
  const auto && tHLF = [](_TT v){ return v / 2; };

  auto scaledOffsetResult =
    (tHLF(highestTo) - tHLF(lowestTo)) * 
    ((fHLF(value) - fHLF(lowestFrom)) / (fHLF(highestFrom) - fHLF(lowestFrom)));

  return (_TT)(scaledOffsetResult + lowestTo + scaledOffsetResult); //seperated to prevent overflow
}
{% endhighlight %}