---
layout: post
title:  Variadic Template of C++
comments: true
category: c++
---

Variadic template is a feature in c++11. It enables function to accept different number of parameters. The following code is a simple usecase where `adder` return summation of all its incoming parameter.

```c++
// Base case
template<typename T>
T adder(const T& v) {
    std::cout << __PRETTY_FUNCTION__ << "\n";
    return v;
}

// typename... Args : template paramemter pack
// const Args&... args : function parameter pack
template<typename T, typename... Args>
T adder(const T& v, const Args&... args) {
    std::cout << __PRETTY_FUNCTION__ << "\n";
    return v + adder(args...);
}

int main() {
    std::cout<<adder(1,2,3,4,5,6)<<std::endl; 
//T adder(const T &, const Args &...) [T = int, Args = <int, int, int, int, int>]
//T adder(const T &, const Args &...) [T = int, Args = <int, int, int, int>]
//T adder(const T &, const Args &...) [T = int, Args = <int, int, int>]
//T adder(const T &, const Args &...) [T = int, Args = <int, int>]
//T adder(const T &, const Args &...) [T = int, Args = <int>]
//T adder(const T &) [T = int]
//21
}
```

The `adder` function is defined by two parts. One is base part and the other is general part. In general part, the function will peel of head of argument list and recursively call itself. The idea is very similar to the loop in functional programming shown below:

```
let sum l =
    match l with
    | [] -> 0
    | hd :: tl -> hd + sum tl
;;
```

A more complex and interesting is a `Tuple struct` holding varying amount of objects of different types.

```c++
// main part of Tuple definition
template<typename... Args> struct Tuple {};
template<typename T, typename...Args>
struct Tuple<T, Args...> : Tuple<Args...> {
    Tuple(T tv, Args... args) : Tuple<Args...>(args...), v(tv) {}
    T v;
};

// type retrieval helper function
template<size_t, class> struct tuple_element_holder {};
template<typename T, typename... Args> struct tuple_element_holder<0, Tuple<T, Args...>> { typedef T type; };
template<size_t k, typename T, typename... Args> struct tuple_element_holder<k, Tuple<T, Args...>> {
    typedef typename tuple_element_holder<k-1, Tuple<Args...>>::type type;
};


// value retrieval helper function
template<size_t k, typename T, typename... Args> 
typename std::enable_if<k != 0, typename tuple_element_holder<k, Tuple<T, Args...>>::type>::type get(const Tuple<T, Args...>& t) {
    const Tuple<Args...>& base = t;
    return get<k-1, Args...>(base);
}

template<size_t k, typename T, typename... Args>
typename std::enable_if<k == 0, typename tuple_element_holder<k, Tuple<T, Args...>>::type>::type get(const Tuple<T, Args...>& t) {
    return t.v;
}

int main() {
    Tuple<int, double, const char*, string> t(1, 0.2, "333", "321");
    std::cout<<get<1>(t)<<std::endl;
    std::cout<<get<2>(t)<<std::endl;
    std::cout<<get<3>(t)<<std::endl;
}
```

About the performance of variadic template, it is better than original C-style variadic parameter since most of work is done in compile time rather than runtime. And with the helper of compiler, nested function calls could be inlined and incur little runtime cost.

## Reference 

https://eli.thegreenplace.net/2014/variadic-templates-in-c/