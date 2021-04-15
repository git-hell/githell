.text
_isprint:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	leal -32(%rdi),%esi
	cmpl $95,%esi
	setb %sil
	movzbl %sil,%eax
L3:
	popq %rbp
	ret
L8:
.globl _isprint
