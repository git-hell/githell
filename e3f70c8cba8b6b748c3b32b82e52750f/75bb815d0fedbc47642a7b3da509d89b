.text
_promote:
L4:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L28:
	movl %esi,%r12d
	movq %rdi,%rbx
	movq 8(%rbx),%rsi
	movq (%rsi),%r13
	andq $131071,%r13
	testq $16384,%r13
	jz L9
L7:
	movq %rbx,%rdi
	call _tree_addrof
	movq %rax,%rbx
L9:
	testq $62,%r13
	jz L12
L10:
	movq %rbx,%rdi
	movl $64,%esi
	call _tree_cast_bits
	movq %rax,%rbx
L12:
	cmpl $1,%r12d
	jnz L15
L16:
	testq $1024,%r13
	jz L15
L13:
	movq %rbx,%rdi
	movl $2048,%esi
	call _tree_cast_bits
	movq %rax,%rbx
L15:
	movq 8(%rbx),%rsi
	movq (%rsi),%rsi
	testq $393216,%rsi
	jz L22
L20:
	leaq 8(%rbx),%rsi
	movq %rbx,%rdi
	call _tree_cast
	movq %rax,%rbx
	movq 8(%rax),%rsi
	andq $-393217,(%rsi)
L22:
	testq $8192,%r13
	jz L25
L23:
	movq %rbx,%rdi
	call _tree_addrof
	movq %rax,%rbx
	leaq 8(%rax),%rdi
	call _type_fix_array
L25:
	movq %rbx,%rax
L6:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L30:
_usuals:
L32:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L61:
	movq %rdi,%rbx
	movq 24(%rbx),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rsi
	testq $8190,%rsi
	jz L34
L38:
	movq 32(%rbx),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rsi
	testq $8190,%rsi
	jz L34
L35:
	movl $4096,%r12d
L43:
	movq 24(%rbx),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rsi
	andq $131071,%rsi
	testq %r12,%rsi
	jnz L45
L49:
	movq 32(%rbx),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rsi
	andq $131071,%rsi
	testq %r12,%rsi
	jnz L45
L44:
	sarq $1,%r12
	cmpq $64,%r12
	jnz L43
L45:
	movq 24(%rbx),%rdi
	movq 8(%rdi),%rsi
	movq (%rsi),%rsi
	andq $131071,%rsi
	testq %r12,%rsi
	jnz L56
L54:
	movq 32(%rbx),%rsi
	addq $8,%rsi
	call _tree_cast
	movq %rax,24(%rbx)
L56:
	movq 32(%rbx),%rdi
	movq 8(%rdi),%rsi
	movq (%rsi),%rsi
	andq $131071,%rsi
	testq %r12,%rsi
	jnz L34
L57:
	movq 24(%rbx),%rsi
	addq $8,%rsi
	call _tree_cast
	movq %rax,32(%rbx)
L34:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L63:
_lvalue:
L65:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L97:
	movl %edx,%r12d
	movl %esi,%ebx
	movq %rdi,%r13
	movl (%r13),%esi
	cmpl $2147483650,%esi
	jz L70
L71:
	movl (%r13),%esi
	cmpl $1073741829,%esi
	jz L70
L68:
	pushq %rbx
	pushq $L75
	pushq $1
	call _error
	addq $24,%rsp
L70:
	cmpl $2,%r12d
	jnz L78
L83:
	movl (%r13),%esi
	cmpl $2147483650,%esi
	jnz L78
L79:
	movq 32(%r13),%rsi
	movl 12(%rsi),%esi
	testl $128,%esi
	jz L78
L76:
	pushq %rbx
	pushq $L87
	pushq $1
	call _error
	addq $24,%rsp
L78:
	cmpl $1,%r12d
	jnz L67
L91:
	movq 8(%r13),%rsi
	movq (%rsi),%rsi
	testq $131072,%rsi
	jz L67
L88:
	pushq %rbx
	pushq $L95
	pushq $1
	call _error
	addq $24,%rsp
L67:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L99:
_pointer_left:
L101:
	pushq %rbp
	movq %rsp,%rbp
L102:
	movl (%rdi),%esi
	testl $536870912,%esi
	jz L103
L107:
	movq 32(%rdi),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rsi
	testq $32768,%rsi
	jz L103
L104:
	call _tree_commute
L103:
	popq %rbp
	ret
L114:
_null_constant:
L116:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L142:
	movq %rdi,%rbx
	movq 24(%rbx),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rsi
	testq $32768,%rsi
	jz L118
L122:
	movq 32(%rbx),%rdi
	movq 8(%rdi),%rsi
	movq (%rsi),%rsi
	testq $1022,%rsi
	jz L118
L119:
	call _tree_simplify
	movq %rax,32(%rbx)
	movl (%rax),%esi
	cmpl $2147483649,%esi
	jnz L118
L137:
	movq 8(%rax),%rsi
	movq (%rsi),%rsi
	testq $1022,%rsi
	jz L118
L133:
	movq 24(%rax),%rsi
	cmpq $0,%rsi
	jnz L118
L129:
	cmpq $0,32(%rax)
	jnz L118
L126:
	movq 24(%rbx),%rsi
	addq $8,%rsi
	movq %rax,%rdi
	call _tree_cast
	movq %rax,32(%rbx)
L118:
	popq %rbx
	popq %rbp
	ret
L144:
_permute0:
L146:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L147:
	call _tree_cast
	movq %rax,%rbx
	movq 24(%rbx),%rsi
	addq $8,%rsi
	leaq 8(%rbx),%rdi
	call _type_requalify
	movq %rbx,%rax
L148:
	popq %rbx
	popq %rbp
	ret
L153:
_permute_voids:
L155:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
	pushq %r12
L185:
	movq %rdi,%rbx
	movq 24(%rbx),%rdi
	movq 32(%rbx),%rcx
	leaq -16(%rbp),%rdx
	xorps %xmm0,%xmm0
	movups %xmm0,-16(%rbp)
	movq %rdx,-8(%rbp)
	movq 8(%rdi),%rdx
	movq (%rdx),%r8
	testq $32768,%r8
	jz L157
L165:
	movq 8(%rcx),%r8
	movq (%r8),%r8
	testq $32768,%r8
	jz L157
L161:
	cmpq $0,%rax
	jz L169
L173:
	movq 16(%rdx),%rax
	movq (%rax),%rax
	testq $1,%rax
	jnz L158
L169:
	movq 8(%rcx),%rax
	movq (%rax),%rcx
	testq $32768,%rcx
	jz L157
L177:
	movq 16(%rax),%rax
	movq (%rax),%rax
	testq $1,%rax
	jz L157
L158:
	cmpl $0,%esi
	jnz L182
L181:
	leaq 8(%rdi),%rsi
	movq 32(%rbx),%rdi
	call _permute0
	movq %rax,32(%rbx)
	jmp L157
L182:
	leaq -16(%rbp),%r12
	movq %r12,%rdi
	movl $32768,%esi
	call _type_append_bits
	movq %r12,%rdi
	movl $1,%esi
	call _type_append_bits
	movq 24(%rbx),%rdi
	movq %r12,%rsi
	call _permute0
	movq %rax,24(%rbx)
	movq 32(%rbx),%rdi
	movq %r12,%rsi
	call _permute0
	movq %rax,32(%rbx)
	movq %r12,%rdi
	call _type_clear
L157:
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L187:
_scale:
L189:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L204:
	movq %rdi,%r12
	movq 24(%r12),%rbx
	movq 32(%r12),%r14
	movq %r12,%rdi
	call _pointer_left
	movl (%r12),%esi
	testl $268435456,%esi
	jz L193
L195:
	movq 8(%rbx),%rsi
	movq (%rsi),%rsi
	testq $32768,%rsi
	jz L193
L192:
	addq $8,%rbx
	movq %rbx,%rdi
	movl $1,%esi
	call _type_sizeof
	movq _target(%rip),%rsi
	movq 8(%rsi),%rdi
	movq %rax,%rsi
	call _tree_i
	movq %rax,%r13
	movq 8(%r14),%rsi
	movq (%rsi),%rsi
	testq $32768,%rsi
	jnz L200
L199:
	movq _target(%rip),%rsi
	movq 8(%rsi),%rsi
	movq %r14,%rdi
	call _tree_cast_bits
	movl $549454359,%edi
	movq %rax,%rsi
	movq %r13,%rdx
	call _tree_binary
	movq %rax,%r13
	movq _target(%rip),%rsi
	movq 8(%rsi),%rsi
	leaq 8(%r13),%rdi
	call _type_append_bits
	movq %r13,32(%r12)
	leaq 8(%r12),%rdi
	movq %rbx,%rsi
	call _type_copy
	jmp L194
L200:
	movq _target(%rip),%rsi
	movq 8(%rsi),%rsi
	leaq 8(%r12),%rdi
	call _type_append_bits
	movl $278,%edi
	movq %r12,%rsi
	movq %r13,%rdx
	call _tree_binary
	movq %rax,%r12
	movq _target(%rip),%rsi
	movq 8(%rsi),%rsi
	leaq 8(%rax),%rdi
	call _type_append_bits
	jmp L194
L193:
	leaq 8(%rbx),%rsi
	leaq 8(%r12),%rdi
	call _type_copy
L194:
	movq %r12,%rax
L191:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L206:
_fix_struct_assign:
L208:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L216:
	movq %rdi,%rbx
	movq 8(%rbx),%rsi
	movq (%rsi),%rsi
	testq $65536,%rsi
	jz L213
L211:
	movq 24(%rbx),%rdi
	call _tree_addrof
	movq %rax,24(%rbx)
	movq 32(%rbx),%rdi
	call _tree_addrof
	movq %rax,32(%rbx)
	movl $43,(%rbx)
	leaq 8(%rbx),%r12
	movq %r12,%rdi
	call _type_clear
	movq 24(%rbx),%rsi
	addq $8,%rsi
	movq %r12,%rdi
	call _type_copy
	movq %rbx,%rdi
	xorl %esi,%esi
	call _tree_fetch
	movq %rax,%rdi
	call _tree_rvalue
	movq %rax,%rbx
L213:
	movq %rbx,%rax
L210:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L218:
.data
.align 4
_map:
	.int 201326602
	.int 2
	.int 3
	.int 134218251
	.int 2
	.int 1
	.int 134217996
	.int 2
	.int 1
	.int 134220045
	.int 0
	.int 1
	.int 402653966
	.int 1
	.int 2
	.int 402654223
	.int 1
	.int 2
	.int 134219280
	.int 0
	.int 1
	.int 134219025
	.int 0
	.int 1
	.int 134219538
	.int 0
	.int 1
	.int 134219795
	.int 0
	.int 1
	.int 134217748
	.int 0
	.int 1
	.int 549453845
	.int 0
	.int 1
	.int 278
	.int 2
	.int 1
	.int 549454359
	.int 2
	.int 1
	.int 817890072
	.int 1
	.int 2
	.int 276825113
	.int 1
	.int 3
	.int 33554458
	.int 2
	.int 2
	.int 134219035
	.int 0
	.int 1
	.int 33554460
	.int 2
	.int 2
	.int 33554461
	.int 2
	.int 2
	.int 134219294
	.int 0
	.int 1
	.int 33554463
	.int 2
	.int 2
	.int 549455648
	.int 0
	.int 1
	.int 184549409
	.int 6
	.int 1
	.int 637534242
	.int 2
	.int 2
	.int 637534243
	.int 2
	.int 2
	.int 549455908
	.int 0
	.int 1
	.int 184549413
	.int 6
	.int 1
	.int 2342
	.int 0
	.int 1
	.int 67108904
	.int 2
	.int 4
.align 8
_operands:
	.quad 1022
	.quad 1022
	.quad 32768
	.quad 1022
	.quad 8190
	.quad 8190
	.quad 32768
	.quad 32768
	.quad 65536
	.quad 65536
	.quad 1
	.quad 1
	.quad 40958
	.quad 40958
.text
_check_operands:
L222:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L282:
	movl %edx,%ebx
	movl %ebx,%eax
	andl $520093696,%eax
	sarl $24,%eax
	movslq %eax,%rax
	imulq $12,%rax
	movl _map+4(%rax),%ecx
	movl _map+8(%rax),%r13d
	xorl %eax,%eax
	cmpl $11534393,%ebx
	jz L231
L277:
	cmpl $486539286,%ebx
	jz L229
L226:
	xorl %edx,%edx
	jmp L227
L229:
	movl $8,%edx
	jmp L227
L231:
	movl $20,%edx
L227:
	xorl %r8d,%r8d
L233:
	cmpl %r13d,%r8d
	jge L236
L234:
	movq 8(%rdi),%r9
	movq (%r9),%r9
	andq $131071,%r9
	leal (%rcx,%r8),%r12d
	movslq %r12d,%r12
	shlq $4,%r12
	movq _operands(%r12),%r14
	testq %r14,%r9
	jz L235
L240:
	movq 8(%rsi),%r9
	movq (%r9),%r9
	andq $131071,%r9
	movq _operands+8(%r12),%r12
	testq %r12,%r9
	jnz L236
L235:
	addl $1,%r8d
	jmp L233
L236:
	cmpl %r13d,%r8d
	jnz L246
L245:
	movl $1,%eax
	jmp L279
L246:
	movq 8(%rdi),%rcx
	movq (%rcx),%rcx
	testq $32768,%rcx
	jz L251
L255:
	movq 8(%rsi),%rcx
	movq (%rcx),%rcx
	testq $32768,%rcx
	jnz L248
L251:
	movq 8(%rdi),%rcx
	movq (%rcx),%rcx
	testq $65536,%rcx
	jz L279
L259:
	movq 8(%rsi),%rcx
	movq (%rcx),%rcx
	testq $65536,%rcx
	jz L279
L248:
	addq $8,%rsi
	addq $8,%rdi
	call _type_compat
L279:
	cmpl $1,%eax
	jz L267
L280:
	cmpl $2,%eax
	jz L273
	jnz L224
L267:
	cmpl $11534393,%ebx
	jnz L269
L268:
	pushq $L271
	pushq $1
	call _error
	addq $16,%rsp
	jmp L273
L269:
	pushq %rbx
	pushq $L272
	pushq $1
	call _error
	addq $24,%rsp
L273:
	pushq $L274
	pushq $1
	call _error
	addq $16,%rsp
L224:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L284:
_map_binary:
L286:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L312:
	movl %edx,%r13d
	movq %rdi,%rbx
	movq %rsi,%rdi
	movl %r13d,%esi
	andl $520093696,%esi
	sarl $24,%esi
	movslq %esi,%rsi
	imulq $12,%rsi
	movl _map(%rsi),%r14d
	xorl %esi,%esi
	call _promote
	movl %r14d,%edi
	movq %rbx,%rsi
	movq %rax,%rdx
	call _tree_binary
	movq %rax,%rbx
	movq %rbx,%r12
	movq %rbx,%rdi
	call _pointer_left
	testl $67108864,%r14d
	jz L291
L289:
	movq %rbx,%rdi
	call _null_constant
	cmpl $486539286,%r13d
	jnz L293
L292:
	movl $1,%esi
	jmp L294
L293:
	xorl %esi,%esi
L294:
	movq %rbx,%rdi
	call _permute_voids
L291:
	movq 32(%rbx),%rsi
	movq 24(%rbx),%rdi
	movl %r13d,%edx
	call _check_operands
	testl $16777216,%r14d
	jz L297
L295:
	movq 24(%rbx),%rdi
	movl $424673333,%esi
	call _test_expression
	movq %rax,24(%rbx)
	movq 32(%rbx),%rdi
	movl $424673333,%esi
	call _test_expression
	movq %rax,32(%rbx)
L297:
	andl $15728640,%r13d
	cmpl $11534336,%r13d
	jnz L300
L298:
	movq 24(%rbx),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rsi
	testq $8190,%rsi
	jz L300
L301:
	andq $131071,%rsi
	movq 32(%rbx),%rdi
	call _tree_cast_bits
	movq %rax,32(%rbx)
L300:
	testl $134217728,%r14d
	jnz L306
L304:
	movq %rbx,%rdi
	call _usuals
L306:
	testl $33554432,%r14d
	jz L308
L307:
	leaq 8(%rbx),%rdi
	movl $64,%esi
	call _type_append_bits
	jmp L309
L308:
	movq %rbx,%rdi
	call _scale
	movq %rax,%r12
L309:
	movq %r12,%rax
L288:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L314:
_crement:
L316:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L334:
	movl %esi,%r12d
	movl %edx,%r14d
	movq %rdi,%r13
	cmpl $27,%r14d
	jnz L320
L319:
	movl $1,%ebx
	jmp L321
L320:
	movl $-1,%ebx
L321:
	movq %r13,%rdi
	movl %r14d,%esi
	movl $1,%edx
	call _lvalue
	movq 8(%r13),%rsi
	movq (%rsi),%rsi
	testq $40958,%rsi
	jnz L324
L322:
	pushq %r14
	pushq $L325
	pushq $1
	call _error
	addq $24,%rsp
L324:
	movq 8(%r13),%rsi
	movq (%rsi),%rdi
	testq $32768,%rdi
	jz L327
L326:
	movslq %ebx,%rsi
	movq _target(%rip),%rdi
	movq 8(%rdi),%rdi
	call _tree_i
	jmp L328
L327:
	testq $1022,%rdi
	jz L330
L329:
	movslq %ebx,%rsi
	andq $131071,%rdi
	call _tree_i
	jmp L328
L330:
	cvtsi2sdl %ebx,%xmm0
	andq $131071,%rdi
	call _tree_f
L328:
	movl %r12d,%edi
	movq %r13,%rsi
	movq %rax,%rdx
	call _tree_binary
	movq %rax,%rdi
	call _scale
L318:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L336:
_primary:
L338:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L339:
	movl _token(%rip),%esi
	cmpl $2,%esi
	jz L359
L384:
	cmpl $3,%esi
	jz L345
L385:
	cmpl $4,%esi
	jz L347
L386:
	cmpl $5,%esi
	jz L349
L387:
	cmpl $6,%esi
	jz L351
L388:
	cmpl $7,%esi
	jz L353
L389:
	cmpl $8,%esi
	jz L355
L390:
	cmpl $9,%esi
	jz L357
L391:
	cmpl $262145,%esi
	jz L363
L392:
	cmpl $262156,%esi
	jz L361
L342:
	pushq %rsi
	pushq $L380
	pushq $1
	call _error
	addq $24,%rsp
	jmp L343
L361:
	call _lex
	call _comma
	movq %rax,%rbx
	movl $13,%edi
	call _lex_match
	jmp L343
L363:
	movq _token+8(%rip),%r12
	call _lex
	movq _token+8(%rip),%rdx
	movl _current_scope(%rip),%edi
	movl $2,%esi
	movl $268436472,%ecx
	call _symbol_lookup
	movq %rax,%rbx
	cmpq $0,%rax
	jz L365
L364:
	movl 12(%rax),%esi
	testl $512,%esi
	jz L366
L367:
	movl 64(%rax),%esi
	movslq %esi,%rsi
	movl $64,%edi
	call _tree_i
	movq %rax,%rbx
	jmp L343
L365:
	movl _token(%rip),%esi
	cmpl $262156,%esi
	jz L373
L371:
	pushq %r12
	pushq $L374
	pushq $1
	call _error
	addq $24,%rsp
L373:
	movq %r12,%rdi
	call _symbol_implicit
	movq %rax,%rbx
L366:
	movl 12(%rbx),%esi
	testl $8,%esi
	jz L377
L375:
	pushq %r12
	pushq $L378
	pushq $1
	call _error
	addq $24,%rsp
L377:
	orl $1073741824,12(%rbx)
	movq %rbx,%rdi
	call _tree_sym
	movq %rax,%rbx
	jmp L343
L357:
	movsd _token+8(%rip),%xmm0
	movl $4096,%edi
	call _tree_f
	movq %rax,%rbx
	call _lex
	jmp L343
L355:
	movsd _token+8(%rip),%xmm0
	movl $2048,%edi
	call _tree_f
	movq %rax,%rbx
	call _lex
	jmp L343
L353:
	movsd _token+8(%rip),%xmm0
	movl $1024,%edi
	call _tree_f
	movq %rax,%rbx
	call _lex
	jmp L343
L351:
	movq _token+8(%rip),%rsi
	movl $512,%edi
	call _tree_i
	movq %rax,%rbx
	call _lex
	jmp L343
L349:
	movq _token+8(%rip),%rsi
	movl $256,%edi
	call _tree_i
	movq %rax,%rbx
	call _lex
	jmp L343
L347:
	movq _token+8(%rip),%rsi
	movl $128,%edi
	call _tree_i
	movq %rax,%rbx
	call _lex
	jmp L343
L345:
	movq _token+8(%rip),%rsi
	movl $64,%edi
	call _tree_i
	movq %rax,%rbx
	call _lex
	jmp L343
L359:
	movq _token+8(%rip),%rdi
	call _string_symbol
	movq %rax,%rdi
	call _tree_sym
	movq %rax,%rbx
	call _lex
L343:
	movq %rbx,%rax
L340:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L396:
_access:
L398:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L399:
	leaq -16(%rbp),%rsi
	xorps %xmm0,%xmm0
	movups %xmm0,-16(%rbp)
	movq %rsi,-8(%rbp)
	xorl %esi,%esi
	call _promote
	movq %rax,%r13
	movl _token(%rip),%esi
	cmpl $18,%esi
	jnz L402
L401:
	movl (%rax),%esi
	cmpl $2147483650,%esi
	jz L405
L404:
	movl (%rax),%esi
	cmpl $1073741829,%esi
	jnz L406
L405:
	movl $1,%r14d
	jmp L407
L406:
	xorl %r14d,%r14d
L407:
	movq %rax,%rdi
	call _tree_addrof
	movq %rax,%r13
	jmp L403
L402:
	movl $1,%r14d
L403:
	movq 8(%r13),%rsi
	movq (%rsi),%rdi
	testq $32768,%rdi
	jz L409
L411:
	movq 16(%rsi),%rsi
	movq (%rsi),%rdi
	testq $65536,%rdi
	jz L409
L408:
	movq 8(%rsi),%r12
	jmp L410
L409:
	movl _token(%rip),%esi
	pushq %rsi
	pushq $L415
	pushq $1
	call _error
	addq $24,%rsp
L410:
	movl 12(%r12),%esi
	testl $2147483648,%esi
	jnz L418
L416:
	pushq %r12
	pushq $L419
	pushq $1
	call _error
	addq $24,%rsp
L418:
	call _lex
	movl $262145,%edi
	call _lex_expect
	movq _token+8(%rip),%rsi
	movq %r12,%rdi
	call _member_lookup
	movq %rax,%rbx
	cmpq $0,%rbx
	jnz L422
L420:
	movq _token+8(%rip),%rsi
	pushq %r12
	pushq %rsi
	pushq $L423
	pushq $1
	call _error
	addq $32,%rsp
L422:
	leaq 32(%rbx),%rsi
	leaq -16(%rbp),%r12
	movq %r12,%rdi
	call _type_copy
	movq 8(%r13),%rsi
	movq 16(%rsi),%rsi
	movq (%rsi),%rsi
	andq $393216,%rsi
	movq %r12,%rdi
	call _type_qualify
	movl 64(%rbx),%esi
	movslq %esi,%rsi
	movq _target(%rip),%rdi
	movq 8(%rdi),%rdi
	call _tree_i
	movl $817890072,%edi
	movq %r13,%rsi
	movq %rax,%rdx
	call _tree_binary
	movq %rax,%rbx
	leaq 8(%rbx),%rdi
	movq %r12,%rsi
	call _type_ref
	movq %r12,%rdi
	call _type_clear
	call _lex
	movq %rbx,%rdi
	xorl %esi,%esi
	call _tree_fetch
	movq %rax,%rsi
	cmpl $0,%r14d
	jnz L426
L424:
	movq %rax,%rdi
	call _tree_rvalue
	movq %rax,%rsi
L426:
	movq %rsi,%rax
L400:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L431:
_array:
L433:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L434:
	xorl %esi,%esi
	call _promote
	movq %rax,%rbx
	movl $14,%edi
	call _lex_match
	call _comma
	movl $817890072,%edi
	movq %rbx,%rsi
	movq %rax,%rdx
	call _tree_binary
	movq %rax,%rbx
	movl $15,%edi
	call _lex_match
	movq %rbx,%rdi
	call _pointer_left
	movq 24(%rbx),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rsi
	testq $32768,%rsi
	jz L436
L439:
	movq 32(%rbx),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rsi
	testq $1022,%rsi
	jnz L438
L436:
	pushq $L443
	pushq $1
	call _error
	addq $16,%rsp
L438:
	movq %rbx,%rdi
	call _scale
	movq %rax,%rdi
	xorl %esi,%esi
	call _tree_fetch
L435:
	popq %rbx
	popq %rbp
	ret
L448:
_call:
L450:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
L451:
	leaq -16(%rbp),%rsi
	xorps %xmm0,%xmm0
	movups %xmm0,-16(%rbp)
	movq %rsi,-8(%rbp)
	xorl %esi,%esi
	call _promote
	movq %rax,%rbx
	movq 8(%rbx),%rsi
	movq (%rsi),%rdi
	testq $32768,%rdi
	jz L453
L456:
	movq 16(%rsi),%rsi
	movq (%rsi),%rsi
	testq $16384,%rsi
	jnz L455
L453:
	pushq $L460
	pushq $1
	call _error
	addq $16,%rsp
L455:
	leaq 8(%rbx),%rsi
	leaq -16(%rbp),%r12
	movq %r12,%rdi
	call _type_deref
	movl $1073741827,%edi
	movq %rbx,%rsi
	call _tree_unary
	movq %rax,%r13
	leaq 8(%rax),%rdi
	movq %r12,%rsi
	call _type_deref
	call _lex
	movq -16(%rbp),%rsi
	movq 8(%rsi),%r12
	movl _token(%rip),%esi
	cmpl $13,%esi
	jz L463
L465:
	call _assignment
	movq %rax,%rbx
	cmpq $0,%r12
	jnz L469
L468:
	movq -16(%rbp),%rsi
	movq (%rsi),%rdi
	movq $4611686018427387904,%rcx
	testq %rcx,%rdi
	jnz L471
L474:
	movq (%rsi),%rsi
	movq $-9223372036854775808,%rdi
	testq %rdi,%rsi
	jz L472
L471:
	movq %rax,%rdi
	movl $1,%esi
	call _promote
	movq %rax,%rbx
	jmp L479
L472:
	pushq $L478
	pushq $1
	call _error
	addq $16,%rsp
	jmp L479
L469:
	leaq 16(%r12),%rdi
	movq %rax,%rsi
	call _fake_assign
	movq %rax,%rbx
	movq 32(%r12),%r12
L479:
	leaq 48(%rbx),%rsi
	movq $0,48(%rbx)
	movq 40(%r13),%rdi
	movq %rdi,56(%rbx)
	movq 40(%r13),%rdi
	movq %rbx,(%rdi)
	movq %rsi,40(%r13)
	movl _token(%rip),%esi
	cmpl $13,%esi
	jz L463
L483:
	movl $524309,%edi
	call _lex_match
	jmp L465
L463:
	movl $13,%edi
	call _lex_match
	cmpq $0,%r12
	jz L488
L486:
	pushq $L489
	pushq $1
	call _error
	addq $16,%rsp
L488:
	leaq -16(%rbp),%rdi
	call _type_clear
	movq %r13,%rax
L452:
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L494:
_postfix:
L496:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L497:
	call _primary
	movq %rax,%rbx
L500:
	movl _token(%rip),%edx
	cmpl $14,%edx
	jz L513
L520:
	cmpl $18,%edx
	jz L511
L521:
	cmpl $26,%edx
	jz L511
L522:
	cmpl $27,%edx
	jb L523
L524:
	cmpl $28,%edx
	ja L523
L508:
	movq %rbx,%rdi
	movl $268436265,%esi
	call _crement
	movq %rax,%rbx
	call _lex
	jmp L500
L523:
	cmpl $262156,%edx
	jz L515
L504:
	movq %rbx,%rax
L498:
	popq %rbx
	popq %rbp
	ret
L515:
	movq %rbx,%rdi
	call _call
	movq %rax,%rbx
	jmp L500
L511:
	movq %rbx,%rdi
	call _access
	movq %rax,%rbx
	jmp L500
L513:
	movq %rbx,%rdi
	call _array
	movq %rax,%rbx
	jmp L500
L528:
_unary:
L531:
	pushq %rbp
	movq %rsp,%rbp
	subq $32,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L532:
	leaq -16(%rbp),%r13
	xorps %xmm0,%xmm0
	movups %xmm0,-16(%rbp)
	leaq -8(%rbp),%rbx
	movq %r13,-8(%rbp)
	movl _token(%rip),%r14d
	cmpl $25,%r14d
	jz L542
L602:
	cmpl $27,%r14d
	jb L603
L604:
	cmpl $28,%r14d
	ja L603
L575:
	call _lex
	call _unary
	movq %rax,%rsi
	movq %rsi,%rdi
	movl $402653966,%esi
	movl %r14d,%edx
	call _crement
	jmp L533
L603:
	cmpl $29,%r14d
	jz L558
L605:
	cmpl $81,%r14d
	jz L577
L606:
	cmpl $219414559,%r14d
	jz L544
L607:
	cmpl $236978208,%r14d
	jz L540
L608:
	cmpl $253755425,%r14d
	jz L538
L609:
	cmpl $375390250,%r14d
	jz L560
L535:
	call _postfix
	jmp L533
L560:
	call _lex
	call _cast
	movq %rax,%rbx
	movq %rbx,%rdi
	movl $375390250,%esi
	movl $2,%edx
	call _lvalue
	movl (%rbx),%esi
	cmpl $1073741829,%esi
	jnz L563
L564:
	movq 24(%rbx),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rdi
	testq $32768,%rdi
	jz L563
L568:
	movq 16(%rsi),%rsi
	movq (%rsi),%rsi
	movl $2147483648,%edi
	testq %rdi,%rsi
	jz L563
L561:
	pushq $L572
	pushq $1
	call _error
	addq $16,%rsp
L563:
	movq %rbx,%rdi
	call _tree_addrof
	jmp L533
L538:
	movl $1082130439,%ebx
	movl $8190,%r13d
	jmp L536
L540:
	movl $1073741832,%ebx
	movl $8190,%r13d
	jmp L536
L544:
	call _lex
	call _cast
	movq %rax,%rsi
	movq %rsi,%rdi
	xorl %esi,%esi
	call _promote
	movq %rax,%rbx
	movq 8(%rbx),%rsi
	movq (%rsi),%rdi
	testq $32768,%rdi
	jz L545
L548:
	cmpq $0,%r12
	jz L547
L552:
	movq 16(%rsi),%rsi
	movq (%rsi),%rsi
	testq $1,%rsi
	jz L547
L545:
	pushq $L556
	pushq $1
	call _error
	addq $16,%rsp
L547:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _tree_fetch
	jmp L533
L577:
	call _lex
	movl _token(%rip),%esi
	cmpl $262156,%esi
	jnz L580
L578:
	leaq -32(%rbp),%rsi
	movq %rsi,%rdi
	call _lex_peek
	subq $16,%rsp
	movups -32(%rbp),%xmm0
	movups %xmm0,(%rsp)
	call _k_decl
	addq $16,%rsp
	movl %eax,%esi
	cmpl $0,%esi
	jnz L582
L581:
	call _unary
	movq %rax,%rsi
	leaq 8(%rsi),%rdi
	movq 8(%rsi),%rax
	cmpq $0,%rax
	jz L585
L587:
	movq (%rbx),%rcx
	movq %rax,(%rcx)
	movq 16(%rsi),%rax
	movq %rax,(%rbx)
	movq $0,8(%rsi)
	movq %rdi,16(%rsi)
L585:
	movq %rsi,%rdi
	call _tree_free
	jmp L580
L582:
	call _lex
	movq %r13,%rdi
	call _abstract_declarator
	movl $13,%edi
	call _lex_match
L580:
	leaq -16(%rbp),%rbx
	movq %rbx,%rdi
	xorl %esi,%esi
	call _type_sizeof
	movq %rax,%rsi
	movq _target(%rip),%rdi
	movq 16(%rdi),%rdi
	call _tree_i
	movq %rax,%r12
	movq %rbx,%rdi
	call _type_clear
	movq %r12,%rax
	jmp L533
L558:
	call _lex
	call _cast
	movq %rax,%rsi
	movq %rsi,%rdi
	movl $407896116,%esi
	call _test_expression
	jmp L533
L542:
	movl $1082130441,%ebx
	movl $1022,%r13d
L536:
	call _lex
	call _cast
	movq %rax,%rsi
	movq %rsi,%rdi
	xorl %esi,%esi
	call _promote
	movq %rax,%r12
	movq 8(%r12),%rsi
	movq (%rsi),%rsi
	testq %r13,%rsi
	jnz L597
L595:
	pushq %r14
	pushq $L598
	pushq $1
	call _error
	addq $24,%rsp
L597:
	movl %ebx,%edi
	movq %r12,%rsi
	call _tree_unary
	movq %rax,%rbx
	movq 24(%rbx),%rsi
	addq $8,%rsi
	leaq 8(%rbx),%rdi
	call _type_copy
	movq %rbx,%rax
L533:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L613:
_cast:
L614:
	pushq %rbp
	movq %rsp,%rbp
	subq $32,%rsp
	pushq %rbx
	pushq %r12
L615:
	leaq -16(%rbp),%rbx
	xorps %xmm0,%xmm0
	movups %xmm0,-16(%rbp)
	movq %rbx,-8(%rbp)
	movl _token(%rip),%esi
	cmpl $262156,%esi
	jnz L618
L620:
	leaq -32(%rbp),%rdi
	call _lex_peek
	subq $16,%rsp
	movups -32(%rbp),%xmm0
	movups %xmm0,(%rsp)
	call _k_decl
	addq $16,%rsp
	cmpl $0,%eax
	jz L618
L617:
	movl $262156,%edi
	call _lex_match
	movq %rbx,%rdi
	call _abstract_declarator
	movl $13,%edi
	call _lex_match
	call _cast
	movq %rax,%rdi
	xorl %esi,%esi
	call _promote
	movq %rax,%rbx
	movq 8(%rbx),%rsi
	movq (%rsi),%rsi
	testq $40958,%rsi
	jz L627
L639:
	movq -16(%rbp),%rdi
	movq (%rdi),%rdi
	testq $40958,%rdi
	jz L627
L635:
	testq $7168,%rsi
	jz L631
L643:
	testq $32768,%rdi
	jnz L627
L631:
	movq 8(%rbx),%rsi
	movq (%rsi),%rsi
	testq $32768,%rsi
	jz L626
L647:
	movq -16(%rbp),%rsi
	movq (%rsi),%rsi
	testq $7168,%rsi
	jz L626
L627:
	movq -16(%rbp),%rsi
	movq (%rsi),%rsi
	testq $1,%rsi
	jnz L626
L624:
	pushq $L651
	pushq $1
	call _error
	addq $16,%rsp
L626:
	leaq -16(%rbp),%r12
	movq %rbx,%rdi
	movq %r12,%rsi
	call _tree_cast
	movq %rax,%rbx
	movq %r12,%rdi
	call _type_clear
	movq %rbx,%rax
	jmp L616
L618:
	call _unary
L616:
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L657:
_binary:
L658:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L670:
	movl %edi,%r14d
	cmpl $0,%r14d
	jnz L663
L661:
	call _cast
	jmp L660
L663:
	leal -1048576(%r14),%edi
	call _binary
	movq %rax,%rbx
L665:
	movl _token(%rip),%r12d
	movl %r12d,%esi
	andl $15728640,%esi
	cmpl %r14d,%esi
	jnz L667
L666:
	call _lex
	leal -1048576(%r14),%edi
	call _binary
	movq %rax,%r13
	movq %rbx,%rdi
	xorl %esi,%esi
	call _promote
	movq %rax,%rdi
	movq %r13,%rsi
	movl %r12d,%edx
	call _map_binary
	movq %rax,%rbx
	jmp L665
L667:
	movq %rbx,%rax
L660:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L672:
_ternary:
L674:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L675:
	xorl %r14d,%r14d
	movl $10485760,%edi
	call _binary
	movq %rax,%rbx
	movl _token(%rip),%esi
	cmpl $24,%esi
	jnz L679
L677:
	movq %rax,%rdi
	movl $424673333,%esi
	call _test_expression
	movq %rax,%rbx
	call _lex
	call _comma
	movq %rax,%rdi
	xorl %esi,%esi
	call _promote
	movq %rax,%r12
	movl $486539286,%edi
	call _lex_match
	call _ternary
	movq %r12,%rdi
	movq %rax,%rsi
	movl $486539286,%edx
	call _map_binary
	movq %rax,%r12
	movq 24(%r12),%rdi
	movq 8(%rdi),%rsi
	movq (%rsi),%rsi
	testq $65536,%rsi
	jz L682
L680:
	call _tree_addrof
	movq %rax,24(%r12)
	movq 32(%r12),%rdi
	call _tree_addrof
	movq %rax,32(%r12)
	leaq 8(%r12),%r13
	movq %r13,%rdi
	call _type_clear
	movq 24(%r12),%rsi
	addq $8,%rsi
	movq %r13,%rdi
	call _type_copy
	movl $1,%r14d
L682:
	movl $39,%edi
	movq %rbx,%rsi
	movq %r12,%rdx
	call _tree_binary
	movq %rax,%r13
	movq %r13,%rbx
	leaq 8(%r12),%rsi
	leaq 8(%r13),%rdi
	call _type_copy
	cmpl $0,%r14d
	jz L679
L683:
	movq %r13,%rdi
	xorl %esi,%esi
	call _tree_fetch
	movq %rax,%rdi
	call _tree_rvalue
	movq %rax,%rbx
L679:
	movq %rbx,%rax
L676:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L690:
_assignment:
L691:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L692:
	call _ternary
	movq %rax,%rbx
	movl _token(%rip),%r12d
	movl %r12d,%esi
	andl $15728640,%esi
	cmpl $11534336,%esi
	jnz L693
L694:
	call _lex
	call _assignment
	movq %rax,%r13
	movq %rbx,%rdi
	movl %r12d,%esi
	movl $1,%edx
	call _lvalue
	movq %rbx,%rdi
	movq %r13,%rsi
	movl %r12d,%edx
	call _map_binary
	movq %rax,%rdi
	call _fix_struct_assign
L693:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L701:
_comma:
L702:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L703:
	call _assignment
	movq %rax,%r13
	movq %r13,%rbx
	movl _token(%rip),%esi
	cmpl $524309,%esi
	jnz L707
L705:
	call _lex
	call _comma
	movq %rax,%r14
	movq 8(%rax),%rsi
	movq (%rsi),%rsi
	testq $65536,%rsi
	jz L709
L708:
	movq %rax,%rdi
	call _tree_addrof
	movq %rax,%r14
	movl $1,%r12d
	jmp L710
L709:
	xorl %r12d,%r12d
L710:
	movl $42,%edi
	movq %r13,%rsi
	movq %r14,%rdx
	call _tree_binary
	movq %rax,%r13
	movq %r13,%rbx
	leaq 8(%r14),%rsi
	leaq 8(%r13),%rdi
	call _type_copy
	cmpl $0,%r12d
	jz L707
L711:
	movq %r13,%rdi
	xorl %esi,%esi
	call _tree_fetch
	movq %rax,%rbx
L707:
	movq %rbx,%rax
L704:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L718:
_expression:
L719:
	pushq %rbp
	movq %rsp,%rbp
L720:
	call _comma
	movq %rax,%rdi
	call _tree_simplify
L721:
	popq %rbp
	ret
L726:
_init_expression:
L727:
	pushq %rbp
	movq %rsp,%rbp
L728:
	call _assignment
L729:
	popq %rbp
	ret
L734:
_static_expression:
L735:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L736:
	call _ternary
	movq %rax,%rdi
	xorl %esi,%esi
	call _promote
	movq %rax,%rdi
	call _tree_simplify
	movq %rax,%rbx
	movl (%rbx),%esi
	cmpl $2147483649,%esi
	jz L740
L738:
	pushq $L741
	pushq $1
	call _error
	addq $16,%rsp
L740:
	movq %rbx,%rax
L737:
	popq %rbx
	popq %rbp
	ret
L746:
_int_expression:
L747:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L797:
	movq %rdi,%r12
	testl $1,%esi
	jz L751
L750:
	xorl %ebx,%ebx
	jmp L752
L751:
	movl $1,%ebx
L752:
	call _static_expression
	movq %rax,%r13
	movl (%r13),%esi
	cmpl $2147483649,%esi
	jnz L753
L760:
	cmpq $0,32(%r13)
	jnz L753
L756:
	movq 8(%r13),%rsi
	movq (%rsi),%rsi
	testq $1022,%rsi
	jnz L755
L753:
	pushq $L764
	pushq $1
	call _error
	addq $16,%rsp
L755:
	movq 24(%r13),%r14
	testq $64,%r12
	jz L767
L772:
	cmpq $-2147483648,%r14
	jl L767
L768:
	cmpq $2147483647,%r14
	jle L776
L767:
	testq $192,%r12
	jz L780
L785:
	cmpq $0,%r14
	jl L780
L781:
	movl $4294967295,%esi
	cmpq %rsi,%r14
	jle L776
L780:
	testq $768,%r12
	jnz L776
L792:
	pushq $L794
	pushq %rbx
	call _error
	addq $16,%rsp
L776:
	movq %r13,%rdi
	call _tree_free
	movq %r14,%rax
L749:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L799:
_size_expression:
L800:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L809:
	movl $128,%edi
	xorl %esi,%esi
	call _int_expression
	movq %rax,%rbx
	cmpl $134217728,%ebx
	jle L805
L803:
	pushq $L806
	pushq $1
	call _error
	addq $16,%rsp
L805:
	movl %ebx,%eax
L802:
	popq %rbx
	popq %rbp
	ret
L811:
_test_expression:
L812:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L821:
	movl %esi,%r12d
	xorl %esi,%esi
	call _promote
	movq %rax,%rbx
	movq 8(%rbx),%rsi
	movq (%rsi),%rsi
	testq $40958,%rsi
	jnz L817
L815:
	pushq $L818
	pushq $1
	call _error
	addq $16,%rsp
L817:
	movl $64,%edi
	xorl %esi,%esi
	call _tree_i
	movq %rbx,%rdi
	movq %rax,%rsi
	movl %r12d,%edx
	call _map_binary
L814:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L823:
_assign:
L824:
	pushq %rbp
	movq %rsp,%rbp
L825:
	movl $11534393,%edx
	call _map_binary
	movq %rax,%rdi
	call _fix_struct_assign
L826:
	popq %rbp
	ret
L831:
_fake_assign:
L832:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L837:
	movq %rsi,%r13
	movq %rdi,%r12
	xorl %edi,%edi
	movl $128,%esi
	call _symbol_new
	movq %rax,%rbx
	leaq 32(%rbx),%rdi
	movq %r12,%rsi
	call _type_copy
	movq %rbx,%rdi
	call _tree_sym
	movq %rax,%rdi
	movq %r13,%rsi
	movl $11534393,%edx
	call _map_binary
	movq %rax,%r12
	movq %r12,%rdi
	call _tree_commute
	movq %r12,%rdi
	call _tree_chop_binary
	movq %rax,%r12
	movq %rbx,%rdi
	call _symbol_free
	movq %r12,%rdi
	call _tree_simplify
L834:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L839:
L415:
	.byte 105,108,108,101,103,97,108,32
	.byte 108,101,102,116,32,115,105,100
	.byte 101,32,111,102,32,37,107,32
	.byte 111,112,101,114,97,116,111,114
	.byte 0
L651:
	.byte 105,110,118,97,108,105,100,32
	.byte 99,97,115,116,0
L572:
	.byte 99,97,110,39,116,32,116,97
	.byte 107,101,32,97,100,100,114,101
	.byte 115,115,32,111,102,32,98,105
	.byte 116,32,102,105,101,108,100,0
L556:
	.byte 99,97,110,39,116,32,100,101
	.byte 114,101,102,101,114,101,110,99
	.byte 101,32,116,104,97,116,0
L95:
	.byte 111,112,101,114,97,116,111,114
	.byte 32,37,107,32,114,101,113,117
	.byte 105,114,101,115,32,110,111,110
	.byte 45,99,111,110,115,116,32,111
	.byte 112,101,114,97,110,100,0
L419:
	.byte 37,65,32,105,115,32,97,110
	.byte 32,105,110,99,111,109,112,108
	.byte 101,116,101,32,116,121,112,101
	.byte 0
L325:
	.byte 111,112,101,114,97,116,111,114
	.byte 32,37,107,32,114,101,113,117
	.byte 105,114,101,115,32,115,99,97
	.byte 108,97,114,32,116,121,112,101
	.byte 0
L271:
	.byte 105,110,99,111,109,112,97,116
	.byte 105,98,108,101,32,116,121,112
	.byte 101,0
L87:
	.byte 99,97,110,39,116,32,97,112
	.byte 112,108,121,32,37,107,32,116
	.byte 111,32,114,101,103,105,115,116
	.byte 101,114,32,118,97,114,105,97
	.byte 98,108,101,0
L443:
	.byte 105,110,118,97,108,105,100,32
	.byte 111,112,101,114,97,110,100,115
	.byte 32,116,111,32,91,93,0
L423:
	.byte 109,101,109,98,101,114,32,39
	.byte 37,83,39,32,110,111,116,32
	.byte 105,110,32,37,65,0
L489:
	.byte 110,111,116,32,101,110,111,117
	.byte 103,104,32,102,117,110,99,116
	.byte 105,111,110,32,97,114,103,117
	.byte 109,101,110,116,115,0
L478:
	.byte 116,111,111,32,109,97,110,121
	.byte 32,102,117,110,99,116,105,111
	.byte 110,32,97,114,103,117,109,101
	.byte 110,116,115,0
L274:
	.byte 100,105,115,99,97,114,100,115
	.byte 32,113,117,97,108,105,102,105
	.byte 101,114,115,0
L818:
	.byte 115,99,97,108,97,114,32,101
	.byte 120,112,114,101,115,115,105,111
	.byte 110,32,114,101,113,117,105,114
	.byte 101,100,0
L764:
	.byte 105,110,116,101,103,101,114,32
	.byte 99,111,110,115,116,97,110,116
	.byte 32,101,120,112,114,101,115,115
	.byte 105,111,110,32,114,101,113,117
	.byte 105,114,101,100,0
L741:
	.byte 99,111,110,115,116,97,110,116
	.byte 32,101,120,112,114,101,115,115
	.byte 105,111,110,32,114,101,113,117
	.byte 105,114,101,100,0
L806:
	.byte 115,105,122,101,32,101,120,112
	.byte 114,101,115,115,105,111,110,32
	.byte 111,117,116,32,111,102,32,114
	.byte 97,110,103,101,0
L794:
	.byte 105,110,116,101,103,101,114,32
	.byte 101,120,112,114,101,115,115,105
	.byte 111,110,32,111,117,116,32,111
	.byte 102,32,114,97,110,103,101,0
L75:
	.byte 111,112,101,114,97,116,111,114
	.byte 32,37,107,32,114,101,113,117
	.byte 105,114,101,115,32,97,110,32
	.byte 108,118,97,108,117,101,0
L378:
	.byte 39,37,83,39,32,105,115,32
	.byte 97,32,116,121,112,101,100,101
	.byte 102,0
L380:
	.byte 101,120,112,114,101,115,115,105
	.byte 111,110,32,109,105,115,115,105
	.byte 110,103,32,40,103,111,116,32
	.byte 37,107,41,0
L598:
	.byte 105,108,108,101,103,97,108,32
	.byte 111,112,101,114,97,110,100,32
	.byte 116,111,32,117,110,97,114,121
	.byte 32,37,107,0
L272:
	.byte 105,110,99,111,109,112,97,116
	.byte 105,98,108,101,32,111,112,101
	.byte 114,97,110,100,115,32,116,111
	.byte 32,98,105,110,97,114,121,32
	.byte 37,107,0
L460:
	.byte 40,41,32,114,101,113,117,105
	.byte 114,101,115,32,102,117,110,99
	.byte 116,105,111,110,32,111,114,32
	.byte 112,111,105,110,116,101,114,45
	.byte 116,111,45,102,117,110,99,116
	.byte 105,111,110,0
L374:
	.byte 39,37,83,39,32,105,115,32
	.byte 117,110,107,110,111,119,110,0
.globl _member_lookup
.globl _symbol_lookup
.globl _type_clear
.globl _abstract_declarator
.globl _error
.globl _target
.globl _symbol_implicit
.globl _tree_cast
.globl _type_compat
.globl _lex_expect
.globl _current_scope
.globl _tree_commute
.globl _symbol_new
.globl _lex
.globl _type_fix_array
.globl _tree_cast_bits
.globl _type_append_bits
.globl _symbol_free
.globl _tree_free
.globl _tree_rvalue
.globl _tree_addrof
.globl _tree_f
.globl _type_sizeof
.globl _type_deref
.globl _type_ref
.globl _tree_fetch
.globl _lex_match
.globl _binary
.globl _tree_simplify
.globl _tree_chop_binary
.globl _tree_binary
.globl _tree_unary
.globl _tree_i
.globl _type_copy
.globl _type_requalify
.globl _type_qualify
.globl _lex_peek
.globl _string_symbol
.globl _k_decl
.globl _tree_sym
.globl _fake_assign
.globl _assign
.globl _test_expression
.globl _size_expression
.globl _int_expression
.globl _static_expression
.globl _init_expression
.globl _expression
.globl _token
