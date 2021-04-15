.text
_do_write:
L2:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L14:
	movl %edx,%ebx
	movl %edi,%r13d
	movq %rsi,%r12
L5:
	movslq %ebx,%rdx
	movl %r13d,%edi
	movq %r12,%rsi
	call _write
	cmpl $0,%eax
	jle L7
L8:
	cmpl %ebx,%eax
	jge L7
L6:
	movslq %eax,%rsi
	subl %eax,%ebx
	addq %rsi,%r12
	jmp L5
L7:
	cmpl $0,%eax
	setg %sil
	movzbl %sil,%eax
L4:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L16:
___flushbuf:
L17:
	pushq %rbp
	movq %rsp,%rbp
	subq $8,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
L110:
	movl %edi,%r12d
	movq %rsi,%rbx
	movq $___stdio_cleanup,___exit_cleanup(%rip)
	movl 4(%rbx),%esi
	cmpl $0,%esi
	jge L22
L20:
	movzbl %r12b,%eax
	jmp L19
L22:
	movl 8(%rbx),%esi
	testl $2,%esi
	jnz L26
L24:
	movl $-1,%eax
	jmp L19
L26:
	testl $128,%esi
	jz L30
L31:
	testl $16,%esi
	jnz L30
L28:
	movl $-1,%eax
	jmp L19
L30:
	movl 8(%rbx),%esi
	andl $-129,%esi
	movl %esi,8(%rbx)
	orl $256,%esi
	movl %esi,8(%rbx)
	testl $4,%esi
	jnz L38
L36:
	cmpq $0,16(%rbx)
	jnz L38
L39:
	movq $___stdout,%rsi
	cmpq %rbx,%rsi
	jnz L43
L45:
	movl ___stdout+4(%rip),%edi
	call _isatty
	cmpl $0,%eax
	jz L43
L42:
	movl $1024,%edi
	call _malloc
	movq %rax,16(%rbx)
	cmpq $0,%rax
	jnz L50
L49:
	orl $4,8(%rbx)
	jmp L44
L50:
	orl $72,8(%rbx)
	movl $1024,12(%rbx)
	movl $-1,(%rbx)
	jmp L44
L43:
	movl $1024,%edi
	call _malloc
	movq %rax,16(%rbx)
	cmpq $0,%rax
	jnz L53
L52:
	orl $4,8(%rbx)
	jmp L44
L53:
	orl $8,8(%rbx)
	movl $1024,12(%rbx)
	movl 8(%rbx),%esi
	testl $64,%esi
	jnz L56
L55:
	movl $1023,(%rbx)
	jmp L44
L56:
	movl $-1,(%rbx)
L44:
	movq 16(%rbx),%rsi
	movq %rsi,24(%rbx)
L38:
	movl 8(%rbx),%esi
	testl $4,%esi
	jz L59
L58:
	movb %r12b,-8(%rbp)
	movl $0,(%rbx)
	movl 8(%rbx),%esi
	testl $512,%esi
	jz L63
L61:
	movl 4(%rbx),%edi
	xorl %esi,%esi
	movl $2,%edx
	call _lseek
	cmpq $-1,%rax
	jnz L63
L64:
	orl $32,8(%rbx)
	movl $-1,%eax
	jmp L19
L63:
	leaq -8(%rbp),%rsi
	movl 4(%rbx),%edi
	movl $1,%edx
	call _write
	cmpq $1,%rax
	jz L70
L68:
	orl $32,8(%rbx)
	movl $-1,%eax
	jmp L19
L70:
	movzbl %r12b,%eax
	jmp L19
L59:
	testl $64,%esi
	jz L74
L73:
	movq 24(%rbx),%rsi
	movq %rsi,%rdi
	addq $1,%rsi
	movq %rsi,24(%rbx)
	movb %r12b,(%rdi)
	cmpl $10,%r12d
	jz L76
L79:
	movl (%rbx),%esi
	movl 12(%rbx),%edi
	negl %edi
	cmpl %edi,%esi
	jnz L60
L76:
	movl (%rbx),%r13d
	negl %r13d
	movq 16(%rbx),%rsi
	movq %rsi,24(%rbx)
	movl $0,(%rbx)
	movl 8(%rbx),%esi
	testl $512,%esi
	jz L85
L83:
	movl 4(%rbx),%edi
	xorl %esi,%esi
	movl $2,%edx
	call _lseek
	cmpq $-1,%rax
	jnz L85
L86:
	orl $32,8(%rbx)
	movl $-1,%eax
	jmp L19
L85:
	movq 16(%rbx),%rsi
	movl 4(%rbx),%edi
	movl %r13d,%edx
	call _do_write
	cmpl $0,%eax
	jnz L60
L90:
	orl $32,8(%rbx)
	movl $-1,%eax
	jmp L19
L74:
	movq 24(%rbx),%r13
	subq 16(%rbx),%r13
	movl 12(%rbx),%esi
	addl $-1,%esi
	movl %esi,(%rbx)
	movq 16(%rbx),%rsi
	addq $1,%rsi
	movq %rsi,24(%rbx)
	cmpl $0,%r13d
	jle L96
L94:
	movl 8(%rbx),%esi
	testl $512,%esi
	jz L99
L97:
	movl 4(%rbx),%edi
	xorl %esi,%esi
	movl $2,%edx
	call _lseek
	cmpq $-1,%rax
	jnz L99
L100:
	orl $32,8(%rbx)
	movl $-1,%eax
	jmp L19
L99:
	movq 16(%rbx),%rsi
	movl 4(%rbx),%edi
	movl %r13d,%edx
	call _do_write
	cmpl $0,%eax
	jnz L96
L104:
	movq 16(%rbx),%rsi
	movb %r12b,(%rsi)
	orl $32,8(%rbx)
	movl $-1,%eax
	jmp L19
L96:
	movq 16(%rbx),%rsi
	movb %r12b,(%rsi)
L60:
	movzbl %r12b,%eax
L19:
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L112:
.globl ___stdio_cleanup
.globl ___exit_cleanup
.globl ___stdout
.globl _write
.globl _malloc
.globl ___flushbuf
.globl _isatty
.globl _lseek
