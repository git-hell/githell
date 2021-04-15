.text
_fflush:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L73:
	movq %rdi,%r12
	xorl %ebx,%ebx
	cmpq $0,%r12
	jnz L6
L4:
	xorl %r12d,%r12d
L8:
	movslq %r12d,%rsi
	movq ___iotab(,%rsi,8),%rdi
	cmpq $0,%rdi
	jz L9
L14:
	call _fflush
	cmpl $0,%eax
	jz L9
L11:
	movl $-1,%ebx
L9:
	addl $1,%r12d
	cmpl $20,%r12d
	jl L8
L10:
	movl %ebx,%eax
	jmp L3
L6:
	cmpq $0,16(%r12)
	jz L19
L22:
	movl 8(%r12),%esi
	testl $128,%esi
	jnz L21
L26:
	testl $256,%esi
	jnz L21
L19:
	xorl %eax,%eax
	jmp L3
L21:
	movl 8(%r12),%edi
	testl $128,%edi
	jz L32
L31:
	xorl %esi,%esi
	cmpq $0,16(%r12)
	jz L36
L37:
	testl $4,%edi
	jnz L36
L34:
	movl (%r12),%esi
	negl %esi
L36:
	movl $0,(%r12)
	movslq %esi,%rsi
	movl 4(%r12),%edi
	movl $1,%edx
	call _lseek
	cmpq $-1,%rax
	jnz L43
L41:
	orl $32,8(%r12)
	movl $-1,%eax
	jmp L3
L43:
	movl 8(%r12),%esi
	testl $2,%esi
	jz L47
L45:
	andl $-385,%esi
	movl %esi,8(%r12)
L47:
	movq 16(%r12),%rsi
	movq %rsi,24(%r12)
	xorl %eax,%eax
	jmp L3
L32:
	testl $4,%edi
	jz L33
L49:
	xorl %eax,%eax
	jmp L3
L33:
	testl $1,%edi
	jz L55
L53:
	andl $-257,%edi
	movl %edi,8(%r12)
L55:
	movq 24(%r12),%rbx
	movq 16(%r12),%rsi
	movq %rsi,24(%r12)
	subq %rsi,%rbx
	cmpl $0,%ebx
	jg L58
L56:
	xorl %eax,%eax
	jmp L3
L58:
	movl 8(%r12),%esi
	testl $512,%esi
	jz L62
L60:
	movl 4(%r12),%edi
	xorl %esi,%esi
	movl $2,%edx
	call _lseek
	cmpq $-1,%rax
	jnz L62
L63:
	orl $32,8(%r12)
	movl $-1,%eax
	jmp L3
L62:
	movslq %ebx,%rdx
	movq 16(%r12),%rsi
	movl 4(%r12),%edi
	call _write
	movl $0,(%r12)
	cmpl %eax,%ebx
	jnz L69
L67:
	xorl %eax,%eax
	jmp L3
L69:
	orl $32,8(%r12)
	movl $-1,%eax
L3:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L75:
___stdio_cleanup:
L76:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L77:
	xorl %ebx,%ebx
L80:
	movslq %ebx,%rsi
	movq ___iotab(,%rsi,8),%rdi
	cmpq $0,%rdi
	jz L81
L86:
	movl 8(%rdi),%esi
	testl $256,%esi
	jz L81
L83:
	call _fflush
L81:
	addl $1,%ebx
	cmpl $20,%ebx
	jl L80
L78:
	popq %rbx
	popq %rbp
	ret
L93:
.globl ___stdio_cleanup
.globl _write
.globl ___iotab
.globl _fflush
.globl _lseek
