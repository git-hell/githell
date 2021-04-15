.text
_iscntrl:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	movslq %edi,%rsi
	movzbl ___ctype+1(%rsi),%eax
	andl $32,%eax
L3:
	popq %rbp
	ret
L8:
.globl ___ctype
.globl _iscntrl
