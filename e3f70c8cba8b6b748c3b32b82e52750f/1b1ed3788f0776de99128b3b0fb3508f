.text
_dead0:
L4:
	pushq %rbp
	movq %rsp,%rbp
	subq $24,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L43:
	movq %rdi,%r13
	xorps %xmm0,%xmm0
	movups %xmm0,-24(%rbp)
	movq $0,-8(%rbp)
	leaq -16(%rbp),%rsi
	movq %rsi,-8(%rbp)
L7:
	xorl %r14d,%r14d
	movq 8(%r13),%r12
L10:
	cmpq $0,%r12
	jz L9
L11:
	movl (%r12),%esi
	cmpl $524288,%esi
	jz L17
L16:
	movq %r12,%rdi
	call _insn_defs_mem
	cmpl $0,%eax
	jnz L17
L21:
	movq %r12,%rdi
	call _insn_side_effects
	cmpl $0,%eax
	jnz L17
L25:
	leaq -24(%rbp),%rbx
	movq %r12,%rdi
	movq %rbx,%rsi
	xorl %edx,%edx
	call _insn_defs_regs
	movq %r12,%rdi
	call _insn_defs_cc
	cmpl $0,%eax
	jz L29
L27:
	movq %rbx,%rdi
	movl $2147483648,%esi
	movl $1,%edx
	call _regs_lookup
L29:
	movq -16(%rbp),%rbx
L30:
	cmpq $0,%rbx
	jz L33
L31:
	movl 8(%r12),%edx
	movl (%rbx),%esi
	leaq 32(%r13),%rdi
	call _range_by_def
	movl 12(%rax),%esi
	cmpl $0,%esi
	jnz L17
L32:
	movq 8(%rbx),%rbx
	jmp L30
L33:
	movl $1,_changed(%rip)
	addl $1,%r14d
	pushq $524288
	pushq %r12
	call _insn_replace
	addq $16,%rsp
	movl 8(%r12),%esi
	leaq 32(%r13),%rdi
	call _live_kill_insn
	cmpl $0,%eax
	jnz L17
L38:
	movl $1,_invalidated(%rip)
L17:
	leaq -24(%rbp),%rdi
	call _regs_clear
	movq 64(%r12),%r12
	jmp L10
L9:
	cmpl $0,%r14d
	jnz L7
L8:
	xorl %eax,%eax
L6:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L45:
_dead:
L46:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L47:
	xorl %ebx,%ebx
	movl $1,_invalidated(%rip)
L49:
	movl _invalidated(%rip),%esi
	cmpl $0,%esi
	jz L54
L52:
	call _live_analyze
L54:
	movl $0,_changed(%rip)
	movl $0,_invalidated(%rip)
	movq $_dead0,%rdi
	call _blocks_iter
	movl _changed(%rip),%esi
	cmpl $0,%esi
	jz L51
L55:
	movl $1,%ebx
L51:
	movl _changed(%rip),%esi
	cmpl $0,%esi
	jz L50
L58:
	movl _invalidated(%rip),%esi
	cmpl $0,%esi
	jnz L49
L50:
	cmpl $0,%ebx
	jz L48
L62:
	call _nop
L48:
	popq %rbx
	popq %rbp
	ret
L68:
.globl _nop
.globl _regs_lookup
.globl _blocks_iter
.globl _regs_clear
.globl _insn_defs_regs
.globl _live_analyze
.globl _insn_side_effects
.globl _insn_defs_cc
.local _invalidated
.comm _invalidated, 4, 4
.local _changed
.comm _changed, 4, 4
.globl _dead
.globl _insn_replace
.globl _range_by_def
.globl _insn_defs_mem
.globl _live_kill_insn
