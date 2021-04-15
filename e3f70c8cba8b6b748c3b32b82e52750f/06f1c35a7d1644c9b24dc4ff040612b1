.text
_fopen:
L1:
	pushq %rbp
	movq %rsp,%rbp
	subq $8,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L74:
	movq %rdi,%r15
	movl $0,-8(%rbp)	 # spill
	xorl %r12d,%r12d
L4:
	movslq %r12d,%rdi
	cmpq $0,___iotab(,%rdi,8)
	jz L7
L5:
	cmpl $19,%r12d
	jl L6
L8:
	xorl %eax,%eax
	jmp L3
L6:
	addl $1,%r12d
	jmp L4
L7:
	movzbl (%rsi),%edi
	addq $1,%rsi
	cmpl $97,%edi
	jz L20
L68:
	cmpl $114,%edi
	jz L16
L69:
	cmpl $119,%edi
	jz L18
L13:
	xorl %eax,%eax
	jmp L3
L18:
	movl $258,%ebx
	movl $1,%r13d
	movl $576,-8(%rbp)	 # spill
	jmp L23
L16:
	movl $129,%ebx
	xorl %r13d,%r13d
	jmp L23
L20:
	movl $770,%ebx
	movl $1,%r13d
	movl $1088,-8(%rbp)	 # spill
L23:
	movzbl (%rsi),%edi
	cmpl $0,%edi
	jz L25
L24:
	movzbl (%rsi),%edi
	addq $1,%rsi
	cmpl $43,%edi
	jz L32
L72:
	cmpl $98,%edi
	jz L23
	jnz L25
L32:
	movl $2,%r13d
	orl $3,%ebx
	jmp L23
L25:
	testl $512,-8(%rbp)	 # spill
	jnz L36
L39:
	pushq %r13
	pushq %r15
	call _open
	addq $16,%rsp
	movl %eax,%r14d
	cmpl $0,%eax
	jge L38
L43:
	testl $64,-8(%rbp)	 # spill
	jz L38
L36:
	movq %r15,%rdi
	movl $438,%esi
	call _creat
	movl %eax,%r14d
	cmpl $0,%eax
	jle L38
L50:
	movl %ebx,%esi
	orl $1,%esi
	cmpl $0,%esi
	jz L38
L47:
	movl %eax,%edi
	call _close
	pushq %r13
	pushq %r15
	call _open
	addq $16,%rsp
	movl %eax,%r14d
L38:
	cmpl $0,%r14d
	jge L56
L54:
	xorl %eax,%eax
	jmp L3
L56:
	movl $32,%edi
	call _malloc
	cmpq $0,%rax
	jnz L60
L58:
	movl %r14d,%edi
	call _close
	xorl %eax,%eax
	jmp L3
L60:
	movl %ebx,%esi
	andl $3,%esi
	cmpl $3,%esi
	jnz L64
L62:
	andl $-385,%ebx
L64:
	movl $0,(%rax)
	movl %r14d,4(%rax)
	movl %ebx,8(%rax)
	movq $0,16(%rax)
	movslq %r12d,%rsi
	movq %rax,___iotab(,%rsi,8)
L3:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L76:
.globl _creat
.globl ___iotab
.globl _malloc
.globl _close
.globl _open
.globl _fopen
