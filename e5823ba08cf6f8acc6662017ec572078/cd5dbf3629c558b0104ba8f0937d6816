##############################################################################
#
#                                                        ncc standard library
#
#                   Copyright (c) 2021 Charles E. Youse (charles@gnuless.org).
#                      All rights reserved. See LICENSE file for more details.
#
##############################################################################

CC=$(NCC_HOME)/bin/ncc
CFLAGS=-n -Iinclude
AS=$(CC) -c

default:
	@echo do not invoke this makefile directly.
	@echo 'make lib' from the top level file
	@echo will clean, build, and install libc.

clean:
	rm -f *.o amd64/*.o libc.a

all:	amd64/crt0.o libc.a

install:
	cp amd64/crt0.o libc.a $(NCC_HOME)/lib
	mkdir -p $(NCC_HOME)/include
	cp -r include/* $(NCC_HOME)/include

amd64/crt0.o:	amd64/crt0.s

HDRS=		include/ctype.h include/fcntl.h include/float.h \
		include/limits.h include/math.h include/ncc.h \
		include/stdarg.h include/stddef.h include/stdint.h \
		include/stdio.h include/stdlib.h include/string.h \
		include/termios.h include/time.h include/unistd.h \
		include/sys/ioctl.h include/sys/types.h include/sys/wait.h

OBJS=		amd64/syscall.o amd64/memset.o \
		__ctype.o __dtefg.o __fillbuf.o __flushbuf.o __ieee_val.o \
		__pow10.o atoi.o atol.o brk.o bsearch.o clearerr.o ctime.o \
		exit.o execvp.o execvpe.o fclose.o fflush.o fgetpos.o \
		fgets.o fileno.o fopen.o fprintf.o fputc.o fputs.o fread.o \
		frexp.o fscanf.o fseek.o fsetpos.o ftell.o fwrite.o getenv.o \
		getopt.o gets.o isalnum.o isalpha.o iscntrl.o isdigit.o \
		isgraph.o islower.o isprint.o ispunct.o isspace.o isupper.o \
		isxdigit.o isatty.o malloc.o memchr.o memcmp.o memcpy.o \
		memmove.o modf.o printf.o puts.o qsort.o rand.o remove.o \
		rewind.o scanf.o setbuf.o setvbuf.o sprintf.o sscanf.o \
		stdio.o strcat.o strchr.o strcmp.o strcpy.o strerror.o \
		strftime.o strlen.o strncat.o strncmp.o strncpy.o strrchr.o \
		strtod.o strtof.o strtol.o tcgetattr.o tolower.o toupper.o \
		ungetc.o vfprintf.o vfscanf.o vsprintf.o wait.o

libc.a:		$(OBJS)
	rm -f libc.a
	ar rc libc.a $(OBJS)

amd64/syscall.o:	amd64/syscall.s
amd64/memset.o:		amd64/memset.s
__ctype.o:		__ctype.c $(HDRS)
__fillbuf.o:		__fillbuf.c $(HDRS)
__flushbuf.o:		__flushbuf.c $(HDRS)
__ieee_val.o:		__ieee_val.s
__pow10.o:		__pow10.c $(HDRS)
atoi.o:			atoi.c $(HDRS)
atol.o:			atol.c $(HDRS)
brk.o:			brk.c $(HDRS)
bsearch.o: 		bsearch.c $(HDRS)
clearerr.o:		clearerr.c $(HDRS)
ctime.o:		ctime.c $(HDRS)
exit.o:			exit.c $(HDRS)
execvp.o:		execvp.c $(HDRS)
execvpe.o:		execvpe.c $(HDRS)
fclose.o:		fclose.c $(HDRS)
fflush.o:		fflush.c $(HDRS)
fgetpos.o:		fgetpos.c $(HDRS)
fgets.o:		fgets.c $(HDRS)
fileno.o:		fileno.c $(HDRS)
fopen.o:		fopen.c $(HDRS)
fprintf.o:		fprintf.c $(HDRS)
fputc.o:		fputc.c $(HDRS)
fputs.o:		fputs.c $(HDRS)
fread.o:		fread.c $(HDRS)
frexp.o:		frexp.c $(HDRS)
fscanf.o:		fscanf.c $(HDRS)
fseek.o:		fseek.c $(HDRS)
fsetpos.o:		fsetpos.c $(HDRS)
ftell.o:		ftell.c $(HDRS)
fwrite.o:		fwrite.c $(HDRS)
getenv.o:		getenv.c $(HDRS)
getopt.o:		getopt.c $(HDRS)
gets.o:			gets.c $(HDRS)
isalnum.o:		isalnum.c $(HDRS)
isalpha.o:		isalpha.c $(HDRS)
iscntrl.o:		iscntrl.c $(HDRS)
isdigit.o:		isdigit.c $(HDRS)
isgraph.o:		isgraph.c $(HDRS)
islower.o:		islower.c $(HDRS)
isprint.o:		isprint.c $(HDRS)
ispunct.o:		ispunct.c $(HDRS)
isspace.o:		isspace.c $(HDRS)
isupper.o:		isupper.c $(HDRS)
isxdigit.o:		isxdigit.c $(HDRS)
isatty.o:		isatty.c $(HDRS)
malloc.o:		malloc.c $(HDRS)
memchr.o:		memchr.c $(HDRS)
memcmp.o:		memcmp.c $(HDRS)
memcpy.o:		memcpy.c $(HDRS)
memmove.o:		memmove.c $(HDRS)
modf.o:			modf.c $(HDRS)
printf.o:		printf.c $(HDRS)
puts.o:			puts.c $(HDRS)
qsort.o:		qsort.c $(HDRS)
rand.o:			rand.c $(HDRS)
remove.o:		remove.c $(HDRS)
rewind.o:		rewind.c $(HDRS)
scanf.o:		scanf.c $(HDRS)
setbuf.o:		setbuf.c $(HDRS)
setvbuf.o:		setvbuf.c $(HDRS)
sprintf.o:		sprintf.c $(HDRS)
sscanf.o:		sscanf.c $(HDRS)
stdio.o:		stdio.c $(HDRS)
strcat.o:		strcat.c $(HDRS)
strchr.o:		strchr.c $(HDRS)
strcmp.o:		strcmp.c $(HDRS)
strcpy.o:		strcpy.c $(HDRS)
strerror.o:		strerror.c $(HDRS)
strftime.o:		strftime.c $(HDRS)
strlen.o:		strlen.c $(HDRS)
strncat.o:		strncat.c $(HDRS)
strncmp.o:		strncmp.c $(HDRS)
strncpy.o:		strncpy.c $(HDRS)
strrchr.o:		strrchr.c $(HDRS)
strtod.o:		strtod.c $(HDRS)
strtof.o:		strtof.c $(HDRS)
strtol.o:		strtol.c $(HDRS)
tcgetattr.o:		tcgetattr.c $(HDRS)
tolower.o:		tolower.c $(HDRS)
toupper.o:		toupper.c $(HDRS)
ungetc.o:		ungetc.c $(HDRS)
vfprintf.o:		vfprintf.c $(HDRS)
vfscanf.o:		vfscanf.c $(HDRS)
vsprintf.o:		vsprintf.c $(HDRS)
wait.o:			wait.c $(HDRS)
