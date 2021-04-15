#!/bin/sh

# a simple regression test for ncc.
#
# we build the compiler tools and library three times:
#
# 1. stage1: cross-build ncc with the system compiler (gcc or llvm)
# 2. stage2: build ncc with the ncc built in stage1 with the system compiler
# 3. stage3: build ncc with the ncc built in stage2 with ncc
#
# with each of the three versions of ncc, we build the cpp and cc1 c sources
# to assembly output. finally, we diff the stage1 and stage2 output, creating
# stage1.stage2.diff, and do the same to create stage2.stage3.diff. if both
# files are empty, then there is a high degree of confidence that the compiler
# is working properly.
#
# we save the stage3 output of selected files (SAVED_FILES and LIBC_FUNCS)
# in regress/.  these files are included in the repository and can be used
# to check for ... regressions ... and to compare the quality of code output
# between revisions of the compiler. we try to choose a selection of files
# whose sources are stable and which present varied input to the compiler.

SAVED_FILES="cpp.s directive.s evaluate.s input.s token.s macro.s vstring.s \
	     lex.s expr.s fold.s gen.s stmt.s tree.s regs.s string.s symbol.s \
             bitset.s blks.s block.s copy.s dead.s insn.s loop.s prop.s"

LIBC_FUNCS=
for i in `ls libc/*.c`
do
	LIBC_FUNCS="`basename -s .c $i` $LIBC_FUNCS"
done

ROOT=/tmp/ncc-regress

build_stage()
{
	echo
	echo ..................................................... building $2
	echo
	mkdir -p $ROOT/$2

	for i in clean all install
	do
		make CC=$1 NCC_HOME=$ROOT/$2 $i
	done

	make CC=$ROOT/$2/bin/ncc NCC_HOME=$ROOT/$2 lib

	mkdir -p $ROOT/$2/out
	make clean

	$ROOT/$2/bin/ncc -S cc1/*.c
	mv cc1/*.s $ROOT/$2/out
	$ROOT/$2/bin/ncc -S cpp/*.c
	mv cpp/*.s $ROOT/$2/out

	for i in $LIBC_FUNCS
	do
		$ROOT/$2/bin/ncc -S -n -Ilibc/include libc/$i.c
		mv libc/$i.s $ROOT/$2/out
	done
}

diff_stages()
{
	echo diffing $1 $2
	rm -f $1.$2.diff

	for i in $ROOT/$1/out/*.s
	do
		diff -U 5 $i $ROOT/$2/out/`basename $i` >> $1.$2.diff
	done
}

set -e
rm -rf $ROOT

build_stage cc stage1
build_stage $ROOT/stage1/bin/ncc stage2
build_stage $ROOT/stage2/bin/ncc stage3

set +e

diff_stages stage1 stage2
diff_stages stage2 stage3

for i in $SAVED_FILES
do
	mv $ROOT/stage3/out/$i regress/
done

for i in $LIBC_FUNCS
do
	mv $ROOT/stage3/out/$i.s regress/
done

#rm -rf $ROOT
