.text
_fsetpos:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	movq (%rsi),%rsi
	xorl %edx,%edx
	call _fseek
L3:
	popq %rbp
	ret
L8:
.globl _fsetpos
.globl _fseek
