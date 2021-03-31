# makefile_notes


Makefiles are made up of rules:
```make
target: prerequisite
  command
  command
  command
```
Rules are made up of:

`target` = output file

`prerequisites` = input file

`command` = command to execute

```make
blah.o: blah.c
    cc -c blah.c -o blah.o
```

Rules can also include other rules as prerequisites:

```make
blah.o: blah.c
    cc -c blah.c -o blah.o
    
blah: blah.o
    cc blah.o -o blah # Runs third
```




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

```make

```



