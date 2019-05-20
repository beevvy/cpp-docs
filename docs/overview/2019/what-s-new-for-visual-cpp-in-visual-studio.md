---
title: "What's New for C++ in Visual Studio 2019"
ms.date: "05/13/2019"
ms.technology: "cpp-ide"
ms.assetid: 8801dbdb-ca0b-491f-9e33-01618bff5ae9
author: "mikeblome"
ms.author: "mblome"
---

# What's New for C++ in Visual Studio 2019

Visual Studio 2019 brings many updates and fixes to the Microsoft C++ environment. We've fixed many bugs and issues in the compiler and tools, many submitted by customers through the [Report a Problem](/visualstudio/how-to-report-a-problem-with-visual-studio-2017) and [Provide a Suggestion](https://developercommunity.visualstudio.com/spaces/62/index.html) options under **Send Feedback**. Thank you for reporting bugs! For more information on what's new in all of Visual Studio, visit [What's new in Visual Studio](/visualstudio/ide/whats-new-visual-studio-2019).

## C++ compiler

- Enhanced support for C++17 features and correctness fixes, plus experimental support for C++20 features such as modules and coroutines. For detailed information, see [C++ Conformance Improvements in Visual Studio 2019](../cpp-conformance-improvements.md).

- The `/std:c++latest` option now includes C++20 features that aren't necessarily complete, including initial support for the C++20 operator \<=> ("spaceship") for three-way comparison.

- The C++ compiler switch `/Gm` is now deprecated. Consider disabling the `/Gm` switch in your build scripts if it's explicitly defined. However, you can also safely ignore the deprecation warning for `/Gm`, because it's not treated as an error when using "Treat warnings as errors" (`/WX`).

- As MSVC begins implementing features from the C++20 standard draft under the `/std:c++latest` flag, `/std:c++latest` is now incompatible with `/clr` (all flavors), `/ZW`, and `/Gm`. In Visual Studio 2019, use `/std:c++17` or `/std:c++14` modes when compiling with `/clr`, `/ZW` or `/Gm` (but see previous bullet).

- Precompiled headers are no longer generated by default for C++ console and desktop apps.

### Codegen, security, diagnostics, and versioning

Improved analysis with `/Qspectre` for providing mitigation assistance for Spectre Variant 1 (CVE-2017-5753). For more information, see [Spectre Mitigations in MSVC](https://devblogs.microsoft.com/cppblog/spectre-mitigations-in-msvc/).

## C++ Standard Library improvements

- Implementation of additional C++17 and C++20 library features and correctness fixes. For detailed information, see [C++ Conformance Improvements in Visual Studio 2019](../cpp-conformance-improvements.md).

- Clang-Format has been applied to the C++ Standard Library headers for improved readability.

- Because Visual Studio now supports Just My Code for C++, the Standard Library no longer needs to provide custom machinery for `std::function` and `std::visit` to achieve the same effect. Removing that machinery largely has no user-visible effects, except that the compiler will no longer produce diagnostics that indicate issues on line 15732480 or 16707566 of \<type_traits> or \<variant>.

## Performance/throughput improvements in the compiler and Standard Library

- Build throughput improvements, including the way the linker handles File I/O, and link time in PDB type merging and creation.

- Added basic support for OpenMP SIMD vectorization. You can enable it using the new compiler switch `-openmp:experimental`. This option allows loops annotated with `#pragma omp simd` to potentially be vectorized. The vectorization isn't guaranteed, and loops annotated but not vectorized will get a warning reported. No SIMD clauses are supported, they're simply ignored with a warning reported.

- Added a new inlining command-line switch `-Ob3`, which is a more aggressive version of `-Ob2`. `-O2` (optimize the binary for speed) still implies `-Ob2` by default. If you find that the compiler doesn't inline aggressively enough, consider passing `-O2 -Ob3`.

- To support hand vectorization of loops with calls to math library functions, and certain other operations like integer division, we've added support for Short Vector Math Library (SVML) intrinsic functions. These functions compute the 128-bit, 256-bit, or 512-bit vector equivalents. See the [Intel Intrinsic Guide](https://software.intel.com/sites/landingpage/IntrinsicsGuide/#!=undefined&techs=SVML) for definitions of the supported functions.

- New and improved optimizations:

  - Constant-folding and arithmetic simplifications for expressions using SIMD vector intrinsics, for both float and integer forms.

  - A more powerful analysis for extracting information from control flow (if/else/switch statements) to remove branches always proven to be true or false.

  - Improved memset unrolling to use SSE2 vector instructions.

  - Improved removal of useless struct/class copies, especially for C++ programs that pass by value.

  - Improved optimization of code using `memmove`, such as `std::copy` or `std::vector` and `std::string` construction.

- Optimized the Standard Library physical design to avoid compiling parts of the Standard Library not #include'd, cutting in half the build time of an empty file that includes only \<vector>. As a result of this change, you may need to add #include directives for headers that were previously indirectly included. For example, code that uses `std::out_of_range` may now need to #include \<stdexcept>. Code that uses a stream insertion operator may now need to #include \<ostream>. The benefit is that only translation units actually using \<stdexcept> or \<ostream> components pay the throughput cost to compile them.

- `if constexpr` was applied in more places in the Standard Library for improved throughput and reduced code size in copy operations, in permutations like reverse and rotate, and in the parallel algorithms library. 

- The Standard Library now internally uses `if constexpr` to reduce compile times even in C++14 mode.

- The runtime dynamic linking detection for the parallel algorithms library no longer uses an entire page to store the function pointer array. Marking this memory read-only was deemed no longer relevant for security purposes.

- `std::thread`'s constructor no longer waits for the thread to start, and no longer inserts so many layers of function calls between the underlying C library `_beginthreadex` and the supplied callable object. Previously `std::thread` put 6 functions between `_beginthreadex` and the supplied callable object, which has been reduced to only 3 (2 of which are just `std::invoke`). This also resolves an obscure timing bug where `std::thread`'s constructor would hang if the system clock changed at the exact moment a `std::thread` was being created.

- Fixed a performance regression in `std::hash` that we introduced when implementing `std::hash<std::filesystem::path>`.

- In several places the Standard Library now uses destructors instead of catch blocks to achieve correctness. This results in better debugger interaction; exceptions you throw through the Standard Library in the affected locations will now show up as being thrown from their original throw site, rather than our rethrow. Not all Standard Library catch blocks have been eliminated; we expect the number of catch blocks to be reduced in subsequent releases of MSVC.

- Suboptimal codegen in `std::bitset` caused by a conditional throw inside a noexcept function was fixed by factoring out the throwing path.

- The `std::list` and `std::unordered_*` family use non-debugging iterators internally in more places.

- Several `std::list` members were changed to reuse list nodes where possible rather than deallocating and reallocating them. For example, given a `list<int>` that already has a size of 3, a call to `assign(4, 1729)` will now overwrite the ints in the first 3 list nodes, and allocate one new list node with the value 1729, rather than deallocating all 3 list nodes and then allocating 4 new list nodes with the value 1729.

- All Standard Library calls to `erase(begin(), end())` were changed to `clear()`.

- `std::vector` now initializes and erases elements more efficiently in certain cases.

- Improvements to `std::variant` to make it more optimizer-friendly, resulting in better generated code. Code inlining is now much better with `std::visit`.

## C++ IDE

### Live Share C++ support

[Live Share](/visualstudio/liveshare/) now supports C++, allowing developers using Visual Studio or Visual Studio Code to collaborate in real time. For more information, see [Announcing Live Share for C++: Real-Time Sharing and Collaboration](https://devblogs.microsoft.com/cppblog/cppliveshare/)

### IntelliCode for C++

IntelliCode is an optional extension (added as a workload component in 16.1) that uses its own extensive training and your code context to put what you’re most likely to use at the top of your completion list. It can often eliminate the need to scroll down through the list. For C++, IntelliCode offers the most help when using popular libraries such as the Standard Library. For more information, see [AI-Assisted Code Completion Suggestions Come to C++ via IntelliCode](https://devblogs.microsoft.com/cppblog/cppintellicode/).

### Template IntelliSense

The **Template Bar** now uses the **Peek Window** UI rather than a modal window, supports nested templates, and pre-populates any default arguments into the **Peek Window**. For more information, see [Template IntelliSense Improvements for Visual Studio 2019 Preview 2](https://devblogs.microsoft.com/cppblog/template-intellisense-improvements-for-visual-studio-2019-preview-2/). A **Most Recently Used** dropdown in the **Template Bar** enables you to quickly switch between previous sets of sample arguments.

### New Start window experience

When launching the IDE, a new Start window appears with options to open recent projects, clone code from source control, open local code as a solution or a folder, or create a new project. The New Project dialog has also been overhauled into a search-first, filterable experience.

### New names for some project templates

We've modified several project template names and descriptions to fit with the updated New Project dialog.

### Various productivity improvements

Visual Studio 2019 includes the following features that will help make coding easier and more intuitive:

- Quick fixes for:
  - Add missing #include
  - NULL to nullptr
  - Add missing semicolon
  - Resolve missing namespace or scope
  - Replace bad indirection operands (\* to & and & to \*)
- Quick Info for a block by hovering on closing brace
- Peek Header / Code File
- Go to Definition on #include opens the file

For more information, see [C++ Productivity Improvements in Visual Studio 2019 Preview 2](https://devblogs.microsoft.com/cppblog/c-productivity-improvements-in-visual-studio-2019-preview-2/).

**Visual Studio 2019 version 16.1**

### QuickInfo improvements

The Quick Info tooltip now respects the semantic colorization of your editor. It also has a new **Search Online** link that will search for online docs to learn more about the hovered code construct. For red-squiggled code, the link provided by Quick Info will search for the error online. This way you don’t need to retype the message into your browser. For more information, see [Quick Info Improvements in Visual Studio 2019: Colorization and Search Online](https://devblogs.microsoft.com/cppblog/quick-info-improvements-in-visual-studio-2019-colorization-and-search-online/).

### IntelliCode available in C++ workload

IntelliCode now ships as an optional component in the **Desktop Development with C++** workload. For more information, see [Improved C++ IntelliCode now Ships with Visual Studio 2019](https://devblogs.microsoft.com/cppblog/).

## CMake support

- Support for CMake 3.14

- Visual Studio can now open existing CMake caches generated by external tools, such as CMakeGUI, customized meta-build systems or build scripts that invoke cmake.exe themselves.

- Improved IntelliSense performance.

- A new settings editor provides an alternative to manually editing the CMakeSettings.json file, and provides some parity with CMakeGUI.

- Visual Studio helps bootstrap your C++ development with CMake on Linux by detecting if you have a compatible version of CMake on your Linux machine, and if not offers to install it for you.

- Incompatible settings in CMakeSettings, such as mismatched architectures or incompatible CMake generator settings, show squiggles in the JSON editor and errors in the error list.

- The vcpkg toolchain is automatically detected and enabled for CMake projects that are opened in the IDE once `vcpkg integrate install` has been run. This behavior can be turned off by specifying an empty toolchain file in CMakeSettings.

- CMake projects now enable Just My Code debugging by default.

- Static analysis warnings can now be processed in the background and displayed in the editor for CMake projects.

- Clearer build and configure 'begin' and 'end' messages for CMake projects and support for Visual Studio's build progress UI. Additionally, there's now a CMake verbosity setting in **Tools > Options** to customize the detail level of CMake build and configuration messages in the Output Window.

- The `cmakeToolchain` setting is now supported in CMakeSettings.json to specify toolchains without manually modifying the CMake command line.

- A new **Build All** menu shortcut **Ctrl+Shift+B**.

**Visual Studio 2019 version 16.1**

- Integrated support for editing, building, and debugging CMake projects with Clang/LLVM. For more information, see [Clang/LLVM Support in Visual Studio](https://devblogs.microsoft.com/cppblog/clang-llvm-support-in-visual-studio/).

## Linux and WSL

**Visual Studio 2019 version 16.1**

- Support for [AddressSanitizer (ASan)](https://github.com/google/sanitizers/wiki/AddressSanitizer) in Linux and CMake cross-platform projects. For more information, see [AddressSanitizer (ASan) for the Linux Workload in Visual Studio 2019](https://devblogs.microsoft.com/cppblog/addresssanitizer-asan-for-the-linux-workload-in-visual-studio-2019/).

- Integrated Visual Studio support for using C++ with the Windows Subsystem for Linux (WSL). For more information, see [C++ with Visual Studio 2019 and Windows Subsystem for Linux (WSL)](https://devblogs.microsoft.com/cppblog/c-with-visual-studio-2019-and-windows-subsystem-for-linux-wsl/).

## IncrediBuild integration

IncrediBuild is included as an optional component in the **Desktop development with C++** workload. The IncrediBuild Build Monitor is fully integrated in the Visual Studio IDE. For more information, see [Visualize your build with IncrediBuild’s Build Monitor and Visual Studio 2019](https://devblogs.microsoft.com/cppblog/visualize-your-build-with-incredibuilds-build-monitor-and-visual-studio-2019/).
 
## Debugging

- For C++ applications running on Windows, PDB files now load in a separate 64-bit process. This change addresses a range of crashes caused by the debugger running out of memory when debugging applications that contain a large number of modules and PDB files.

- Search is enabled in the **Watch**, **Autos**, and **Locals** windows.

## Windows desktop development with C++

- These C++ ATL/MFC wizards are no longer available:

  - ATL COM+ 1.0 Component Wizard
  - ATL Active Server Pages Component Wizard
  - ATL OLE DB Provider Wizard
  - ATL Property Page Wizard
  - ATL OLE DB Consumer Wizard
  - MFC ODBC Consumer
  - MFC class from ActiveX control
  - MFC class from Type Lib.

  Sample code for these technologies is archived at Microsoft Docs and the VCSamples GitHub repository.

- The Windows 8.1 SDK is no longer available in the Visual Studio installer. We recommend you upgrade your C++ projects to the latest Windows 10 SDK. If you have a hard dependency on 8.1, you can download it from the Windows SDK archive.

- Windows XP targeting will no longer be available for the latest C++ toolset. XP targeting with VS 2017-level MSVC compiler & libraries is still supported and can be installed via "Individual components."

- Our documentation actively discourages usage of Merge Modules for Visual C++ Runtime deployment. We're taking the extra step this release of marking our MSMs as deprecated. Consider migrating your VCRuntime central deployment from MSMs to the redistributable package.

## Mobile development with C++ (Android and iOS)

The C++ Android experience now defaults to Android SDK 25 and Android NDK 16b.

## Clang/C2 platform toolset

The Clang/C2 experimental component has been removed. Use the MSVC toolset for full C++ standards conformance with `/permissive-` and `/std:c++17`, or the Clang/LLVM toolchain for Windows.

## Code analysis

- Code analysis now runs automatically in the background. Warnings display as green squiggles in-editor as you type. For more information, see [In-editor code analysis in Visual Studio 2019 Preview 2](https://devblogs.microsoft.com/cppblog/in-editor-code-analysis-in-visual-studio-2019-preview-2/).

- New experimental ConcurrencyCheck rules for well-known Standard Library types from the \<mutex> header. For more information, see [Concurrency Code Analysis in Visual Studio 2019](https://devblogs.microsoft.com/cppblog/concurrency-code-analysis-in-visual-studio-2019/).

- An updated partial implementation of the [Lifetime profile checker](https://herbsutter.com/2018/09/20/lifetime-profile-v1-0-posted/), which detects dangling pointers and references. For more information, see [Lifetime Profile Update in Visual Studio 2019 Preview 2](https://devblogs.microsoft.com/cppblog/lifetime-profile-update-in-visual-studio-2019-preview-2/).

- More coroutine-related checks, including C26138, C26810, C26811, and the experimental rule C26800. For more information, see [New Code Analysis Checks in Visual Studio 2019: use-after-move and coroutine](https://devblogs.microsoft.com/cppblog/new-code-analysis-checks-in-visual-studio-2019-use-after-move-and-coroutine/).

**Visual Studio 2019 version 16.1**

New quick fixes for uninitialized variable checks. For more information, see [New code analysis quick fixes for uninitialized memory (C6001) and use before init (C26494) warnings](https://devblogs.microsoft.com/cppblog/new-code-analysis-quick-fixes-for-uninitialized-memory-c6001-and-use-before-init-c26494-warnings/).

## Unit testing

The Managed C++ Test Project template is no longer available. You can continue using the Managed C++ Test framework in your existing projects. For new unit tests, consider using one of the native test frameworks for which Visual Studio provides templates (MSTest, Google Test), or the Managed C# Test Project template.