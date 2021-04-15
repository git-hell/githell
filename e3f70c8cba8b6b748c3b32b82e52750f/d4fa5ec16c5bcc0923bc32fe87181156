.text
_exit:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L8:
	movl %edi,%ebx
	movq ___exit_cleanup(%rip),%rsi
	cmpq $0,%rsi
	jz L6
L4:
	call *%rsi
L6:
	movl %ebx,%edi
	call ___exit
L3:
	popq %rbx
	popq %rbp
	ret
L10:
.comm ___exit_cleanup, 8, 8
.globl ___exit_cleanup
.globl _exit
.globl ___exit
