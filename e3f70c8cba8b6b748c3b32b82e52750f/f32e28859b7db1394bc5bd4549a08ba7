.text
_loop_init:
L1:
	pushq %rbp
	movq %rsp,%rbp
L7:
	leaq 8(%rdi),%rsi
	movq $0,8(%rdi)
	movq %rsi,16(%rdi)
	movl $0,(%rdi)
	movl $0,24(%rdi)
L3:
	popq %rbp
	ret
L13:
_loop_clear:
L14:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L18:
	movq %rdi,%rbx
	call _blks_clear
	movl $0,24(%rbx)
L16:
	popq %rbx
	popq %rbp
	ret
L20:
_bstack_push:
L22:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L29:
	movq %rsi,%r12
	movq %rdi,%rbx
	movl $16,%edi
	call _safe_malloc
	movq %r12,(%rax)
	movq (%rbx),%rsi
	movq %rsi,8(%rax)
	movq %rax,(%rbx)
L24:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L31:
_bstack_pop:
L33:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L41:
	movq %rdi,%rsi
	movq (%rsi),%rdi
	movq 8(%rdi),%rax
	movq %rax,(%rsi)
	movq (%rdi),%rbx
	call _free
	movq %rbx,%rax
L35:
	popq %rbx
	popq %rbp
	ret
L43:
.data
.align 8
_bstack:
	.quad 0
.text
_loop_blocks:
L46:
	pushq %rbp
	movq %rsp,%rbp
	subq $24,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L72:
	movq %rsi,%r15
	movq %rdi,%r12
	leaq -24(%rbp),%rbx
	xorps %xmm0,%xmm0
	movups %xmm0,-24(%rbp)
	movq $0,-8(%rbp)
	leaq -16(%rbp),%rsi
	movq %rsi,-8(%rbp)
	movq %rbx,%rdi
	movq %r15,%rsi
	movl $1,%edx
	call _blks_lookup
	cmpq %r15,%r12
	jz L51
L52:
	movq %rbx,%rdi
	movq %r12,%rsi
	xorl %edx,%edx
	call _blks_lookup
	cmpq $0,%rax
	jnz L58
L55:
	movq %rbx,%rdi
	movq %r12,%rsi
	movl $1,%edx
	call _blks_lookup
	movq $_bstack,%rdi
	movq %r12,%rsi
	call _bstack_push
L58:
	cmpq $0,_bstack(%rip)
	jz L51
L59:
	movq $_bstack,%rdi
	call _bstack_pop
	movq %rax,%r14
	xorl %r13d,%r13d
L61:
	movq %r14,%rdi
	movl %r13d,%esi
	call _block_get_predecessor_n
	movq %rax,%rbx
	cmpq $0,%rbx
	jz L58
L65:
	movq 8(%rbx),%rsi
	leaq -24(%rbp),%r12
	movq %r12,%rdi
	xorl %edx,%edx
	call _blks_lookup
	cmpq $0,%rax
	jnz L63
L68:
	movq 8(%rbx),%rsi
	movq %r12,%rdi
	movl $1,%edx
	call _blks_lookup
	movq 8(%rbx),%rsi
	movq $_bstack,%rdi
	call _bstack_push
L63:
	addl $1,%r13d
	jmp L61
L51:
	leaq -24(%rbp),%rbx
	leaq 456(%r15),%rdi
	movq %rbx,%rsi
	call _blks_union
	movq %rbx,%rdi
	call _blks_clear
L48:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L74:
_loop0:
L76:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L81:
	movq %rdi,%rbx
	leaq 456(%rbx),%rdi
	call _blks_clear
	movl $0,480(%rbx)
	xorl %eax,%eax
L78:
	popq %rbx
	popq %rbp
	ret
L83:
_loop1:
L85:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L97:
	movq %rdi,%r12
	xorl %ebx,%ebx
L88:
	movq %r12,%rdi
	movl %ebx,%esi
	call _block_get_successor_n
	movq %rax,%r13
	cmpq $0,%r13
	jz L91
L89:
	movq 8(%r13),%rsi
	leaq 432(%r12),%rdi
	xorl %edx,%edx
	call _blks_lookup
	cmpq $0,%rax
	jz L90
L92:
	movq 8(%r13),%rsi
	movq %r12,%rdi
	call _loop_blocks
L90:
	addl $1,%ebx
	jmp L88
L91:
	xorl %eax,%eax
L87:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L99:
_loop2:
L101:
	pushq %rbp
	movq %rsp,%rbp
L102:
	movq 464(%rdi),%rsi
L104:
	cmpq $0,%rsi
	jz L107
L105:
	movq (%rsi),%rdi
	addl $1,480(%rdi)
	movq 8(%rsi),%rsi
	jmp L104
L107:
	xorl %eax,%eax
L103:
	popq %rbp
	ret
L112:
_loop_analyze:
L113:
	pushq %rbp
	movq %rsp,%rbp
L114:
	call _dom_analyze
	movq $_loop0,%rdi
	call _blocks_iter
	movq $_loop1,%rdi
	call _blocks_iter
	movq $_loop2,%rdi
	call _blocks_iter
L115:
	popq %rbp
	ret
L119:
.data
.align 8
_invariants:
	.int 0
	.space 4, 0
	.quad 0
	.quad _invariants+8
.align 8
_out_blks:
	.int 0
	.space 4, 0
	.quad 0
	.quad _out_blks+8
.align 8
_maybe_invs:
	.int 0
	.space 12, 0
.text
_maybe_invs_lookup:
L127:
	pushq %rbp
	movq %rsp,%rbp
L128:
	movq 8(%rdi),%rax
L130:
	cmpq $0,%rax
	jz L133
L131:
	movl (%rax),%edi
	cmpl %esi,%edi
	jz L129
L132:
	movq 24(%rax),%rax
	jmp L130
L133:
	xorl %eax,%eax
L129:
	popq %rbp
	ret
L142:
_maybe_invs_insert:
L144:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L166:
	movl %esi,%r12d
	movq %rdi,%rbx
	movq 8(%rbx),%rsi
L147:
	cmpq $0,%rsi
	jz L150
L148:
	movl (%rsi),%edi
	cmpl %r12d,%edi
	jz L150
L149:
	movq 24(%rsi),%rsi
	jmp L147
L150:
	cmpq $0,%rsi
	jnz L157
L155:
	movl $40,%edi
	call _safe_malloc
	movq %rax,%rsi
	movl %r12d,(%rax)
	movq 8(%rbx),%rcx
	leaq 24(%rax),%rdi
	movq %rcx,24(%rax)
	cmpq $0,%rcx
	jz L163
L161:
	movq 8(%rbx),%rcx
	movq %rdi,32(%rcx)
L163:
	leaq 8(%rbx),%rdi
	movq %rax,8(%rbx)
	movq %rdi,32(%rax)
	addl $1,(%rbx)
L157:
	movq %rsi,%rax
L146:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L168:
_maybe_invs_unset:
L170:
	pushq %rbp
	movq %rsp,%rbp
L188:
	movq %rdi,%rcx
	movq 8(%rcx),%rdi
L173:
	cmpq $0,%rdi
	jz L172
L174:
	movl (%rdi),%eax
	cmpl %esi,%eax
	jnz L175
L177:
	addl $-1,(%rcx)
	movq 24(%rdi),%rsi
	cmpq $0,%rsi
	jz L185
L183:
	movq 32(%rdi),%rax
	movq %rax,32(%rsi)
L185:
	movq 24(%rdi),%rsi
	movq 32(%rdi),%rax
	movq %rsi,(%rax)
	call _free
	jmp L172
L175:
	movq 24(%rdi),%rdi
	jmp L173
L172:
	popq %rbp
	ret
L190:
_maybe_invs_clear:
L192:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L205:
	movq %rdi,%rbx
L195:
	movq 8(%rbx),%rdi
	cmpq $0,%rdi
	jz L194
L196:
	addl $-1,(%rbx)
	movq 24(%rdi),%rsi
	cmpq $0,%rsi
	jz L203
L201:
	movq 32(%rdi),%rax
	movq %rax,32(%rsi)
L203:
	movq 24(%rdi),%rsi
	movq 32(%rdi),%rax
	movq %rsi,(%rax)
	call _free
	jmp L195
L194:
	popq %rbx
	popq %rbp
	ret
L207:
_invariants0:
L209:
	pushq %rbp
	movq %rsp,%rbp
L210:
	andl $-3,4(%rdi)
	xorl %eax,%eax
L211:
	popq %rbp
	ret
L216:
_nexthead0:
L218:
	pushq %rbp
	movq %rsp,%rbp
L219:
	movl 456(%rdi),%esi
	cmpl $0,%esi
	jz L223
L228:
	movq _head(%rip),%rax
	cmpq $0,%rax
	jz L224
L232:
	movl 480(%rdi),%esi
	movl 480(%rax),%eax
	cmpl %eax,%esi
	jle L223
L224:
	movl 4(%rdi),%esi
	testl $2,%esi
	jnz L223
L221:
	movq %rdi,_head(%rip)
L223:
	xorl %eax,%eax
L220:
	popq %rbp
	ret
L240:
_unique_def:
L242:
	pushq %rbp
	movq %rsp,%rbp
	subq $48,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L272:
	movl %esi,%r14d
	xorps %xmm0,%xmm0
	movups %xmm0,-24(%rbp)
	movq $0,-8(%rbp)
	leaq -16(%rbp),%rsi
	movq %rsi,-8(%rbp)
	xorps %xmm0,%xmm0
	movups %xmm0,-48(%rbp)
	movq $0,-32(%rbp)
	leaq -40(%rbp),%rsi
	movq %rsi,-32(%rbp)
	xorl %r13d,%r13d
	xorl %ebx,%ebx
	movq 8(%rdi),%r12
L245:
	cmpq $0,%r12
	jz L248
L246:
	leaq -24(%rbp),%rsi
	movq %r12,%rdi
	xorl %edx,%edx
	call _insn_defs_regs
	leaq -48(%rbp),%rdi
	movl %r14d,%esi
	xorl %edx,%edx
	call _regs_lookup
	cmpq $0,%rax
	jz L251
L252:
	cmpq $0,%r13
	jnz L251
L249:
	movl $1,%ebx
L251:
	leaq -24(%rbp),%rdi
	movl %r14d,%esi
	xorl %edx,%edx
	call _regs_lookup
	cmpq $0,%rax
	jz L258
L256:
	cmpq $0,%r13
	jz L260
L259:
	movl $1,%ebx
	jmp L258
L260:
	movq %r12,%r13
L258:
	leaq -24(%rbp),%rdi
	call _regs_clear
	leaq -48(%rbp),%rdi
	call _regs_clear
	cmpl $0,%ebx
	jnz L248
L247:
	movq 64(%r12),%r12
	jmp L245
L248:
	cmpl $0,%ebx
	jz L267
L266:
	xorl %eax,%eax
	jmp L244
L267:
	movq %r13,%rax
L244:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L274:
_loop_scan:
L276:
	pushq %rbp
	movq %rsp,%rbp
	subq $96,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
L277:
	leaq -24(%rbp),%rdx
	xorps %xmm0,%xmm0
	movups %xmm0,-24(%rbp)
	movq $0,-8(%rbp)
	leaq -16(%rbp),%rsi
	movq %rsi,-8(%rbp)
	xorps %xmm0,%xmm0
	movups %xmm0,-48(%rbp)
	movq $0,-32(%rbp)
	leaq -40(%rbp),%rsi
	movq %rsi,-32(%rbp)
	xorps %xmm0,%xmm0
	movups %xmm0,-72(%rbp)
	movq $0,-56(%rbp)
	leaq -64(%rbp),%rsi
	movq %rsi,-56(%rbp)
	xorps %xmm0,%xmm0
	movups %xmm0,-96(%rbp)
	movq $0,-80(%rbp)
	leaq -88(%rbp),%rsi
	movq %rsi,-80(%rbp)
	movq _head(%rip),%rsi
	leaq 456(%rsi),%rdi
	xorl %esi,%esi
	call _kill_gather
	movq -16(%rbp),%rbx
L279:
	cmpq $0,%rbx
	jz L282
L280:
	leaq -96(%rbp),%r12
	leaq -48(%rbp),%rdx
	movl (%rbx),%edi
	movq $_out_blks,%rsi
	movq %r12,%rcx
	call _kill_gather_blks
	leaq -72(%rbp),%rdx
	movq _head(%rip),%rsi
	movl (%rbx),%edi
	addq $456,%rsi
	movq %r12,%rcx
	call _kill_gather_blks
	movl -72(%rbp),%esi
	cmpl $0,%esi
	jnz L285
L283:
	movl (%rbx),%esi
	movq $_invariants,%rdi
	movl $1,%edx
	call _regs_lookup
	jmp L286
L285:
	cmpl $0,%esi
	jz L290
L291:
	movl -48(%rbp),%esi
	cmpl $0,%esi
	jnz L286
L290:
	movl -72(%rbp),%esi
	cmpl $1,%esi
	ja L286
L298:
	movq -64(%rbp),%r13
	movl (%rbx),%esi
	movq (%r13),%rdi
	call _unique_def
	movq %rax,%r12
	cmpq $0,%r12
	jz L286
L303:
	leaq -96(%rbp),%rsi
	movq (%r13),%rdi
	call _dominates_all
	cmpl $0,%eax
	jz L286
L302:
	movl (%r12),%esi
	testl $65536,%esi
	jz L286
L310:
	movl 4(%r12),%esi
	testl $1,%esi
	jnz L286
L314:
	movq %r12,%rdi
	call _insn_defs_mem
	cmpl $0,%eax
	jnz L286
L318:
	movl _load_ok(%rip),%esi
	cmpl $0,%esi
	jnz L322
L323:
	movq %r12,%rdi
	call _insn_uses_mem
	cmpl $0,%eax
	jnz L286
L322:
	movl (%rbx),%esi
	movq $_maybe_invs,%rdi
	call _maybe_invs_insert
	movq (%r13),%rsi
	movq %rsi,8(%rax)
	movq %r12,16(%rax)
L286:
	leaq -48(%rbp),%rdi
	call _blks_clear
	leaq -72(%rbp),%rdi
	call _blks_clear
	leaq -96(%rbp),%rdi
	call _blks_clear
	movq 8(%rbx),%rbx
	jmp L279
L282:
	leaq -24(%rbp),%rdi
	call _regs_clear
L278:
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L331:
_make_preheader:
L334:
	pushq %rbp
	movq %rsp,%rbp
L335:
	cmpq $0,_preheader(%rip)
	jnz L336
L337:
	movq _head(%rip),%rdi
	leaq 456(%rdi),%rsi
	call _block_preheader
	movq %rax,_preheader(%rip)
	movl $1,_cfg_changed(%rip)
L336:
	popq %rbp
	ret
L343:
_movable:
L345:
	pushq %rbp
	movq %rsp,%rbp
	subq $24,%rsp
	pushq %rbx
L346:
	xorps %xmm0,%xmm0
	movups %xmm0,-24(%rbp)
	movq $0,-8(%rbp)
	leaq -16(%rbp),%rsi
	movq %rsi,-8(%rbp)
	movl (%rdi),%esi
	cmpl $1879244817,%esi
	jz L354
L366:
	cmpl $1879244822,%esi
	jz L354
L367:
	cmpl $1880293392,%esi
	jnz L350
L354:
	movq 40(%rdi),%rsi
	movq 8(%rsi),%rsi
	movq _target(%rip),%rcx
	movq 8(%rcx),%rax
	movq 16(%rcx),%rcx
	orq %rcx,%rax
	orq $192,%rax
	testq %rax,%rsi
	jz L350
L355:
	xorl %eax,%eax
	jmp L347
L350:
	leaq -24(%rbp),%rbx
	movq %rbx,%rsi
	xorl %edx,%edx
	call _insn_uses_regs
	movq %rbx,%rdi
	movq $_invariants,%rsi
	call _regs_diff
	movl -24(%rbp),%esi
	cmpl $0,%esi
	jnz L360
L359:
	movl $1,%eax
	jmp L347
L360:
	movq %rbx,%rdi
	call _regs_clear
	xorl %eax,%eax
L347:
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L371:
_loop_motion:
L373:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L376:
	xorl %r13d,%r13d
	movq _maybe_invs+8(%rip),%rbx
L379:
	cmpq $0,%rbx
	jz L378
L380:
	movq 24(%rbx),%r12
	movq 16(%rbx),%rdi
	call _movable
	cmpl $0,%eax
	jz L381
L383:
	call _make_preheader
	movq 16(%rbx),%rdi
	call _insn_dup
	movq _preheader(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	movq 16(%rbx),%rsi
	movq 8(%rbx),%rdi
	addq $8,%rdi
	call _insns_remove
	movl (%rbx),%esi
	movq $_invariants,%rdi
	movl $1,%edx
	call _regs_lookup
	movl (%rbx),%esi
	movq $_maybe_invs,%rdi
	call _maybe_invs_unset
	movl $1,%r13d
L381:
	movq %r12,%rbx
	jmp L379
L378:
	cmpl $0,%r13d
	jnz L376
L375:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L389:
_loop_invariants:
L390:
	pushq %rbp
	movq %rsp,%rbp
L391:
	call _webs_analyze
	movq $_invariants0,%rdi
	call _blocks_iter
	movl $1,_cfg_changed(%rip)
L394:
	movl _cfg_changed(%rip),%esi
	cmpl $0,%esi
	jz L399
L397:
	call _kill_analyze
	call _loop_analyze
	movl $0,_cfg_changed(%rip)
L399:
	movq $0,_head(%rip)
	movq $_nexthead0,%rdi
	call _blocks_iter
	movq _head(%rip),%rsi
	cmpq $0,%rsi
	jnz L401
L396:
	call _webs_strip
L392:
	popq %rbp
	ret
L401:
	orl $2,4(%rsi)
	movq $0,_preheader(%rip)
	movq _head(%rip),%rsi
	leaq 456(%rsi),%rdi
	call _blocks_def_mem
	cmpl $0,%eax
	setz %sil
	movzbl %sil,%esi
	movl %esi,_load_ok(%rip)
	movq $_out_blks,%rdi
	call _blks_all
	movq _head(%rip),%rsi
	addq $456,%rsi
	movq $_out_blks,%rdi
	call _blks_diff
	call _loop_scan
	call _loop_motion
	movq $_out_blks,%rdi
	call _blks_clear
	movq $_invariants,%rdi
	call _regs_clear
	movq $_maybe_invs,%rdi
	call _maybe_invs_clear
	jmp L394
L407:
.globl _blks_lookup
.globl _webs_strip
.globl _regs_lookup
.globl _insn_dup
.local _preheader
.comm _preheader, 8, 8
.globl _blocks_iter
.globl _block_preheader
.globl _loop_clear
.globl _blks_clear
.globl _kill_gather
.globl _regs_clear
.globl _kill_gather_blks
.globl _insn_uses_regs
.globl _insn_defs_regs
.globl _target
.globl _loop_init
.globl _insn_append
.globl _dom_analyze
.globl _loop_analyze
.globl _kill_analyze
.globl _webs_analyze
.globl _insns_remove
.globl _blks_diff
.globl _regs_diff
.globl _dominates_all
.globl _blks_all
.globl _loop_invariants
.globl _safe_malloc
.local _cfg_changed
.comm _cfg_changed, 4, 4
.local _head
.comm _head, 8, 8
.globl _free
.local _load_ok
.comm _load_ok, 4, 4
.globl _blocks_def_mem
.globl _insn_uses_mem
.globl _insn_defs_mem
.globl _block_get_predecessor_n
.globl _block_get_successor_n
.globl _blks_union
