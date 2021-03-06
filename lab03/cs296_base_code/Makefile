.SUFFIXES: .cpp .hpp

# Programs
SHELL 	= bash
CC     	= g++
LD	= ld
RM 	= rm
ECHO	= /bin/echo
CAT	= cat
PRINTF	= printf
SED	= sed
DOXYGEN = doxygen
######################################
# Project Name (generate executable with this name)
TARGET = cs296_13_exe

# Project Paths
PROJECT_ROOT :=$(CURDIR)
EXTERNAL_ROOT=$(PROJECT_ROOT)/external
SRCDIR = $(PROJECT_ROOT)/src
OBJDIR = $(PROJECT_ROOT)/myobjs
BINDIR = $(PROJECT_ROOT)/mybins
LIBDIR= $(PROJECT_ROOT)/mylibs
DOCDIR = $(PROJECT_ROOT)/doc

# Library Paths
BOX2D_ROOT=$(EXTERNAL_ROOT)
GLUI_ROOT=/usr
GL_ROOT=/usr/include/

#Libraries
LIBS = -lBox2D -lglui -lglut -lGLU -lGL
STATLIBTARGET=./mylibs/libCS296test.a
DYNLIBTARGET=./mylibs/libCS296test.so

# Compiler and Linker flags
CPPFLAGS =-g -O3 -Wall -fno-strict-aliasing -fPIC
CPPFLAGS+=-I $(BOX2D_ROOT)/include -I $(GLUI_ROOT)/include
LDFLAGS+=-L $(BOX2D_ROOT)/lib -L $(GLUI_ROOT)/lib

######################################

NO_COLOR=\e[0m
OK_COLOR=\e[1;32m
ERR_COLOR=\e[1;31m
WARN_COLOR=\e[1;33m
MESG_COLOR=\e[1;34m
FILE_COLOR=\e[1;37m

OK_STRING="[OK]"
ERR_STRING="[ERRORS]"
WARN_STRING="[WARNINGS]"
OK_FMT="${OK_COLOR}%30s\n${NO_COLOR}"
ERR_FMT="${ERR_COLOR}%30s\n${NO_COLOR}"
WARN_FMT="${WARN_COLOR}%30s\n${NO_COLOR}"
######################################

SRCS := $(wildcard $(SRCDIR)/*.cpp)
INCS := $(wildcard $(SRCDIR)/*.hpp)
OBJS := $(SRCS:$(SRCDIR)/%.cpp=$(OBJDIR)/%.o)
CORLIB := $(filter-out $(OBJDIR)/main.o,$(OBJS))

######################################
EXTINC=$(EXTERNAL_ROOT)/include
EXTLIB=$(EXTERNAL_ROOT)/lib
EXTSRC=$(EXTERNAL_ROOT)/src
SHARED_LIB=TRUE

.PHONY: all setup clean distclean doc report

all: setup exe $(STATLIBTARGET) $(DYNLIBTARGET) exelib 

setup:
	@$(ECHO) "Setting up compilation..."
	@mkdir -p myobjs
	@mkdir -p mylibs
	@mkdir -p mybins
	@if test -e ./external/lib/libBox2D.a ; then \
	$(ECHO) "Box 2D already installed."; \
	else \
	cd "$(EXTSRC)"; \
	tar xvzf Box2D.tgz; \
	cd Box2D; \
	mkdir -p build296; \
	cd ./build296; \
	cmake ../ ; \
	make ; \
	make install; \
	fi;
exe: $(OBJS)
	@$(PRINTF) "$(MESG_COLOR)Building executable:$(NO_COLOR) $(FILE_COLOR) %16s$(NO_COLOR)" "$(notdir $(BINDIR)/$(TARGET))"
	@$(CC) -o $(BINDIR)/$(TARGET) $(LDFLAGS) $(OBJS) $(LIBS) 2> temp.log || touch temp.err
	@if test -e temp.err; \
	then $(PRINTF) $(ERR_FMT) $(ERR_STRING) && $(CAT) temp.log; \
	elif test -s temp.log; \
	then $(PRINTF) $(WARN_FMT) $(WARN_STRING) && $(CAT) temp.log; \
	else $(PRINTF) $(OK_FMT) $(OK_STRING); \
	fi;
	@$(RM) -f temp.log temp.err

-include $(OBJS:.o=.d)

$(OBJS): $(OBJDIR)/%.o : $(SRCDIR)/%.cpp
	@$(PRINTF) "$(MESG_COLOR)Compiling: $(NO_COLOR) $(FILE_COLOR) %25s$(NO_COLOR)" "$(notdir $<)"
	@$(CC) $(CPPFLAGS) -c $< -o $@ -MD 2> temp.log || touch temp.err
	@if test -e temp.err; \
	then $(PRINTF) $(ERR_FMT) $(ERR_STRING) && $(CAT) temp.log; \
	elif test -s temp.log; \
	then $(PRINTF) $(WARN_FMT) $(WARN_STRING) && $(CAT) temp.log; \
	else printf "${OK_COLOR}%30s\n${NO_COLOR}" "[OK]"; \
	fi;
	@$(RM) -f temp.log temp.err


$(STATLIBTARGET): $(CORLIB)
	@if test "$(SHARED_LIB)" = "FALSE"; \
	then ar -cvq $@ $(CORLIB);\
	fi;

$(DYNLIBTARGET): $(CORLIB)
	@if test "$(SHARED_LIB)" = "TRUE"; \
	then $(CC) -shared -Wl,-soname,$(CURDIR)/mylibs/libCS296test.so -o $(CURDIR)/mylibs/libCS296test.so $(CORLIB) -lc; \
	fi;

exelib: $(DYNLIBTARGET) $(STATLIBTARGET)
	@if test "$(SHARED_LIB)" = "TRUE"; \
	then $(CC) -o ./mybins/cs296_13_exelib $(LDFLAGS) -L ./external/lib/ ./myobjs/main.o ./mylibs/libCS296test.so $(LIBS); \
	else $(CC) -o ./mybins/cs296_13_exelib $(LDFLAGS) -L ./mylibs/ -L ./external/lib/ ./myobjs/main.o -lCS296test $(LIBS); \
	fi;
clean:
	@$(ECHO) -n "Cleaning up..."
	@$(RM) -rf $(OBJDIR) $(LIBDIR) $(BINDIR) *~ $(DEPS) $(SRCDIR)/*~ $(PROJECT_ROOT)/doc/html $(PROJECT_ROOT)/doc/*.pdf $(PROJECT_ROOT)/doc/*.dvi $(PROJECT_ROOT)/doc/*.aux $(PROJECT_ROOT)/doc/*.log $(PROJECT_ROOT)/doc/*.bbl $(PROJECT_ROOT)/doc/*.blg 
	@$(ECHO) "Done"

distclean: clean
	@$(ECHO) -n "Cleaning up..."
	@$(RM) -rf ./external/include/*
	@$(RM) -rf ./external/lib/*
	@$(RM) -rf ./external/src/Box2D	
	@$(ECHO) "Done"	

doc:
	@$(ECHO) -n "Generating Doxygen Documentation ...  "
	@$(RM) -rf doc/html
	@$(DOXYGEN) $(DOCDIR)/Doxyfile 2 > /dev/null
	@$(ECHO) "Done"

report:
	@$(ECHO) -n "Generating Report ...  "
	@cd ./doc; \
	latex cs296_report_13; \
	bibtex cs296_report_13; \
	latex cs296_report_13; \
	latex cs296_report_13; \
	dvipdf cs296_report_13.dvi cs296_report_13.pdf; \

