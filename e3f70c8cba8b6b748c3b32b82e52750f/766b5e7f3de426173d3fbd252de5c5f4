.data
.align 8
_input_stack:
	.quad 0
.text
_input_push:
L2:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L9:
	movq %rsi,%r12
	movq %rdi,%r13
	movl $48,%edi
	call _safe_malloc
	movq %rax,%r14
	leaq 8(%r14),%rbx
	movq %rbx,%rdi
	call _vstring_init
	movq %rbx,%rdi
	movq %r12,%rsi
	call _vstring_puts
	movq %r13,(%r14)
	movl $0,32(%r14)
	movq _input_stack(%rip),%rsi
	movq %rsi,40(%r14)
	movq %r14,_input_stack(%rip)
	movb $1,_need_sync(%rip)
L4:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L11:
_input_pop:
L13:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L14:
	movq _input_stack(%rip),%rbx
	cmpq $0,40(%rbx)
	jnz L19
L16:
	call _directive_check
L19:
	movq _input_stack(%rip),%rsi
	movq 40(%rsi),%rsi
	movq %rsi,_input_stack(%rip)
	movq (%rbx),%rdi
	call _fclose
	leaq 8(%rbx),%rdi
	call _vstring_free
	movq %rbx,%rdi
	call _free
	movb $1,_need_sync(%rip)
L15:
	popq %rbx
	popq %rbp
	ret
L25:
_erase_comments:
L28:
	pushq %rbp
	movq %rsp,%rbp
L29:
	xorl %esi,%esi
	xorl %eax,%eax
L31:
	movzbl (%rdi),%edx
	cmpl $0,%edx
	jz L33
L32:
	cmpl $0,%esi
	jz L35
L34:
	cmpl %esi,%edx
	jnz L39
L37:
	xorl %esi,%esi
L39:
	movzbl (%rdi),%ecx
	cmpl $92,%ecx
	jnz L36
L43:
	leaq 1(%rdi),%rcx
	movzbl 1(%rdi),%edx
	cmpl %esi,%edx
	jnz L36
L40:
	movq %rcx,%rdi
	jmp L36
L35:
	movzbl _in_comment(%rip),%ecx
	cmpl $0,%ecx
	jz L48
L47:
	cmpl $42,%edx
	jnz L52
L53:
	movzbl 1(%rdi),%ecx
	cmpl $47,%ecx
	jnz L52
L50:
	movb $32,1(%rdi)
	movb $0,_in_comment(%rip)
L52:
	movb $32,(%rdi)
	jmp L36
L48:
	cmpl $47,%edx
	jnz L58
L60:
	movzbl 1(%rdi),%ecx
	cmpl $42,%ecx
	jnz L58
L57:
	movb $32,(%rdi)
	movb $32,1(%rdi)
	movb $1,_in_comment(%rip)
	jmp L36
L58:
	movzbl (%rdi),%ecx
	cmpl $34,%ecx
	jz L64
L67:
	cmpl $39,%ecx
	jnz L36
L64:
	movzbl (%rdi),%esi
L36:
	movzbl (%rdi),%ecx
	cmpl $32,%ecx
	jnz L72
L71:
	cmpq $0,%rax
	jnz L73
L74:
	movq %rdi,%rax
	jmp L73
L72:
	xorl %eax,%eax
L73:
	addq $1,%rdi
	jmp L31
L33:
	cmpq $0,%rax
	jz L30
L77:
	movb $0,(%rax)
L30:
	popq %rbp
	ret
L83:
_unwind:
L85:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L108:
	movl %edi,%ebx
	movl $-1,%r12d
L88:
	movq _input_stack(%rip),%rsi
	cmpq $0,%rsi
	jz L90
L89:
	movq (%rsi),%rsi
	movl (%rsi),%edi
	addl $-1,%edi
	movl %edi,(%rsi)
	cmpl $0,%edi
	jl L92
L91:
	movq _input_stack(%rip),%rsi
	movq (%rsi),%rsi
	movq 24(%rsi),%rdi
	movq %rdi,%rax
	addq $1,%rdi
	movq %rdi,24(%rsi)
	movzbl (%rax),%eax
	jmp L93
L92:
	movq _input_stack(%rip),%rsi
	movq (%rsi),%rdi
	call ___fillbuf
L93:
	movl %eax,%r12d
	cmpl $-1,%eax
	jnz L95
L94:
	movzbl _in_comment(%rip),%esi
	cmpl $0,%esi
	jz L99
L97:
	pushq $L100
	call _error
	addq $8,%rsp
L99:
	cmpl $0,%ebx
	jz L90
L103:
	call _input_pop
	jmp L88
L95:
	movq _input_stack(%rip),%rsi
	movq (%rsi),%rsi
	movl %eax,%edi
	call _ungetc
L90:
	movl %r12d,%eax
L87:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L110:
.data
.align 8
L115:
	.byte 1
	.space 23, 0
.text
_concat_line:
L112:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L113:
	xorl %r12d,%r12d
	call _unwind
	cmpl $-1,%eax
	jnz L118
L116:
	xorl %eax,%eax
	jmp L114
L118:
	movq _input_stack(%rip),%rsi
	addl $1,32(%rsi)
	movq $L115,%rdi
	call _vstring_clear
L121:
	cmpl $92,%r12d
	setz %sil
	movzbl %sil,%ebx
	movq _input_stack(%rip),%rsi
	movq (%rsi),%rsi
	movl (%rsi),%edi
	addl $-1,%edi
	movl %edi,(%rsi)
	cmpl $0,%edi
	jl L125
L124:
	movq _input_stack(%rip),%rsi
	movq (%rsi),%rsi
	movq 24(%rsi),%rdi
	movq %rdi,%rax
	addq $1,%rdi
	movq %rdi,24(%rsi)
	movzbl (%rax),%eax
	jmp L126
L125:
	movq _input_stack(%rip),%rsi
	movq (%rsi),%rdi
	call ___fillbuf
L126:
	movl %eax,%r12d
	cmpl $10,%r12d
	jz L131
L143:
	cmpl $-1,%r12d
	jz L136
L128:
	movq $L115,%rdi
	movl %eax,%esi
	call _vstring_putc
	jmp L121
L131:
	cmpl $0,%ebx
	jz L136
L132:
	movq $L115,%rdi
	call _vstring_rubout
	movq _input_stack(%rip),%rsi
	addl $1,32(%rsi)
	jmp L121
L136:
	movl L115(%rip),%esi
	testl $1,%esi
	jz L138
L137:
	movq $L115+1,%rax
	jmp L114
L138:
	movq L115+16(%rip),%rax
L114:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L147:
_input_tokenize:
L148:
	pushq %rbp
	movq %rsp,%rbp
	subq $8,%rsp
	pushq %rbx
	pushq %r12
L159:
	movq %rdi,%r12
	movq %rsi,-8(%rbp)
	xorl %ebx,%ebx
L151:
	leaq -8(%rbp),%rsi
	movq -8(%rbp),%rdi
	movzbl (%rdi),%eax
	cmpl $0,%eax
	jz L153
L152:
	call _token_scan
	leaq 32(%rax),%rsi
	movq $0,32(%rax)
	movq 8(%r12),%rdi
	movq %rdi,40(%rax)
	movq 8(%r12),%rdi
	movq %rax,(%rdi)
	movq %rsi,8(%r12)
	addl $1,%ebx
	jmp L151
L153:
	movl %ebx,%eax
L150:
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L161:
_input_tokens:
L162:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L171:
	movq %rsi,%rbx
	call _concat_line
	movq %rax,%r12
	cmpq $0,%r12
	jnz L167
L165:
	movl $-1,%eax
	jmp L164
L167:
	movq %r12,%rdi
	call _erase_comments
	movq %rbx,%rdi
	movq %r12,%rsi
	call _input_tokenize
L164:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L173:
.data
.align 8
_system_dirs:
	.quad 0
.text
_input_dir:
L174:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L181:
	movq %rdi,%r12
	movl $32,%edi
	call _safe_malloc
	movq %rax,%rbx
	movq %rbx,%rdi
	call _vstring_init
	movq %rbx,%rdi
	movq %r12,%rsi
	call _vstring_puts
	movq _system_dirs(%rip),%rsi
	movq %rsi,24(%rbx)
	movq %rbx,_system_dirs(%rip)
L176:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L183:
_input_open:
L184:
	pushq %rbp
	movq %rsp,%rbp
	subq $24,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L239:
	movq %rdi,%r14
	movl %esi,%r12d
	leaq -24(%rbp),%rdi
	call _vstring_init
	movq _system_dirs(%rip),%r13
L188:
	leaq -24(%rbp),%rbx
	movq %rbx,%rdi
	call _vstring_clear
	cmpl $0,%r12d
	jz L195
L236:
	cmpl $1,%r12d
	jz L210
L237:
	cmpl $2,%r12d
	jnz L193
L198:
	movq _input_stack(%rip),%rsi
	addq $8,%rsi
	movq %rbx,%rdi
	call _vstring_concat
L199:
	movl -24(%rbp),%esi
	testl $1,%esi
	jz L207
L206:
	shll $24,%esi
	sarl $25,%esi
	movslq %esi,%rsi
	jmp L208
L207:
	movq -16(%rbp),%rsi
L208:
	cmpq $0,%rsi
	jz L201
L202:
	leaq -24(%rbp),%rbx
	movq %rbx,%rdi
	call _vstring_last
	movzbl %al,%esi
	cmpl $47,%esi
	jz L201
L200:
	movq %rbx,%rdi
	call _vstring_rubout
	jmp L199
L201:
	leaq -24(%rbp),%rdi
	movq %r14,%rsi
	call _vstring_puts
	movl $1,%r12d
	jmp L193
L210:
	cmpq $0,%r13
	jnz L213
L211:
	pushq %r14
	pushq $L214
	call _error
	addq $16,%rsp
L213:
	leaq -24(%rbp),%rbx
	movq %rbx,%rdi
	movq %r13,%rsi
	call _vstring_concat
	movq %rbx,%rdi
	movl $47,%esi
	call _vstring_putc
	movq %rbx,%rdi
	movq %r14,%rsi
	call _vstring_puts
	movq 24(%r13),%r13
L193:
	movl -24(%rbp),%esi
	testl $1,%esi
	jz L220
L219:
	leaq -23(%rbp),%rdi
	jmp L221
L220:
	movq -8(%rbp),%rdi
L221:
	xorl %esi,%esi
	call _access
	cmpl $0,%eax
	jnz L188
	jz L196
L195:
	movq %rbx,%rdi
	movq %r14,%rsi
	call _vstring_puts
L196:
	movl -24(%rbp),%esi
	testl $1,%esi
	jz L225
L224:
	leaq -23(%rbp),%rdi
	jmp L226
L225:
	movq -8(%rbp),%rdi
L226:
	movq $L223,%rsi
	call _fopen
	movq %rax,%rbx
	cmpq $0,%rbx
	jnz L229
L227:
	pushq %r14
	pushq $L230
	call _error
	addq $16,%rsp
L229:
	movl -24(%rbp),%esi
	testl $1,%esi
	jz L232
L231:
	leaq -23(%rbp),%rsi
	jmp L233
L232:
	movq -8(%rbp),%rsi
L233:
	movq %rbx,%rdi
	call _input_push
	leaq -24(%rbp),%rdi
	call _vstring_free
L186:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L241:
L230:
	.byte 99,97,110,39,116,32,111,112
	.byte 101,110,32,39,37,115,39,32
	.byte 102,111,114,32,114,101,97,100
	.byte 105,110,103,0
L223:
	.byte 114,0
L100:
	.byte 101,110,100,45,111,102,45,102
	.byte 105,108,101,32,105,110,32,99
	.byte 111,109,109,101,110,116,0
L214:
	.byte 99,97,110,39,116,32,102,105
	.byte 110,100,32,39,37,115,39,0
.globl _input_dir
.globl _vstring_clear
.globl _error
.globl _vstring_putc
.globl _need_sync
.globl _access
.globl _ungetc
.globl _vstring_last
.globl _vstring_concat
.globl _vstring_init
.globl _vstring_rubout
.globl _input_tokenize
.globl _system_dirs
.globl _input_tokens
.globl _vstring_puts
.globl _safe_malloc
.local _in_comment
.comm _in_comment, 1, 1
.globl _vstring_free
.globl _free
.globl _fclose
.globl ___fillbuf
.globl _directive_check
.globl _input_stack
.globl _input_open
.globl _token_scan
.globl _fopen
