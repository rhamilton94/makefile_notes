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



## Rules
### Single Target Rules
Makefiles are made up of rules:
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

## Variables

`=` assignment defines recursive variable (child variables checked when parent variable is used, not declared)
```make
one = one ${later_variable}
later_variable = later

all: 
    echo $(one)

```
```
> one later
```


`:=` assignment defines non-recursive variable (allows appending, prevents infinite loops)
```make
one = hello
one := ${one} there

all: 
    echo $(one)

```
```
> hello there
```



`?=` only sets variable if they haven't been set
```make
one = hello
one ?= goodbye
two ?= world

all: 
    echo $(one)
    echo $(two)

```
```
> hello world
```


## Automatic Variables

`$@` = target

`$<` = first prerequisite

`$^` = all prerequisites

`$?` = all prerequisites newer than target

`$(@D)` = directory of target

`$(@F)` = filename of target (no path)


## Important Variables
Important variables to use when writing a makefile are:

`SHELL`: Default shell to execute commands, default is `/bin/bash`

`CC`: Program for compiling C programs, default is `cc`

`CXX`: Program for compiling C++ programs, default is `G++`

`CFLAGS`: Extra flags to give to the C compiler

`CXXFLAGS`: Extra flags to give to the C++ compiler

`CPPFLAGS`: Extra flags to give to the C preprosessor

`LDFLAGS`: Extra flags to give to compilers when they are supposed to invoke the linker


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
Static pattern rules are defined in the format \<targets\>:\<target pattern\>:\<prerequisite pattern\>
```make
objects = foo.o bar.o all.o
all: $(objects)
    
$(objects): %.o: %.c    # c file compiled to o file using complicit rule

all.c:
    echo "int main() { return 0; }" > all.c

%.c:
    touch $@
    
```






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






## Functions

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





## Useful Examples
Replace all C files in directory with object files
```
$(patsubst %.c,%.o,$(wildcard *.c))
```

