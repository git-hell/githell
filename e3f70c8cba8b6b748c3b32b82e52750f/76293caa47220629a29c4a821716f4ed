.text
_hash:
L3:
	pushq %rbp
	movq %rsp,%rbp
L4:
	xorl %eax,%eax
	movl (%rdi),%esi
	testl $1,%esi
	jz L7
L6:
	leaq 1(%rdi),%rsi
	jmp L9
L7:
	movq 16(%rdi),%rsi
L9:
	movzbl (%rsi),%edi
	cmpl $0,%edi
	jz L5
L10:
	shll $3,%eax
	movzbl (%rsi),%edi
	addq $1,%rsi
	xorl %edi,%eax
	jmp L9
L5:
	popq %rbp
	ret
L16:
_insert:
L18:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L35:
	movq %rdi,%r13
	call _hash
	andl $63,%eax
	movl %eax,%ebx
	movl $80,%edi
	call _safe_malloc
	movq %rax,%r12
	movq %r12,%rdi
	call _vstring_init
	movq %r12,%rdi
	movq %r13,%rsi
	call _vstring_concat
	leaq 32(%r12),%rsi
	movq $0,32(%r12)
	movq %rsi,40(%r12)
	leaq 48(%r12),%rsi
	movq $0,48(%r12)
	movq %rsi,56(%r12)
	movl $0,24(%r12)
	movslq %ebx,%rsi
	shlq $4,%rsi
	movq _buckets(%rsi),%rax
	leaq 64(%r12),%rdi
	movq %rax,64(%r12)
	cmpq $0,%rax
	jz L31
L30:
	movq _buckets(%rsi),%rsi
	movq %rdi,72(%rsi)
	jmp L32
L31:
	movq %rdi,_buckets+8(%rsi)
L32:
	movslq %ebx,%rsi
	shlq $4,%rsi
	leaq _buckets(%rsi),%rdi
	movq %r12,_buckets(%rsi)
	movq %rdi,72(%r12)
	movq %r12,%rax
L20:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L37:
.data
.align 8
_predefs:
	.quad L39
	.int 32
	.space 4, 0
	.quad L40
	.int 16
	.space 4, 0
	.quad L41
	.int 8
	.space 4, 0
	.quad L42
	.int 4
	.space 4, 0
	.quad L43
	.int 2
	.space 4, 0
	.quad L44
	.int 1
	.space 4, 0
.text
_macro_predef:
L45:
	pushq %rbp
	movq %rsp,%rbp
	subq $24,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
L46:
	xorps %xmm0,%xmm0
	movups %xmm0,-24(%rbp)
	movq $0,-8(%rbp)
	orl $1,-24(%rbp)
	xorl %r13d,%r13d
L48:
	movslq %r13d,%rsi
	cmpq $6,%rsi
	jae L51
L49:
	leaq -24(%rbp),%r12
	movq %r12,%rdi
	call _vstring_clear
	movslq %r13d,%rbx
	shlq $4,%rbx
	movq _predefs(%rbx),%rsi
	movq %r12,%rdi
	call _vstring_puts
	movq %r12,%rdi
	call _insert
	movq %rax,%r12
	movl _predefs+8(%rbx),%esi
	movl %esi,24(%r12)
	xorl %eax,%eax
	cmpl $32,%esi
	jnz L54
L56:
	movl $1,%edi
	call _token_number
L54:
	cmpq $0,%rax
	jz L50
L61:
	leaq 32(%rax),%rsi
	movq $0,32(%rax)
	movq 56(%r12),%rdi
	movq %rdi,40(%rax)
	movq 56(%r12),%rdi
	movq %rax,(%rdi)
	movq %rsi,56(%r12)
L50:
	addl $1,%r13d
	jmp L48
L51:
	leaq -24(%rbp),%rdi
	call _vstring_free
L47:
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L69:
_tokens:
L71:
	pushq %rbp
	movq %rsp,%rbp
	subq $72,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
L111:
	movq %rsi,%rbx
	movq %rdi,%r13
	movl 24(%r13),%esi
	andl $63,%esi
	cmpl $2,%esi
	jz L93
L107:
	cmpl $4,%esi
	jz L85
L108:
	cmpl $8,%esi
	jz L80
L109:
	cmpl $16,%esi
	jnz L75
L78:
	movq _input_stack(%rip),%rsi
	movl 32(%rsi),%edi
	call _token_number
	jmp L102
L80:
	movq _input_stack(%rip),%rsi
	movl 8(%rsi),%edi
	testl $1,%edi
	jz L82
L81:
	leaq 9(%rsi),%rdi
	jmp L83
L82:
	movq 24(%rsi),%rdi
L83:
	call _token_string
L102:
	leaq 32(%rax),%rsi
	movq $0,32(%rax)
	movq 8(%rbx),%rdi
	movq %rdi,40(%rax)
	movq 8(%rbx),%rdi
	movq %rax,(%rdi)
	movq %rsi,8(%rbx)
	jmp L73
L85:
	cmpq $0,48(%r13)
	jnz L93
L86:
	leaq -8(%rbp),%r12
	movq %r12,%rdi
	call _time
	movq %r12,%rdi
	call _localtime
	leaq -72(%rbp),%r12
	movq %r12,%rdi
	movl $64,%esi
	movq $L89,%rdx
	movq %rax,%rcx
	call _strftime
	movq %r12,%rdi
	call _token_string
	leaq 32(%rax),%rsi
	movq $0,32(%rax)
	movq 56(%r13),%rdi
	movq %rdi,40(%rax)
	movq 56(%r13),%rdi
	movq %rax,(%rdi)
	movq %rsi,56(%r13)
L93:
	cmpq $0,48(%r13)
	jnz L75
L94:
	leaq -8(%rbp),%r12
	movq %r12,%rdi
	call _time
	movq %r12,%rdi
	call _localtime
	leaq -72(%rbp),%r12
	movq %r12,%rdi
	movl $64,%esi
	movq $L97,%rdx
	movq %rax,%rcx
	call _strftime
	movq %r12,%rdi
	call _token_string
	leaq 32(%rax),%rsi
	movq $0,32(%rax)
	movq 56(%r13),%rdi
	movq %rdi,40(%rax)
	movq 56(%r13),%rdi
	movq %rax,(%rdi)
	movq %rsi,56(%r13)
L75:
	leaq 48(%r13),%rsi
	movq %rbx,%rdi
	call _list_copy
L73:
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L113:
_macro_lookup:
L114:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L139:
	movq %rdi,%r13
	call _hash
	andl $63,%eax
	movl %eax,%ebx
	movslq %eax,%rsi
	shlq $4,%rsi
	movq _buckets(%rsi),%r12
L117:
	cmpq $0,%r12
	jz L120
L118:
	movq %r12,%rdi
	movq %r13,%rsi
	call _vstring_same
	cmpl $0,%eax
	jz L119
L124:
	movq 64(%r12),%rsi
	cmpq $0,%rsi
	jz L128
L127:
	movq 72(%r12),%rdi
	movq %rdi,72(%rsi)
	jmp L129
L128:
	movq 72(%r12),%rsi
	movslq %ebx,%rdi
	shlq $4,%rdi
	movq %rsi,_buckets+8(%rdi)
L129:
	leaq 64(%r12),%rsi
	movq 64(%r12),%rdi
	movq 72(%r12),%rax
	movq %rdi,(%rax)
	movslq %ebx,%rdi
	shlq $4,%rdi
	movq _buckets(%rdi),%rax
	movq %rax,64(%r12)
	cmpq $0,%rax
	jz L134
L133:
	movq _buckets(%rdi),%rdi
	movq %rsi,72(%rdi)
	jmp L135
L134:
	movq %rsi,_buckets+8(%rdi)
L135:
	movslq %ebx,%rsi
	shlq $4,%rsi
	leaq _buckets(%rsi),%rdi
	movq %r12,_buckets(%rsi)
	movq %rdi,72(%r12)
	movq %r12,%rax
	jmp L116
L119:
	movq 64(%r12),%r12
	jmp L117
L120:
	xorl %eax,%eax
L116:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L141:
_macro_define:
L142:
	pushq %rbp
	movq %rsp,%rbp
	subq $32,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L218:
	movq %rdi,%rbx
	leaq -16(%rbp),%rsi
	xorps %xmm0,%xmm0
	movups %xmm0,-16(%rbp)
	movq %rsi,-8(%rbp)
	xorl %r13d,%r13d
	leaq -24(%rbp),%rdx
	movq %rbx,%rdi
	movl $49,%esi
	call _list_match
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L147
L148:
	movl (%rsi),%esi
	cmpl $536870927,%esi
	jnz L147
L145:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
	movl $-2147483648,%r13d
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L160
L155:
	movl (%rsi),%esi
	cmpl $536870928,%esi
	jz L154
L160:
	movq %rbx,%rdi
	call _list_strip_ends
	leaq -32(%rbp),%rdx
	movq %rbx,%rdi
	movl $49,%esi
	call _list_match
	movq -32(%rbp),%rsi
	movq $0,32(%rsi)
	movq -8(%rbp),%rsi
	movq -32(%rbp),%rdi
	movq %rsi,40(%rdi)
	movq -8(%rbp),%rsi
	movq -32(%rbp),%rdi
	movq %rdi,(%rsi)
	movq -32(%rbp),%rsi
	addq $32,%rsi
	movq %rsi,-8(%rbp)
	movq %rbx,%rdi
	call _list_strip_ends
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L154
L169:
	movl (%rsi),%esi
	cmpl $536870959,%esi
	jnz L154
L166:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
	jmp L160
L154:
	movq %rbx,%rdi
	movl $536870928,%esi
	xorl %edx,%edx
	call _list_match
L147:
	leaq -16(%rbp),%r14
	movq %rbx,%rdi
	movq %r14,%rsi
	call _list_normalize
	movq -24(%rbp),%rsi
	leaq 8(%rsi),%rdi
	call _macro_lookup
	movq %rax,%r12
	cmpq $0,%r12
	jz L175
L174:
	movl 24(%r12),%esi
	testl $63,%esi
	jz L179
L177:
	movl (%r12),%esi
	testl $1,%esi
	jz L182
L181:
	leaq 1(%r12),%rsi
	jmp L183
L182:
	movq 16(%r12),%rsi
L183:
	pushq %rsi
	pushq $L180
	call _error
	addq $16,%rsp
L179:
	leaq -16(%rbp),%rsi
	leaq 32(%r12),%rdi
	call _list_same
	cmpl $0,%eax
	jz L184
L191:
	leaq 48(%r12),%rdi
	movq %rbx,%rsi
	call _list_same
	cmpl $0,%eax
	jz L184
L187:
	movl 24(%r12),%esi
	cmpl %r13d,%esi
	jz L186
L184:
	movl (%r12),%esi
	testl $1,%esi
	jz L197
L196:
	leaq 1(%r12),%rsi
	jmp L198
L197:
	movq 16(%r12),%rsi
L198:
	pushq %rsi
	pushq $L195
	call _error
	addq $16,%rsp
L186:
	leaq -16(%rbp),%rdi
	xorl %esi,%esi
	call _list_cut
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_cut
	jmp L176
L175:
	movq -24(%rbp),%rsi
	leaq 8(%rsi),%rdi
	call _insert
	movl %r13d,24(%rax)
	movq -16(%rbp),%rsi
	cmpq $0,%rsi
	jz L208
L202:
	movq 40(%rax),%rdi
	movq %rsi,(%rdi)
	movq 40(%rax),%rsi
	movq -16(%rbp),%rdi
	movq %rsi,40(%rdi)
	movq -8(%rbp),%rsi
	movq %rsi,40(%rax)
	movq $0,-16(%rbp)
	movq %r14,-8(%rbp)
L208:
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L176
L211:
	movq 56(%rax),%rdi
	movq %rsi,(%rdi)
	movq 56(%rax),%rsi
	movq (%rbx),%rdi
	movq %rsi,40(%rdi)
	movq 8(%rbx),%rsi
	movq %rsi,56(%rax)
	movq $0,(%rbx)
	movq %rbx,8(%rbx)
L176:
	movq -24(%rbp),%rdi
	call _token_free
L144:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L220:
_macro_cmdline:
L221:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
L244:
	movq %rdi,%rsi
	leaq -16(%rbp),%rbx
	xorps %xmm0,%xmm0
	movups %xmm0,-16(%rbp)
	movq %rbx,-8(%rbp)
	movq %rbx,%rdi
	call _input_tokenize
	movq %rbx,%rdi
	call _list_strip_ends
	movq -16(%rbp),%rsi
	cmpq $0,%rsi
	jz L224
L227:
	movl (%rsi),%edi
	cmpl $49,%edi
	jz L226
L224:
	xorl %eax,%eax
	jmp L223
L226:
	movq 32(%rsi),%rsi
	cmpq $0,%rsi
	jz L233
L232:
	movl (%rsi),%edi
	cmpl $536870925,%edi
	jz L237
L235:
	xorl %eax,%eax
	jmp L223
L237:
	movq %rbx,%rdi
	call _list_drop
	jmp L234
L233:
	movl $1,%edi
	call _token_number
	leaq 32(%rax),%rsi
	movq $0,32(%rax)
	movq -8(%rbp),%rdi
	movq %rdi,40(%rax)
	movq -8(%rbp),%rdi
	movq %rax,(%rdi)
	movq %rsi,-8(%rbp)
L234:
	leaq -16(%rbp),%rdi
	call _macro_define
	movl $1,%eax
L223:
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L246:
_macro_undef:
L247:
	pushq %rbp
	movq %rsp,%rbp
	subq $8,%rsp
	pushq %rbx
L248:
	leaq -8(%rbp),%rdx
	movl $49,%esi
	call _list_match
	movq -8(%rbp),%rsi
	leaq 8(%rsi),%rdi
	call _macro_lookup
	movq %rax,%rbx
	cmpq $0,%rbx
	jz L252
L250:
	movl 24(%rbx),%esi
	testl $63,%esi
	jz L255
L253:
	movl (%rbx),%esi
	testl $1,%esi
	jz L258
L257:
	leaq 1(%rbx),%rsi
	jmp L259
L258:
	movq 16(%rbx),%rsi
L259:
	pushq %rsi
	pushq $L256
	call _error
	addq $16,%rsp
L255:
	leaq 32(%rbx),%rdi
	xorl %esi,%esi
	call _list_cut
	leaq 48(%rbx),%rdi
	xorl %esi,%esi
	call _list_cut
	movq %rbx,%rdi
	call _vstring_free
	movq -8(%rbp),%rsi
	leaq 8(%rsi),%rdi
	call _hash
	andl $63,%eax
	movq 64(%rbx),%rsi
	cmpq $0,%rsi
	jz L264
L263:
	movq 72(%rbx),%rdi
	movq %rdi,72(%rsi)
	jmp L265
L264:
	movq 72(%rbx),%rsi
	movslq %eax,%rdi
	shlq $4,%rdi
	movq %rsi,_buckets+8(%rdi)
L265:
	movq 64(%rbx),%rsi
	movq 72(%rbx),%rdi
	movq %rsi,(%rdi)
	movq %rbx,%rdi
	call _free
L252:
	movq -8(%rbp),%rdi
	call _token_free
L249:
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L269:
_arg_new:
L271:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L282:
	movq %rdi,%rbx
	movl $32,%edi
	call _safe_malloc
	movq $0,(%rax)
	movq %rax,8(%rax)
	leaq 16(%rax),%rsi
	movq $0,16(%rax)
	movq 8(%rbx),%rdi
	movq %rdi,24(%rax)
	movq 8(%rbx),%rdi
	movq %rax,(%rdi)
	movq %rsi,8(%rbx)
L273:
	popq %rbx
	popq %rbp
	ret
L284:
_args_clear:
L286:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L299:
	movq %rdi,%r12
L289:
	movq (%r12),%rbx
	cmpq $0,%rbx
	jz L288
L290:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_cut
	movq %rbx,%rdi
	call _free
	movq 16(%rbx),%rsi
	cmpq $0,%rsi
	jz L296
L295:
	movq 24(%rbx),%rdi
	movq %rdi,24(%rsi)
	jmp L297
L296:
	movq 24(%rbx),%rsi
	movq %rsi,8(%r12)
L297:
	movq 16(%rbx),%rsi
	movq 24(%rbx),%rdi
	movq %rsi,(%rdi)
	jmp L289
L288:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L301:
_arg_i:
L303:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L304:
	movq (%rdi),%rbx
L306:
	cmpl $0,%esi
	jle L309
L307:
	movq 16(%rbx),%rbx
	addl $-1,%esi
	jmp L306
L309:
	cmpq $0,%rbx
	jnz L312
L310:
	pushq $L313
	call _error
	addq $8,%rsp
L312:
	movq %rbx,%rax
L305:
	popq %rbx
	popq %rbp
	ret
L318:
_gather:
L320:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L386:
	movq %rdx,%r14
	movq %rdi,%rbx
	movq %rsi,%r15
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
	movq 32(%r15),%r13
L323:
	cmpq $0,%r13
	jz L325
L324:
	movq %r14,%rdi
	call _arg_new
	movq %rax,%r12
	xorl %ecx,%ecx
L326:
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L328
L327:
	cmpl $0,%ecx
	jnz L331
L332:
	movl (%rsi),%edi
	cmpl $536870959,%edi
	jz L328
L336:
	cmpl $536870928,%edi
	jz L328
L331:
	movl (%rsi),%edi
	cmpl $536870927,%edi
	jnz L343
L341:
	addl $1,%ecx
L343:
	movl (%rsi),%edi
	cmpl $536870928,%edi
	jnz L347
L344:
	addl $-1,%ecx
L347:
	movq 32(%rsi),%rdi
	cmpq $0,%rdi
	jz L351
L350:
	movq 40(%rsi),%rax
	movq %rax,40(%rdi)
	jmp L352
L351:
	movq 40(%rsi),%rdi
	movq %rdi,8(%rbx)
L352:
	leaq 32(%rsi),%rdi
	movq 32(%rsi),%rax
	movq 40(%rsi),%rdx
	movq %rax,(%rdx)
	movq $0,32(%rsi)
	movq 8(%r12),%rax
	movq %rax,40(%rsi)
	movq 8(%r12),%rax
	movq %rsi,(%rax)
	movq %rdi,8(%r12)
	jmp L326
L328:
	movq %r12,%rdi
	call _list_strip_ends
	movq %r12,%rdi
	call _list_fold_spaces
	movq %r12,%rdi
	call _list_placeholder
	movq 32(%r13),%r13
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L325
L359:
	movl (%rsi),%esi
	cmpl $536870959,%esi
	jnz L325
L356:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
	jmp L323
L325:
	cmpq $0,%r13
	jz L366
L364:
	movl (%r15),%esi
	testl $1,%esi
	jz L369
L368:
	leaq 1(%r15),%rsi
	jmp L370
L369:
	movq 16(%r15),%rsi
L370:
	pushq %rsi
	pushq $L367
	call _error
	addq $16,%rsp
L366:
	movq (%rbx),%rdi
	call _list_skip_spaces
	movq %rax,%r12
	cmpq $0,%r12
	jnz L373
L371:
	movl (%r15),%esi
	testl $1,%esi
	jz L376
L375:
	leaq 1(%r15),%rsi
	jmp L377
L376:
	movq 16(%r15),%rsi
L377:
	pushq %rsi
	pushq $L374
	call _error
	addq $16,%rsp
L373:
	movl (%r12),%esi
	cmpl $536870928,%esi
	jz L380
L378:
	movl (%r15),%esi
	testl $1,%esi
	jz L383
L382:
	leaq 1(%r15),%rsi
	jmp L384
L383:
	movq 16(%r15),%rsi
L384:
	pushq %rsi
	pushq $L381
	call _error
	addq $16,%rsp
L380:
	movq 32(%r12),%rsi
	movq %rbx,%rdi
	call _list_cut
L322:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L388:
_stringize:
L390:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L415:
	movq %rsi,%r15
	movq %rdi,%rbx
	movq (%rbx),%r12
L393:
	cmpq $0,%r12
	jz L392
L394:
	movl (%r12),%esi
	cmpl $1610612790,%esi
	jnz L397
L396:
	movq %rbx,%rdi
	movq %r12,%rsi
	call _list_drop
	movq %rax,%r12
L399:
	cmpq $0,%r12
	jz L401
L402:
	movl (%r12),%esi
	cmpl $48,%esi
	jnz L401
L400:
	movq %rbx,%rdi
	movq %r12,%rsi
	call _list_drop
	movq %rax,%r12
	jmp L399
L401:
	cmpq $0,%r12
	jz L406
L409:
	movl (%r12),%esi
	cmpl $2147483706,%esi
	jz L408
L406:
	pushq $L413
	call _error
	addq $8,%rsp
L408:
	movq 8(%r12),%rsi
	movq %r15,%rdi
	call _arg_i
	movq %rax,%r14
	movq %rbx,%rdi
	movq %r12,%rsi
	call _list_drop
	movq %rax,%r13
	movq %r13,%r12
	movq %r14,%rdi
	call _list_stringize
	movq %rbx,%rdi
	movq %r13,%rsi
	movq %rax,%rdx
	call _list_insert
	jmp L393
L397:
	movq 32(%r12),%r12
	jmp L393
L392:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L417:
_expand:
L419:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L436:
	movq %rsi,%r14
	movq %rdi,%rbx
	leaq -16(%rbp),%rsi
	xorps %xmm0,%xmm0
	movups %xmm0,-16(%rbp)
	movq %rsi,-8(%rbp)
	movq (%rbx),%r12
L422:
	cmpq $0,%r12
	jz L421
L423:
	movl (%r12),%esi
	cmpl $2147483706,%esi
	jnz L426
L425:
	movq 8(%r12),%rsi
	movq %r14,%rdi
	call _arg_i
	leaq -16(%rbp),%r13
	movq %r13,%rdi
	movq %rax,%rsi
	call _list_copy
	movq %rbx,%rdi
	movq %r12,%rsi
	movl $1610612791,%edx
	call _list_next_is
	cmpl $0,%eax
	jnz L430
L431:
	movq %rbx,%rdi
	movq %r12,%rsi
	movl $1610612791,%edx
	call _list_prev_is
	cmpl $0,%eax
	jnz L430
L428:
	movq %r13,%rdi
	call _macro_replace_all
L430:
	movq %rbx,%rdi
	movq %r12,%rsi
	call _list_drop
	movq %rax,%r12
	leaq -16(%rbp),%rdx
	movq %rbx,%rdi
	movq %rax,%rsi
	call _list_insert_list
	jmp L422
L426:
	movq 32(%r12),%r12
	jmp L422
L421:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L438:
_paste:
L440:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L458:
	movq %rdi,%rbx
	movq (%rbx),%r12
L443:
	cmpq $0,%r12
	jz L442
L444:
	movl (%r12),%esi
	cmpl $1610612791,%esi
	jnz L447
L446:
	movq %rbx,%rdi
	movq %r12,%rsi
	call _list_strip_around
	movq 40(%r12),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%r14
	movq 32(%r12),%r13
	cmpq $0,%r14
	jz L449
L452:
	cmpq $0,%r13
	jnz L451
L449:
	pushq $L456
	call _error
	addq $8,%rsp
L451:
	movq %r14,%rdi
	movq %r13,%rsi
	call _token_paste
	movq %rax,%r15
	movq %rbx,%rdi
	movq %r14,%rsi
	call _list_drop
	movq %rbx,%rdi
	movq %r13,%rsi
	call _list_drop
	movq %rbx,%rdi
	movq %r12,%rsi
	call _list_drop
	movq %rax,%r12
	movq %rbx,%rdi
	movq %rax,%rsi
	movq %r15,%rdx
	call _list_insert
	jmp L443
L447:
	movq 32(%r12),%r12
	jmp L443
L442:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L460:
_macro_replace:
L461:
	pushq %rbp
	movq %rsp,%rbp
	subq $32,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L507:
	movq %rdi,%r13
	leaq -16(%rbp),%rbx
	xorps %xmm0,%xmm0
	movups %xmm0,-16(%rbp)
	movq %rbx,-8(%rbp)
	leaq -32(%rbp),%rsi
	xorps %xmm0,%xmm0
	movups %xmm0,-32(%rbp)
	movq %rsi,-24(%rbp)
	movq (%r13),%r12
	movl (%r12),%esi
	cmpl $49,%esi
	jz L466
L464:
	xorl %eax,%eax
	jmp L463
L466:
	leaq 8(%r12),%rdi
	call _macro_lookup
	movq %rax,%r14
	cmpq $0,%r14
	jz L468
L471:
	movl 24(%r14),%esi
	testl $1,%esi
	jz L470
L468:
	xorl %eax,%eax
	jmp L463
L470:
	movl 24(%r14),%esi
	testl $2147483648,%esi
	jz L477
L476:
	movq 32(%r12),%rdi
	call _list_skip_spaces
	cmpq $0,%rax
	jz L479
L482:
	movl (%rax),%esi
	cmpl $536870927,%esi
	jz L481
L479:
	xorl %eax,%eax
	jmp L463
L481:
	movq %r13,%rdi
	movq %rax,%rsi
	call _list_cut
	movq %r13,%rdi
	movq %r14,%rsi
	movq %rbx,%rdx
	call _gather
	jmp L478
L477:
	movq %r13,%rdi
	xorl %esi,%esi
	call _list_pop
L478:
	leaq -32(%rbp),%rbx
	movq %r14,%rdi
	movq %rbx,%rsi
	call _tokens
	leaq -16(%rbp),%r12
	movq %rbx,%rdi
	movq %r12,%rsi
	call _stringize
	movq %rbx,%rdi
	movq %r12,%rsi
	call _expand
	movq %rbx,%rdi
	call _paste
	movq %rbx,%rdi
	movq %r14,%rsi
	call _list_ennervate
	movq (%r13),%rsi
	cmpq $0,%rsi
	jz L496
L490:
	movq -24(%rbp),%rdi
	movq %rsi,(%rdi)
	movq -24(%rbp),%rsi
	movq (%r13),%rdi
	movq %rsi,40(%rdi)
	movq 8(%r13),%rsi
	movq %rsi,-24(%rbp)
	movq $0,(%r13)
	movq %r13,8(%r13)
L496:
	leaq -32(%rbp),%rsi
	movq -32(%rbp),%rdi
	cmpq $0,%rdi
	jz L497
L499:
	movq 8(%r13),%rax
	movq %rdi,(%rax)
	movq 8(%r13),%rdi
	movq -32(%rbp),%rax
	movq %rdi,40(%rax)
	movq -24(%rbp),%rdi
	movq %rdi,8(%r13)
	movq $0,-32(%rbp)
	movq %rsi,-24(%rbp)
L497:
	leaq -16(%rbp),%rdi
	call _args_clear
	movl $1,%eax
L463:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L509:
_macro_replace_all:
L510:
	pushq %rbp
	movq %rsp,%rbp
	subq $24,%rsp
	pushq %rbx
	pushq %r12
L532:
	movq %rdi,%rbx
	leaq -16(%rbp),%rsi
	xorps %xmm0,%xmm0
	movups %xmm0,-16(%rbp)
	movq %rsi,-8(%rbp)
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L522
L516:
	movq %rsi,-16(%rbp)
	movq -8(%rbp),%rsi
	movq (%rbx),%rdi
	movq %rsi,40(%rdi)
	movq 8(%rbx),%rsi
	movq %rsi,-8(%rbp)
	movq $0,(%rbx)
	movq %rbx,8(%rbx)
L522:
	leaq -16(%rbp),%r12
	cmpq $0,-16(%rbp)
	jz L512
L523:
	movq %r12,%rdi
	call _macro_replace
	cmpl $0,%eax
	jnz L522
L525:
	leaq -24(%rbp),%rsi
	movq %r12,%rdi
	call _list_pop
	movq -24(%rbp),%rsi
	movq $0,32(%rsi)
	movq 8(%rbx),%rsi
	movq -24(%rbp),%rdi
	movq %rsi,40(%rdi)
	movq 8(%rbx),%rsi
	movq -24(%rbp),%rdi
	movq %rdi,(%rsi)
	movq -24(%rbp),%rsi
	addq $32,%rsi
	movq %rsi,8(%rbx)
	jmp L522
L512:
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L534:
L456:
	.byte 109,105,115,115,105,110,103,32
	.byte 111,112,101,114,97,110,100,115
	.byte 32,116,111,32,112,97,115,116
	.byte 101,32,40,35,35,41,32,111
	.byte 112,101,114,97,116,111,114,0
L97:
	.byte 37,72,58,37,77,58,37,83
	.byte 0
L89:
	.byte 37,98,32,37,100,32,37,89
	.byte 0
L43:
	.byte 95,95,84,73,77,69,95,95
	.byte 0
L42:
	.byte 95,95,68,65,84,69,95,95
	.byte 0
L41:
	.byte 95,95,70,73,76,69,95,95
	.byte 0
L40:
	.byte 95,95,76,73,78,69,95,95
	.byte 0
L39:
	.byte 95,95,83,84,68,67,95,95
	.byte 0
L313:
	.byte 67,80,80,32,73,78,84,69
	.byte 82,78,65,76,58,32,97,114
	.byte 103,117,109,101,110,116,32,111
	.byte 117,116,32,111,102,32,98,111
	.byte 117,110,100,115,0
L44:
	.byte 100,101,102,105,110,101,100,0
L381:
	.byte 116,111,111,32,109,97,110,121
	.byte 32,97,114,103,117,109,101,110
	.byte 116,115,32,116,111,32,109,97
	.byte 99,114,111,32,39,37,115,39
	.byte 0
L374:
	.byte 100,101,102,111,114,109,101,100
	.byte 32,97,114,103,117,109,101,110
	.byte 116,115,32,116,111,32,109,97
	.byte 99,114,111,32,39,37,115,39
	.byte 0
L367:
	.byte 116,111,111,32,102,101,119,32
	.byte 97,114,103,117,109,101,110,116
	.byte 115,32,116,111,32,109,97,99
	.byte 114,111,32,39,37,115,39,0
L256:
	.byte 99,97,110,39,116,32,35,117
	.byte 110,100,101,102,32,114,101,115
	.byte 101,114,118,101,100,32,105,100
	.byte 101,110,116,105,102,105,101,114
	.byte 32,39,37,115,39,0
L195:
	.byte 105,110,99,111,109,112,97,116
	.byte 105,98,108,101,32,114,101,45
	.byte 35,100,101,102,105,110,105,116
	.byte 105,111,110,32,111,102,32,39
	.byte 37,115,39,0
L180:
	.byte 99,97,110,39,116,32,35,100
	.byte 101,102,105,110,101,32,114,101
	.byte 115,101,114,118,101,100,32,105
	.byte 100,101,110,116,105,102,105,101
	.byte 114,32,39,37,115,39,0
L413:
	.byte 105,108,108,101,103,97,108,32
	.byte 111,112,101,114,97,110,100,32
	.byte 116,111,32,115,116,114,105,110
	.byte 103,105,122,101,32,40,35,41
	.byte 0
.globl _macro_lookup
.globl _list_drop
.globl _list_pop
.globl _list_placeholder
.globl _token_number
.globl _vstring_clear
.globl _error
.globl _list_fold_spaces
.globl _list_skip_spaces
.globl _list_prev_is
.globl _list_next_is
.globl _list_insert_list
.globl _list_strip_around
.globl _list_cut
.globl _vstring_concat
.globl _vstring_init
.globl _input_tokenize
.globl _macro_cmdline
.globl _macro_define
.globl _list_ennervate
.globl _list_stringize
.globl _list_normalize
.globl _token_paste
.globl _token_string
.globl _macro_replace_all
.local _buckets
.comm _buckets, 1024, 8
.globl _list_strip_ends
.globl _vstring_puts
.globl _safe_malloc
.globl _list_insert
.globl _macro_replace
.globl _list_same
.globl _token_free
.globl _vstring_same
.globl _vstring_free
.globl _free
.globl _time
.globl _strftime
.globl _localtime
.globl _macro_undef
.globl _macro_predef
.globl _list_match
.globl _list_copy
.globl _input_stack
