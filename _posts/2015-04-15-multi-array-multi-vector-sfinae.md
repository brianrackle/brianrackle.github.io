---
layout: post
title:  "multi_array and multi_vector Template Aliases With SFINAE"
date:   2015-04-15 22:02:44
categories: update c++
tags: c++ 
author: brian
---

Let's face it, the `std::array` and `std::vector` containers suck for multi-dimensional data. The situation is so bad it makes me long for the days of `T array[N][M];` and `T** array`. Well, it turns out, to fix our modern C++11 and C++14 array containers, doesn't take much work.

## The Problem
Defining a multi-dimensional array using C++11/14 containers is no good:
<script src="https://gist.github.com/brianrackle/59034be202b5c88fc899.js"></script>

The good old days (which can still be used) were much better in some ways:
<script src="https://gist.github.com/brianrackle/78b6539dc836c3de75b5.js"></script>

And worse in other ways:
<script src="https://gist.github.com/brianrackle/68a7bbc0339fbf7d9a61.js"></script>

## The Solution
Why not a syntax for multi-dimensional container style arrays and vectors? You could get the best of both worlds like this:
<script src="https://gist.github.com/brianrackle/dec845d171094d4a1add.js"></script>

## The Code
It turns out, with the help of alias templates, and some class based SFINAE not much code is needed to achieve our desired syntax. 

`multi::array`:
<script src="https://gist.github.com/brianrackle/b343bb62fa909c9195ef.js"></script>

`multi::vector`:
<script src="https://gist.github.com/brianrackle/4a89852286745c0fe126.js"></script>

We don't need to reinvent the same data structures over and over when we can just use what we already have and let the templates do all the work.

Repo for this code: [https://github.com/brianrackle/multi](https://github.com/brianrackle/multi)
