---
layout: post
title:  "Displaying Data (Engineering a Solution)"
date:   2014-08-03 12:36:00
categories: update c++ data visualization engineering
tags: c++ type traits template class stream output operator
author: brian
---

Visualizing data is hugely important to the world of algorithms. Throughout my career I have encountered innumerable problems that are incredibly hard to solve until I can view the problem outside of code. Many problems are just easier to understand when drawn. Simple mathematical expressions can become mangled and hard to understand when written as code. And even with the best debugging tools, things like data scope can turn understanding simple data into a practice of jumping between scopes, and dealing with hard to navigate data structures.

## Building a Framework

So how can the process of understanding our own algorithms be simplified? Perhaps we should build some tools visualize our algorithms. To build tools we need to establish the scope of our problem and then define the scope of our solution.

### Problems

* __changing scopes__
  * varying scopes make it hard to view the problem scope or dataset as a single entity
* __complex data structures__
  * data structures can hide the values we are interested in
* __varying lifetimes__
  * objects have varying lifetimes making it difficult to view related values side-by-side

### Requirements

* __scope invariant__
  * allow for side-by-side viewing of values that are in different scopes
* __simple representation__
  * simple representation of data nested inside of complex data structures
* __data retention__
  * retain values even when the lifetime of the value ends

__bonus feature__

* __KISS (keep it simple stupid)__
  * don't over-engineer the solution


### Solution

While there are infinite ways to approach this problem, I am adding the additional requirement of keeping the implementation simple. I propose using a file based approach, simplifying the problems with data persistence. Outputting values to a document will fulfill requirements __1 (scope invariant)__, __3 (data retention)__ and bonus feature __4 (KISS)__. Naturally, storing the values in a document leads us to consider formatting to fulfill requirement __2__ (simple representation). Considering the options, HTML stands out as a viable choice. However, dealing with the DOM across scopes and lifetimes is a fairly complex task. Alternatively, [Markdown](http://daringfireball.net/projects/markdown/) gives us HTML-like functionality and lets us completely bypass the DOM problem, as we can keep the output linear.

Great, let's build the solution!

## The Code

[The Full Solution](https://github.com/brianrackle/brainstem_breakfast/blob/master/BrainstemBreakfast/BrainstemBreakfast/markdown.hpp)

Markdown, is a human readable, way of formatting text that is convertible to proper HTML syntax. All we need to do is take Markdown syntax, make functions out of it and we are good to go. I will use [GitHub Flavored Markdown](https://help.github.com/articles/github-flavored-markdown) since it has extras that can come in handy. I will break down the construction of a single Markdown element, [the table element](https://help.github.com/articles/github-flavored-markdown#tables), and then use that as a guide.


### Table

* __column headers__
* __column orientation__
* __contents__

The table needs to allow creation of multiple column headers, orientations and contents. This can be accomplished using C++11 parameter packs. A parameter pack let's a function take a variable number of templated arguments. Inside the body of the function, we can expand that parameter pack to feed our next function call.

#### Row
{% highlight c++ %}
//create markdown table row
template <typename ... Types>
inline std::string table_row(const char * first_content, Types  ... rest_content)
{
	return std::string("| ") + first_content + " " + table_row(rest_content...);
}

//table_row recursion base case
inline std::string table_row()
{
	return "|\n";
}
{% endhighlight %}

The `table_row` functions work together to recurse through the parameter pack, `Types ... rest_content`, consuming a single parameter at a time. This functionality is beneficial because the parameter pack is expanded at compile time, so every parameter is checked by the compiler to be a `const char *`, much safer than C-style compile time variadic expansion. Obviously we need a base case for our recursion which is where `std::string table_row()` comes in, serving to terminate the recursion.

However, there is something incorrect about the `table_row` function. Our base case, while correct when called during recursion, is not correct when called on it's own. To fix it we need to modify the parameters so a row must consist of at least a single piece of content.

{% highlight c++ %}
//table_row recursion base case
inline std::string table_row(const char * content)
{
	return  std::string("| ") + content + "|\n";
}
{% endhighlight %}

We still have satisfied the base case, and also present a function which is usable on it's own.

#### Header
{% highlight c++ %}
//create markdown table header
template<class ... Types>
inline std::string table_header(const char * format, const char  * first_name, Types ... rest_name)
{
	return std::string("| ") + first_name + " " + table_header(format, rest_name...);
}

//table_header recursion base case
inline std::string table_header(const char * format, const char  * name)
{
	std::string output = std::string("| ") + name + "|\n";

	for (char const* pos = format; *pos; ++pos)
		switch (*pos)
	{
		case 'l':
			output += "|:--- ";
			break;
		case 'r':
			output += "| ---:";
			break;
		case 'c':
			output += "|:---:";
			break;
		default:
			break;
	}
	return output + "|" + "\n";
}
{% endhighlight %}

The `table_header` functions are similar to the `table_row` functions. `const char * format` is the alignment parameter which accepts C-style string constants using 'l' 'c' 'r' to represent left, center and right alignment. The name parameters are the column headers. Similar to the `table_row` functions, the `table_header` functions work recursively to consume the parameter pack. This time however, the format parameter is carried through to the base case since, the column orientation occurs after the column headers. 

### The Output

Here is simple code to test the table:

{% highlight c++ %}

std::ofstream outfile("outfile.txt");
outfile << table_header("llrr", "Word 1", "Word 2", "Word 3", "Word 4")
	<< table_row("hi", "how", "are", "you")
	<< table_row("I", "am", "fine", "thanks");

{% endhighlight %}

And the output:

| Word 1 | Word 2 | Word 3 | Word 4|
|:--- |:--- | ---:| ---:|
| hi | how | are | you|
| I | am | fine | thanks|

### All The Codes!
Below are all the Markdown functions I have tackled for my initial needs. You can find the current project on [github](https://github.com/brianrackle/brainstem_breakfast/blob/master/BrainstemBreakfast/BrainstemBreakfast/Markdown.hpp) Now we can start exploring our data.

{% highlight c++ %}
//create markdown table header
template<class ... Types>
inline std::string table_header(const char * format, const char  * first_name, Types ... rest_name)
{
	return std::string("| ") + first_name + " " + table_header(format, rest_name...);
}

//table_header recursion base case
inline std::string table_header(const char * format, const char  * name)
{
	std::string output = std::string("| ") + name + "|\n";

	for (char const* pos = format; *pos; ++pos)
		switch (*pos)
	{
		case 'l':
			output += "|:--- ";
			break;
		case 'r':
			output += "| ---:";
			break;
		case 'c':
			output += "|:---:";
			break;
		default:
			break;
	}
	return output + "|" + "\n";
}

//create markdown table row
template <class ... Types>
inline std::string table_row(const char * first_content, Types  ... rest_content)
{
	return std::string("| ") + first_content + " " + table_row(rest_content...);
}

//table_row recursion base case
inline std::string table_row(const char * content)
{
	return  std::string("| ") + content + "|\n";
}

//create markdown heading
inline std::string heading(unsigned char const level, const char * name)
{
	std::string output;
	for (std::remove_const_t<decltype(level)> i = 0; i < level; ++i)
		output += "#";
	return output + " " + name + "\n";
}

//create an html anchor. To be paired with an anchor link
inline std::string anchor(const char * name)
{
	return std::string("<a name = \"") + name + "\"></a>\n";
}

//create a markdown anchor link. To be paired with anchor
inline std::string anchor_link(const char * content, const char * name)
{
	return std::string("[") + content + "](#" + name + ")\n";
}

//create a markdown paragraph
inline std::string pgf(const char * content)
{
	return std::string("\n\n") + content + "\n\n";
}

//create a markdown block quote
inline std::string block_quote(const char * content)
{
	std::regex regex("\n");
	return "\n" + std::regex_replace(std::string("> ") + content, regex, "$&> ") + "\n";
}
{% endhighlight %}
