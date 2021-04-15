.text
_bitset_init:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L5:
	movq %rdi,%rbx
	movl %esi,%esi
	addq $31,%rsi
	shrq $5,%rsi
	shlq $5,%rsi
	movl %esi,%edi
	shrq $5,%rdi
	movl %edi,(%rbx)
	shrl $3,%esi
	movl %esi,%edi
	call _safe_malloc
	movq %rax,8(%rbx)
	movq %rbx,%rdi
	call _bitset_zero_all
L3:
	popq %rbx
	popq %rbp
	ret
L7:
_bitset_clear:
L8:
	pushq %rbp
	movq %rsp,%rbp
L9:
	movq 8(%rdi),%rdi
	call _free
L10:
	popq %rbp
	ret
L14:
_bitset_zero_all:
L15:
	pushq %rbp
	movq %rsp,%rbp
L16:
	movl (%rdi),%esi
	movl %esi,%esi
	leaq (,%rsi,4),%rdx
	movq 8(%rdi),%rdi
	xorl %esi,%esi
	call _memset
L17:
	popq %rbp
	ret
L21:
_bitset_one_all:
L22:
	pushq %rbp
	movq %rsp,%rbp
L23:
	movl (%rdi),%esi
	movl %esi,%esi
	leaq (,%rsi,4),%rdx
	movq 8(%rdi),%rdi
	movl $255,%esi
	call _memset
L24:
	popq %rbp
	ret
L28:
_bitset_set:
L29:
	pushq %rbp
	movq %rsp,%rbp
L30:
	movslq %esi,%rax
	movq %rax,%rcx
	andq $31,%rcx
	movl $1,%esi
	shll %cl,%esi
	movq 8(%rdi),%rdi
	sarq $5,%rax
	orl %esi,(%rdi,%rax,4)
L31:
	popq %rbp
	ret
L35:
_bitset_reset:
L36:
	pushq %rbp
	movq %rsp,%rbp
L37:
	movslq %esi,%rax
	movq %rax,%rcx
	andq $31,%rcx
	movl $1,%esi
	shll %cl,%esi
	notl %esi
	movq 8(%rdi),%rdi
	sarq $5,%rax
	andl %esi,(%rdi,%rax,4)
L38:
	popq %rbp
	ret
L42:
_bitset_get:
L43:
	pushq %rbp
	movq %rsp,%rbp
L44:
	movq 8(%rdi),%rdi
	movslq %esi,%rcx
	movq %rcx,%rsi
	sarq $5,%rsi
	movl (%rdi,%rsi,4),%eax
	andq $31,%rcx
	movl $1,%esi
	shll %cl,%esi
	andl %esi,%eax
L45:
	popq %rbp
	ret
L50:
_bitset_copy:
L51:
	pushq %rbp
	movq %rsp,%rbp
L52:
	xorl %eax,%eax
L54:
	movl (%rdi),%ecx
	cmpl %ecx,%eax
	jae L53
L55:
	movq 8(%rsi),%rdx
	movslq %eax,%rcx
	movl (%rdx,%rcx,4),%edx
	movq 8(%rdi),%r8
	movl %edx,(%r8,%rcx,4)
	addl $1,%eax
	jmp L54
L53:
	popq %rbp
	ret
L61:
_bitset_and:
L62:
	pushq %rbp
	movq %rsp,%rbp
L63:
	xorl %eax,%eax
L65:
	movl (%rdi),%ecx
	cmpl %ecx,%eax
	jae L64
L66:
	movq 8(%rsi),%rdx
	movslq %eax,%rcx
	movl (%rdx,%rcx,4),%edx
	movq 8(%rdi),%r8
	andl %edx,(%r8,%rcx,4)
	addl $1,%eax
	jmp L65
L64:
	popq %rbp
	ret
L72:
_bitset_or:
L73:
	pushq %rbp
	movq %rsp,%rbp
L74:
	xorl %eax,%eax
L76:
	movl (%rdi),%ecx
	cmpl %ecx,%eax
	jae L75
L77:
	movq 8(%rsi),%rdx
	movslq %eax,%rcx
	movl (%rdx,%rcx,4),%edx
	movq 8(%rdi),%r8
	orl %edx,(%r8,%rcx,4)
	addl $1,%eax
	jmp L76
L75:
	popq %rbp
	ret
L83:
_bitset_bic:
L84:
	pushq %rbp
	movq %rsp,%rbp
L85:
	xorl %eax,%eax
L87:
	movl (%rdi),%ecx
	cmpl %ecx,%eax
	jae L86
L88:
	movq 8(%rsi),%rdx
	movslq %eax,%rcx
	movl (%rdx,%rcx,4),%edx
	notl %edx
	movq 8(%rdi),%r8
	andl %edx,(%r8,%rcx,4)
	addl $1,%eax
	jmp L87
L86:
	popq %rbp
	ret
L94:
_bitset_same:
L95:
	pushq %rbp
	movq %rsp,%rbp
L96:
	movl (%rdi),%eax
	movl %eax,%eax
	leaq (,%rax,4),%rdx
	movq 8(%rsi),%rsi
	movq 8(%rdi),%rdi
	call _memcmp
	cmpl $0,%eax
	setz %sil
	movzbl %sil,%eax
L97:
	popq %rbp
	ret
L102:
.globl _memcmp
.globl _bitset_clear
.globl _bitset_or
.globl _bitset_get
.globl _bitset_reset
.globl _bitset_set
.globl _bitset_and
.globl _bitset_init
.globl _memset
.globl _bitset_one_all
.globl _bitset_zero_all
.globl _bitset_bic
.globl _safe_malloc
.globl _bitset_same
.globl _free
.globl _bitset_copy
