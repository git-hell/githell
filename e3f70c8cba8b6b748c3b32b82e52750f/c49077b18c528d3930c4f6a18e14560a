.text
_printf:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	movq 16(%rbp),%rsi
	leaq 24(%rbp),%rdx
	movq $___stdout,%rdi
	call _vfprintf
L3:
	popq %rbp
	ret
L8:
.globl ___stdout
.globl _vfprintf
.globl _printf
