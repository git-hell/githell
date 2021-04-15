.data
.align 1
_directives:
	.byte 100,101,102,105,110,101,0,0
	.byte 101,108,105,102,0,0,0,0
	.byte 101,108,115,101,0,0,0,0
	.byte 101,110,100,105,102,0,0,0
	.byte 101,114,114,111,114,0,0,0
	.byte 105,102,0,0,0,0,0,0
	.byte 105,102,100,101,102,0,0,0
	.byte 105,102,110,100,101,102,0,0
	.byte 105,110,99,108,117,100,101,0
	.byte 108,105,110,101,0,0,0,0
	.byte 112,114,97,103,109,97,0,0
	.byte 117,110,100,101,102,0,0,0
.text
_lookup:
L3:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L22:
	movq %rdi,%r12
	movl (%r12),%esi
	cmpl $49,%esi
	jnz L8
L6:
	xorl %ebx,%ebx
L9:
	movslq %ebx,%rsi
	cmpq $12,%rsi
	jae L8
L10:
	movl 8(%r12),%esi
	testl $1,%esi
	jz L17
L16:
	leaq 9(%r12),%rsi
	jmp L18
L17:
	movq 24(%r12),%rsi
L18:
	movslq %ebx,%rdi
	leaq _directives(,%rdi,8),%rdi
	call _strcmp
	cmpl $0,%eax
	jnz L11
L13:
	movl %ebx,%eax
	jmp L5
L11:
	addl $1,%ebx
	jmp L9
L8:
	movl $-1,%eax
L5:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L24:
_state_push:
L27:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L40:
	movl %edi,%ebx
	movl $16,%edi
	call _safe_malloc
	movq _state_stack(%rip),%rsi
	cmpq $0,%rsi
	jz L30
L33:
	movzbl (%rsi),%esi
	cmpl $0,%esi
	jnz L30
L31:
	movb $0,(%rax)
	movb $1,1(%rax)
	jmp L32
L30:
	movb %bl,1(%rax)
	movb %bl,(%rax)
L32:
	movb $0,2(%rax)
	movq _state_stack(%rip),%rsi
	movq %rsi,8(%rax)
	movq %rax,_state_stack(%rip)
L29:
	popq %rbx
	popq %rbp
	ret
L42:
_state_pop:
L44:
	pushq %rbp
	movq %rsp,%rbp
L45:
	movq _state_stack(%rip),%rdi
	movq 8(%rdi),%rsi
	movq %rsi,_state_stack(%rip)
	call _free
L46:
	popq %rbp
	ret
L53:
_do_ifdef:
L55:
	pushq %rbp
	movq %rsp,%rbp
	subq $8,%rsp
	pushq %rbx
L66:
	movl %edi,%ebx
	leaq -8(%rbp),%rdx
	movq %rsi,%rdi
	movl $49,%esi
	call _list_match
	movq -8(%rbp),%rsi
	leaq 8(%rsi),%rdi
	call _macro_lookup
	cmpq $0,%rax
	jz L60
L58:
	movl 24(%rax),%esi
	testl $1,%esi
	jnz L60
L59:
	movl $1,%edi
	jmp L61
L60:
	xorl %edi,%edi
L61:
	cmpl $6,%ebx
	jnz L63
L62:
	call _state_push
	jmp L64
L63:
	cmpl $0,%edi
	setz %sil
	movzbl %sil,%edi
	call _state_push
L64:
	movq -8(%rbp),%rdi
	call _token_free
L57:
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L68:
_do_if:
L70:
	pushq %rbp
	movq %rsp,%rbp
L71:
	movq _state_stack(%rip),%rsi
	cmpq $0,%rsi
	jz L73
L76:
	movzbl (%rsi),%esi
	cmpl $0,%esi
	jnz L73
L74:
	xorl %edi,%edi
	call _state_push
	jmp L72
L73:
	call _evaluate
	movl %eax,%edi
	call _state_push
L72:
	popq %rbp
	ret
L82:
_do_elif:
L84:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L99:
	movq %rdi,%rbx
	cmpq $0,_state_stack(%rip)
	jnz L89
L87:
	pushq $L90
	call _error
	addq $8,%rsp
L89:
	movq _state_stack(%rip),%rsi
	movzbl 2(%rsi),%esi
	cmpl $0,%esi
	jz L93
L91:
	pushq $L94
	call _error
	addq $8,%rsp
L93:
	movq _state_stack(%rip),%rsi
	movzbl 1(%rsi),%esi
	cmpl $0,%esi
	jnz L86
L95:
	movq %rbx,%rdi
	call _evaluate
	movq _state_stack(%rip),%rsi
	movb %al,1(%rsi)
	movq _state_stack(%rip),%rsi
	movzbl 1(%rsi),%edi
	movb %dil,(%rsi)
L86:
	popq %rbx
	popq %rbp
	ret
L101:
_do_else:
L103:
	pushq %rbp
	movq %rsp,%rbp
L104:
	cmpq $0,_state_stack(%rip)
	jnz L108
L106:
	pushq $L109
	call _error
	addq $8,%rsp
L108:
	movq _state_stack(%rip),%rsi
	movzbl 2(%rsi),%esi
	cmpl $0,%esi
	jz L112
L110:
	pushq $L113
	call _error
	addq $8,%rsp
L112:
	movq _state_stack(%rip),%rsi
	movzbl 1(%rsi),%edi
	cmpl $0,%edi
	setz %dil
	movzbl %dil,%edi
	movb %dil,(%rsi)
	movq _state_stack(%rip),%rsi
	movb $1,2(%rsi)
L105:
	popq %rbp
	ret
L117:
_do_endif:
L119:
	pushq %rbp
	movq %rsp,%rbp
L127:
	cmpq $0,_state_stack(%rip)
	jnz L124
L122:
	pushq $L125
	call _error
	addq $8,%rsp
L124:
	call _state_pop
L121:
	popq %rbp
	ret
L129:
_do_line:
L131:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
	pushq %r12
L141:
	movq %rdi,%rbx
	leaq -16(%rbp),%r12
	movq $0,-16(%rbp)
	movq %rbx,%rdi
	call _list_strip_all
	leaq -8(%rbp),%rdx
	movq %rbx,%rdi
	movl $51,%esi
	call _list_match
	cmpq $0,(%rbx)
	jz L136
L134:
	movq %rbx,%rdi
	movl $52,%esi
	movq %r12,%rdx
	call _list_match
L136:
	movq -8(%rbp),%rdi
	call _token_convert_number
	movq -8(%rbp),%rsi
	movq 8(%rsi),%rdi
	movq _input_stack(%rip),%rsi
	addq $-1,%rdi
	movl %edi,32(%rsi)
	cmpq $0,-16(%rbp)
	jz L139
L137:
	movq _input_stack(%rip),%rsi
	leaq 8(%rsi),%rdi
	call _vstring_clear
	movq -16(%rbp),%rsi
	movq _input_stack(%rip),%rdi
	addq $8,%rsi
	addq $8,%rdi
	call _vstring_concat
	movq -16(%rbp),%rdi
	call _token_free
L139:
	movq -8(%rbp),%rdi
	call _token_free
	movb $1,_need_sync(%rip)
L133:
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L143:
_do_error:
L145:
	pushq %rbp
	movq %rsp,%rbp
L146:
	call _list_stringize
	movl 8(%rax),%esi
	testl $1,%esi
	jz L150
L149:
	leaq 9(%rax),%rsi
	jmp L151
L150:
	movq 24(%rax),%rsi
L151:
	pushq %rsi
	pushq $L148
	call _error
	addq $16,%rsp
L147:
	popq %rbp
	ret
L155:
_do_include:
L157:
	pushq %rbp
	movq %rsp,%rbp
	subq $32,%rsp
	pushq %rbx
	pushq %r12
L205:
	movq %rdi,%rbx
	xorps %xmm0,%xmm0
	movups %xmm0,-24(%rbp)
	movq $0,-8(%rbp)
	orl $1,-24(%rbp)
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L163
L167:
	movl (%rsi),%esi
	cmpl $52,%esi
	jz L162
L163:
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L160
L171:
	movl (%rsi),%esi
	cmpl $536871946,%esi
	jz L162
L160:
	movq %rbx,%rdi
	call _macro_replace_all
	movq %rbx,%rdi
	call _list_strip_ends
L162:
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L176
L178:
	movl (%rsi),%esi
	cmpl $52,%esi
	jnz L176
L175:
	leaq -32(%rbp),%rsi
	movq %rbx,%rdi
	call _list_pop
	leaq -24(%rbp),%rsi
	movq -32(%rbp),%rdi
	call _token_dequote
	movq -32(%rbp),%rdi
	call _token_free
	movl $2,%r12d
	jmp L177
L176:
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L183
L185:
	movl (%rsi),%esi
	cmpl $536871946,%esi
	jnz L183
L182:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_pop
L189:
	movq (%rbx),%rsi
	cmpq $0,%rsi
	jz L191
L192:
	cmpq $0,%rsi
	jz L190
L196:
	movl (%rsi),%esi
	cmpl $536871944,%esi
	jz L191
L190:
	leaq -32(%rbp),%rsi
	movq %rbx,%rdi
	call _list_pop
	leaq -24(%rbp),%rsi
	movq -32(%rbp),%rdi
	call _token_text
	movq -32(%rbp),%rdi
	call _token_free
	jmp L189
L191:
	movq %rbx,%rdi
	movl $536871944,%esi
	xorl %edx,%edx
	call _list_match
	movl $1,%r12d
	jmp L177
L183:
	pushq $L200
	call _error
	addq $8,%rsp
L177:
	movl -24(%rbp),%esi
	testl $1,%esi
	jz L202
L201:
	leaq -23(%rbp),%rdi
	jmp L203
L202:
	movq -8(%rbp),%rdi
L203:
	movl %r12d,%esi
	call _input_open
	leaq -24(%rbp),%rdi
	call _vstring_free
L159:
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L207:
_directive:
L208:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L324:
	movq %rdi,%rbx
	movq (%rbx),%rdi
	call _list_skip_spaces
	cmpq $0,%rax
	jz L224
L214:
	movl (%rax),%esi
	cmpl $1610612748,%esi
	jnz L224
L211:
	movq 32(%rax),%rdi
	call _list_skip_spaces
	movq %rax,%r13
	movq %r13,%rdi
	cmpq $0,%r13
	jz L219
L218:
	movq %r13,%rdi
	call _lookup
	movl %eax,%r12d
	movq 32(%r13),%rdi
	cmpl $10,%eax
	jnz L223
	jz L224
L219:
	movl $12,%r12d
L223:
	call _list_skip_spaces
	movq %rbx,%rdi
	movq %rax,%rsi
	call _list_cut
	cmpl $6,%r12d
	jae L323
L299:
	cmpl $3,%r12d
	jz L241
	jb L300
L307:
	cmpl $5,%r12d
	jz L235
	ja L228
L308:
	cmpl $4,%r12d
	jnz L228
L270:
	movq _state_stack(%rip),%rsi
	cmpq $0,%rsi
	jz L271
L274:
	movzbl (%rsi),%esi
	cmpl $0,%esi
	jz L228
L271:
	movq %rbx,%rdi
	call _do_error
	jmp L228
L235:
	movq %rbx,%rdi
	call _do_if
	jmp L228
L300:
	cmpl $1,%r12d
	jz L237
	jb L301
L304:
	cmpl $2,%r12d
	jnz L228
L239:
	movq %rbx,%rdi
	call _do_else
	jmp L228
L301:
	cmpl $0,%r12d
	jnz L228
L243:
	movq _state_stack(%rip),%rsi
	cmpq $0,%rsi
	jz L244
L247:
	movzbl (%rsi),%esi
	cmpl $0,%esi
	jz L228
L244:
	movq %rbx,%rdi
	call _macro_define
	jmp L228
L237:
	movq %rbx,%rdi
	call _do_elif
	jmp L228
L241:
	call _do_endif
	jmp L228
L323:
	cmpl $7,%r12d
	jbe L233
L312:
	cmpl $11,%r12d
	jz L251
	jb L313
L318:
	cmpl $-1,%r12d
	jz L259
	ja L228
L319:
	cmpl $12,%r12d
	jz L228
	jnz L228
L259:
	movq _state_stack(%rip),%rsi
	cmpq $0,%rsi
	jz L260
L263:
	movzbl (%rsi),%esi
	cmpl $0,%esi
	jz L228
L260:
	pushq $L266
	call _error
	addq $8,%rsp
	jmp L228
L313:
	cmpl $9,%r12d
	jz L268
	ja L228
L314:
	cmpl $8,%r12d
	jnz L228
L230:
	movq %rbx,%rdi
	call _do_include
	jmp L228
L268:
	movq %rbx,%rdi
	call _do_line
	jmp L228
L251:
	movq _state_stack(%rip),%rsi
	cmpq $0,%rsi
	jz L252
L255:
	movzbl (%rsi),%esi
	cmpl $0,%esi
	jz L228
L252:
	movq %rbx,%rdi
	call _macro_undef
	jmp L228
L233:
	movl %r12d,%edi
	movq %rbx,%rsi
	call _do_ifdef
L228:
	movq _state_stack(%rip),%rsi
	cmpq $0,%rsi
	jz L283
L287:
	movzbl (%rsi),%esi
	cmpl $0,%esi
	jz L224
L283:
	cmpq $0,(%rbx)
	jz L224
L280:
	pushq $L290
	call _error
	addq $8,%rsp
L224:
	movq _state_stack(%rip),%rsi
	cmpq $0,%rsi
	jz L210
L294:
	movzbl (%rsi),%esi
	cmpl $0,%esi
	jnz L210
L291:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _list_cut
L210:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L326:
_directive_check:
L327:
	pushq %rbp
	movq %rsp,%rbp
L335:
	cmpq $0,_state_stack(%rip)
	jz L329
L330:
	pushq $L333
	call _error
	addq $8,%rsp
L329:
	popq %rbp
	ret
L337:
L148:
	.byte 35,101,114,114,111,114,32,100
	.byte 105,114,101,99,116,105,118,101
	.byte 58,32,37,115,0
L290:
	.byte 116,114,97,105,108,105,110,103
	.byte 32,103,97,114,98,97,103,101
	.byte 32,97,102,116,101,114,32,100
	.byte 105,114,101,99,116,105,118,101
	.byte 0
L266:
	.byte 117,110,107,110,111,119,110,32
	.byte 100,105,114,101,99,116,105,118
	.byte 101,0
L200:
	.byte 101,120,112,101,99,116,101,100
	.byte 32,102,105,108,101,32,110,97
	.byte 109,101,32,97,102,116,101,114
	.byte 32,35,105,110,99,108,117,100
	.byte 101,0
L113:
	.byte 100,117,112,108,105,99,97,116
	.byte 101,32,35,101,108,115,101,0
L94:
	.byte 35,101,108,105,102,32,97,102
	.byte 116,101,114,32,35,101,108,115
	.byte 101,0
L333:
	.byte 101,110,100,45,111,102,45,102
	.byte 105,108,101,32,105,110,32,35
	.byte 105,102,47,105,102,100,101,102
	.byte 47,105,102,110,100,101,102,0
L125:
	.byte 35,101,110,100,105,102,32,119
	.byte 105,116,104,111,117,116,32,35
	.byte 105,102,0
L109:
	.byte 35,101,108,115,101,32,119,105
	.byte 116,104,111,117,116,32,35,105
	.byte 102,0
L90:
	.byte 35,101,108,105,102,32,119,105
	.byte 116,104,111,117,116,32,35,105
	.byte 102,0
.globl _macro_lookup
.globl _list_pop
.globl _strcmp
.globl _token_convert_number
.globl _vstring_clear
.globl _error
.globl _list_skip_spaces
.globl _need_sync
.globl _list_cut
.globl _vstring_concat
.globl _directive
.globl _evaluate
.globl _macro_define
.globl _list_stringize
.globl _token_dequote
.globl _macro_replace_all
.globl _list_strip_all
.globl _list_strip_ends
.globl _safe_malloc
.globl _token_text
.globl _token_free
.globl _vstring_free
.globl _free
.globl _macro_undef
.globl _list_match
.local _state_stack
.comm _state_stack, 8, 8
.globl _directive_check
.globl _input_stack
.globl _input_open
