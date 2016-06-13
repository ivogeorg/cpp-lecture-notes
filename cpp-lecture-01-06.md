### Build (2)

The build process is initiated when you press **Run-->Build** in CLion. It prepares your program for execution. It has three stages: 

  1. Preprocessor
  2. Compiler
  3. Linker
  
The preprocessor and compiler work on one file at a time. The linker puts together all files into a single executable program file.

The preprocessor takes a source code file and:

  * prepares the file for the compiler
  * handles all [preprocessor directives](http://www.cplusplus.com/doc/tutorial/preprocessor/) (e.g. `#include`, `#define`) and removes them
    * the #include directive results in the inclusion of the header file into the current file
    * expands in place all macros that have been defined with #define
    * substitutes in place all constants defined with #define
  * removes all comments
  * generates a translation unit, which is the input the compiler actually compiles

The compiler takes a translation unit and:

translates the C++ code into machine code (using the instruction format of the target processor architecture (Links to an external site.))
produces an object file (NOTE: Not to be confused with C++ class objects.)
includes information about all declared functions and all defined functions in the object file
The linker takes all the files and:

resolves all function declarations and definitions across all files, making sure there is one and only one definition per function
assembles all object files into a single executable program
Class design (1)
Inline functions are functions which:

are simple and fast (e.g. a single assignment)
do not need to incur the overhead of having a stack frame when called
are defined either within the class definition body or outside with a prefix inline
get their code substituted by the compiler every place they are called (so, should not be overused) by the rest of the code
Here is an example with the Point class:
