.text
_isatty:
L1:
	pushq %rbp
	movq %rsp,%rbp
	subq $40,%rsp
L2:
	leaq -40(%rbp),%rsi
	call _tcgetattr
	cmpl $-1,%eax
	setnz %sil
	movzbl %sil,%eax
L3:
	movq %rbp,%rsp
	popq %rbp
	ret
L8:
.globl _tcgetattr
.globl _isatty
