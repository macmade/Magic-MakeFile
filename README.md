Magic MakeFile
==============

About
-----

The goal of the Magic MakeFile is to provide developers with an automated build system, that can be used for various compiled projects, and easily configured to suit the projects' needs.

It has been created in order to match the following features:

 * To be much easier to use than the GNU AutoTools.
 * To be compatible with many programming languages.
 * To be extendable so actually unsupported programming languages can be added.
 * To allow the generation of shared objects and/or libraries.
 * To deal with dependancies between final executable file(s) and shared objects or libraries.
 * To be highly configurable.
 * To give the end user a clean and localizable output.

### Supported languages

Actually, the Magic MakeFile can be used for projects written in C, C++, and Objective-C.  

Documentation
-------------

### 1. Getting started

In order to follow this tutorial, simply download the Magic MakeFile.  
Please also ensure that the 'make' command is available on your system, and that its version is at least 3.81.

To check the version of GNU make, type the following from a Terminal window:

    make -v

You should see the following output:

    GNU Make 3.81
    Copyright (C) 2006  Free Software Foundation, Inc.
    This is free software; see the source for copying conditions.
    There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

If you see `bash: make: command not found`, or if the GNU make version is smaller that 3.81, you will need to install or update GNU make.

GNU make can be downloaded from the GNU website.  
If you are using Linux, make should be available from your favourite package manager system.

### 2. Files & directories

After decompression, you will get a directory with the following structure:


 * `build` The directory containing all the build files, by category (see below).
 * `bin` The directory where the final executable files are stored.
 * `lib` The directory where the generated libraries (if any) are stored.
 * `obj` The directory where the resulting object code files for your source files are stored.
 * `makefile` The magic makefile itself (which should not be edited).
 * `makefile-code` The directory containing the per-language configuration options.
   * `C.mk` The settings for the C programming language.
   * `C++.mk` The settings for the C++ programming language.
   * `Objective-C.mk` The settings for the Objective-C programming language.
 * `makefile-config.mk` The project's configuration file
 * `makefile-lang` The directory where localization files are stored.
   * `en.mk` The english labels.
   * `fr.mk` The french labels.
 * `source` The directory for your source code.
   * `include` The directory for your project's include files.
 * `lib` The directory for the source code of your project's libraries (if any).
   * `include` The directory for the include files of your project's libraries (if any).

### 3. Starting & configuring a new project

When starting a new project, the only file you have to edit is the `makefile-config.mk` file, that contains all the build settings related to your project.

No other file needs to be edited.

We will create a typical `HelloWorld` C program.  
So in the `source` directory, create a `hello.c` file. This will be our main executable.

Place the following code in that file:

    #include <stdlib.h>
    #include <stdio.h>
    
    int main( void )
    {
        printf( "hello, world\n" );
        return EXIT_SUCCESS;
    }

We need to configure the Magic MakeFile so it builds an executable from our C file.

Open the `makefile-config.mk` file and look for the following line:

    EXEC = main
    
This defines the names of the final executable files. You can add multiple names by separating each one by a space.  
For the example, simply replace that line by the following:

    EXEC = hello
    
The Magic MakeFile now knows that it should generate an executable named `hello` from our `hello.c` file.

Also notice the line saying:

    code = C
    
This is where you configure the programming language you use.

Now, from a Terminal window, cd into the project's directory and type:

    make clean
    make
    
The first line ensures all output files are removed, so all of our source code is compiled again.   Otherwise, it just re-compiles the files that have been modified since the last build.

The second line will produce the following output, meaning the executable was successfully generated:

    #--------------------------------------------------
    # Beginning make script
    #
    # Version of GNU Make needed: 3.81
    # Current version of GNU Make: 3.81
    #--------------------------------------------------
    
    --- Finding and building the libraries
    --- Done - All libraries were processed
    
    --- Finding and building the shared objects
    
    ------ Building the object file for ./source/hello.c in ./build/obj/
    ------ Done
    
    --- Done - All shared objects were processed
    --- Finding and building the executables
    
    ------ Finding dependancies for hello
    ------ Done - hello does not depend on shared objects
    
    ------ Building the executable hello for build/obj/hello.o in ./build/bin/
    ------ Done
    
    --- Done - All executables were processed
    
    #--------------------------------------------------
    # End of the make script
    #
    # Thanx for using this makefile
    # Have a nice day
    #--------------------------------------------------

You will find the resulting executable in the `build/bin/` directory.  
To test it, type:

    ./build/bin/hello
    
### 4. Dependancies

Now what if we want to have a 'sayHello' function, placed in a separate file, and called from our C main function?

The Magic MakeFile can automatically take care of that kind of dependancy.

So let's create another C file called 'functions.c' in the 'source' directory. Place the following code in that file:

    #include <stdio.h>
    #include "functions.h"
    
    void sayHello( void )
    {
        printf( "hello, world\n" );
    }
    
Also create a C header file called `functions.h` in the `source/include/` directory, with the following code:

    #ifndef FUNCTIONS_H
    #define FUNCTIONS_H
    #pragma once
    
    void sayHello( void );
    
    #endif
    
Now, let's modify the `hello.c` file, so it uses the 'sayHello' function:

    #include <stdlib.h>
    #include <stdio.h>
    #include "functions.h"
    
    int main( void )
    {
        sayHello();
        return EXIT_SUCCESS;
    }

Note that even if we placed the header file in the `source/include/` directory, we can include it by using the file name, without the `include/` prefix, because the Magic MakeFile automatically searches the `source/include/` directory for header files.

The last step is to let the Magic MakeFile know that the `hello.c` file has a dependancy on the `functions.c` file.

This is done by editing the `makefile-config.mk` file.

Locate the line that says:

    DEPS_main =

That's the place where we can put dependancies for an executable called 'main'.

As our executable is called 'hello', simply replace that line by the following one:

    DEPS_hello = functions

You can place multiple dependancies by separating them by a space.  
If you have multiple executables, you can of course add other `DEPS_` lines, one by executable.

Now, if we build our project:

    #--------------------------------------------------
    # Beginning make script
    #
    # Version of GNU Make needed: 3.81
    # Current version of GNU Make: 3.81
    #--------------------------------------------------
    
    --- Finding and building the libraries
    --- Done - All libraries were processed
    
    --- Finding and building the shared objects
    
    ------ Building the object file for ./source/functions.c in ./build/obj/
    ------ Done
    
    ------ Building the object file for ./source/hello.c in ./build/obj/
    ------ Done
    
    --- Done - All shared objects were processed
    
    --- Finding and building the executables
    
    ------ Finding dependancies for hello

    --------- functions
    
    ------ Done
    
    ------ Building the executable hello in ./build/bin/ by linking ./build/obj/hello.o with its dependancies: 
    
    --------- ./build/obj/functions.o
    
    ------ Done
    
    --- Done - All executables were processed
    
    #--------------------------------------------------
    # End of the make script
    #
    # Thanx for using this makefile
    # Have a nice day
    #--------------------------------------------------

We can see the executable was successfully generated.

The Magic MakeFile produced two intermediate object code files (`functions.o` and `hello.o`), and produced the final executable by linking the two object code files together.

### 5. Libraries

Rather than generating object code files for the dependancies, you can also choose to tell the Magic MakeFile to generate static libraries.

To do this, simply put the C code of your libraries in the `source/lib/` directory (and the related header files in the `source/lib/include/` directory).

In the `makefile-config.mk` file, use the `DEPS_LIB_` variable instead of the `DEPS_` variable.  
Let's demonstrate that.

Create a new C file called `libfunctions.c` in the `source/lib/` directory, and it's header file, `libfunctions.h` in the `source/lib/include/` directory.

Note that the name of the library files MUST begin with the `lib` prefix.  
This is a requirement of the linker.

Here's the source code of `libfunctions.c`:

    #include <stdio.h>
    #include "libfunctions.h"
    
    void sayHelloFromLib( void )
    {
        printf( "Hello Universe!\n" );
    }

Here's the source code of `libfunctions.h`:

    #ifndef LIBFUNCTIONS_H
    #define LIBFUNCTIONS_H
    #pragma once
    
    void sayHelloFromLib( void );
    
    #endif

Modify the main C file, so we also call the `sayHelloFromLib` function:

    #include <stdlib.h>
    #include <stdio.h>
    #include "functions.h"
    #include "libfunctions.h"
    
    int main( void )
    {
        sayHello();
        sayHelloFromLib();
        return EXIT_SUCCESS;
    }

Adds the dependancy in the `makefile-config.mk` file:

    DEPS_LIB_hello = libfunctions

You can now build your project.

### 6. System libraries

You can also link final executables with system libraries.  
For instance, in order to link with the libcrypto library:

    DEPS_SYSLIB_hello = crypto

That will tell the makefile to add additional linker flags for the system libraries (in that case -lcrypto).

### 7. Debug

You can tell the Magic MakeFile to display the commands it uses to produce the executables by setting the `DEBUG_` variables of the `makefile-config.mk` file to one.

Note that you can also set those variables directly from the command line.  
For instance, for the previous example:

    make clean
    make DEBUG_LIBTOOL=1 DEBUG_CC=1

You will be able to see how the magic MakeFile generates the build files.

License
-------

Magic MakeFile is released under the terms of the BSD license.

Repository Infos
----------------

    Owner:			Jean-David Gadina - XS-Labs
    Web:			www.xs-labs.com
    Blog:			www.noxeos.com
    Twitter:		@macmade
    GitHub:			github.com/macmade
    LinkedIn:		ch.linkedin.com/in/macmade/
    StackOverflow:	stackoverflow.com/users/182676/macmade
