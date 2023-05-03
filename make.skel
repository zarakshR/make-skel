# r - diables built-in rules
# R - diables built-in variables
MAKEFLAGS += -rR

# Make all targets depend on this Makefile
.EXTRA_PREREQS:= $(abspath $(lastword $(MAKEFILE_LIST)))

# Program name
P=

INCLUDE_DIR=inc
OBJECT_DIR=obj
BUILD_DIR=build
SOURCE_DIR=src

CC=gcc
CFLAGS=-Wall -Wextra -Wpedantic -I$(INCLUDE_DIR) -I$(OBJECT_DIR) # OBJECT_DIR contains the precompiled headers
LDFLAGS=
DBGFLAGS=-fsanitize=address,undefined,integer-divide-by-zero -fno-omit-frame-pointer

LDLIBS=

HEADERS=algos sorts array shared # Header files
UNITS=algos sorts array # Compilation units

# Generate lists of headers, sources, objects, and object-with-debugging-symbols
HDRS=$(patsubst %, $(OBJECT_DIR)/%.h.gch, $(HEADERS))
SRCS=$(patsubst %,$(SOURCE_DIR)/%.c, $(UNITS))
OBJS=$(patsubst %, $(OBJECT_DIR)/%.o, $(UNITS))
OBJS_DEBUG=$(patsubst %,$(OBJECT_DIR)/%.o.debug, $(UNITS))

# build main binary by default
.PHONY: default
default: $(BUILD_DIR)/$(P)

# build main binary, main binary with debug symbols, and main binary built for valgrind
.PHONY: all
all: $(BUILD_DIR)/$(P) $(BUILD_DIR)/$(P)_debug $(BUILD_DIR)/$(P)_grind

# Build pre-compiled header file
.SECONDARY: $(OBJECT_DIR)/%.gch
$(OBJECT_DIR)/%.gch: $(INCLUDE_DIR)/%
	@echo Building pre-compiled header: $@
	@mkdir -p $(OBJECT_DIR)
	$(CC) $(CFLAGS) $(LDFLAGS) $< -o $@

# Build object file
.SECONDARY: $(OBJECT_DIR)/%.o
$(OBJECT_DIR)/%.o: $(SOURCE_DIR)/%.c $(DEPS)
	@echo Building object: $@
	@mkdir -p $(OBJECT_DIR)
	$(CC) $(CFLAGS) $(LDFLAGS) $< -c -o $@

# Build object file with debugging symbols
.SECONDARY: $(OBJECT_DIR)/%.o.debug
$(OBJECT_DIR)/%.o.debug: LDFLAGS+=-g
$(OBJECT_DIR)/%.o.debug: $(SOURCE_DIR)/%.c $(DEPS)
	@echo Building object with debug symbols: $@
	@mkdir -p $(OBJECT_DIR)
	$(CC) $(CFLAGS) $(LDFLAGS) $< -c -o $@

# Build main binary
$(BUILD_DIR)/$(P): $(OBJS)
	@echo Building $@
	@mkdir -p $(BUILD_DIR)
	$(CC) $(CFLAGS) $(LDFLAGS) $^ -O3 -o $@

# Build binary with debugging symbols. Uses -g and ASan. Cannot be used for valgrind
$(BUILD_DIR)/$(P)_debug: LDFLAGS+=-g
$(BUILD_DIR)/$(P)_debug: $(OBJS_DEBUG)
	@echo Building $@
	@mkdir -p $(BUILD_DIR)
	$(CC) $(CFLAGS) $(LDFLAGS) $(DBGFLAGS) $^ -o $@

# Build binary with debugging symbols. Uses -g only. Cannot be used for valgrind
$(BUILD_DIR)/$(P)_grind: LDFLAGS+=-g
$(BUILD_DIR)/$(P)_grind: $(OBJS_DEBUG)
	@echo Building $@
	@mkdir -p $(BUILD_DIR)
	$(CC) $(CFLAGS) $(LDFLAGS) $^ -o $@

# Run program
.PHONY: run
run: $(BUILD_DIR)/$(P)
	$^

# Start gdb on debug build
.PHONY: debug
debug: $(BUILD_DIR)/$(P)_debug
	gdb $^

# Run valgrind on grind build
.PHONY: grind
grind: $(BUILD_DIR)/$(P)_grind
	valgrind --track-origins=yes --leak-check=full --leak-resolution=high $^

# Run linter
.PHONY: lint
lint: $(SRCS)
	clang-tidy\
	 --quiet\
	 --checks=*,\
	-llvmlibc-restrict-system-libc-headers,\
	-altera-id-dependent-backward-branch,\
	-modernize-macro-to-enum,\
	-altera-unroll-loops,\
	-llvm-include-order,\
	-bugprone-reserved-identifier,\
	-cert-dcl37-c,\
	-cert-dcl51-cpp,\
	-misc-no-recursion,\
	-google-readability-todo\
	 $^\
	 -- -I$(INCLUDE_DIR) # Have to include this or clang will complain about not being able to find header files. See https://stackoverflow.com/a/56457021/15446749

# Format all sources
.PHONY: format
format: $(SRCS)
	clang-format --fallback-style=LLVM -i $(SRCS)

.PHONY: clean
clean:
	rm -f $(OBJECT_DIR)/*.o
	rm -f $(OBJECT_DIR)/*.o.debug
	rm -f $(OBJECT_DIR)/*.h.gch
	rm -f $(BUILD_DIR)/$(P){,_debug,_grind}
	rmdir obj
	rmdir build
