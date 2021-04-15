.data
.align 4
_next:
	.int 1
.text
_rand:
L2:
	pushq %rbp
	movq %rsp,%rbp
L3:
	movl _next(%rip),%esi
	imull $1103515245,%esi
	addl $12345,%esi
	movl %esi,_next(%rip)
	movl %esi,%eax
	movl $65536,%esi
	cqto
	idivq %rsi
	movl $32768,%esi
	cqto
	idivq %rsi
	movl %edx,%eax
L4:
	popq %rbp
	ret
L9:
_srand:
L10:
	pushq %rbp
	movq %rsp,%rbp
L11:
	movl %edi,_next(%rip)
L12:
	popq %rbp
	ret
L16:
.globl _srand
.globl _rand
