---
layout: post
title:  "Displaying Data"
date:   2014-07-31 22:34:00
categories: update c++ data visualization engineering
tags: c++ type traits template class stream output operator
author: brian
---

Visualizing data is hugely important to the world of algorithms. Throughout my career I have encountered innumerable problems that are incredibly hard to solve until I can view the problem outside of code. Many problems are just easier to understand when drawn. Simple mathematical expressions can become mangled and hard to understand when written as code. And even with the best debugging tools, things like data scope can turn understanding simple data into a practice of jumping between scopes, and dealing with hard to navigate data structures.

## Building a Framework

So how can one simplify the process of understanding our own algorithms? Well, do what humans have been doing for thousands of years, build tools. To build tools we need to establish the scope of our problem and then define the scope of our solution.

### Problems

* changing scopes
  * varying scopes make it hard to view the problem scope or dataset as a single entity
* complex data structures
  * data structures can hide the values we are looking for
* varying lifetimes
  * objects have varying lifetimes making impossible to view values side-by-side

### Requirements

* scope invariant
  * allow for side-by-side viewing of values that are in different scopes
* simple representation
  * capable of representing the parts nested inside of complex data structures
* data retention
  * retain values even when the lifetime of the value ends
__bonus feature__
* KISS (keep it simple stupid)


### Solution

While there are infinite ways to approach this problem, I am adding an additional requirement of keeping the implementation simple (at least for the first version). Outputting values to a document would let us fulfill requirements 1 (scope invariant), 3 (data retention) and bonus feature 4 (KISS). Naturally, storing the values in a document leads us to considering formatting to fulfill requirement 2 (simple representation). Considering our options HTML stands out as a choice. However, dealing with the DOM across scopes and lifetimes is a fairly complex task. Alternatively, Markdown gives us HTML-like functionality and lets us completely bypass the DOM path, as we can keep our output linear.

Great, let's build the solution!

### Markdown Output


