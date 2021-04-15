.text
.align 8
L242:
	.quad 0xbff0000000000000
_fold0:
L3:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L240:
	movq %rdi,-16(%rbp)	 # spill
	xorl %r14d,%r14d
	movq -16(%rbp),%r10	 # spill
	movq 8(%r10),%rbx
L6:
	cmpq $0,%rbx
	jz L9
L7:
	movq %rbx,%rdi
	call _insn_normalize
	xorl %r12d,%r12d
	movq 48(%rbx),%rsi
	cmpq $0,%rsi
	jz L12
L17:
	movl (%rsi),%edi
	cmpl $1,%edi
	jnz L12
L13:
	cmpq $0,32(%rsi)
	jnz L12
L10:
	movl (%rbx),%edi
	cmpl $1610809356,%edi
	jz L39
L224:
	cmpl $1610809357,%edi
	jz L26
L225:
	cmpl $1610809358,%edi
	jnz L12
L35:
	movq 24(%rsi),%rdi
	notq %rdi
	leaq -8(%rbp),%rsi
	movq %rdi,-8(%rbp)
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	call _con_normalize
	jmp L32
L26:
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	testq $7168,%rdi
	jz L30
L29:
	movsd 24(%rsi),%xmm0
	mulsd L242(%rip),%xmm0
	movsd %xmm0,-8(%rbp)
	jmp L31
L30:
	movq 24(%rsi),%rsi
	negq %rsi
	movq %rsi,-8(%rbp)
L31:
	leaq -8(%rbp),%rsi
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	call _con_normalize
	jmp L32
L39:
	leaq 24(%rsi),%rax
	movq 8(%rsi),%rcx
	leaq -8(%rbp),%rsi
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	movq %rcx,%rdx
	movq %rax,%rcx
	call _con_cast
	jmp L32
L12:
	movl (%rbx),%esi
	cmpl $1879244817,%esi
	jnz L43
L52:
	movq 48(%rbx),%rsi
	cmpq $0,%rsi
	jz L43
L56:
	movl (%rsi),%edi
	cmpl $1,%edi
	jnz L43
L48:
	movq 56(%rbx),%rdi
	cmpq $0,%rdi
	jz L43
L60:
	movl (%rdi),%eax
	cmpl $1,%eax
	jnz L43
L44:
	movq 32(%rdi),%rax
	cmpq 32(%rsi),%rax
	jnz L43
L41:
	movq $0,32(%rdi)
	movq 48(%rbx),%rsi
	movq $0,32(%rsi)
L43:
	movq 56(%rbx),%rsi
	cmpq $0,%rsi
	jz L66
L71:
	movl (%rsi),%edi
	cmpl $1,%edi
	jnz L66
L67:
	cmpq $0,32(%rsi)
	jnz L66
L64:
	movq 48(%rbx),%rdi
	cmpq $0,%rdi
	jz L77
L78:
	movl (%rdi),%eax
	cmpl $1,%eax
	jnz L77
L75:
	movl (%rbx),%eax
	cmpl $1879244817,%eax
	jz L97
L228:
	cmpl $1880293392,%eax
	jnz L77
L86:
	movq 32(%rdi),%r12
	movq 40(%rbx),%rax
	movq 8(%rax),%rax
	testq $7168,%rax
	jz L91
L90:
	movsd 24(%rdi),%xmm1
	movsd 24(%rsi),%xmm0
	addsd %xmm0,%xmm1
	movsd %xmm1,-8(%rbp)
	jmp L92
L91:
	testq $340,%rax
	jz L94
L93:
	movq 24(%rdi),%rdi
	movq 24(%rsi),%rsi
	addq %rdi,%rsi
	movq %rsi,-8(%rbp)
	jmp L92
L94:
	movq 24(%rdi),%rdi
	movq 24(%rsi),%rsi
	addq %rdi,%rsi
	movq %rsi,-8(%rbp)
L92:
	leaq -8(%rbp),%rsi
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	call _con_normalize
	jmp L32
L97:
	movq 32(%rdi),%r12
	movq 40(%rbx),%rax
	movq 8(%rax),%rax
	testq $7168,%rax
	jz L102
L101:
	movsd 24(%rdi),%xmm1
	movsd 24(%rsi),%xmm0
	subsd %xmm0,%xmm1
	movsd %xmm1,-8(%rbp)
	jmp L103
L102:
	testq $340,%rax
	jz L105
L104:
	movq 24(%rdi),%rdi
	movq 24(%rsi),%rsi
	subq %rsi,%rdi
	movq %rdi,-8(%rbp)
	jmp L103
L105:
	movq 24(%rdi),%rdi
	subq 24(%rsi),%rdi
	movq %rdi,-8(%rbp)
L103:
	leaq -8(%rbp),%rsi
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	call _con_normalize
	jmp L32
L77:
	movq 48(%rbx),%rsi
	cmpq $0,%rsi
	jz L66
L115:
	movl (%rsi),%edi
	cmpl $1,%edi
	jnz L66
L111:
	cmpq $0,32(%rsi)
	jnz L66
L108:
	movl (%rbx),%edi
	cmpl $872415247,%edi
	jz L201
L231:
	cmpl $1879244819,%edi
	jz L123
L232:
	cmpl $1879244820,%edi
	jz L138
L233:
	cmpl $1879244821,%edi
	jz L162
L234:
	cmpl $1879244822,%edi
	jz L170
L235:
	cmpl $1880293394,%edi
	jz L151
L236:
	cmpl $1880293399,%edi
	jz L178
L237:
	cmpl $1880293400,%edi
	jz L186
L238:
	cmpl $1880293401,%edi
	jnz L66
L194:
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	testq $340,%rdi
	jz L198
L197:
	movq 24(%rsi),%rsi
	movq 56(%rbx),%rdi
	movq 24(%rdi),%rdi
	andq %rdi,%rsi
	movq %rsi,-8(%rbp)
	jmp L199
L198:
	movq 24(%rsi),%rsi
	movq 56(%rbx),%rdi
	andq 24(%rdi),%rsi
	movq %rsi,-8(%rbp)
L199:
	leaq -8(%rbp),%rsi
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	call _con_normalize
	jmp L32
L186:
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	testq $340,%rdi
	jz L190
L189:
	movq 24(%rsi),%rsi
	movq 56(%rbx),%rdi
	movq 24(%rdi),%rdi
	orq %rdi,%rsi
	movq %rsi,-8(%rbp)
	jmp L191
L190:
	movq 24(%rsi),%rsi
	movq 56(%rbx),%rdi
	orq 24(%rdi),%rsi
	movq %rsi,-8(%rbp)
L191:
	leaq -8(%rbp),%rsi
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	call _con_normalize
	jmp L32
L178:
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	testq $340,%rdi
	jz L182
L181:
	movq 24(%rsi),%rsi
	movq 56(%rbx),%rdi
	movq 24(%rdi),%rdi
	xorq %rdi,%rsi
	movq %rsi,-8(%rbp)
	jmp L183
L182:
	movq 24(%rsi),%rsi
	movq 56(%rbx),%rdi
	xorq 24(%rdi),%rsi
	movq %rsi,-8(%rbp)
L183:
	leaq -8(%rbp),%rsi
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	call _con_normalize
	jmp L32
L151:
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	testq $7168,%rdi
	jz L155
L154:
	movsd 24(%rsi),%xmm1
	movq 56(%rbx),%rsi
	movsd 24(%rsi),%xmm0
	mulsd %xmm0,%xmm1
	movsd %xmm1,-8(%rbp)
	jmp L156
L155:
	testq $340,%rdi
	jz L158
L157:
	movq 24(%rsi),%rsi
	movq 56(%rbx),%rdi
	movq 24(%rdi),%rdi
	imulq %rdi,%rsi
	movq %rsi,-8(%rbp)
	jmp L156
L158:
	movq 24(%rsi),%rsi
	movq 56(%rbx),%rdi
	imulq 24(%rdi),%rsi
	movq %rsi,-8(%rbp)
L156:
	leaq -8(%rbp),%rsi
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	call _con_normalize
	jmp L32
L170:
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	testq $340,%rdi
	jz L174
L173:
	movq 24(%rsi),%rsi
	movq 56(%rbx),%rdi
	movq 24(%rdi),%rcx
	shlq %cl,%rsi
	movq %rsi,-8(%rbp)
	jmp L175
L174:
	movq 24(%rsi),%rsi
	movq 56(%rbx),%rdi
	movq 24(%rdi),%rcx
	shlq %cl,%rsi
	movq %rsi,-8(%rbp)
L175:
	leaq -8(%rbp),%rsi
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	call _con_normalize
	jmp L32
L162:
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	testq $340,%rdi
	jz L166
L165:
	movq 24(%rsi),%rsi
	movq 56(%rbx),%rdi
	movq 24(%rdi),%rcx
	sarq %cl,%rsi
	movq %rsi,-8(%rbp)
	jmp L167
L166:
	movq 24(%rsi),%rsi
	movq 56(%rbx),%rdi
	movq 24(%rdi),%rcx
	shrq %cl,%rsi
	movq %rsi,-8(%rbp)
L167:
	leaq -8(%rbp),%rsi
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	call _con_normalize
	jmp L32
L138:
	movq 56(%rbx),%rdi
	call _operand_is_zero
	cmpl $0,%eax
	jnz L66
L142:
	movq 40(%rbx),%rsi
	movq 8(%rsi),%rsi
	testq $340,%rsi
	jz L146
L145:
	movq 48(%rbx),%rsi
	movq 24(%rsi),%rax
	movq 56(%rbx),%rsi
	movq 24(%rsi),%rsi
	cqto
	idivq %rsi
	movq %rdx,-8(%rbp)
	jmp L147
L146:
	movq 48(%rbx),%rsi
	movq 24(%rsi),%rax
	movq 56(%rbx),%rsi
	movq 24(%rsi),%rsi
	xorl %edx,%edx
	divq %rsi
	movq %rdx,-8(%rbp)
L147:
	leaq -8(%rbp),%rsi
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	call _con_normalize
	jmp L32
L123:
	movq 56(%rbx),%rdi
	call _operand_is_zero
	cmpl $0,%eax
	jnz L66
L127:
	movq 40(%rbx),%rsi
	movq 8(%rsi),%rsi
	testq $7168,%rsi
	jz L131
L130:
	movq 48(%rbx),%rsi
	movsd 24(%rsi),%xmm1
	movq 56(%rbx),%rsi
	movsd 24(%rsi),%xmm0
	divsd %xmm0,%xmm1
	movsd %xmm1,-8(%rbp)
	jmp L132
L131:
	testq $340,%rsi
	jz L134
L133:
	movq 48(%rbx),%rsi
	movq 24(%rsi),%rax
	movq 56(%rbx),%rsi
	movq 24(%rsi),%rsi
	cqto
	idivq %rsi
	movq %rax,-8(%rbp)
	jmp L132
L134:
	movq 48(%rbx),%rsi
	movq 24(%rsi),%rax
	movq 56(%rbx),%rsi
	movq 24(%rsi),%rsi
	xorl %edx,%edx
	divq %rsi
	movq %rax,-8(%rbp)
L132:
	leaq -8(%rbp),%rsi
	movq 40(%rbx),%rdi
	movq 8(%rdi),%rdi
	call _con_normalize
	jmp L32
L201:
	movl $1,_changed(%rip)
	movl $1,%r14d
	movq 56(%rbx),%rsi
	movq 48(%rbx),%rdi
	leaq 24(%rsi),%rax
	leaq 24(%rdi),%rsi
	movq 8(%rdi),%rdi
	movq %rax,%rdx
	call _con_cmp
	movl %eax,%r15d
	pushq $524288
	pushq %rbx
	call _insn_replace
	addq $16,%rsp
	jmp L8
L66:
	movl (%rbx),%esi
	testl $8388608,%esi
	jz L205
L206:
	cmpl $0,%r14d
	jz L205
L203:
	leal -1082654746(%rsi),%ecx
	movl $1,%esi
	shll %cl,%esi
	testl %esi,%r15d
	jz L211
L210:
	movq $1,-8(%rbp)
	jmp L32
L211:
	movq $0,-8(%rbp)
L32:
	movq 40(%rbx),%rsi
	movq 8(%rsi),%rsi
	subq $8,%rsp
	movq -8(%rbp),%rdi
	movq %rdi,(%rsp)
	movq %rsi,%rdi
	call _operand_con
	addq $8,%rsp
	movq %rax,%r13
	movq 40(%rbx),%rdi
	call _operand_dup
	pushq %r13
	pushq %rax
	pushq $1611333643
	pushq %rbx
	call _insn_replace
	addq $32,%rsp
	movq %r12,32(%r13)
	movl $1,_changed(%rip)
	jmp L8
L205:
	movq %rbx,%rdi
	call _insn_defs_cc
	cmpl $0,%eax
	jz L8
L214:
	xorl %r14d,%r14d
L8:
	movq 64(%rbx),%rbx
	jmp L6
L9:
	cmpl $0,%r14d
	jz L220
L218:
	movq -16(%rbp),%rdi	 # spill
	movl %r15d,%esi
	call _block_known_ccs
L220:
	xorl %eax,%eax
L5:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L243:
_fold:
L244:
	pushq %rbp
	movq %rsp,%rbp
L245:
	movl $0,_changed(%rip)
	movq $_fold0,%rdi
	call _blocks_iter
	movl _changed(%rip),%esi
	cmpl $0,%esi
	jz L246
L247:
	call _nop
	call _unreach
	call _prop
L246:
	popq %rbp
	ret
L253:
.globl _nop
.globl _con_cmp
.globl _prop
.globl _operand_dup
.globl _blocks_iter
.globl _block_known_ccs
.globl _fold
.globl _con_cast
.globl _con_normalize
.globl _insn_normalize
.globl _operand_is_zero
.globl _insn_defs_cc
.local _changed
.comm _changed, 4, 4
.globl _insn_replace
.globl _unreach
.globl _operand_con
