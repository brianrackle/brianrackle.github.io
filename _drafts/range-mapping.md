---
layout: post
title:  "Range Mapping and Projecting Values"
date:   2014-08-17 22:02:44
categories: update c++
tags: c++ 
author: brian
---

Projecting values from one range to another is a common need in certain problem sets. Simply getting a percent of a total is a basic form of range mapping. You have values in a given range and you need to translate those values into a range from 0 to 1. This is a basic operation we learn in middle school. Let's see if we can generalize the operation to map any range to any other range.

## The Basics

To start, it is important to understand the easiest form of range mapping, finding a percent. Finding a percent in a range starting from zero is actually the easiest starting point. This is easy because both the max range and the value also represent their distance from the min range. Meaning, if we have a range from 0 to 255, 255 is the max range, it is also the number of values from the min range. This applies to any value in the range as well. 32 is a value and the distance of the value from 0. Distance from the origin is important, because it will allow us to deduce the total percentage within the range. Given a value of 32, and a range of 0 to 255. The following equation will give the 

## Generalizing

Let's project values in a range from 234 to 6734, (yes, I just smashed my hand on the keyboard) to a range from 454522 to 97879213 (more hand smashing). Our starting value is 234, that way we know our startnig value must map to the lowest value in our second range. 

$$
\begin{align*}
  & \phi(x,y) = \phi \left(\sum_{i=1}^n x_ie_i, \sum_{j=1}^n y_je_j \right)
  = \sum_{i=1}^n \sum_{j=1}^n x_i y_j \phi(e_i, e_j) = \\
  & (x_1, \ldots, x_n) \left( \begin{array}{ccc}
      \phi(e_1, e_1) & \cdots & \phi(e_1, e_n) \\
      \vdots & \ddots & \vdots \\
      \phi(e_n, e_1) & \cdots & \phi(e_n, e_n)
    \end{array} \right)
  \left( \begin{array}{c}
      y_1 \\
      \vdots \\
      y_n
    \end{array} \right)
\end{align*}
$$