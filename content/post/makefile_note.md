---
title: Some random important Makefile Notes 
date: Oct 26, 2024
categories:
- Programming_Language 
tags:
- random_notes
---

## This is just a quick personal note for [Learn Makefiles With the tastiest examples](https://makefiletutorial.com/#makefile-cookbook).

> Note: Makefiles must be indented using TABs and not spaces or make will fail.

### most simple 

A sample makefile. There can be multiple commands in one target. 

```makefile
targets: prerequisites
	command
	command
	command
```

- The targets are **file names**, **separated by spaces (can be multiple?)**. Typically, there is only one per rule.
- The commands are a series of steps typically used to make the target(s). These need to start with a tab character, not spaces.
- The prerequisites are also file names, separated by spaces. These files need to exist before the commands for the target are run. These are also called dependencies
- make will check whether the prerequisites are newer than the targets, if yes, rerun the commands. 

### Wildcards

For wildcard `*` and `%`, always wrap them into the `$(wildcard *.c)` function. 
- `*` searches your filesystem for matching filenames.

> `%` is really useful, but is somewhat confusing because of the variety of situations it can be used in.
> - When used in "matching" mode, it matches one or more characters in a string. This match is called the stem.
> - When used in "replacing" mode, it takes the stem that was matched and replaces that in a string.
> - % is most often used in rule definitions and in some specific functions.

### Static Pattern Rules (usage of wildcard `%`):

For all *.o targets, depend on the corresponding *.c. 

```makefile
# Syntax - targets ...: target-pattern: prereq-patterns ...
# In the case of the first target, foo.o, the target-pattern matches foo.o and sets the "stem" to be "foo".
# It then replaces the '%' in prereq-patterns with that stem
$(objects): %.o: %.c
	$(CC) -c $^ -o $@
```

and pattern rules: 

```makefile
# Define a pattern rule that compiles every .c file into a .o file
%.o : %.c
		$(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

### Static Pattern Rules and Filter: 

Filter from the values of the variable. 

```makefile
obj_files = foo.result bar.o lose.o
src_files = foo.raw bar.c lose.c

all: $(obj_files)
# Note: PHONY is important here. Without it, implicit rules will try to build the executable "all", since the prereqs are ".o" files.
.PHONY: all 

# Ex 1: .o files depend on .c files. Though we don't actually make the .o file.
$(filter %.o,$(obj_files)): %.o: %.c
	echo "target: $@ prereq: $<"

# Ex 2: .result files depend on .raw files. Though we don't actually make the .result file.
$(filter %.result,$(obj_files)): %.result: %.raw
	echo "target: $@ prereq: $<" 

%.c %.raw:
	touch $@

clean:
	rm -f $(src_files)
```

### Automatic variables:

```makefile
hey: one two
	# Outputs "hey", since this is the target name
	echo $@

	# Outputs all prerequisites newer than the target
	echo $?

	# Outputs all prerequisites
	echo $^

	# Outputs the first prerequisite
	echo $<
```

### Double-Colon Rules:

The double-colon rules are independent rules. Depend on the prerequisites, they can be invoke none, one, many or all together. 
The target either all ordinary or all double-colon. 

```makefile
all: blah

blah::
	echo "hello"

blah::
	echo "hello again"
```

### Other tricks:

Add an @ before a command to stop it from being printed
You can also run make with -s to add an @ before each line
```
all: 
	@echo "This make line will not be printed"
	echo "But this will"
```

Each line is its own shell: 

```makefile
all: 
	cd ..
	# The cd above does not affect this line, because each command is effectively run in a new shell
	echo `pwd`

	# This cd command affects the next because they are on the same line
	cd ..;echo `pwd`

	# Same as above
	cd ..; \
	echo `pwd`
```

Default shell: 
```
SHELL=/bin/bash

cool:
	echo "Hello from bash"
```

If you want a string to have a dollar sign, you can use `$$`. This is how to use a shell variable in bash or sh.

Use $(MAKE) to invoke another make, then it will **pass the make flags for you and won't itself be affected by them**.

Use `export` to pass make variables across sub-makes. 
    - `.EXPORT_ALL_VARIABLES` exports all variables for you.

### Error handling with `-k`, `-i`, and `-`

Add `-k` when running make to continue running even in the face of errors. Helpful if you want to see all the errors of Make at once.
Add a `-` before a command to suppress the error
Add `-i` to make to have this happen for every command. (modify makefile source?)


### Variables advanced:

There are two flavors of variables:
- recursive (use `=`) - only looks for the variables when the command is used, not when it's defined.
- simply expanded (use `:=`) - like normal imperative programming -- only those defined so far get expanded
    - This can be used as self-append. (see example below)
    - or use `+=` to append. 

```makefile
one = hello
# one gets defined as a simply expanded variable (:=) and thus can handle appending
one := ${one} there

all: 
	echo $(one)
```

- `?=` only sets variables if they have not yet been set

- You can override variables that come from the command line by using override. Here we ran make with make option_one=hi

```makefile
# Overrides command line arguments
override option_one = did_override
# Does not override command line arguments
option_two = not_override
all: 
	echo $(option_one)
	echo $(option_two)
```


- Target-specific variables
    - Variables can be set for specific targets
```makefile
all: one = cool

all: 
	echo one is defined: $(one)

other:
	echo one is nothing: $(one)
```

### Conditional statement: 

- Conditional if/else

```makefile
foo = ok

all:
ifeq ($(foo), ok)
	echo "foo equals ok"
else
	echo "nope"
endif
```

- Check if a variable is empty

```makefile
nullstring =
foo = $(nullstring) # end of line; there is a space here

all:
ifeq ($(strip $(foo)),)
	echo "foo is empty after being stripped"
endif
ifeq ($(nullstring),)
	echo "nullstring doesn't even have spaces"
endif
```

- Check if a variable is defined
```makefile
ifdef does not expand variable references; it just sees if something is defined at all
bar =
foo = $(bar)

all:
ifdef foo
	echo "foo is defined"
endif
ifndef bar
	echo "but bar is not"
endif
```

### Functions: 

Call functions with `$(fn, arguments)` or `${fn, arguments}`. 
String Substitution `$(patsubst pattern,replacement,text)` and `$(foreach var,list,text)`. [link](https://makefiletutorial.com/#string-substitution)

### The vpath Directive
Use vpath to specify where some set of prerequisites exist. The format is vpath <pattern> <directories, space/colon separated> <pattern> can have a %, which matches any zero or more characters. You can also do this globallyish with the variable VPATH

```makefile
vpath %.h ../headers ../other-directory

# Note: vpath allows blah.h to be found even though blah.h is never in the current directory
some_binary: ../headers blah.h
	touch some_binary

../headers:
	mkdir ../headers

# We call the target blah.h instead of ../headers/blah.h, because that's the prereq that some_binary is looking for
# Typically, blah.h would already exist and you wouldn't need this.
blah.h:
	touch ../headers/blah.h

clean:
	rm -rf ../headers
	rm -f some_binary
```


### .phony
Adding `.PHONY` to a target will prevent Make from confusing the phony target with a file name. In this example, if the file clean is created, make clean will still be run. Technically, I should have used it in every example with all or clean, but I wanted to keep the examples clean. Additionally, "phony" targets typically have names that are rarely file names, and in practice many people skip this.

```makefile
some_file:
	touch some_file
	touch clean

.PHONY: clean
clean:
	rm -f some_file
	rm -f clean
```

Final cookbook: 

```makefile
# Thanks to Job Vranish (https://spin.atomicobject.com/2016/08/26/makefile-c-projects/)
TARGET_EXEC := final_program

BUILD_DIR := ./build
SRC_DIRS := ./src

# Find all the C and C++ files we want to compile
# Note the single quotes around the * expressions. The shell will incorrectly expand these otherwise, but we want to send the * directly to the find command.
SRCS := $(shell find $(SRC_DIRS) -name '*.cpp' -or -name '*.c' -or -name '*.s')

# Prepends BUILD_DIR and appends .o to every src file
# As an example, ./your_dir/hello.cpp turns into ./build/./your_dir/hello.cpp.o
OBJS := $(SRCS:%=$(BUILD_DIR)/%.o)

# String substitution (suffix version without %).
# As an example, ./build/hello.cpp.o turns into ./build/hello.cpp.d
DEPS := $(OBJS:.o=.d)

# Every folder in ./src will need to be passed to GCC so that it can find header files
INC_DIRS := $(shell find $(SRC_DIRS) -type d)
# Add a prefix to INC_DIRS. So moduleA would become -ImoduleA. GCC understands this -I flag
INC_FLAGS := $(addprefix -I,$(INC_DIRS))

# The -MMD and -MP flags together generate Makefiles for us!
# These files will have .d instead of .o as the output.
# WHAT IS THIS??????
CPPFLAGS := $(INC_FLAGS) -MMD -MP

# The final build step.
$(BUILD_DIR)/$(TARGET_EXEC): $(OBJS)
	$(CXX) $(OBJS) -o $@ $(LDFLAGS)

# Build step for C source
$(BUILD_DIR)/%.c.o: %.c
	mkdir -p $(dir $@)
	$(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@

# Build step for C++ source
$(BUILD_DIR)/%.cpp.o: %.cpp
	mkdir -p $(dir $@)
	$(CXX) $(CPPFLAGS) $(CXXFLAGS) -c $< -o $@


.PHONY: clean
clean:
	rm -r $(BUILD_DIR)

# Include the .d makefiles. The - at the front suppresses the errors of missing
# Makefiles. Initially, all the .d files will be missing, and we don't want those
# errors to show up.
-include $(DEPS)
```