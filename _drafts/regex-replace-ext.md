---
layout: post
title:  "Extending Regex Replace"
date:   2014-09-20 16:27:00
categories: update c++
tags: c++ regex replace token iterator string template stl
author: brian
---

Recently I was working with the STL regular expression library, found in `<regex>`. I was building a simple utility to identify tokens in a string and replace them with pre-defined values. First, I needed all replacements to occur in a single pass. Second, I needed to identify each token with it's paired replacement value. As an example, let's say I am using tokens to define specific directory paths. These are the tokens and their matching values:

| Token | Value |
|:-----:|:-----:|
|{MyApp}|brainstem_breakfast|
|{InstallDir}|/home/brian/apps/|

I want to do a `regex_replace` on the string: {InstallDir}{MyApp}

`regex_replace` is unable to to perform this job because it only takes a string as a formatter. The `fmt` variable can use character groups using the $xx syntax, however it can't differentiate between groups dynamically. I am no regular expression wizard, I try to keep the regular expressions short and understandable, but I think `regex_replace` will not be able to fulfill my needs here.

## Regex Iterators

If `regex_replace` cannot match tokens to values, it is necessary to look through [`<regex>`](http://en.cppreference.com/w/cpp/regex/){:target="_blank"} to find something that will fill that roll. The prime candidate is `regex_token_iterator`, a ForwardIterator that allows tokenizing both matched and unmatched expressions. Using `regex_token_iterator` it is possible to iterate expressions, and uniquely identify unmatched expressions, matched expressions, and submatched expressions. Controlling what the tokenizer iterates is done through the `submatch`or`submatches` parameter of the constructor. `-1` iterates unmatched portions, `0` matches the entire regular expression, and values `1` and greater match groups/sub-matches.

## Extending regex_replace

Using the `regex_token_iterator` it will be possible to construct a new `regex_replace` function. The new function, rather than taking an `std::basic_string` or `CharT * `, will take a `std::function<std::basic_string (int submatch, std::basic_string)>`. In fact, the Boost [regex_replace](http://www.boost.org/doc/libs/1_44_0/libs/regex/doc/html/boost_regex/ref/regex_replace.html){:target="_blank"}, upon which the STL version was built, provides a similar functionality. In Boost `fmt` is defined as a Formatter, which can be C style string, a container of CharT (like `std::basic_string`, although I suppose `std::vector<CharT>` would also work), or a "unary, binary or ternary functor that computes the replacement string from a function call".

## The Solution

[The Full Solution](https://github.com/brianrackle/brainstem_breakfast/blob/master/source/regex_replace_ext.hpp)

The basic idea is to iterate unmatched and submatched expressions. With unmatched expressions, the token will be passed along to the output string. With matched expressions, the token and the subexpression index will be passed to the `fmt` function. The `fmt` function will determine the replacement value and return it to be inserted into the return string. 

Here I have created a string to show off the capabilities of the new `regex_replace_ext` as well as a regular expression to match the tokens in the string.

{% highlight c++ %}
const std::string bss("{Id_1} [Fill_0] {Id_2} [Fill_1] {Id_3} {Id_4} {Id_5}.");
const std::regex re("(\\{.*?\\})|(\\[.*?\\])");
{% endhighlight %}

Additionally, I need a mapping between tokens and values. I have seperated the "{ }" and "[ ]" tokens to highlight the flexibility of the new `regex_replace_ext`. I also, have created the function arguement for the `fmt` parameter. Match the submatch index to the correct dictionary, and then look for the token in the dictionary. 

{% highlight c++ %}
    using dictionary = std::map<std::string,std::string>;
    const std::vector<const dictionary> dict
      {
	{
	  {"{Id_1}","This"},
	  {"{Id_2}","test"},
	  {"{Id_3}","my"},
	  {"{Id_4}","favorite"},
	  {"{Id_5}","hotdog"}
	},
	{
	  {"[Fill_0]","is a"},
	  {"[Fill_1]","of"}
	}
      };

    auto fmt1 = [&dict](const unsigned smatch, const std::string & s)->std::string
      {
	auto dict_smatch = smatch - 1;
	if(dict_smatch > dict.size()-1)
	  return s; //more submatches than expected
  
        const auto it = dict[dict_smatch].find(s); 
	return it != dict[dict_smatch].cend() ? it->second : s;
      };
{% endhighlight %}

With all the arguments ready it is time to put the new `regex_replace_ext` into action.

{% highlight c++ %}
  template< class Traits, class CharT,
	    class STraits, class SAlloc >
  inline std::basic_string<CharT,STraits,SAlloc> 
  regex_replace_ext( const std::basic_string<CharT,STraits,SAlloc>& s,
		     const std::basic_regex<CharT,Traits>& re,
		     const typename std::common_type<std::function<std::basic_string<CharT,STraits,SAlloc> 
		     (const unsigned, const std::basic_string<CharT,STraits,SAlloc> &)>>::type& fmt,
			  std::regex_constants::match_flag_type flags =
			  std::regex_constants::match_default)
  {
    std::vector<int> smatches{-1};
    if(re.mark_count() == 0)
	smatches.push_back(0);
    else
      {
	smatches.resize(1+re.mark_count());
	std::iota(std::next(smatches.begin()), smatches.end(), 1); //-1, 1, 2, etc...    
      }

    unsigned smatch_count = smatches.size();
    unsigned count = 0;

    std::regex_token_iterator
      <typename std::basic_string<CharT,STraits,SAlloc>::const_iterator> 
      tbegin(s.begin(), s.end(), re, smatches, flags), tend;            

    std::basic_stringstream<CharT,STraits,SAlloc> ret_val;
    std::for_each(tbegin, tend, [&count,&smatch_count,&ret_val,&fmt]
		  (const std::basic_string<CharT,STraits,SAlloc> & token)
		  {
		    if(token.size() != 0)
		      {
			if(!count) 
			  ret_val << token;
			else
			  ret_val << fmt(count,token);
		      }
		    count = ++count % smatch_count;
		  });
    return ret_val.str();

  }
{% endhighlight %}

Let's break down the functionality into easy to understand parts. First, `regex_token_iterator` takes a submatch value or a vector of values, and I want to include the unmatched parts of the string. So I populate, -1 (the unmatched), skip 0 (the whole expression), and iterate incrementing values for as many submatches as exist in the regular expression.

{% highlight c++ %}
    std::vector<int> smatches{-1};
    if(re.mark_count() == 0)
	smatches.push_back(0);
    else
      {
	smatches.resize(1+re.mark_count());
	std::iota(std::next(smatches.begin()), smatches.end(), 1); //-1, 1, 2, etc...    
      }

    unsigned smatch_count = smatches.size();
    unsigned count = 0;
{% endhighlight %}

Constructing the regular expression iterator.

{% highlight c++ %}
    std::regex_token_iterator
      <typename std::basic_string<CharT,STraits,SAlloc>::const_iterator> 
      tbegin(s.begin(), s.end(), re, smatches, flags), tend;          
{% endhighlight %}

Finally, allowing the `regex_token_iterator` to do most of the work, I simply iterate the tokens. The `regex_token_iterator` will iterate the submatch tokens in the other they were passed, so if a token isn't found the value for the token will be an empty string. It necessary to keep track of which submatch the iterator is currently on, this is possible simply by incrementing a counter with a modulus of the total submatches. Finally, when the token is on `count == 0` that means the iterator is matching the unmatched token, so we simply put that value in the return string. Otherwise, `count` and `token` are sent to the `fmt` function which we defined earlier for the replacement values.

{% highlight c++ %}
    std::basic_stringstream<CharT,STraits,SAlloc> ret_val;
    std::for_each(tbegin, tend, [&count,&smatch_count,&ret_val,&fmt]
		  (const std::basic_string<CharT,STraits,SAlloc> & token)
		  {
		    if(token.size() != 0)
		      {
			if(count == 0) 
			  ret_val << token;
			else
			  ret_val << fmt(count,token);
		      }
		    count = ++count % smatch_count;
		  });
    return ret_val.str();
{% endhighlight %}

Thanks for reading!
Brian Rackle