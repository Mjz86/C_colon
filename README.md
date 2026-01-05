 C colon , Express colon and the mcc toolchain:
 
 
 
 whats c colon

![Logo](https://github.com/Mjz86/c_colon/blob/main/icon/c_colon.png)


a language inspired by c++ and rust, and some functional principles.

this language aims to be in the "c++ successor" language  categories, 
having c++ like syntax but with memory safety ,
aiming to be able to express c++'s full power while  freeing the language from the abi stability nightmare  that the wg21 standard committee made and the stack consuming windows calling convention ABI.

this languages goals include:


1. the option of performance at the cost of verbosity.
2. striving for zero cost abstractions.
3. safety as the default, with unsafe as escape hatch
4. compile time code.
5. multi paradigm language.
6. abi stability with ever changing libraries: the recursive hash abi , ( similar  to the dependancy hashing in caching systems) and its cycle braking operators aim to solve the abi stability nightmare,   see the details in the mcc document.
7. type qualifier deriven optimizations
8. reflection the verbosity away.
9. JIT constexpr code execution.
10. unique programming and debugging experience.
11. easy to use package management with cpp like compile times.




express colon  or "E:":

a language  for those who like simplicity while writing c colon.

this language aims to be in the simple systems languages category.

this languages goals include:

the first and foremost goal is simplicity,  in contrast to c colon.
the second goal is being exceptionally safe ( pun intended).
the third goal is being blazingly fast ( lower priority than simplicity though).
 i categorize them , and show what is needed to achieve these :


1. deep integration with c colon( safe only subset).
 2. simplicity.
3.  safety :type safety,  thread safety with cancelation safety and structured concurrency, lifetime safety, memory safety,  leak safety,  exception safety,  contract violation ( instead of UB ) safety , functional-like safety ( refrences are disallowed , instead input and output and inout prams are used and ect...), see how all of this is achieved in the mcc document. 
4. speed  : systems language with no GC and no deafult runtime ( other than cases where using the standard asynchronous frameworks)
5. rich set of libraries. 
6. easy to use package management with abi stability.
7. safe with rare  borrowing errors.
8. fast enough application logic.
9.  elegant parallel programming with safety and structured concurrency.
10. easy errors.
11. exception safety: with the unique and fast with result like speeds exception system,  and the integrated contact system,  developers can do fast and easy at once.
12. value and data oriented code .





MCC:

a toolchain with a radical abi , designed to have the best theoretical calling conventions and a fast exception system, 
with  parallel compilation and a cargo like module system,  while supporting dynamic linking, and with the recursive abi , having no fear of braking odr .
the abi calling conventions is very radical  ( dual return address or table, in out inout registers, used and unused registers ect)
 for more details,  see the mcc document 
 
 ---
    Copyright (C) 2025 Mjz86 

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

---

refrences

 
 Mjz C colon summary:
https://github.com/Mjz86/c_colon


Mjz  colon compiler design and details:
https://github.com/Mjz86/c_colon/blob/main/mcc.md



---
