.text
_fclose:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L42:
	movq %rdi,%r12
	xorl %ebx,%ebx
	xorl %esi,%esi
L5:
	movslq %esi,%rdi
	cmpq %r12,___iotab(,%rdi,8)
	jnz L6
L8:
	movq $0,___iotab(,%rdi,8)
	jmp L7
L6:
	addl $1,%esi
	cmpl $20,%esi
	jl L5
L7:
	cmpl $20,%esi
	jl L14
L12:
	movl $-1,%eax
	jmp L3
L14:
	movq %r12,%rdi
	call _fflush
	cmpl $0,%eax
	jz L18
L16:
	movl $-1,%ebx
L18:
	movl 4(%r12),%edi
	call _close
	cmpl $0,%eax
	jz L21
L19:
	movl $-1,%ebx
L21:
	movl 8(%r12),%esi
	testl $8,%esi
	jz L24
L25:
	movq 16(%r12),%rdi
	cmpq $0,%rdi
	jz L24
L22:
	call _free
L24:
	movq $___stdin,%rsi
	cmpq %r12,%rsi
	jz L31
L36:
	movq $___stdout,%rsi
	cmpq %r12,%rsi
	jz L31
L32:
	movq $___stderr,%rsi
	cmpq %r12,%rsi
	jz L31
L29:
	movq %r12,%rdi
	call _free
L31:
	movl %ebx,%eax
L3:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L44:
.globl ___stdout
.globl ___stderr
.globl ___iotab
.globl _close
.globl _free
.globl _fclose
.globl _fflush
.globl ___stdin
