---
layout: post
title:  "Extending Regex Replace"
date:   2014-09-20 16:27:00
categories: update c++
tags: c++ regex replace token iterator string template stl
author: brian
---

Recently I was working with the STL regular expression library, found in `<regex>`. I was building a simple utility to identify tokens in a string and replace them with pre-defined values. I had a few requirements. First, I needed all replacements to occur in a single pass. Second, I need to identify each token with it's paired replacement value. As an example, let's say I am generating paths and using tokens to define specific paths. These are the tokens and their matching values:

| Token | Value |
|:-----:|:-----:|
|{MyApp}|brainstem_breakfast|
|{InstallDir}|/home/brian/apps/|

I want to do a `regex_replace` on the string:

"{InstallDir}{MyApp}"

Great, `regex_replace` should be able to handle this right? Well, not quite. Looking at the [definitions for regex replace](http://en.cppreference.com/w/cpp/regex/regex_replace/){:target="_blank"}, there are 2 constructors that take `CharT *` as input, and both take some form of string for the `fmt` parameter. 'fmt' is the parameter that gives the value that replaces tokens matched by the passed in `basic_regex`. So what is the problem? Well I have a number of tokens value pairs I need to match but I can only use a string in the `fmt` parameter. The only flexiblity `fmt` provides is in it's ability to match character groups using the `$xx` syntax. I am no regular expression wizard, I try to keep the regular expressions short and understandable, but I think `regex_replace` will not be able to fulfill my needs here.

## regex_token_iterator

Well, if `regex_replace` cannot match tokens to values, it is necessary to look through [`<regex>`](http://en.cppreference.com/w/cpp/regex/){:target="_blank"} to find something that will fill that roll. The prime candidate is `regex_token_iterator`, a ForwardIterator that allows tokenizing both matched and unmatched expressions. Using `regex_token_iterator` it is possible to iterate the tokens of a string, and uniquely identify the unmatched expressions, the matched expressions, and the submatched expressions. Controlling what the tokenizer iterates is done through, the `submatch`or`submatches` parameter of the constructor. `-1` iterates unmatched portions, `0` matches the entire regular expression, and values `1` and greater match groups/sub-matches.

## Extending regex_replace

Using the `regex_token_iterator` it will be possible to construct a new `regex_replace` function. The new function, rather than taking an `std::basic_string` or `CharT * ` will take a `std::function<std::basic_string (int submatch, std::basic_string)>`. In fact, the Boost [regex_replace](http://www.boost.org/doc/libs/1_44_0/libs/regex/doc/html/boost_regex/ref/regex_replace.html){:target="_blank"}, upon which the STL version was built, provides a similar functionality. In Boost `fmt` is defined as a Formatter, which can be C style string, a container of CharT (like `std::basic_string`, although I suppose `std::vector<CharT>` would also work), or a "unary, binary or ternary functor that computes the replacement string from a function call".

## The Solution


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

- 

 For example, one  `regex_replace` is a very handy function, however,

### Contents
{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}


{% highlight c++ %}

{% endhighlight %}