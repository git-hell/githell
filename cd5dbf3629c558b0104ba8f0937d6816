##############################################################################
#
#					              ncc, the new c compiler
#
#                   Copyright (c) 2021 Charles E. Youse (charles@gnuless.org).
#                      All rights reserved. See LICENSE file for more details.
#
##############################################################################

# to build, set NCC_HOME to an (otherwise-unoccupied/nonexistent) directory
# where ncc should live, add $NCC_HOME/bin to your PATH, and then:
#
#		make clean; make; make install; make lib
#
# then invoke ncc to compile files, in the usual Unix tradition, e.g.,
#
#		ncc test.c 	(compiles, assembles, links to a.out)
#
# for the moment we're reliant on GNU ar/as/ld, expected to be in /usr/bin.

NCC_HOME=/home/charles/ncc-bin

CC=cc
CFLAGS=-DROOT=\"$(NCC_HOME)\"

all:	cc cpp/cpp cc1/cc1

clean:
	rm -f cc
	rm -f common/*.o
	rm -f cpp/*.o cpp/cpp
	rm -f cc1/*.o cc1/amd64/*.o cc1/cc1
	rm -f *.diff

install:
	rm -rf $(NCC_HOME)
	mkdir -p $(NCC_HOME)
	mkdir -p $(NCC_HOME)/bin
	mkdir -p $(NCC_HOME)/lib

	cp cc $(NCC_HOME)/bin/ncc
	cp cpp/cpp $(NCC_HOME)/bin
	cp cc1/cc1 $(NCC_HOME)/lib/cc1

lib:
	(cd libc; make NCC_HOME=$(NCC_HOME) clean)
	(cd libc; make NCC_HOME=$(NCC_HOME) all)
	(cd libc; make NCC_HOME=$(NCC_HOME) install)

cc:	cc.c
	$(CC) -DROOT=\"$(NCC_HOME)\" \
		-DAS=\"/usr/bin/as\" \
		-DLD=\"/usr/bin/ld\" \
		$(CFLAGS) -o cc cc.c

COMMON_HDRS=	common/escape.h common/slist.h common/tailq.h common/util.h
COMMON_OBJS=	common/escape.o

common/escape.o:	common/escape.c $(COMMON_HDRS)

CPP_HDRS=	cpp/cpp.h cpp/directive.h cpp/evaluate.h cpp/input.h \
		cpp/macro.h cpp/token.h cpp/vstring.h

CPP_OBJS=	cpp/cpp.o cpp/directive.o cpp/evaluate.o cpp/input.o \
		cpp/macro.o cpp/token.o cpp/vstring.o

cpp/cpp.o: 		cpp/cpp.c $(CPP_HDRS) $(COMMON_HDRS)
cpp/directive.o:	cpp/directive.c $(CPP_HDRS) $(COMMON_HDRS)
cpp/evaluate.o:		cpp/evaluate.c $(CPP_HDRS) $(COMMON_HDRS)
cpp/input.o:		cpp/input.c $(CPP_HDRS) $(COMMON_HDRS)
cpp/macro.o:		cpp/macro.c $(CPP_HDRS) $(COMMON_HDRS)
cpp/token.o:		cpp/token.c $(CPP_HDRS) $(COMMON_HDRS)
cpp/vstring.o:		cpp/vstring.c $(CPP_HDRS) $(COMMON_HDRS)

cpp/cpp: $(CPP_OBJS) $(COMMON_OBJS)
	$(CC) $(CFLAGS) -o cpp/cpp $(CPP_OBJS) $(COMMON_OBJS)

AMD64_HDRS=	cc1/amd64/target_block.h cc1/amd64/target_insn.h \
		cc1/amd64/amd64.h cc1/amd64/fuse.h cc1/amd64/gen.h \
		cc1/amd64/insn.h cc1/amd64/peep.h cc1/amd64/reg.h

AMD64_OBJS=	cc1/amd64/target.o \
		cc1/amd64/amd64.o cc1/amd64/fuse.o cc1/amd64/gen.o \
		cc1/amd64/insn.o cc1/amd64/peep.o cc1/amd64/reg.o

CC1_HDRS=	cc1/cc1.h cc1/algebra.h cc1/bitset.h cc1/blks.h cc1/block.h \
		cc1/cast.h cc1/con.h cc1/copy.h cc1/dead.h cc1/dealias.h \
		cc1/decl.h cc1/dom.h cc1/expr.h cc1/field.h cc1/fold.h \
		cc1/gen.h cc1/graph.h cc1/init.h cc1/insn.h cc1/kill.h \
		cc1/lex.h cc1/live.h cc1/load.h cc1/loop.h cc1/opt.h \
		cc1/output.h cc1/prop.h cc1/regs.h cc1/sched.h cc1/sign.h \
		cc1/slvn.h cc1/stmt.h cc1/string.h cc1/switch.h cc1/symbol.h \
		cc1/target.h cc1/testz.h cc1/tree.h cc1/type.h cc1/webs.h \
		cc1/assoc.h cc1/codes.h cc1/set.h cc1/stack.h \
		$(AMD64_HDRS)

CC1_OBJS=	cc1/cc1.o cc1/algebra.o cc1/bitset.o cc1/blks.o cc1/block.o \
		cc1/cast.o cc1/con.o cc1/copy.o cc1/dead.o cc1/dealias.o \
		cc1/decl.o cc1/dom.o cc1/expr.o cc1/field.o cc1/fold.o \
		cc1/gen.o cc1/graph.o cc1/init.o cc1/insn.o cc1/kill.o \
		cc1/lex.o cc1/live.o cc1/load.o cc1/loop.o cc1/opt.o \
		cc1/output.o cc1/prop.o cc1/regs.o cc1/sched.o cc1/sign.o \
		cc1/slvn.o cc1/stmt.o cc1/string.o cc1/switch.o cc1/symbol.o \
		cc1/target.o cc1/testz.o cc1/tree.o cc1/type.o cc1/webs.o

cc1/cc1.o:		cc1/cc1.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/algebra.o:		cc1/algebra.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/bitset.o:		cc1/bitset.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/blks.o:		cc1/blks.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/block.o:		cc1/block.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/cast.o:		cc1/cast.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/con.o:		cc1/con.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/copy.o:		cc1/copy.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/dead.o:		cc1/dead.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/dealias.o:		cc1/dealias.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/decl.o:		cc1/decl.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/dom.o:		cc1/dom.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/expr.o:		cc1/expr.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/field.o:		cc1/field.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/fold.o:		cc1/fold.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/gen.o:		cc1/gen.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/graph.o:		cc1/graph.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/init.o:		cc1/init.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/insn.o:		cc1/insn.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/kill.o:		cc1/kill.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/lex.o:		cc1/lex.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/live.o:		cc1/live.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/load.o:		cc1/load.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/loop.o:		cc1/loop.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/opt.o:		cc1/opt.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/output.o:		cc1/output.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/prop.o:		cc1/prop.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/regs.o:		cc1/regs.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/sched.o:		cc1/sched.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/sign.o:		cc1/sign.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/slvn.o:		cc1/slvn.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/stmt.o:		cc1/stmt.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/string.o:		cc1/string.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/switch.o:		cc1/switch.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/symbol.o:		cc1/symbol.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/target.o:		cc1/target.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/testz.o:		cc1/testz.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/tree.o:		cc1/tree.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/type.o:		cc1/type.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/webs.o:		cc1/webs.c $(CC1_HDRS) $(COMMON_HDRS)

cc1/amd64/amd64.o:   	cc1/amd64/amd64.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/amd64/gen.o:	cc1/amd64/gen.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/amd64/insn.o:   	cc1/amd64/insn.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/amd64/peep.o:   	cc1/amd64/peep.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/amd64/reg.o:	cc1/amd64/reg.c $(CC1_HDRS) $(COMMON_HDRS)
cc1/amd64/target.o:   	cc1/amd64/target.c $(CC1_HDRS) $(COMMON_HDRS)

cc1/cc1: $(CC1_OBJS) $(COMMON_OBJS) $(AMD64_OBJS)
	$(CC) $(CFLAGS) -o cc1/cc1 $(CC1_OBJS) $(COMMON_OBJS) $(AMD64_OBJS)
