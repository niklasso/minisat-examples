###################################################################################################

.PHONY:	r d p sh cr cd cp csh lr ld lp lsh config all install install-bin clean distclean
all:	r

## Load Previous Configuration ####################################################################

-include config.mk

## Configurable options ###########################################################################

# Directory to store object files, libraries, executables, and dependencies:
BUILD_DIR      ?= build

# Include debug-symbols in release builds
MINUMERATE_RELSYM ?= -g

# Sets of compile flags for different build types
MINUMERATE_REL    ?= -O3 -D NDEBUG
MINUMERATE_DEB    ?= -O0 -D DEBUG 
MINUMERATE_PRF    ?= -O3 -D NDEBUG
MINUMERATE_FPIC   ?= -fpic

# Dependencies
MINISAT_INCLUDE?=
MINISAT_LIB    ?=-lminisat

# GNU Standard Install Prefix
prefix         ?= /usr/local

## Write Configuration  ###########################################################################

config:
	@( echo 'BUILD_DIR?=$(BUILD_DIR)'                 ; \
	   echo 'MINUMERATE_RELSYM?=$(MINUMERATE_RELSYM)' ; \
	   echo 'MINUMERATE_REL?=$(MINUMERATE_REL)'       ; \
	   echo 'MINUMERATE_DEB?=$(MINUMERATE_DEB)'       ; \
	   echo 'MINUMERATE_PRF?=$(MINUMERATE_PRF)'       ; \
	   echo 'MINUMERATE_FPIC?=$(MINUMERATE_FPIC)'     ; \
	   echo 'MINISAT_INCLUDE?=$(MINISAT_INCLUDE)'     ; \
	   echo 'MINISAT_LIB?=$(MINISAT_LIB)'	          ; \
	   echo 'prefix?=$(prefix)'                       ) > config.mk

## Configurable options end #######################################################################

INSTALL ?= install

# GNU Standard Install Variables
exec_prefix ?= $(prefix)
includedir  ?= $(prefix)/include
bindir      ?= $(exec_prefix)/bin
libdir      ?= $(exec_prefix)/lib
datarootdir ?= $(prefix)/share
mandir      ?= $(datarootdir)/man

# Target file names
MINUMERATE      = minumerate#       Name of main executable.

SRCS            = Main.cc Minumerate.cc

# Compile flags
MINUMERATE_CXXFLAGS = -I. -D __STDC_LIMIT_MACROS -D __STDC_FORMAT_MACROS -Wall -Wno-parentheses -Wextra $(MINISAT_INCLUDE)
MINUMERATE_LDFLAGS  = -Wall -lz $(MINISAT_LIB)

ifeq ($(VERB),)
ECHO=@
VERB=@
else
ECHO=#
VERB=
endif

r:	$(BUILD_DIR)/release/bin/$(MINUMERATE)
d:	$(BUILD_DIR)/debug/bin/$(MINUMERATE)
p:	$(BUILD_DIR)/profile/bin/$(MINUMERATE)
sh:	$(BUILD_DIR)/dynamic/bin/$(MINUMERATE)

## Build-type Compile-flags:
$(BUILD_DIR)/release/%.o:			MINUMERATE_CXXFLAGS +=$(MINUMERATE_REL) $(MINUMERATE_RELSYM)
$(BUILD_DIR)/debug/%.o:				MINUMERATE_CXXFLAGS +=$(MINUMERATE_DEB) -g
$(BUILD_DIR)/profile/%.o:			MINUMERATE_CXXFLAGS +=$(MINUMERATE_PRF) -pg
$(BUILD_DIR)/dynamic/%.o:			MINUMERATE_CXXFLAGS +=$(MINUMERATE_REL) $(MINUMERATE_FPIC)

## Build-type Link-flags:
$(BUILD_DIR)/profile/bin/$(MINUMERATE):		MINUMERATE_LDFLAGS += -pg
$(BUILD_DIR)/release/bin/$(MINUMERATE):		MINUMERATE_LDFLAGS += --static $(MINUMERATE_RELSYM)
$(BUILD_DIR)/profile/bin/$(MINUMERATE_CORE):	MINUMERATE_LDFLAGS += -pg
$(BUILD_DIR)/release/bin/$(MINUMERATE_CORE):	MINUMERATE_LDFLAGS += --static $(MINUMERATE_RELSYM)

## Executable dependencies
$(BUILD_DIR)/release/bin/$(MINUMERATE):	 	$(foreach o, $(SRCS:.cc=.o), $(BUILD_DIR)/release/$o)
$(BUILD_DIR)/debug/bin/$(MINUMERATE):	 	$(foreach o, $(SRCS:.cc=.o), $(BUILD_DIR)/debug/$o)
$(BUILD_DIR)/profile/bin/$(MINUMERATE):	 	$(foreach o, $(SRCS:.cc=.o), $(BUILD_DIR)/profile/$o)
$(BUILD_DIR)/dynamic/bin/$(MINUMERATE):	 	$(foreach o, $(SRCS:.cc=.o), $(BUILD_DIR)/dynamic/$o)

## Compile rules (these should be unified, buit I have not yet found a way which works in GNU Make)
$(BUILD_DIR)/release/%.o:	%.cc
	$(ECHO) echo Compiling: $@
	$(VERB) mkdir -p $(dir $@) $(dir $(BUILD_DIR)/dep/$*.d)
	$(VERB) $(CXX) $(MINUMERATE_CXXFLAGS) $(CXXFLAGS) -c -o $@ $< -MMD -MF $(BUILD_DIR)/dep/$*.d

$(BUILD_DIR)/profile/%.o:	%.cc
	$(ECHO) echo Compiling: $@
	$(VERB) mkdir -p $(dir $@) $(dir $(BUILD_DIR)/dep/$*.d)
	$(VERB) $(CXX) $(MINUMERATE_CXXFLAGS) $(CXXFLAGS) -c -o $@ $< -MMD -MF $(BUILD_DIR)/dep/$*.d

$(BUILD_DIR)/debug/%.o:	%.cc
	$(ECHO) echo Compiling: $@
	$(VERB) mkdir -p $(dir $@) $(dir $(BUILD_DIR)/dep/$*.d)
	$(VERB) $(CXX) $(MINUMERATE_CXXFLAGS) $(CXXFLAGS) -c -o $@ $< -MMD -MF $(BUILD_DIR)/dep/$*.d

$(BUILD_DIR)/dynamic/%.o:	%.cc
	$(ECHO) echo Compiling: $@
	$(VERB) mkdir -p $(dir $@) $(dir $(BUILD_DIR)/dep/$*.d)
	$(VERB) $(CXX) $(MINUMERATE_CXXFLAGS) $(CXXFLAGS) -c -o $@ $< -MMD -MF $(BUILD_DIR)/dep/$*.d

## Linking rule
$(BUILD_DIR)/release/bin/$(MINUMERATE) $(BUILD_DIR)/debug/bin/$(MINUMERATE) $(BUILD_DIR)/profile/bin/$(MINUMERATE) $(BUILD_DIR)/dynamic/bin/$(MINUMERATE):
	$(ECHO) echo Linking Binary: $@
	$(VERB) mkdir -p $(dir $@)
	$(VERB) $(CXX) $^ $(MINUMERATE_LDFLAGS) $(LDFLAGS) -o $@

install:	install-bin

install-bin: $(BUILD_DIR)/dynamic/bin/$(MINUMERATE)
	$(INSTALL) -d $(DESTDIR)$(bindir)
	$(INSTALL) -m 755 $(BUILD_DIR)/dynamic/bin/$(MINUMERATE) $(DESTDIR)$(bindir)

clean:
	rm -f $(foreach t, release debug profile dynamic, $(foreach o, $(SRCS:.cc=.o), $(BUILD_DIR)/$t/$o)) \
	  $(foreach d, $(SRCS:.cc=.d), $(BUILD_DIR)/dep/$d) \
	  $(foreach t, release debug profile dynamic, $(BUILD_DIR)/$t/bin/$(MINUMERATE))

distclean:	clean
	rm -f config.mk

## Include generated dependencies
## NOTE: dependencies are assumed to be the same in all build modes at the moment!
-include $(foreach s, $(SRCS:.cc=.d), $(BUILD_DIR)/dep/$s)
