.text
_qexchange:
L3:
	pushq %rbp
	movq %rsp,%rbp
L6:
	movq %rdx,%rax
	addq $-1,%rdx
	cmpq $0,%rax
	jz L5
L7:
	movzbl (%rdi),%eax
	movzbl (%rsi),%ecx
	movb %cl,(%rdi)
	addq $1,%rdi
	movb %al,(%rsi)
	addq $1,%rsi
	jmp L6
L5:
	popq %rbp
	ret
L12:
_q3exchange:
L14:
	pushq %rbp
	movq %rsp,%rbp
L17:
	movq %rcx,%rax
	addq $-1,%rcx
	cmpq $0,%rax
	jz L16
L18:
	movzbl (%rdi),%eax
	movzbl (%rdx),%r8d
	movb %r8b,(%rdi)
	movzbl (%rsi),%r8d
	addq $1,%rdi
	movb %r8b,(%rdx)
	addq $1,%rdx
	movb %al,(%rsi)
	addq $1,%rsi
	jmp L17
L16:
	popq %rbp
	ret
L23:
_qsort1:
L25:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L65:
	movq %rdi,-8(%rbp)	 # spill
	movq %rsi,-16(%rbp)	 # spill
	movq %rdx,%r12
L29:
	movq -8(%rbp),%r10	 # spill
	cmpq %r10,-16(%rbp)	 # spill
	ja L34
L27:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L34:
	movq -8(%rbp),%r15	 # spill
	movq -16(%rbp),%rbx	 # spill
	movq -16(%rbp),%rsi	 # spill
	subq -8(%rbp),%rsi	 # spill
	leaq (%rsi,%r12),%rax
	leaq (,%r12,2),%rsi
	xorl %edx,%edx
	divq %rsi
	movq %r12,%rsi
	imulq %rax,%rsi
	movq -8(%rbp),%r10	 # spill
	leaq (%r10,%rsi),%r13
	movq %r13,%r14
L37:
	cmpq %r14,%r15
	jae L47
L40:
	movq _qcompar(%rip),%rax
	movq %r15,%rdi
	movq %r14,%rsi
	call *%rax
	cmpl $0,%eax
	jg L47
L38:
	cmpl $0,%eax
	jge L45
L44:
	addq %r12,%r15
	jmp L37
L45:
	subq %r12,%r14
	movq %r15,%rdi
	movq %r14,%rsi
	movq %r12,%rdx
	call _qexchange
	jmp L37
L47:
	cmpq %r13,%rbx
	jbe L49
L48:
	movq _qcompar(%rip),%rax
	movq %rbx,%rdi
	movq %r13,%rsi
	call *%rax
	cmpl $0,%eax
	jge L51
L50:
	cmpq %r14,%r15
	jae L55
L53:
	movq %r15,%rdi
	movq %rbx,%rsi
	movq %r12,%rdx
	call _qexchange
	addq %r12,%r15
	subq %r12,%rbx
	jmp L37
L55:
	addq %r12,%r13
	movq %r15,%rdi
	movq %r13,%rsi
	movq %rbx,%rdx
	movq %r12,%rcx
	call _q3exchange
	addq %r12,%r14
	movq %r14,%r15
	jmp L47
L51:
	cmpl $0,%eax
	jnz L58
L57:
	addq %r12,%r13
	movq %rbx,%rdi
	movq %r13,%rsi
	movq %r12,%rdx
	call _qexchange
	jmp L47
L58:
	subq %r12,%rbx
	jmp L47
L49:
	cmpq %r14,%r15
	jae L62
L60:
	subq %r12,%r14
	movq %rbx,%rdi
	movq %r14,%rsi
	movq %r15,%rdx
	movq %r12,%rcx
	call _q3exchange
	subq %r12,%r13
	movq %r13,%rbx
	jmp L37
L62:
	subq %r12,%r14
	movq -8(%rbp),%rdi	 # spill
	movq %r14,%rsi
	movq %r12,%rdx
	call _qsort1
	leaq (%r13,%r12),%r10
	movq %r10,-8(%rbp)	 # spill
	jmp L29
L67:
_qsort:
L68:
	pushq %rbp
	movq %rsp,%rbp
L69:
	cmpq $0,%rsi
	jz L70
L73:
	movq %rcx,_qcompar(%rip)
	addq $-1,%rsi
	imulq %rdx,%rsi
	addq %rdi,%rsi
	call _qsort1
L70:
	popq %rbp
	ret
L78:
.local _qcompar
.comm _qcompar, 8, 8
.globl _qsort
