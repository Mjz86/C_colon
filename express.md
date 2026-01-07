 # how express colon aims to achieve memory safety:
 
 lets start with a simple  question,  what does everyone who starts C complain about ?
 i think most people would say pointers , lets try to make a system language without pointer and sew what happens, 
 first  , we need a way to  modify objects in place , for example  by passing them to a function, 
 lets try to understand how  a CPU  does that. all modorn cpus have registers,  and coincidentally these registers dont usually have address in memory, 
 if we think about it , the programmer cannot take a pointer  to a register,  how do they pass them to functions then? well , the cpu just does it for them ,
 so , lets go back to our theoretical language,  lets say when i pass a value to a function  i just do what the cpu did  ,
 but , modorn cpus have more than registers , so for the language to not be useless,  we need some why to access memory, 
 how can we access memory without a pointer in the language?  
 
 
 thats where c colon , the co language that has all the complexity come in , 
 this language is intended for low level code , and can do very powerful things like what c++ does .
 but this comes at a cost of immense complexity, and to shield everyone , the first language can only interface this language in safe and sinple ways.
 
 
 lets view the goals of express colon :
 
 the first and foremost goal is simplicity,  in contrast to c colon.
 we achieve this by banning any complexity that comes .
 
 
the second goal is being exceptionally safe ( pun intended).
 we achieve this  by banning pointers and references, instead only using containers or reference counted container types.
with  context dependent information  we prevent stack overflow before the stack gets created,  
and the recursive application binary interface compile time checks make sure the reference counted types dont form a cycle 


the third goal is being blazingly fast ( lower priority than simplicity though).
 our system language that despite a garbage collector doesn't have memory leaks,
 this is because its value oriented  and many paradigms like inheritance are unsafe,  because mutable type erasure bypasses the compiler type graph cycle detection,  its simply disallowed, however we still have easy to use concurrency mechanisms that are managed under the hood.
 the language  also aims to use registers heavily and fix the slow exception and result type returns in the ABI.
 
 
 
 
