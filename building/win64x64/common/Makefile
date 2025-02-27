#############################################################################
# Common Makefile for Win32 VM using gcc, cygwin and gnu make
# Do make init to allow make -n to function.
#############################################################################

#############################################################################
# Parameters:
# VM the name of the exe to build, defaults to Squeak (=> Squeak.exe)
# VM_NAME the name the VM will print when reporting, defaults to $(VM)
# VMSRCDIR the directory containing the interpreter, optional cogit and interp.h
# CONFIGURATION configuration of VM to build from product, assert & debug
# THREADING whether to build a multi-threaded FFI VM
# COGDEFS supply any command-line defines to use, and may be null.

.PHONY: all
all: default

VM?=Squeak
VM_NAME?=$(VM)
CONFIGURATION=product
THREADING=single
export VM VM_NAME CONFIGURATION COGDEFS

# Is this a Croquet VM (defaults to OGL instead of D3D)?
CROQUET:=-DCROQUET

#############################################################################
# Default locations
#

ifeq ($(THREADING),multi)
	ifeq ($(CONFIGURATION),product)
		BUILD:=buildmt
	else ifeq ($(CONFIGURATION),assert)
		BUILD:=buildmtast
	else
		BUILD:=buildmtdbg
	endif
else
	ifeq ($(CONFIGURATION),product)
		BUILD:=build
	else ifeq ($(CONFIGURATION),assert)
		BUILD:=buildast
	else
		BUILD:=builddbg
	endif
endif
$(shell mkdir -p deps >/dev/null) # deps is the dependencies directory
PLUGINSRCDIR:= ../../../src
export PLUGINSRCDIR
OBJDIR:= $(BUILD)/vm

PLATDIR:=../../../platforms
export PLATDIR
CROSSDIR:=$(PLATDIR)/Cross/vm
TPDIR:=$(PLATDIR)/Cross/third-party
WIN32DIR:=$(PLATDIR)/win32/vm
WIN32PLUGINSDIR:=$(PLATDIR)/win32/plugins
WIN32MISCDIR:=$(PLATDIR)/win32/misc
WIN32UTILDIR:=$(PLATDIR)/win32/util

CROSSSRC:= $(wildcard $(CROSSDIR)/*.c) $(wildcard $(CROSSDIR)/*.cpp)
WIN32SRC:= $(wildcard $(WIN32DIR)/*.c) $(wildcard $(WIN32DIR)/*.cpp)
ifeq ($(THREADING),multi)
MAKERSRC:= $(wildcard $(VMSRCDIR)/gcc3x-*interpmt.c $(VMSRCDIR)/cogit.c)
else
MAKERSRC:= $(wildcard $(VMSRCDIR)/gcc3x-*interp.c $(VMSRCDIR)/cogit.c)
endif
VMSRC:= $(notdir $(MAKERSRC) $(WIN32SRC) $(CROSSSRC))

VPATH:= $(VMSRCDIR) $(WIN32DIR) $(CROSSDIR)

#############################################################################
# The internal (.lib) and external (.dll) plugins
#
include plugins.ext
include plugins.int

LIBS:= $(addprefix $(OBJDIR)/, $(addsuffix .lib, $(INTERNAL_PLUGINS)))
DLLS:= $(addprefix $(OBJDIR)/, $(addsuffix .dll, $(EXTERNAL_PLUGINS)))

ifdef BIT_IDENTICAL_FLOATING_POINT
LIBS:=$(BUILD)/fdlibm/libm.a $(LIBS)
export BIT_IDENTICAL_FLOATING_POINT
endif

#############################################################################
# The third-party libraries
#
ifneq ($(THIRDPARTYLIBS),)
include ../common/Makefile.lib.extra
THIRDPARTYPREREQS:=$(THIRDPARTYCACHEDIR) $(THIRDPARTYOUTDIR) 
endif

#############################################################################
# Generic VM source file definitions
#
VMOBJ:=	$(VMSRC:.c=.o)
VMOBJ:= $(addprefix $(OBJDIR)/,$(VMOBJ))

#############################################################################
# SqueakVM definitions
#
ifeq ($(CONFIGURATION),product)
VMEXE:=  $(OBJDIR)/$(VM)Unstripped.exe
CONSOLEVMEXE:=  $(OBJDIR)/$(VM)ConsoleUnstripped.exe
STRIPEXE:=  $(OBJDIR)/$(VM).exe
STRIPCONSOLEEXE:=  $(OBJDIR)/$(VM)Console.exe
else
VMEXE:=  $(OBJDIR)/$(VM).exe
CONSOLEVMEXE:=  $(OBJDIR)/$(VM)Console.exe
STRIPEXE:=
STRIPCONSOLEEXE:=
endif
VMDEF:=	$(VM).def
VMEXP:=	$(OBJDIR)/$(VM).exp
VMMAP:=	$(OBJDIR)/$(VM).map
CONSOLEVMMAP:=	$(OBJDIR)/$(VM)Console.map
VMLIB:=	$(VM).lib
VMRES:=	$(VM).res
VMDEFIN:=$(VM).def.in
BTOBJ:= $(OBJDIR)/btext.o
ETOBJ:= $(OBJDIR)/etext.o
JMPASM:=_setjmp-x64.asm
JMPOBJ:=$(OBJDIR)/_setjmp-x64.o

.PRECIOUS: mkNamedPrims.exe

#############################################################################
# Toolchain
#
include ../common/Makefile.tools

INCLUDES:= -I. -I$(VMSRCDIR) -I$(WIN32DIR) -I$(CROSSDIR) $(XINC)

.SUFFIXES:
.SUFFIXES:	.ccg .cc .c .o .s .i .rc .res .cg .hg .ccg .cpp

#############################################################################
# Common build rules
#

default: print-settings init $(VMEXE) $(CONSOLEVMEXE) $(DLLS) $(STRIPEXE) $(STRIPCONSOLEEXE) nukelibs $(APPPOST)

svnver:
	echo $(RC) $(RCFLAGS)

productmt:;
	$(MAKE) -f $(MAKEFILE_LIST) CONFIGURATION=product THREADING=multi $(@,product=) default

product:;
	$(MAKE) -f $(MAKEFILE_LIST) CONFIGURATION=product THREADING=single $(@,product=) default

assertmt:
	$(MAKE) -f $(MAKEFILE_LIST) CONFIGURATION=assert THREADING=multi $(@,assert=) default

assert:
	$(MAKE) -f $(MAKEFILE_LIST) CONFIGURATION=assert THREADING=single $(@,assert=) default

debugmt:
	$(MAKE) -f $(MAKEFILE_LIST) CONFIGURATION=debug THREADING=multi $(@,debug=) default

debug:
	$(MAKE) -f $(MAKEFILE_LIST) CONFIGURATION=debug THREADING=single $(@,debug=) default

# Do make init to allow make -n to function.
init: $(THIRDPARTYPREREQS) $(OBJDIR) mkNamedPrims.exe

cleanall:	clean cleanmt cleanast cleanmtast cleandbg cleanmtdbg

clean:
	rm -rf sqNamedPrims.h mkNamedPrims.exe build

cleanmt:
	rm -rf sqNamedPrims.h mkNamedPrims.exe buildmt

cleanast:
	rm -rf sqNamedPrims.h mkNamedPrims.exe buildast

cleanmtast:
	rm -rf sqNamedPrims.h mkNamedPrims.exe buildmtast

cleandbg:
	rm -rf sqNamedPrims.h mkNamedPrims.exe builddbg

cleanmtdbg:
	rm -rf sqNamedPrims.h mkNamedPrims.exe buildmtdbg

nukelibs:
	rm $(LIBS)

.PHONY : print-settings

print-settings:
	$(info ---------------- Makefile settings ------------------)
	$(info CONFIGURATION=$(CONFIGURATION))
	$(info THREADING=$(THREADING))
	$(info VPATH=$(VPATH))
	$(info INCLUDES=$(INCLUDES))
	$(info CFLAGS=$(CFLAGS))
	$(info INTERNAL_PLUGINS=$(INTERNAL_PLUGINS))
	$(info EXTERNAL_PLUGINS=$(EXTERNAL_PLUGINS))
	$(info OBJDIR=$(OBJDIR))
	$(info LIBS=$(LIBS))
	$(info DLLS=$(DLLS))
	$(info -----------------------------------------------------)

.PHONY : print-objects

print-objects:
	$(info ---------------- Makefile objects ------------------)
	$(info VMOBJ=$(VMOBJ))
	$(info -----------------------------------------------------)

ignore := $(addsuffix .%, $(basename $(wildcard $(BUILD)/vm/*.ignore)))

mingw32ver.exe: $(WIN32UTILDIR)/mingw32ver.c
	$(CC) -o $@ -O1 -mconsole -m64 $<

mkNamedPrims.exe: $(WIN32UTILDIR)/mkNamedPrims.c
	$(CC) -o $@ -mconsole -m64 $<

$(BTOBJ):	$(WIN32MISCDIR)/btext.c
	$(CC) -c -o $@ -fomit-frame-pointer -O2 $<

$(ETOBJ):	$(WIN32MISCDIR)/etext.c
	$(CC) -c -o $@ -fomit-frame-pointer -O2 $<

$(JMPOBJ):	$(WIN32MISCDIR)/$(JMPASM)
	$(CC) -c $< -o $@

vm:	$(VMEXE)

$(VMEXE): $(OBJDIR) $(THIRDPARTYLIBS) $(VMOBJ) $(LIBS) $(VMEXP) resource.o $(BTOBJ) $(ETOBJ) $(JMPOBJ)
	$(CC) -o $(OBJDIR)/version.o $(CFLAGS) $(INCLUDES) $(DEFS) -c $(WIN32DIR)/version.c
	$(LD) $(LDFLAGS) -o $(VMEXE) \
			$(BTOBJ) $(VMOBJ) $(VMEXP) $(JMPOBJ) $(OBJDIR)/resource.o $(filter-out $(call ignore), $(LIBS)) $(STDLIBS) $(ETOBJ)
	$(NM) --numeric-sort --defined-only -f bsd $(VMEXE) >$(VMMAP)

$(CONSOLEVMEXE): $(VMOBJ) $(LIBS) $(VMEXP) resource.o $(BTOBJ) $(ETOBJ) $(JMPOBJ)
	$(CC) -o $(OBJDIR)/version.o $(CFLAGS) $(INCLUDES) $(DEFS) -c $(WIN32DIR)/version.c
	$(LD) $(CONSOLELDFLAGS) -o $(CONSOLEVMEXE) \
			$(BTOBJ) $(VMOBJ) $(VMEXP) $(JMPOBJ) $(OBJDIR)/resource.o $(filter-out $(call ignore), $(LIBS)) $(STDLIBS) $(ETOBJ)
	$(NM) --numeric-sort --defined-only -f bsd $(CONSOLEVMEXE) >$(CONSOLEVMMAP)

ifneq ($STRIPEXE,)
$(STRIPEXE): $(VMEXE)
	$(STRIP) --strip-unneeded -o $(STRIPEXE) $(VMEXE)
	$(OBJCOPY) --add-gnu-debuglink=$(VMEXE) $(STRIPEXE)

$(STRIPCONSOLEEXE): $(CONSOLEVMEXE)
	$(STRIP) --strip-unneeded -o $(STRIPCONSOLEEXE) $(CONSOLEVMEXE)
	$(OBJCOPY) --add-gnu-debuglink=$(CONSOLEVMEXE) $(STRIPCONSOLEEXE)
endif

$(OBJDIR):
	@-mkdir $(BUILD)
	mkdir $(OBJDIR)
ifneq ($(INTERNAL_PLUGINS),)
	mkdir $(addprefix $(BUILD)/, $(INTERNAL_PLUGINS))
endif
ifneq ($(EXTERNAL_PLUGINS),)
	mkdir $(addprefix $(BUILD)/, $(EXTERNAL_PLUGINS))
endif

#############################################################################
# The exports for named primitives from Squeak (required by VM)
#
$(VMDEF) $(VMEXP) $(VMLIB): $(VMOBJ)
	$(DLLTOOL) --input-def $(VMDEFIN) --output-def $(OBJDIR)/$(VMDEF) --output-exp $(VMEXP) --output-lib $(OBJDIR)/$(VMLIB) $(VMOBJ)

#############################################################################
# Building plugins
#

.PHONY: $(OBJDIR)/%.lib $(OBJDIR)/%.dll

# Check for Makefile in win32 plugins directory otherwise use default Makefile
plugin-makefile = $(realpath $(firstword $(wildcard $(WIN32PLUGINSDIR)/$(1)/Makefile.plugin ../common/Makefile.plugin)))

# Internal plugin.  Build as lib then link in lib
$(OBJDIR)/%.lib: $(call plugin-makefile,$(*F))
	@-mkdir -p $(BUILD)/$(*F)
	rm -f $(BUILD)/vm/$(*F).ignore
	$(MAKE) $(MFLAGS) BUILD=$(BUILD) \
		-f $(call plugin-makefile,$(*F)) \
		LIBNAME=$(*F) INTERNAL_PLUGIN=1 VMSRCDIR=$(VMSRCDIR) \
		COGDEFS="$(COGDEFS) -DSQUEAK_BUILTIN_PLUGIN=$(*F)" $(OBJDIR)/$(*F).lib

# External plugin.  Build as dll and copy to vm dir ($(OBJDIR)).
$(OBJDIR)/%.dll: $(call plugin-makefile,$(*F))
	@-mkdir -p $(BUILD)/$(*F)
	rm -f $(BUILD)/$(*F).ignore
	$(MAKE) $(MFLAGS) BUILD=$(BUILD) \
		-f $(call plugin-makefile,$(*F)) \
		LIBNAME=$(*F) EXTERNAL_PLUGIN=1 VMSRCDIR=$(VMSRCDIR) \
		JMPOBJ=$(JMPOBJ) VMLIB=$(OBJDIR)/$(VMLIB) \
		COGDEFS="$(COGDEFS) -DSQUEAK_EXTERNAL_PLUGIN=$(*F)" $(OBJDIR)/$(*F).dll

$(BUILD)/fdlibm/libm.a:
	@-mkdir -p $(@D)
	$(MAKE) CC='$(CC)' CFLAGS="$(CFLAGS) -D_IEEE_LIBM" -C $(@D) \
		TP=../../$(TPDIR) -f ../../$(TPDIR)/fdlibm/Makefile.remote


#############################################################################
# Basic rules
#
include ../common/Makefile.rules

$(VMRES): $(VM).rc
	$(RC) $(RCFLAGS) -i $(VM).rc -o $(OBJDIR)/$@
	$(CP) $(VM).exe.manifest $(OBJDIR)
	$(CP) $(VM).exe.manifest $(OBJDIR)/$(VM)Console.exe.manifest

resource.o:	$(VMRES)
	$(RC) $(RCFLAGS) -i $(OBJDIR)/$< -o $(OBJDIR)/$@

#############################################################################
# Extra specific dependencies
#
sqNamedPrims.h: plugins.int mkNamedPrims.exe
	./mkNamedPrims.exe $(INTERNAL_PLUGINS) > sqNamedPrims.h

$(OBJDIR)/sqNamedPrims.o:	sqNamedPrims.c sqNamedPrims.h
