.text
_isupper:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	leal -65(%rdi),%esi
	cmpl $26,%esi
	setb %sil
	movzbl %sil,%eax
L3:
	popq %rbp
	ret
L8:
.globl _isupper
