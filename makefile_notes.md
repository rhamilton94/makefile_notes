# makefile_notes

## What is the Purpose of Makefiles?
Makefiles are designed to keep track of:
1. Which files to include when compiling
2. Which files are dependent on other files
3. Which order to compile files in
4. Which files are up to date and which need recompiling


## How Does C Compilation Work?
1. Preprocessing 	- remove comments, replace variables and #defines with actual values, insert header files
2. Compiling 		- convert C code to Assembly (specific to target CPU) 	
3. Assembly		- convert Assembly code to Machine Code (binary)
4. Linking		- combine object files into single executable, link code to library functions

program.c > `preprocessing, compiling, assembly` > program.o > `linking` > program

<br/><br/>

## Rules
Makefiles consist of various rules about how to compile each file needed for the program
### Single Target Rules
```make
target: prerequisite
    command
    command
    command
```
Rules are made up of:

`target` = output file(s)

`prerequisites` = input file(s)

`command` = command to execute

A set of commands belonging to a rule is called a `recipe`

```make
blah.o: blah.c
    cc -c blah.c -o blah.o
```
### Multiple Target Rules
If multiple targets specified, same commands can be executed for all targets
```make
f1.o f2.o:
    echo $@
```
Is equivalent to:
```make
f1.o:
    echo $@
f2.o:
    echo $@
```

A rule defined for a file that is a prerequisite of another rule will be executed first:

```make
blah: blah.o			# will execute second
    cc blah.o -o blah
    
blah.o: blah.c			# will execute first
    cc -c blah.c -o blah.o
```


### Implicit Rules
Certain rules do not have to be defined, the makefile understands what needs to happen when a certain target and prerequisite pairing is used.

The rule `file.o: file.c` will implicitly contain the command `$(CC) -c file.c -o file.o $(CPPFLAGS) $(CFLAGS)`

The rule `file.o: file.cpp` will implicitly contain the command `$(CXX) -c file.cc -o file.cpp $(CPPFLAGS) $(CXXFLAGS)`

The rule `file: file.o` will automatically execute the command `$(CC) $(LDFLAGS) file.o $(LOADLIBES) $(LDLIBS)`


<br/><br/>


## Command Line Arguments

Specify file name
```
make -f <filename>
```

Silent mode (prepend `@` to every command)
```
make -s
```

Continue running even if errors occur
```
make -k
```

Ignore all errors (prepend `-` to every command)
```
make -i
```
<br/><br/>

## Variables

`=` assignment defines recursive variable (child variables checked when parent variable is used, not declared)
```make
one = one ${later_variable}
later_variable = later

all: 
    echo $(one)

```
```
> make all
one later
```


`:=` assignment defines non-recursive variable (allows appending, prevents infinite loops)
```make
one = hello
one := ${one} there

all: 
    echo $(one)

```
```
> make all
hello there
```



`?=` only sets variable if they haven't been set
```make
one = hello
one ?= goodbye
two ?= there

all: 
    echo $(one)
    echo $(two)

```
```
> make all
hello there
```
  
  
`+=` appends to a variable
```make
foo := hello
foo += there

all: 
    echo $(foo)
```
```
> make all
hello there
```
 
<br/><br/>

Variables can also be set for specific targets or patterns only:
```make
all: one = cool
%.c: two = groovy

all: 
    echo one is defined: $(one)

foo.c:
	echo two is defined: $(two)

other:
    echo one is nothing: $(one)
	echo two is nothing: $(two)
```


  
<br/><br/>

Variables must be set **outside** of any rules, otherwise they will be interpreted as build commands and cause an error.


Executing this:
```make
# echo BAR > foo
FOO = `cat foo` 		
all:
	@echo THE CONTENTS OF FOO IS $(FOO)
```
Outputs the intended result:
```
> make
THE CONTENTS OF FOO IS BAR
```

While if done like this:
```make
# echo BAR > foo
all:
	FOO = `cat foo` # echo BAR > foo
	@echo THE CONTENTS OF FOO IS $(FOO)
```
The Make program will interpret `FOO` as a program, and attempt to execute it, causing this:
```
> make
FOO = `cat foo` # echo BAR > foo
/bin/sh: 1: FOO: not found
make: *** [makefile:2: all] Error 127
```
By using `=` and not `:=` to define a variable outside of a recipe, the value can be set when it is used, not when it is defined.


<br/><br/>

## Automatic Variables

`$@` = target

`$<` = first prerequisite

`$^` = all prerequisites

`$?` = all prerequisites newer than target

`$(@D)` = directory of target

`$(@F)` = filename of target (no path)

<br/><br/>

## Important Variables
Important variables to use when writing a makefile are:

`SHELL`: Default shell to execute commands, default is `/bin/bash`

`CC`: Program for compiling C programs, default is `cc`

`CXX`: Program for compiling C++ programs, default is `G++`

`CFLAGS`: Extra flags to give to the C compiler

`CXXFLAGS`: Extra flags to give to the C++ compiler

`CPPFLAGS`: Extra flags to give to the C PreProcessor

`LDFLAGS`: Extra flags to give to compilers when they are supposed to invoke the linker

<br/><br/>

## Wildcards

`$(wildcard *)` Searches for matching filenames in current working directory
```make
$(wildcard *.c)     # find all .c files
```

`%` can be used in several ways:

1. As a string replacement with `patsubst` (pattern substitution):
```make
foo := a.o b.o l.a c.o
bar := $(patsubst %.o,%.c,$(foo))
@echo $(bar)
```
```
> a.c b.c l.a c.c
```

2. To define a rule for all files of a certain type:
```
# when hellomake is called, this rule 
# will execute for both hellomake.o and hellofunc.o
%.o: %.c
	$(CC) -c -o $@ $<

hellomake: hellomake.o hellofunc.o 
	$(CC) -o hellomake hellomake.o hellofunc.o 
```


3. When using static pattern rules

Static pattern rules are defined in the format `<targets>:<target pattern>:<prerequisite pattern>`
```make
objects = foo.o bar.o all.o
all: $(objects)
    
$(objects): %.o: %.c    # c file compiled to o file using complicit rule

all.c:
    echo "int main() { return 0; }" > all.c

%.c:
    touch $@
    
```


<br/><br/>

## Rule Precedence
If there are multiple rules that match a target name, the rule will be chosen based on this order of precedence, from highest to lowest:

Explicit filename with path: `lib/foo.c`

Implicit filename with path: `lib/%.c`

Explicit filename: `foo.c`

Implicit filename: `%.c`

```make
all: lib/foo.c
	echo "finished"

%.c:
	echo "%.c chosen"

foo.c:
	echo "foo.c chosen"

lib/%.c:
	echo "lib/%.c chosen"

lib/foo.c:
	echo "lib/foo.c chosen"
```

<br/><br/>


## Conditional Statements
Use `ifeq` and `ifneq` for if and not if statements:
```make
foo = ok

all:
ifeq ($(foo), ok)
    echo "foo equals ok"
else
    echo "nope"
endif
```

If variable is empty:
```make
nullstring =
foo = $(nullstring)

all:
ifeq ($(strip $(foo)),)
    echo "foo is empty after being stripped"
endif
ifeq ($(nullstring),)
    echo "nullstring doesn't even have spaces"
endif
```

If variable is not defined:
```make
bar =
foo = $(bar)

all:
ifdef foo
    echo "foo is defined"
endif
ifdef bar
    echo "but bar is not"
endif
```
```
>make all
foo is defined
```

<br/><br/>

## MAKEFLAGS
`$(MAKEFLAGS)` represents a string of all command line flags used to run the Makefile.
```make
all:
	@echo $(MAKEFLAGS)
```
```
> make all -e -i -k -s
eiks
```
The Makefile can make decisions based on what command line flags have been used.
This is done by checking that the output of the `findstring` command is `n`ot `eq`ual to nothing:
```make
ifneq (,$(findstring i, $(MAKEFLAGS)))
    echo "i was passed to MAKEFLAGS"
endif
```


## Commands
Execute a shell command and return it's output by inserting it between backticks:
```make
FOO := `cat test`
all:
	@echo FOO IS $(FOO)
```
Or use the `shell` command:
```
BAR := $(shell cat test)
all:
	@echo BAR IS $(BAR)
```


Each line of the makefile is executed separately, for example this:
```make
    cd ..
    echo `pwd`
```
will print the current working directory, **not** the parent directory, while this:
```make
    cd ..; echo `pwd`
```
and this:
```make
    cd ..; \
    echo `pwd`
```
WILL print the parent directory, as the two commands are now executed in the same "shell"

<br/><br/>


## Recursive Make
When compiling a program, you might want to include a dependency that has it's own makefile.
To include this in the primary makefile, use `$(MAKE)` instead of `make`.

Using the `$(MAKE)` variable will use the same program (e.g. `/bin/make`, `cmake` etc.) and flags that were used to execute the primary makefile 
```make
cd subdir && $(MAKE)
```

<br/><br/>

## Passing Variables to Child Makefiles
To pass a variable from a parent Makefile to a child Makefile, use the `export` command:
```make
foo = bar
export foo
```
or
```make
export foo = bar
```
To prevent a variable from being passed, use the `unexport` command.

To pass ALL variables from a parent Makefile to a child Makefile, define a rule without a recipe or prerequisites containing `.EXPORT_ALL_VARIABLES` as the target:
```make
.EXPORT_ALL_VARIABLES:
```

Variables explicitly defined in a child Makefile take precedence over variables passed to it from the parent Makefile, unless the `-e` command line parameter is used. 
```
make all -e
```


## Overriding Command Line Arguments
To ensure that a variable defined in the Makefile takes precedence over a command line argument, use the `override` command.
For example if the following Makefile is executed:
```make
foo = bar
all:
	@echo the value of foo is $(foo)
```
With the command `make all foo=hello`, the output will be:
```
the value of foo is hello
```
However, if the `override` command is used:
```make
override foo = bar
all:
	@echo the value of foo is $(foo)
```
The output of the same command `make all foo=hello` will be:
```
the value of foo is bar
```
<br/><br/>

## Define Directive

The `define` directive can be used to define multiple commands under a single name (each command is still executed separately):
```make
define two_lines
@echo "foo"
@echo "bar"
endef

all:
	$(two_lines)

```
```
> make all
foo
bar
```

<br/><br/>


## Functions

Functions are used for text processing, structured in the format
```make
$(fn, arguments)
```
or 
```make
${fn, arguments}
```

<br/>

### Pattern Substitute String (patsubst)
```make
$(patsubst pattern,replacement,text)
```

Finds whitespace-separated words in `text` that match `pattern` and replaces them with `replacement`. Here pattern may contain a `%` which acts as a wildcard, matching any number of any characters within a word. If replacement also contains a `%`, the `%` is replaced by the text that matched the `%` in pattern. Only the first `%` in the pattern and replacement is treated this way; any subsequent `%` is unchanged.

<br/>

### For Each

Performs an action against each word in a space separated list of words:
```make
$(foreach var,list,text)
```
For example:
```make
foo := who are you
# For each "word" in foo, output that same word with an exclamation after
bar := $(foreach wrd,$(foo),$(wrd)!)

all:
    @echo $(bar)
```
Outputs
```
who! are! you!
```

<br/>

### IF
Slightly different to the conditional statement explained previously
```make
foo = a.c
bar = a.a a.b a.c a.d a.e

statement := $(if $(filter $(foo),$(bar)),@echo "found", @echo "not found")

all:
	$(statement)
```
Executing `make all` outputs
```
found
```

<br/>


### Filter
Specify files that match specific pattern
```make
$(filter <pattern>,<files>)
```
For example:
```make
obj_files = a.o b.o c.a d.o
echo $(filter %.o,$(obj_files))
```
```
> a.o b.o d.o
```

<br/>

## Call

Allows the creation of custom functions. The syntax is `$(call function_name,param,param)`
Positional arguments are used within the function:
```
$(0) = function_name
$(1) = first param
$(2) = second param
```
For example
```make
custom_func = Function Name: $(0) First: $(1) Second: $(2) Empty Variable: $(3)

all:
    @echo $(call custom_func, foo, bar)
```
Executing `make all` outputs
```
Function Name: custom_func First: foo Second: bar Empty Variable:
```

<br/>

### Shell

Execute shell commands, but replaces newlines with spaces so presentation is messy
```make
all: 
    @echo $(shell ls -la) 
```

<br/><br/>


## Include other makefiles
To execute another makefile from inside the current makefile, use the command
```make
include /path/to/new/makefile
```

<br/><br/>

## VPATH
The vpath is used to specify the location of various prerequisite files. The syntax is
```
vpath <pattern> <directories (space or colon separated)>
```
For example, to add all `.h` files present in the directories `../headers` and `../other-directory`
```make
vpath %.h ../headers ../other-directory
```

<br/><br/>

## Multiline commands
Use a backslash `\` to continue a command on the next line:
```make
some_file: 
    echo This line is too long, so \
        it is broken up into multiple lines
```


<br/><br/>

## .PHONY
If the makefile creates a file that has the same name as a recipe, the recipe will not execute if the file is up to date.
For example, if a recipe creates a file called `clean`, the commonly used recipe `clean` will not run if the file is up to date.
To get around this, use `.PHONY: <target name>` like this:
```make
some_file:
    touch some_file
    touch clean

.PHONY: clean
clean:
    rm -f some_file
    rm -f clean
```
Adding `.PHONY: <target name>` before the recipe will add the target as a prerequisite to the `.PHONY` target.
Once this is done, `make clean` will run the recipe regardless of whether there is a file named `clean`.

<br/><br/>



## .DELETE_ON_ERROR
If `.DELETE_ON_ERROR:` is added at the start of the makefile, all targets will be deleted if the make tool encounters an error during execution.
For example:
```make
.DELETE_ON_ERROR:
all: one two

one:
    touch one
    false

two:
    touch two
    false
```


<br/><br/>



## Useful Examples
Replace all C files in current directory with object files
```
$(patsubst %.c,%.o,$(wildcard *.c))
```

<br/><br/>


## Resources

Official GNU Reference Manual
https://www.gnu.org/software/make/manual/make.html

Quick reference
https://www.gnu.org/software/make/manual/html_node/Quick-Reference.html

Functions reference
https://www.gnu.org/software/make/manual/html_node/Functions.html

Best tutorial with examples (including blank template)
https://makefiletutorial.com/

Other tutorials
https://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/

Cheat sheets
https://devhints.io/makefile
https://gist.github.com/tuannvm/c0dcf6bd052d5607fff040c235103928
https://gist.github.com/rueycheng/42e355d1480fd7a33ee81c866c7fdf78

C compilation process
https://medium.datadriveninvestor.com/compilation-process-db17c3b58e62


