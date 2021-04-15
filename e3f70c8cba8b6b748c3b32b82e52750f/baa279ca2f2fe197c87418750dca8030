.text
_copyps_clear:
L3:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L16:
	movq %rdi,%rbx
L6:
	movq 8(%rbx),%rdi
	cmpq $0,%rdi
	jz L5
L7:
	addl $-1,(%rbx)
	movq 8(%rdi),%rsi
	cmpq $0,%rsi
	jz L14
L12:
	movq 16(%rdi),%rax
	movq %rax,16(%rsi)
L14:
	movq 8(%rdi),%rsi
	movq 16(%rdi),%rax
	movq %rsi,(%rax)
	call _free
	jmp L6
L5:
	popq %rbx
	popq %rbp
	ret
L18:
_copyps_lookup:
L20:
	pushq %rbp
	movq %rsp,%rbp
L21:
	movq 8(%rdi),%rax
L23:
	cmpq $0,%rax
	jz L26
L24:
	movl (%rax),%edi
	cmpl %esi,%edi
	jz L22
L25:
	movq 8(%rax),%rax
	jmp L23
L26:
	xorl %eax,%eax
L22:
	popq %rbp
	ret
L35:
_copyps_unset:
L37:
	pushq %rbp
	movq %rsp,%rbp
L55:
	movq %rdi,%rcx
	movq 8(%rcx),%rdi
L40:
	cmpq $0,%rdi
	jz L39
L41:
	movl (%rdi),%eax
	cmpl %esi,%eax
	jnz L42
L44:
	addl $-1,(%rcx)
	movq 8(%rdi),%rsi
	cmpq $0,%rsi
	jz L52
L50:
	movq 16(%rdi),%rax
	movq %rax,16(%rsi)
L52:
	movq 8(%rdi),%rsi
	movq 16(%rdi),%rax
	movq %rsi,(%rax)
	call _free
	jmp L39
L42:
	movq 8(%rdi),%rdi
	jmp L40
L39:
	popq %rbp
	ret
L57:
_copyps_insert:
L59:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L81:
	movl %esi,%r12d
	movq %rdi,%rbx
	movq 8(%rbx),%rsi
L62:
	cmpq $0,%rsi
	jz L65
L63:
	movl (%rsi),%edi
	cmpl %r12d,%edi
	jz L65
L64:
	movq 8(%rsi),%rsi
	jmp L62
L65:
	cmpq $0,%rsi
	jnz L72
L70:
	movl $24,%edi
	call _safe_malloc
	movq %rax,%rsi
	movl %r12d,(%rax)
	movq 8(%rbx),%rcx
	leaq 8(%rax),%rdi
	movq %rcx,8(%rax)
	cmpq $0,%rcx
	jz L78
L76:
	movq 8(%rbx),%rcx
	movq %rdi,16(%rcx)
L78:
	leaq 8(%rbx),%rdi
	movq %rax,8(%rbx)
	movq %rdi,16(%rax)
	addl $1,(%rbx)
L72:
	movq %rsi,%rax
L61:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L83:
_copyps_update:
L85:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L89:
	movl %edx,%ebx
	call _copyps_insert
	movl %ebx,4(%rax)
L87:
	popq %rbx
	popq %rbp
	ret
L91:
_copyps_invalidate:
L93:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L115:
	movq %rdi,%rbx
	movq 8(%rsi),%r12
L96:
	cmpq $0,%r12
	jz L95
L100:
	movq 8(%rbx),%rsi
	movl (%r12),%edi
L101:
	cmpq $0,%rsi
	jz L98
L102:
	movl (%rsi),%eax
	cmpl %edi,%eax
	jz L105
L108:
	movl 4(%rsi),%eax
	cmpl %edi,%eax
	jnz L103
L105:
	movl (%rsi),%esi
	movq %rbx,%rdi
	call _copyps_unset
	jmp L100
L103:
	movq 8(%rsi),%rsi
	jmp L101
L98:
	movq 8(%r12),%r12
	jmp L96
L95:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L117:
_local:
L119:
	pushq %rbp
	movq %rsp,%rbp
	subq $40,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
L137:
	movq %rsi,%r13
	xorps %xmm0,%xmm0
	movups %xmm0,-24(%rbp)
	movq $0,-8(%rbp)
	leaq -16(%rbp),%rsi
	movq %rsi,-8(%rbp)
	movq 8(%rdi),%rbx
L122:
	cmpq $0,%rbx
	jz L121
L123:
	leaq -32(%rbp),%rdx
	leaq -40(%rbp),%rsi
	movq %rbx,%rdi
	call _insn_copy
	cmpl $0,%eax
	jz L127
L126:
	movl -40(%rbp),%esi
	movl -32(%rbp),%edx
	movq %r13,%rdi
	call _copyps_update
	jmp L124
L127:
	movq 8(%r13),%r12
L129:
	cmpq $0,%r12
	jz L132
L130:
	movl 4(%r12),%edx
	movl (%r12),%esi
	movq %rbx,%rdi
	movl $1,%ecx
	call _insn_substitute_reg
	cmpl $0,%eax
	jz L131
L133:
	movl $1,_changed(%rip)
L131:
	movq 8(%r12),%r12
	jmp L129
L132:
	leaq -24(%rbp),%r12
	movq %rbx,%rdi
	movq %r12,%rsi
	xorl %edx,%edx
	call _insn_defs_regs
	movq %r13,%rdi
	movq %r12,%rsi
	call _copyps_invalidate
	movq %r12,%rdi
	call _regs_clear
L124:
	movq 64(%rbx),%rbx
	jmp L122
L121:
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L139:
_local0:
L141:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
L142:
	leaq -16(%rbp),%rbx
	xorps %xmm0,%xmm0
	movups %xmm0,-16(%rbp)
	movq %rbx,%rsi
	call _local
	movq %rbx,%rdi
	call _copyps_clear
	xorl %eax,%eax
L143:
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L148:
.data
.align 8
_copies:
	.quad 0
.align 4
_next_copy_bit:
	.int -1
.text
_copies_new:
L153:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L160:
	movq %rdx,%r12
	movl %esi,%ebx
	movl %edi,%r14d
	movl %ecx,%r13d
	movl $32,%edi
	call _safe_malloc
	movl %r14d,(%rax)
	movl %ebx,4(%rax)
	movq %r12,8(%rax)
	movl %r13d,16(%rax)
	movq _copies(%rip),%rsi
	movq %rsi,24(%rax)
	movq %rax,_copies(%rip)
	addl $1,_nr_copies(%rip)
L155:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L162:
_copies_clear:
L164:
	pushq %rbp
	movq %rsp,%rbp
L167:
	movq _copies(%rip),%rdi
	cmpq $0,%rdi
	jz L169
L170:
	movq 24(%rdi),%rsi
	movq %rsi,_copies(%rip)
	call _free
	jmp L167
L169:
	movl $0,_nr_copies(%rip)
	movl $-1,_next_copy_bit(%rip)
L166:
	popq %rbp
	ret
L176:
_copies_invalidate:
L178:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L193:
	movq %rdi,%r14
	movl %esi,%r13d
	xorl %ebx,%ebx
	movq _copies(%rip),%r12
L181:
	cmpq $0,%r12
	jz L180
L182:
	movl (%r12),%esi
	cmpl %r13d,%esi
	jz L185
L188:
	movl 4(%r12),%esi
	cmpl %r13d,%esi
	jnz L187
L185:
	movq %r14,%rdi
	movl %ebx,%esi
	call _bitset_reset
L187:
	addl $1,%ebx
	movq 24(%r12),%r12
	jmp L181
L180:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L195:
_copies0:
L197:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
	pushq %r12
L209:
	movq %rdi,%rbx
	movq 8(%rbx),%r12
L200:
	cmpq $0,%r12
	jz L203
L201:
	leaq -8(%rbp),%rdx
	leaq -16(%rbp),%rsi
	movq %r12,%rdi
	call _insn_copy
	cmpl $0,%eax
	jz L202
L204:
	movl 8(%r12),%ecx
	movl -16(%rbp),%edi
	movl -8(%rbp),%esi
	movq %rbx,%rdx
	call _copies_new
L202:
	movq 64(%r12),%r12
	jmp L200
L203:
	xorl %eax,%eax
L199:
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L211:
_init0:
L213:
	pushq %rbp
	movq %rsp,%rbp
	subq $40,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
L235:
	movq %rdi,%rbx
	xorps %xmm0,%xmm0
	movups %xmm0,-24(%rbp)
	movq $0,-8(%rbp)
	leaq -16(%rbp),%rsi
	movq %rsi,-8(%rbp)
	movl _nr_copies(%rip),%esi
	leaq 248(%rbx),%rdi
	call _bitset_init
	movl _nr_copies(%rip),%esi
	leaq 280(%rbx),%rdi
	call _bitset_init
	movl _nr_copies(%rip),%esi
	leaq 264(%rbx),%rdi
	call _bitset_init
	movl _nr_copies(%rip),%esi
	leaq 296(%rbx),%r12
	movq %r12,%rdi
	call _bitset_init
	movl _nr_copies(%rip),%esi
	leaq 312(%rbx),%rdi
	call _bitset_init
	movq %r12,%rdi
	call _bitset_one_all
	movl _next_copy_bit(%rip),%esi
	cmpl $-1,%esi
	jnz L218
L216:
	movl _nr_copies(%rip),%esi
	movl %esi,_next_copy_bit(%rip)
L218:
	cmpq %rbx,_entry_block(%rip)
	jz L221
L219:
	leaq 248(%rbx),%rdi
	call _bitset_one_all
L221:
	movq 8(%rbx),%r13
L222:
	cmpq $0,%r13
	jz L225
L223:
	leaq -24(%rbp),%rsi
	movq %r13,%rdi
	xorl %edx,%edx
	call _insn_defs_regs
	movq -16(%rbp),%r12
L226:
	cmpq $0,%r12
	jz L229
L227:
	movl (%r12),%esi
	leaq 264(%rbx),%rdi
	call _copies_invalidate
	movl (%r12),%esi
	leaq 296(%rbx),%rdi
	call _copies_invalidate
	movq 8(%r12),%r12
	jmp L226
L229:
	leaq -24(%rbp),%rdi
	call _regs_clear
	leaq -32(%rbp),%rdx
	leaq -40(%rbp),%rsi
	movq %r13,%rdi
	call _insn_copy
	cmpl $0,%eax
	jz L224
L230:
	movl _next_copy_bit(%rip),%esi
	addl $-1,%esi
	movl %esi,_next_copy_bit(%rip)
	leaq 264(%rbx),%rdi
	call _bitset_set
L224:
	movq 64(%r13),%r13
	jmp L222
L225:
	xorl %eax,%eax
L215:
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L237:
_copy0:
L239:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L255:
	movq %rdi,%rbx
	movq 504(%rbx),%r12
L242:
	cmpq $0,%r12
	jz L245
L243:
	cmpq %r12,504(%rbx)
	jnz L247
L246:
	movq 8(%r12),%rsi
	addq $280,%rsi
	leaq 248(%rbx),%rdi
	call _bitset_copy
	jmp L244
L247:
	movq 8(%r12),%rsi
	addq $280,%rsi
	leaq 248(%rbx),%rdi
	call _bitset_and
L244:
	movq 32(%r12),%r12
	jmp L242
L245:
	leaq 248(%rbx),%rsi
	leaq 312(%rbx),%r12
	movq %r12,%rdi
	call _bitset_copy
	leaq 296(%rbx),%rsi
	movq %r12,%rdi
	call _bitset_and
	leaq 264(%rbx),%rsi
	movq %r12,%rdi
	call _bitset_or
	addq $280,%rbx
	movq %r12,%rdi
	movq %rbx,%rsi
	call _bitset_same
	cmpl $0,%eax
	jz L251
L249:
	xorl %eax,%eax
	jmp L241
L251:
	movq %rbx,%rdi
	movq %r12,%rsi
	call _bitset_copy
	movl $2,%eax
L241:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L257:
_rewrite0:
L259:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
L274:
	movq %rdi,%r13
	xorps %xmm0,%xmm0
	movups %xmm0,-16(%rbp)
	xorl %ebx,%ebx
	movq _copies(%rip),%r12
L262:
	movl _nr_copies(%rip),%esi
	cmpl %esi,%ebx
	jge L265
L263:
	leaq 248(%r13),%rdi
	movl %ebx,%esi
	call _bitset_get
	cmpl $0,%eax
	jz L264
L266:
	movl 4(%r12),%edx
	movl (%r12),%esi
	leaq -16(%rbp),%rdi
	call _copyps_update
L264:
	addl $1,%ebx
	movq 24(%r12),%r12
	jmp L262
L265:
	leaq -16(%rbp),%rbx
	movl -16(%rbp),%esi
	cmpl $0,%esi
	jz L271
L269:
	movq %r13,%rdi
	movq %rbx,%rsi
	call _local
	movq %rbx,%rdi
	call _copyps_clear
L271:
	xorl %eax,%eax
L261:
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L276:
_clear0:
L278:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L283:
	movq %rdi,%rbx
	leaq 248(%rbx),%rdi
	call _bitset_clear
	leaq 280(%rbx),%rdi
	call _bitset_clear
	leaq 264(%rbx),%rdi
	call _bitset_clear
	leaq 296(%rbx),%rdi
	call _bitset_clear
	leaq 312(%rbx),%rdi
	call _bitset_clear
	xorl %eax,%eax
L280:
	popq %rbx
	popq %rbp
	ret
L285:
_copy:
L286:
	pushq %rbp
	movq %rsp,%rbp
L287:
	movl $0,_changed(%rip)
	movq $_local0,%rdi
	call _blocks_iter
	movq $_copies0,%rdi
	call _blocks_iter
	movl _nr_copies(%rip),%esi
	cmpl $0,%esi
	jle L291
L289:
	movq $_init0,%rdi
	call _blocks_iter
	movq $_copy0,%rdi
	call _blocks_iter
	movq $_rewrite0,%rdi
	call _blocks_iter
	movq $_clear0,%rdi
	call _blocks_iter
L291:
	call _copies_clear
	movl _changed(%rip),%esi
	cmpl $0,%esi
	jz L288
L292:
	call _dead
L288:
	popq %rbp
	ret
L298:
.globl _blocks_iter
.globl _bitset_clear
.globl _bitset_or
.globl _regs_clear
.local _nr_copies
.comm _nr_copies, 4, 4
.globl _insn_defs_regs
.globl _bitset_get
.globl _bitset_reset
.globl _bitset_set
.globl _bitset_and
.globl _bitset_init
.globl _bitset_one_all
.globl _safe_malloc
.local _changed
.comm _changed, 4, 4
.globl _dead
.globl _bitset_same
.globl _free
.globl _insn_substitute_reg
.globl _copy
.globl _bitset_copy
.globl _insn_copy
.globl _entry_block
