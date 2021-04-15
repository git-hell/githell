.text
_cessor_new:
L3:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L11:
	movq %rdx,%r13
	movq %rdi,%r12
	movl %esi,%ebx
	movl $48,%edi
	call _safe_malloc
	movl %ebx,(%rax)
	movq %r13,8(%rax)
	leaq 32(%rax),%rsi
	movq $0,32(%rax)
	movq 8(%r12),%rdi
	movq %rdi,40(%rax)
	movq 8(%r12),%rdi
	movq %rax,(%rdi)
	movq %rsi,8(%r12)
L5:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L13:
_cessor_free:
L15:
	pushq %rbp
	movq %rsp,%rbp
L18:
	movq 32(%rsi),%rax
	cmpq $0,%rax
	jz L22
L21:
	movq 40(%rsi),%rdi
	movq %rdi,40(%rax)
	jmp L23
L22:
	movq 40(%rsi),%rax
	movq %rax,8(%rdi)
L23:
	movq 32(%rsi),%rdi
	movq 40(%rsi),%rax
	movq %rdi,(%rax)
	movq %rsi,%rdi
	call _free
L17:
	popq %rbp
	ret
L27:
_block_new:
L28:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L29:
	movl $552,%edi
	call _safe_malloc
	movq %rax,%rbx
	movl _last_asm_label(%rip),%esi
	addl $1,%esi
	movl %esi,_last_asm_label(%rip)
	movl %esi,(%rbx)
	leaq 8(%rbx),%rsi
	movq $0,8(%rbx)
	movq %rsi,16(%rbx)
	movl $2,24(%rbx)
	leaq 32(%rbx),%rdi
	call _live_init
	leaq 168(%rbx),%rdi
	call _kill_init
	leaq 456(%rbx),%rdi
	call _loop_init
	leaq 440(%rbx),%rsi
	movq $0,440(%rbx)
	movq %rsi,448(%rbx)
	movl $0,432(%rbx)
	leaq 488(%rbx),%rsi
	movq $0,488(%rbx)
	movq %rsi,496(%rbx)
	leaq 504(%rbx),%rsi
	movq $0,504(%rbx)
	movq %rsi,512(%rbx)
	leaq 536(%rbx),%rsi
	movq $0,536(%rbx)
	movq _blocks+8(%rip),%rdi
	movq %rdi,544(%rbx)
	movq _blocks+8(%rip),%rdi
	movq %rbx,(%rdi)
	movq %rsi,_blocks+8(%rip)
	movq %rbx,%rax
L30:
	popq %rbx
	popq %rbp
	ret
L56:
_block_free:
L57:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L70:
	movq %rdi,%rbx
	leaq 8(%rbx),%rdi
	call _insns_clear
	leaq 32(%rbx),%rdi
	call _live_clear
	leaq 168(%rbx),%rdi
	call _kill_clear
	leaq 432(%rbx),%rdi
	call _blks_clear
	leaq 456(%rbx),%rdi
	call _loop_clear
	movq 352(%rbx),%rdi
	call _operand_free
L60:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _block_get_successor_n
	cmpq $0,%rax
	jz L63
L61:
	movq %rbx,%rdi
	movq %rax,%rsi
	call _block_remove_successor
	jmp L60
L63:
	movq 536(%rbx),%rsi
	cmpq $0,%rsi
	jz L67
L66:
	movq 544(%rbx),%rdi
	movq %rdi,544(%rsi)
	jmp L68
L67:
	movq 544(%rbx),%rsi
	movq %rsi,_blocks+8(%rip)
L68:
	movq 536(%rbx),%rsi
	movq 544(%rbx),%rdi
	movq %rsi,(%rdi)
	movq %rbx,%rdi
	call _free
L59:
	popq %rbx
	popq %rbp
	ret
L72:
_block_add_successor:
L73:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L78:
	movq %rdi,%r12
	movq %rdx,%rbx
	leaq 488(%r12),%rdi
	movq %rbx,%rdx
	call _cessor_new
	movq %rax,%r13
	leaq 504(%rbx),%rdi
	movl $11,%esi
	movq %r12,%rdx
	call _cessor_new
	movq %r13,%rax
L75:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L80:
_block_find_predecessor:
L82:
	pushq %rbp
	movq %rsp,%rbp
L83:
	movq 504(%rdi),%rax
L85:
	cmpq 8(%rax),%rsi
	jz L84
L87:
	movq 32(%rax),%rax
	jmp L85
L84:
	popq %rbp
	ret
L93:
_block_remove_successor:
L94:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L98:
	movq %rdi,%r12
	movq 8(%rsi),%rbx
	leaq 488(%r12),%rdi
	call _cessor_free
	movq %rbx,%rdi
	movq %r12,%rsi
	call _block_find_predecessor
	leaq 504(%rbx),%rdi
	movq %rax,%rsi
	call _cessor_free
L96:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L100:
_block_remove_successors:
L101:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L108:
	movq %rdi,%rbx
L104:
	movq 488(%rbx),%rsi
	cmpq $0,%rsi
	jz L106
L105:
	movq %rbx,%rdi
	call _block_remove_successor
	jmp L104
L106:
	movq 352(%rbx),%rdi
	call _operand_free
	movq $0,352(%rbx)
L103:
	popq %rbx
	popq %rbp
	ret
L110:
_block_switch_sort:
L112:
	pushq %rbp
	movq %rsp,%rbp
L115:
	movq 488(%rdi),%rsi
L116:
	cmpq $0,%rsi
	jz L114
L117:
	xorl %ecx,%ecx
	movl (%rsi),%eax
	cmpl $12,%eax
	jnz L118
L123:
	movq 32(%rsi),%rax
	cmpq $0,%rax
	jz L125
L130:
	movl (%rax),%edx
	cmpl $12,%edx
	jnz L125
L126:
	movq 16(%rax),%rdx
	cmpq 16(%rsi),%rdx
	jae L125
L134:
	cmpq $0,%rax
	jz L138
L137:
	movq 40(%rsi),%rcx
	movq %rcx,40(%rax)
	jmp L139
L138:
	movq 40(%rsi),%rcx
	movq %rcx,496(%rdi)
L139:
	leaq 32(%rsi),%rcx
	movq 32(%rsi),%rdx
	movq 40(%rsi),%r8
	movq %rdx,(%r8)
	movq 32(%rax),%rdx
	movq %rdx,32(%rsi)
	cmpq $0,%rdx
	jz L144
L143:
	movq %rcx,40(%rdx)
	jmp L145
L144:
	movq %rcx,496(%rdi)
L145:
	leaq 32(%rax),%rcx
	movq %rsi,32(%rax)
	movq %rcx,40(%rsi)
	movl $1,%ecx
	jmp L123
L125:
	cmpl $0,%ecx
	jnz L115
L118:
	movq 32(%rsi),%rsi
	jmp L116
L114:
	popq %rbp
	ret
L153:
_block_switch_coalesce:
L155:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L181:
	movq %rdi,%r12
	call _block_switch_sort
	movq 488(%r12),%rbx
L158:
	cmpq $0,%rbx
	jz L157
L159:
	movl (%rbx),%esi
	cmpl $12,%esi
	jnz L160
L162:
	movq 32(%rbx),%rsi
	cmpq $0,%rsi
	jz L160
L176:
	movl (%rsi),%edi
	cmpl $12,%edi
	jnz L160
L172:
	movq 8(%rbx),%rdi
	cmpq 8(%rsi),%rdi
	jnz L160
L168:
	movq 16(%rsi),%rdi
	movq 24(%rbx),%rax
	addq $1,%rax
	cmpq %rax,%rdi
	jnz L160
L165:
	movq 24(%rsi),%rdi
	movq %rdi,24(%rbx)
	movq %r12,%rdi
	call _block_remove_successor
L160:
	movq 32(%rbx),%rbx
	jmp L158
L157:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L183:
_block_coalesce_successors:
L185:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L207:
	movq %rdi,%rbx
	cmpq $0,352(%rbx)
	jz L190
L188:
	movq %rbx,%rdi
	call _block_switch_coalesce
L190:
	movq 488(%rbx),%rdi
	movq %rdi,%rsi
	cmpq $0,%rdi
	jz L187
L194:
	movl (%rdi),%eax
	cmpl $10,%eax
	jz L187
L193:
	movq 8(%rdi),%r12
L199:
	movq 32(%rsi),%rdi
	movq %rdi,%rsi
	cmpq $0,%rdi
	jz L201
L200:
	cmpq 8(%rdi),%r12
	jz L199
	jnz L187
L201:
	movq %rbx,%rdi
	call _block_remove_successors
	movq %rbx,%rdi
	movl $10,%esi
	movq %r12,%rdx
	call _block_add_successor
	movq 352(%rbx),%rdi
	call _operand_free
	movq $0,352(%rbx)
L187:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L209:
_block_known_ccs:
L210:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L224:
	movq %rdi,%rbx
	movl %esi,%r13d
	movq %rbx,%rdi
	call _block_conditional
	cmpl $0,%eax
	jz L212
L213:
	movq 488(%rbx),%rsi
L216:
	cmpq $0,%rsi
	jz L212
L217:
	movq 32(%rsi),%r12
	movl (%rsi),%ecx
	movl $1,%edi
	shll %cl,%edi
	testl %edi,%r13d
	jz L221
L220:
	movl $10,(%rsi)
	jmp L218
L221:
	movq %rbx,%rdi
	call _block_remove_successor
L218:
	movq %r12,%rsi
	jmp L216
L212:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L226:
_block_redirect_successors:
L227:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L247:
	movq %rdi,%r14
	movq %rdx,%r13
	movq %rsi,%r12
	movq 488(%r14),%rbx
L230:
	cmpq $0,%rbx
	jz L233
L231:
	cmpq 8(%rbx),%r12
	jnz L232
L234:
	movq %r12,%rdi
	movq %r14,%rsi
	call _block_find_predecessor
	movq 32(%rax),%rsi
	cmpq $0,%rsi
	jz L241
L240:
	movq 40(%rax),%rdi
	movq %rdi,40(%rsi)
	jmp L242
L241:
	movq 40(%rax),%rsi
	movq %rsi,512(%r12)
L242:
	leaq 32(%rax),%rsi
	movq 32(%rax),%rdi
	movq 40(%rax),%rcx
	movq %rdi,(%rcx)
	movq $0,32(%rax)
	movq 512(%r13),%rdi
	movq %rdi,40(%rax)
	movq 512(%r13),%rdi
	movq %rax,(%rdi)
	movq %rsi,512(%r13)
	movq %r13,8(%rbx)
L232:
	movq 32(%rbx),%rbx
	jmp L230
L233:
	movq %r14,%rdi
	call _block_coalesce_successors
L229:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L249:
_block_preheader:
L250:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L267:
	movq %rsi,%r14
	movq %rdi,%r12
	call _block_new
	movq %rax,%rbx
	movq %rax,%rdi
	movl $10,%esi
	movq %r12,%rdx
	call _block_add_successor
	movq _blocks(%rip),%r13
L253:
	cmpq $0,%r13
	jz L256
L254:
	cmpq %r13,%rbx
	jz L255
L260:
	movq %r14,%rdi
	movq %r13,%rsi
	xorl %edx,%edx
	call _blks_lookup
	cmpq $0,%rax
	jnz L255
L259:
	movq %r13,%rdi
	movq %r12,%rsi
	movq %rbx,%rdx
	call _block_redirect_successors
L255:
	movq 536(%r13),%r13
	jmp L253
L256:
	movq %rbx,%rax
L252:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L269:
_block_conditional:
L270:
	pushq %rbp
	movq %rsp,%rbp
L271:
	xorl %esi,%esi
	call _block_get_successor_n
	cmpq $0,%rax
	jz L274
L276:
	movl (%rax),%esi
	cmpl $10,%esi
	jge L274
L273:
	movl $1,%eax
	jmp L272
L274:
	xorl %eax,%eax
L272:
	popq %rbp
	ret
L285:
_block_ccs:
L286:
	pushq %rbp
	movq %rsp,%rbp
L287:
	xorl %eax,%eax
	movq 488(%rdi),%rdi
L289:
	cmpq $0,%rdi
	jz L288
L290:
	movl (%rdi),%ecx
	cmpl $10,%ecx
	jge L291
L293:
	movl $1,%esi
	shll %cl,%esi
	orl %esi,%eax
L291:
	movq 32(%rdi),%rdi
	jmp L289
L288:
	popq %rbp
	ret
L300:
_block_switch:
L301:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L305:
	movq %rdi,%rbx
	movq %rsi,%r12
	movq %rbx,%rdi
	movl $13,%esi
	call _block_add_successor
	movq %r12,%rdi
	call _operand_leaf
	movq %rax,352(%rbx)
L303:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L307:
_block_switch_case:
L308:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L319:
	movq %rdi,%r14
	movq %rsi,%r12
	movq %rdx,%r13
	movq 488(%r14),%rbx
L311:
	movq 32(%rbx),%rsi
	movq %rsi,%rbx
	cmpq $0,%rsi
	jz L313
L312:
	cmpq %r12,16(%rsi)
	jnz L311
L314:
	pushq $L317
	pushq $1
	call _error
	addq $16,%rsp
	jmp L311
L313:
	movq %r14,%rdi
	movl $12,%esi
	movq %r13,%rdx
	call _block_add_successor
	movq %r12,24(%rax)
	movq %r12,16(%rax)
L310:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L321:
_block_switch_done:
L322:
	pushq %rbp
	movq %rsp,%rbp
L323:
	call _block_coalesce_successors
L324:
	popq %rbp
	ret
L328:
_block_switch_lookup:
L329:
	pushq %rbp
	movq %rsp,%rbp
L330:
	movq 488(%rdi),%rax
	movq %rax,%rdi
	movq 8(%rax),%rax
L332:
	movq 32(%rdi),%rcx
	movq %rcx,%rdi
	cmpq $0,%rcx
	jz L331
L333:
	cmpq 16(%rcx),%rsi
	jb L332
L338:
	cmpq 24(%rcx),%rsi
	ja L332
L335:
	movq 8(%rcx),%rax
L331:
	popq %rbp
	ret
L347:
_get_cessor_n:
L349:
	pushq %rbp
	movq %rsp,%rbp
L350:
	movq (%rdi),%rax
L352:
	cmpl $0,%esi
	jz L351
L356:
	cmpq $0,%rax
	jz L351
L354:
	addl $-1,%esi
	movq 32(%rax),%rax
	jmp L352
L351:
	popq %rbp
	ret
L364:
_block_get_successor_n:
L365:
	pushq %rbp
	movq %rsp,%rbp
L366:
	addq $488,%rdi
	call _get_cessor_n
L367:
	popq %rbp
	ret
L372:
_block_get_predecessor_n:
L373:
	pushq %rbp
	movq %rsp,%rbp
L374:
	addq $504,%rdi
	call _get_cessor_n
L375:
	popq %rbp
	ret
L380:
_nr_cessors:
L382:
	pushq %rbp
	movq %rsp,%rbp
L383:
	xorl %eax,%eax
	movq (%rdi),%rsi
L385:
	cmpq $0,%rsi
	jz L384
L387:
	addl $1,%eax
	movq 32(%rsi),%rsi
	jmp L385
L384:
	popq %rbp
	ret
L393:
_block_nr_successors:
L394:
	pushq %rbp
	movq %rsp,%rbp
L395:
	addq $488,%rdi
	call _nr_cessors
L396:
	popq %rbp
	ret
L401:
_block_nr_predecessors:
L402:
	pushq %rbp
	movq %rsp,%rbp
L403:
	addq $504,%rdi
	call _nr_cessors
L404:
	popq %rbp
	ret
L409:
_block_cc_successor:
L410:
	pushq %rbp
	movq %rsp,%rbp
L411:
	movq 488(%rdi),%rax
L413:
	cmpq $0,%rax
	jz L416
L414:
	movl (%rax),%edi
	cmpl %esi,%edi
	jz L412
L415:
	movq 32(%rax),%rax
	jmp L413
L416:
	xorl %eax,%eax
L412:
	popq %rbp
	ret
L425:
_block_always_successor:
L426:
	pushq %rbp
	movq %rsp,%rbp
L427:
	movq 488(%rdi),%rax
	cmpq $0,%rax
	jz L430
L432:
	movl (%rax),%esi
	cmpl $10,%esi
	jz L428
L430:
	xorl %eax,%eax
L428:
	popq %rbp
	ret
L441:
_block_sole_predecessor:
L442:
	pushq %rbp
	movq %rsp,%rbp
L443:
	movq 504(%rdi),%rax
	cmpq $0,%rax
	jz L446
L448:
	cmpq $0,32(%rax)
	jz L444
L446:
	xorl %eax,%eax
L444:
	popq %rbp
	ret
L457:
_block_rewrite_zs:
L458:
	pushq %rbp
	movq %rsp,%rbp
L459:
	xorl %eax,%eax
	movq 488(%rdi),%rdi
	movl %esi,%edx
	xorl $1,%edx
L461:
	cmpq $0,%rdi
	jz L460
L462:
	movl (%rdi),%ecx
	cmpl $0,%ecx
	jz L469
L479:
	cmpl $1,%ecx
	jnz L460
L471:
	movl %esi,(%rdi)
	jmp L467
L469:
	movl %edx,(%rdi)
L467:
	movl $1,%eax
	movq 32(%rdi),%rdi
	jmp L461
L460:
	popq %rbp
	ret
L483:
_block_dup_successors:
L484:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L495:
	movq %rsi,%r12
	movq %rdi,%rbx
	call _block_remove_successors
	movq 488(%r12),%r13
L487:
	cmpq $0,%r13
	jz L490
L488:
	movq 8(%r13),%rdx
	movl (%r13),%esi
	movq %rbx,%rdi
	call _block_add_successor
	movq 32(%r13),%r13
	jmp L487
L490:
	movq 352(%r12),%rdi
	cmpq $0,%rdi
	jz L486
L491:
	call _operand_dup
	movq %rax,352(%rbx)
L486:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L497:
_block_substitute_reg:
L498:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L510:
	movl %edx,%r14d
	movl %esi,%r13d
	xorl %ebx,%ebx
	movq 8(%rdi),%r12
L501:
	cmpq $0,%r12
	jz L504
L502:
	movq %r12,%rdi
	movl %r13d,%esi
	movl %r14d,%edx
	movl $3,%ecx
	call _insn_substitute_reg
	cmpl $0,%eax
	jz L503
L505:
	movl $1,%ebx
L503:
	movq 64(%r12),%r12
	jmp L501
L504:
	movl %ebx,%eax
L500:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L512:
_blocks_substitute_reg:
L513:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L525:
	movl %esi,%r14d
	movl %edi,%r13d
	xorl %ebx,%ebx
	movq _blocks(%rip),%r12
L516:
	cmpq $0,%r12
	jz L519
L517:
	movq %r12,%rdi
	movl %r13d,%esi
	movl %r14d,%edx
	call _block_substitute_reg
	cmpl $0,%eax
	jz L518
L520:
	movl $1,%ebx
L518:
	movq 536(%r12),%r12
	jmp L516
L519:
	movl %ebx,%eax
L515:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L527:
_block_split:
L528:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L533:
	movq %rdi,%r12
	movl %esi,%r13d
	call _block_new
	movq %rax,%rbx
	movq %rbx,%rdi
	movq %r12,%rsi
	call _block_dup_successors
	movq %r12,%rdi
	call _block_remove_successors
	movq %r12,%rdi
	movl $10,%esi
	movq %rbx,%rdx
	call _block_add_successor
	movl 24(%r12),%edx
	subl %r13d,%edx
	leaq 8(%r12),%rsi
	leaq 8(%rbx),%rdi
	call _insns_push
	movq %rbx,%rax
L530:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L535:
_block_split_edge:
L536:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L541:
	movq %rdi,%r13
	movq %rsi,%r15
	movl (%r15),%r14d
	movq 8(%r15),%rbx
	call _block_new
	movq %rax,%r12
	movq %r12,%rdi
	movl $10,%esi
	movq %rbx,%rdx
	call _block_add_successor
	movq %r13,%rdi
	movq %r15,%rsi
	call _block_remove_successor
	movq %r13,%rdi
	movl %r14d,%esi
	movq %r12,%rdx
	call _block_add_successor
	movq %r12,%rax
L538:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L543:
_block_defs_mem:
L544:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L545:
	movq 8(%rdi),%rbx
L547:
	cmpq $0,%rbx
	jz L550
L548:
	movq %rbx,%rdi
	call _insn_defs_mem
	cmpl $0,%eax
	jz L549
L551:
	movl $1,%eax
	jmp L546
L549:
	movq 64(%rbx),%rbx
	jmp L547
L550:
	xorl %eax,%eax
L546:
	popq %rbx
	popq %rbp
	ret
L559:
_blocks_def_mem:
L560:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L561:
	movq 8(%rdi),%rbx
L563:
	cmpq $0,%rbx
	jz L566
L564:
	movq (%rbx),%rdi
	call _block_defs_mem
	cmpl $0,%eax
	jz L565
L567:
	movl $1,%eax
	jmp L562
L565:
	movq 8(%rbx),%rbx
	jmp L563
L566:
	xorl %eax,%eax
L562:
	popq %rbx
	popq %rbp
	ret
L575:
_walk0:
L577:
	pushq %rbp
	movq %rsp,%rbp
L578:
	andl $-2,4(%rdi)
	xorl %eax,%eax
L579:
	popq %rbp
	ret
L584:
_walk1:
L586:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L603:
	movq %rdi,%r14
	movq %rdx,%r12
	movq %rsi,%rbx
	movl 4(%r14),%esi
	testl $1,%esi
	jnz L588
L589:
	orl $1,%esi
	movl %esi,4(%r14)
	cmpq $0,%rbx
	jz L594
L592:
	movq %r14,%rdi
	call *%rbx
L594:
	xorl %r13d,%r13d
L595:
	movq %r14,%rdi
	movl %r13d,%esi
	call _block_get_successor_n
	cmpq $0,%rax
	jz L598
L596:
	movq 8(%rax),%rdi
	movq %rbx,%rsi
	movq %r12,%rdx
	call _walk1
	addl $1,%r13d
	jmp L595
L598:
	cmpq $0,%r12
	jz L588
L599:
	movq %r14,%rdi
	call *%r12
L588:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L605:
_blocks_walk:
L606:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L610:
	movq %rsi,%rbx
	movq %rdi,%r12
	movq $_walk0,%rdi
	call _blocks_iter
	movq _entry_block(%rip),%rdi
	movq %r12,%rsi
	movq %rbx,%rdx
	call _walk1
L608:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L612:
_blocks_iter:
L613:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L635:
	movq %rdi,%r13
L616:
	xorl %ebx,%ebx
	movq _blocks(%rip),%rdi
L619:
	cmpq $0,%rdi
	jz L618
L620:
	movq 536(%rdi),%r12
	call *%r13
	cmpl $1,%eax
	jz L627
L633:
	cmpl $2,%eax
	jnz L621
L629:
	movl $1,%ebx
L621:
	movq %r12,%rdi
	jmp L619
L627:
	movl $1,%eax
	jmp L615
L618:
	cmpl $0,%ebx
	jnz L616
L617:
	xorl %eax,%eax
L615:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L637:
_sequence0:
L639:
	pushq %rbp
	movq %rsp,%rbp
L642:
	movq 536(%rdi),%rsi
	cmpq $0,%rsi
	jz L646
L645:
	movq 544(%rdi),%rax
	movq %rax,544(%rsi)
	jmp L647
L646:
	movq 544(%rdi),%rsi
	movq %rsi,_blocks+8(%rip)
L647:
	leaq 536(%rdi),%rsi
	movq 536(%rdi),%rax
	movq 544(%rdi),%rcx
	movq %rax,(%rcx)
	movq _blocks(%rip),%rax
	movq %rax,536(%rdi)
	cmpq $0,%rax
	jz L652
L651:
	movq _blocks(%rip),%rax
	movq %rsi,544(%rax)
	jmp L653
L652:
	movq %rsi,_blocks+8(%rip)
L653:
	movq %rdi,_blocks(%rip)
	movq $_blocks,544(%rdi)
L641:
	popq %rbp
	ret
L657:
_blocks_sequence:
L658:
	pushq %rbp
	movq %rsp,%rbp
L662:
	xorl %edi,%edi
	movq $_sequence0,%rsi
	call _blocks_walk
L660:
	popq %rbp
	ret
L664:
_func_new:
L665:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
L666:
	leaq -16(%rbp),%rbx
	xorps %xmm0,%xmm0
	movups %xmm0,-16(%rbp)
	movq %rbx,-8(%rbp)
	movq %rdi,_func_sym(%rip)
	movq $0,_func_ret_type(%rip)
	movq $_func_ret_type,_func_ret_type+8(%rip)
	leaq 32(%rdi),%rsi
	movq $_func_ret_type,%rdi
	call _type_deref
	movq $0,_blocks(%rip)
	movq $_blocks,_blocks+8(%rip)
	call _block_new
	movq %rax,_entry_block(%rip)
	call _block_new
	movq %rax,_current_block(%rip)
	movq _entry_block(%rip),%rdi
	movl $10,%esi
	movq %rax,%rdx
	call _block_add_successor
	call _block_new
	movq %rax,_exit_block(%rip)
	movq _target(%rip),%rsi
	movq 32(%rsi),%rsi
	call *%rsi
	movq _func_ret_type(%rip),%rsi
	movq (%rsi),%rsi
	testq $65536,%rsi
	jz L675
L674:
	movq %rbx,%rdi
	movq $_func_ret_type,%rsi
	call _type_ref
	movq %rbx,%rdi
	call _symbol_temp
	movq %rax,_func_strun_ret(%rip)
	orl $134217728,12(%rax)
	movq _target(%rip),%rsi
	movq 80(%rsi),%rsi
	movq _func_strun_ret(%rip),%rdi
	call *%rsi
	movq %rbx,%rdi
	call _type_clear
	jmp L680
L675:
	movq $0,_func_strun_ret(%rip)
L680:
	movq $0,_func_regs+8(%rip)
	movq $_func_regs+8,_func_regs+16(%rip)
	movl $0,_func_regs(%rip)
L667:
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L686:
_func_free:
L687:
	pushq %rbp
	movq %rsp,%rbp
L690:
	movq _blocks(%rip),%rdi
	cmpq $0,%rdi
	jz L692
L691:
	call _block_free
	jmp L690
L692:
	movq $_func_ret_type,%rdi
	call _type_clear
	movq $0,_func_strun_ret(%rip)
	movq $_func_regs,%rdi
	call _regs_clear
L689:
	popq %rbp
	ret
L696:
L317:
	.byte 100,117,112,108,105,99,97,116
	.byte 101,32,99,97,115,101,32,108
	.byte 97,98,101,108,0
.globl _block_switch_lookup
.globl _blks_lookup
.globl _operand_dup
.globl _symbol_temp
.globl _blocks_iter
.globl _block_remove_successor
.globl _block_sole_predecessor
.globl _block_always_successor
.globl _block_cc_successor
.globl _block_add_successor
.globl _block_preheader
.globl _loop_clear
.globl _blks_clear
.globl _kill_clear
.globl _live_clear
.globl _regs_clear
.globl _insns_clear
.globl _type_clear
.globl _error
.local _blocks
.comm _blocks, 16, 8
.comm _func_regs, 24, 8
.globl _func_regs
.globl _block_known_ccs
.globl _block_ccs
.globl _target
.comm _func_strun_ret, 8, 8
.globl _func_strun_ret
.globl _block_split
.globl _loop_init
.globl _kill_init
.globl _live_init
.comm _func_ret_type, 16, 8
.globl _func_ret_type
.globl _block_switch_done
.globl _func_new
.globl _block_new
.globl _blocks_walk
.globl _block_nr_predecessors
.globl _block_nr_successors
.globl _block_dup_successors
.globl _block_rewrite_zs
.globl _block_remove_successors
.globl _block_redirect_successors
.globl _safe_malloc
.globl _func_free
.globl _blocks_sequence
.globl _block_switch_case
.globl _block_split_edge
.globl _block_free
.globl _operand_free
.globl _free
.globl _operand_leaf
.globl _type_deref
.globl _type_ref
.globl _blocks_substitute_reg
.globl _block_substitute_reg
.globl _insn_substitute_reg
.globl _block_switch
.globl _insns_push
.comm _current_block, 8, 8
.globl _current_block
.comm _exit_block, 8, 8
.globl _exit_block
.comm _entry_block, 8, 8
.globl _entry_block
.globl _block_conditional
.globl _last_asm_label
.comm _func_sym, 8, 8
.globl _func_sym
.globl _blocks_def_mem
.globl _block_defs_mem
.globl _insn_defs_mem
.globl _block_get_predecessor_n
.globl _block_get_successor_n
