# Kokkos Tutorial Questions and Answers

This file is for storing already asked and answered questions regarding the Kokkos module you are actually in and to ask new ones and submit them via Pull Request.

# FAQ

## Question 1

Thibault Cimic (Original Poster OP):
    - In the first Module and with the first exercise, we were shown how to turn a pretty simple for-loop into its equivalent in Kokkos and were explain a bit in term of C++ how Kokkos does this via Functors (C++ Struct with the adequat overload of operator ()) :
    ```C++
    int M = 100;
    auto f = ...;
    auto x = static_cast<double*>(std::malloc(M * sizeof(double)));
    for (int i = 0; i < M; ++i) {
        x[i] = f[i];
    }
    ```

    into :
    ```C++
    int M = 100;
    auto f = ...;
    auto x = static_cast<double*>(std::malloc(M * sizeof(double)));
    Kokkos::parallel_for("Init::x", M, [=](const int64_t i){
        x[i] = f[i];
    });
    ```

    But actually no intakes on what Kokkos does internally in term of parallelization is given :

    Let's say Kokkos is given, by the user, the number of threads $nbThreads$ to run on, what will Kokkos actually do when this code is run on say CPU+OpenMP ? CPU+GPU/Cuda Nvidia ? Will it like cut the $M$ unit of work into $nbThreads$ unit of work to be done on each threads ? Than what if the actual computational cost of doing `f[i]` depends on `i` and can actually be unbalanced ?

### Answer

Trévis Morvany :
    - The distribution of the work units is specific to each backend, but can be controlled using Kokkos::Schedule<option> (option being Kokkos::static or Kokkos::dynamic) as a template parameter for the execution policy. It is introduced (along other execution policy options) at page 20 of the third lecture, or in the doc here

Thibault CIMIC (OP) :
    - Yeah that's what I figured : different for each backend. But then if too much different I guess it kind of kill any hope of having performance portability right ? So I guess Kokkos must try to do things in a similar sense cross backend though no ?