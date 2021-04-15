.text
_isalpha:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	movslq %edi,%rsi
	movzbl ___ctype+1(%rsi),%eax
	andl $3,%eax
L3:
	popq %rbp
	ret
L8:
.globl _isalpha
.globl ___ctype
