.data
.align 4
_opterr:
	.int 1
.align 4
_optind:
	.int 1
.align 4
L4:
	.int 1
.text
_getopt:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L59:
	movq %rsi,%r14
	movl %edi,%r13d
	movq %rdx,%rbx
	movl L4(%rip),%esi
	cmpl $1,%esi
	jnz L7
L5:
	movl _optind(%rip),%esi
	cmpl %r13d,%esi
	jge L8
L15:
	movslq %esi,%rsi
	movq (%r14,%rsi,8),%rdi
	movzbl (%rdi),%esi
	cmpl $45,%esi
	jnz L8
L11:
	movzbl 1(%rdi),%esi
	cmpl $0,%esi
	jnz L9
L8:
	movl $-1,%eax
	jmp L3
L9:
	movq $L23,%rsi
	call _strcmp
	cmpl $0,%eax
	jnz L7
L20:
	addl $1,_optind(%rip)
	movl $-1,%eax
	jmp L3
L7:
	movl _optind(%rip),%esi
	movslq %esi,%rsi
	movq (%r14,%rsi,8),%rsi
	movl L4(%rip),%edi
	movslq %edi,%rdi
	movzbl (%rsi,%rdi),%r12d
	movl %r12d,_optopt(%rip)
	cmpl $58,%r12d
	jz L25
L28:
	movq %rbx,%rdi
	movl %r12d,%esi
	call _strchr
	cmpq $0,%rax
	jnz L27
L25:
	movl _opterr(%rip),%esi
	cmpl $0,%esi
	jz L34
L32:
	movq (%r14),%rsi
	pushq %r12
	pushq %rsi
	pushq $L35
	pushq $___stderr
	call _fprintf
	addq $32,%rsp
L34:
	movl _optind(%rip),%esi
	movslq %esi,%rsi
	movq (%r14,%rsi,8),%rsi
	movl L4(%rip),%edi
	addl $1,%edi
	movl %edi,L4(%rip)
	movslq %edi,%rdi
	movzbl (%rsi,%rdi),%esi
	cmpl $0,%esi
	jnz L38
L36:
	addl $1,_optind(%rip)
	movl $1,L4(%rip)
L38:
	movl $63,%eax
	jmp L3
L27:
	movzbl 1(%rax),%esi
	cmpl $58,%esi
	jnz L41
L40:
	movl _optind(%rip),%edi
	movslq %edi,%rsi
	movq (%r14,%rsi,8),%rsi
	movl L4(%rip),%eax
	addl $1,%eax
	movslq %eax,%rax
	movzbl (%rsi,%rax),%esi
	cmpl $0,%esi
	jz L44
L43:
	leal 1(%rdi),%esi
	movl %esi,_optind(%rip)
	movslq %edi,%rsi
	movq (%r14,%rsi,8),%rsi
	movl L4(%rip),%edi
	addl $1,%edi
	movslq %edi,%rdi
	addq %rdi,%rsi
	movq %rsi,_optarg(%rip)
	jmp L45
L44:
	leal 1(%rdi),%esi
	movl %esi,_optind(%rip)
	cmpl %r13d,%esi
	jl L47
L46:
	movl _opterr(%rip),%esi
	cmpl $0,%esi
	jz L51
L49:
	movq (%r14),%rsi
	pushq %r12
	pushq %rsi
	pushq $L52
	pushq $___stderr
	call _fprintf
	addq $32,%rsp
L51:
	movl $1,L4(%rip)
	movl $63,%eax
	jmp L3
L47:
	addl $2,%edi
	movl %edi,_optind(%rip)
	movslq %esi,%rsi
	movq (%r14,%rsi,8),%rsi
	movq %rsi,_optarg(%rip)
L45:
	movl $1,L4(%rip)
	jmp L42
L41:
	movl _optind(%rip),%esi
	movslq %esi,%rsi
	movq (%r14,%rsi,8),%rsi
	movl L4(%rip),%edi
	addl $1,%edi
	movl %edi,L4(%rip)
	movslq %edi,%rdi
	movzbl (%rsi,%rdi),%esi
	cmpl $0,%esi
	jnz L56
L54:
	movl $1,L4(%rip)
	addl $1,_optind(%rip)
L56:
	movq $0,_optarg(%rip)
L42:
	movl %r12d,%eax
L3:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L61:
L52:
	.byte 37,115,58,32,111,112,116,105
	.byte 111,110,32,114,101,113,117,105
	.byte 114,101,115,32,97,110,32,97
	.byte 114,103,117,109,101,110,116,32
	.byte 45,45,32,37,99,10,0
L35:
	.byte 37,115,58,32,105,108,108,101
	.byte 103,97,108,32,111,112,116,105
	.byte 111,110,32,45,45,32,37,99
	.byte 10,0
L23:
	.byte 45,45,0
.globl _strcmp
.globl _optind
.globl _fprintf
.comm _optarg, 8, 8
.globl _optarg
.globl _opterr
.globl _strchr
.globl ___stderr
.globl _getopt
.comm _optopt, 4, 4
.globl _optopt
