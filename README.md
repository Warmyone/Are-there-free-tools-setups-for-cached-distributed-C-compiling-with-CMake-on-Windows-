# Are there free tools/setups for cached, distributed C++ compiling with CMake on Windows?

This problem touches too many topics outside of the CMake, C++, Windows world I am familiar with. I kind of hit a wall of way to many options to explore each one. Actually I am hoping for someone posting their workflow which we can adopt. Otherwise answers to these questions that came up while [researching](#research) would help me to come closer to a solution:

* If a tool states to support a compiler, does that equal to support debugging with it? Or is MSVC support but without PDB files an exception?
* Is Cygwin the only way to use some of the Unix-like tools for Windows?
* Do we fall under the Cygwin licencing if the tools we use to compile our code were build with Cygwin? If we would only internally use that tools not to distribute the software would the licence apply?
* Would it be feasible use [sccache] and run the servers for distribution in a VM with Linux on our Windows Systems?
* What is the progress on the [ccache] Windows adoption?
* I ruled out some of the option due to there lack of maintenance, does someone have experience with them? Uses them in production?

## Goal

We want to improve the build time of our project. Other steps like dependency reduction or reduced template instantiations and more are known but done/explored by others. I am supposed to setup a compile environment that is as fast as possible. My goal is to cache the compiler results, compile in parallel and distributed. Speeding up the debugging process is more important than building a release. The jobs should be distribution on Windows workstations. If possible, distribution should consider the capacities of each system.

## Setup/Limitations

We use CMake 3.16 and if somehow possible we do not want to change that. All workstations run Windows 10 which cannot be changed. We use C++17 with Qt 5.11.3 and QML. Our current development environments are Visual Studio Code using the CMake Ninja generator or Visual Studio Community in the CMake folder mode. In the end both run the MSVC 15 2017 or MSVC 16 2019 compiler. Our software needs to build for Windows as 64 Bit. We are searching for free solution to use in our commercially used software.


## Research

### Summary
There are great free solutions for Unix-like systems but they do not support Windows. For Windows with CMake there are some free but outdated options for caching. [sccache] seems to be the only applicable option which might force us to switch the compiler to be able to cache when debugging. There seems to be no free solution for distribution. Also there are fee required options for caching and distribution. [FastBuild] would be a perfect fit but there is no CMake generator for it.

### CMake
On Unix-like systems one can easily setup caching with CMake. *code from [modern-cmake](https://cliutils.gitlab.io/modern-cmake/chapters/features/utilities.html)*

```CMake
    find_program(CCACHE_PROGRAM ccache)
    if(CCACHE_PROGRAM)
        set(CMAKE_C_COMPILER_LAUNCHER "${CCACHE_PROGRAM}")
        set(CMAKE_CXX_COMPILER_LAUNCHER "${CCACHE_PROGRAM}")
    endif()
```
And one needs to set `CCACHE_PREFIX=distcc` to enable distribution. This would also apply on Windows but [ccache] is not officially supported on that platform. Same goes for distribution with [distcc]. Therefore I researched which tools there are and listed the problems with them.

### Cache

* [ccache]
    * Does not officially support Windows. They [state](https://github.com/ccache/ccache/issues/447#issue-473739018), it could be build with MinGW but give no instructions on how to do that. Also they state that they want to have native Windows support at some point but I find no information about their progress or where to get a current version.
    * There seems to also be a way to build an executable with Cygwin but I do not fully understand the process described in the comments of this Stack Overflow [question](https://stackoverflow.com/questions/55610898/how-to-set-up-and-use-ccache-with-cygwin-on-windows) due to lack of Unix-like knowledge, furthermore I don't know if this would bound our application to the Cygwin licencing. I have not put much afford into these types of solutions as I hope to find a Windows native one.
    * Does only support GCC-like. ["Only works with GCC"][ccache]
    * Does not support debug caching? ["...multi-file compilation, linking, etc) will silently fall back..."][ccache]
* [clcache] is no longer maintained. ["It's unmaintained, yes"](https://github.com/frerich/clcache/issues/365#issuecomment-558682022)
* [CClash] is also no longer maintained. Last changes are form Sep 2017.
* [Stashed] uses a remote cache for all system. But it is not free.
* [sccache] also uses a shared cache. They list MSVC as supported but I read that people have problems with it. But it does not support debugging with it anyway. ["It's effectively impossible to cache PDB files that MSVC outputs"](https://github.com/mozilla/sccache/issues/242)

### Distribution

* [distcc]
   * Native Windows is not supported which raises the same problems as with [ccache]. ["On Windows, you need to use the Cygwin (or perhaps MinGW?) software, which provides a Unix-like environment for running gcc."](https://distcc.github.io/faq.html)
   * Does only work with GCC and Clang
* [sccache-dist] is basically [sccache] but build with a new distributed feature enabled. It does support Windows as client but the server has to be Unix-like.
* [Icecream] distributes to the fasted free server but does not support Windows.
* [IncrediBuild] is not free.
* [Wuild] looks good, but it was developed for only one year and the last changes are from  18.08.2018.

### Other

* [FastBuild] is  an entire build system including caching, distribution and more, supporting all major compilers. Nearly everything I found about it was positive but it has [no CMake generator](https://gitlab.kitware.com/cmake/cmake/issues/15294) yet.

I hope the question does satisfy the Stack Overflow requirements especially with that many not directly related sub questions. I am looking for a CMake solution to this problem but am aware that it might end up in a "software recommendations"-like question, so I hope this is the right place to ask.

[ccache]: https://ccache.dev/
[distcc]: https://github.com/distcc/distcc
[clcache]: https://github.com/frerich/clcache
[CClash]: https://github.com/inorton/cclash
[Stashed]: https://stashed.io
[sccache]: https://github.com/mozilla/sccache
[sccache-dist]: https://github.com/mozilla/sccache/blob/master/docs/DistributedQuickstart.md#configure-a-client
[FastBuild]: http://www.fastbuild.org/docs/features.html
[Wuild]: https://github.com/mapron/Wuild
[Icecream]: https://github.com/icecc/icecream
[IncrediBuild]: https://www.incredibuild.com/

