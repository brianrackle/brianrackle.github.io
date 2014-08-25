---
layout: post
title:  "Effective Modern C++ Review"
date:   2014-08-17 22:36:30
categories: update c++ review
tags: c++ review effective modern c++ scott meyers
author: brian
---

Let me get this out of the way, I recommend this book. I don't have a rating system but will go ahead and rate the book eight fruit doves out of nine fruit doves.



## The Review

<a href=" http://shop.oreilly.com/product/0636920033707.do"><img src="/assets/emc_fruit_dove.jpg" alt="Effective Modern C++" width="200" style="float:right;"></a>
Scott Meyers, the author of ["Effective Modern C++"](http://shop.oreilly.com/product/0636920033707.do), is one of the foremost experts on the C++ programming language. In his latest book, he follows the format of his previous "Effective ..." books by focusing on how to take full advantage of the language. This is not a book that should be used to learn C++. In fact, it would be impossible to learn C++ with this book because it only covers a small but specific subset of the C++ programming language. Rather, "Effective Modern C++" gives you the details about how to use the features of C++11/14 to improve your code. Scott Meyers covers details of the language that are not covered anywhere else. And boy does he cover them... Want to know every reason why scoped enums are "preferable" to unscoped enums? Scott Meyers has you covered, chapter 3 item 10, and seven detail filled pages about any circumstance you could encounter while using scoped enums. This book covers details that you could only get from a true master of the language and for that reason I fully recommend it. 

## Preview
Here is the current table of contents, to give you a taste of the topics covered in the pre-release version of the book:

### __*Update*__
Scott Meyers has provided an [update](http://scottmeyers.blogspot.com/2014/08/near-final-draft-of-effective-modern-c.html) on the book. With the nearly finalized TOC. It looks like there are a lot of new topics compared to the previous release. I haven't reread the book but, certainly will as soon as possible.

### The Latest Table of Contents

#### CHAPTER 1  Deducing Types

-	Item 1:  Understand template type deduction. 
-	Item 2:  Understand auto type deduction. 
-	Item 3:  Understand decltype. 
-	Item 4:  Know how to view deduced types. 

#### CHAPTER 2  auto

-	Item 5:  Prefer auto to explicit type declarations. 
-	Item 6:  Use the explicitly typed initializer idiom when auto deduces undesired types. 

#### CHAPTER 3  Moving to Modern C++

-	Item 7:  Distinguish between () and {} when creating objects. 
-	Item 8:  Prefer nullptr to 0 and NULL. 
-	Item 9:  Prefer alias declarations to typedefs. 
-	Item 10: Prefer scoped enums to unscoped enums. 
-	Item 11: Prefer deleted functions to private undefined ones. 
-	Item 12: Declare overriding functions override. 
-	Item 13: Prefer const_iterators to iterators. 
-	Item 14: Declare functions noexcept if they won't emit exceptions. 
-	Item 15: Use constexpr whenever possible. 
-	Item 16: Make const member functions thread-safe. 
-	Item 17: Understand special member function generation. 

#### CHAPTER 4  Smart Pointers

-	Item 18: Use std::unique_ptr for exclusive-ownership resource management. 
-	Item 19: Use std::shared_ptr for shared-ownership resource management. 
-	Item 20: Use std::weak_ptr for std::shared_ptr-like pointers that can dangle.
-	Item 21: Prefer std::make_unique and std::make_shared to direct use of new.
-	Item 22: When using the Pimpl Idiom, define special member functions in the implementation file.

#### CHAPTER 5  Rvalue References, Move Semantics, and Perfect Forwarding

-	Item 23: Understand std::move and std::forward. 
-	Item 24: Distinguish universal references from rvalue references. 
-	Item 25: Use std::move on rvalue references, std::forward on universal references.
-	Item 26: Avoid overloading on universal references. 
-	Item 27: Familiarize yourself with alternatives to overloading on universal references.
-	Item 28: Understand reference collapsing. 
-	Item 29: Assume that move operations are not present, not cheap, and not used. 
-	Item 30: Familiarize yourself with perfect forwarding failure cases. 

#### CHAPTER 6  Lambda Expressions

-	Item 31: Avoid default capture modes. 
-	Item 32: Use init capture to move objects into closures. 
-	Item 33: Use decltype on auto&& parameters to std::forward them. 
-	Item 34: Prefer lambdas to std::bind. 

#### CHAPTER 7  The Concurrency API

-	Item 35: Prefer task-based programming to thread-based. 
-	Item 36: Specify std::launch::async if asynchronicity is essential. 
-	Item 37: Make std::threads unjoinable on all paths. 
-	Item 38: Be aware of varying thread handle destructor behavior. 
-	Item 39: Consider void futures for one-shot event communication. 
-	Item 40: Use std::atomic for concurrency, volatile for special memory. 

#### CHAPTER 8  Tweaks

-	Item 41: Consider pass by value for copyable parameters that are cheap to move and always copied.
-	Item 42: Consider emplacement instead of insertion. 
