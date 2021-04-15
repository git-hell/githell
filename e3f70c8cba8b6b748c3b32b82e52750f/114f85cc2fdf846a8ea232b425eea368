.data
.align 8
_begin:
	.quad _buf+4096
.align 8
_pos:
	.quad _buf+4096
.align 8
_invalid:
	.quad _buf+4096
.text
_refill:
L7:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L8:
	movq _pos(%rip),%rdi
	movq _invalid(%rip),%rdx
	movq %rdx,%rsi
	subq %rdi,%rsi
	cmpq $3,%rsi
	jge L9
L13:
	movl _fd(%rip),%esi
	cmpl $-1,%esi
	jz L9
L10:
	movq _begin(%rip),%rsi
	movq %rsi,%rbx
	subq $_buf,%rbx
	subq %rsi,%rdx
	movq $_buf,%rdi
	call _memmove
	subq %rbx,_begin(%rip)
	subq %rbx,_pos(%rip)
	movq _invalid(%rip),%rsi
	subq %rbx,%rsi
	movq %rsi,_invalid(%rip)
	movq $_buf+4096,%rbx
	subq %rsi,%rbx
	cmpq $0,%rbx
	jnz L19
L17:
	pushq $L20
	pushq $1
	call _error
	addq $16,%rsp
L19:
	movl _fd(%rip),%edi
	movq _invalid(%rip),%rsi
	movq %rbx,%rdx
	call _read
	movq %rax,%r12
	cmpq $-1,%r12
	jnz L23
L21:
	pushq $L24
	pushq $1
	call _error
	addq $16,%rsp
L23:
	movq _invalid(%rip),%rsi
	addq %r12,%rsi
	movq %rsi,_invalid(%rip)
	cmpq %rbx,%r12
	jae L9
L25:
	movb $0,(%rsi)
	movl _fd(%rip),%edi
	call _close
	movl $-1,_fd(%rip)
L9:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L31:
_ident:
L33:
	pushq %rbp
	movq %rsp,%rbp
L36:
	movq _pos(%rip),%rsi
	movzbq (%rsi),%rdi
	movl %edi,%eax
	movzbl ___ctype+1(%rax),%eax
	testl $7,%eax
	jnz L43
L39:
	movzbl %dil,%edi
	cmpl $95,%edi
	jnz L38
L43:
	movq _pos(%rip),%rdi
	addq $1,%rdi
	movq %rdi,_pos(%rip)
	movq _invalid(%rip),%rsi
	subq %rdi,%rsi
	cmpq $3,%rsi
	jge L36
L46:
	call _refill
	jmp L36
L38:
	movq _begin(%rip),%rdi
	subq %rdi,%rsi
	call _string_new
	movq %rax,_token+8(%rip)
	movl 16(%rax),%eax
L35:
	popq %rbp
	ret
L53:
_delimit:
L55:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L56:
	movq _pos(%rip),%rsi
	movzbl (%rsi),%r13d
	movl $1,%ebx
L58:
	movq _pos(%rip),%rsi
	movzbl (%rsi),%r12d
	cmpl $0,%r12d
	jz L60
L61:
	leaq 1(%rsi),%rdi
	movq %rdi,_pos(%rip)
	movq _invalid(%rip),%rsi
	subq %rdi,%rsi
	cmpq $3,%rsi
	jge L62
L64:
	call _refill
L62:
	cmpl %r13d,%r12d
	jnz L69
L70:
	cmpl $0,%ebx
	jz L57
L69:
	cmpl $0,%ebx
	jz L76
L75:
	xorl %ebx,%ebx
	jmp L58
L76:
	cmpl $92,%r12d
	setz %sil
	movzbl %sil,%ebx
	jmp L58
L60:
	cmpl $39,%r13d
	jnz L79
L78:
	pushq $L81
	pushq $1
	call _error
	addq $16,%rsp
	jmp L57
L79:
	pushq $L82
	pushq $1
	call _error
	addq $16,%rsp
L57:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L86:
_strlit:
L88:
	pushq %rbp
	movq %rsp,%rbp
	subq $8,%rsp
	pushq %rbx
	pushq %r12
L91:
	movq _pos(%rip),%rsi
	movzbl (%rsi),%esi
	cmpl $34,%esi
	jnz L93
L92:
	call _delimit
	movq _pos(%rip),%rbx
	subq _begin(%rip),%rbx
L94:
	movq _pos(%rip),%rsi
	movzbq (%rsi),%rdi
	movzbl ___ctype+1(%rdi),%edi
	testl $8,%edi
	jz L91
L97:
	leaq 1(%rsi),%rdi
	movq %rdi,_pos(%rip)
	movq _invalid(%rip),%rsi
	subq %rdi,%rsi
	cmpq $3,%rsi
	jge L94
L100:
	call _refill
	jmp L94
L93:
	movq _begin(%rip),%rsi
	addq %rbx,%rsi
	movq %rsi,_pos(%rip)
	movq _begin(%rip),%rsi
	movq %rsi,%r12
	addq $1,%rsi
	movq %rsi,-8(%rbp)
L103:
	movq _pos(%rip),%rsi
	cmpq -8(%rbp),%rsi
	jz L105
L106:
	leaq -8(%rbp),%rdi
	movq -8(%rbp),%rsi
	movzbl (%rsi),%esi
	cmpl $34,%esi
	jz L113
L107:
	call _escape
	movl %eax,%ebx
	cmpl $-1,%ebx
	jnz L111
L109:
	pushq $L112
	pushq $1
	call _error
	addq $16,%rsp
L111:
	movb %bl,(%r12)
	addq $1,%r12
	jmp L106
L113:
	movq -8(%rbp),%rsi
	addq $1,%rsi
	movq %rsi,-8(%rbp)
	cmpq %rsi,_pos(%rip)
	jz L103
L116:
	movzbq (%rsi),%rsi
	movl %esi,%edi
	movzbl ___ctype+1(%rdi),%edi
	testl $8,%edi
	jz L103
L114:
	movzbl %sil,%esi
	cmpl $10,%esi
	jnz L113
L120:
	addl $1,_error_line_no(%rip)
	jmp L113
L105:
	movq _begin(%rip),%rdi
	subq %rdi,%r12
	movq %r12,%rsi
	call _string_new
	movq %rax,_token+8(%rip)
	movl $2,%eax
L90:
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L127:
_ccon:
L129:
	pushq %rbp
	movq %rsp,%rbp
L130:
	call _delimit
	addq $1,_begin(%rip)
	movq $_begin,%rdi
	call _escape
	movslq %eax,%rsi
	movq %rsi,_token+8(%rip)
	cmpq $-1,%rsi
	jnz L134
L132:
	pushq $L135
	pushq $1
	call _error
	addq $16,%rsp
L134:
	movq _begin(%rip),%rsi
	movzbl (%rsi),%esi
	cmpl $39,%esi
	jz L138
L136:
	pushq $L139
	pushq $1
	call _error
	addq $16,%rsp
L138:
	movl $3,%eax
L131:
	popq %rbp
	ret
L144:
_number:
L146:
	pushq %rbp
	movq %rsp,%rbp
	subq $8,%rsp
	pushq %rbx
	pushq %r12
L147:
	xorl %r12d,%r12d
	xorl %ebx,%ebx
L149:
	movq _pos(%rip),%rsi
	movzbq (%rsi),%rsi
	movl %esi,%edi
	movzbl ___ctype+1(%rdi),%edi
	testl $7,%edi
	jnz L150
L156:
	movzbl %sil,%esi
	cmpl $46,%esi
	jz L150
L152:
	cmpl $95,%esi
	jnz L151
L150:
	movq _pos(%rip),%rsi
	movzbl (%rsi),%edi
	call _toupper
	cmpl $69,%eax
	jnz L171
L163:
	movq _pos(%rip),%rsi
	movzbl 1(%rsi),%esi
	cmpl $43,%esi
	jz L160
L167:
	cmpl $45,%esi
	jnz L171
L160:
	addq $1,_pos(%rip)
L171:
	movq _pos(%rip),%rdi
	addq $1,%rdi
	movq %rdi,_pos(%rip)
	movq _invalid(%rip),%rsi
	subq %rdi,%rsi
	cmpq $3,%rsi
	jge L149
L174:
	call _refill
	jmp L149
L151:
	movl $0,_errno(%rip)
	leaq -8(%rbp),%rsi
	movq _begin(%rip),%rdi
	xorl %edx,%edx
	call _strtoul
	movq %rax,_token+8(%rip)
	movq -8(%rbp),%rsi
	movzbl (%rsi),%edi
	call _toupper
	cmpl $85,%eax
	jnz L179
L177:
	movl $1,%r12d
	addq $1,-8(%rbp)
L179:
	movq -8(%rbp),%rsi
	movzbl (%rsi),%edi
	call _toupper
	cmpl $76,%eax
	jnz L182
L180:
	movl $1,%ebx
	addq $1,-8(%rbp)
L182:
	movq -8(%rbp),%rsi
	movzbl (%rsi),%edi
	call _toupper
	cmpl $85,%eax
	jnz L185
L183:
	movl $1,%r12d
	addq $1,-8(%rbp)
L185:
	movq _pos(%rip),%rsi
	cmpq -8(%rbp),%rsi
	jnz L193
L189:
	movl _errno(%rip),%esi
	cmpl $0,%esi
	jz L188
L193:
	movl $0,_errno(%rip)
	leaq -8(%rbp),%rbx
	movq _begin(%rip),%rdi
	movq %rbx,%rsi
	call _strtod
	movsd %xmm0,_token+8(%rip)
	movq -8(%rbp),%rsi
	movzbl (%rsi),%edi
	call _toupper
	cmpl $70,%eax
	jz L242
L256:
	cmpl $76,%eax
	jz L240
L237:
	movl $8,_token(%rip)
	jmp L238
L240:
	movl $9,_token(%rip)
	addq $1,-8(%rbp)
	jmp L238
L242:
	movq _begin(%rip),%rdi
	movq %rbx,%rsi
	call _strtof
	cvtss2sd %xmm0,%xmm0
	movsd %xmm0,_token+8(%rip)
	movl $7,_token(%rip)
	addq $1,-8(%rbp)
L238:
	movq _pos(%rip),%rsi
	cmpq -8(%rbp),%rsi
	jz L247
L245:
	pushq $L248
	pushq $1
	call _error
	addq $16,%rsp
L247:
	movl _errno(%rip),%esi
	cmpl $0,%esi
	jz L251
L249:
	pushq $L252
	pushq $1
	call _error
	addq $16,%rsp
L251:
	movl _token(%rip),%eax
	jmp L148
L188:
	movzbl %bl,%esi
	cmpl $0,%esi
	jz L196
L198:
	movzbl %r12b,%esi
	cmpl $0,%esi
	jz L196
L195:
	movl $6,%eax
	jmp L148
L196:
	movzbl %bl,%esi
	cmpl $0,%esi
	jz L204
L203:
	movq _token+8(%rip),%rsi
	movq $9223372036854775807,%rdi
	cmpq %rdi,%rsi
	jbe L207
L206:
	movl $6,%eax
	jmp L148
L207:
	movl $5,%eax
	jmp L148
L204:
	movzbl %r12b,%esi
	cmpl $0,%esi
	jz L212
L211:
	movq _token+8(%rip),%rsi
	movl $4294967295,%edi
	cmpq %rdi,%rsi
	jbe L215
L214:
	movl $6,%eax
	jmp L148
L215:
	movl $4,%eax
	jmp L148
L212:
	movq _token+8(%rip),%rsi
	movq $9223372036854775807,%rdi
	cmpq %rdi,%rsi
	jbe L220
L219:
	movl $6,%eax
	jmp L148
L220:
	movl $4294967295,%edi
	cmpq %rdi,%rsi
	jbe L224
L223:
	movl $5,%eax
	jmp L148
L224:
	cmpq $2147483647,%rsi
	jbe L228
L227:
	movq _begin(%rip),%rsi
	movzbl (%rsi),%esi
	cmpl $48,%esi
	jnz L231
L230:
	movl $4,%eax
	jmp L148
L231:
	movl $5,%eax
	jmp L148
L228:
	movl $3,%eax
L148:
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L260:
_lex0:
L262:
	pushq %rbp
	movq %rsp,%rbp
L263:
	movq _pos(%rip),%rsi
	movq %rsi,_begin(%rip)
	call _refill
L265:
	movq _pos(%rip),%rdi
	movzbq (%rdi),%rsi
	movl %esi,%eax
	movzbl ___ctype+1(%rax),%eax
	testl $8,%eax
	jz L267
L268:
	movzbl %sil,%esi
	cmpl $10,%esi
	jz L267
L272:
	addq $1,%rdi
	movq %rdi,_pos(%rip)
	movq _invalid(%rip),%rsi
	subq %rdi,%rsi
	cmpq $3,%rsi
	jge L273
L275:
	call _refill
L273:
	movq _pos(%rip),%rsi
	movq %rsi,_begin(%rip)
	jmp L265
L267:
	movq _pos(%rip),%rdi
	movzbl (%rdi),%esi
	cmpl $48,%esi
	jae L682
L614:
	cmpl $40,%esi
	jz L294
	jb L615
L632:
	cmpl $44,%esi
	jz L292
	jb L633
L640:
	cmpl $46,%esi
	jz L528
	jb L641
L644:
	cmpl $47,%esi
	jnz L279
L466:
	leaq 1(%rdi),%rsi
	movzbl 1(%rdi),%eax
	cmpl $61,%eax
	jnz L470
L468:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $45088819,%eax
	jmp L264
L470:
	movq %rsi,_pos(%rip)
	movl $202375198,%eax
	jmp L264
L641:
	cmpl $45,%esi
	jnz L279
L507:
	leaq 1(%rdi),%rax
	movzbl 1(%rdi),%ecx
	cmpl $62,%ecx
	jnz L517
L508:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $26,%eax
	jmp L264
L517:
	cmpl %esi,%ecx
	jnz L521
L519:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $28,%eax
	jmp L264
L521:
	cmpl $61,%ecx
	jnz L525
L523:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $95420464,%eax
	jmp L264
L525:
	movq %rax,_pos(%rip)
	movl $253755425,%eax
	jmp L264
L528:
	movzbl 1(%rdi),%esi
	cmpl $46,%esi
	jnz L531
L532:
	movzbl 2(%rdi),%esi
	cmpl $46,%esi
	jnz L531
L529:
	leaq 3(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $19,%eax
	jmp L264
L531:
	movq _pos(%rip),%rdi
	leaq 1(%rdi),%rsi
	movzbl 1(%rdi),%edi
	addl $-48,%edi
	cmpl $10,%edi
	jb L550
L537:
	movq %rsi,_pos(%rip)
	movl $18,%eax
	jmp L264
L633:
	cmpl $42,%esi
	jz L449
	jb L634
L637:
	cmpl $43,%esi
	jnz L279
L496:
	leaq 1(%rdi),%rax
	movzbl 1(%rdi),%ecx
	cmpl %esi,%ecx
	jnz L500
L498:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $27,%eax
	jmp L264
L500:
	cmpl $61,%ecx
	jnz L504
L502:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $78643249,%eax
	jmp L264
L504:
	movq %rax,_pos(%rip)
	movl $236978208,%eax
	jmp L264
L634:
	cmpl $41,%esi
	jnz L279
L296:
	leaq 1(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $13,%eax
	jmp L264
L449:
	leaq 1(%rdi),%rsi
	movzbl 1(%rdi),%eax
	cmpl $61,%eax
	jnz L453
L451:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $28311602,%eax
	jmp L264
L453:
	movq %rsi,_pos(%rip)
	movl $219414559,%eax
	jmp L264
L292:
	leaq 1(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $524309,%eax
	jmp L264
L615:
	cmpl $35,%esi
	jz L284
	jb L616
L625:
	cmpl $38,%esi
	jz L428
	jb L626
L629:
	cmpl $39,%esi
	jnz L279
L308:
	call _ccon
	jmp L264
L626:
	cmpl $37,%esi
	jnz L279
L483:
	leaq 1(%rdi),%rsi
	movzbl 1(%rdi),%eax
	cmpl $61,%eax
	jnz L487
L485:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $61866039,%eax
	jmp L264
L487:
	movq %rsi,_pos(%rip)
	movl $470810678,%eax
	jmp L264
L428:
	leaq 1(%rdi),%rax
	movzbl 1(%rdi),%ecx
	cmpl %esi,%ecx
	jnz L432
L430:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $395313195,%eax
	jmp L264
L432:
	cmpl $61,%ecx
	jnz L436
L434:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $145752108,%eax
	jmp L264
L436:
	movq %rax,_pos(%rip)
	movl $375390250,%eax
	jmp L264
L616:
	cmpl $33,%esi
	jz L339
	jb L617
L622:
	cmpl $34,%esi
	jnz L279
L310:
	call _strlit
	jmp L264
L617:
	cmpl $10,%esi
	jz L282
	ja L279
L618:
	cmpl $0,%esi
	jnz L279
L606:
	cmpq %rdi,_invalid(%rip)
	jnz L279
L607:
	xorl %eax,%eax
	jmp L264
L282:
	leaq 1(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $11,%eax
	jmp L264
L339:
	leaq 1(%rdi),%rsi
	movzbl 1(%rdi),%eax
	cmpl $61,%eax
	jnz L343
L341:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $424673333,%eax
	jmp L264
L343:
	movq %rsi,_pos(%rip)
	movl $29,%eax
	jmp L264
L284:
	leaq 1(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $10,%eax
	jmp L264
L294:
	leaq 1(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $262156,%eax
	jmp L264
L682:
	cmpl $57,%esi
	jbe L550
L647:
	cmpl $93,%esi
	jz L304
	jb L648
L666:
	cmpl $123,%esi
	jz L298
	jb L667
L675:
	cmpl $125,%esi
	jz L300
	jb L676
L679:
	cmpl $126,%esi
	jnz L279
L306:
	leaq 1(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $25,%eax
	jmp L264
L676:
	cmpl $124,%esi
	jnz L279
L411:
	leaq 1(%rdi),%rax
	movzbl 1(%rdi),%ecx
	cmpl %esi,%ecx
	jnz L415
L413:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $463470638,%eax
	jmp L264
L415:
	cmpl $61,%ecx
	jnz L419
L417:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $162529327,%eax
	jmp L264
L419:
	movq %rax,_pos(%rip)
	movl $444596269,%eax
	jmp L264
L300:
	leaq 1(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $17,%eax
	jmp L264
L667:
	cmpl $95,%esi
	jz L604
	jb L668
L671:
	cmpl $97,%esi
	jb L279
L674:
	cmpl $122,%esi
	jbe L604
	ja L279
L668:
	cmpl $94,%esi
	jnz L279
L398:
	leaq 1(%rdi),%rsi
	movzbl 1(%rdi),%eax
	cmpl $61,%eax
	jnz L402
L400:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $179306552,%eax
	jmp L264
L402:
	movq %rsi,_pos(%rip)
	movl $191889428,%eax
	jmp L264
L298:
	leaq 1(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $16,%eax
	jmp L264
L648:
	cmpl $62,%esi
	jz L368
	jb L649
L658:
	cmpl $65,%esi
	jae L665
L659:
	cmpl $63,%esi
	jnz L279
L286:
	leaq 1(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $24,%eax
	jmp L264
L665:
	cmpl $90,%esi
	jbe L604
L662:
	cmpl $91,%esi
	jnz L279
L302:
	leaq 1(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $14,%eax
	jmp L264
L604:
	call _ident
	jmp L264
L649:
	cmpl $60,%esi
	jz L347
	jb L650
L655:
	cmpl $61,%esi
	jnz L279
L318:
	leaq 1(%rdi),%rax
	movzbl 1(%rdi),%ecx
	cmpl %esi,%ecx
	jnz L326
L320:
	leaq 2(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $407896116,%eax
	jmp L264
L326:
	movq %rax,_pos(%rip)
	movl $11534393,%eax
	jmp L264
L650:
	cmpl $59,%esi
	jz L290
	ja L279
L651:
	cmpl $58,%esi
	jz L288
L279:
	movq _pos(%rip),%rsi
	movzbl (%rsi),%esi
	andl $255,%esi
	pushq %rsi
	pushq $L611
	pushq $1
	call _error
	addq $24,%rsp
	jmp L264
L288:
	leaq 1(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $486539286,%eax
	jmp L264
L290:
	leaq 1(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $524311,%eax
	jmp L264
L347:
	movzbl 1(%rdi),%eax
	cmpl %esi,%eax
	jnz L352
L353:
	movzbl 2(%rdi),%esi
	cmpl $61,%esi
	jnz L352
L350:
	leaq 3(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $112197673,%eax
	jmp L264
L352:
	movq _pos(%rip),%rcx
	leaq 1(%rcx),%rsi
	movzbl 1(%rcx),%edi
	movzbl (%rcx),%eax
	cmpl %eax,%edi
	jnz L360
L358:
	leaq 2(%rcx),%rsi
	movq %rsi,_pos(%rip)
	movl $338690087,%eax
	jmp L264
L360:
	cmpl $61,%edi
	jnz L364
L362:
	leaq 2(%rcx),%rsi
	movq %rsi,_pos(%rip)
	movl $356515880,%eax
	jmp L264
L364:
	movq %rsi,_pos(%rip)
	movl $322961446,%eax
	jmp L264
L368:
	movzbl 1(%rdi),%eax
	cmpl %esi,%eax
	jnz L373
L374:
	movzbl 2(%rdi),%esi
	cmpl $61,%esi
	jnz L373
L371:
	leaq 3(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $128974885,%eax
	jmp L264
L373:
	movq _pos(%rip),%rcx
	leaq 1(%rcx),%rsi
	movzbl 1(%rcx),%edi
	movzbl (%rcx),%eax
	cmpl %eax,%edi
	jnz L381
L379:
	leaq 2(%rcx),%rsi
	movq %rsi,_pos(%rip)
	movl $288358435,%eax
	jmp L264
L381:
	cmpl $61,%edi
	jnz L385
L383:
	leaq 2(%rcx),%rsi
	movq %rsi,_pos(%rip)
	movl $306184228,%eax
	jmp L264
L385:
	movq %rsi,_pos(%rip)
	movl $272629794,%eax
	jmp L264
L304:
	leaq 1(%rdi),%rsi
	movq %rsi,_pos(%rip)
	movl $15,%eax
	jmp L264
L550:
	call _number
L264:
	popq %rbp
	ret
L685:
.data
.align 8
_next:
	.int 11
	.space 12, 0
.text
_lex1:
L688:
	pushq %rbp
	movq %rsp,%rbp
L695:
	movl _next(%rip),%esi
	cmpl $0,%esi
	jnz L692
L691:
	call _lex0
	movl %eax,_token(%rip)
	jmp L690
L692:
	movups _next(%rip),%xmm0
	movups %xmm0,_token(%rip)
	movl $0,_next(%rip)
L690:
	popq %rbp
	ret
L697:
_lex_peek:
L698:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
L706:
	movq %rdi,%rbx
	movl _next(%rip),%esi
	cmpl $0,%esi
	jnz L703
L701:
	movups _token(%rip),%xmm0
	movups %xmm0,-16(%rbp)
	call _lex
	movups _token(%rip),%xmm0
	movups %xmm0,_next(%rip)
	movups -16(%rbp),%xmm0
	movups %xmm0,_token(%rip)
L703:
	movups _next(%rip),%xmm0
	movups %xmm0,(%rbx)
L700:
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L708:
_lex:
L709:
	pushq %rbp
	movq %rsp,%rbp
L729:
	call _lex1
L712:
	movl _token(%rip),%esi
	cmpl $11,%esi
	jnz L711
L713:
	addl $1,_error_line_no(%rip)
	call _lex1
	movl _token(%rip),%esi
	cmpl $10,%esi
	jnz L712
L715:
	call _lex1
	movl _token(%rip),%esi
	cmpl $3,%esi
	jnz L720
L718:
	movq _token+8(%rip),%rsi
	movl %esi,_error_line_no(%rip)
	call _lex1
	movl _token(%rip),%esi
	cmpl $2,%esi
	jnz L720
L721:
	movq _token+8(%rip),%rsi
	movq %rsi,_error_path(%rip)
	call _lex1
L720:
	movl _token(%rip),%esi
	cmpl $11,%esi
	jz L726
L724:
	pushq $L727
	pushq $1
	call _error
	addq $16,%rsp
L726:
	call _lex1
	jmp L712
L711:
	popq %rbp
	ret
L731:
_lex_init:
L732:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L740:
	movq %rdi,%rbx
	pushq $0
	pushq %rbx
	call _open
	addq $16,%rsp
	movl %eax,_fd(%rip)
	cmpl $-1,%eax
	jnz L737
L735:
	pushq %rbx
	pushq $L738
	pushq $1
	call _error
	addq $24,%rsp
L737:
	movq %rbx,%rdi
	call _strlen
	movq %rbx,%rdi
	movq %rax,%rsi
	call _string_new
	movq %rax,_error_path(%rip)
	call _lex
L734:
	popq %rbx
	popq %rbp
	ret
L742:
.data
.align 8
_text:
	.quad L744
	.quad L745
	.quad L746
	.quad L747
	.quad L747
	.quad L747
	.quad L747
	.quad L751
	.quad L751
	.quad L751
	.quad 0
	.quad 0
	.quad L754
	.quad L755
	.quad L756
	.quad L757
	.quad L758
	.quad L759
	.quad L760
	.quad L761
	.quad L762
	.quad L763
	.quad L764
	.quad L765
	.quad L766
	.quad L767
	.quad L768
	.quad L769
	.quad L770
	.quad L771
	.quad L772
	.quad L773
	.quad L774
	.quad L775
	.quad L776
	.quad L777
	.quad L778
	.quad L779
	.quad L780
	.quad L781
	.quad L782
	.quad L783
	.quad L784
	.quad L785
	.quad L786
	.quad L787
	.quad L788
	.quad L789
	.quad L790
	.quad L791
	.quad L792
	.quad L793
	.quad L794
	.quad L795
	.quad L796
	.quad L797
	.quad L798
	.quad L799
.text
_lex_print_k:
L800:
	pushq %rbp
	movq %rsp,%rbp
L814:
	movq %rdi,%rax
	movl %esi,%edi
	andl $255,%edi
	movslq %edi,%rcx
	cmpq $58,%rcx
	jae L804
L803:
	movslq %edi,%rsi
	movq _text(,%rsi,8),%rdi
	cmpq $0,%rdi
	jz L802
L806:
	movzbq (%rdi),%rsi
	movzbl ___ctype+1(%rsi),%esi
	testl $7,%esi
	jz L810
L809:
	movq %rax,%rsi
	call _fputs
	jmp L802
L810:
	pushq %rdi
	pushq $L812
	pushq %rax
	call _fprintf
	addq $24,%rsp
	jmp L802
L804:
	movq %rax,%rdi
	call _string_print_k
L802:
	popq %rbp
	ret
L816:
_lex_print_token:
L817:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L849:
	movq %rdi,%rbx
	movl _token(%rip),%esi
	movq %rbx,%rdi
	call _lex_print_k
	movl _token(%rip),%esi
	cmpl $3,%esi
	jz L825
L842:
	cmpl $4,%esi
	jz L829
L843:
	cmpl $5,%esi
	jz L825
L844:
	cmpl $6,%esi
	jz L829
L845:
	cmpl $7,%esi
	jb L846
L847:
	cmpl $9,%esi
	ja L846
L834:
	movsd _token+8(%rip),%xmm0
	subq $8,%rsp
	movsd %xmm0,(%rsp)
	pushq $L835
	pushq %rbx
	call _fprintf
	addq $24,%rsp
	jmp L819
L846:
	cmpl $262145,%esi
	jnz L819
L837:
	movq _token+8(%rip),%rsi
	addq $40,%rsi
	pushq %rsi
	pushq $L838
	pushq %rbx
	call _fprintf
	addq $24,%rsp
	jmp L819
L829:
	movq _token+8(%rip),%rsi
	pushq %rsi
	pushq $L830
	pushq %rbx
	call _fprintf
	addq $24,%rsp
	jmp L819
L825:
	movq _token+8(%rip),%rsi
	pushq %rsi
	pushq $L826
	pushq %rbx
	call _fprintf
	addq $24,%rsp
L819:
	popq %rbx
	popq %rbp
	ret
L851:
_lex_expect:
L852:
	pushq %rbp
	movq %rsp,%rbp
L853:
	movl _token(%rip),%esi
	cmpl %edi,%esi
	jz L854
L855:
	subq $16,%rsp
	movups _token(%rip),%xmm0
	movups %xmm0,(%rsp)
	pushq %rdi
	pushq $L858
	pushq $1
	call _error
	addq $40,%rsp
L854:
	popq %rbp
	ret
L862:
_lex_match:
L863:
	pushq %rbp
	movq %rsp,%rbp
L864:
	call _lex_expect
	call _lex
L865:
	popq %rbp
	ret
L869:
_k_decl:
L870:
	pushq %rbp
	movq %rsp,%rbp
L871:
	movl 16(%rbp),%esi
	testl $131072,%esi
	jnz L873
L876:
	cmpl $262145,%esi
	jnz L875
L880:
	movq 24(%rbp),%rdi
	call _symbol_typename
	cmpq $0,%rax
	jz L875
L873:
	movl $1,%eax
	jmp L872
L875:
	xorl %eax,%eax
L872:
	popq %rbp
	ret
L889:
L771:
	.byte 33,0
L745:
	.byte 105,100,101,110,116,105,102,105
	.byte 101,114,0
L796:
	.byte 37,0
L744:
	.byte 101,110,100,45,111,102,45,102
	.byte 105,108,101,0
L727:
	.byte 109,97,108,102,111,114,109,101
	.byte 100,32,100,105,114,101,99,116
	.byte 105,118,101,0
L785:
	.byte 38,38,0
L784:
	.byte 38,0
L20:
	.byte 116,111,107,101,110,32,116,111
	.byte 111,32,108,111,110,103,0
L754:
	.byte 40,0
L858:
	.byte 101,120,112,101,99,116,101,100
	.byte 32,37,107,32,40,102,111,117
	.byte 110,100,32,37,116,41,0
L755:
	.byte 41,0
L611:
	.byte 105,110,118,97,108,105,100,32
	.byte 99,104,97,114,97,99,116,101
	.byte 114,32,40,65,83,67,73,73
	.byte 32,37,100,41,0
L773:
	.byte 42,0
L774:
	.byte 43,0
L763:
	.byte 44,0
L830:
	.byte 32,91,37,108,117,93,0
L797:
	.byte 37,61,0
L795:
	.byte 33,61,0
L794:
	.byte 61,61,0
L793:
	.byte 47,61,0
L791:
	.byte 43,61,0
L790:
	.byte 45,61,0
L775:
	.byte 45,0
L768:
	.byte 45,62,0
L761:
	.byte 46,46,46,0
L760:
	.byte 46,0
L772:
	.byte 47,0
L751:
	.byte 102,108,111,97,116,105,110,103
	.byte 45,112,111,105,110,116,32,99
	.byte 111,110,115,116,97,110,116,0
L747:
	.byte 105,110,116,101,103,114,97,108
	.byte 32,99,111,110,115,116,97,110
	.byte 116,0
L248:
	.byte 109,97,108,102,111,114,109,101
	.byte 100,32,110,117,109,101,114,105
	.byte 99,32,99,111,110,115,116,97
	.byte 110,116,0
L139:
	.byte 109,117,108,116,105,45,99,104
	.byte 97,114,97,99,116,101,114,32
	.byte 99,111,110,115,116,97,110,116
	.byte 0
L135:
	.byte 109,97,108,102,111,114,109,101
	.byte 100,32,101,115,99,97,112,101
	.byte 32,115,101,113,117,101,110,99
	.byte 101,32,105,110,32,99,104,97
	.byte 114,97,99,116,101,114,32,99
	.byte 111,110,115,116,97,110,116,0
L81:
	.byte 117,110,116,101,114,109,105,110
	.byte 97,116,101,100,32,99,104,97
	.byte 114,97,99,116,101,114,32,99
	.byte 111,110,115,116,97,110,116,0
L252:
	.byte 102,108,111,97,116,105,110,103
	.byte 45,112,111,105,110,116,32,99
	.byte 111,110,115,116,97,110,116,32
	.byte 111,117,116,32,111,102,32,114
	.byte 97,110,103,101,0
L838:
	.byte 32,39,37,115,39,0
L812:
	.byte 39,37,115,39,0
L738:
	.byte 99,97,110,39,116,32,111,112
	.byte 101,110,32,39,37,115,39,32
	.byte 40,37,69,41,0
L24:
	.byte 105,110,112,117,116,32,73,47
	.byte 79,32,101,114,114,111,114,32
	.byte 40,37,69,41,0
L764:
	.byte 58,0
L769:
	.byte 43,43,0
L765:
	.byte 59,0
L758:
	.byte 123,0
L756:
	.byte 91,0
L788:
	.byte 124,124,0
L787:
	.byte 124,0
L781:
	.byte 60,60,0
L780:
	.byte 60,0
L746:
	.byte 115,116,114,105,110,103,32,108
	.byte 105,116,101,114,97,108,0
L112:
	.byte 109,97,108,102,111,114,109,101
	.byte 100,32,101,115,99,97,112,101
	.byte 32,115,101,113,117,101,110,99
	.byte 101,32,105,110,32,115,116,114
	.byte 105,110,103,32,108,105,116,101
	.byte 114,97,108,0
L82:
	.byte 117,110,116,101,114,109,105,110
	.byte 97,116,101,100,32,115,116,114
	.byte 105,110,103,32,108,105,116,101
	.byte 114,97,108,0
L835:
	.byte 32,91,37,102,93,0
L826:
	.byte 32,91,37,108,100,93,0
L799:
	.byte 61,0
L798:
	.byte 94,61,0
L792:
	.byte 42,61,0
L789:
	.byte 124,61,0
L786:
	.byte 38,61,0
L783:
	.byte 60,60,61,0
L782:
	.byte 60,61,0
L779:
	.byte 62,62,61,0
L778:
	.byte 62,61,0
L770:
	.byte 45,45,0
L759:
	.byte 125,0
L757:
	.byte 93,0
L777:
	.byte 62,62,0
L776:
	.byte 62,0
L767:
	.byte 126,0
L762:
	.byte 94,0
L766:
	.byte 63,0
.globl _error
.globl _toupper
.local _fd
.comm _fd, 4, 4
.globl _lex_expect
.globl _lex_init
.globl _escape
.globl ___ctype
.globl _memmove
.globl _fprintf
.globl _string_new
.globl _lex
.comm _error_path, 8, 8
.globl _error_path
.comm _error_line_no, 4, 4
.globl _error_line_no
.globl _errno
.globl _fputs
.globl _read
.globl _strtod
.globl _symbol_typename
.globl _close
.local _buf
.comm _buf, 4096, 1
.globl _strtof
.globl _lex_match
.globl _string_print_k
.globl _lex_print_k
.globl _lex_peek
.globl _k_decl
.globl _strtoul
.globl _lex_print_token
.comm _token, 16, 8
.globl _token
.globl _open
.globl _strlen
