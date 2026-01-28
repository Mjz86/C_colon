
c colon lang , its brothers and the mcc ABI 



> introduction







in this document, we specify the application binary interface ( ABI ) for c colon programs: that is, the object code interfaces between different user-provided c colon program fragments and between those fragments and the implementation-provided runtime and libraries. this includes the memory layout for c colon data objects, including both predefined and user-defined data types, as well as internal compiler generated objects such as virtual tables. it also includes function calling interfaces, exception handling interfaces, global naming, and various object code conventions.



in general, this document is meant to serve as a generic specification which can be used by c colon implementations on a variety of platforms. this is inspired by the famous Itanium ABI , there are sometimes assumptions of 64-bit targets, it is usually straightforward to recognize these non portable assumptions and translate them appropriately, e.g. by replacing a 64-bit pointer with a 32-bit pointer.



this document is not an authoritative definition of the c colon ABI for any particular platform. platform vendors retain the ultimate power to define the c colon ABI for their platform. platforms using this ABI for c colon should declare that they do so, either unmodified or with a certain set of changes.



also, this is generally incomplete, the c colon spec and mcc ABI of that spec will be more refined in each revisions.



the formatting of this document is currently not very well, under scores are not meant for italic.





---



what's c colon



![Logo](https://github.com/Mjz86/c_colon/blob/main/icon/c_colon.png)





a language inspired by c++ and rust, and some functional principles.



this language aims to be in the "c++ successor" language  categories, 

having c++ like syntax but with memory safety ,

aiming to be able to express c++'s full power while  freeing the language from the ABI stability nightmare  that the wg21 standard committee made and the stack consuming windows calling convention ABI.





this languages goals include:





1. the option of performance at the cost of verbosity:

the mcc toolchain needs as much information as it can about the program,

this information helps immensely in optimizations and it also makes the intention of the developers clear.

the language has lifetime annotations,  borrow rules , qualifier rules , pointers and many sorts of reference, operator overloading , a powerful context-type for explicit  control over many implicit operations,  all trying to make it safe and efficient to execute.



2. striving for zero cost abstractions:

each operation that can be optimized at compile time will be optimized at compile time,

the link times in mcc may increase from the calling conventions burden, but it's for the runtime.





3. safety as the default, with unsafe(...) as escape hatch:

similar to rust, c: has a borrow checker, a powerful one, the local safety and borrow rules prevent global uninitialized access, use after free and more.

similar to rust, c: has unsafe blocks, these unsafe blocks are specified with their safety control, for example unsafe(pointer-use) or unsafe(pointer-cast), unsafe(unrestricted), unsafe(variable) and more unsafe specifiers.

the rust language, although very fast, still lacks the option of non trivial moves, the option of elegant linked lists, the option of self referential small string/buffer optimization. it can be achieved with pointers, but it will not look good.

although like rust these are unsafe in c:, these are not totally disallowed, they are just unrestricted in c colon. unrestricted keyword aims to be of use for those who want fast iteration speed like in game dev,

although, at the cost of safety (using unsafe(unrestricted) ) and maybe performance (for example because its not safe to assume in the compiler that two spans are non overlapping so less SIMD usage).

note that some qualifier are unsafe to add or remove, for example the no alias qualifiers may lead to UB if removed so it has to be unsafe.

unlike  c++'s default of code not being able to run at compile time,  code using mutability and unstable, 

c: would use and be written with functional looking defaults,

for example a variable with no qualifier would be implicitly stable , const , safe and etc..

or an `inout` would be implicitly mutable , and so on,

these would make most code value oriented and functional,  

and a code similar to CPP with rust like safety would be achieved. 









4. compile time code:

the c colon spec aims to use way more compile time code.





5. multi paradigm language:

its like c++ but the legacy has been striped away.





6. ABI stability with ever changing libraries:

the recursive hash ABI aims to tag each component with all the dependencies with a hash, without the need for inline namespaces, and it even propagates.

this property of propagation through every type, function and namespace, makes us able to link everything old with everything new without ODR violations, and new components can just use a middle API to interface to new code.

imagine a world were the old std::regex was used for old code and the same std regex was 10 times faster when used new code both coexisting.

this truly makes it paying for what we used. the old programmer paid for old slow usage, but new code did not have to pay for the burdens in old code.

and each independent code that stayed unchanged did not need duplications, a dream for the CPP committee, and a possibility in mcc, the c colon ABI .

the hashes although not cryptographically secure are big enough to be unique, basically like a UUID (because its just a name mangling scheme).

if there's a cycle in the hash dependency chain, the problem is ill formed and notified of the need to use the  `abi=`  operator that doesn't rely on something  else at least one node in that cycle,

although  a rare occurrence might be that  the f() , `abi=(f())`  when calling the `constexpr` f , itself needs f, so , while such issues cannot be avoided due to the halting problem,  its still relatively easy to fix when noticing the `constexpr` max evaluation steps has been reached. 

also , using `abi=` on a member doesn't change the members real type and hash , and even if in some cases it  does, its still possible  to `unsafe(abi-cast)` the hash to the original type's hash ( variable definition having the type be type1 `abi=(abiof(type2))` or at most an unsafe `reinterpret_cast` known as pointer-cast or a bit cast)
 
 there's also an optimization i call hash elision,  under the as if rule ( as if the hash was calculated even if the calculation didn't happen) ,
 note that the visible symbols in the binary are  all fixed size 256bit backend hashes 

also for common graphs where the dependency  chain of the ABI can be resolved without using max evaluation,  the compiler gives the heuristically best point in the cycle to do the breaking , also patterns prone to hit the max evaluation ( doing a complex calculation to get the value of `abi=`) generate a warning.


 * as a gist :

It's similar to Itanium,  however, 

The name has an appended `xxhash128` , of its signature, 

Every type function and namespace has these hashes , and they recursively depend on each other ( for a cycle,  like in a graphs node type , there are operators such as `abi=,abi+,abiof` that operate on the 128 hashes)

This makes the language entirely ABI stable , at least within the ecosystem



7. type qualifier driven optimizations:

the qualifiers change throughout the code, the same expression have lead to different qualifiers, its ill-formed, but this makes uninitialized variables truly safe to use.

because the function either throws or initializes them, if not initialized it is an error.







8. reflection the verbosity away:

each namespace has an special function, implemented by default, it adds context objects to functions, and automates many of the verbose writing

for example  a var is stable const restricted safe and etc. by default,  no need to opt in , 

the reflection functions work on the AST , after all the de reflection Operations are completed,  the AST can fully turn into IR,

these operations can happen in parallel,  if they are located in separate dependency chains .

similarly the ABI hashing is also done in parallel but is lazy hashes and cashed once completed based on dependency chain, although the `abiof` operator might make it happen sooner than usual. 








9. JIT `constexpr` code execution:

each function  even in its template form can be turned into IR , because mcc IR is different,  it has two types , `constexpr` IR and `mutexpr` IR ,

the borrow checking and lifetime management itself is `constexpr` IR with assertions.

`constexpr` IR is necessary  ran in the compiler , so , even if the template types are unknown,  the IR is generated to  retrieve the template type then de-reflect the type away in just-in-time generated IR.







10. unique programming and debugging experience:

The debugging would be in the context types ,

Async debugging is hard in other languages,  but seamless in c colon because the stack trace is automatically build in the context functions for debuggability when we unwind,

the explicit allocator problem in c++ is also eliminated , making using debug allocators easier, 

the contract violations are captured in the context object,  vs the static global violation handler function 

async destructors work with the `co_value` keyword  and much more.







11. easy to use package management with CPP like compile times:

like cargo , c colon has a package management system integrated in the tool chain ,

unlike header files and CPP files, c colon aims to be more modular , each file being a module, like c++ modules,  while allowing for static and dynamic linking of libraries like in c++,

each module has hashes of its dependent modules, to help cashing and compile times ,

because the AST can be converted to IR via JIT , the compiler can  use many cashing schemes for templated entities and namespaces 

, each component and modules in the package management system have precompiled some IR and AST steps , that make it easier for compilers to use them if more download speeds result in faster compilations , if not the compiler can do it by hand without IRs.









what i think will be the strategy in this language to make it easy



beginners:

avoiding most qualifiers and completely, sticking to using mut when necessary, 

sticking to value semantics, for functions,  `inout` , in , out  and non reference,  reference-like alternative that don't cause much confusion, 

and coping most things , occasionally using more complex code.

avoiding any borrowing and  unsafe.



developers who use libraries:

writing value oriented,  functional-like mutability avoiding code , knowing how to design easy to use APIs , with OOP and more to use 





library authors:

writing complex performant code that can be used easily using value oriented designs



low level library authors:

using the language  to its fullest,  writing effective and efficient code without worrying much about ABI braking, 

using the latest technologies and theories to develop code that can be understood well by the optimizer, 

writing utilities that help automate rust, c++ and other language bindings and more.









---



a note from Mjz86 on what to do in "c:":



try not to get boggled down with ways to make it faster,

it is rewarding,  but its hard, the easiest way to write safe code in both rust and c: is to do more functional programming ( E colon ),

try only focusing on what needs to be optimized before making it hard for yourself, 

the compiler isn't there to demand the best code ,  it's not bad to mostly write value oriented code,

those who get boggled down in mcc or rust lifetimes want speed , but c: isn't just about speed,

its about speed if you want it , yes, but its about easier libraries , and not needing to link everything statically like rust,

its about writing embedded code knowing that your const declared variable wont change or be casted to mutable ,

and about moving away from the burden of ABI legacy , from killing the standard library's networking proposal because its not gonna be ABI stable,

its about using a regex without saying that PHP's regex is faster.









---



considerations



with these goals in mind , i aim to improve this spec , to make it practical, i often start with over engineering things , then chop unnecessary additions, to make it more practical, as of right now , were in over engineering phase.

this ABI aims to have minimal compatibility with the Itanium ABI , such that an `extern "c++" represent_cxx` statement can make most code usable in a c++ platform,

 some concepts may be excluded from such inclusions , for instance , a c++ type's declaration must be expressible in c colon and vise versa .





---



definitions



value qualifiers



volatile / nonvolatile( default)



a change, read or write in a volatile used region of memory may not be optimized out.

 volatile has to come with the unstable+unrestricted qualifiers , similar to other unsafe qualifiers , it is incompatible with them by design,






mut / const( default)



a change, read or write in a mutable region of memory is allowed
const unstable is not a truthful constant,  similar to c++ that defines const cast ,
i want the const correct code to benefit from optimizations,
but code that is not truthfully const correct cannot assume stability,  and therfore can exist in the unstable region, allowing  the ability to express what a c++ constant is(all neccecery for `represent_cxx`  to say the correct story ).
however stable const must not be casted away , and is a truthful statement of constness.





 




unstable / stable( default)

 the stable qualifier is statically proven when paired with restricted via the borrow checker , otherwise the program is ill-formed. 



for a pointer p declared within its lifetime L ,   to an stable const region of memory r, if a stable value const v loaded from address a originating from p is loaded from memory , then until the end of L , the expression `std::memcmp(std::addressof(v),a,sizeof(v))==0` must be true( the values at that memory  region must not be modified) , otherwise the behavior is undefined.



 for a pointer p declared within a code  lifetime L , to an stable mut region of memory r, if a stable byte/bit const v stored to address a originating from p is used , then until the end of L , the expression `v== (byte/bit)*a` must be true , if not , a value has been stored to address a originating from p in L or by a function called in L who is given a non const-stable pointer originating from p , otherwise the behavior is undefined
 


 note:

 1. mut-stable is like restrict in c, but a const-stable is like a rust constant

 2. its dangerous ( unsafe(unrestricted-stable)) for stable values to be declared unrestricted, it is as if a c restrict isn't proven to not alias.

 

  `stabilized` (deafult)/ `unstabilized`:
  stabilized is  a recursive qualifier,
  meaning that all subobjects must be either stabilized or casted as stabilized via unsafe(stabilized-cast),
  and if pointers , pointing to other stabilized types.
  having a stabilized object means that  the object and all its sub-objects are stable ,
  and all regions pointed to by the objects are stable as well.
  note that stabilized is a very strict stability grantee, a gold mine for the optimizer,
  for example an stabilized const bit pointer freezes all memory that it aliases, 
  and any pointer read from it is also freezed.
  however we recognize that not all objects can be stabilized.
   an example of an unstabilized `thread_safe`  stable object is the `arc_t<mutex<T>>`.
  however  `std::rc_t` is `thread_unsafe` stable unstabilized.
   note that a cast that disregards stabilized is unsafe,  and may lead to undefined behaviour.
  



`thread_safe` / `thread_unsafe`:
`thread_safe`  is  a recursive qualifier, we can think of it as a weak version of the stabilized qualifier. 
 these qualifiers are typically used as indicators of whether or not something is safe to pass and use between threads , the default is , 
any stabilized variable is thread safe and otherwise  unsafe, the stable mut is exclusively owned , and so safe to pass around, however, if it is not stabilized its not thread safe , so in any async boundaries the c colon libraries can notice via reflection that a mutable thread unsafe variable is used, so they can disallow it.
 also , if a member  is thread unsafe in E: ( not C:) , because of the qualifier visibility ban and the unsafe ban , that qualifier cannot be overridden by using a thread safe type with an `aliasset`.
`unsafe(thread_safe-cast)`  can also be used to cast the qualifier back ( for example the std mutex internals  are not safe , but yhe mutex itself is)
 note that , unlike the stronger `stabilized` grantee , `thread_safe` is not a compiler truth, its a developer promise.

uninitialized /initialized/ intermediate/ erroneous



a read from uninitialized valued via unsafe pointer  is undefined and without it  ill formed, but an store is valid, and will make  the qualifier go away.
however,  while the initialized qualifier requires  that value is safety readable,  and uninitialized says that its not readable, 
the inintimidates qualifier is more relaxed ,
we can use pointer optimization fences to  allow code to work while not adhering to the one qualifier set per expression rule.
 for example the new  and renew have an intimidate pointer parameter,
 that doesn't mean that all bytes are uninitialized , it means that all bytes can be uninitialized, 
 the c++ memory model like pointers are intermediate.
erroneous is more of a relaxed form of uninitialized that has defined behavior when its read.
only read of initialized and erroneous values is safe,
however read of other intermediate is unsafe(intermediate).



unrestricted / restricted( default)



unrestricted disables the exclusive mutability borrow rule in the compiler.

 unrestricted often has to come with the unstable qualifier so explicit stable exclusive or unstable qualification is required.







`represent_cxx` ( default only in fundamental types  `cxx_T_t`)/ `no_represent_cxx`(default in any other type):

for this to be used  many rules need to be checked  to have a cxx compatible type , 

for example  the alignment must not be fractional and must be at least `alignof(cxx_char_t)`, 

these types must be representable under the Itanium spec, for example virtual classes with `represent_cxx` must follow the Itanium spec for ABI layout, and functions with this qualifier (such as virtual functions in the Itanium style v-table) must follow Itanium compatible calling conventions ,this requires some nasty trade-offs, for example , supporting unsafe interposition C colon and allowing two stage dynamic loaders ( the cxx one ignores mcc binary , the mcc one independently loads),
this deep divide between these two ecosystem's ABI is limiting performance for mixed language code ,
however , if we did use cxx, cxx did use lib unwind , so we paid for cxx and lib unwind and the whole std cxx lib. 
using cxx FFI is already unsafe , however mixing cxx and c colon types is highly discouraged.
and needs `unsafe(represent_cxx)`,  another thing is that the `abiof` operator cannot work on these types , the inclusion of these types in a non `represent_cxx` ABI dependency  mandates the use of `abi=`





`noaliastype`( default) / `aliastype`



`aliastype` means that any store/load/modification through this type can influence types outside the alias set.





`mutexpr` / `constexpr`



a value declared `constexpr` is known at compile time.





`no_dll_comparable_address`/`dll_comparable_address`(default)(dynamic loader used Function and variable ( any symbol) qualifier):
 this qualifier is not a contributor to the ABI hash or the name mangle,
 a function,  variable or storage space,  used in the dynamic shared  library, may have a different address to the static function, a `dll_comparable_address` mandates that the storage address is unique,

 comparing two variables with  `no_dll_comparable_address` 
 is dependent on dynamic execution order of the loader and its given priorities,
 in a binary with  `dllexport` we will get that export's  address when we store it and  it can also be inlined freely ( because of modular builds within a single parallel translation unit we don't really have static linking ODR violations in mcc because they are ill-formed , but multiple translation units can benefit from the abi operators. ).
 however in a  build with `dllimport`   , we get the `dllexport` address with highest priority at dll initialization ,
when using this qualifier ,  we must use `unsafe(no_dll_comparable_address)` .

 a symbol of `dll_comparable_address` definition  is required to be loaded in start up and unloaded at exit,
this means that all linkable binaries with this qualifier  must be available when the program starts,
the program  will pick the highest priority between these , and also , 
no inlining is allowed for any function called with this qualifier , except when `dllhidden`.
the addresses of these variables is overridden at load time , and the memory  section is set to read execute for the duration of the program. 
 
 
 `interpositioned`: 
 this is a contributor to the name mangle, and if not `dllhidden`, has the overhead of both an atomic load and no inlining  is allowed at all.
there is an `unsafe(interpositioned)` qualifier that can use `set_interposition(fn,address)` to store ( memory  order release) function address on a global atomic static , using  the address-of on this symbol will load by a memory order of acquire,
or `atomic_interposition(fn)` which gives an `fn**` to manage the atomic by hand.
therefore the address  is controlled by developers.
also , an `interpositioned` function  when exported can be set to null, `...f(...)...=0;`.
also , inlining is not allowed for these functions.
 note that  paring both  `unsafe(no_dll_comparable_address,interpositioned)` , is doubly unsafe, and no inlining allowed. 
if a call to an interpositioned function aquires a null , an implicit contract violation is made.




 `dllhidden(default)/dllexport/dllimport`( only on static symbols,  like functions or static variables): 
this qualifier is not a contributor to the ABI hash or the name mangle,
   makes the symbol an export/import for DLL linking by providing a symbol hash.

   also `dllhidden` is the default and removes the symbol  hash in the binary .
 
- the  semi-inlining  (Thin)LTO register reassignment optimization( enabled by `dllhidden`  or `dllinline` ):
the register assigner is allowed to change the calling convention of an internal symbol as long as all other call sites act as if it never happened, 
this is useful for functions that  often have the static call site visible and want to eliminate  register to register mov instructions ( copy of a register to another).
if a function pointer is taken , in particular,  if the cost of the function without reassignment is significant,  the static function is wrapped  and the dynamic function pointer is the pointer to a function with the   register assignment of the calling convention ( maybe even a thunk to the  function with atypical register assignment) .
for this optimization to be possible,  the function must be inlinable ,
and this is a way to use the inlining knowledge of that function without actually copying an inlined version.
this makes it so that the baseline performance of a static function call is effectively reduced to the minimum.
the register  reassigner is free to change all registers except the 2  stack pointer and the instruction pointer , however the other 3 registers can be reassigned.

- non trivial construction or destruction of static symbols:
while most symbols i mentioned are functions, static variables and other stuff can have symbols as well , and these may not be trivially destructable or trivially bit copyable.
if inlining  or  knowing the address is permitted,  we just act as if we own the symbol with the highest priority so its lifetime is similar to hidden symbols.
on module load , the symbols is constructed if the address of module storage matches the address of the static.
on module unload , the symbol is destructed if the address of module storage matches the address of the static.
this has the effect that , a dllexport static symbol  loses the predictability of static order , and is dependent on priorities in dll startup.
the case for construction or destruction of interpositioned symbols is however  harder, because it must be safe, and the symbol must be unique, ,

 we have an interpositioned symbol  S  that has initilization,
 and an atomic pointer PS  for   symbol S,
 and the pointer  pointer  PP.
  init flag  F.// doesn't need to be atomic.
 
 PS is initialized  with the address of PS ( technically the offset, but then converted into the pointer after load )
PP is initialized with the address of the highest priority symbol S'es pointer. (&SP)


on module load , we do(pudo code):
if(PP->compare exchange strong(expect=&myPS,value=nullptr,aquire release)) {
 init myS
  F=1;
 // now and only after now PP represents something other than a nullptr.
 PP-> store ( &myS, release );
}


and for destruction its :
if(F){ 
PP->compare exchange strong(expect=&myS,value=nullptr,aquire release);
deinit myS;

}
 
 // now we're good.

and for getting the value :
p= PP->load(aquire);
// the second representable nullptr is the one that is not constructed.
if(p==p)return nullptr;
return p;


although,  a warning will be omitted if the interpositioned symbol needs to be non trivial.
because the overhead is more , and it's also not immediately available after load time.
 think about it , you initilized an static dllexport  by calling f , now , f is not called , but g instead.
 this is rather unintuitive,  but thats dll interposition in a nutshell.
 ( this is also true for functions,  a `dll_comparable_address` function may do something like print yes , but if we override it by higher priority,  we can make it print no , even tho the source  says its yes),
 and this makes us have the last dll qualifiers :
 
 `dllinline`/ `dllinited`:
 this qualifier is not a contributor to the ABI hash or the name mangle,
 these two are the defualt when dllhidden,  or dll_comparable_address.
 but  ill-formed   in dllimport .
 - `dllinline` on function :
 the function is assumed to have the highest priority ,and therfore is inlined.
 however if the function address  is anything other than the right one during a call or a caputure of its address, its an implicit violation of contract.
 similarly,  the interposition load  occurs to check the contract, and conceptually if the contract failed  we would have just used a dynamic call  , but the important distinction is that we didn't  , because  of the violation ending the control flow through that path.
  - `dllinline` on  objects : 
  the object is assumed to be have the highest priority.
  however if the address observed is different, its an implicit violation of contract. 

- `dllinited` on  objects : 
 the object static (de)initilization sequence happens unconditionally ,
 but the object  is unused if its priority is not the highest.
 
 
 
 



- static linking qualifiers ( between modules and between translation units):  
   these qualifiers are not a contributor to the ABI hash or the name mangle.
   
-  `moinline`:
   a definition is processed with the root module in the tree in mind, meaning that until a static declaration uses it , it is not evaluated.
   for example,  a template specilization in the root can be used.
   this is deferred evaluation of the code graph.
  
  
 - `mostatic`:
  a static definition is one that only depends on the pervious definitions in the module tree.
  any  declaration  until  this point that is used within this scope is pinned as the only  declaration .
  if a code section after the original modified this declaration ,  its an ODR violation, and the program is ill-formed.
  however there is a nuance that if the abi hashes don't conflict,  its not counted as the same declaration, and that dllexport  symbols with diffrent priorities dont count as the same symbol.
  the reason for this is that in c we have header file and c files,
  header files compile dependent on the translation unit.
  but c files compile independently of each other.
  so , we need a way to combine this without the hassle of definition duplications (  least the duplication being optional insteadof mandatory),
  in c++ the inline , template and constexpr keywords allow this ,
  but at the cost of   duplication of symbol before the linker,
  we can mitigate this by using a more selective approach.
  a static code section is defined independent of   the whole translation unit,
  making a translation sub unit.
  
  - `mohidden`:
this , means that a symbol is only visible inside of this module.
this is not the default.

- `tuexport`( deafult):
this , means that a symbol is exported by the module,  but is be provided during link time of the translation units ( static linking)
- `tuimport`:
this , means that a symbol is imported by the module,  but must be provided during link time of the translation units  ( static linking)
- `tuhidden`:
this , means that a symbol is only visible inside of this translation unit.
this is not the default, because it may lead to symbol duplications in an static binary.


storage qualifiers



- external storage:

the storage semantic is unknown.

- static storage:

this value is valid for the entire program.

- `thread_local` storage:

this value is valid while the current thread of execution is alive.

- automatic storage:

value is in stack , automatically destroyed.

- dynamic storage:

value is in heap, manually destroyed.

- register storage:

value's storage is valid in the currant expression, but the address may change at any time with relocation, but for a given unique owning reference to the register its the same

- opaque storage:

value's storage is somewhere , but its valid at least till the current block ends

- `nostorage` storage:

no address is given for the value





`refexpr` / `valexpr`( default)



the address of a `valexpr` value may not be unique, the state may be used to represent other objects as well ,

its like `no_unique_address` in c++:

makes this member sub-object potentially-overlapping, i.e., allows this member to be overlapped with other non-static data members or base class sub-objects of its class. this means that if the member has an empty class type (e.g. stateless allocator), the compiler may optimize it to occupy no space, just like if it were an empty base. if the member is not empty, any tail padding in it may be also reused to store other data members.

also if reorder is possible , any padding bytes or invalid state may be used to represent such object,

if its a `std::flag_t` , even more possibilities are made , as long as other declared object's are preserved

also a value expression function pointer can be the same as other    function pointers( de duplications) . 
also a value expression string litteral  ( string litteral doesn't have null terminator in c colon unless explicitly using "\0" ) or trivialy relocatable and trivially destructable object can overlap with any static constant stable memory region.

an array of empty value expression structures is itself an empty structure,
subtraction  of two pointers of empty value expressions is ill-formed.

`reorderable/not_reorderable`(struct/class qualifier )



we can do rust-like reorder optimization if this enabled



`offset_dependant/not_offset_dependant(default)/member_offset_dependant/member_not_offset_dependant(default)`(struct/class qualifier,and type qualifier):



a `not_offset_dependant` type is a type whose inner structs can be scattered in memory when accessed,

 specifically ,a pointer to a sub object cannot be used to reliably get to the main object by a subtraction of the offset (other than a base class to derived class cast).

 but `offset_dependant` objects can use the offset to gain a pointer to the main object.

`member_offset_dependant` objects are the ones that specifically can be used for casts by offset , base classes are `member_offset_dependant` by default.
 
  `(bit_)offsetof` is ill-formed if a type is `not_offset_dependant/member_not_offset_dependant` , 
   a member pointer `T::*` is also  ill-formed.
   note that the value of the `T::*` is the same as the offset of value.
   however a null member pointer is  same as   ( assuming non `represent_cxx`)   `~((~0)>>1)` ( the sign bit only being set , other bits being 0)
   note that offsets can be negative in a virtual base class , 
   a dyn member pointer ` T dyn(...)::*` layout is implementation defined.








`forceref` / `unforceref`( default)



`forceref`  makes the reference a pointer-like type that hold the address of the referenced type ,

`unforceref` allows the reference to be a custom type , for example a const str& is more appropriate as a string-view like type than a reference to a string.





unaligned / aligned( default)



the alignment of this type might be as low as 1 byte,
however if the type itself is fractionally aligned the alignment can be as low as 1 bit.
unaligned data is almost always trivially relocatable , trivially destructable and without pointers ,  basically,  the requirements for a safe bit cast)to be useful , because its probably a network or external  packet anyway.




unsafe / safe( default)



a value modifier that makes usage of this value an unsafe operation.



 `no_virtual_rtti`/`virtual_rtti`(default)/`this_virtual_rtti`/`no_this_virtual_rtti`(default):
determine if rtti is emitted for the class, 
the v table  pointer has a flag in the alignment bits that can indicate if the v table only consists of functions or is more than that,
if no rtti is emitted,  dynamic cast will return nullptr on all cases ( but not on cast to byte/void pointer because the offset is available , and in cases where a base object is accessed it would need to be rewriten as a static cast anyway).
this means that the class is purely using inheritance for function calls.
the root is the one determining if rtti is enabled or not.
if the most derived class has `no_virtual_rtti` but a base class has `virtual_rtti`, the program is ill-formed.
 this binary choice is a bit irritating because sometimes we need only down cast , but we can just do that by using a member function, 
 for example: `virtual  my_down_cast( ...desttypeid...)`
 
- `this_virtual_rtti`:
the virtual cast base is in the castation-table if the most derived type is `virtual_rtti`.
-`no_this_virtual_rtti`:
the virtual cast base has no entry in the castation-table.
this is the deafult because most types don't use dynamic cast.



interface / final( default for types with a virtual table) / virtual / nonvirtual( default for other types)



- virtual:

 a virtual type is

 a type pointer + virtual table pointer.

 any virtual inheritance , polymorphism and etc., has an owner , but every other reference to it's sub types is virtually qualified.

 because there are no virtual table/base pointers in the type, but offsets and function pointers in the table.

- `nonvirtual`:

 a type with no v-table

- interface:

 an empty type with a v-table and/or virtual bases, its ` object_offset_current` is removed and the dynamic cast entry  in the castation-table is removed 

- final:

 a type with a v-table who's dynamic reference type matches the static type.





`noaliasset`/ `usealiasset`(default)



`noaliasset` means the alias set of the type cannot alias the type,

for example `intn_t` can alias the `uintn_t` and `mintn_t` types , but `noaliasset intn_t` cannot.

there's also a similar usage not for static types but for identifiers , of dclaration of two unstable memory regions of having noaliasset on each other, that is , saying that they will not alias one another.

theres also entangled stable pointers, note that the cast to stable can introduce UB if not used carefully , this is why unsafe(ptr-cast) is almost always necessary in "valid"( as in borderline not UB) usage of entangled pointers, meaning that we cast two unstable pointers as stable , but entangle them , to satisfy the stable definition ,
an example may be that :

a , b   are overlapping regions but independent of other regions.
 c , d , e may overlap in some way but independent of other regions.

we entangle a and b together. 
we also entangle c , d and e together.

we do a loop on a , b , c , d and e.
the compiler has some options:
0. see that they are all overlapping ( all being unstable, because if a and b were not entangled and stable , it would be UB):
 bad assumption,  less vectorized code.
 1. see that all regions are stable but ignore entanglement ( this is wrong and the program will be UB, but the standard  has defined this , so its not a valid compilation):
 invalid compilation.
 2. recognize that there are partially overlapping regions but some regions are not overlapping with others ( being able to make a ven diagram of overlap):
 the best way , it can do vectorized instructions if appropriate,  but also recognizes that the stability grantee is not as strong as an independent region of memory.
 
 



`mayelide` (default)/ `noelide`:
a pointer to mayelide region of memory may have flag bits inside , its layout is implementation defined.
conversion from a elidable pointer to a non elidable pointer loses information and is considered a pointer use ( the conversion back produces an  elided flag of true), 
using the function `std::get_elided(p) ,std::set_elided(p,flag)`





borrowed / referenced / owned



- referenced:

object is borrowed elsewhere,object cannot be used.

-  borrowed:

can be used but does not drop.

- owned:

an owned object can use and must drop after use.




 `drop_on_throw/presist_on_throw,drop_on_ret/presist_on_ret,xvaluexpr, rvaluexpr/lvaluexpr , (i)(o)valuexpr`(function arguments qualifiers): 
 drop on X makes it so that the object is  when X , (`xvaluexpr` is unconditional drop),`rvaluexpr` promotes move semantics.
 `(i)(o)valuexpr` does the automatic `in(i)/out(o)/inout(io)/inval(none)` dropping semantic to the current reference or if trivial , via register 
 

determines the general usage and call convention in a function argument.

- `valuexpr` ( translates to `mut ivaluexpr`  or the `in-val` register):

a stable  mutable value that is initialized on the call site. the value qualification

- `ivaluexpr`:

a stable const value. the input qualification

- `ovaluexpr`:

a stable uninitialized mutable value that will initialize the caller argument after a successful call. the output qualification

- `iovaluexpr`:

a stable mutable value. the `inout` qualification
 








`qualiexpr(T V)/qualiexpr()` ( default):

an special qualifier.

 V itself has a type with its qualifiers, 

 the value of V can be changed by the `constexpr` code section by `qualiexprof(identifier)`  that access the V inside (needs to have `unsafe(qualiexprof)` , because E colon does not need this level of power).

 any function attempting to capture this qualifier inside its scope needs to be a template,

 however , the outside of scope qualifier checks can still be there without changing the function signature;

c colon code:


`template<autoexpr T-outside>`

`...requires... consteval pre... consteval post...`

`template<autoexpr T-inside>`

`...requires...`

` ...f( T-outside (<-optional) name (<-optional):T-inside,... )...;`

T-outside is the complete outside type, the checks of the outside , however the T-inside is what the function body actually gets as the argument type of name, 

however if there is a dependency from T-inside to T-outside, the function is considered fully templated , but if not , the function body is considered to be non templated , but the function call site still does execute the requires clause, which may contain checks or info about the type,

for example , a safe function may need to get a sorted vector ,  but the vector cant really prove without O(n) ops that it is sorted ,  and we really don't want static overhead , 

we can now declare a `qualiexpr(bool sorted_flag=false)` , (the empty qualiexpr being recognized as not sorted), and declare that the class member has `qualiexpr(bool sorted_flag=true)` and any operation that preserves that invariant are valid, and therefore we can statically assert that a binary search is not undefined behavior.







- `qualicast<Q>(identifier)` :
unsafe(qualicast), is used to cast qualifiers away or cast them to existence, 
for example,  a logically  dropped value can be set as uninitialized without a destructor call.


 

- dyn :
dyn is more of a prefix on key words.

- dyn(vals...) struct/union/enum...:
a structure that will have a dynamic memory layout based on  specific vals ( vals are of type `size_t`),
based on the alignment, which can used by (bit) alignof ,  its size can only be known  on an object that has been created in the sizeof or bit sizeof.
a dyn type can have other dyn types within it via specifying the val arguments, 
however a non dyn type cannot have dyn objects within it ( refrences to dynamic objects dont count)
either on the definition of a dyn object or on its declaration we need to specify the vals it uses,
if a dyn type has all values statically known it is considered a static type again,
however if the dyn type info is stored in a variable not known  at compile time or cannot be elided , 
the dynamic object creation might need to store it in the heap .


-dyn this: in a dyn struct , either only one member can have this , or its implicitly the same as the static structures,
by specifing a dyn this on a member you can set the address of that member as the address of that structure.

- `dyn (nullable) (cow) `:
an object declared dyn is as if its allocated in the heap via implicit elidable calls to (re)new and delete of the context-type , note that compiler may insert bytes before or after the allocated object depending on its needs ,
a dyn object member is a pointer to that member , if nullable it can be null , and access to it would result in an implicit  contract violation.
the cow qualification can be used on this object, creating any mutable reference will make the cow copy a unique reference to that copy , creating a copy of the pointer does a shallow cow copy using an implicit atomic reference counter before or after the object( can be padded in the compiler if alignment requirements say so).
the reason for this being a language construct instead of a library one is that the dyn(Q) purity qualification would be able to support trees with cow , this would help in making functional colon a viable pure language.
note that the byte  value, or size of the dyn pointer is implemented defined , implementations are encouraged to use a bit in the pointer to indicate if the allocation is elided, similar to what the coroutine handle does.

---



function qualifiers



- async(...),debug(...),optimize(...),lang(...): 
these don't really mean anything to the compiler , the are not  relevant to ODR ( the declaration is allowed to not include these while  the definition may) in c colon ,( not even used in the ABI hash), however E colon can put these, to allow the context-type to be implicitly changed via reflection to reflect that functions intent 
 for example  debug(std::debug::obfuscated) to do debugging in release or debug(std::debug::unwind) , to debug during unwind. 
 or lang(std::python) to make bindings.

-target(deafult)/target(...):
the target qualifier is a qualifier that allows a function to have implementation defined calling conventions, 
this qualifier's prameter must satisfy the  constexpr architecture concept , however the deafult is the architecture dependent ( based on compiler flag) conversion of `std::targets::current`.
the object specified in the target can do reflection on the function caller site and or callee site,  make a thunk, insert asm directives , ect. to achieve the specified convention.
note that conventions may be implemented in terms of implementation defined compiler intrinsics.
target is similar to `represent_cxx` , however,  if the c colon spec cannot be mapped well to the target , it requires `represent_cxx target(...)` to indicate that the target is not only architecture dependent,  but also outside of the mcc specification.
for example  `represent_cxx target(std::targets::thiscall)`.



- none(in terms of purity,default on dynamic calls or declared non visible symbols):
not having anything fancy at all.

- synth( the implicit is default on static definitions):
only static definitions can have synth qualification , the synth qualification either is ill-formed or transformed into the appropriate qualification on the ABI hash, after the compile , the synth engine also spits out an info dump on every explicit (not implicit) synth it made  ,
explicit synth puts more effort to synthesize the best it can , if the step limit is exceed the program is ill-formed, while implicit synth involves less trial and is more conservative if the heuristics show its not looking good.
synth analyzes the function body and does the purity qualification automatically , however its purity is easily breakable if the developer just  does a single wrong thing,
the explicit synth is more appropriate for formal proof engines or similar things,
there can be a compiler flag to also info dump on implicit synth and that hits at missed static optimizations opertonities.

- `weak_predictable`( default): 
a weak predictable call must not do a longjump or setjump ,  
an `weak_predictable`  expression must be composed of only other `weak_predictable`  expressions OR must be casted via `unsafe(as-weak_predictable)` .
the behaviour is undefined if the function  long jumps to non `weak_predictable` functions or is entered via a setjump or a computed goto.
this is the deafult because RAII  clean up code must run.

- `predictable`: 
is  `weak_predictable`  and, a predictable call must not do a longjump , terminate,  or a dynamic call ( through a function pointer or dll), 
an `predictable`  expression must be composed of only other `predictable`  expressions OR must be casted via `unsafe(as-predictable)` .
the behaviour is undefined if a predictable function  jumps to non predictable functions.
( this somewhat restricts our workload,  because we must either throw or return, and  terminate is ill-formed)
a critical system might require all functions to be predictable.
however,  for forcing an abrupt exit we can call std abort.

- `effectless`:

an evaluation of a function call is `effectless` if any store operation that is sequenced during the call is the modification of an object that synchronizes with the call; if additionally the operation is observable, all access to the object must be based on a unique pointer parameter of the function.
an `effectless`  expression must be composed of only other `effectless`  expressions OR must be casted via `unsafe(as-effectless)` .


- `weak_idempotent`:

an evaluation e is `weak_idempotent` if a second evaluation of e can be sequenced immediately after the original one without changing the resulting value, if any, and the program will behave as if the repeated count of this evaluation was irrelevant( in terms of contract assertions,  logging is a side effect that isnt relevant to the program flow).
any expression can be declared `weak_idempotent`.
however, the one qualifier per expression  on rule must still hold , meaning that a second evaluation of an `weak_idempotent` expression must result in the same qualifiers as the first one, adding to contract assertion safety .


- idempotent:

an evaluation e is idempotent if a second evaluation of e can be sequenced immediately after the original one without changing the resulting value, if any, or the observable state of the execution, an idempotent expression is also an `weak_idempotent` expression.

an `idempotent`  expression must be composed of only other `idempotent`  expressions OR must be casted via unsafe(as-idempotent) ( because some code that looks like it modifies really dose not, for example an i++ in an internal for loop) OR in the case of paring synth(idempotent) via compiler proof that it's IR block was idempotent , if none were true , the program is ill-formed



- `viewstate`:

a function f is stateless if any definition of an object of static or thread storage duration in f or in a function that is called by f is stabilized but not volatile qualified, 
no modifications to these values are allowed. 
an `viewstate`  expression must be composed of only other `viewstate`  expressions OR must be casted via `unsafe(as-viewstate)` ..

- stateless:

a function f is stateless if any definition of an object of static or thread storage duration in f or in a function that is called by f is const+stabilized but not volatile qualified, and is `viewstate`.

an `stateless`  expression must be composed of only other `stateless`  expressions OR must be casted via `unsafe(as-stateless)` ..

- independent:

a function f is independent if for any object x that is observed by a call to f through an l-value that is not based on a parameter of the call, all accesses to x in all calls to f during the same program execution observe the same value; otherwise if the access is based on a pointer parameter, there shall be a unique such pointer parameter p such that any access to x shall be to an l-value that is based on p.

an object x is observed by a function call if both synchronize, if x is not local to the call, if x has a lifetime that starts before the function call, and if an access of x is sequenced during the call; the last value of x, if any, that is stored before the call is said to be the value of x that is observed by the call.


an `independent`  expression must be composed of only other `independent`  expressions or is casted as `independent` via `unsafe(as-independent)`.






- `unsequenced`:

indicates that a function is `effectless`, idempotent, stateless, and independent.

an `unsequenced`  expression must be composed of only other `unsequenced` expressions OR must be casted via `unsafe(as-unsequenced)` ...

- reproducible :

indicates that a function is `effectless` and idempotent.

an `reproducible`  expression must be composed of only other `reproducible` expressions OR must be casted via `unsafe(as-reproducible)` .





- `mostly_functional`: 

 a function f is `mostly_functional` if The    returned or output-ed value ( via out )  by a call to f depends exclusively on, The values of its direct function arguments,The values of any non-volatile global, static, or thread-local memory observed at the time of the call, The values of any memory locations pointed to by its arguments (provided those locations are not volatile).

  f performs no write operations to any memory location visible outside its own activation record, including, global, static, or thread-local objects, Memory pointed to by its arguments (even if the arguments are non-const pointers), but excluding out and `inout` argument's value. 
  and f performs no write accesses to volatile-qualified objects.
  `viewstate` and idempotent .
  a `mostly_functional`  expression must be composed of only other `mostly_functional`  expressions OR must be casted via `unsafe(as-mostly_functional)` .
( basically gnu::pure if no `inout` is used) 



- `purely_functional`: 

 a function f is purely functional if The  returned or output-ed value ( via  out) by a call to f depends exclusively on The values of its direct function arguments , and   The values of any non-volatile stabilized constant  global or static objects observed

 and   f performs no read operations from non-volatile global, static, or thread-local memory that is mutable

 and f performs no write operations to any memory location visible outside its own activation record, including Global, static, or thread-local objects and Memory pointed to by its arguments ,even if the arguments are non-const pointers, 

 only the `inout` and out arguments are modifiable reference like arguments.

 and f and its callees performs no accesses to volatile-qualified objects.

 and f is `unsequenced`  .

   a `purely_functional`  expression must be composed of only other `purely_functional`  expressions OR must be casted via `unsafe(as-purely_functional)` .

 ( basically gnu::const if no `inout` is used) 


-`dyn (Q)`:
a dyn purity qualifier Q , can be declared on an expression,  this asserts the purity qualification Q , however,  the allocation and deallocation of dyn objects doesn't count in side effects of the program, 
because the usage of dyn objects is exactly similar to stack objects,  but with the difference that the compiler implicitly manages the memory via operator (re)new and operator delete in the context-type, and knows the lifetimes,  we can use this qualifier to our advantage, 
one reason being that F colon functions need dynamic arrays but still are pure. 





* note: throwing a trivially relocatable throw-value via a special hand crafted functional supporting context-types can still allow the throw expression to be `purely_functional` if the function is not `represent_cxx`, 
this is because  throw is not unwind based and doesn't use globals at all , and  can be used without any side effects nor pointer usage, if an exception is not possible , then use noexcept to complete the purity qualifications.

 

 

* these qualifiers have restrictions, for example a stateless function can only call other stateless functions a stateless function can only call other stateless functions in safe code, and cannot modify `thread_local` or `static` variables or access global non-stable mutable variables , many  of these qualified functions can only call functions that have these qualities. 





- `enumret`&`enumcatch`( this is implicitly given based on the type of the return,  if its an `enum` type with this specifier , then the function returning it would have this property)( function or `enum` type qualifier):

 `enumret` and `enumcatch` are similar to each other , but one applies to the normal return type , and one applies to the throw-value ( catching return type) ,

 both return pointers can have this property. 

 any function with these qualifiers  necessarily has to have a match expression in the call site ( or the catch site)( if throw-value  is this way , via operator catch(auto) , auto corresponding to the  `enum` entries) 
 
  * restrictions for  `enum`s specified of this use :
   all  `enum` entries must be continuous,if not the biggest and smallest one should not be more than 255( or an architecture dependent value ) values apart  ,( if the number is specified) ,
   warnings or errors will be given in cases where big number of entries generate massive jump tables or missed performance, typically anything more than 32( or an architecture dependent value )   entries gives a warning and anything more than 256 ( or an architecture dependent value ) table entries ( accounting for both `enum`s of throw and return together) is ill-formed, not because of could , but should , if we need 2 lookups (only 1 if continuous) and more than `8*255` bytes ( non continuous max before ill-formed) ( or an architecture dependent value ) ... we really aren't fast are we. 
   these architectural values can be queried.
  


 * definition of what  `enum` return and call means:

   the return pointer of that specific channel ( return or throw) would not be a pointer to the return address , instead,  it will point to a lookup table of return addresses,  corresponding to the `enum`  declaration order, that is handled in the return path.

   the way these functions are called ( necessarily having a match expression on the call site) makes it so that the return jump is to the match path corresponding to the `enum`  type,

   the table will be as big as the number of `enum`  entries.



- `noexcept(default in non critical system)/throws(default in critical system)`:

`noexcept` means the behavior is undefined if the function returns to the caller using the catching return register.

-  `noreturn/mayreturn(default)`:

`noreturn` means the behavior is undefined if the function returns to the caller using the normal return register.





- transactional memory:
( read the c++ specification , its mostly that but a bit more restricted)

the  atomic blocks have  a separate context-type that is created for the block and destroyed afterwards as if its a function call.
the transaction context-type must be a special context type  that is transaction safe, 
leaving an atomic block by any  means other than exception  commits the transaction.

0. `atomic_cancel`:
a transaction( its `transaction_safe` expression).
 depending on the context-type and the exception its either canceled, aborted,  or committed on an exception.

1. `atomic_commit`:
a transaction( its `transaction_safe` expression).
if an exception is thrown ,   the transaction is committed normally.

2. `atomic_noexcept`: 
a transaction( its `transaction_safe` expression).
if an exception is thrown , std abort is called.

3. synchronized:
a synchronized block is a `weak_predictable` expression that executes the code block as if under a global lock, 
all outermost synchronized blocks execute  in a single total order , the end of each synchronized block synchronizes with the beginning of the next block in that order,
synchronized blocks that are nested within other synchronized blocks have no special effect .
synchronized blocks are not transactions  and my call transaction unsafe functions. 
leaving the synchronized block by any means exists the block ans synchronizes with the next block .
the synchronized block has a separate context-type that is created for the block and destroyed afterwards as if its a function call.


4. `transaction_safe`:
a transaction safe expression is a `weak_predictable` expression that can only use  stabilized trivially relocatable  objects,  
or if not stabilized,  the value must be transactional  nonvolatile qualified and be trivially relocatable .
 a `transaction_safe`  expression must be composed of only other `transaction_safe`  expressions OR must be casted via `unsafe(as-transaction_safe)` .
if a transaction safe function is idempotent,  it has better optimization capabilities.
a transaction safe function has implementation defined calling conventions .


5. `transaction_safe_dynamic`: 
 a dynamic function  call ( call through the function pointer) that is `transaction_safe`

6. transactional ( value qualifier):
a transactional memory region can only be accessed inside a transaction.
the layout of a transactional value  pointer is implementation defined, 
the  object must be trivially relocatable  to be transactional.


7. `optimize_for_synchronized`:
indicates that a function definition is optimized for invocation inside a synchronized block.
the optimization preformed is usually merge of smaller synchronized blocks into bigger ones.






- `fastdyncaller(default)/fastdyncallee`:

  functions are `fastdyncaller` by default, 
 

  a `fastdyncaller` ( fast  caller site, callee save register) qualifier makes the dynamic call,  have no variables in the used set.

  a `fastdyncallee` ( fast callee site, caller save register) doesn't do much to the function's dynamic signature,  but  the registers in the used set increase a lot.

 

 - `nodyncontract/dyncontract( default)`:

 `dyncontract`  allows a function's contract to execute in the call site based on the callers contract evaluation requirements. 
 an `in-val` argument is implicitly passed to make contract violation controlled. 



  * the `fastdyncaller` transformation :

 this is intended  for functions that want minimal register usage in the functions who use the dynamic call,

 usually smaller functions with less used  registers benefit from the `fastdyncaller` specification, but those with many moving parts who already have large register usage are better used with the `fastdyncallee` qualifier. 

 if F's address isn't stored or used , then the transformation code  is optimized away.

 the `fastdyncallee` , on the other hand  doesn't have a transformation in the function's assembly.







 
// the `enum` ret and catch are similar , however they have a secondary lookup( often deduplicated across the link-unit )    table with all the offsets+ret pointing to RC ,then RC finally does a secondary jump lookup to the caller table  , this is only one of the many ways of doing this , and we havent even done callee-inlining the thrunk yet. 
    RC:
    
    restore the catching return  to the return pointer.

    jump to ret.
 
   RN:

   restore the normal return  to the return pointer

   ret:

  restore all registers specified as used in F in stack.
  
jump to the return pointer.
 
 absolute value of `(&contact_checked_F-&dyn F)`

  dynamic F( where the dynamic pointer will point):

   save all  registers specified as used in F in stack.
 
  save both return pointers.
   
set both return pointers to &RN and &RC.

    static F:
  
  F's code...
  
  contact_checked_F: 

    contract's code...    

    

    

   

   a call to `&(dynamic F)` is made through the function pointer.

   





the difference to `fastdyncallee` is :




absolute value of (&contact_checked_F-&dyn F)

dynamic F( where the dynamic pointer will point):

static F:

F's code...

contact_checked_F: 

contract's code...    





---

exact mechanism of return pointers:



 conceptually,  we have 2 return pointers, 

 and or potentially table pointers,

 however,  most of these are offsets, 

 the real way we store them is in 4 cases( 2 and 0 being the most common) :

 -  special cases:

0. noexcept: 
 enumcatch is irrelevant, and the caller doesn't generate a sad path.

1. noreturn:
enumret is irrelevant,and the caller doesn't generate a happy path.
the end of the callee scope in the control flow ( return) is ill-formed,  it must be unreachable ( at most by calling std::unreachable and using unsafe(unreachable) ).

2. only 1 path is expected by the caller (ie: only return, only throw, only one jump table entry ):
the path's adress is given to `return_ptr`, and the `catching_return_ptr`register is treated as if it was not an special register.

3. only two paths expected by the caller (ie: only 2 return entry, only 2 throw entry, only 2 jump table entry ):
one path's adresss  is given in the `return_ptr`,
and the other pathes offset from the  `return_ptr` is stored in the `catching_return_ptr`.

4. noexcept and noreturn combined:
mostly used in terminate like functions,  no return pointer is provided and the base pointer is also not provided. 
only the stack pointer  and the instruction pointer are special registers, 
the used set is irrelevant because the caller cannot continue its execution.
a tail call is granteed.
having any out or inout prameters in the function signature is ill-formed, 
the context-type is an inval argument in such function signatures, and the callee is responsible for its destruction,
although,  these types of functions are unsafe(longjump) to call because,  well , its a terminated program and no RAII unwind code was executed.






0. no tables( `enum` ret) :

 `return_ptr`= the absolute pointer to the return path, calculated via instruction pointer in caller.

 `catching_return_ptr`=   the relative offset of the absolute path to the return pointer.

 

 

1. `enum` normal return table, catching return:

`return_ptr`= the absolute pointer to the first return path. 

 `catching_return_ptr`=`table_pointer`= absolute address of the table.

  -  table: 

    contains reletive offsets to the paths, the pointer points to the first return offset( granteed as zero)

  `{catching_return_offset, nth_retuen_path_offset....}`.

     

2. `enum` catching table, normal return:

`return_ptr`= the absolute pointer to the return path.

 `catching_return_ptr`=`table_pointer`= absolute address of the table.

  -  table: 

    contains reletive offsets to the paths,the pointer points to the first catching return offset, the catching offsets are accessible via positive indexes to the table.

  `{ nth_catching_return_offset....}`.

 

3. `enum` catching table, `enum` normal return table:

`return_ptr`= the absolute pointer to the first return path.

 `catching_return_ptr`=`table_pointer`= absolute address of the table.

  -  table: 

    contains reletive offsets to the paths, the pointer points to the first return offset( granteed as zero), the catching offsets are accessible via  negative indexes to the table.

  `{...(-n)th_catching_return_offset, nth_retuen_path_offset....}`.


- example:
 assuming that the return enum indexes are  non negative and start at 0,
 if not , we add an offset of the negative of the least enum index to make it start at 0 and be  non negative.

    
    //call site
    
    `{catchs={clables.....}, rets={lables.....}};`
    
    
  `CRA= IP+(rets-IP);`// constant offset from IP to support ASLR
 
`NRA= IP+lable;`
 
` jump func;`// IP=func

many lables:....

 .....happyy...
 
many clables:....

 ....sad.....
 
 
 
 func:

 `BP=SP;` // preserve CRA, NRA,BP  throughout the call
 
// stuff....

 // stuff...   of first enum entry ret

`SP=BP;`

// return of the first enumret entry is  not a table lookup 
 
`jump NRA;`// IP=NRA
 
 // stuff...   of other enum rets
 
`SP=BP;`

// return of the happy path is positive indexes,  index being the same as the return value's enum numeric ( or if the enum is a sum type , the type tag)

`NRA+=*(CRA+retval);`

 `jump NRA;`// IP=NRA
 
 
 // stuff...   of  enum throws
 
`SP=BP;`

// return of the sad path is  negative indexes( with -1 being the first),  index being the same as the  throw value's enum numeric ( or if the enum is a sum type , the type tag)

// the bit not is the way we use two's compliment to map the negative indexes ( the reason for using this is the following both path)


`NRA+=*(CRA+ (~throwval) );`

 `jump NRA;`// IP=NRA
 
 
 
 
// branchless both  path multi dispatch:
....
`SP=BP;`

`val=~throwval;`

 `val = throws?val:retval;`// cmove
 
`NRA+=*(CRA+ val );`

 `jump NRA;// IP=NRA`

   
   
// did you notice?  only 2 jumps, both being necessary ( one as call, one as ret)
    
    
    
    
    
    
    


--- 

the symbol table and dynamic loader:

 any c colon symbol with `dllimport` or `dllexport` qualifier has its cryptographic 256 bit hash stored in the binary, 

 this is for dynamic linking and the dynamic loader to be able to load the DLL.





- c colon symbols are *not* interposition-ed by default :

0. we do not want the overhead of the global offset table(GOT) by default, as it takes a toll on all calls from shared libraries even those thar are never interposition-ed (see the shared libraries `CppCon` video).

1.  it has an uneasy coexistence with the c colon ODR.

2. it is rarely used and can be mimicked by a mechanism like cxx `set_terminate`.

3. GOT being writable during the execution of the program is a security  problem because of being able to change the common functions to do something malicious( glaring attack surface).

4. we do not want the overhead of the procedural lookup table by default.

 





---

 constant sharing :

 
- in function  arguments when a stable byte B with address A is trivially relocatable within its type, if passed as a stable constant , we can trivially copy it to address B , we know it will not get changed,  and we know the original value will not change until it gets relocated back ,

   but, we can still use that byte B within other *constant* arguments,  that's because :

   1. B is within its lifetime. 

   2. B will not change until the scope ends.

   3. we can assume that each time    we access a copy of B , that its as if we trivially relocated it from the original.

   4. its as if we propagate a const reference borrow without the stable address  

   5. the callee will not drop the exclusive in parameters  in any code path.





-  immutable data structures and CoW: 

  there are many benefits to such structures , especially in a value oriented language like C colon , and by extension ,E colon.

  there might be more incentive on doing these styles of data structures for data oriented designs. 







- strings :

   string types heavily benefit from both SSO and COW ,

   in C colon,  each string has to have an encoding, 

   the most common encoding is the utf8 encoding, the standard library  should strive to support this encoding and be up to date on the Unicode standard.

  because of the language safety rules,  we have both thread unsafe string types for single-threaded like strings and we also have thread safe strings, 

   however,  constant strings are different from mutable strings , so , these are specializations on the common standard basic string type,

   the format includes `std::(m)(z)(u)string` , for the:

   0. m:

   mutability , it is not allowed for this string type to use copy on Write on the current buffer of text.

   1. z:

   the string mut be null terminated. 

   2. u: 

   the string can be thread unsafe in the way it is reference counted   

   

 --- 

 coroutines:



- context-type in the signature:

instead of the default `std::context_t<optimization-level>`  use `std::async_context_t<optimization-level>`  ( or equivalent non standard types )

 

0.  has to have a `noop` coroutine function  :

  
   `context-type-coro-return noop_resume( context-type-coro-input) context-type;`
   

1. has to have a promise-type ( dependent on function signature ) declared inside it , the promise type is the callee facing context type , because the callee always knows its coroutine status, it can always know if its the context type or the promise type as the context type , on each resume , the promise type is refreshed with new caller facing context types , but the promise within manages the callee.

2. has to have a `context-type-coro-return` and `context-type-coro-input` ( independent of the function signature) to be returned from a resume.



3. the promise  function's :

  - first function  that is called once resumed ,the input is typically the callers coroutine handle and other information :

  

  `promise_resumed(this promise&  self, context-type-coro-input,in is_cancled) context-type-> promise-cache;`

   

- last function that is called resulting in suspension: 

  note that the reason for using references instead of `inout` here is because the callee will probably throw , resulting in the drop of `inout` , but references don't drop self on throw.

  `promise_suspend(this promise& self, promise-cache,bool& is_cancled )context-type->context-type-coro-return;`

 

- (a)symmetric transfer:

if a promise wants ( decided in the `awaiter` suspension via returned `transfered_handle` being `noop` or not ) to do a asymmetric transfer , instead of the `promise_suspend` , the `promise_transfer` is called, they should have the same context-type to be compatible, the return value is typically the  coroutine handle and other information:

`promise_transfer(this promise& self,promise-cache ,in transfered_handle,bool& is_cancled )context-type->transfered-context-type-coro-input;`





- cancelation grantees: 

 an error type in the catch scope is either a base of the empty cancelation token type , or its a base of the common violation token or isn't either , for cancelation and violation catches ( a catch with these types) unsafe(ignore-cancelation) and unsafe(ignore-violation) is applied , but otherwise its safe to do in the coroutine,  also the  catch(throw-value), equivalent to catch(...) in c++  is also unsafe(catch-all-tokens).

 so , the c colon libraries can distinguish if the throw is a cancelation or not and do the appropriate safe thing for E:

 

- destroyed only when everything is canceled and "done()".



- promise-cache :

the promise cache is an object only visible in the promise, with lifetime between resume and suspend ( therefore  may flow in registers), its accessible as an `inout` like object to most inner functions , its mostly because the promise type's storage is hard to optimize because the caller can get it, therefore,  this can be used for intermediate variables in the coroutine,  for example the caller handle , 



 - ABI :

  the coroutine handle  is a pointer to the structure with the following layout:



` struct frame{`

  `context-type-coro-return (* resume_function ) ( frame* ptr, context-type-coro-input) context-type;`// fastdyncaller , and  dyncontract  by default 

 `intptr_t  program_switch_counter;`// positive indexes show normal control flow, negative indexes show the same suspension's catching/cancelation control flow,  0 shows that the function and all of its variables will be destroyed on next resume ( final suspend) .

 // if the function's last destination ( the counter being set to zero) throws by exception, the resume pointer will be reassigned to soly point to the frame deallocation destructor,  the frame wouldn't be destroyed,  but rather,  the exception would  be caught in the catch and stored on the stack  then the frame will finally be destroyed by calling the resume pointer again. 

 // if a the deallocation of the frame fails by exception the program will terminate ( we can assume free and delete will never fail so this isn't an issue) 

 ` byte[....];`// coroutine frame storage 

  

  // the fastdyncaller transformation is a bit different for asymmetric transfer,  each jump to a sibling restores the used set as if it was a return. 





 ` };`
  
  
  `__raw_handle{` // pack.
`frame* ptr;`// the alignment bits are used for the flags.
`flag_t elided;`// elided flag is in the least significant bit.

`};`

    

 // these are simplified code , the actual code would check to see if its a noop coroutine first .

  // all of these are `unsafe(explicit-coroutine-handles)` . 

// `bool    done() == (program_switch_counter==0 )`

// `bool elided()== (ptr&1)`


//note that a cancelation of a function is only observable while it hasn't  reached final suspension,  in the final suspend it had done all of its work so even if the resume throws an exception,  it has already been a finished routine. 

//  `bool canceled() ==(program_switch_counter<0)`

// it is necessary for safety that all coroutines are called with their context-type known ( context-type  is acting like the Itanium promise type)



// `context-type-coro-return  resume(context-type-coro-input) context-type {return resume_function(ptr...);}`

//  `void cancel() {ptr->program_switch_counter=-abs(ptr->program_switch_counter);}`// if there are no `co_value` expressions , then only one resume is necessary before its "done".

 // `context-type-promise & promise() ...`

  

   

   `std::coroutine_handle_t<context-type, std::meta(^^coro-function)>` is the non type erased handle , with superior static calls ( calling and optimizing via the reflection information) 

  , however the type erased one has an empty reflection value of  `std::meta_t`  , and does a dynamic call, and is needed for callees to preform the control transfer via resume on the caller.




---
 allocation and the as if rule:
  
  the  allocators ( defined via the `std::allocator_c` concept(s)) in mcc followed the as if rule ,
  if multiple allocations can be elided safety, they will,
  the allocators fall into two categories,  the ones that are thread-safe and the ones that aren't,
  thread-safe allocators are more expensive to use , although non thread-safe allocators can be used via channels , its still not a good choice.
 its similar to how cxx does coroutine frame elision, but this time the stack frame is also available. 

 however,  this time , we have lifetime tokens in the language ,
 so , an example may be:
 for a memory region M allocated with lifetime L between calls to elidable (re)new and delete  calls ,
 or a dynamic  object O that implicitly does allocate M ,
the compiler is allowed to provide the region M with other means than new and is obligated to not call the corresponding delete if it does so.
 analyzing the lifetime L , if L is within B , B being the lifetime of the stack frame or a region of memory R,  the compiler is allowed to expand R or the stack frame under the as-if rule.
 note that , doing so correctly may need meta data.
 
for example  a size typed capacity field behind the allocation region,
or a high bit in a pointer reference to such region.
or simply having a flag next to the reference that indicates elision.
in my view,  using some bits in the pointer is superior to addling a capacity feild.

there also might be ways to have a call to new or renew  produce a flag , and delete  to consume it.

 
 



---

 references( there are more combinations of qualifiers  , but the common ones include )

 

 out/in/`inout` T:

 

a value oriented reference-like type ,

for function arguments,  these don't necessarily mean that T will have the same address,  unless T isn't trivially relocatable, which will make T relocate into the stack in the caller.



 T&:
(`lvaluexpr` is default)


a typical l-value reference like rust , and if unrestricted , like c++.

its like `inout`,for passing around T without potentially changing the address of T , unlike `inout`.


its const is used in the copy constructors.


`rvaluexpr T&`:



a typical r-value reference like c++ but borrow  checked like T&, and if unrestricted , like c++.

its mut is used in the move constructors. 



 


`iovaluexpr T&`:


a typical l-value reference like `inout` , this is the answer to `inout` like semantics without the intent to steal.

its like `inout`,for passing around T without potentially changing the address of T , unlike `inout`.


if a function throws by exception,  this value is considered dropped/uninitialized if T is mutable .

however if returned,  this value would still be considered in its lifetime. 






`xvaluexpr T&`:



a dropping reference (like r value or x value),  meaning that after the lifetime  of the `xvaluexpr T&` the entity will be dropped.

used in the relocation constructors.



non `forceref` user defined references:



are user defined types who's purpose is referencing variables. 







---







Itanium-like definitions:



the descriptions below make use of the following definitions:



- alignment of a type t (or object x): a value a such that any object x of type t has an address satisfying the constraint that &x modulo a == 0.



- base class of a class t: when this document refers to base classes of a class t, unless otherwise specified, it means t itself as well as all of the classes from which it is derived, directly or indirectly, virtually or non-virtually. we use the term proper base class to exclude t itself from the list.



- base object destructor of a class t: a function that runs the destructors for non-static data members of t and non-virtual direct base classes of t.



- basic ABI properties of a type t: the basic representational properties of a type decided by the base mcc ABI , including its size, its alignment, its treatment by calling conventions, and the representation of pointers to it.



- complete object destructor of a class t: a function that, in addition to the actions required of a base object destructor, runs the destructors for the virtual base classes of t.



- deleting destructor of a class t: a function that, in addition to the actions required of a complete object destructor, calls the appropriate deallocation function (i.e,. operator delete) for t.



- direct base class order ( if `not_reorderable`): when the direct base classes of a class are viewed as an ordered set, the order assumed is the order declared, left-to-right.



- diamond-shaped inheritance: a class has diamond-shaped inheritance if it has a virtual base class that can be reached by distinct inheritance graph paths through more than one direct base.



- dynamic class: a class requiring a virtual table pointer (because it or its bases have one or more virtual member functions or virtual base classes).



- empty class: a class with no non-static data members other than empty data members, no unnamed bit-fields other than zero-width bit-fields, no virtual functions, no virtual base classes, and no non-empty non-virtual proper base classes. such type can be declared with no-storage qualifier.



- empty data member: a potentially-overlapping non-static data member of empty class type. such type can be declared with no-storage qualifier.



- inheritance graph: a graph with nodes representing a class and all of its sub-objects, and arcs connecting each node with its direct bases.



- idea of the inheritance graph order:
when making the `castation-table` in the compiler we need to generate it via a graph treversal, to make it as if we did that treversal at runtime.
 the ordering on a class object and all its sub-objects obtained by a depth-first traversal of its inheritance graph, from the most-derived class object to base objects, where:

    - no node is visited more than once. (so, a virtual base sub-object, and all of its base sub-objects, will be visited only once.)

    - the sub-objects of a node are visited in the order in which they were declared. (so, given class a : public b, public c, a is walked first, then b and its sub-objects, and then c and its sub-objects.)

    - note that the traversal may be pre order or post order. unless otherwise specified, preorder (derived classes before their bases) is intended.



- instantiation-dependent: an expression is instantiation-dependent if it is type-dependent or value-dependent, or it has a subexpression that is type-dependent or value-dependent. for example, if p is a type-dependent identifier, the expression `sizeof(sizeof(p))` is neither type-dependent, nor value-dependent, but it is instantiation-dependent (and could turn out to be invalid if after substitution of template arguments p turns out to have an incomplete type). similarly, a type expressed in source code is instantiation-dependent if the source form includes an instantiation-dependent expression. for example, the type form `double[sizeof(sizeof(p))]` (with p a type dependent identifier) is instantiation-dependent.



- morally virtual: a sub-object x is a morally virtual base of y if x is either a virtual base of y, or the direct or indirect base of a virtual base of y.



- nearly empty class: a class that contains a virtual pointer, but no other data except (possibly) virtual bases. in particular, it:

    - has no non-static data members and no non-zero-width unnamed bit-fields,

    - has no direct base classes that are not either empty, nearly empty, or virtual,

    - has at most one non-virtual, nearly empty direct base class, and

    - has no proper base class that is empty, not morally virtual, and at an offset other than zero.

    - such classes may be primary base classes even if virtual, sharing a virtual pointer with the derived class.



- non-trivial for the purposes of calls: a type is considered non-trivial for the purposes of calls if:

    - it has a non-trivial realloc-constructor `T( xvaluexpr T&);`, or if its realloc constructors are deleted.

    - this definition, as applied to class types, a type which is trivial for the purposes of the ABI will be passed and returned according to the rules of the base mcc ABI , e.g. in registers; often this has the effect of performing a trivial reallocation of the type.

    - if non trivial,  it is passed as if it had a stable `forceref` (i)(o)valuexpr qualifier, as if  passed by reference,  the relocation is allowed to be optimized out if the passed variable is (i)(o)valuexpr qualified or is a temporary at the call site, note that if the variable has internal mutability  as an input , it is ill formed to pass it by value ( input) and its relocation constructors cannot be trivial.



- potentially-overlapping sub-object: a base class sub-object or a non-static data member declared with the valexpr qualifier.

- primary base class: for a dynamic class, the unique base class (if any) with which it shares the virtual pointer at offset 0.

- secondary virtual table: the instance of a virtual table for a base class that is embedded in the virtual table of a class derived from it.

- templated entity:

    - an entity that is defined or created within a template, such as:

        1. an instantiation of a class, function, or variable template, including from a partial specialization, but not including an explicit specialization;

        2. a member or friend function definition of a templated class;

        3. an enumerator of a templated enum;

        4. a local entity in a templated function;

        5. an entity within a templated namespace or

        6. a lambda in a templated entity.

- thunk: a segment of code associated (in this ABI ) with a target function, which is called instead of the target function for the purpose of modifying parameters (e.g. this) or other parts of the environment before transferring control to the target function, and possibly making further modifications after its return. a thunk may contain as little as an instruction to be executed prior to falling through to an immediately following target function, or it may be a full function with its own stack frame that does a full call to

the target function.



 - interposition :

  overriding a symbol in one binary fron another. 






- memory orders( similar to cxx) :

The default behavior of all atomic operations in the library provides for sequentially consistent ordering . That default can hurt performance, but the library's atomic operations can be given an additional `std::memory_order` argument to specify the exact constraints, beyond atomicity, that the compiler and processor must enforce for that operation.


1. Relaxed : there are no synchronization or ordering constraints imposed on other reads or writes, only this operation's atomicity is guaranteed (see Relaxed ordering below). 

2. Acquire: A load operation with this memory order performs the acquire operation on the affected memory location: no reads or writes in the current thread can be reordered before this load. All writes in other threads that release the same atomic variable are visible in the current thread (see Release-Acquire ordering below).

3. Release :A store operation with this memory order performs the release operation: no reads or writes in the current thread can be reordered after this store. All writes in the current thread are visible in other threads that acquire the same atomic variable (see Release-Acquire ordering below) .

4. acquire release :A read-modify-write operation with this memory order is both an acquire operation and a release operation. No memory reads or writes in the current thread can be reordered before the load, nor after the store. All writes in other threads that release the same atomic variable are visible before the modification and the modification is visible in other threads that acquire the same atomic variable.

5. sequencal consistency :A load operation with this memory order performs an acquire operation, a store performs a release operation, and read-modify-write performs both an acquire operation and a release operation, plus a single total order exists in which all threads observe all modifications in the same order (see Sequentially-consistent ordering below).

- note: 
atomic refrence cannot  refrence fractional alignment types,
and an atomic with a fractional type inside will padd it.
the `std::atomic<std::flag_t>` is the only atomic that is granteed to be lock-free , however it has implementation defined layout and alignment ( as required by the compare exchange instruction  support in the hardware, which is virtually all multi-core hardware , or if not supported , an intrupt disable critical section in embedded single core architectures)
this is because an atomic load or store to a bit requires the byte to be synchronized accordingly, 
but a bit pointer doesn't use atomic operations so its a race condition , however to avoid it, a fractional atomic is  ill-formed.


no changes  were really made  from [the c++26 definitions](https://en.cppreference.com/w/cpp/atomic/memory_order.html) , as its a great well-defined memory model that c colon stands on.




















- vague linkage: the treatment of entities -- e.g. inline functions, templates, virtual tables -- with external linkage that can be defined in multiple translation units, while the ODR requires that the program behave as if there were only a single definition.



- virtual table (or v-table): a dynamic class has an associated table (often several instances, but not one per object) which contains information about its dynamic attributes, e.g. virtual function pointers, virtual base class offsets, etc.

- virtual table group: the primary virtual table for a class along with all of the associated secondary virtual tables for its proper base classes.





 layout:

 

in what follows, we define the memory layout for c colon data objects. specifically, for each type, we specify the following information about an object o of that type:



- the bit-size of an object, `bit_sizeof(o)`;

- the bit-alignment of an object, `bit_alignof(o)`; and

- the bit-offset within o, `bit_offsetof(c)`, of each data component c, i.e. base or member,
note that a dynamic class cannot have fractional alignment  for its bases, if it does , the program is ill-formed 




virtual table layout contains:
 
 - note for virtual base classes  : 
  for each virtual base class we have a special implicit function pointer entry corresponding to that base,
 the function pointer entry is more of a flag than a  code path, its  either ~((~0)>>1) ( member pointer null value) or  the base offset to the current( to avoid loading the base offset to most derived),
 if non null we know that we need to call the virtual base class constructor on the base offset, 
 if null value we know that the virtual base class is already constructed,
 only one  non null entry can correspond to a specific virtual base.
 if the construction is inlined and doesn't call virtual functions the constructor virtual tables can be elided.
 
 * the mandatory this pram advantage(  virtual table thunk sinks to the function):
  think about a virtual function signature 
  `virtual f(this virtual base&self)` , an override of this function will be using this signature,  so it would have the same pointer, 
  if an overrides signature doesn't match the actual function,  then we might do a thunk ( because of type conversion mismatch).
  
* the construction  object illusion and the restrictions:
`base( this virtual ovaluexpr base&self)`, the self reference is the actual real virtual table, 
however , passing the self ( even implicitly ) without using the  final qualifier ( to basically not ommit construction of itanium-like v table tables , which is a massive itanium section that i simply don't know how to remove the bloat from and is obscure enough to not include) gives an error (" type is not formally constructed,  virtual calls may have different effects than programming expectations  use `represent_cxx` to allow virtual table table dispatch"),
 also,  using final is basically a good way to say that the type is a standalone entity so even if a virtual call on it occurs,  it doesn't need to use a VTT  , however using  the final qualifier in this specific instance has also restrictions on having virtual bases.
 while we can reconsider this in future versions by changing the abi version,  i still think it's  fundemental requirement of grapth generation( n bases means n tables  and means n nodes , so graph , incompatible with castation) will make the cost too much, the VTT is  `represent_cxx`along with personality routines and array cookies.




- type-v-table( 64 byte aligned) :

  0. `castation-table`( used in dynamic cast) :

  we have the search table , to do a binary search,  for each type , there might be a different lookup table depending on class visibility and the hagiarchy,

  but , for this type , all the accessible types are in the lookup, this , although taking up more space,  is more efficient than a graph traversal algorithm at runtime,  either way, it is already a bad practice to do dynamic inheritance, and most things would be resolved via `enum` types anyway, with simplicity , and more safety

  

``` 

//https://github.com/Mjz86/String 
using u256_t = mjz::uintN_t<version_v, 256>;

// may be buggy,needs proof
MJZ_CX_FN uintlen_t
caclulate_truncated_hash_byte_count(std::span<u256_t> sorted_mangles) noexcept {
  if (!sorted_mangles.size()) {
    return 0;
  }
  uintlen_t similar_bit_max{};
  u256_t prev{sorted_mangles[0]};
  sorted_mangles = sorted_mangles.subspan(1);
  for (u256_t current : sorted_mangles) {
    uintlen_t similar_bit_cnt = countl_zero(current ^ prev);
    prev = current;
    uintlen_t similar_bit_cnt_without_equal = similar_bit_cnt & 255;
    similar_bit_max = std::max(similar_bit_max, similar_bit_cnt_without_equal);
  }
  uintlen_t unique_bit_max = similar_bit_max + 1;
  uintlen_t unique_byte_max = (unique_bit_max + 7) >> 3;
  return unique_byte_max;
}
struct base_castation_table_t;
union cast_visibility_t {
  // sizeof(cast_visibility_t)*8 >= tatal_base_count_v
  std::array<uint64_t, (sizeof(u256_t) - sizeof(void *) * 2) / 8> inline_bits;
  // sizeof(cast_visibility_t)*8 < tatal_base_count_v, this is a bit pointer ,
  // pointing to the beggining of the cast visibility bit array, the bit shift
  // stored plus the pointer to u64 holding it.
  std::pair<const uint64_t *, uint8_t> outline_bits;
};
struct castation_table_ref_t {
  u256_t hash_current;
  // O(#tatal_base_count_v/8) , note that we store visibility in bits , because
  // the O(#tatal_base_count_v) per base factor must be negligible.
  //  if tatal_base_count_v is less than or equal to bit sizeof buffer use
  //  the inline one

  cast_visibility_t cast_visibility;

  // the constructor initilizes this either on dll load or on the static
  // initilization phase , note that dllimport vtable layouts are partially
  // known, but their content this is used to access the cast table
  const base_castation_table_t *castation_ptr;
  // reletive  to most derived, used in cast to void/byte/bit pointer or access
  // a virtual base class by subtraction of these offsets
  intptr_t object_offset_current;
};

struct base_castation_table_t {
  // this  can be used to access the most derived type info in the node ,note
  // that c colon std::type_info_t (given by typeid(expr)) is just the 256 bit
  // backend mangle , not even a pointer.
  const castation_table_ref_t *most_derived_table;
  // tatal_base_cnt_and_trk_cnt= (tatal_base_count_v << 5) |
  // (truncated_hash_byte_count - 1);
  uintptr_t tatal_base_cnt_and_trk_cnt;
};

template <size_t tatal_base_count_v, size_t truncated_hash_byte_count>
struct castation_table_t {
  
  // these are sorted in the order of the hashes.
  std::array<const castation_table_ref_t *, tatal_base_count_v> node_ptrs;
  // between 1 and 32 is truncated-count, as a 5 bit , and rest of the bits are
  // for number-of-type
  static_assert(size_t(truncated_hash_byte_count - 1) < 32);
    // the negative offsets are the first members. 
  base_castation_table_t base;
  // the 256 bit hashes aren't 256 values , instead we have a martrix pointer ,
  // truncated-count ( max of 32) arrays of  big endian bytes , the reason for
  // this is presented in the dynamic cast spec. note , if the hashes are unique
  // when truncated ( very often the case ) the least amount of bytes of hash is
  // used while still keeping every hash's value in the table is unique ,O(1+)
  // Amortized.
  std::array<std::array<uint8_t, tatal_base_count_v>, truncated_hash_byte_count>
      sorted_big_endian_hashes;
};
// may be buggy,needs proof
MJZ_NCX_FN const std::byte *mcc_dynamic_cast(const std::byte *This,
                                             const std::byte *vtable,
                                             u256_t dest_id) noexcept {
  castation_table_ref_t ref =
      *std::launder(reinterpret_cast<const castation_table_ref_t *>(
          vtable - sizeof(castation_table_ref_t)));
  This -= ref.object_offset_current;
  // typeid of 0 is reserved for void casts.
  if (!dest_id) {
    return This;
  }

  if (ref.hash_current == dest_id) {
    return ref.object_offset_current + This;
  }
  const uintptr_t tatal_base_cnt_and_trk_cnt =
      ref.castation_ptr->tatal_base_cnt_and_trk_cnt;
  const uintptr_t truncated_hash_byte_count = tatal_base_cnt_and_trk_cnt & 31;
  const uintptr_t tatal_base_count_v = tatal_base_cnt_and_trk_cnt >> 5;
  const uint8_t *const search_matrix_ptr =
      reinterpret_cast<const uint8_t *>(ref.castation_ptr + 1);

  const std::span<const castation_table_ref_t *const> search_tbls(
      reinterpret_cast<const castation_table_ref_t *const *>(
          ref.castation_ptr) -
          tatal_base_count_v,
      tatal_base_count_v);
  std::array<uint8_t, 32> dest_raw =
      std::bit_cast<std::array<uint8_t, 32>>(dest_id);
  if constexpr (std::endian::big != std::endian::native) {
    static_assert(std::endian::native == std::endian::little);
    std::ranges::reverse(dest_raw);
  }
  uintlen_t start_index = 0;
  uintlen_t count = tatal_base_count_v;
  for (uintlen_t i{}; i < truncated_hash_byte_count; i++) {
    const std::span<const uint8_t> search_space(
        search_matrix_ptr + i * tatal_base_count_v + start_index, count);
    const auto sub = std::ranges::equal_range(search_space, dest_raw[i]);
    start_index =
        uintlen_t(std::ranges::begin(sub) - std::ranges::begin(search_space));
    count = std::ranges::size(sub);
  }
  if (!count) {
    return nullptr;
  }

  mjz::uint_dyn_t<version_v, true> vis_map{};

  uintlen_t vis_map_offset = 0;
  if (sizeof(cast_visibility_t) < tatal_base_count_v) {
    vis_map_offset = ref.cast_visibility.outline_bits.second;
    vis_map.words = std::span(ref.cast_visibility.outline_bits.first,
                              (count + vis_map_offset + 63) / 64);
  } else {
    static_assert(sizeof(cast_visibility_t) == sizeof(ref.cast_visibility));
    vis_map.words = ref.cast_visibility.inline_bits;
  }

  uintlen_t count_invisble{};
  if (count == 1) {
    if(!vis_map.nth_bit(vis_map_offset + start_index)) return nullptr;
  } else {
    count_invisble = mjz::countr_zero(vis_map, vis_map_offset + start_index);
  }
  if (count <= count_invisble) {
    return nullptr;
  }
  
  auto desttbl=*search_tbls[start_index + count_invisble];
  if (desttbl. hash_current!=== dest_id)  return  nullptr;
   
  return desttbl.object_offset_current +
         This;
}
 
// note that this is technically a tree , however most of its functions,offset,
// and base vtable pointers storages are just spans into the tree  table array ,
// O(#direct-virtual-bases) Amortized and O(#direct-virtual-function) Amortized.
// note that all of this v-table structure is offset_dependant not_reorderable
// refexpr.
template <size_t tatal_base_count_v, size_t truncated_hash_byte_count,
          class conceptual_most_drived_node>
struct conceptual_root_data_t {
  conceptual_most_drived_node node;
  castation_table_t<tatal_base_count_v,truncated_hash_byte_count> castation_table;

  // this is a conditional part of the castation table accessed using the
  // outline_bits member only if the total base count is above the threashold,
  // note that crossing this threashold will emit a warning of for example :
  // "base count is more than 128 = (sizeof(inline_bits)*8),falling back to using
  // O(n^2)=(... <vtable size in byte> ..) space storage, are you sure
  // inheritance is the right way?  ".
  std::conditional_t<
      (sizeof(cast_visibility_t) * 8 < tatal_base_count_v),
      std::array<uint64_t, (tatal_base_count_v * tatal_base_count_v + 63) / 64>,
      void>
      cast_visibility_bit_arrays;
};


 

template <size_t num_funcs, class... conceptual_base_nodes>
struct conceptual_node_data_t {
// the reason is that the root is  at the beginning,  and the castation-table is after the end ,so , for a down cast , we can simply do a range check.
  castation_table_ref_t castation_table_ref;
  // the v-table pointer points at &virt_func_table[0].
  std::array<const void *, num_funcs> virt_func_table;
  std::tuple<conceptual_base_nodes...> bases;
}; 

 // down_cast:
 // we have the base vtable pointer , we know that if the cast was successful,  it would be that the drived table has the base table indide it at a known offset ,
 // so , we first see if the space between  the root and the castation-table  can allow such a thing ,
// if not , cast fails, if it can , we go at that offset , and get the castation_table_ref assuming its valid and see if the 256bit hash matches, 
// if the hash matches we are in the correct place, if we are , we're done and successful , we just use the offsets.
// so this is O(1) operation,  not even the search is necessary 
//  note that this is a algorithm dependens on the assumption of unique hashes,  and the failure would be reading a memory that is exactly what the hash is , but not being the castation-table hash, this is astronomically unlikely because its 256 bits  that are needed to match and its  a cryptographic hash , so even more unlikely,  practically impossible.
 






// dllhidden is deafult, however,  we can use :
// virtual dllhidden/dllexport/dllimport ; in the definition of class.



// the case of no_virtual_rtti:
//vtable is :
template <size_t num_funcs, class... conceptual_funcptrs>
struct conceptual_funcptr_data_t {
  intptr_t  object_offset_current;
  // the v-table pointer points at &virt_func_table[0].
  // note that nothing other than function pointers are stored... , dynamic cast  returns null on all cases if no rtti is enabled.
 std::array<const void *, num_funcs> virt_func_table;
  std::tuple<conceptual_funcptrs...> funcptrs_of_bases;
}; 


//  role of object_offset_current:
// the virtual pointer being a fat pointer to the current object  means that if we want to have the most derived pointer we need the offset
// so we make it so that this is only stored for at least being able to call a base function with its pointer,  or have a virtual base class.






 

```

 
 

  

  

  

  a virtual reference is :

virtual table pointer

and

type pointer







- dynamic cast( a cache-friendly binary search , instead of a graph traversal in itanum) :

  we first  do a binary search for the lowest and highest  most significant  bytes  with equal value to the most significant bytes from the cast destination.

  if the value is not found we return nullptr ( range is empty) , if only one value in range , we use that index for getting type v table , then if our hash is  not equal to hash-of-current-types-name , we return nullptr , else, then use the offset to get the pointer,  else:

  we do a binary search  on the next most significant bytes , similar to the first step.

  we do this loop until we either reach the hash , or we get nullptr or , if  all the truncated-count ( max 32)  bytes are searched,  we return nullptr.

  * the reason for doing this unusual storage of hash:

    when we search , each time we do so , we search for the byte , and in each section,  the bytes are next to each other , so 64 bytes (64 bases) can be easily searched through without a cashe miss , and in most cases,  we have few bases , so , the first steps , which are the most likely to find the hash in ( cryptographic hashes are uniformly distributed, so in 256 bases ( which is a lot) we are likely to have 1 to 5 bases, so the next step will likely resolve the search ).

    also ,either way we will need to access the offset anyway,  and because of the 32 byte alignment , we know that ( assuming 64 byte cache line) we only access one cache line to get both the offset , and the 256bit hash to check equality with to see if we have a match, also , the base-visibility-bit-flag at the index must be true for a successful  cast, because an inaccessibile base cast results in nullptr.

    this strategy helps mitigate the binary search cache miss penalty. 

    if the number of bases are less than 64 , we most likely have only 4 cache lookups ( 1 is the baseline for v table access , 1 for the first search step , and assume the first step is successful, another one for the  v table lockup  , and last one for check and pointer offset lookup, but the visibleity is inlined)
    
    in practice most structures have less than astronomical base counts , truncated-count helps to reduce all that 256 of precision,  untill we reach a  sorted set that will lose uniqueness as soon as we truncade more , gaining  the same speed with  equivalent functionality and practically fewer bytes in the table.

    truncated-count is the minimum value that it can be , while having each truncated-hash in the table unique.
   

    

---









calling convention:



the gist is ,

The caller can store values into registers that the callee has promised to not modify, 

To reduce stack spills,

to reduce spills , the function signature should also require minimal usage ( that's where overlap optimization comes in) and also ,

for reducing braching , most of the branches known in the caller to occur at call site have been moved to the return branch in the callee , the callees return acting like a switch statement to the caller code, but because the return is already a necessary dynamic branch , the cost of 2 branches ( return , and a check of return   value in caller) is reduced  to one.

also , because of the lifetime problem( understood in rust dyn calls with lifetime signatures that convert to other signatures) in dynamic calls , any call through a dynamic  function pointer or dll  has either provable valud lifetimes or is unsafe(lifetime-dyn-call) or  for casting between lifetimes using unsafe(lifetime-cast)



a mcc signature has:



return-type function-mangled-name ( arg-type-(in/out/`inout`) args... ) context-type (noexcept/throws) (noreturn/mayreturn) ( other-function-qualifiers);



- note:



the context type is as if its an `inout` argument.



the throw-value specified in the context type is as if its an out argument, this out argument  can overload with any other arguments except the context object, because this is the only out argument other than the context object that is preserved  in a unsuccessful call to a function, as a result  this often overlaps with the return argument, however a non trivially relocatable throw value will need to be passed via a non overlapping  throw-value-pointer input, for this reason it is discouraged that throw values be non trivially relocatable.



the return type is as if its an out argument.



there are special registers:



1. the stack pointer (callee saved) and the base pointer(caller saved) :



stack pointer is like itanum ,  the caller can assume its value is the same after the call.
 the base pointer is a register that isnt pinned ,usually some optimizations might use other stratch registers as base pointer as well .


2. the instruction pointer (caller passed, no save):



like itanium



3. the normal return address (caller passed, no save):



the return address to the happy path section in the caller.

 if specified `enum` return , its the   absolute address of the first table entry, and the offset of all other table pointers. 



4. the catching return address (caller passed, no save):



the return address to the unwind/sad path code section in the caller.

if specified `enum` return or catch, its the table pointer to the table of return entries corresponding  to each `enum` entry  path.





( only 5 pointer sized registers, 2 of which are already used in all architectures for that purpose)

( the other 3 can be pushed before general purpose use and poped/re assigned when needed at the call boundaries to reduce register pressure when register utilization is too much)


the conceptual call site( note that its implementation defined for each function signature if the jump instruction is used or a  call, however  it is encouraged to use the one with the best semantic):

// in this  one its a jump site caller, with no enum ret or catch .
// old BP is saved somewhere to be restored later.

 // the cost of  an exception ( or any more than one return path really)  is this mov of a constant to a  register that is pinned as important for multi return. 
 
`CRA= catchlable-lable;`// constant offset

 `NRA= IP+lable;`
 
 `jump func;`// IP=func
 
 lable:
 .....happyy...
 catchlable:
 .....sad....
 
 
 // or if a tail call:
 func:
 `BP=SP;` // preserve CRA, NRA,BP  throughout the call
 
 // stuff...
`SP=BP;`
 `jump func;`// IP=func
 
 
 
 
 
 
 
 // jump site callee 
 func :
 
 `BP=SP;` // preserve CRA, NRA,BP  throughout the call

 //  stuff ... thats happy....

 `SP=BP;`
 
 `jump NRA;`
 
 //  stuff ... thats  sad....
 
 `SP=BP;`
 
 `jump NRA+CRA;`
 
 
 
 
 // if using call and ret, the call site  , NRA being either a cpu known registers for return,  or a stack position, and CRA being a caller passed register.
 // however  the implementation must acknowledge that of the base pointer is moved how many bytes to ensure that a stack argument position is known at a fixed offset. 
 
 `CRA= catchlable-lable;`// constant offset
 
//  is implicitly assigned in the call , the call  instruction is discouraged if call does big shadow stores 
  
`call func;`// NRA=IP ,IP=func, after call

 lable:// exactly after  call
 .....happyy...
 catchlable:
 .....sad....
 
 
 // callee ret site:
 
 // preserve CRA, NRA,BP  throughout the call
 
 func :
 //  stuff ... thats happy....
 
 `ret;`
 
 //  stuff ... thats  sad....
 
 `NRA+=CRA;`// if the NRA is in the stack  for example some calling conventions the call instructions does stack push , this then becomes an override of the return pointer in the stack

 `ret;`
 
 
 
 
 
 
 
 







a function's manipulated registers come in 5 categories ( aside from special ones):

1.  in ( const in argument ) :



cannot  be written to by callee, only read from .  

 

2.  in-val ( mut in argument ) :



categorized among in, but with ability to write.

can be written to by callee, and read from but its undefined if the caller reads these after the call , except when initialized again.



3.   out:





cannot be read from by callee( before the initialization),

and must be written to at some point in callee.







4.  `inout`:



free to read or write,

once the callee throws , this value is considered dropped by the caller.





5. used ( scratch registers),(caller saved):



may be read from after initilization or written to  , but its undefined if the caller reads these after the call , except when initialized again.







important note: 



 in and out registers may overlap in the calling convention , this doesn't mean that they will be `inout` , only that the registers who are used for input purposes, will have output purposes after the call,  because its faster as an `inout` amd has less register pressure.



any registers not used or not in any signature is unused ,

any register not needed after a call , or a fastdyncallee dynamic call doesn't need to be saved ,unless proven better by compiler.

for example if i do a call to a fastdyncallee dynamic function in an almost empty function, no registers are saved , only the function will have many used registers.



a function signature , or a function pointer type will determine the :

in , out, and `inout` registers.





- in the rare occasion  of using all registers for parameter passing , the caller pushes arguments to the stack , the caller is responsible for the cleanup of the  stack parameters, 
( the parameter is just in the caller stack frame , but only the last part of the frame that sp is in one edge of)



- the registers allocator priority goes like( lower means more priority for being in a registers):

1.   all `inout` registers  ( including the ones made implicitly via the overloap of in and out registers).

2.  all out registers. 

3.  all in-val registers. 

4. all in registers. 



- after stable sorting of arguments based on in/out/`inout` the stack arguments are pushed onto the stack from right to left.



 - the responsibility of cleanup of stack variables:
 these arguments are cleaned up by the caller. 

 the reason is that the caller might read these `inout/out` ( while in and in value are unneeded, its faster if there was only one responsibility ) so they do  need cleanup by the caller.

 these arguments are allocated on the stack before assignments of the stack pointer  to the base pointer  in the callee.
 







* the unsafe(dyn-args) dynamic variading functions  :
 printf in C is one of the examples, although these functions are unsafe and therefore  bad practice to write.









the used registers set is :



for a fastdyncallee dynamic function call ( through a function pointer or a dll call) : 



all the registers not used in the signature ( except for the base pointer register and other special registers), so the callee's function pointer  is as if it was to its static call , but ,all the callers who use this function have the burden of saving intermediate values into the stack.



for a fastdyncaller dynamic function call  ( through a function pointer or a dll call) : 



no registers are in the used set , this means that the callee  saved all the callee used registers in the dynamic transformation code,  so the caller is free to assume that all  registers not in the function signature are preserved,  making the caller more performant and reducing caller code size.

for a static call:



all registers used in the callee and through the function call .



a used registers is a registers whose value is potentially modified, but the initial value is not restored in the function call to be used after the call.



for example, a push of r and a pop of r at the end in return means r is restored so r is not a used register, even though it might be modified.



a formal description might be:



a registers who's observed values before the call might be different after the call , in at least one code path.





- dynamic function contract tracking:

any dyncontract function who's address is captured for dynamic calls must have a pointer-sized function pointer before the function's code itself. 

this function pointer is the contract checked F pointer , it points to the contract handler,

the contact handler checkes the pre or post conditions of a function, based on an implicit  in-val argument  that contains:

0. check the pre vs check the post bit.

1. expected contact violation semantic of caller.



- more explanation:



this set grows linearly until the registers load is too high , then for these registers , the caller stores them to stack and pops back after return from calle, this makes sure there is minimal stack usage,



( because the register assigner is used after the main optimization passes and in the linker, any recursive graph can be known to store the registers in stack)

 

however because fastdyncallee dynamic/external calls don't have the luxury of known assembly, so ,

every register might be used , so , the intermediate registers need storing before the fastdyncallee dynamic call and re storing afterwards, just like how the call and ret instructions work via stack push and jumps, or how the c++ async resume and suspend is defined via jumps,

this is just more explicit, because we have no control over what call instruction saves but we do for ret.



there are also 2 return paths ,

instead of a branch after a call like most std::expected, we do an optimization, not valid in c, that isn't try catch with cold paths , but ,

the caller happy paths have no need for a branch because a throw will return to the catch path in the caller from the catch register address, this is also very fast , like a single return statement, and the only cost is that a register is occupied , not bad compared to throw , or even the if statement in my opinion



this is also possible because of the radical exception handling mechanism ( a statically known context type specified in the function signature, to be the vessel for the allocators,exception handling, debugging and stack trace information in debug builds , and maybe other information, this means that the exception type is statically known , unless abstracted away by a std::any-like object in the context type, also the debugging could be more information rich , for example skipping unnecessary name mangles or helper functions , because the debugger is just code with some trap brake points),

basically i don't need to tell about all of it , but every function has any catch statements or raii clean up codes in the catch path , this doesn't need any extra unwinder, because there is no data structure for the unwinder, it's just code , and the return is directly to the unwind code instead of calling many cxx throw functions and using thread local or dynamic storage



this also makes reverse engineering way more difficult, which is a good thing for those who want a program with no debug info to be uncrackable, although not a goal , this is a side effect of using registers in unconventional ways.



note that this ABI is fully abstractable under Itanium , basically, only the outer functions needs itanum for compatibility,

at most the catching return points to a cxx throw for compatibility.



note that , as far as i know, the call and ret instructions already store much unnecessary registers in the stack, so i dont think the dynamic overhead is much different from a normal dynamic call ,

also , i believe that allowing the return , arguments and more be able to expand , be even simd registers is far more beneficial than a restricted set of registers as function arguments and a single return registers, let alone the catch register



there might also be optimizations:




f:

init:...

code:...

if ... jump to happy

(throw code ...)

move catch ret register to normal ret .

( this will make the return at the end a throwing return)

jump to clean

happy:

....

clean:

....

end and ret:

....

ret to normal ret



instead of duplicated cleanup code in happy and sad paths in the c++ throw conversions, or returning to an unnecessary brach that is known to be happy or sad in the calle.



in x86 specifically ,some things to note is that , while x86 has like 1024 bytes of zmm registers, its all caller saved in itanum , so... the compiler often wont use their full potential even in static calls, it is already not used ,

and with the movement towards less virtual calls and more compike time polymorphism, the gains might be very beneficial.



also, even at worse cases of using all 1024 registers , at most we do 16x2 simd zmm load and stores , although, i still believe that with the amount of avoidance of the people from dynamic calls, the overall optimization will be worth it , even if it costs 32 simd instructions in the virtual call site.



an important consideration is comparing the dynamic call to the call instruction,

any register necess



ary after the call is either:

special( like sp,ip)

pushed in the call sub instructions,

or

pushed before the call instruction.



even tho the call instruction is faster than pushing them manually, the registers are still being spilled to the stack , so in every case , a dynamic call necessarily has to store the registers necessary in the stack .

all we did was , for static calls, reduce the burden of the runtime to the link time analysis.

* note on custom dynamic call register usage. (  more caller saved stuff , but without all of them) :
an  `uninitilized xvaluexpr   ` argument can be used as a dummy in val argument that isnt initialized in nither caller nor callee,
it syntactically makes a used register without actually  having overhead with the fastdyncaller transformation because its effective category is in val.
or an `uninitilized ivaluexpr   ` if the caller needs an extra callee saved register ( the in category)


--- 

 enum/pattern matching functions:

 a function returning an `enum` type , depending on the enum-type's properties can fall into two categories:

 

 - normal control flow: 

 the `enum` type doesn't have any match specifier 

 

 - `enum` control flow:

  the `enum` type returned dictates what caller return address entry the callee jumps to .

  this is especially useful because many throw-values are enums of :

  0. many violation exceptions.

  1. many cancelation tokens.

  2. many user code exceptions.



  but often the return pointer doesn't need indirections because often the return is not an enum.

  



--- 

program initialization:
when c colon initialization  is activated for the initial  binary components,  the loader loads the dynamic data structures for that components, 
then initilizes the root module of that component.

on de-initialization,  the dll de initilizer is called.
 note that this section of the specification may need to change a bit , but generally the goal is to make module initilization predictable based on the source code  and each module is initilized as if wrapped by a synchronized block.
 the    synchronization mechanism may change in next revisions,  but the goal is stated.


(de)initilization sequence of modules:
the module constructor that runs :
0. fetch add 1  to the initilization atomic  and if it was not 0 at the beginning, return.

1.   set the data segment memory accesses (ie. make the  read only sections read only)

2. all the dependant modules get initilized .

3. all the static variables get initialized in order of declaration.




program code: 

 4. the main function in the module will run if defined.
 
 
 the module destructor that runs :
 5. fetch sub 1 to the init atomic , if it didnt went to zero , return. 

 6.    destroy every static variable in the reverse order of declaration and deinitilize each module that we initilized.



 



*  in a dll initilization the loader has different work to do , so the first step is loading , and after the destruction theres unloading.

 
 * each import/export declaration of a module has an import/export symbol of the depenant  module identifier 256bit backend hashes, the identifier is  as if its a function  named with the module name within a `bool __mccabiv1::modules::<name of module and its module fragment>(bool)` that is similar  to the  the dll of the module `mayde_initilization_fn_offset` function, however it only (de)initializes the module itself , not all the modules in the dll,
 once all the modules in a dll are uninitialized,  then `mayde_initilization_fn_offset` can give permission to unload it.
 


--- 
read only dynamic  symbol table layout:


// constant read only global section.

// at offset  0 
// this is used in dllmap , not really useful otherwise.
`needed-linker-abi256-hash;`


`dll-identifier-abi256-hash;`

// this is the initilization function,  the bool it returns says the status of initilization ( true meaning  success), but on deinitilization,  if it returns true then that means that it was the last one to be uninitialized  (  to free the memory if its the last)
 
` bool  (*mayde_initilization_fn_offset)( bool init);`

`global_loader_ptr_offset;`

  `size_t  symbol_count;`

`size_t dll_count;`


// padding

// this is the sorted 256bit hash back-end mangle :

` uint256_t  symbol_fragment[symbol_count];`

// this is the corresponding symbol data to the symbol mangle.

` uintptr_t symbol_ptr_offset_and_mask[symbol_count]`

 ...
 // arch is `3-log2(alignof(void*))`, and note that if the offset  overflows the program is ill-formed ,however on 32-bit or 16 bit systems this is not really a concern.

 //the `symbol_ptr_offset_and_mask`'s value is defined  with `(uintptr_t(offset)<<(arch)) | viability`,

 // the viability mask only can use 3 bits :
 
// 1 : `dll_comparable_address` vs `no_dll_comparable_address`

 // 2 : interpositioned vs no-interposition

 // 3 : dllimport vs dllexport
 
 //if  no-interposition and dllexport and `no_dll_comparable_address` pointer is the offset of the function adress reletive  to offset 0  

 //  if its interpositioned and dllexport and `no_dll_comparable_address`  , pointer is the offset of the reserved static atomic storage for the address of the function reletive  to offset 0
 
//if  no-interposition and dllexport and `dll_comparable_address` pointer is the offset of the function pointer reletive  to offset 0  , this function pointer has the function adress reletive  to offset 0   
 
//  if its interpositioned and dllexport and `dll_comparable_address`  ,   pointer is the offset of the reserved static atomic storage pointer for the address of the function reletive  to offset 0 , this storage pointer has the reserved static atomic storage for the address of the function reletive  to offset 0

// if no-interposition and dllimport    pointer is the offset of the function pointer reletive  to offset 0  
 
//  if its interpositioned and dllimport  , pointer is the offset of the reserved static atomic storage pointer for the address of the function reletive  to offset 0
 
 
 
 // write only once in initilization, then make readonly section: 

`global_loader_t*const global_loader;`


`uint32_t dll_priority;`

// the pointers to the interpositioned function pointers.

`const void**const  imported_interposition_fnptr_ptrs[....];`

 
// the pointer to the imported function.

`const void* const imported_non_interposition_fnptrs[....];`

// the pointer to the imported function, first having the offset to the current pointer .

`const void**const  exported_interposition_dll_comp_address_fnptrs[....];`

...
  
  // read-write section, note that based on compiler flags each of these can be aligned to the destructive interfaces size.

// initilized  with the function definition 

`void* exported_interposition_fnptrs[....];`
 

...








// if the binary definition has the mcc dynamic  loader.

// read write section global loader :

`mutex...;` or `atomic flag... ;`

`size_t total_symbol_count;`

`size_t dll_count;`

`uint256_t  sorted_symbols(*)[total_symbol_count];`

// `symbol_ptr` is just like `symbol_ptr_offset_and_mask`, but it has absolute addresses calculated from those offsets.

`uintptr_t   symbol_ptr(*)[total_symbol_count];`

// the 3 low bits are  for the viability mask , the high bits are more than enough  to indicate dll info index.

`uintptr_t dll_info_index_and_mask(*)[total_symbol_count];`
 
// sorted infos based on dll info's dll-identifier-abi256-hash 

`dll_info dll_infoes(*)[dll_count];`

`uint32_t dll_prioritis(*)[dll_count];`


// note that for security a sanity check for sorted ness van b done in `O(total_symbol_count)` time complexity 


// this is done before the module initilization, `O(total_symbol_count)` time complexity 

- dynamic load :
0.make  lockgard mutex.
1. inspect the list of given binaries meta datas and their priorities and make sure needed-linker-abi256-hash is compatible with the current version.
2. allocate at least the size of all given symbols and ptr-masks.
3. merge all sorted arrays of symbols in all the given binaries  into the region while also turning offsets into absolute values and assigning priorities( also sort based on the  priorities as if  the least significant sorting indicator, and because each binary is sorted and its indicator is constant, we can just do `O((total_symbol_count)*log(#binaries))` merge with this via a stable sort merge step or if the `log(#binaries)` in merge dominate(which is very rare because the number of dll files would b exponential)  we do a   `O(total_symbol_count)` radix sort ).( either via  merge step or radix sort based on the appropriate heuristics) and while doing so , for all duplicate regions ( next to each other symbols of equal value)  do the duplication region check function and fail if it fails.
4. if checks were not successful , then fail.
5. assign all given libraries loader pointer to the address of global loader, and set `dll_priority` es appropriately. 
6. do a scan to find  duplicate regions , then for each do the duplicate resolution function.
7. freeze  the "write only once in initilization" section to readonly 
8. return successfully ( and unlock)


- duplication region check:
0. get the M symbol masks
1. count all the symbols that are dllimport, are interpositioned are `dll_comparable_address`. O(M)
2. check that  all priorities between ajason external symbols is unique or not. O(M), the symbols must have unique dllexport priorities , if not we fail.
3.   if the loader is not doing  a start up load , all `dll_comparable_address` counts must be from those qualified as dllimport.
4.   if the loader has  imports but not any export,  we fail.
5 . return successfully 


// if the start  up loader failed in a duplication region check, the program is ill-formed  .
// however if the hot loading loader failed , its a contract violation. 


- duplicate resolution : 
0. find the highest priority export adress .
1. assign all the other adresss to this address.
3 . return successfully 



// this is done after the module deinitilization `O(total_symbol_count)` time complexity 
- dynamic unload ( unchecked)
0.make  lockgard mutex.
1. put each dll info needing unload into an unload state ( via a flag) 
2.  with `std::remove_if` style algorithm , remove any symbol that has a dll info needing unload at its dll info index.
3. update `total_symbol_count` and return successfully ( and unlock)


// safer but `O(total_symbol_count*log(#binaries))` time complexity 
- dynamic unload ( with use after free check, but a bit slower) :
0.make  lockgard mutex.
1. get the beginning and end pointers for each binary that needs unloading,  and sort theses  via radix sort to allowe for binary search  for range check. 
2. for all elements in `symbol_ptr`:
3. if element and its internal function and storage being pointed to  is not in range of any of the binary pointers , we continue  to next.
4. if the element itself was not in binary unload range we fail  by termination ( try to unload symbol while its used ,obvious and easy to check use after free).
5. remove this element  from the array ( `std:: remove_if` style removal over the entire loop) 
6. continue till all are done.
8. update `total_symbol_count` and return successfully ( and unlock)



//  PLT style lazy loading is highly discouraged and frankly very hard to implement under mcc , if we lazy load we will get O(n²) because a trigger of the loader is O(n) and we do it for every symbol on load,  so batch loading makes it better if we stay in the mcc loader, however a `represent_cxx` loader is not in scope of thisABI .


``

// high level overview of what radix sort means in this context( not the definitive way , just a way ):

// first , we know that the merge sort algorithm heuristics told us that `O(log(#binary)*n)` was too big ,so  we have to make an algorithm that works for very large n , 
// it probably has a million or more symbols.

// we allocate  a buffer with 2 ^ 16   stacks
// we sort the hashes based on their 32 most significant bits, by:
// 0. put elem in stack with `index =  uint16_t( priority)`. pop the stacks according to the order of stable sort. put elem in stack with `index =  uint16_t( priority >> 16)`. pop the stacks according to the order of stable sort.
// 1.  put elem in stack with `index =  uint16_t(elem>> ( 256-   64))`. pop the stacks according to the order of stable sort.
// 2.  put elem in stack with `index =  uint16_t(elem>> ( 256-    48))`.pop the stacks according to the order of stable sort.
// 3.  put elem in stack with `index =  uint16_t(elem>> ( 256-    32))`. same
// 4.  put elem in stack with `index =  uint16_t(elem>> ( 256-  16))`. same
// now , based on the knowledge that these are cryptographic hashes with uniform distribution, also considering that they even were sorted partitions at the beginning, 
// we assume that we have  partitioned the memory on this distribution  and so on average  we say that each slot has  `(N/ 2 ^  64)` elements that are not sorted,   this means that for practical purposes,  we can say that most parts are sorted.
// we do a scan of the regions , and for each part where it didn't have sorted order ,
// we do  a similar algorithm for sorting on the sub regions or probably a merge sort , or just do a bubble sort if it was tiny , we say its `O(m*logm)` if we had an unluky hit
// on average its `O(6*O(1+)*n) + f(n) )`
// `O(1+)`= push back in stack O(1) Amortized. 
//  so , we need to calculate why f(n) is O( n) on average.
// `f(n) =  O(n)+  (sum i=0 to 2 ^ 64  ,selecting for ies that existof  O(mlogm)* P( hash conflict that m  symbols with non equal least significant portions  has the same upper hash of I in a uniform distribution))`
// we can say that  the probability is very small and negligible that its close to 0.
// this  btw is the reason that hash tables are considered fast.
// lets say it more rigorously:
// uniform distribution means on avrage `(N/ 2 ^  64)` symbols have the same hash per each part ,
//  the probability of sameness of the upper 64 bits  for m symbols while having non equal  lower parts is :
// `( 2 ^  -64)^ (m)  =2^(-64m)`
// for selecting in the summation , we observe that for  avr partion size of M , in N elements,  we have around N/M partions.

// we multiply the time complexity and the selection sumation:
//  `O(n)+ O(n/avr(m))*O(mlogm)*(2^-64m)`
// we say that  m is represented by its average.
// `O(n)+ O(n/m)*O(mlogm)*(2^(-64m)) =`
// `O(n)+ O(n/m*mlogm*2^(-64m)) =`
// `O(n)+  O(n*logm*2^(-64m)) =`
// `O(n)+ O(n*2^O(-64m+loglogm)) =`
//  note that the biggest factor of the exponential  is m , not loglogm , and its negative, so , we can just transform it into O(1)
//  `O(n) + O(n*O(1+))= O(n+)` , amortized linear time.
// so , it is `7n+O(n+)` amortized 
// note that a non probabilistic algorithms is also doable ,
// to preform a sort using pure radix sort ,
// it does 16 repetitions of the  put elem in stack  and pop from it steps.
// so its  `16n+O(n+)` granteed,  however,  `log(#binaries)` is probably not going to be more than 16 ,
 // who has 65536 dll files!?... and who expects these many files to load fast?!, even the operating system  has B trees of its files that give `O(n*logn)` time when accessing n files,  thats not really a CPU bound task   , it's  more  file storage speed bound.
// i believe that practically  merge sort may be the only algorithm that is necessary to achieve max speed because of the nature of this operation.
// however,  dllmap files are still the most significant contribution in binary  loading. 









 there's also a secondary tactic used to make dll loads faster after the first memorization in the program installation,  its called a dllmap, and the linker recognizes dllmaps by using the dll count of non 1.


// constant read only global section.
// at offset  0 

`needed-linker-abi256-hash;`

`dll-identifier-abi256-hash;`

`void (*mayde_initilization_fn_offset)( bool init);`

 `global_loader_ptr_offset;`
 
`size_t  symbol_count;`

`size_t dll_count;`

// padding
// this is the sorted 256bit hash back-end mangle :

 `uint256_t  symbol_fragment[symbol_count];`
 
// this is the corresponding symbol data to the symbol mangle.


`uintptr_t offset_and_mask[symbol_count];`

`uintptr_t dll_info_index[symbol_count];`


// sorted infos based on dll info's dll-identifier-abi256-hash , this is because the loader may rearrange the given dll map paths if they are path independent( if  path dependent,  meaning that the dllmap has the reletive path of the dll it needs to load compared to its own path , it doesn't even need the dll arguments at all, in most cases a path dependent dll loader is enough for application installation.), also a mix of these can be used if the dynamic loader implementation supports it.
`dll_info dll_infoes[dll_count];`
`uint32_t dll_prioritis[dll_count];`


 
 
 // the algorithm for this is kinda  just an extension to the base dll loader.
 // first , all dll symbols with  known priorities and known dlls will get merged into this big sorted table for the creation  of dllmap, in an algorithm very similar to the dll loader ( dllmap loader is a variant of it).
 // because the execution of  this dllmap creator algorithm is only done at the program installer, the cost of the symbol merge algorithms are payed omly once.
 // a dllmap leader is given both the dll it mapped and the current program symbol table , and simply merges them similar  to a dll only loader , however because its a dllmap,  at the end of the merge operation it uses the `dll_info` data to identify the target dll ( for example this info can be an id , a path , a index , even file path independant dll-identifier-abi256-hash can be used, ect, based on the architecture and implementation), and uses the offset and priority , to set the dll data it needs to load , note that the dll table itself is not really used to lookup this operation, but the dllmap info is used instead,  each offset represents a pointer to the dll , and after the load( using `dll_info`) , its absolute address is calculated and is initialized as if it actually did the dll load.
// note that dllmaps can also merge into bigger dllmaps.
// the reason for the existence of dllmap is to not have the `log(#binaries)` factor grow , because a dllmap is a single binary that represents many binaries.
// also , most needed dlls in the real world are known at the program installation phase,  and the ones that do not , they don't need dllmaps , but the ones that do can still benefit from more memorization.
// we can think of a dllmap as a snapshot/save  of the dynamic state of the dll loader , but with non absolute addresses and reletive offseting, and using it is like restoring a snapshot, instead of building it up.

// the c colon dynamic loader is consistent of the dll (un)loader , dll map (un)loader and the dll map linker( to make dllmaps).










 
 





 









---



namespace and header



this ABI specifies a number of type and function apis supplemental to those required by the c colon standard. a header file named mccabi.h will be provided by implementations that declares these apis. the reference header file included with this ABI definition shall be the authoritative definition of the apis.



these apis will be placed in a namespace `__mccabiv1`. the header file will also declare a namespace alias ABI for  `__mccabiv1`. it is expected that users will use the alias, and the remainder of the ABI specification will use it as well.



in general, API objects defined as part of this ABI are assumed to be extern "c:". however, some (many?) are specified to be extern "c" or extern "c++" ( and `represent_cxx`) if they:



- are expected to be called by users from c/c++ code, e.g. `longjmp_unwind`/throw/...; or

- are expected to be called only implicitly by compiled code, and are likely to be implemented in c/c++.





---

 module dependency resolutions:  

  each file , has many kind of cached compiler outputs relevant to it in the build directory.





  0. import modules :

    lists all the modules imported from this file.



  1. export modules:

    lists all the modules exported from this file.

  

  2. import identifiers:

  lists all the identifiers and a reference to their relevant AST/IR code exported from this file.

  

  3. export identifiers:

  lists all the identifiers and a reference to their relevant AST/IR code  imported from this file.

  

 4. identifier contents(AST & JIT-IR &  IR & binary) :

 lists all the identifiers and their relevant AST/IR/asm code,

 also , each of the identifiers lists who their dependancies. 

 this includes but is not limited to : call graph, reference graph,  ABI graph, static variables used, register pressure, optimization qualifier and  etc.

 

  5. import library:

  lists all the dependancies from the web C colon repository. 

  6. export library: 

  lists all the dependancies that will be build for  the web C colon repository. 

  7. import and export  ajason:
  an ajason module is a module that is defined in a different translation unit,
  however we have a dependancy on this unit's executable,  but we dont import its content, 
  note that we can define ajason units in a module just by making the    definition , and saying exactly what needs to export , and what part needs no symbol imported.
  
  note that scopes can either only depend on the dependancies or the wholw translation unit.
  
  



 

 

  - type and function identifier categories:

    

    0. non templated:

    these are straightforward to make a graph of.



    

    1. full template specilization:

    these are similar to that of a non template in the graph contributions. 

    

    2. partial template specializations:

    these are similar to that of an overload set.

    

    3.  template overload set ( set of templates and non templates with this name):

     because templates in c++  and similarly C colon are turning complete  , we cannot determine if the template graphes will end their cycles or not ,

     so , for the resolution of this , we first need to resolve all the dependancies by execution of the `constexpr` IR-JIT code, the IR code will build the graph during its execution, 

     if the step count doesn't exceed,  then we have the  the graph in all templates being specialized,  and so being usable.

     while non specialized  templates are hard to parrarelize ,  the non dependent parts of the global graph can still run their template resolutions in parallel. 

 

  dependency graph building:

   for each part of the system,  we have a node in a dependency graph ,

   we start from parts where theres  no dependency and propagate to the nodes where the dependancies are resolved, 

   each time a template's full non template/specialized dependancies are resolved,  it  tries to resolve the rest via `constexpr` execution,

   this is repeated till we either reach max evaluation step or we resolve all identifiers to move to the next step.



---



compatibility



- extern "c:" :

 the function has c colon ABI , no restrictions.

- extern "c++"  (` represent_cxx`):

 the function signature, should have types that are not reorderable , templates that are valid in c++ , and structures consistent of only fundemental cxx types , this will be more exactly specified in next revisions,

 these functions have large thunks, although,  most of it is deterministic,  and the cxx throw and catch is already bloated anyway.

requirements  afterwards include having the OS loader step, lib unwind if  cxx exception is enabled and itanium symbol mangling.

 - extern "c"   (` represent_cxx`):

 very bare bones , only fundemental types, trivial structures and pointers .

 these functions have large thunks, similar to the fastdyncaller transformation, because of unknown usage set,

 however c style function pointers ( similar to cxx ones) are mandatory fastdyncallee,  because  obviously we cannot assume anything about the assembly. 
requirements  afterwards include having the OS loader step,  and  OS symbol mangling.

 
- syscalls :
 theres an unsafe low level syscall trunk to allow talking to the OS  
 

-  trunks for dynamic calls with specific calling conventions:
 a non exhaustive list of common calling conventions that would  need this thunk ( used in the target qualifier)

  cdecl
  stdcall
  fastcall
  thiscall
  vectorcall
  regcall
  `ms_abi`
  `sysv_abi`
  pascal
  regparm
  sseregs
  `force_align_arg_pointer`
  aapcs
  aapcs-vfp
  atpcs
  pcs
  `aarch64_vector_pcs`
  `aarch64_sve_pcs`
  swiftcall
  swiftasynccall
  ghc
  cold
  naked
  `preserve_most`
  `preserve_all`
  `preserve_none`
  interrupt
  signal
  mips16
  nomips16
  micromips
  nocompression
  altivec
  `spe_abi`
  eabi
  `m68k_rttd`
  `amdgpu_kernel`
  `amdgpu_cs`
   `amdgpu_gs`
   `amdgpu_ps`
 `amdgpu_vs`
 `amdgpu_hs`
 `amdgpu_ls` 
 `amdgpu_es`


` ret= std::abi_compat_call<convention-type>(fnptr, args...);`
 
 
 for static calls:
 
 ` ret= std::abi_compat<convention-type>::fn( args...);`
 
 note that this compatibility namespace uses reflection to generate the thrunks , maybe even insert asm directives using reflection,  only the declaration needs to be declared inside this namespace. 
 
 - linking:
 note that the mcc linker is independent of the os linker , so it can link non mcc code using custom dll plugins at start up.

- others:

 next revisions.

 

 

 

  * note : for each calling convention ( fast call, vector call, cdecl ,....) we have a unique caller register saver trunk and a unique callee register changer trunk and  exception handling trunks.

 





---





context object:

 the context type satisfying the standard context concept  is a central hub for all things that were previously in thread-local or static storage, 
 the context-type is implicitly initialized by the callees context type via,
 in the debugging environment  every action , would probably have to go through the context-type, but this really helps make everything that is implicit controlled and optimized.
  almost all of these are inlined , especially the ones handeled in the callee. 
  if the type of context-type  of an inlined callee function maches the caller,  and the context satisfies the elidable context concept,  the compiler is allowed to treat the callee context as if it was the caller context, and not call the context making or checking operators,
  debugging contexts however cannot be inlined.
 
 * note: 
 if the context types match and  tail call optimization can be performed ,  and the context satisfies the elidable context concept,
 the context operators can be omitted and the context-type is trivially relocated to the callee.
 
 - operator context( out callee-context-type )caller-context-type :
 constructs the context type of the callee before the call,
 in a debugging environment,  this can have conditional trap instructions. 
 
 - operator ~context( callee-context-type ) caller-context-type :
destructs the context type of the callee after the call,  potentially handing rich unwind information.
in a debugging environment,  this can have conditional trap instructions. 
 
 * note: caller and callee are both protective in the caller and callee respectively.
 
 * note: these operators themselves do not call to context operators , they are special member functions of the context-type,  and writing the definition of them requires unsafe(no-outer-context), similar to the `__mccabiv1::main` function , this is the actual entry point , similar  to the c maim function, can be overriden, however in the standard module main , this isdefined to call the global main function and provide a standard context type.
 
 - operator  caller (   argument-stack-size( optional),stack-pointer( optional), instruction-pointer( optional), callee-pointer or backend-hash  (depending on  dynamic call vs static call) ( optional) )callee-context-type :
in the caller , before the call , the context-type gets a chance to caputure the protection information of the function if it wants to, to protect against stack overflow, and a minimalistic debug info for the stack trace.
note that caputure of backend-hash makes compilation slower because of non elided hash  
in a debugging environment,  this can have conditional trap instructions. 

- operator  callee  (  stack-size( optional),stack-pointer( optional), instruction-pointer ( optional),backend-hash ( optional) )callee-context-type :
in the callee , before the callee code and stack get initialized, the context-type gets a chance to caputure the protection information of the function if it wants to, to protect against stack overflow, and a minimalistic debug info for the stack trace.
note that caputure of backend-hash makes compilation slower because of non elided hash  
in a debugging environment,  this can have conditional trap instructions. 



-operator ~throw(inout throw-value)context-type noexcept  noreturn(...):
the last operation before the function finally returns the control flow to the caller catch path.
this is a noexcept  operator
if the function signature contains a noexcept, this  operator must  have noreturn  qualification (do some things or  make a stack trace  then call terminate  because unwinding is not possible).
otherwise,  its optional.
in a debugging environment,  this can have conditional trap instructions. 




-operator return(args...)context-type->return-value :
this operation happens on the return operator ,or implicit on void returns on function  scope end.
this can control the return object before the destruction of the callee  scope ,its final result is given to `operator ~return` after the cleanup executes.

-operator ~return(inout return-value)context-type :
the last operation before the function finally returns the control flow to the caller happy path.
in a debugging environment,  this can have conditional trap instructions. 



- `operator new( in(out) size,in(out) alignment, out uninitialized/intermediate byte*)context-type`:
 allocation  of a memory region with given size , if size is inout , it can be specified to minimum of the given size , 
 the alignment of the output region must be correct.
 intermediate is used to let the operator new have implementation defined canaries if in debug mode that arent optimized out because of the value being not strictly readable 
 in a debugging environment,  this can have conditional trap instructions. 


 - `operator delete( in size,in alignment,  in uninitialized/intermediate byte*)context-type`:
 deallocate the memory with the size and alignment and arguments matching the one that was outputed (or given, if specified in only) from (re)new,
 the lifetime of the storage will end .
 also note that the storage must be uninitialized  because the destruction of bytes must be trivial in all code paths( also enables invariant optimizations).
 however if the new operator is intermediate ( insertion of canaries) the delete operator   can also be also intermediate ( check the canaries) , but if we do this we lose some invariant optimized code.
this operator must be noexcept.
 in a debugging environment,  this can have conditional trap instructions. 

- `operator  renew(in size, in alignment, inout intermediate byte*,in(out) after_size)context-type  `:
 a combination of new and delete operator, similar to the semantic in cxx std realloc , 
 this can be used for trivially relocatable types or storage that needs to expand or shrink. 
 note that a failure  of expanding or shrinking the memory does not invalidate the previous region of memory, a success will end the lifetime of previous region and begins the new one's lifetime.
in a debugging environment,  this can have conditional trap instructions. 

-`operator debug(instruction-pointer/expression-index)context-type`:
 in a debugging context,  each expression that might involve a break point  will call this operator , in  optimized context this is just not defined.
 this operator can have conditional trap instructions.
 an implementation defined debugging thread might change a table ( atomic store )  ( the table's atomic pointer pointer can be caputured in the meta operator ) to indicate ( atomic load)   that this debug logic should trap or not.
 

- contract-in-val  type is defined to be used in  the `dyncontract`  dynamic  contract code , or the static non inlined contract checked code,
if the contract is not executed then this argument and all post and pre logic  is allowed  to be elided.
also , this can be used to check for specific types of contracts using contact categories. 


- `operator  pass_contract()caller-context-type ->contract-in-val `:
if the contract on callee is checked,  or dyncontract checking is selected , the implicit  value is created to provide the checking semantics for callee.

- `operator (explicit/implicit)_contract(contract-in-val, instruction-pointer( optional))context-type ->bool-convertible-type:`
 the return value indicates if the contract  condition should execute.
this function is executed before the contract.
in a debugging environment,  this can have conditional trap instructions. 


- `operator ~(explicit/implicit)_contract(contract-in-val)context-type `:
this function is executed after the contract  condition, even if the contract throws an exception.
in a debugging environment,  this can have conditional trap instructions. 



- operator  pre(contract-in-val, contract-stack-size( optional),stack-pointer( optional), instruction-pointer( optional))callee-context-type ->bool-convertible-type:
 the return value indicates if the contract pre condition should execute.
this function is executed before the contract.
in a debugging environment,  this can have conditional trap instructions. 

- operator  ~pre(contract-in-val  ,...)callee-context-type :
this function is executed after the contract pre condition, even if the contract throws an exception.
in a debugging environment,  this can have conditional trap instructions. 

- operator  post(contract-in-val , contract-stack-size( optional),stack-pointer( optional), instruction-pointer( optional) )callee-context-type  -> bool-convertible-type:
the return value indicates if the contract post condition should execute.
this function is executed before the contract.
in a debugging environment,  this can have conditional trap instructions. 

- operator  ~post(contract-in-val  )callee-context-type :
this function is executed after the contract post condition, even if the contract throws an exception.
 in a debugging environment,  this can have conditional trap instructions. 
 

* note that the pre and post operators will always be executed after the caller operator, and before the callee operator if the function is not inlined , and the contract is not elided.

- `operator make_meta ( inout std::meta )->meta-input `, `operator make_meta ( inout std::debug_meta )->meta-input`:
a `constexpr` function that makes the meta type based on static reflection information, 
it also can be used to insert canaries and other safety features in debugging,  the output is given to runtime to be used .

- operator  meta ( meta-input )callee-context-type :
 just right after the callee operator this will execute if defined ( if the debug meta is captured,  the context becomes a debugging context , with limited optimizations( almost having very value an exact address offset to the stack pointer or heap allocation begin),  but with immense debugging knowledge),  giving rich debug info to the context type.
in a debugging environment,  this can have conditional trap instructions. 


there is an operator to declare a new context for a code block,
also one to get a reference to a context-type. 

- operator set(constructor-args... )main-context-type->block-context-type:
if set is used in the context,  it creates a code block output the main context , this can be an opt out of debugging for example. 

`set_context<block-context-type>( constructor-args...){`
.....
...`get_context(...);`
`}`



- operator ~set(block-context-type) main-context-type:
executed after the block ends to destroy the block context ,even on throw paths


- operator get(...)context-type -> implementation-defined:
to get the context type with an expression.


- operator continue/break/~break/for/~for(...)lambda-context-type ...:
used in an implicit for each loops lambda ,depending on the iteration primitive,  these control the loops control flow.
note that in a for each loop using goto to an outer loop is ill-formed,  for that , one needs to use the for loop with explicit iterator or indexies and also unsafe(goto)/unsafe(dyn-goto).
but , we can label the for loops using `for break(label)` , and by using `break label;` in an inner loop , the operator ~break will compare the equality of those lables ( the lables are given at runtime via adding offsets of the instruction pointer of the loop end) and if not equal,  perform a break , but if equal,  perform a last break of the loop and stop unwinding.

-`operator co_yeild/ co_await/ co_return /co_break / co_continue(...)context-type ...`:
used similarly to operator break and continue,  however they also return awaitables , to allow for suspension and resumption of the coroutine.
note that while-loops and c-style for loops  do not have an implicit lambda , and cannot become a coroutine by using the co await operator.


- throw-value:

the value that is returned via the catching return address. 

 propagated through the operator catch .

 this is highly encouraged to be trivially relocatable.







- operator throw(self,...) noexcept -> throw-value:

this operator is used when the contexts scope uses the throw operator.
its mandatory that this function is noexcept beacuse, only the void context object can be in the throw signature, although, the throw has access to its context object by a function pram.
this function generates the throw object that is propagated to thr callers catch.
in a debugging environment,  this can have conditional trap instructions. 


- operator catch(self,callee-throw-value,...) context-type:

this operator is used when the contexts scope has an expression resulting in a call that might unwind by exception.
it is usual for this operation to throw an exception to the outer context object , note that , if a destructor throws in an unwind , the context-type may have been already filled with exception information , it is rare that this happens but often implementors terminate in such cases.
uaually specified noreturn, if not the user might struggle writing code.
in a debugging environment,  this can have conditional trap instructions. 


- operator try (out catched-value-in-catch-scope)context-type:
 often if the value is not convertable to be catched in the scope , it will throw to unwind to the rest of the code/ catch blocks.
the two standard context types are :
default support is for common exception string and `enum` categories , and   the common cancelation  and violation tokens,
however on lower levels , stack traces would get richer , and the exceptions would be paired with origin and specific lines of unwind it went through,
what objects did the violation and more 
in a debugging environment,  this can have conditional trap instructions. 

-`std::context_t<optimization-level>` :
this context type is used in synchronous programming,  specifically designed for an optimization level,
the lower the level the more debugging friendly is it.

- `std::async_context_t<optimization-level>` :
this context type is used in asynchronous programming ( paired with the std::scheduler<...>) ,  specifically designed for an optimization level,
the lower the level the more debugging friendly is it.


- `std::debugging_trap_handler( std::trapped_debbug_info_t info)fastdyncaller dll_comparable_address interpositioned dllexport noexcept=0; `: 
 captures standardized  debugging info on the trap instruction ,
 then hands control flow to the debugger, this symbol is `unsafe(interpositioned)`   and .
 the debugger  may use `set_interposition(std::debugging_trap_handler,address)`  in a concurrent debugging thread , to overridde this symbol,
 the debugger may pause the execution of only this or all thread, to inspect the debug context,
 or if the debugging has specific conditions for triggering , overring this with the checker vs trigger  for those conditions is an excellent choice to not slow down all execution.
 

 * note :
   we do not say that  programming bugs should be throw, in fact , it might br better to have quick enforce semantics ,
   but , the context type is the decider for this decision,  i would say that on non critical system code , it would be better to say that  no contract violation is  an exception.
   but in an airplane or any critical system,  i would definitely say that  even stack overflows is a recoverable error, because human lives depend on it.
   based on our definition of an recoverable error,  recoverable errors are exceptions.
   and if the function cannot have a recoverable error, its a noexcept candidate.
   but a  consideration should be that destruction or catch blocks should be made very carefully in a critical system because two exceptions can probably arise together but shouldn't lead to termination.
   for this reason,  it should  be the default ( change  via compiler flag in   a critical system) that almost all contract violations should not throw anything but terminate.
   because of that most code in the standard library can be noexcept even if allocation occurs.
   " ~ 95% of functions are noexcept if...." -  Herb Sutter ,De-fragmenting C++.






---

 contracts :

 

- a contract is an `weak_idempotent` expression that is used as a way to show validity of the current environment state.



-  there are different types of contract executions:



1. dynamic execution:

  

   a nodyncontract has no contact checked version. 

 

  a  dyncontract   function itself  has a  mandatory requirement  for a contract check in the `contact_checked` section of its assembly 



   the dynamic  callee may already have contract checks as a requirement, so , the two versions might be identical (  the function  pointer being the same or at most using a signle jump to the signature)if the callee is checked. 

   

   the caller uses either the checked version or the optimized version based on its contact requirement environment.

   

   

2. static execution:



usually the contract calling code is only repeated once in  the call site if any of the callee or caller contact requirement environments mandates it.



- diffrent types of `contract_assert`:
 

1. pre :


 in a pre expression of function
 note that this contract code is under the callee context-type 

2. post:

 in a post expression of function
 note that this contract code is under the callee context-type 

3. explicit:

 in a `contract_assert` expression of function

4. implicit:

in a  operation that might result in assertion like  a/0,a<<-1,a+maxint
note that this contract code is under the callee context-type  if its in a post or pre condition.
if the contract is implicit and not assumed or enforced, the value given after is an erroneous value, it is explicitly not correct,  however its not UB to read.







- what diffrent types of contract violation semantics do , when violation happens:

1. enforce :

results in calling the `operator contract_assert`.

2. quick enforce :

results in calling the terminate function.

3. ignore ( unsafe(contract-ignore)):

does nothing,  the check is elided at runtime however implicit contracts might not be elided perfectly because of erroneous behavior.

4. observe :

results in calling the operator `contract_assert`.

5. assume( unsafe(contract-UB) ):

results in undefined behaviour on violation, but the benefit  is that the check is elided at runtime and leads to compiler optimizations in compile time.





`operator contract_assert(...)  context-type` :

- gets called when a contract violation occurs 





- uaually specified noreturn , if not theres a chance that it ignores the violation.



* contract speed in this language:
theres a very clever way to both do range check  and have simd instructions,
first we  assume that the iteration-primitive way is not used ( because that is just the better faster way ) and we need a range check because we use indexes.
 
in  express colon  code if operator ~throw  is reached the array  is dropped ( uninitialized specified ) ,
so it can have any value,   inside the loop we set the values , but we didn't really   load them, and  then they got uninitialized, 
so under the as if rule we can just elide  the store operation in the violation path,
so if we do this wr either have : 
0. no violation all writes were preformed:
simd used here
1. one of   the checks failed :
 uninitialized read is ill-formed so we just have no read , so we need no write, 
 no operation. 
 
-  how:
 and in the loop   entry just preforms the superset of all range checks
 and picks one.
 
 
- conclusion:
writing  exception safe code with small simple functions is faster than writing complex code with less  invariant grantees. 
because the invariant is either true on initialized or not true on uninitialized,  we really can optimize based on the invariant and stay within as if rules.




---



fundemental types

- note :  
   a non binary endian architecture must emulate  a binary one , the existence of base 3  , analog and quantum computers must not interfere with declaration of a standard base 2 ABI.
 - l: little endian bytes but system  endian bits( the bits in a byte are in  system order, but a multi byte sequence has its bytes in reverse order of big endian)
 - b: big endian bytes but system  endian bits
 - L:little endian bits( the bits in a byte are in reverse order of big endian and bytes are also in reverse order of big endian)
 - B: big endian bits
 - none: pick system endian.




- `std::(l/b/L/E) (m/u)intN_t`:
 two's compliment integral type.
 N  goes from 1 ( fractional alignment, with math similar to c bit feilds) , 2 , 4, 8 ( byte aligned) , 16,....up to at least 1024 ( the 11 power of 2 starting feom the 0th power)  , while unnecessary, its better to have reliable deafults, especially  because  modorn cpus have massive registers.

 however if N is not a power of two , N must be between 1 and 64 
 with its bit alignment being the  prime factor of powers of 2

note that by using qualiexpr tricks and the std bounded integral types , one can have a fundemental type that the compiler known its a violation if it goes outside of its range.
these might be useful to help the compiler in optimizations of math , and maybe layout .


 1. nothing:

 overflow is a contract  violation, is a signed integral.

 2. u :

 overflow is a contract violation, is an unsigned integral.

 3. m :

 any overflow is well-defined, only devision by 0 is a contract violation,is unsigned modular arethmatic type.



- `std::(l/b/L/E) ((z/n/o/d/u)s)(B/eEmM)(u)(r/n)floatN_t`:

4, 8, 16,  32,64,80, 128 as N.

 

floating point types with N bits.

 also , s stands for stable, as in , the floating point math is platform independent,  although slower.
 however,  not having an s makes the float act as if it had the ffast math flag.
 for a cxx like behaviour one can still use cxx compat types , but using them is not recommended, 
 for cross platform and  multiplayer games using the s floating point is recommended to not have issues from incompatible math.
 for single pc or performance critical code using s is optional.
also if the endian can change across  platforms, using both S and explicit endian is recommended.

 - stable rounding modes ( s is necessary):
 1. to nearest even (n):
 round to nearest floating point , if there are two equidistant ones,choose the one whose least significant digit is even.
 2. towards zero (z):
 rounding to the floating point number closest to zero.
 3. downward (d):
 rounding to the greatest floating point. number blow
 4. upwards (u):
 rounding to least floating  point number above. 
 
 
 5. round to  odd (o):
 rounding to nearest odd floating point number. 
 
 
 -  layout modes:
  - B :
  brain float . 
  ( only   is valid with N of 16 )
  - e number:
  number bits as exponent.
  - m number:
  number bits as  mantisa.
  
  * note that using  e and m necceceraly makes using N not possible, because the addition of e, m and u indicates the bit count and , if bit   count is not devisible by 8 , fractional alignment. 
  
  - u:
  the sign bit is not there and considered 0, unsigned float.
  
  - r (deafult) vs n :
  real float, its a contract violation to have the  result of an expression of type real float not be a real number, 
 its undefined behavior if real float had  a non real value ( infinity or nan ),
 having a negative zero results  is a contract violation but not undefined behavior. 
 n however has implementation defined nan or inf results that do not result in contract violation.
 
  
  - note : 
   some implementations may have a templated non fundemental type that is used to represent  custom mantisa and exponents .
   however the +200  variants that have default mantisa to exponent ratio are fundamental types. 
   
   
 
 - `std::(l/b/L/E)((z/n/o/d/u)s)(u)(eE)(r/n)positN_t`


  - u:
  the sign bit is not there   value is positive, unsigned  posit.
  
  - e number:
  representative of  bit count  of the max es value.   
  
  - others: similar to the float  definition
  
  
  
  
 - `std::(l/b/L/E)((z/n/o/d/u)s)(u)(pP)(r/n)fixedN_t`:
   
  
  -  p number:
  representative of  bit count  of the fixed point fraction.
  the precision.
  
  
  - others: similar to the float  definition, however with fixed point arethmatic.


- `std::(l/b/L/E)charN_t`:

N is one of 8,16,32

charechter types with N bits. 
representative of UTF8, UTF-16 ( little endian  vs big endian) encoding ,UTF-32 ( little endian  vs big endian) encoding .
 not using b and l (endian-ness) picks the default ( for utf8 is big endian( b) , l for char8 is ill-formed but L is usable, others is system endian)



- `std::nullptr_t`:



type of a nullptr, its size is similar to a byte pointer.



  

- `std::(L/E) bool_t`(1 byte) ,`std::flag_t`(1 bit) ( a single bit cant have endian-ness):

  the bolean types 

  

-  `std::bit_t`:

the special bit type with special pointers and references , `sizeof(bit_t)` and any types with fractional alignments ( and therefore  sizes) are ill-formed,  instead,  `bit_sizeof(T)`,`bit_alignof(T)`can be used, also , the bit can alias all types with any  alignment.

 





- `std::(L/E) byte_t` / `void`:

the special byte type with the alias set of all types (with non fractional alignment, although it can alias the memory holing it).
also the special void type , with no layout,  although the size is not fractional


- `std::abi_t` ( hash is endian agnostic):



 the hash type that the abiof operator gives.

 its a 128 bit uuid , it doesn't support any operations outside of the ABI operators,  other than the usual load and store or casts.

 for example,  the hash given by xxhash128 or a better architecture dependent implementation. 

  this is good enough,  because the linker already has to append the cxx-style name mangle to the hash .

   `namenangle__hexed-hash`. 


-`std::debug_meta_t` /`std::meta_t` :
reflection types , similar in spirit and usage to the c++ reflection system, 
however,  these reflection types have  an important  but subtle   distinction, 
its that  their usage  might happen  without sequencal orders, 
 the compilers JIT  engine is responsible to provide language level tools through the standard to help developers making the compile time safer and faster.
 although,  because of the drive for performance, the compiler gives trust to the c colon language more than the c++ compiler,  if an unsafe usage of the reflection meta ,  
 or any  unsafe behavior in the constexpr execution executes UB , probably a violation was specified as unsafe contract ub ,  or a data race , although the implementations are encouraged to use sanitizers if the compiler flags are not set for max evaluation speed, the program is ill-formed no diagnostic required.
 this is rather scary,  however,  because the build system already allows for running code, and almost all projects do custom build commands , 
 this is already something that happens anyway.
 however compilers are encouraged to give warnings when entering an unsafe block in the constexpr runtime.
but , if implicit contract violation  during the constexpr runtime occurs the program is ill-formed, explicit ones are implementation defined if they will throw an exception or be ill-formed, however the deafult should be ill-formed.





- `std::type_info_t`( hash is endian agnostic):
the hash type that the  typeid operator gives.
representative of the back end hash used in the linker or v table.


 its a  256bit cryptographic hash, it doesn't support any operations outside of the compare or equal,  other than the usual load and store or casts.
if the cryptographic hash is given to an implementation defined function  `__mccabiv1::demangle` , it either  gives the empty string or the true front-end name mangle that lead to this hash , this outcome depends on security and rtti flags during compilation.






- `std::(l/b/L/E) cxx_(wchar/...)_t`:



 cxx compat types.
 note that a both endians are mapping to the same itanium name
 only the semantic logic  is preserved via the endian 







* note on the register  assigner:
 on top of the inout priority list , the register allocation  in both the function signature and the function body has preferences of implementation defined ways, 
for example,  bool , bit  and flag may be passed in a single bit of a register , or in a specific ALU flag ( if appropriate) , or the floating  point  values may have a preference of FPU registers( or SSE in x86), 
however this would only be used if the type is passed by value ( ie. trivially relocatable and is not refrence ) , refrence types or pointers also may have preference for specific registers that  can have  load and store ptr operands ( for example   load effective address  in x86)
 it is implementation defined what preferences are used , and if some non fundemental types also have preferences or not ( for example a sso string object may use compiler intrinsics  to say that a specific register category  of ymm is preferred , however  using intrinsics  is unsafe(magic) and probably  would need other unsafe blocks to indicate non standard abi)
 also , implementations are encouraged to make many to many mappings of structures  and  registers,  not  include padding , and pack many arguments into one register, or  brake up one argument to multiple registers, 
  alignment requirements  are relaxed but considered for preference mappings,  however, in general it is implementation defined per function signature  how the bits are  mapped from the arguments to the registers as long as all information is preserved.

* note on overlapping and timelines:
each of these 3 are allowed to overlap with each other,  because in   an instruction of call or return , only one of them is active.

 0. input mapping:
  how in, inout and in val arguments are mapped. 
 1. happy output mapping:
 how out , and inout argument's are mapped.
 return-value, out prameters,  out  part of inout prameters,  and the context-type are here for example.
 2.  sad output mapping:
 how out , and inout argument's are mapped.
  throw-value and the context-type are here for example.






* pointer types are different( system endian): 



`(memcast<uintmax_t>(bit_ptr)&~7)==8*memcast<uintmax_t>(byte_ptr)`.

 note that elidable  pointers have implementation defined layout and size.
 



1. c colon typical pointers( default ) : 

 these point to types with non fractional alignments

 most types do not have fractional alignment.

 these are typically cxx pointer sized 


note that almost all pointer used to store data are non fractional, heap allocators or stack allocators or coroutine  frame or static symbol allocator are all non fractional,
using fractional types in some contexts  adds padding to the end of them to align them to at least a byte.
 
 
 


 2.   c colon fractional pointers: 

  these point to types with fractional alignments, the reason is to make common bit feilds and flag  vectors easy.

  these are cxx  pointer sized in many cases ( 64 bit ptrs are big enough, however 32 bit ones will reduce the memory map 8 simes!, so  its not big enough)

  

 3. cxx pointers: 

 these are cxx compatible pointers  with cxx pointer sizes , pointer to any `represent_cxx` type.

  

  

  ---



limits



the architecture must have at least 5 pointer sized registers.

while possible for a very obscure architecture to use static variables as  registers , 

the design's focus on extreme register utilization might mitigate the gains ,

however,  modorn architectures have more than enough registers at their disposal. 

note that in architectures where `sizeof(cxx_char_t)!=1` , the `represent_cxx` is provided  to be used in ABI boundaries .

and for any cxx pointer `(memcast<uintmax_t>(byte_ptr)&~(sizeof(cxx_char_t)-1))==sizeof(cxx_char_t)*memcast<uintmax_t>(cxx_ptr)`, note that the lower bits in the c colon pointers in these architectures indicate shifts.



- hash sizes:
any implementation may choose hashes with size smaller than 256 or 128 



---
   debugging  :
   
  with current debugger technology, if the context-type satisfied the debugging concept,  the compiler will make code much less optimized, 
  will do little use of  cache friendly reordering , and will implicitly  ( through reflection) give the context-type very granular information using implicit calls providing reflection information and dynamic information ,
  most values will be lifted to stack for traceability,  this is extremely slow , and that's the reason that it only happens in scopes where context-type is mandating it ,
  this can help separate the binary debug vs release building model into function by function or module by module debugging 
  
  

  

---



 ABI  and compatibility:
 
 
 
 - hash elision:
 the hash calculation is only preformed when an externally visible/ v-table dependent symbol of 256 bit backend mangle needs calculation, 
 however because of the as if rule , we have a null hash function that gives the output as 0 , so any front-end mangle without a calculated hash is known to need a hash if its hash  is 0 , and the 0 is replaced with the actual  hash when its backend mangle is stored.
 also , the hash captured by the `abiof` operator is  eagerly calculated, and must not be 0 because of the as if rule.
 a hash of 0 given in `abi=` is ill-formed.



- the abi@() operators:

1. `abi+(t/abi_t)`:

adds the hash as a sult to the ABI hash of the apllied expression.

2. `abi=(t/abi_t)`:

sets the has as the ABI hash of the apllied expression.

3. `abiof(type/id)`:

gets the ABI hash off the inner expression.
anything that can result in an abi brake must be in the hash.

- the abi hash of a general symbol( namespace,  variable,  type,  function):
0. the ABI version number ( any changes to the ABI scheme in the standard will alter this number) 
1. architecture dependent version/hash ( for example , based on the amout and register priorities, the generic x86-64 win or generic  arm v7 ect....) , this can be specified  using the target qualifier  
1.  the   name , including  template prams, and the the parent  namespace or type  , including  template prams ( basically the itanium style mangle) 
3. abi+(...) es ABI hash and dclaration order



- the ABI hash of a  static variable also  depends on:
4. the type and its abi hash.



- the ABI hash of a  namespace also  depends on:
4. nothing more ,  a  namespace doesn't depend on the content. 




- the ABI hash of a type also  depends on:
4. non static member ABI hash and dclaration order
5. if not final qualified, virtual function ABI hashes and declaration order
6. virtual bases ABI hash and declaration order
7. bases ABI hash and dclaration order
8. qualifiers of a type , but order independent
12. diffrent trivially properties of a type.
13. lifetimes  and their dependancies ( the tokens and their hash in the definition of templates , lifetimes, contracts and requirements)
 14. if an enum , its entry values.
* note:
the abi hash of a function does not depend on non virtual member functions,  but only specific ones that effect triviality ( trivial copy , move , reallocation,  destruction, construction,...) do change the  triviality 



- the ABI hash of a  function :
4. throw-value , contract in val,  and promise-type and input and output of async/sync functions of the context and generally the specific kind of context-type and its abi hash.
5. the qualified type of the function pointer  ( so , the full hash of each thing in it , but the qualifier having being sorted to ensure they don't depend on the qualifier order).
6. if the function is a member function,  the qualified type of the self ( the first argument that has a `this` annotation , similar  to c++ newest deduction of this pattern), but , if a virtual function with a virtual self,  the qualifie type of self that  is final qualified. 

* note :
the ABI hash of a function ( not a type)  does NOT depend on the function's  code ( the function inner scope)


* note :
the module ir file name has no effect on abi hashes.



- note :

the c colon linker only uses the backend ABI hashes as  signature for linkage.




- choice of hash:

 because,  the hashes are only relevant for a function/type/namespace... with a specific name ( similar to the uniqueness of  name mangles in Cxx) , and the specific hash ,

 lets say a name has been  changed  N times , 

  the collision probability is one halfs if we have tried 2^64 different versions, based on the birthday paradox.

  as long as we haven't changed the name more than 2^(32) times , we are extremely unlikely to find a collision.

  and , because of the practical nature , we obviously do not have 4 gigabytes of functions/types/... *with the same name mangle* ( other than the hash) in the same binary,

  it is simply too much names  to be a practical limitation .

  


 

 the hashs dependent on their dependancies form a tree ( no cycles ) and any changes in any part of the tree will alter all above sections of the tree , similar to a murcle tree , so any changes in the ABI of a section will force all dependent sections to need a new link target .

 

 

 

---



runtime libraries and compilation



the objective of a full ABI is to allow arbitrary mixing of object files produced by conforming implementations, by fully specifying the binary interface of application programs.



- there are 8 main steps( might ):

 1. determine the dependency graph of the mcc files, irs and modules ( to help realize parallelization opertonities, and determine if some steps are not neccecery if the result is cached and valid).

 2. compile to the AST object/modules.

 3. compile the asts to mcc ir-0 .

 4. for every `constexpr` ir-0 path , evaluate all `constexpr` code and generate mcc ir-1 files.

 5. for every ir-1 file , optimize the code to ir-2 files.

 6.  using all of the summary and dependency graph information,  we make an ir-3 file for each partially-independent unit of execution in the graph to be optimized , we do the processing of these files then we, link all ir-3 files in the mcc linker to an ir-4 file, this is  ThinLTO style .

 7. map ir-4 the code to the target assembly, while optimizing unnececery register usage , for example by register allocation optimizations and generate an object file( also happens with the previous step).

 8. link the object files to an executable.







--- 

the mcc toolchain and ABI outside of c colon:



 with the absolute expressive power of the c colon IR and ABI ,  there might be languages or toochsins who soly focus on a language with c colon like ABI ,

 for example :

 

  1. the mcc-CPP transpiler: 

      a  compiler that compiles c++ into mcc-ir.

  2. mcc-rust:

     a compiler that compiles rust into mcc-ir.

  3.  functional colon ( F colon) ( complete  the CDEF  eco system):
     a functional language that compiles into c colon  or mcc-ir which most functions are implicitly dyn  purely functional  in  the c colon code.

  4.   express colon :
  a simple  language , a subset of c colon ,where only safe c colon code is valid ,most  qualifiers are invisible-implicit and inaccessibile.
  this is a more begginners friendly language that has the same ABI and type system as c colon , but without many of its complexities, so the unsafe parts can be written in libraries in c colon.    

      

---

express colon  or "E:":



a language  for those who like simplicity while writing c colon.



this language aims to be in the simple systems languages category.







this languages goals include:



the first and foremost goal is simplicity,  in contrast to c colon.

the second goal is being exceptionally safe ( pun intended).

the third goal is being blazingly fast ( lower priority than simplicity though).

by banning unsafe , all the complaints of complexity only fall on c colon. While e colon users complain about how boring it is.


 i categorize them , and show what is needed to achieve these :





1. deep integration with c colon:



 the language is an abstraction built on top of c colon,

 the c colon standard and libraries often focus on low level design while express colon is more friendly for newcomers. 

 

 

 2. simplicity:

 

  most complex code are implicit ,

  for example instead of references , developers use in/`inout`/out and values , 

  this makes this code readable and object oriented while being very safe and optimizable.

  containers are represented by value oriented objects and no poiner or reference semantics is required. 

  it  might not be as fast as c: , it may use more dynamic types or copies,  but for application logic its far more elegant,

  and if it wasnt optimal , you can always go a level down to c colon land to optimize it.

 

3.  safety :



     the reason for safety being granteed for express colon, ( if the c colon libraries  used internally are well written and safe ) is that there is no referencing to begin with,  imagine using a member from a vector,  you aren't using a reference to it , so you must copy it ,so you will never worry if the vector reallocation will invalidate anything,  because you do not have any reference to begin with. 

     you dont need to borrow anything because you only need to change its value,  most functions can use either full value semantics or fall into using a reference-counted variable if they truly need multiple ownership.

    any unsafe code is delt with in the internal c colon libraries,  freeing the burden of safety concers from the developers.

    references being disallowed means that express colon by definition cannot use after free, cannot use invalid state,  cannot even intract with dead state, 

    because the only thing allowed to be modified is the value of objects,

    even data races cannot be made in express colon because the reference counted variable provided by c colon is either a totally immutable value or protected via a read write lock ( assuming the c colon libraries dont provide safe looking unsafe abstractions)

    although  dead locks can be a concern in highly parallel environments,  they don't really cause memory unsafty.

    even stack overflows can be tracked  if context objects use the reflection information into a stack trace ( for example by having a counter incremented by the stack usage of a frame when the context begins and having a maximum threshold before a contract violation occurs).

    contracts like pre post and assertions can be checked through any means of function call , so the safety of a function is always  preserved. 

    while,like rust , its still possible to make a self referential  reference counter `std::rc_t`/`std::arc_t` who will be "leaked", this is not a common issue and not a memory unsafty,  it still can be resolved with `std::weak_rc_t` or `std::weak_arc_t` ( proving this will not happen  by the compiler is very hard , and resolution of this issue requires a garbage collector and graph traversal,  which is not what we want, but a memory leak is still not a security exploit but a programmer bug , similar to a deadlock)

    also,  if you noticed that you need an `abi=` in a structure, because the hash of the internal reference needs the hash of the external loop , congratulations, you've found the part that needs a weak reference, this is because the hash dependency chain created via a dependency  of T to its arc<T> and arc<T> to T is exactly the thing that makes arc<T> a candidate for a self referential reference. 

    or your just trying to implement a tree , graph or linked list , which you can do via reference counted variables , but your notified of its potential for a self reference when you used `abi=` to make it compile again. 

    however,  an extreme measure against all cycles is  making the use of `abi=` as unsafe(abi=) , this makes any E colon code unable to make any liked list , graph or tree like structure and etc , and severly limits many forms of inheritance,  but it grantees that all reference counters will be freed .

    any use of `abi=` is unsafe and so E colon programs cannot have memory leaks and need to drop to c colon for creating such structures,   the reason fot this is , lets assume T has a storage mechanism to a tree , this tree either doesn't have T ( which means no cycles to T) or it does , if it does , T's ABI hash would become dependent  on the graph that is itself dependent on T , and because we cannot type erase T to not depend on itself , and  we cannot cause a brake in the ABI chain via `abi=` , then we really cant form a cycle ( assuming c colon libraries dont provide any type erasure primitives , but only sum types ( like rust `enum` or CPP std variant) ) ( because  the virtual table ABI is dependent on the type of the  class argument,and the class  is dependent on the virtual table), ( and std::any like types are not provided to E colon because its too low level for it) 

    arguably this is extreme , and we cant always grantee that no open-set type erasure will be provided from c colon,   but i would say that if E colon developers want to make a self referential type, it would be more elegant in C colon , and probably there are graph, linked list  and tree libraries that can do that.

    and ,abi= is already low level enough to be a c colon only spec,

    for this reason,  all forms of non trivial type erasure ( erasure of a type with non trivial destructor) is unsafe ,

    for example , E colon can only do bitcast if and only if the type of source and dest are trivially relocatable and trivially destructable and have no pointer/references in their layouts ( to prevent memory leak via type erasure), but in C colon , we can do unsafe(bit-cast) to do any form of bit cast( or other casts) . also , beacuse making a thread is unsafe(threads) , the async scheduler is in c colon and E colon remains free of its compications.
    this makes E colon not need any garbage collector for leak prevention,  and because of the hidden thread safety qualifiers ,  e colon can use non atomic refrence counters and only use atomic ones when necessary 
    leak free graph and linked list support is also present in the library. 
 `std::graph_t<T> ,std::node_t<T> ,  std::node_t<T[n]> , std::node_t<T[]>`( note that T of void( type erased ) is still safe , because this is just an index and needs the graph at the end of the day to know the type)
 the standard node is a leak safe node , to be used instead of depending on unsafe ABI operators, 
 the standard node can be thought of as an index into the standard graph ,
 the standard graph manages all lifetimes of all objects es in it ,
 a standard node's contract is violated if  used on a graph that is not the original owener.
 the standard node's internal implementation  can be just a  pointer, an index , an array , or a vector, however because the only way a to access it is using `graph[node]` we ensure that the lifetime is valid because the graph is alive , the check is also a fast range check in the graph's allocated region, also this gives the graph very fast locality because its region is continuous,
the library implementation may use a bit allocator to see which chunks in the region are empty , to not have to use memory movement.
 or the implementation may choose an actual graph implementation,  and have a root node for checks
  
 there are type/lambda erasure primitives that ensure the type within them is    const  stabilized (via reflection, for example a mutex or a non immutable refrence counted object cannot be this way),
 this is because  if the data graph doesn't freeze at creation,  it might make a memory leak when the function calls itself with itself as its prameter and stores it in itself, 
 thats why open set type erasure is highly restricted  in Express colon. 
 
4. speed :



 even in-value oriented code,  functions can throw errors with ease and speed , the values often flow in registers and they are optimized well beacuse of lack of aliases .

 happy path often has less branches compared to using optional/expected/result types, and sad path is more performant than cxx-throw or rust panic, and as performant as optional types.

 the variable access is often faster , because in architectures like x86 , almost 1 kilobytes can be passed between functions via prameters in registers with ease.

 

5. rich set of libraries:



the standard library aims to have many of the widely used utilities,  for example networking libraries,  asynchronous frameworks and web like UI development utilities. 

this is because the standard can change the ABI of these components at later versions so it doesn't need to worry about backwards compatibility. 



6. easy to use package management with ABI stability:



users don't need to worry about using old or vulnerable code , or complex building process. 



7. safe with rare  borrowing errors:



 the E colon domain of the code aims to be  type-safe,memory-safe , exception-safe and violation-safe uncless specified otherwise via dropping into c colon unsafe.



theres no traditional borrowing in express colon , because references aren't allowed,  only value oriented reference-like alternatives are, this is because most simple objectives can be achieved soly via containers, dynamic referencing containers and value types,

the express colon language also tends to look more functional than its c colon counterparts,  many changes to heap variables happening in monadic like fashion if they are garded via a mutex, or value based of they are easily movable and copyable, while the underlying c colon has borrowing rules,  express colon programs still dont need lifetime anotations ( the c colon libraries however probably do)



8. fast enough application logic:

 

 the function argument flow specifiers ( like a reference  but without stable address on trivial reallocation) the in (akin to const & )  , out (akin to mutable uninitialized& ) and `inout` (akin to mutable& ) prameters are all trivial reallocation and register passed, 

 however because of the ABI nature they will remain valid for the duration of the function call so they are inheritly safe. 

 the in-val ( no specifier) does a copy or a drop/relocation  on most occasions .

 these are ideal for register usage ,  but they introduce more copying and occasionally the need for reference counting if dynamic  referencing is necessary, 
 also because of the strict exception safety  that grantees invariants,  we can optimize on those invariants.
although this is fast enough so its good enough,  if not , c colon can be used to optimize  it more.







  

  9.  elegant parallel programming with safety and structured concurrency:

  

  for example,  common  monadic operations can be made in a lambda function that is hidden inside of a for loop, this doesn't look like were using a monadic operation here , but we implicitly are doing that, while still benefiting from readability of c style for loops, iteration-primitive can be a mutex,  can be a vector,  can be an optional, 

  the option for monadic lambdas is provided,  but common  monands can be expressed with ease.

 ( note that iteration-primitive is written in c colon as it involves more complex machinery)



 ` for ( inout variable: iteration-primitive) {`
// function body beginning, the function captues the sate and has an `inout` argument 

// modify variable.

`}`// lambda scope end



`for ( auto [inout a, in b, out c, d ]: iteration-primitive){`// function body beginning, the function captues the state and has an multiple argument provided in the iterator internals.

// d is copied , a , b and c are "referenced" via value input outputs

// modify outputs.

// loop control flow is dictated with iterator context and the context-type 

// it not a coroutine so it has:

// makes the iteration primitive call `operator break(...) context-type ->lambda-return`

`break;`

// makes the iteration primitive call `operator continue(...) context-type  ->lambda-return`

`continue;`

// makes the iteration primitives call `operator return (...) context-type -> lambda-return`

`return...;`

// theres an implicit continue at the end of scope   

`}`// lambda scope end , once the function ends via the iteration-primitive, it can either implicitly return a value or continue execution or throw.





// the context object  acts like the promise type in c++, providing much needed abstractions , providing many low level primitives in c colon , however , most coroutine usage is restricted in express colon to safe usage of libraries. 

// all of these co@ operators do implicit calls reliant on the context-type and the iterator context.

// theres an implicit  transformation for these code , to make it able to do either a ,co await , co return or a throw or simply  continue execution .

 `for co_await (auto [inout a, in b, out c, d ]: parallel-iteration-primitive){`// the iteration primitives may restrict the lambda to only caputure `thread_safe` constant state if it wants to do parallelization , a const unstable `mutex<T>` however has internal  unrestricted unstable qualification of its members, some even atomic, therefore  its valid for it to modify its members even tho it looks constant. 

// can modify a c and d , but cannot modify other variables outside of the for loop , however mutexes can still be modified beacuse they can be modified when constant.

// parallelizable async co routine code....

// converted into  `operator co_value` in the construction and `operator ~co_value` in the destruction 

 `co_value std::async_io_file_t file(...);`// makes an object  capable  of async construction and destruction,  and async destruction runs in cancelation,  cancelation is achieved by throw exceptions after a suspended co await is resumed 



` .... = co_yeild...;`// produces a result 

 `.... = co_await...;`// awaits a result 

` co_return ...;  ` in a parallel loop does a cancelation of its siblings. and a total function return.

` co_break;` in a parallel loop does a cancelation of its siblings. and  function continues.

` co_continue;` // ends the execution of the current coroutine , but doesn't result in a cancelation. 

 `}`// lambda scope end , once the function ends via the iteration-primitive.



// a lambda  can only do  relocations  in its caputure in E colon , `fn(){}` is implicitly `fn[:=](){}`, however in c colon we have other types too, copy in c colon is   like `fn[=](){}` in c++, however with fn before it, but with `fn(){}` the implicit relocation `:=` one, the caputure after fn using `[...]` cannot be specified under E colon, so only relocation constructors of lambda is supported ,for a copy , we can do a copy in the main scope then relocation into the lambda, this can be thought  of making a new function and giving our box to it for usage , so we lost it.
// the implicit lambda for iteration-primitive is the only place where caputure is by reference in E colon , and this is also heavily restricted under the iteration-primitive rules
// also because the scope begins and ends predictably,  and the caller is conceptually paused until the end of the iteration,  this refrences caputure is almost always a pass under implicit borrow rules. 

// note : a   relocation is `dest:=src` in assignment , copy is `dest=src`, move is `dest=std::move(src)`  ( well.. cpp syntax is my choice because im too entrenched in its quirks but im open to other systems of syntax for all languages  as long as E colon code can just compile as C colon code with a file rename and it gains the votes , for example cpp2 or rust )

//... many other for variants to help build readable and reliable abstractions. 




// also a special case  for example used in more relaxed async code,   this doesn't require  thread safe qualification but still requires const qualifier,  using a   non atomic  refrence counter however can allow for mutability , because it only allows a single thread to be scheduled , but the drawback is that the whole structured concurrency tree with this root has its scheduler changed to a single-threaded one,  however the mutex and atomic overhead is  unnecessary when  using this method.
`for co_await (auto [`inout` a,  in b, out c, d ]:  asynchronous-single-threaded-iteration-primitive){`
...
`}`

// we can do a combination of these , for example have multiple single-threaded workers that  each run  in parallel. 


// also we have  `lvl(rw)_mutex<level>` ,
// in the scheduler  each leveled mutex would be  recorded in a async local stack wait data structure, and if a new leveled mutex is used , then its a violation of the try precondition  if the level is higher or equal than the last level in the level stack.

//  there are different compiler flags , however in a critical system,  using a mutex without a level would be `unsafe(deadlock)` while in a non critical system it would just be safe because we don't care about deadlock normally .



// theres also transactional memory which  i will just point to the transactional memory in c++ , they are safe  simple and composable  , although they are more restricted on what is possible ( idempotent expression  only ) 

// thers also  massage passing via channels,  for example multi consumer multi preduces queues










10. easy errors:



i pridict that the worst common error is an easy "must initialized an out prameter" or "make a copy of a variable and use that instead of passing itself to multiple function prameters"( the reason being that variables are relocated to the prameter when the prameter is created and will be relocated back when the prameter is "destroyed"  in c colon land , but , often the only use after-rellocation-error is these , which can be avoided  by declaration of a new variable  ), which is , simple and far better than lifetimes or memory bugs.

or at most a " the catch block cannot caputure a variable that might be dropped in the try block , try copying that variable before the try block to ensure it will not be an output of a throwing function"( the `inout` or out function  arguments  will  have an uninitialized state or qualifier  on that functios throw path in the caller, when used , the same qualifier set per expression rule will make that expression ill formed).

in contrast to C colon ( which allows all of these c++ style things) , Express colon does not have operator overloadding,and also function overloadding,  and template specializations, and only allows simple template declarations ( like the c++ auto concept constraint prameters),

this isn't really a safety problem,  but an error massage problem,  

if express colon wants readable error massages in its express colon module,  it needs to have less type anotations necessary to show that error message,  and therefore,  much type information would not be shown because the function name and at most the namespace would be sufficient for its detection. 

most template specifications can just be ignored by default in express colon,  however C colon errors may not have much clarity without those informations. 

however E colon still recognizes these C colon constructs , allowing custom types like `std::(u)intdyn_t` ( similar to python's big integer types) to be used by their overloaded operators.

 similarly, common pitfalls like cyclic object hash dependency chains have relatively good information on their errors " it seems that your building a graph like structure within your types , however types dependent on themselves tend to be error prone , firstly,  use `abi=` operators to resolve the cyclic hash , secondly,  if dynamic referencing is involved,  consider using weak pointers as well, because it would help avoid leaks".

 note that definition of a destructor, makes it so that a drop on member  is ill formed in many cases ( because the destructor needs a full object,  but one part is uninitilized, especially on the throw path) , this would make it so that using an inout would be incorrect, and would also encourage the use of setters and getters for correct logic ,  but using an in , and a separate out would be correct,  but would result in a copy in the callee because the callee can't change the in pram directly.
 
 there is a similar restriction in c colon , but c colon can just use non dropped references,  however in E colon that is disallowed for exception safety, note that these rules are a combination of borrow checker and qualifiers rules , done in a double entry book keeping style in the compiler. 

 
11. exception safety:



 because of the implicit one qualifier set per expression rule, any E: function that might modify its arguments on the throw statement,  the arguments would not be usable after that throw, 

 that's why all the value oriented catch blocks would not encounter unexpected states (those who were in the process of completion but couldn't because of an exception).

 although this means that often containers who have a single incomplete value would also get dropped or put in an empty state ,

 for example  what happens if an exception happens in a reference counted mutex modification lambda , that mutex would need to be put in an error state , this way , any later code that attempts to access that mutex would throw an incomplete mutex error.

 in my opinion,  this is a good thing,  even rust panics dont have such properties,  because if a value is modified through a reference then it throws and attems to catch a panic ,  the previous value is gone! , it is memory safe to access it yes , but it might be incomplete so its s logic error.

 also , in my view, it is bad to assume panic safety is something obscure, 

 any function and its contracts might fail and need to propegate errors ,

 even if the error is a bug , in a critical system it is not really good to just crash the program or put it in a bad state.

  i think that terminations is very brutal , because of this exception handling mechanism , i think all violations are represented by an exception, 

  this is fast and effective , violations *can* be terminations,  and violation catching is unsafe , but generally program will be fine and exception would be lead to a  terminate in main with very sane stack trace and debug info ( if we choose).
  there is a very subtle but important consideration,  a result return differs in meaning from   an exception in Express colon,   a result return  means that the function successfully did its operation,  however with the nuance of some invalid result,  a throw means that the function failed , and the data it used is also a not usable and  the function  effectively could not do any useful data transformation,  but a result still allows the following functions to use the state from the function signature because its a successful modification,  even though its partial.
  and because all code is exception safe without even trying ( pun intended) , this secondary way can just indicate other forms of recoverable errors, and both can coexist without much overhead( the jump return table and code size isnt that much relevant in  application logic ),
  and an important consideration is that both of these mechanics have exactly the same way of calling ( enumret  spec) so its just a matter of  invariant  ensuring( return) vs non invariant ensuring code( throw).


12. value and data oriented code :
having rust-like `enum` types with pattern matching  are still very performant and value oriented alternatives to inheritance.

 and object oriented ( without inheritance) would still be whidly used ,

 for example constructors would output the self object to  out prameters,

 copy would use the in parameter and moving/ relocation would be automatically generated(  not using any references would make types trivial to automatically relocate if inner c colon types are trivial) . 

 the destructor would also use in-val ( no specifier) to relocate the object for final destruction. 

  these aren't just safe , these are also fast , because trivial values are passed by registers , and this language mostly operates on trivial values 
however,  because of unsafty of non trivial type erasure,  and refrences caputure.  lambda  ,
it also leans to functional programming,  even tho its imperative 

 





--- 

roadmap:



1. build a C colon subset language front-end for llvm in c++:

this in itself will require substantial work .



2. build one or more  standards documents with clear information,  similar to the cxx spec, and the cxx ABI specs:

this would need to be designed carefully and diligently to ensure that the language stays well-defined.



3. ( dependent on 1 and 2) migrate the C colon front-end to a  C colon codebase:

not in scope rn.



4. ( dependent on 3) replacement or extension ( probably a fork ) of the llvm back-end, linker:

not in scope rn, but the reason being is to be able to nativity support the mcc ABI in all platforms instead of piggybacking it on top of the c ABI .
in this step , the license will become  more permissive ,  ( GPL removal)


5. ( dependent on 4) rewriting the full toolchain in c colon:

not in scope rn





- considerations: 

yes , this is too ambitious to build in few years,let alone quickly,  however if we let ourselves to build the 2 first parts by not focusing on performance ( because the ABI is build on top of llvm ir with its conventions around calls ), we may be able to get the language up and running to build itself eventually in the following years to come. 



--- 
the colon dynamic or Colon D language:

a scripting language with a garbage collector,   designed  for fast development,  also for easy debugging, tooling and prototyping.
there are some restrictions on what a  colon D type can do ,  the arguments are implicitly `inout` ,
however the type may remain opaque ( if an explicit type is specified then its a violation for a non value type to be passed)
 colon D also doesn't have refrences similar to E colon, however using `rc(obj)` on them will make the object reference counted ( no atomic required because of the nature of concurrency, for ease of use,  )
a value is always  of type  `stdd::any` ( can be thought of as an index-into-interpreter/value) , also , 
the type can be used within a context with `stdd::interpreter_context_t` , its a violation if the interpreter doesn't match the one that manages the lifetime.
the any type in E colon facing code has set , get , cast, info , name, clear and other functions to be able to be usable.
therfore value must always be trivially relocatable,
must either copy , move  or do contract violation if these are not defined but required.
the value also would need to be destructable.
all functions in colon D are implicitly dynamically exported on definition,  and imported on declaration, 
all functions in colon D  are of type `stdd::function` ( all colon D functions are implicitly asynchronous ) ,
the `stdd::function`  is the way to manage the code ,
also  colon D is executed concurrency( but not in parallel, only via asynchronous concurrency, in a single thread, therefor no atomic or mutex nee) in the `stdd::interpreter`( a drivitive of the standard scheduler, using an interpreter object directly is unsafe) and the functions  have `stdd::interpreter_context_t` ( a drivitive of the `std::async_context_t`)
if a function qualifier  is lang(stdd::lang), the reflection functions make sure it has the appropriate context type.
the D garbage collector is executed in the implicit  context-type operator code.
its similar to java script,  however with less explicit type conversion.
colon D  scripting can be easily used in E colon and  E colon functions can be easily injected into colon D. 
this language can be used in the web , similar to E colon ,via wasm





--- 

 improvments and advancements compared to cxx ( in the performance category):
  0. so maybe one day CPP can be as fast:

  i always have loved C++ , even seeing it now with all its legecy , cxx still can be like c colon , heck i wanted c colon to be a c++ superset , but i dont think i can change anything major in it anytime soon, in hopes that a std c++xx maybe powered by mcc  can be as fast as c colon  in the following decades
  
 1. ABI brakage leads to better implementations:
 the 128 bit recursive hash ABI makes this possible.

 2. minimal and fast stack spill :

 when all the registers modifted are known , we can just put values in those that are not modified,

 when presssure is too high , all the registers are stored(call)/loaded(return) at once (or even have entier hot code sections that do this via a `move ret ip; jump __mccabiv1::pop_all or jump __mccabiv1::push_all`), on architechures like x86 , cpus can parrarelize multiple stores or multiple loads but not intertwined load and stores like those in cxx calls.

 also the fastdyncaller qualifier really helps reduce the register usage.

 all the arguments and their data flow in registers also helps significantly.
 also , i doubt that even asm experts can track register allocation across all programs , its too much mental overhead for humans.
 


 3. dynamic cast and smaller objects:

   a bit bigger v-table sizes but much faster cast with less cache miss.
   smaller object size because of the v-table pointer only being in the refrence type.



 4. fast exception paths, fast happy path,fast `enum` types :

 all with the split return channel, no if on the happy path , no cxx lib unwind on the sad path , just caller code, i envision compilers even collapsing offsets to particular values to avoid table duplication.

 

 5. rich aliasing info , and function purity metrics:
 my philosophy is , Every immutability comes with a disability,in e colon , progrmmers are forced with a disability to gain safety and immutability ,  and even boring simplicity. the c colon programmers however should be allowed to declarar the disability to get the performance from the  immutability, for example stable values dont need to be loaded two times in registers to be captured! , they can be loaded only one time only because of their stability ,  the lack of others to change its value is the disability. 


6. layout optimizations :
  on top of rust like memory reorder , c colon has non `not_offset_dependant` qualifiers,  meaning that if a sub-object is referenced in memory , the entire object does not need to be placed in memory,  but only that sub-object,  especially true for trivially relocatable types ,
  for example  if i have an array of offset independent members,  and reference a member,  i can just only use the memory of that member and other members may not be in ajason stack memory , also it might help in making array of structures to structure of array
  on the stack ,  reflection also can help with this
  
 7. heap allocation elision :
 its similar to how cxx does coroutine frame elision but with std::allocator or custom allocators that satisfy the allocator concept.

 8.  less interposition depending dynamic dispatch  :
   when code is similar and linked together, it can be sometimes de duplicated,
also if a function's address is not taken or exported it doesn't need a fixed address, also the relaxed function addresss help in dew duplicated code.
also static data like strings would be able to merge better with  valexpr ( because  no null termination,  sub string merge is safe)

9.  fast program and dll loader:
all mcc symbols have two kinds of name mangles , the front-end mangle and the back-end mangle ,
the front-end one is like cxx but with the ABI hash appended , the back-end mangle is a 256bit hash of the front-end mangle ,
this is to make all symbols have a fixed size and to be able to store all symbols needing work on startup or dll load in an  array of sorted hashes ( 32 arrayes of bytes beacuse of 256 bit nature) ( similar layout to what mcc dynamic cast `castation-tables` had)
, ( because of the hashed  nature we can use the fast radix sort to make this array in backend in the linker) this helps along side the interposition less code , to help reduce start up times.
when loading a dll , those sorted arrays of hashes combine into one array , similar to the merge (O(n)) in merge sort ,
while doing so , we can spot all duplicate symbols if any and do the appropriate thing accordingly 

10. more parrarelized code  and structured concurrency:
by enabling cancelation in the ABI of coroutines we can swiftly do many structural concurrency patterns, 
std primitives such as tasks , channels,schedulers,  promises etc help here .

11. invariant based optimization:
read the "contract speed in this language" section.
allows transactional invariant preserving code to be very fast


---
```
    Copyright (C) 2025 Mjz86 



    This program is free software: you can redistribute it and/or modify

    it under the terms of the GNU General Public License as published by

    the Free Software Foundation, either version 3 of the License, or

    (at your option) any later version.



    This program is distributed in the hope that it will be useful,

    but WITHOUT ANY WARRANTY; without even the implied warranty of

    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the

    GNU General Public License for more details.
```


---



references



Mjz C colon summary:

[link](https://github.com/Mjz86/c_colon)



Mjz  colon compiler design and details:
[link](https://github.com/Mjz86/c_colon/blob/main/mcc.md))



Quantum-Secure Encryption is Here. And it's WILD ( hash verification) :

[link](https://youtube.com/watch?v=T9fCCGzwHJc&si=BzDNoo-vLqmsSJ7g)



Itanium ABI :

[link](https://Itanium-cxx-abi.github.io/cxx-abi/abi.html#intro)



x86 cxx:



[link](https://gitlab.com/x86-psABIs/x86-64-ABI)



arm cxx:

[link](https://github.com/ARM-software/abi-aa/blob/main/cppabi64/cppabi64.rst)



wg21 std c++ draft standard :

[link](https://wg21.link/n5008)





xxhash128:

[link](https://xxhash.com)



shared libraries and loader in cxx windows and Linux: 

[link](https://www.youtube.com/watch?v=_enXuIxuNV4)




Teresa Johnson “ThinLTO： Scalable and Incremental Link-Time Optimization” :

[link](https://www.youtube.com/watch?v=p9nH2vZ2mNo)





CppCon 2017： Michael Spencer “My Little Object File： How Linkers Implement C++” :

[link](https://www.youtube.com/watch?v=a5L66zguFe4)


Contracts, Safety, and the Art of Cat Herding - Timur Doumler - C++ on Sea 2025 :
[link](https://www.youtube.com/watch?v=gtFFTjQ4eFU) 

Understanding Compiler Optimization - Chandler Carruth - Opening Keynote Meeting C++ 2015 :
[link](https://www.youtube.com/watch?v=FnGCDLhaxKU) 


CppCon 2018： Matt Godbolt “The Bits Between the Bits： How We Get to main()” :

[link](https://www.youtube.com/watch?v=dOfucXtyEsU)


memory order:
[link](https://en.cppreference.com/w/cpp/atomic/memory_order.html)



Balancing the Books,Access Right Tracking for C++ - Lisa Lippincott - C++Now 2025 :
[link](https://youtube.com/watch?v=wQQP_si_VR8)

De-fragmenting C++： Making Exceptions and RTTI More Affordable and Usable - Herb Sutter  CppCon 2019 :
[link](https://youtube.com/watch?v=ARYP83yNAWk)


The Day The Standard Library Died:
[link](https://cor3ntin.github.io/posts/abi/)



---



