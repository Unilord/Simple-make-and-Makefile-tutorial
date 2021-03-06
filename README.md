# Simple-make-and-Makefile-tutorial

![Deathly Hallows](http://bit.ly/2k7EA6R)  

*To prepare to use make, you must write a file called the makefile that describes the relationships among files in your program and provides commands for updating each file. In a program, typically, the executable file is updated from object files, which are in turn made by compiling source files.*

Once a suitable makefile exists, each time you change some source files, this simple shell command:
``` shell
    make
```

suffices to perform all necessary recompilations. The make program uses the makefile data base and the last-modification times of the files to decide which of the files need to be updated. For each of those files, it issues the recipes recorded in the data base. 

## How things usually work 
There is a source file that contains the code, when you compile it then an object file is created 
This file is generated by the compiler and is called the object code ( .obj ), but a program like the "hello world" program is composed by a part that we wrote and a part of the C++ library for an example. The linker links these two parts of a program and produces an executable file. 

## An Introduction to Makefiles
You need a file called a *makefile* to tell *make* what to do. Most often, the *makefile* tells *make* how to compile and link a program.  
Lets get down to real **business** then  

**Writing MakeFiles**  
A simple makefile consists of “rules”  
**What a Rule Looks Like**  
Usually a rule is written in the following way:
```
target … : prerequisites …
        recipe
        …
        …
```
*make* carries out the *recipe* on the *prerequisites* to create or update the target. A rule can also explain how and when to carry out an action. [For more on writing rules click here](https://www.gnu.org/software/make/manual/make.html#Rules)  

A *target* is usually the name of a file that is generated by a program; examples of targets are executable or object files.A target can also be the name of an action to carry out, such as *clean* or *all*.  

A *prerequisite* is a file that is used as input to create the target. A target often depends on several files.  

### A Simple Makefile
Here is a straightforward makefile that describes the way an executable file called edit depends on eight object files which, in turn, depend on eight C source and three header files.  

In this example, all the C files include defs.h, but only those defining editing commands include command.h, and only low level files that change the editor buffer include buffer.h. 
```
edit : main.o kbd.o command.o display.o \
       insert.o search.o files.o utils.o
        cc -o edit main.o kbd.o command.o display.o \
                   insert.o search.o files.o utils.o

main.o : main.c defs.h
        cc -c main.c
kbd.o : kbd.c defs.h command.h
        cc -c kbd.c
command.o : command.c defs.h command.h
        cc -c command.c
display.o : display.c defs.h buffer.h
        cc -c display.c
insert.o : insert.c defs.h buffer.h
        cc -c insert.c
search.o : search.c defs.h buffer.h
        cc -c search.c
files.o : files.c defs.h buffer.h command.h
        cc -c files.c
utils.o : utils.c defs.h
        cc -c utils.c
clean :
        rm edit main.o kbd.o command.o display.o \
           insert.o search.o files.o utils.o
```           

The other rules are processed because their targets appear as prerequisites of the goal. If some other rule is not depended on by the goal (or anything it depends on, etc.), that rule is not processed, unless you tell make to do so (with a command such as make clean). 
 
## How make Processes a Makefile
 
By default, *make* starts with the first target (not targets whose names start with ‘.’). This is called the default goal. (Goals are the targets that *make* strives ultimately to update. __You can override this behavior using the command line ([see Arguments to Specify the Goals](https://www.gnu.org/software/make/manual/make.html#Goals)) or with the .DEFAULT_GOAL special variable ([see Other Special Variables](https://www.gnu.org/software/make/manual/make.html#Special-Variables)).__  
In the simple example of the previous section, the default goal is to update the executable program *edit*; therefore, we put that rule first.  
*make* reads the *makefile* in the current directory and begins by processing the first rule. In the example, this rule is for relinking *edit*; but before *make* can fully process this rule, it must process the rules for the files that *edit* depends on, which in this case are the object files. Each of these files is processed according to its own rule. These rules say to update each ‘.o’ file by compiling its source file. The recompilation must be done if the source file, or any of the header files named as prerequisites, __is more recent than the object file__, or if the object file does not exist.    
After __recompiling__ whichever object files need it, *make* decides whether to relink *edit*. This must be done if the file *edit* does not exist, __or if any of the object files are newer than it__. If an object file was just recompiled, it is now newer than *edit*, so *edit* is relinked.  
 
 
*If you want to use a nonstandard name for your makefile, you can specify the makefile name with the ‘-f’ or ‘--file’ option. The arguments ‘-f name’ or ‘--file=name’ tell make to read the file name as the makefile. If you use more than one ‘-f’ or ‘--file’ option, you can specify several makefiles. All the makefiles are effectively concatenated in the order specified*
 
 
## Yet another example (Kernel Module Programming)

__The hello.c__ (*Dont yet get lost in this just a sassy example*)
```C
#include <linux/module.h>    // included for all kernel modules
#include <linux/kernel.h>    // included for KERN_INFO
#include <linux/init.h>      // included for __init and __exit macros
MODULE_LICENSE("GPL");
MODULE_AUTHOR("Unilord");
MODULE_DESCRIPTION("A Simple Hello World module");
static int __init hello_init(void)
{
    printk(KERN_INFO "Hello world!\n");
    return 0; // Non­zero return means that the module couldn't be loaded
}
static void __exit hello_cleanup(void)
{
    printk(KERN_INFO "Good Bye.\n");
}
module_init(hello_init);
module_exit(hello_cleanup);
```
__Makefile__
```
obj-m += hello.o
all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```
Kernel Makefiles are part of the kbuild system, documented in various places on the web, for example [The relevant excerpt is here:](http://lwn.net/Articles/21835/)  
Goal definitions are the main part (heart) of the kbuild Makefile. These lines define the files to be built, any special compilation options, and any subdirectories to be entered recursively.  
The most simple kbuild makefile contains one line:  

Example: obj-y += foo.o  

This tell kbuild that there is one object in that directory named foo.o. foo.o will be build from foo.c or foo.S.  
If foo.o shall be built as a module, the variable obj-m is used. Therefore the following pattern is often used:  

Example: obj-$(CONFIG_FOO) += foo.o

$(CONFIG_FOO) evaluates to either y (for built-in) or m (for module). If CONFIG_FOO is neither y nor m, then the file will not be compiled nor linked.  
[This is an answer by Peter on Stackoverflow](http://stackoverflow.com/users/1401351/peter)

- *all* and *clean* are phony targets.(refer to the section mentioned below)  
- The *make* command you see in the code is a __Recursive Use of make__ that is calling *make* inside the *makefile* . Now this *make* needs a *makefile* too to know its job hence *__-C__* to change directory which contains big daddy of all the makefiles.
- make M=dir clean Delete all automatically generated files
- make M=dir modules Make all modules in specified dir




## Some cool short definations 

###  1.Rule Syntax
A rule tells make two things: when the targets are out of date, and how to update them when necessary.  
A target is out of date if it does not exist or if it is older than any of the prerequisites (by comparison of last-modification times). The idea is that the contents of the target file are computed based on information in the prerequisites, so if any of the prerequisites changes, the contents of the existing target file are no longer necessarily valid.

### 2.Phony Targets

A phony target is one that is not really the name of a file; rather it is just a name for a recipe to be executed when you make an explicit request. There are two reasons to use a phony target: to avoid a conflict with a file of the same name, and to improve performance. 
```
clean:
        rm *.o temp
 ```

### Topics we have not covered but are important 
1. Types of Prerequisites
2. Using Wildcard Characters in File Names
3. Writing Recipes with Directory Search
4. Overriding Part of Another Makefile
5. Generating Prerequisites Automatically
6. How make Reads a Makefile

 __Some information that you might want to know__  
1. __cc -c__  Compile or assemble the source files, but do not link.The linking stage simply is not done.The ultimate output is in the form of an object file for each source file.  
2. __*make* Deduce the Recipes__ When a ‘.c’ file is used automatically in this way, it is also automatically added to the list of prerequisites. We can therefore omit the ‘.c’ files from the prerequisites, provided we omit the recipe.  

### References 
- [Offcial GNU make manual](https://www.gnu.org/s/make/manual/make.html)
- [Stack Overflow](www.stackoverlow.com)
