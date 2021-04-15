.text
_isdigit:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	leal -48(%rdi),%esi
	cmpl $10,%esi
	setb %sil
	movzbl %sil,%eax
L3:
	popq %rbp
	ret
L8:
.globl _isdigit
