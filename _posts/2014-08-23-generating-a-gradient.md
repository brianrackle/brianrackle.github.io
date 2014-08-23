---
layout: post
title:  "Generating a Color Gradient Part 1"
date:   2014-08-23 14:36:00
categories: update c++
tags: c++ math range gradient axial color theory interpolate projection 
author: brian
---

[<img src="/assets/heat_map_example.png" alt="heat map" width="200" style="float:right;">](http://onemilliontweetmap.com/)
The purpose of data visualization is to put information into a format that is easy to comprehend by the human brain. One way to visualize data is through color. Color is a very effective way to convey changes in data. While a random color palette is good for representing categorical data. Continuous data, such as temperature, time, or population is better represented with a continuous color palette, such as a gradient.

## Color Gradients

A color gradient is one very useful tool for representing changes in a continuous dataset. A properly chosen gradient will allow a viewer to quickly assess both small and large changes in data. Given that gradients are such a useful tool, it is handy to know how to generate a gradient.

A gradient is simply a range of colors determined by their position. For the purpose of this post I will just be covering one dimensional gradients, also known as axial gradients. Being one dimensional, the axial gradient is a simple linear interpolation of values between two points. Commonly, only two colors are interpolated between in axial gradient. Multiple color gradients also exist. For example, the rainbow gradient displays the entire rgb color range (given equal brightness and saturation).

## The Math

Calculating a two color gradient is very easy. For two color axial gradients, it is necessary to project a percent to a range determined by each of the components of an RGB color. For example, to build a gradient from the color <span style="color:#2aa35a;">rgb(42, 163, 90)</span> to color <span style="color:#cf4a21;">rgb(207, 74, 33)</span> there will be three different interpolations. The red component will be interpolated from 42 to 207. The green component will be interpolated from 163 to 74. The blue component will be interpolated from 90 to 33.

My previous post, ["Range Mapping and Projecting Values"](/update/c++/2014/08/18/range-mapping/), covers all the math needed to interpolate between ranges. To recap, given an input value from 0 to 1 and an output range from red component 42 to red component 207 (1 byte color) the following equation can be used to get the projected value:

$$ projected = \frac{(value - range1from) \times (range2to - range2from)}{range1to - range1from} + range2from $$

Given that our input range is clamped to the range 0 to 1, it is possible to simplify the equation to:

$$ projected = (value \times (range2to - range2from)) + range2from $$

$$ projected = (value \times (207 - 42)) + 42 $$

$$ projected = value \times 165 + 42 $$

## The Code

[The Full Solution](https://github.com/brianrackle/brainstem_breakfast/blob/master/BrainstemBreakfast/BrainstemBreakfast/gradient.hpp)

To leverage the code developed for the, ["Range Mapping and Projecting Values"]({% post_url 2014-08-18-range-mapping %}) post, it is convenient to simply use the `scale_value` function to interpolate each color component independently. This allows the `gradient_value` function to be reduced to just a few lines, with each RGB color component being mapped from a range of 0 to 1 to the range of `from_color.(r/g/b)` to `to_color.(r/g/b)`

{% highlight c++ %}
#include "range_map.hpp"

class rgb
{
public:
	using value_type = uint8_t;
	value_type r, g, b;
};

inline rgb gradient_value(const double value, const rgb & from_color, const rgb & to_color)
{
	using namespace range_map;
	return {
		scale_value(value, 0.0, 1.0, from_color.r, to_color.r),
		scale_value(value, 0.0, 1.0, from_color.g, to_color.g),
		scale_value(value, 0.0, 1.0, from_color.b, to_color.b) };
}
{% endhighlight %}

### Optimizations

It it would be handy to leverage the equation simplification from above to reduce unnecessary processing and reduce the number of parameters needed when using using a clamped percentage. So the following code was developed and added to the `range_map.hpp` header:

{% highlight c++ %}
//scales 'value' from the range 0.0/1.0 to the range 'lowestTo'/'highestTo'
template <class _TT,
class = typename std::enable_if<std::is_arithmetic<_TT>::value>::type>
	_TT scale_value(const double_t value, const _TT lowestTo, const _TT highestTo)
{
	if (value <= 0.0)
		return lowestTo;
	if (value >= 1.0)
		return highestTo;

	//scale by half to account for negative and positive range being too large to represent
	const auto && tHLF = [](_TT v){ return v / 2; };

	auto scaledOffsetResult = (tHLF(highestTo) - tHLF(lowestTo)) * value;
	return (_TT)(scaledOffsetResult + lowestTo + scaledOffsetResult); //seperated to prevent overflow
}
{% endhighlight %}	

Now the gradient code can be simplified:

{% highlight c++ %}
#include "markdown.hpp"
#include "range_map.hpp"

class rgb
{
public:
	using value_type = uint8_t;
	value_type r, g, b;
};

inline rgb gradient_value(const double value, const rgb & from_color, const rgb & to_color)
{
	using namespace range_map;
	return {
		scale_value(value, from_color.r, to_color.r),
		scale_value(value, from_color.g, to_color.g),
		scale_value(value, from_color.b, to_color.b) };
}

int main()
{
	using namespace markdown;

	ostream << table_header("cccc", "Transition1", "Transition2", "Transition3", "Transition4");
	for (int i = 0; i <= 10; ++i)
	{
		double v = (double)i / 10.0;
		auto make_cell = [](std::string clr)->std::string
		{return span(clr, color + "#ffffff", bg_color + clr); };

		ostream << table_row(
			make_cell(to_hex(gradient_value(v, { 255, 0, 0 }, { 0, 0, 0 }))),
			make_cell(to_hex(gradient_value(v, { 255, 255, 255 }, { 0, 0, 0 }))),
			make_cell(to_hex(gradient_value(v, { 0, 0, 0 }, { 255, 255, 255 }))),
			make_cell(to_hex(gradient_value(v, { 42, 163, 90 }, { 207, 74, 33 }))));
	}
}
{% endhighlight %}

### The Output

| Transition1 | Transition2 | Transition3 | Transition4 |
|:---:|:---:|:---:|:---: |
| <span style="color:#ffffff;background-color:#ff0000;">#ff0000</span>  | <span style="color:#ffffff;background-color:#ffffff;">#ffffff</span>  | <span style="color:#ffffff;background-color:#000000;">#000000</span>  | <span style="color:#ffffff;background-color:#2aa35a;">#2aa35a</span> |
| <span style="color:#ffffff;background-color:#e50000;">#e50000</span>  | <span style="color:#ffffff;background-color:#e5e5e5;">#e5e5e5</span>  | <span style="color:#ffffff;background-color:#191919;">#191919</span>  | <span style="color:#ffffff;background-color:#3a9a54;">#3a9a54</span> |
| <span style="color:#ffffff;background-color:#cc0000;">#cc0000</span>  | <span style="color:#ffffff;background-color:#cccccc;">#cccccc</span>  | <span style="color:#ffffff;background-color:#323232;">#323232</span>  | <span style="color:#ffffff;background-color:#4a914e;">#4a914e</span> |
| <span style="color:#ffffff;background-color:#b20000;">#b20000</span>  | <span style="color:#ffffff;background-color:#b2b2b2;">#b2b2b2</span>  | <span style="color:#ffffff;background-color:#4c4c4c;">#4c4c4c</span>  | <span style="color:#ffffff;background-color:#5b8848;">#5b8848</span> |
| <span style="color:#ffffff;background-color:#990000;">#990000</span>  | <span style="color:#ffffff;background-color:#999999;">#999999</span>  | <span style="color:#ffffff;background-color:#656565;">#656565</span>  | <span style="color:#ffffff;background-color:#6b7f42;">#6b7f42</span> |
| <span style="color:#ffffff;background-color:#800000;">#800000</span>  | <span style="color:#ffffff;background-color:#808080;">#808080</span>  | <span style="color:#ffffff;background-color:#7f7f7f;">#7f7f7f</span>  | <span style="color:#ffffff;background-color:#7c773d;">#7c773d</span> |
| <span style="color:#ffffff;background-color:#660000;">#660000</span>  | <span style="color:#ffffff;background-color:#666666;">#666666</span>  | <span style="color:#ffffff;background-color:#989898;">#989898</span>  | <span style="color:#ffffff;background-color:#8c6e37;">#8c6e37</span> |
| <span style="color:#ffffff;background-color:#4d0000;">#4d0000</span>  | <span style="color:#ffffff;background-color:#4d4d4d;">#4d4d4d</span>  | <span style="color:#ffffff;background-color:#b1b1b1;">#b1b1b1</span>  | <span style="color:#ffffff;background-color:#9c6531;">#9c6531</span> |
| <span style="color:#ffffff;background-color:#330000;">#330000</span>  | <span style="color:#ffffff;background-color:#333333;">#333333</span>  | <span style="color:#ffffff;background-color:#cbcbcb;">#cbcbcb</span>  | <span style="color:#ffffff;background-color:#ad5c2b;">#ad5c2b</span> |
| <span style="color:#ffffff;background-color:#1a0000;">#1a0000</span>  | <span style="color:#ffffff;background-color:#1a1a1a;">#1a1a1a</span>  | <span style="color:#ffffff;background-color:#e4e4e4;">#e4e4e4</span>  | <span style="color:#ffffff;background-color:#bd5325;">#bd5325</span> |
| <span style="color:#ffffff;background-color:#000000;">#000000</span>  | <span style="color:#ffffff;background-color:#000000;">#000000</span>  | <span style="color:#ffffff;background-color:#ffffff;">#ffffff</span>  | <span style="color:#ffffff;background-color:#cf4a21;">#cf4a21</span> |

There are no tricks or gotcha's with this code, all the work is performed in the `scale_value` function. In part 2 on gradients, I will cover another axial gradient and also create some fun test cases for the new functionality.