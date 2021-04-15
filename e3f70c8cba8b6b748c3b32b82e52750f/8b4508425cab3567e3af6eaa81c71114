.text
_isspace:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	movslq %edi,%rsi
	movzbl ___ctype+1(%rsi),%eax
	andl $8,%eax
L3:
	popq %rbp
	ret
L8:
.globl ___ctype
.globl _isspace
