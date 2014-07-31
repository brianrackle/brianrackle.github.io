---
layout: post
title:  "Rotating Around the Origin"
date:   2014-07-13 22:02:44
categories: update c++ vector algebra trigonometry lambda
tags: c++ vector rotate matrix angle sin cos atan2 lambda polar coordinates line segment origin axis
author: brian
---


| Count | Vector | Calced Theta | True Theta | Delta |
|:--- |:---:| ---:| ---:| ---:|
| 0 | 1.0000000000000000 , 0.0000000000000000 | 0.0000000000000000 | 0.0000000000000000 | 0.0000000000000000 |
| 1 | 0.9999999999999950 , 0.0000001000000000 | 0.0000000999600281 | 0.0000001000000000 | 0.0000000000399719 |
| 2 | 0.9999999999999800 , 0.0000002000000000 | 0.0000001999200562 | 0.0000002000000000 | 0.0000000000799438 |
| 3 | 0.9999999999999550 , 0.0000003000000000 | 0.0000002998800844 | 0.0000003000000000 | 0.0000000001199156 |
| 4 | 0.9999999999999200 , 0.0000004000000000 | 0.0000003998401125 | 0.0000004000000000 | 0.0000000001598875 |
| 5 | 0.9999999999998750 , 0.0000005000000000 | 0.0000005000222247 | 0.0000005000000000 | 0.0000000000222247 |


	//Example 1: Rotating vector with lambda iterator
	//benefits:
	//	-versatile, can pass any worker into angleIterator
	//	-no memory storage
	//alternatives:
	//	-populate an std::vector with the values and then iterate. Excessive memory usage


	//rotate worse because error builds up in vector.
	//vs
	//polar better because vector recalculated on each iteration

	//could also store delta of every-1 angle and make a 1d heatmap out of it
