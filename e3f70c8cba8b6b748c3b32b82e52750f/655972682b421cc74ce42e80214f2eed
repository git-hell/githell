.data
.align 8
_regs_free_list:
	.quad 0
.text
_regs_alloc:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	movl $24000,%edi
	call _safe_malloc
	movl $1,%esi
L8:
	movq _regs_free_list(%rip),%rdi
	movslq %esi,%rcx
	imulq $24,%rcx
	leaq (%rax,%rcx),%rdx
	movq %rdi,8(%rax,%rcx)
	movq %rdx,_regs_free_list(%rip)
	addl $1,%esi
	cmpl $1000,%esi
	jl L8
L3:
	popq %rbp
	ret
L15:
_regs_lookup:
L16:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L55:
	movq %rdi,%rbx
	movl %esi,%r13d
	movq 8(%rbx),%r12
L19:
	cmpq $0,%r12
	jz L22
L20:
	movl (%r12),%esi
	cmpl %r13d,%esi
	jnz L25
L23:
	movq %r12,%rax
	jmp L18
L25:
	cmpl %r13d,%esi
	ja L22
L21:
	movq 8(%r12),%r12
	jmp L19
L22:
	testl $1,%edx
	jz L32
L34:
	movq _regs_free_list(%rip),%rsi
	movq %rsi,%rax
	cmpq $0,%rsi
	jz L38
L40:
	movq 8(%rsi),%rsi
	movq %rsi,_regs_free_list(%rip)
	jmp L35
L38:
	call _regs_alloc
L35:
	movl %r13d,(%rax)
	cmpq $0,%r12
	jz L49
L46:
	movq 16(%r12),%rsi
	movq %rsi,16(%rax)
	leaq 8(%rax),%rsi
	movq %r12,8(%rax)
	movq 16(%r12),%rdi
	movq %rax,(%rdi)
	movq %rsi,16(%r12)
	jmp L45
L49:
	leaq 8(%rax),%rsi
	movq $0,8(%rax)
	movq 16(%rbx),%rdi
	movq %rdi,16(%rax)
	movq 16(%rbx),%rdi
	movq %rax,(%rdi)
	movq %rsi,16(%rbx)
L45:
	addl $1,(%rbx)
	jmp L18
L32:
	xorl %eax,%eax
L18:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L57:
_regs_remove:
L58:
	pushq %rbp
	movq %rsp,%rbp
L59:
	movq 8(%rdi),%rax
L61:
	cmpq $0,%rax
	jz L60
L62:
	movl (%rax),%ecx
	cmpl %esi,%ecx
	jnz L63
L68:
	addl $-1,(%rdi)
	movq 8(%rax),%rsi
	cmpq $0,%rsi
	jz L75
L74:
	movq 16(%rax),%rdi
	movq %rdi,16(%rsi)
	jmp L76
L75:
	movq 16(%rax),%rsi
	movq %rsi,16(%rdi)
L76:
	movq 8(%rax),%rsi
	movq 16(%rax),%rdi
	movq %rsi,(%rdi)
	movq _regs_free_list(%rip),%rsi
	movq %rsi,8(%rax)
	movq %rax,_regs_free_list(%rip)
	jmp L60
L63:
	movq 8(%rax),%rax
	jmp L61
L60:
	popq %rbp
	ret
L84:
_regs_overlap:
L85:
	pushq %rbp
	movq %rsp,%rbp
L86:
	movq 8(%rdi),%rdi
	movq 8(%rsi),%rsi
L88:
	cmpq $0,%rdi
	jz L90
L91:
	cmpq $0,%rsi
	jz L90
L89:
	movl (%rdi),%eax
	movl (%rsi),%ecx
	cmpl %ecx,%eax
	jnz L97
L95:
	movl $1,%eax
	jmp L87
L97:
	cmpl %ecx,%eax
	jae L100
L99:
	movq 8(%rdi),%rdi
	jmp L88
L100:
	movq 8(%rsi),%rsi
	jmp L88
L90:
	xorl %eax,%eax
L87:
	popq %rbp
	ret
L106:
_regs_diff:
L107:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L115:
	movq %rdi,%r12
	movq 8(%rsi),%rbx
L110:
	cmpq $0,%rbx
	jz L109
L111:
	movl (%rbx),%esi
	movq %r12,%rdi
	call _regs_remove
	movq 8(%rbx),%rbx
	jmp L110
L109:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L117:
_regs_union:
L118:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L160:
	movq %rdi,%rbx
	movq 8(%rbx),%r12
	movq 8(%rsi),%r13
L121:
	cmpq $0,%r13
	jz L120
L159:
	movl (%r13),%edi
L125:
	cmpq $0,%r12
	jz L127
L128:
	movl (%r12),%esi
	cmpl %edi,%esi
	jae L127
L126:
	movq 8(%r12),%r12
	jmp L125
L127:
	cmpq $0,%r12
	jz L140
L135:
	movl (%r12),%esi
	movl (%r13),%edi
	cmpl %edi,%esi
	jz L123
L140:
	movq _regs_free_list(%rip),%rsi
	movq %rsi,%rax
	cmpq $0,%rsi
	jz L144
L146:
	movq 8(%rsi),%rsi
	movq %rsi,_regs_free_list(%rip)
	jmp L141
L144:
	call _regs_alloc
L141:
	movl (%r13),%esi
	movl %esi,(%rax)
	cmpq $0,%r12
	jz L155
L152:
	movq 16(%r12),%rsi
	movq %rsi,16(%rax)
	leaq 8(%rax),%rsi
	movq %r12,8(%rax)
	movq 16(%r12),%rdi
	movq %rax,(%rdi)
	movq %rsi,16(%r12)
	jmp L151
L155:
	leaq 8(%rax),%rsi
	movq $0,8(%rax)
	movq 16(%rbx),%rdi
	movq %rdi,16(%rax)
	movq 16(%rbx),%rdi
	movq %rax,(%rdi)
	movq %rsi,16(%rbx)
L151:
	addl $1,(%rbx)
L123:
	movq 8(%r13),%r13
	jmp L121
L120:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L162:
_regs_move:
L163:
	pushq %rbp
	movq %rsp,%rbp
L166:
	leaq 8(%rsi),%rax
	movq 8(%rsi),%rcx
	cmpq $0,%rcx
	jz L167
L169:
	movq 16(%rdi),%rdx
	movq %rcx,(%rdx)
	movq 16(%rdi),%rcx
	movq 8(%rsi),%rdx
	movq %rcx,16(%rdx)
	movq 16(%rsi),%rcx
	movq %rcx,16(%rdi)
	movq $0,8(%rsi)
	movq %rax,16(%rsi)
L167:
	movl (%rsi),%eax
	movl %eax,(%rdi)
	movl $0,(%rsi)
L165:
	popq %rbp
	ret
L178:
_regs_equal:
L179:
	pushq %rbp
	movq %rsp,%rbp
L180:
	movl (%rdi),%eax
	movl (%rsi),%ecx
	cmpl %ecx,%eax
	jz L184
L182:
	xorl %eax,%eax
	jmp L181
L184:
	movq 8(%rdi),%rdi
	movq 8(%rsi),%rsi
L186:
	cmpq $0,%rdi
	jz L188
L187:
	movl (%rdi),%eax
	movl (%rsi),%ecx
	cmpl %ecx,%eax
	jz L191
L189:
	xorl %eax,%eax
	jmp L181
L191:
	movq 8(%rdi),%rdi
	movq 8(%rsi),%rsi
	jmp L186
L188:
	movl $1,%eax
L181:
	popq %rbp
	ret
L197:
_regs_clear:
L198:
	pushq %rbp
	movq %rsp,%rbp
L201:
	movq 8(%rdi),%rsi
	cmpq $0,%rsi
	jz L200
L204:
	addl $-1,(%rdi)
	movq 8(%rsi),%rax
	cmpq $0,%rax
	jz L211
L210:
	movq 16(%rsi),%rcx
	movq %rcx,16(%rax)
	jmp L212
L211:
	movq 16(%rsi),%rax
	movq %rax,16(%rdi)
L212:
	movq 8(%rsi),%rax
	movq 16(%rsi),%rcx
	movq %rax,(%rcx)
	movq _regs_free_list(%rip),%rax
	movq %rax,8(%rsi)
	movq %rsi,_regs_free_list(%rip)
	jmp L201
L200:
	popq %rbp
	ret
L219:
_regs_eliminate_base:
L220:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L231:
	movq %rdi,%rbx
	movl %esi,%r13d
	movq 8(%rbx),%rsi
	andl $2149580799,%r13d
L223:
	cmpq $0,%rsi
	jz L222
L224:
	movq 8(%rsi),%r12
	movl (%rsi),%esi
	movl %esi,%edi
	andl $2149580799,%edi
	cmpl %r13d,%edi
	jnz L228
L226:
	movq %rbx,%rdi
	call _regs_remove
L228:
	movq %r12,%rsi
	jmp L223
L222:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L233:
_regs_eliminate_bases:
L234:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L242:
	movq %rdi,%r12
	movq 8(%rsi),%rbx
L237:
	cmpq $0,%rbx
	jz L236
L238:
	movl (%rbx),%esi
	movq %r12,%rdi
	call _regs_eliminate_base
	movq 8(%rbx),%rbx
	jmp L237
L236:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L244:
_regs_replace_base:
L245:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L249:
	movq %rdi,%r12
	movl %esi,%ebx
	movq %r12,%rdi
	movl %ebx,%esi
	call _regs_eliminate_base
	movq %r12,%rdi
	movl %ebx,%esi
	movl $1,%edx
	call _regs_lookup
L247:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L251:
_regs_select_base:
L252:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L264:
	movq %rdi,%r13
	movl %edx,%r12d
	movq 8(%rsi),%rbx
	andl $2149580799,%r12d
L255:
	cmpq $0,%rbx
	jz L254
L256:
	movl (%rbx),%esi
	movl %esi,%edi
	andl $2149580799,%edi
	cmpl %r12d,%edi
	jnz L257
L259:
	movq %r13,%rdi
	movl $1,%edx
	call _regs_lookup
L257:
	movq 8(%rbx),%rbx
	jmp L255
L254:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L266:
_regs_select_bases:
L267:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L275:
	movq %rsi,%r12
	movq %rdi,%r13
	movq 8(%rdx),%rbx
L270:
	cmpq $0,%rbx
	jz L269
L271:
	movl (%rbx),%edx
	movq %r13,%rdi
	movq %r12,%rsi
	call _regs_select_base
	movq 8(%rbx),%rbx
	jmp L270
L269:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L277:
_regs_intersect_bases:
L278:
	pushq %rbp
	movq %rsp,%rbp
	subq $24,%rsp
	pushq %rbx
	pushq %r12
L282:
	movq %rsi,%rdx
	movq %rdi,%r12
	leaq -24(%rbp),%rbx
	xorps %xmm0,%xmm0
	movups %xmm0,-24(%rbp)
	movq $0,-8(%rbp)
	leaq -16(%rbp),%rsi
	movq %rsi,-8(%rbp)
	movq %rbx,%rdi
	movq %r12,%rsi
	call _regs_select_bases
	movq %r12,%rdi
	call _regs_clear
	movq %r12,%rdi
	movq %rbx,%rsi
	call _regs_move
L280:
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L284:
_regs_output:
L285:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L296:
	movq %rdi,%rbx
	pushq $L288
	call _output
	addq $8,%rsp
	movq 8(%rbx),%rbx
L289:
	cmpq $0,%rbx
	jz L292
L290:
	movl (%rbx),%esi
	pushq %rsi
	pushq $L293
	call _output
	addq $16,%rsp
	movq 8(%rbx),%rbx
	jmp L289
L292:
	pushq $L294
	call _output
	addq $8,%rsp
L287:
	popq %rbx
	popq %rbp
	ret
L298:
L293:
	.byte 37,114,32,0
L288:
	.byte 123,32,0
L294:
	.byte 125,0
.globl _regs_overlap
.globl _regs_lookup
.globl _regs_clear
.globl _regs_intersect_bases
.globl _regs_select_bases
.globl _regs_eliminate_bases
.globl _regs_output
.globl _regs_free_list
.globl _output
.globl _regs_move
.globl _regs_remove
.globl _regs_diff
.globl _regs_alloc
.globl _safe_malloc
.globl _regs_select_base
.globl _regs_replace_base
.globl _regs_eliminate_base
.globl _regs_equal
.globl _regs_union
