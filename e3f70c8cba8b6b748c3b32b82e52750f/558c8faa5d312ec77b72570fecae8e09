.text
_unary:
L3:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L49:
	movq %rdi,%rbx
	movq (%rbx),%rdi
	cmpq $0,%rdi
	jz L8
L6:
	movl (%rdi),%esi
	cmpl $51,%esi
	jz L33
L40:
	cmpl $53,%esi
	jz L31
L41:
	cmpl $536870926,%esi
	jz L19
L42:
	cmpl $536870927,%esi
	jz L21
L43:
	cmpl $536870958,%esi
	jz L17
L44:
	cmpl $536871424,%esi
	jz L13
L45:
	cmpl $536871425,%esi
	jz L15
L46:
	cmpl $-2147483592,%esi
	jb L8
L48:
	cmpl $-2147483591,%esi
	jbe L5
	ja L8
L15:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
	movq %rbx,%rdi
	call _unary
	movq (%rbx),%rsi
	movq 8(%rsi),%rdi
	negq %rdi
	movq %rdi,8(%rsi)
	jmp L5
L13:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
	movq %rbx,%rdi
	call _unary
	jmp L5
L17:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
	movq %rbx,%rdi
	call _unary
	movq (%rbx),%rsi
	movq 8(%rsi),%rdi
	notq %rdi
	movq %rdi,8(%rsi)
	jmp L5
L21:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
	movq %rbx,%rdi
	call _expression
	movq (%rbx),%rsi
	movq 32(%rsi),%r12
	cmpq $0,%r12
	jz L22
L25:
	movl (%r12),%esi
	cmpl $536870928,%esi
	jz L24
L22:
	pushq $L29
	call _error
	addq $8,%rsp
L24:
	movq %rbx,%rdi
	movq %r12,%rsi
	call _list_drop
	jmp L5
L19:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
	movq %rbx,%rdi
	call _unary
	movq (%rbx),%rsi
	movq 8(%rsi),%rdi
	cmpq $0,%rdi
	setz %dil
	movzbq %dil,%rdi
	movq %rdi,8(%rsi)
	jmp L5
L31:
	call _token_convert_char
	jmp L5
L33:
	call _token_convert_number
	jmp L5
L8:
	pushq $L37
	call _error
	addq $8,%rsp
L5:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L51:
_binary:
L53:
	pushq %rbp
	movq %rsp,%rbp
	subq $24,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
L201:
	movq %rsi,%r12
	movl %edi,%r13d
	cmpl $0,%r13d
	jnz L58
L56:
	movq %r12,%rdi
	call _unary
	jmp L55
L58:
	leal -256(%r13),%edi
	movq %r12,%rsi
	call _binary
L61:
	movq (%r12),%rdi
	leaq -8(%rbp),%rsi
	movq %rdi,-8(%rbp)
	movq 32(%rdi),%rdi
	leaq -16(%rbp),%rbx
	movq %rdi,-16(%rbp)
	cmpq $0,%rdi
	jz L55
L67:
	movl (%rdi),%edi
	andl $3840,%edi
	cmpl %r13d,%edi
	jz L66
L55:
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L66:
	movq %r12,%rdi
	call _list_pop
	movq %r12,%rdi
	movq %rbx,%rsi
	call _list_pop
	leal -256(%r13),%edi
	movq %r12,%rsi
	call _binary
	leaq -24(%rbp),%rsi
	movq %r12,%rdi
	call _list_pop
	xorl %edi,%edi
	call _token_int
	movq %rax,%rbx
	movq -8(%rbp),%rsi
	movl (%rsi),%esi
	cmpl $2147483705,%esi
	jz L72
L75:
	movq -24(%rbp),%rsi
	movl (%rsi),%esi
	cmpl $2147483705,%esi
	jnz L74
L72:
	movl $-2147483591,(%rbx)
L74:
	movq -16(%rbp),%rsi
	movl (%rsi),%esi
	cmpl $536871971,%esi
	jz L120
	jb L165
L184:
	cmpl $536872711,%esi
	jz L93
	jb L185
L194:
	cmpl $536873247,%esi
	jz L144
	jb L195
L198:
	cmpl $536873505,%esi
	jnz L80
L150:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	cmpq $0,%rsi
	jnz L152
L151:
	movq -24(%rbp),%rsi
	movq 8(%rsi),%rsi
	cmpq $0,%rsi
	jz L153
L152:
	movl $1,%esi
	jmp L154
L153:
	xorl %esi,%esi
L154:
	movslq %esi,%rsi
	movq %rsi,8(%rbx)
	movl $-2147483592,(%rbx)
	jmp L157
L195:
	cmpl $536872966,%esi
	jnz L80
L91:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rdi
	orq %rdi,%rsi
	movq %rsi,8(%rbx)
	jmp L157
L144:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	cmpq $0,%rsi
	jz L147
L145:
	movq -24(%rbp),%rsi
	movq 8(%rsi),%rsi
	cmpq $0,%rsi
	jz L147
L146:
	movl $1,%esi
	jmp L148
L147:
	xorl %esi,%esi
L148:
	movslq %esi,%rsi
	movq %rsi,8(%rbx)
	movl $-2147483592,(%rbx)
	jmp L157
L185:
	cmpl $536872214,%esi
	jz L142
	jb L186
L191:
	cmpl $536872453,%esi
	jnz L80
L89:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rdi
	andq %rdi,%rsi
	movq %rsi,8(%rbx)
	jmp L157
L186:
	cmpl $536872213,%esi
	jz L140
	ja L80
L187:
	cmpl $536871973,%esi
	jnz L80
L135:
	movl (%rbx),%esi
	cmpl $2147483705,%esi
	jnz L137
L136:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	cmpq 8(%rdi),%rsi
	setbe %sil
	movzbq %sil,%rsi
	movq %rsi,8(%rbx)
	jmp L138
L137:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rdi
	cmpq %rdi,%rsi
	setle %sil
	movzbq %sil,%rsi
	movq %rsi,8(%rbx)
L138:
	movl $-2147483592,(%rbx)
	jmp L157
L140:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rdi
	cmpq %rdi,%rsi
	setz %sil
	movzbq %sil,%rsi
	movq %rsi,8(%rbx)
	movl $-2147483592,(%rbx)
	jmp L157
L142:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rdi
	cmpq %rdi,%rsi
	setnz %sil
	movzbq %sil,%rsi
	movq %rsi,8(%rbx)
	movl $-2147483592,(%rbx)
	jmp L157
L93:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rdi
	xorq %rdi,%rsi
	movq %rsi,8(%rbx)
	jmp L157
L165:
	cmpl $536871425,%esi
	jz L85
	jb L166
L175:
	cmpl $536871944,%esi
	jz L125
	jb L176
L181:
	cmpl $536871946,%esi
	jnz L80
L130:
	movl (%rbx),%esi
	cmpl $2147483705,%esi
	jnz L132
L131:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	cmpq 8(%rdi),%rsi
	setb %sil
	movzbq %sil,%rsi
	movq %rsi,8(%rbx)
	jmp L133
L132:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rdi
	cmpq %rdi,%rsi
	setl %sil
	movzbq %sil,%rsi
	movq %rsi,8(%rbx)
L133:
	movl $-2147483592,(%rbx)
	jmp L157
L176:
	cmpl $536871691,%esi
	jz L95
	ja L80
L177:
	cmpl $536871689,%esi
	jnz L80
L97:
	movl (%rbx),%esi
	cmpl $2147483705,%esi
	jnz L99
L98:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rcx
	shrq %cl,%rsi
	movq %rsi,8(%rbx)
	jmp L157
L99:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rcx
	sarq %cl,%rsi
	movq %rsi,8(%rbx)
	jmp L157
L95:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rcx
	shlq %cl,%rsi
	movq %rsi,8(%rbx)
	jmp L157
L125:
	movl (%rbx),%esi
	cmpl $2147483705,%esi
	jnz L127
L126:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	cmpq 8(%rdi),%rsi
	seta %sil
	movzbq %sil,%rsi
	movq %rsi,8(%rbx)
	jmp L128
L127:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rdi
	cmpq %rdi,%rsi
	setg %sil
	movzbq %sil,%rsi
	movq %rsi,8(%rbx)
L128:
	movl $-2147483592,(%rbx)
	jmp L157
L166:
	cmpl $536871172,%esi
	jz L102
	jb L167
L172:
	cmpl $536871424,%esi
	jnz L80
L83:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rdi
	addq %rdi,%rsi
	movq %rsi,8(%rbx)
	jmp L157
L167:
	cmpl $536871171,%esi
	jz L111
	ja L80
L168:
	cmpl $536871170,%esi
	jz L87
L80:
	pushq $L156
	call _error
	addq $8,%rsp
	jmp L157
L87:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rdi
	imulq %rdi,%rsi
	movq %rsi,8(%rbx)
	jmp L157
L111:
	movq -24(%rbp),%rsi
	movq 8(%rsi),%rsi
	cmpq $0,%rsi
	jnz L114
L112:
	pushq $L115
	call _error
	addq $8,%rsp
L114:
	movl (%rbx),%esi
	cmpl $2147483705,%esi
	jnz L117
L116:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rax
	movq -24(%rbp),%rsi
	movq 8(%rsi),%rsi
	xorl %edx,%edx
	divq %rsi
	movq %rax,8(%rbx)
	jmp L157
L117:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rax
	movq -24(%rbp),%rsi
	movq 8(%rsi),%rsi
	cqto
	idivq %rsi
	movq %rax,8(%rbx)
	jmp L157
L102:
	movq -24(%rbp),%rsi
	movq 8(%rsi),%rsi
	cmpq $0,%rsi
	jnz L105
L103:
	pushq $L106
	call _error
	addq $8,%rsp
L105:
	movl (%rbx),%esi
	cmpl $2147483705,%esi
	jnz L108
L107:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rax
	movq -24(%rbp),%rsi
	movq 8(%rsi),%rsi
	xorl %edx,%edx
	divq %rsi
	movq %rdx,8(%rbx)
	jmp L157
L108:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rax
	movq -24(%rbp),%rsi
	movq 8(%rsi),%rsi
	cqto
	idivq %rsi
	movq %rdx,8(%rbx)
	jmp L157
L85:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rdi
	subq %rdi,%rsi
	movq %rsi,8(%rbx)
	jmp L157
L120:
	movl (%rbx),%esi
	cmpl $2147483705,%esi
	jnz L122
L121:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	cmpq 8(%rdi),%rsi
	setae %sil
	movzbq %sil,%rsi
	movq %rsi,8(%rbx)
	jmp L123
L122:
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	movq -24(%rbp),%rdi
	movq 8(%rdi),%rdi
	cmpq %rdi,%rsi
	setge %sil
	movzbq %sil,%rsi
	movq %rsi,8(%rbx)
L123:
	movl $-2147483592,(%rbx)
L157:
	movq (%r12),%rdi
	leaq 32(%rbx),%rsi
	movq %rdi,32(%rbx)
	cmpq $0,%rdi
	jz L161
L160:
	movq (%r12),%rdi
	movq %rsi,40(%rdi)
	jmp L162
L161:
	movq %rsi,8(%r12)
L162:
	movq %rbx,(%r12)
	movq %r12,40(%rbx)
	movq -8(%rbp),%rdi
	call _token_free
	movq -16(%rbp),%rdi
	call _token_free
	movq -24(%rbp),%rdi
	call _token_free
	jmp L61
L203:
_ternary:
L205:
	pushq %rbp
	movq %rsp,%rbp
	subq $24,%rsp
	pushq %rbx
	pushq %r12
L235:
	movq %rdi,%rbx
	movl $2560,%edi
	movq %rbx,%rsi
	call _binary
	movq (%rbx),%rsi
	leaq -8(%rbp),%r12
	movq %rsi,-8(%rbp)
	movq %rbx,%rdi
	movl $536870957,%edx
	call _list_next_is
	cmpl $0,%eax
	jz L207
L208:
	movq %rbx,%rdi
	movq %r12,%rsi
	call _list_pop
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
	movq %rbx,%rdi
	call _expression
	leaq -16(%rbp),%rsi
	movq %rbx,%rdi
	call _list_pop
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L211
L214:
	movl (%rsi),%esi
	cmpl $536870956,%esi
	jz L213
L211:
	pushq $L218
	call _error
	addq $8,%rsp
L213:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
	movq %rbx,%rdi
	call _ternary
	leaq -24(%rbp),%rsi
	movq %rbx,%rdi
	call _list_pop
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rsi
	cmpq $0,%rsi
	jz L228
L222:
	movq (%rbx),%rsi
	movq -16(%rbp),%rdi
	movq %rsi,32(%rdi)
	cmpq $0,%rsi
	jz L226
L225:
	movq -16(%rbp),%rdi
	movq (%rbx),%rsi
	addq $32,%rdi
	movq %rdi,40(%rsi)
	jmp L227
L226:
	movq -16(%rbp),%rsi
	addq $32,%rsi
	movq %rsi,8(%rbx)
L227:
	movq -16(%rbp),%rsi
	movq %rsi,(%rbx)
	movq -16(%rbp),%rsi
	movq %rbx,40(%rsi)
	movq -24(%rbp),%rdi
	call _token_free
	jmp L221
L228:
	movq (%rbx),%rsi
	movq -24(%rbp),%rdi
	movq %rsi,32(%rdi)
	cmpq $0,%rsi
	jz L232
L231:
	movq -24(%rbp),%rdi
	movq (%rbx),%rsi
	addq $32,%rdi
	movq %rdi,40(%rsi)
	jmp L233
L232:
	movq -24(%rbp),%rsi
	addq $32,%rsi
	movq %rsi,8(%rbx)
L233:
	movq -24(%rbp),%rsi
	movq %rsi,(%rbx)
	movq -24(%rbp),%rsi
	movq %rbx,40(%rsi)
	movq -16(%rbp),%rdi
	call _token_free
L221:
	movq -8(%rbp),%rdi
	call _token_free
L207:
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L237:
_expression:
L238:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L250:
	movq %rdi,%rbx
L242:
	movq %rbx,%rdi
	call _ternary
	movq (%rbx),%rsi
	movq %rbx,%rdi
	movl $536870959,%edx
	call _list_next_is
	cmpl $0,%eax
	jz L240
L245:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
	jmp L242
L240:
	popq %rbx
	popq %rbp
	ret
L252:
_undefined:
L254:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L264:
	movq %rdi,%rbx
	movq (%rbx),%r12
L257:
	cmpq $0,%r12
	jz L256
L258:
	movl (%r12),%esi
	cmpl $49,%esi
	jnz L261
L260:
	movq %rbx,%rdi
	movq %r12,%rsi
	call _list_drop
	movq %rax,%r13
	movq %r13,%r12
	xorl %edi,%edi
	call _token_int
	movq %rbx,%rdi
	movq %r13,%rsi
	movq %rax,%rdx
	call _list_insert
	jmp L257
L261:
	movq 32(%r12),%r12
	jmp L257
L256:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L266:
_defined:
L268:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L319:
	movq %rdi,%rbx
	movq (%rbx),%r12
L271:
	cmpq $0,%r12
	jz L270
L272:
	movl (%r12),%esi
	cmpl $49,%esi
	jnz L275
L281:
	leaq 8(%r12),%rdi
	call _macro_lookup
	cmpq $0,%rax
	jz L275
L277:
	movl 24(%rax),%esi
	testl $1,%esi
	jz L275
L274:
	movq %rbx,%rdi
	movq %r12,%rsi
	call _list_drop
	movq %rax,%r12
	cmpq $0,%rax
	jz L286
L288:
	movl (%rax),%esi
	cmpl $536870927,%esi
	jnz L286
L285:
	movq %rbx,%rdi
	movq %rax,%rsi
	call _list_drop
	movq %rax,%r12
	movl $1,%r14d
	jmp L287
L286:
	xorl %r14d,%r14d
L287:
	cmpq $0,%r12
	jz L292
L295:
	movl (%r12),%esi
	cmpl $49,%esi
	jz L294
L292:
	pushq $L299
	call _error
	addq $8,%rsp
L294:
	leaq 8(%r12),%rdi
	call _macro_lookup
	cmpq $0,%rax
	jz L301
L303:
	movl 24(%rax),%esi
	testl $1,%esi
	jnz L301
L300:
	movl $1,%edi
	call _token_int
	movq %rbx,%rdi
	movq %r12,%rsi
	movq %rax,%rdx
	call _list_insert
	jmp L302
L301:
	xorl %edi,%edi
	call _token_int
	movq %rbx,%rdi
	movq %r12,%rsi
	movq %rax,%rdx
	call _list_insert
L302:
	movq %rbx,%rdi
	movq %r12,%rsi
	call _list_drop
	movq %rax,%r13
	movq %r13,%r12
	cmpl $0,%r14d
	jz L271
L307:
	cmpq $0,%r13
	jz L310
L313:
	movl (%r13),%esi
	cmpl $536870928,%esi
	jz L312
L310:
	pushq $L317
	call _error
	addq $8,%rsp
L312:
	movq %rbx,%rdi
	movq %r13,%rsi
	call _list_drop
	movq %rax,%r12
	jmp L271
L275:
	movq 32(%r12),%r12
	jmp L271
L270:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L321:
_evaluate:
L322:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L343:
	movq %rdi,%rbx
	call _list_strip_all
	movq %rbx,%rdi
	call _defined
	movq %rbx,%rdi
	call _macro_replace_all
	movq %rbx,%rdi
	call _undefined
	movq %rbx,%rdi
	call _list_strip_all
	movq %rbx,%rdi
	call _expression
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L328
L332:
	movl (%rsi),%esi
	cmpl $2147483704,%esi
	jz L327
L328:
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L325
L336:
	movl (%rsi),%esi
	cmpl $2147483705,%esi
	jz L327
L325:
	pushq $L340
	call _error
	addq $8,%rsp
L327:
	movq (%rbx),%rsi
	movq 8(%rsi),%rdi
	cmpq $0,%rdi
	setnz %dil
	movzbl %dil,%r12d
	movq %rbx,%rdi
	call _list_drop
	movl %r12d,%eax
L324:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L345:
L156:
	.byte 67,80,80,32,73,78,84,69
	.byte 82,78,65,76,58,32,117,110
	.byte 107,110,111,119,110,32,98,105
	.byte 110,97,114,121,32,111,112,101
	.byte 114,97,116,111,114,0
L317:
	.byte 109,105,115,115,105,110,103,32
	.byte 99,108,111,115,105,110,103,32
	.byte 112,97,114,101,110,116,104,101
	.byte 115,105,115,32,97,102,116,101
	.byte 114,32,39,100,101,102,105,110
	.byte 101,100,39,0
L299:
	.byte 109,105,115,115,105,110,103,32
	.byte 105,100,101,110,116,105,102,105
	.byte 101,114,32,97,102,116,101,114
	.byte 32,39,100,101,102,105,110,101
	.byte 100,39,0
L115:
	.byte 100,105,118,105,115,105,111,110
	.byte 32,98,121,32,122,101,114,111
	.byte 0
L106:
	.byte 109,111,100,117,108,117,115,32
	.byte 122,101,114,111,0
L218:
	.byte 109,105,115,115,105,110,103,32
	.byte 39,58,39,32,97,102,116,101
	.byte 114,32,39,63,39,0
L340:
	.byte 67,80,80,32,73,78,84,69
	.byte 82,78,65,76,58,32,98,111
	.byte 116,99,104,101,100,32,101,120
	.byte 112,114,101,115,115,105,111,110
	.byte 32,101,118,97,108,117,97,116
	.byte 105,111,110,0
L37:
	.byte 109,97,108,102,111,114,109,101
	.byte 100,32,101,120,112,114,101,115
	.byte 115,105,111,110,0
L29:
	.byte 117,110,109,97,116,99,104,101
	.byte 100,32,112,97,114,101,110,116
	.byte 104,101,115,101,115,32,105,110
	.byte 32,101,120,112,114,101,115,115
	.byte 105,111,110,0
.globl _macro_lookup
.globl _list_drop
.globl _list_pop
.globl _token_convert_char
.globl _token_convert_number
.globl _error
.globl _list_next_is
.globl _evaluate
.globl _macro_replace_all
.globl _list_strip_all
.globl _list_insert
.globl _token_int
.globl _token_free
