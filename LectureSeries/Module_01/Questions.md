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

Daniel Arndt :
  - RangePolicy parallel_for is used when you want to execute N independent work items organized in a 1-dimensional index space and you don’t care about scratch space or mapping to the underlying execution model (or an abstraction thereof). The assumption is that all workitems are pretty much all equally expensive (since there is no way indicate to Kokkos which ones might be more expensive). Kokkos::Schedule is only used for host-parallel backends but might help in the case of unbalanced work loads.

Thibault Cimic :
    - Ok I understand, so yeah Kokkos is great, powerful and works pretty well but not magical to the point where you could (right now) say something like "Kokkos allows the user to get a good chance at reaching performance portability by giving the user tools to express parallelism in one formal way even if this parallelism is somewhat unbalanced" ?
    It'll be of course then up to the user to provide Kokkos with workitems that are pretty much all equally expensive as you said. And not up to the user to tell Kokkos : "it is unbalanced in this way, you should deal with it by distributing like that" ? (modifié) 

Daniel Arndt:
    - Yes, something like that. In the end the question still is if any other strategy would perform better. The GPU backends are also scheduling blocks/workgroups dynamically to the execution units mitigating the lack of explicit dynamic scheduling on the Kokkos side (within in a block/workgroup).