---
layout: blog
title: Type-Theoretic Constructs in C++
date: 2023-03-01
categories:
  - Theory, Data Structures, Algorithms, Programming Languages, Design Patterns
tags:
  - Essays
---

# Fixed-point Combinators, Tying the Recursive Knot, and Recursive Lambda Expressions

In Lambda Calculus, we cannot refer to the Lambda Abstraction *itself* within a Lambda Abstraction. Similarly, C++ does not allow defining a recursive lambda expression. Thus, we cannot straightforwardly implement recursion.

A workaround for this is to define a lambda expression that:

- Add an *additional first parameter* to the lambda expression.
- Call that *additional first parameter* inside the lambda expression at each recursive call site.

Such an additional first parameter should have the value of a yet-unknown hypothetical recursive function. Thus, we should use `auto` to represent its type. Using `auto` in a lambda expression's parameter list requires C++14 or above.

For example, we can define the following lambda expression to calculate the `n`th Fibonacci Number recursively:

```c++
auto fib = [](
    auto recursive_fib,
    unsigned int n
) -> unsigned long {
    if (n == 0) return 0UL;
    else {
        if (n == 1) return 1UL;
        else return recursive_fib(n - 1) + recursive_fib(n - 2);
    }
};
```

After defining such a lambda expression, we can use a [Fixed-point Combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator#Fixed-point_combinators_in_lambda_calculus) to [tie the recursive knot](https://courses.cs.cornell.edu/cs3110/2021sp/textbook/mut/ex_recursion_without_rec.html) and return a new recursive function without the additional first parameter.

What is a Fixed-Point Combinator?

In mathematics, a *fixed-point* for function $f$ refers to an element $x$ that is mapped to itself by the function, i.e., $x = f(x)$. For example, given $f(x)=x^{2}-3x+4$, then $2$ is a fixed point of $f$, because $f(2) = 2$.

A *combinator* is a function that operates on a function (i.e., a *higher-order function*).

A *fixed-point combinator* `g` for function `f` satisfies `g(f)(...) = f(g(f), ...)`. This is reminiscent of the form $x = f(x)$ for fixed points in mathematics, and `g(f)` can be seen as a *fixed-point* of function `f`.

This implies that *a fixed-point combinator `g`, when called on `f`, returns a new function (the `g(f)` above), that, when called with parameters `...`, is equivalent to directly calling `f` with both `g(f)` and `...`*. This means that a fixed-point combinator *returns a new recursive function without the additional first parameter of `f`*. 

This is done by *tying the recursive knot of `f`*. *Tying the recursive knot* refers to, for such a previously defined lambda expression `f`, passing a function that represents the hypothetical recursive function, which in this case is `g(f)`, to its first parameter.

We can implement fixed-point combinators in C++ using the following struct, whose instance represents `g(f)` and contains an `operator()` method, in which can use `this` to self-reference to `g(f)`, allowing us to support `f(g(f), ...)`:

```c++
template <typename F, typename R, typename... Args> struct FixedPointCombinator {
    F f;

    R operator()(Args... args) const {
        return f(*this, args...);
    }
};
```

C++ supports compiler optimizations for this pattern. For example, the following code:

```c++
#include <stdio.h>


template <typename F, typename R, typename... Args> struct FixedPointCombinator {
    F f;

    R operator()(Args... args) const {
        return f(*this, args...);
    }
};


auto fib = [](
    auto recursive_fib,
    unsigned int n
) -> unsigned long {
    if (n == 0) return 0UL;
    else {
        if (n == 1) return 1UL;
        else return recursive_fib(n - 1) + recursive_fib(n - 2);
    }
};


int main() {
    auto fib_ = FixedPointCombinator<decltype(fib), unsigned long, unsigned int> {fib};

    unsigned int input;
    scanf("%u", &input);

    unsigned long result = fib_(input);
    printf("%lu\n", result);

    return 0;
}
```
compiles to the following LLVM IR under `clang++ -std=c++14 -O2 -S -emit-llvm`, in which `FixedPointCombinator<decltype(fib), unsigned long, unsigned int> {fib}` has been optimized to the recursive function `@_ZNK3$_0clI20FixedPointCombinatorIS_mJjEEEEmT_j`:

```
; ModuleID = 'fib_recursive.cpp'
source_filename = "fib_recursive.cpp"
target datalayout = "e-m:e-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-unknown-linux-gnu"

@.str = private unnamed_addr constant [3 x i8] c"%u\00", align 1
@.str.1 = private unnamed_addr constant [5 x i8] c"%lu\0A\00", align 1

; Function Attrs: norecurse nounwind uwtable
define dso_local i32 @main() local_unnamed_addr #0 {
  %1 = alloca i32, align 4
  %2 = bitcast i32* %1 to i8*
  call void @llvm.lifetime.start.p0i8(i64 4, i8* nonnull %2) #4
  %3 = call i32 (i8*, ...) @scanf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str, i64 0, i64 0), i32* nonnull %1)
  %4 = load i32, i32* %1, align 4, !tbaa !2
  %5 = call fastcc i64 @"_ZNK3$_0clI20FixedPointCombinatorIS_mJjEEEEmT_j"(i32 %4) #4
  %6 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([5 x i8], [5 x i8]* @.str.1, i64 0, i64 0), i64 %5)
  call void @llvm.lifetime.end.p0i8(i64 4, i8* nonnull %2) #4
  ret i32 0
}

; Function Attrs: argmemonly nounwind
declare void @llvm.lifetime.start.p0i8(i64 immarg, i8* nocapture) #1

; Function Attrs: nofree nounwind
declare dso_local i32 @scanf(i8* nocapture readonly, ...) local_unnamed_addr #2

; Function Attrs: nofree nounwind
declare dso_local i32 @printf(i8* nocapture readonly, ...) local_unnamed_addr #2

; Function Attrs: argmemonly nounwind
declare void @llvm.lifetime.end.p0i8(i64 immarg, i8* nocapture) #1

; Function Attrs: inlinehint nounwind readnone uwtable
define internal fastcc i64 @"_ZNK3$_0clI20FixedPointCombinatorIS_mJjEEEEmT_j"(i32) unnamed_addr #3 align 2 {
  switch i32 %0, label %3 [
    i32 0, label %9
    i32 1, label %2
  ]

2:                                                ; preds = %1
  br label %9

3:                                                ; preds = %1
  %4 = add i32 %0, -1
  %5 = tail call fastcc i64 @"_ZNK3$_0clI20FixedPointCombinatorIS_mJjEEEEmT_j"(i32 %4) #4
  %6 = add i32 %0, -2
  %7 = tail call fastcc i64 @"_ZNK3$_0clI20FixedPointCombinatorIS_mJjEEEEmT_j"(i32 %6) #4
  %8 = add i64 %7, %5
  ret i64 %8

9:                                                ; preds = %1, %2
  %10 = phi i64 [ 1, %2 ], [ 0, %1 ]
  ret i64 %10
}

attributes #0 = { norecurse nounwind uwtable "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "less-precise-fpmad"="false" "min-legal-vector-width"="0" "no-frame-pointer-elim"="false" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }
attributes #1 = { argmemonly nounwind }
attributes #2 = { nofree nounwind "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "less-precise-fpmad"="false" "no-frame-pointer-elim"="false" "no-infs-fp-math"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }
attributes #3 = { inlinehint nounwind readnone uwtable "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "less-precise-fpmad"="false" "min-legal-vector-width"="0" "no-frame-pointer-elim"="false" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }
attributes #4 = { nounwind }

!llvm.module.flags = !{!0}
!llvm.ident = !{!1}

!0 = !{i32 1, !"wchar_size", i32 4}
!1 = !{!"clang version 9.0.1 (https://github.com/conda-forge/clangdev-feedstock 2ea3b72da24769de0dfc6dac99251a5d7a46144d)"}
!2 = !{!3, !3, i64 0}
!3 = !{!"int", !4, i64 0}
!4 = !{!"omnipotent char", !5, i64 0}
!5 = !{!"Simple C++ TBAA"}
```

We can slightly modify the code above to support a fixed-point combinator that also provides *memoization*. Note that **the instance of the fixed-point combinator is now stateful, and we should pass it by reference to `fib` (i.e., change the first parameter of `fib` to reference type)**:

```c++
#include <stdio.h>

#include <map>
#include <tuple>


template <typename F, typename R, typename... Args> struct FixedPointCombinatorWithMemoization {
    F f;
    std::map<std::tuple<Args...>, R> memo;

    R operator()(Args... args) {
        std::tuple<Args...> args_tuple(args...);

        // Check if result is in the memoization map
        auto found = memo.find(args_tuple);
        if (found != memo.end()) {
            return found->second; // Return the cached result
        }

        // Otherwise, compute and store the result
        R result = f(*this, args...);
        memo[args_tuple] = result;
        return result;
    }
};


auto fib = [](
    auto& recursive_fib,
    unsigned int n
) -> unsigned long {
    if (n == 0) return 0UL;
    else {
        if (n == 1) return 1UL;
        else return recursive_fib(n - 1) + recursive_fib(n - 2);
    }
};


int main() {
    auto fib_ = FixedPointCombinatorWithMemoization<decltype(fib), unsigned long, unsigned int> {fib};

    unsigned int input;
    scanf("%u", &input);

    unsigned long result = fib_(input);
    printf("%lu\n", result);

    return 0;
}
```


# Type Abstractions and Template Functions

Polymorphic Lambda Calculus (also known as Second Order Lambda Calculus or System F) introduces Type Abstractions and Type Applications.

- A Type Abstraction, written as `λ X . t`, represents a Term (often a Lambda Abstraction) `t` containing a Type Variable `X`.
- A Type Application, written as `t [T]`, uses a Concrete Type `T` to replace all instances of the Type Variable in the Term of the Type Abstraction.

This can be used to implement Polymorphic Lambda Abstractions.

For example, the following Type Abstraction representing a Polymorphic Identity Function:

```
id = λ X . λ x: X . x
```

can be instantiated to yield any concrete identity function that may be required, such as `id [Nat]: Nat -> Nat`.

Such Type Abstractions can be implemented in C++ using template functions:

```c++
template <typename X> auto id(X x) {
    return x;
}
```

while Type Applications correspond to template instantiations:

```c++
id<int>
```

Should the template function be passed a callable, we usually want to use a template typename to support functions, function pointers, functors, and lambda expressions. Alternatively, we can also use `auto` to represent its type in the template function's parameter list. Note that using `auto` in a (non-lambda expression) function's parameter list requires C++20 or above.

For example, the following Type Abstraction:

```
double = λ X . λ f: X -> X . λ a: X . f(f a)
```

can be represented using the following template function:

```c++
template <typename X, typename F> auto double_(const F f) {
    return [f](X a) {
        return f(f(a));
    };
}
```

or in C++20 or above:

```c++
template <typename X> auto double_(const auto f) {
    return [f](X a) {
        return f(f(a));
    };
}
```

C++ compilers support aggressive inlining optimizations when lambda expressions are used. For example, the call to `const auto g = double_<int>([](int x) { return 2 * x; });` in the following source code:

```c++
#include <stdio.h>


template <typename X> auto double_(const auto f) {
    return [f](X a) {
        return f(f(a));
    };
}


int main() {
    const auto g = double_<int>([](int x) { return 2 * x; });
    
    int input;
    scanf("%d", &input);

    printf("%d\n", g(input));
    
    return 0;
}
```

has been completely inlined to `%5 = shl i32 %4, 2` in the LLVM IR generated with `clang++ -std=c++20 -O2 -S -emit-llvm`:

```llvm
@.str = private unnamed_addr constant [3 x i8] c"%d\00", align 1
@.str.1 = private unnamed_addr constant [4 x i8] c"%d\0A\00", align 1

; Function Attrs: norecurse nounwind uwtable
define dso_local i32 @main() local_unnamed_addr #0 {
  %1 = alloca i32, align 4
  %2 = bitcast i32* %1 to i8*
  call void @llvm.lifetime.start.p0i8(i64 4, i8* nonnull %2) #3
  %3 = call i32 (i8*, ...) @__isoc99_scanf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str, i64 0, i64 0), i32* nonnull %1)
  %4 = load i32, i32* %1, align 4, !tbaa !2
  %5 = shl i32 %4, 2
  %6 = call i32 (i8*, ...) @printf(i8* nonnull dereferenceable(1) getelementptr inbounds ([4 x i8], [4 x i8]* @.str.1, i64 0, i64 0), i32 %5)
  call void @llvm.lifetime.end.p0i8(i64 4, i8* nonnull %2) #3
  ret i32 0
}

; Function Attrs: argmemonly nounwind willreturn
declare void @llvm.lifetime.start.p0i8(i64 immarg, i8* nocapture) #1

; Function Attrs: nofree nounwind
declare dso_local i32 @__isoc99_scanf(i8* nocapture readonly, ...) local_unnamed_addr #2

; Function Attrs: nofree nounwind
declare dso_local i32 @printf(i8* nocapture readonly, ...) local_unnamed_addr #2

; Function Attrs: argmemonly nounwind willreturn
declare void @llvm.lifetime.end.p0i8(i64 immarg, i8* nocapture) #1
```

# Type Classes and Concepts in C++

Template Metaprogramming in C++ had been untyped, with template parameters being generic type variables substituted at template instantiation.

In C++20, a type system has been added to this untyped template language through concepts. They are Boolean predicates on template parameters evaluated at the point of, not after, template instantiation. The compiler will produce a clear error immediately if a programmer tries to use a template parameter that doesn't meet the requirements of a concept.

This starkly contrasts the challenging-to-grasp errors reported after an invalid type substitutes a generic type variable emanating from the implementation context rather than the template instantiation itself.

For instance, the first two arguments to `std::sort` must be random-access iterators. If an argument is not a random-access iterator, an error will occur when `std::sort` attempts to use it as a bidirectional iterator.


```c++
std::list<int> l = {2, 1, 3};
std::sort(l.begin(), l.end());
```

Without concepts, compilers may produce large amounts of error information, starting with an equation that failed to compile when it tried to subtract two non-random-access iterators:

```
In instantiation of 'void std::__sort(_RandomAccessIterator, _RandomAccessIterator, _Compare) [with _RandomAccessIterator = std::_List_iterator<int>; _Compare = __gnu_cxx::__ops::_Iter_less_iter]':
 error: no match for 'operator-' (operand types are 'std::_List_iterator<int>' and 'std::_List_iterator<int>')
 std::__lg(__last - __first) * 2,
```

However, if concepts are used, the problem can be found and reported at template instantiation:

```
error: cannot call function 'void std::sort(_RAIter, _RAIter) [with _RAIter = std::_List_iterator<int>]'
note:   concept 'RandomAccessIterator()' was not satisfied
```

It is straightforward to implement Type Classes with concepts. For instance, the Type Class below specifies the equal (==) operations for Type Constructors that are its instances: 

```haskell
class Eq a where
  (==) :: a -> a -> Bool
```

This can be implemented using the following C++ concept:

```c++
#include <concepts>


// Declaration of the concept "Eq",
// which is satisfied by any type 'T' such that for values 't' of type 'T', the expression t == t compiles and its type satisfies the concept std::same_as<bool>
// This is represented using a "requires expression" which returns a bool
template <typename T> concept Eq = requires (T t) {
    { t == t } -> std::same_as<bool>;
};
```

Afterwards, such a concept can be specified when template parameters are being introduced in a template definition, to indicate that the corresponding template parameter must satisfy the concept.

```c++
template<Eq T> void f(const T& t) {
    // ...
}
```

or (using a "requires clause"):

```c++
template<typename T> requires Eq<T> void f(const T& t) {
    // ...
}
```

# References

- https://stackoverflow.com/questions/6948166/javas-interface-and-haskells-type-class-differences-and-similarities
- https://cs.gmu.edu/~sean/stuff/java-objc.html
- https://functionalcpp.wordpress.com/2013/08/16/type-classes/
- https://stackoverflow.com/questions/32124627/how-are-c-concepts-different-to-haskell-typeclasses
- https://wiki.haskell.org/OOP_vs_type_classes
- https://doi.org/10.1145/1411318.1411324
- https://www.foonathan.net/2021/07/concepts-structural-nominal/
- https://www.reddit.com/r/haskell/comments/1e9f49/concepts_in_c_template_programming_and_type/
