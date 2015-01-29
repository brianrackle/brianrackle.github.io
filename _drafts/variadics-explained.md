
from to output parameter

The syntax for declaring a variadic template parameter is `class... A`, seen in `template<class T, class R, class... A>`. The next syntax, for declaring a parameter pack, which is necessary because we are specializing our variadic template is, `A...`, seen in `struct mft<R (T::*)(A...)>`. Going down the code line by line the next related syntax is `sizeof...(A)`; which expands the templat parameter pack and gives the count. Finally, the last related syntax is `std::tuple<A...>, which expands the parameter pack `A`, for the template `tuple`.