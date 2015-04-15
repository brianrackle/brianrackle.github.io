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
```c++
std::array<std::array<int, 2>, 2> cpp11_array = {{ {0,1}, {2,3}  }};
std::vector<std::vector<int>> cpp11_vector = { {0,1}, {2,3}  };
```

The good old days (which can still be used) were much better in some ways:
```c++
int old_array[2][2] = { {0,1}, {2,3} };
```

And worse in other ways:
```c++
int** old_vector = new int*[M];
for(size_t i = 0; i < M; ++i)
    old_vector[i] = new int[N];
```

## The Solution
Why not a syntax for multi-dimensional container style arrays and vectors? You could get the best of both worlds like this:
```c++
multi::array<int, 2, 2> new_array = {{ {0,1}, {2,3} }};
multi::vector<int, 2> new_vector = { {0,1}, {2,3}  };
```

## The Code
It turns out, with the help of alias templates, and some class based SFINAE not much code is needed to achieve our desired syntax. 

`multi_array`:
```c++
namespace multi
{
    template <class T, size_t N, size_t ... Ns>
    struct array_helper;

    template <class T, size_t N>
    struct array_helper <T, N>
    {
        using type = std::array<T, N>;
    };

    template <class T, size_t N, size_t ... Ns>
    struct array_helper
    {
        using type = std::array<typename array_helper<T, Ns...>::type, N>;
    };

    template <class T, size_t ... Ns>
    using array = typename array_helper<T, Ns...>::type;
}
```

`multi_vector`:
```c++
namespace multi
{
    template <class T, size_t N>
    struct vector_helper;

    template <class T>
    struct vector_helper <T, 1>
    {
        using type = std::vector<T>;
    };

    template <class T, size_t N>
    struct vector_helper
    {
        using type = std::vector<typename vector_helper<T, N - 1>::type>;
    };

    template <class T, size_t N>
    using vector = typename vector_helper<T, N>::type;
}
```

We don't need to reinvent the same data structures over and over when we can just use what we already have and let the templates do all the work.

Repo for this code: [https://github.com/brianrackle/multi](https://github.com/brianrackle/multi)
