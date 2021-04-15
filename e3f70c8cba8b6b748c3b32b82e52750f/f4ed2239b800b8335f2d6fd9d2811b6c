.data
.align 8
_blks_free_list:
	.quad 0
.text
_blks_alloc:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	movl $24000,%edi
	call _safe_malloc
	movl $1,%esi
L8:
	movq _blks_free_list(%rip),%rdi
	movslq %esi,%rcx
	imulq $24,%rcx
	leaq (%rax,%rcx),%rdx
	movq %rdi,8(%rax,%rcx)
	movq %rdx,_blks_free_list(%rip)
	addl $1,%esi
	cmpl $1000,%esi
	jl L8
L3:
	popq %rbp
	ret
L15:
_blks_lookup:
L16:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L55:
	movq %rdi,%rbx
	movq %rsi,%r13
	movq 8(%rbx),%r12
L19:
	cmpq $0,%r12
	jz L22
L20:
	movq (%r12),%rsi
	cmpq %rsi,%r13
	jnz L25
L23:
	movq %r12,%rax
	jmp L18
L25:
	cmpq %r13,%rsi
	ja L22
L21:
	movq 8(%r12),%r12
	jmp L19
L22:
	testl $1,%edx
	jz L32
L34:
	movq _blks_free_list(%rip),%rsi
	movq %rsi,%rax
	cmpq $0,%rsi
	jz L38
L40:
	movq 8(%rsi),%rsi
	movq %rsi,_blks_free_list(%rip)
	jmp L35
L38:
	call _blks_alloc
L35:
	movq %r13,(%rax)
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
_blks_remove:
L58:
	pushq %rbp
	movq %rsp,%rbp
L59:
	movq 8(%rdi),%rax
L61:
	cmpq $0,%rax
	jz L60
L62:
	cmpq (%rax),%rsi
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
	movq _blks_free_list(%rip),%rsi
	movq %rsi,8(%rax)
	movq %rax,_blks_free_list(%rip)
	jmp L60
L63:
	movq 8(%rax),%rax
	jmp L61
L60:
	popq %rbp
	ret
L84:
_blks_diff:
L85:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L93:
	movq %rdi,%r12
	movq 8(%rsi),%rbx
L88:
	cmpq $0,%rbx
	jz L87
L89:
	movq (%rbx),%rsi
	movq %r12,%rdi
	call _blks_remove
	movq 8(%rbx),%rbx
	jmp L88
L87:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L95:
_blks_union:
L96:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L138:
	movq %rdi,%rbx
	movq 8(%rbx),%r12
	movq 8(%rsi),%r13
L99:
	cmpq $0,%r13
	jz L98
L137:
	movq (%r13),%rsi
L103:
	cmpq $0,%r12
	jz L105
L106:
	cmpq %rsi,(%r12)
	jae L105
L104:
	movq 8(%r12),%r12
	jmp L103
L105:
	cmpq $0,%r12
	jz L118
L113:
	movq (%r13),%rsi
	cmpq (%r12),%rsi
	jz L101
L118:
	movq _blks_free_list(%rip),%rsi
	movq %rsi,%rax
	cmpq $0,%rsi
	jz L122
L124:
	movq 8(%rsi),%rsi
	movq %rsi,_blks_free_list(%rip)
	jmp L119
L122:
	call _blks_alloc
L119:
	movq (%r13),%rsi
	movq %rsi,(%rax)
	cmpq $0,%r12
	jz L133
L130:
	movq 16(%r12),%rsi
	movq %rsi,16(%rax)
	leaq 8(%rax),%rsi
	movq %r12,8(%rax)
	movq 16(%r12),%rdi
	movq %rax,(%rdi)
	movq %rsi,16(%r12)
	jmp L129
L133:
	leaq 8(%rax),%rsi
	movq $0,8(%rax)
	movq 16(%rbx),%rdi
	movq %rdi,16(%rax)
	movq 16(%rbx),%rdi
	movq %rax,(%rdi)
	movq %rsi,16(%rbx)
L129:
	addl $1,(%rbx)
L101:
	movq 8(%r13),%r13
	jmp L99
L98:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L140:
_blks_intersect:
L141:
	pushq %rbp
	movq %rsp,%rbp
L142:
	movq 8(%rdi),%rax
	movq 8(%rsi),%rsi
L144:
	cmpq $0,%rax
	jz L172
L147:
	cmpq $0,%rsi
	jz L172
L145:
	movq (%rsi),%rcx
	movq (%rax),%rdx
	cmpq %rdx,%rcx
	jnz L152
L151:
	movq 8(%rax),%rax
	movq 8(%rsi),%rsi
	jmp L144
L152:
	cmpq %rcx,%rdx
	jae L155
L154:
	movq 8(%rax),%rdx
	addl $-1,(%rdi)
	movq 8(%rax),%rcx
	cmpq $0,%rcx
	jz L164
L163:
	movq 16(%rax),%r8
	movq %r8,16(%rcx)
	jmp L165
L164:
	movq 16(%rax),%rcx
	movq %rcx,16(%rdi)
L165:
	movq 8(%rax),%rcx
	movq 16(%rax),%r8
	movq %rcx,(%r8)
	movq _blks_free_list(%rip),%rcx
	movq %rcx,8(%rax)
	movq %rax,_blks_free_list(%rip)
	movq %rdx,%rax
	jmp L144
L155:
	cmpq %rcx,%rdx
	jbe L144
L169:
	movq 8(%rsi),%rsi
	jmp L144
L172:
	cmpq $0,%rax
	jz L143
L173:
	movq 8(%rax),%rcx
	addl $-1,(%rdi)
	movq 8(%rax),%rsi
	cmpq $0,%rsi
	jz L182
L181:
	movq 16(%rax),%rdx
	movq %rdx,16(%rsi)
	jmp L183
L182:
	movq 16(%rax),%rsi
	movq %rsi,16(%rdi)
L183:
	movq 8(%rax),%rsi
	movq 16(%rax),%rdx
	movq %rsi,(%rdx)
	movq _blks_free_list(%rip),%rsi
	movq %rsi,8(%rax)
	movq %rax,_blks_free_list(%rip)
	movq %rcx,%rax
	jmp L172
L143:
	popq %rbp
	ret
L190:
_blks_move:
L191:
	pushq %rbp
	movq %rsp,%rbp
L194:
	leaq 8(%rsi),%rax
	movq 8(%rsi),%rcx
	cmpq $0,%rcx
	jz L195
L197:
	movq 16(%rdi),%rdx
	movq %rcx,(%rdx)
	movq 16(%rdi),%rcx
	movq 8(%rsi),%rdx
	movq %rcx,16(%rdx)
	movq 16(%rsi),%rcx
	movq %rcx,16(%rdi)
	movq $0,8(%rsi)
	movq %rax,16(%rsi)
L195:
	movl (%rsi),%eax
	movl %eax,(%rdi)
	movl $0,(%rsi)
L193:
	popq %rbp
	ret
L206:
_blks_clear:
L207:
	pushq %rbp
	movq %rsp,%rbp
L210:
	movq 8(%rdi),%rsi
	cmpq $0,%rsi
	jz L209
L213:
	addl $-1,(%rdi)
	movq 8(%rsi),%rax
	cmpq $0,%rax
	jz L220
L219:
	movq 16(%rsi),%rcx
	movq %rcx,16(%rax)
	jmp L221
L220:
	movq 16(%rsi),%rax
	movq %rax,16(%rdi)
L221:
	movq 8(%rsi),%rax
	movq 16(%rsi),%rcx
	movq %rax,(%rcx)
	movq _blks_free_list(%rip),%rax
	movq %rax,8(%rsi)
	movq %rsi,_blks_free_list(%rip)
	jmp L210
L209:
	popq %rbp
	ret
L228:
_all0:
L231:
	pushq %rbp
	movq %rsp,%rbp
L236:
	movq %rdi,%rsi
	movq _all_blks(%rip),%rdi
	movl $1,%edx
	call _blks_lookup
	xorl %eax,%eax
L233:
	popq %rbp
	ret
L238:
_blks_all:
L239:
	pushq %rbp
	movq %rsp,%rbp
L240:
	movq %rdi,_all_blks(%rip)
	movq $_all0,%rdi
	call _blocks_iter
L241:
	popq %rbp
	ret
L245:
_blks_output:
L246:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L257:
	movq %rdi,%rbx
	pushq $L249
	call _output
	addq $8,%rsp
	movq 8(%rbx),%rbx
L250:
	cmpq $0,%rbx
	jz L253
L251:
	movq (%rbx),%rsi
	movl (%rsi),%esi
	pushq %rsi
	pushq $L254
	call _output
	addq $16,%rsp
	movq 8(%rbx),%rbx
	jmp L250
L253:
	pushq $L255
	call _output
	addq $8,%rsp
L248:
	popq %rbx
	popq %rbp
	ret
L259:
L249:
	.byte 123,0
L254:
	.byte 32,37,76,0
L255:
	.byte 32,125,0
.globl _blks_lookup
.globl _blocks_iter
.globl _blks_clear
.local _all_blks
.comm _all_blks, 8, 8
.globl _blks_output
.globl _blks_intersect
.globl _blks_free_list
.globl _output
.globl _blks_move
.globl _blks_remove
.globl _blks_diff
.globl _blks_all
.globl _blks_alloc
.globl _safe_malloc
.globl _blks_union
