.text
_refill:
L3:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L20:
	movl %edi,%r14d
	leal 5(%r14),%ecx
	movl $1,%ebx
	shlq %cl,%rbx
	movq %rbx,%r12
	xorl %edi,%edi
	call _sbrk
	leaq 4095(%rax),%rsi
	shrq $12,%rsi
	shlq $12,%rsi
	subq %rax,%rsi
	cmpq $4096,%rbx
	jae L7
L6:
	movl $4096,%eax
	xorl %edx,%edx
	divq %rbx
	movl %eax,%r13d
	jmp L8
L7:
	movl $1,%r13d
L8:
	movl %r13d,%edi
	imull %ebx,%edi
	addl %edi,%esi
	movslq %esi,%rdi
	call _sbrk
	movq %rax,%rsi
	cmpq $-1,%rax
	jnz L11
L9:
	xorl %eax,%eax
	jmp L5
L11:
	xorl %edi,%edi
	movslq %r14d,%rax
L13:
	cmpl %r13d,%edi
	jge L16
L14:
	movq _buckets(,%rax,8),%rcx
	movq %rcx,(%rsi)
	movq %rsi,_buckets(,%rax,8)
	addq %r12,%rsi
	addl $1,%edi
	jmp L13
L16:
	movl %r13d,%eax
L5:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L22:
_malloc:
L23:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L24:
	addq $8,%rdi
	xorl %ebx,%ebx
L27:
	leal 5(%rbx),%ecx
	movl $1,%esi
	shlq %cl,%rsi
	cmpq %rsi,%rdi
	jbe L29
L28:
	addl $1,%ebx
	cmpl $26,%ebx
	jl L27
L29:
	cmpl $26,%ebx
	jnz L36
L34:
	xorl %eax,%eax
	jmp L25
L36:
	movslq %ebx,%rsi
	cmpq $0,_buckets(,%rsi,8)
	jnz L40
L38:
	movl %ebx,%edi
	call _refill
	cmpl $0,%eax
	jnz L40
L41:
	xorl %eax,%eax
	jmp L25
L40:
	movslq %ebx,%rsi
	movq _buckets(,%rsi,8),%rdi
	movq (%rdi),%rax
	movq %rax,_buckets(,%rsi,8)
	movl %ebx,4(%rdi)
	movl $1265200743,(%rdi)
	leaq 8(%rdi),%rax
L25:
	popq %rbx
	popq %rbp
	ret
L49:
_free:
L50:
	pushq %rbp
	movq %rsp,%rbp
L51:
	leaq -8(%rdi),%rsi
	movl -8(%rdi),%eax
	cmpl $1265200743,%eax
	jnz L52
L53:
	movl -4(%rdi),%eax
	movslq %eax,%rcx
	movq _buckets(,%rcx,8),%rcx
	movq %rcx,-8(%rdi)
	movslq %eax,%rdi
	movq %rsi,_buckets(,%rdi,8)
L52:
	popq %rbp
	ret
L59:
.globl _sbrk
.local _buckets
.comm _buckets, 208, 8
.globl _malloc
.globl _free
