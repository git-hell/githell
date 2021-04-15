.text
_gen_branch:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L5:
	movq %rdi,%rbx
	movq %rsi,%r12
	movq %rdx,%r14
	movl $64,%edi
	xorl %esi,%esi
	xorl %edx,%edx
	call _operand_i
	movq %rax,%r13
	movq %rbx,%rdi
	call _operand_leaf
	pushq %r13
	pushq %rax
	pushq $872415247
	call _insn_new
	addq $24,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	movq _current_block(%rip),%rdi
	xorl %esi,%esi
	movq %r14,%rdx
	call _block_add_successor
	movq _current_block(%rip),%rdi
	movl $1,%esi
	movq %r12,%rdx
	call _block_add_successor
L3:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L7:
_gen_frame:
L9:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L14:
	movq %rdi,%r12
	call _symbol_storage
	movq _target(%rip),%rsi
	movq 16(%rsi),%rdi
	call _symbol_temp_reg
	movl %eax,%ebx
	movl 64(%r12),%esi
	movslq %esi,%rsi
	movl $64,%edi
	xorl %edx,%edx
	call _operand_i
	movq %rax,%r12
	movq _target(%rip),%rsi
	movq 16(%rsi),%rdi
	movl %ebx,%esi
	call _operand_reg
	pushq %r12
	pushq %rax
	pushq $1611268097
	call _insn_new
	addq $24,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	movl %ebx,%eax
L11:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L16:
_gen_args0:
L18:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L39:
	movq %rdi,%rbx
	movq 32(%rbx),%rsi
	movq (%rsi),%rdi
	testq $65536,%rdi
	jnz L20
L23:
	movl 12(%rbx),%esi
	testl $128,%esi
	jz L26
L28:
	testq $262144,%rdi
	jnz L26
L25:
	movl 56(%rbx),%esi
	cmpl $2147483649,%esi
	jz L33
L32:
	call _operand_reg
	movq %rax,%r12
	movq %rbx,%rdi
	call _symbol_reg
	movq 32(%rbx),%rsi
	movq (%rsi),%rdi
	movl %eax,%esi
	call _operand_reg
	pushq %r12
	pushq %rax
	pushq $1611333643
	call _insn_new
	addq $24,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	jmp L20
L33:
	movq %rbx,%rdi
	call _gen_frame
	movq _target(%rip),%rsi
	movq 16(%rsi),%rdi
	movl %eax,%esi
	call _operand_reg
	movq %rax,%r12
	movq %rbx,%rdi
	call _symbol_reg
	movq 32(%rbx),%rsi
	movq (%rsi),%rdi
	movl %eax,%esi
	call _operand_reg
	pushq %r12
	pushq %rax
	pushq $1644888066
	call _insn_new
	addq $24,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	jmp L20
L26:
	movl 56(%rbx),%esi
	cmpl $2147483649,%esi
	jz L20
L35:
	movq %rbx,%rdi
	call _gen_frame
	movl %eax,%r12d
	movl 56(%rbx),%esi
	movq 32(%rbx),%rdi
	movq (%rdi),%rdi
	call _operand_reg
	movq %rax,%rbx
	movq _target(%rip),%rsi
	movq 16(%rsi),%rdi
	movl %r12d,%esi
	call _operand_reg
	pushq %rbx
	pushq %rax
	pushq $1632108547
	call _insn_new
	addq $24,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
L20:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L41:
_gen_args:
L42:
	pushq %rbp
	movq %rsp,%rbp
L43:
	movq _entry_block(%rip),%rdi
	call _block_always_successor
	movq _entry_block(%rip),%rdi
	movq %rax,%rsi
	call _block_split_edge
	movq %rax,_current_block(%rip)
	movq $_gen_args0,%rdi
	call _scope_walk_args
L44:
	popq %rbp
	ret
L48:
_gen_load:
L50:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L57:
	movq %rdi,%r12
	movq %rsi,%r13
	movq %r13,%rdi
	call _operand_leaf
	movq %rax,%rbx
	movq %r12,%rdi
	call _operand_sym
	pushq %rbx
	pushq %rax
	pushq $1644888066
	call _insn_new
	addq $24,%rsp
	movq 8(%r13),%rsi
	movq 16(%rsi),%rsi
	movq (%rsi),%rsi
	testq $262144,%rsi
	jz L55
L53:
	orl $1,4(%rax)
L55:
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
L52:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L59:
_gen_store:
L61:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L68:
	movq %rdi,%r12
	movq %rsi,%rdi
	call _operand_leaf
	movq %rax,%rbx
	movq %r12,%rdi
	call _operand_leaf
	pushq %rbx
	pushq %rax
	pushq $1632108547
	call _insn_new
	addq $24,%rsp
	movq 8(%r12),%rsi
	movq 16(%rsi),%rsi
	movq (%rsi),%rsi
	testq $262144,%rsi
	jz L66
L64:
	orl $1,4(%rax)
L66:
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
L63:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L70:
_gen_addrof:
L72:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L80:
	movq %rdi,%r13
	movq 24(%r13),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,24(%r13)
	leaq 8(%r13),%rdi
	call _symbol_temp
	movq %rax,%rbx
	movq 24(%r13),%rsi
	movq 32(%rsi),%r12
	movq %r12,%rdi
	call _symbol_storage
	movl 12(%r12),%esi
	testl $256,%esi
	jz L77
L75:
	andl $-257,%esi
	movl %esi,12(%r12)
	orl $64,%esi
	movl %esi,12(%r12)
L77:
	movl 64(%r12),%esi
	movslq %esi,%rsi
	movl $64,%edi
	xorl %edx,%edx
	call _operand_i
	movq %rax,%r12
	movq %rbx,%rdi
	call _operand_sym
	pushq %r12
	pushq %rax
	pushq $1611268097
	call _insn_new
	addq $24,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	movq %r13,%rdi
	call _tree_free
	movq %rbx,%rdi
	call _tree_sym
L74:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L82:
_gen_blkasg:
L84:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L89:
	movq %rdi,%r13
	movq 24(%r13),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,24(%r13)
	movq 32(%r13),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,32(%r13)
	leaq 8(%r13),%rdi
	movl $1,%esi
	call _type_sizeof
	movq _target(%rip),%rsi
	movq 16(%rsi),%rdi
	movq %rax,%rsi
	xorl %edx,%edx
	call _operand_i
	movq %rax,%rbx
	movq 32(%r13),%rdi
	call _operand_leaf
	movq %rax,%r12
	movq 24(%r13),%rdi
	call _operand_leaf
	pushq %rbx
	pushq %r12
	pushq %rax
	pushq $1900019716
	call _insn_new
	addq $32,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	movq %r13,%rdi
	call _tree_chop_binary
L86:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L91:
_extract:
L93:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L97:
	movl %esi,%r12d
	movl %edx,%ebx
	movq %rdi,%r14
	leaq 32(%r14),%rdi
	xorl %esi,%esi
	call _type_sizeof
	leal (,%rax,8),%r13d
	leal (%r12,%rbx),%esi
	movl %r13d,%edi
	subl %esi,%edi
	movslq %edi,%rsi
	subl %r12d,%r13d
	movl $64,%edi
	xorl %edx,%edx
	call _operand_i
	movq %rax,%rbx
	movq %r14,%rdi
	call _operand_sym
	movq %rax,%r12
	movq %r14,%rdi
	call _operand_sym
	pushq %rbx
	pushq %r12
	pushq %rax
	pushq $1879244822
	call _insn_new
	addq $32,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	movslq %r13d,%rsi
	movl $64,%edi
	xorl %edx,%edx
	call _operand_i
	movq %rax,%rbx
	movq %r14,%rdi
	call _operand_sym
	movq %rax,%r12
	movq %r14,%rdi
	call _operand_sym
	pushq %rbx
	pushq %r12
	pushq %rax
	pushq $1879244821
	call _insn_new
	addq $32,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
L95:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L99:
_gen_fetch:
L101:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L123:
	movl %esi,%r13d
	movq %rdi,%r12
	xorl %ebx,%ebx
	movq 24(%r12),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,24(%r12)
	movq 8(%r12),%rsi
	movq (%rsi),%rsi
	testq $65536,%rsi
	jnz L106
L104:
	leaq 8(%r12),%rdi
	call _symbol_temp
	movq %rax,%r14
	movq %r14,%rbx
	movq 24(%r12),%rsi
	movq %r14,%rdi
	call _gen_load
	movq 24(%r12),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rdi
	testq $32768,%rdi
	jz L106
L110:
	movq 16(%rsi),%rsi
	movq (%rsi),%rsi
	movl $2147483648,%edi
	testq %rdi,%rsi
	jz L106
L107:
	movq $270582939648,%rdi
	movq %rsi,%rdx
	andq %rdi,%rdx
	sarq $32,%rdx
	movq $34909494181888,%rdi
	andq %rdi,%rsi
	sarq $38,%rsi
	movq %r14,%rdi
	call _extract
L106:
	cmpl $1,%r13d
	jz L116
L114:
	movq %r12,%rdi
	call _tree_free
L116:
	cmpq $0,%rbx
	jz L118
L117:
	movq %rbx,%rdi
	call _tree_sym
	jmp L103
L118:
	call _tree_v
L103:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L125:
_insert:
L127:
	pushq %rbp
	movq %rsp,%rbp
	subq $24,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L132:
	movl %edx,%r12d
	movq %rdi,-24(%rbp)	 # spill
	movl %ecx,%ebx
	movq %rsi,-8(%rbp)	 # spill
	movq -8(%rbp),%r10	 # spill
	leaq 8(%r10),%r13
	movq %r13,%rdi
	xorl %esi,%esi
	call _type_sizeof
	movq %rax,%rsi
	shll $3,%esi
	movslq %esi,%rsi
	movl $64,%edi
	subq %rsi,%rdi
	movq $-1,%rsi
	movq %rdi,%rcx
	shrq %cl,%rsi
	movq %rsi,%r15
	movl %r12d,%edi
	movl %ebx,%esi
	call _field_mask
	movq %rax,%r14
	movq %r13,%rdi
	call _symbol_temp
	movq %rax,-16(%rbp)	 # spill
	movq %r13,%rdi
	call _symbol_temp
	movq %rax,%r13
	movslq %ebx,%rsi
	movl $64,%edi
	xorl %edx,%edx
	call _operand_i
	movq %rax,%rbx
	movq -8(%rbp),%rdi	 # spill
	call _operand_leaf
	movq %rax,%r12
	movq %r13,%rdi
	call _operand_sym
	movq %rax,%rsi
	pushq %rbx
	pushq %r12
	pushq %rsi
	pushq $1879244822
	call _insn_new
	addq $32,%rsp
	movq %rax,%rsi
	movq _current_block(%rip),%rdi
	addq $8,%rdi
	call _insn_append
	movq -8(%rbp),%r10	 # spill
	movq 8(%r10),%rsi
	movq (%rsi),%rsi
	andq $131071,%rsi
	movq %rsi,%rdi
	movq %r14,%rsi
	xorl %edx,%edx
	call _operand_i
	movq %rax,%rbx
	movq %r13,%rdi
	call _operand_sym
	movq %rax,%r12
	movq %r13,%rdi
	call _operand_sym
	movq %rax,%rsi
	pushq %rbx
	pushq %r12
	pushq %rsi
	pushq $1880293401
	call _insn_new
	addq $32,%rsp
	movq %rax,%rsi
	movq _current_block(%rip),%rdi
	addq $8,%rdi
	call _insn_append
	movq %r14,%rsi
	notq %rsi
	andq %r15,%rsi
	movq %rsi,%rbx
	movq -16(%rbp),%rdi	 # spill
	movq -24(%rbp),%rsi	 # spill
	call _gen_load
	movq -8(%rbp),%r10	 # spill
	movq 8(%r10),%rsi
	movq (%rsi),%rsi
	andq $131071,%rsi
	movq %rsi,%rdi
	movq %rbx,%rsi
	xorl %edx,%edx
	call _operand_i
	movq %rax,%rbx
	movq -16(%rbp),%rdi	 # spill
	call _operand_sym
	movq %rax,%r12
	movq -16(%rbp),%rdi	 # spill
	call _operand_sym
	movq %rax,%rsi
	pushq %rbx
	pushq %r12
	pushq %rsi
	pushq $1880293401
	call _insn_new
	addq $32,%rsp
	movq %rax,%rsi
	movq _current_block(%rip),%rdi
	addq $8,%rdi
	call _insn_append
	movq %r13,%rdi
	call _operand_sym
	movq %rax,%rbx
	movq -16(%rbp),%rdi	 # spill
	call _operand_sym
	movq %rax,%r12
	movq -16(%rbp),%rdi	 # spill
	call _operand_sym
	movq %rax,%rsi
	pushq %rbx
	pushq %r12
	pushq %rsi
	pushq $1880293400
	call _insn_new
	addq $32,%rsp
	movq %rax,%rsi
	movq _current_block(%rip),%rdi
	addq $8,%rdi
	call _insn_append
	movq -8(%rbp),%rdi	 # spill
	call _tree_free
	movq -16(%rbp),%rdi	 # spill
	call _tree_sym
L129:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L134:
_gen_asg:
L136:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L158:
	movq %rdi,%rbx
	movq 32(%rbx),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,32(%rbx)
	movq 24(%rbx),%rsi
	movl (%rsi),%edi
	cmpl $1073741829,%edi
	jnz L140
L139:
	movq 24(%rsi),%rdi
	xorl %esi,%esi
	call _gen
	movq 24(%rbx),%rsi
	movq %rax,24(%rsi)
	movq 24(%rbx),%rsi
	movq 24(%rsi),%rdi
	movq 8(%rdi),%rsi
	movq (%rsi),%rax
	testq $32768,%rax
	jz L144
L145:
	movq 16(%rsi),%rsi
	movq (%rsi),%rdx
	movl $2147483648,%esi
	testq %rsi,%rdx
	jz L144
L142:
	movq $270582939648,%rsi
	movq %rdx,%rcx
	andq %rsi,%rcx
	sarq $32,%rcx
	movq $34909494181888,%rsi
	andq %rsi,%rdx
	sarq $38,%rdx
	movq 32(%rbx),%rsi
	call _insert
	movq %rax,32(%rbx)
L144:
	movq 32(%rbx),%rsi
	movq 24(%rbx),%rdi
	movq 24(%rdi),%rdi
	call _gen_store
	movq 24(%rbx),%rsi
	movq 24(%rsi),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rdi
	testq $32768,%rdi
	jz L141
L152:
	movq 16(%rsi),%rsi
	movq (%rsi),%rsi
	movl $2147483648,%edi
	testq %rdi,%rsi
	jz L141
L149:
	movq 32(%rbx),%rsi
	leaq 8(%rsi),%rdi
	call _symbol_temp
	movq %rax,%r13
	movq 32(%rbx),%rdi
	call _operand_leaf
	movq %rax,%r12
	movq %r13,%rdi
	call _operand_sym
	pushq %r12
	pushq %rax
	pushq $1611333643
	call _insn_new
	addq $24,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	movq 24(%rbx),%rsi
	movq 24(%rsi),%rsi
	movq 8(%rsi),%rsi
	movq 16(%rsi),%rsi
	movq (%rsi),%rsi
	movq $270582939648,%rdi
	movq %rsi,%rdx
	andq %rdi,%rdx
	sarq $32,%rdx
	movq $34909494181888,%rdi
	andq %rdi,%rsi
	sarq $38,%rsi
	movq %r13,%rdi
	call _extract
	movq 32(%rbx),%rdi
	call _tree_free
	movq %r13,%rdi
	call _tree_sym
	movq %rax,32(%rbx)
	jmp L141
L140:
	movq %rax,%rdi
	call _operand_leaf
	movq %rax,%r12
	movq 24(%rbx),%rdi
	call _operand_leaf
	pushq %r12
	pushq %rax
	pushq $1611333643
	call _insn_new
	addq $24,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
L141:
	movq %rbx,%rdi
	call _tree_commute
	movq %rbx,%rdi
	call _tree_chop_binary
	movq %rax,%rdi
	xorl %esi,%esi
	call _gen
L138:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L160:
_gen_cast:
L162:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L178:
	movq %rdi,%r13
	movq 24(%r13),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,24(%r13)
	movq 8(%r13),%rsi
	movq (%rsi),%rsi
	testq $1,%rsi
	jz L166
L165:
	movq %r13,%rdi
	call _tree_free
	call _tree_v
	jmp L164
L166:
	leaq 8(%r13),%rdi
	call _symbol_temp
	movq %rax,%rbx
	movq 8(%r13),%rsi
	movq (%rsi),%rsi
	testq $33790,%rsi
	jz L170
L172:
	movq %r13,%rdi
	call _cast_narrow
	cmpl $0,%eax
	jz L170
L169:
	movq 24(%r13),%rdi
	call _operand_leaf
	movq %rax,%r12
	movq %rbx,%rdi
	call _operand_sym
	pushq %r12
	pushq %rax
	pushq $1611333643
	call _insn_new
	addq $24,%rsp
	movq 40(%rax),%rsi
	movq 8(%rsi),%rsi
	movq 48(%rax),%rdi
	movq %rsi,8(%rdi)
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	jmp L171
L170:
	movq 24(%r13),%rdi
	call _operand_leaf
	movq %rax,%r12
	movq %rbx,%rdi
	call _operand_sym
	pushq %r12
	pushq %rax
	pushq $1610809356
	call _insn_new
	addq $24,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
L171:
	movq %r13,%rdi
	call _tree_free
	movq %rbx,%rdi
	call _tree_sym
L164:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L180:
_gen_unary:
L182:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L190:
	movq %rdi,%r14
	movl (%r14),%esi
	cmpl $1082130439,%esi
	jnz L186
L185:
	movl $1610809357,%r13d
	jmp L187
L186:
	movl $1610809358,%r13d
L187:
	movq 24(%r14),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,24(%r14)
	leaq 8(%r14),%rdi
	call _symbol_temp
	movq %rax,%rbx
	movq 24(%r14),%rdi
	call _operand_leaf
	movq %rax,%r12
	movq %rbx,%rdi
	call _operand_sym
	pushq %r12
	pushq %rax
	pushq %r13
	call _insn_new
	addq $24,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	movq %r14,%rdi
	call _tree_free
	movq %rbx,%rdi
	call _tree_sym
L184:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L192:
.data
.align 4
_bin:
	.int 1880293399
	.int 1879244819
	.int 1880293394
	.int 1880293392
	.int 1879244817
	.int 1879244821
	.int 1879244822
	.int 1880293401
	.int 1880293400
	.int 1879244820
.text
_gen_binary:
L195:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L200:
	movq %rdi,%r14
	movq 24(%r14),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,24(%r14)
	movq 32(%r14),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,32(%r14)
	leaq 8(%r14),%rdi
	call _symbol_temp
	movq %rax,%r13
	movq 32(%r14),%rdi
	call _operand_leaf
	movq %rax,%rbx
	movq 24(%r14),%rdi
	call _operand_leaf
	movq %rax,%r12
	movq %r13,%rdi
	call _operand_sym
	movl (%r14),%esi
	andl $3840,%esi
	sarl $8,%esi
	movslq %esi,%rsi
	movl _bin(,%rsi,4),%esi
	pushq %rbx
	pushq %r12
	pushq %rax
	pushq %rsi
	call _insn_new
	addq $32,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	movq %r14,%rdi
	call _tree_free
	movq %r13,%rdi
	call _tree_sym
L197:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L202:
_gen_compound:
L204:
	pushq %rbp
	movq %rsp,%rbp
	subq $8,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L216:
	movq %rdi,%r12
	movl (%r12),%esi
	cmpl $268436265,%esi
	setz %sil
	movzbl %sil,%r15d
	leaq 8(%r12),%rsi
	movq %rsi,%rdi
	call _symbol_temp
	movq %rax,%rdi
	call _tree_sym
	movq %rax,-8(%rbp)	 # spill
	movq 32(%r12),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,32(%r12)
	movq 24(%r12),%rsi
	movl (%rsi),%edi
	cmpl $2147483650,%edi
	jnz L208
L207:
	movq %rsi,%rdi
	call _operand_leaf
	movq %rax,%rbx
	movq -8(%rbp),%rdi	 # spill
	call _operand_leaf
	pushq %rbx
	pushq %rax
	pushq $1611333643
	call _insn_new
	addq $24,%rsp
	movq %rax,%rsi
	movq _current_block(%rip),%rdi
	addq $8,%rdi
	call _insn_append
	movq 32(%r12),%rdi
	call _operand_leaf
	movq %rax,%r13
	movq 24(%r12),%rdi
	call _operand_leaf
	movq %rax,%rbx
	movq 24(%r12),%rdi
	call _operand_leaf
	movl (%r12),%esi
	andl $3840,%esi
	sarl $8,%esi
	movslq %esi,%rsi
	movl _bin(,%rsi,4),%esi
	pushq %r13
	pushq %rbx
	pushq %rax
	pushq %rsi
	call _insn_new
	addq $32,%rsp
	movq %rax,%rsi
	movq _current_block(%rip),%rdi
	addq $8,%rdi
	call _insn_append
	movq %r12,%rdi
	call _tree_chop_binary
	movq %rax,%rbx
	jmp L209
L208:
	movq %rsi,%rdi
	movl $1,%esi
	call _gen_fetch
	movq %rax,%r14
	movq %r14,%rdi
	call _operand_leaf
	movq %rax,%rbx
	movq -8(%rbp),%rdi	 # spill
	call _operand_leaf
	pushq %rbx
	pushq %rax
	pushq $1611333643
	call _insn_new
	addq $24,%rsp
	movq %rax,%rsi
	movq _current_block(%rip),%rdi
	addq $8,%rdi
	call _insn_append
	movq 32(%r12),%rdi
	call _operand_leaf
	movq %rax,%r13
	movq %r14,%rdi
	call _operand_leaf
	movq %rax,%rbx
	movq %r14,%rdi
	call _operand_leaf
	movl (%r12),%esi
	andl $3840,%esi
	sarl $8,%esi
	movslq %esi,%rsi
	movl _bin(,%rsi,4),%esi
	pushq %r13
	pushq %rbx
	pushq %rax
	pushq %rsi
	call _insn_new
	addq $32,%rsp
	movq %rax,%rsi
	movq _current_block(%rip),%rdi
	addq $8,%rdi
	call _insn_append
	movq 32(%r12),%rdi
	call _tree_free
	movq %r14,32(%r12)
	movq %r12,%rdi
	call _gen_asg
	movq %rax,%rbx
L209:
	cmpl $0,%r15d
	jz L211
L210:
	movq %rbx,%rdi
	call _tree_free
	movq -8(%rbp),%rax	 # spill
	jmp L206
L211:
	movq -8(%rbp),%rdi	 # spill
	call _tree_free
	movq %rbx,%rax
L206:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L218:
_gen_rel:
L220:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L260:
	movq %rdi,%rbx
	leaq 8(%rbx),%rdi
	call _symbol_temp
	movq %rax,%r14
	movq 24(%rbx),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,24(%rbx)
	movq 32(%rbx),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,32(%rbx)
	movq %rax,%rdi
	call _operand_leaf
	movq %rax,%r13
	movq 24(%rbx),%rdi
	call _operand_leaf
	pushq %r13
	pushq %rax
	pushq $872415247
	call _insn_new
	addq $24,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	movl (%rbx),%esi
	cmpl $33554458,%esi
	jz L231
L254:
	cmpl $33554460,%esi
	jz L241
L255:
	cmpl $33554461,%esi
	jz L236
L256:
	cmpl $33554463,%esi
	jz L246
L257:
	cmpl $637534242,%esi
	jz L227
L258:
	cmpl $637534243,%esi
	jnz L225
L229:
	movl $1,%r12d
	jmp L225
L227:
	xorl %r12d,%r12d
	jmp L225
L246:
	movq 24(%rbx),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rsi
	testq $340,%rsi
	jz L248
L247:
	movl $3,%r12d
	jmp L225
L248:
	movl $7,%r12d
	jmp L225
L236:
	movq 24(%rbx),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rsi
	testq $340,%rsi
	jz L238
L237:
	movl $5,%r12d
	jmp L225
L238:
	movl $9,%r12d
	jmp L225
L241:
	movq 24(%rbx),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rsi
	testq $340,%rsi
	jz L243
L242:
	movl $4,%r12d
	jmp L225
L243:
	movl $8,%r12d
	jmp L225
L231:
	movq 24(%rbx),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rsi
	testq $340,%rsi
	jz L233
L232:
	movl $2,%r12d
	jmp L225
L233:
	movl $6,%r12d
L225:
	movq %r14,%rdi
	call _operand_sym
	leal 1082654746(%r12),%esi
	pushq %rax
	pushq %rsi
	call _insn_new
	addq $16,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
	movq %rbx,%rdi
	call _tree_free
	movq %r14,%rdi
	call _tree_sym
L222:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L262:
_gen_log:
L264:
	pushq %rbp
	movq %rsp,%rbp
	subq $16,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L272:
	movq %rdi,-8(%rbp)	 # spill
	movq -8(%rbp),%r10	 # spill
	leaq 8(%r10),%rdi
	call _symbol_temp
	movq %rax,-16(%rbp)	 # spill
	call _block_new
	movq %rax,%r15
	call _block_new
	movq %rax,%r14
	call _block_new
	movq %rax,%r13
	call _block_new
	movq %rax,%rbx
	movl $64,%edi
	movl $1,%esi
	xorl %edx,%edx
	call _operand_i
	movq %rax,%r12
	movq -16(%rbp),%rdi	 # spill
	call _operand_sym
	pushq %r12
	pushq %rax
	pushq $1611333643
	call _insn_new
	addq $24,%rsp
	leaq 8(%r14),%rdi
	movq %rax,%rsi
	call _insn_append
	movl $64,%edi
	xorl %esi,%esi
	xorl %edx,%edx
	call _operand_i
	movq %rax,%r12
	movq -16(%rbp),%rdi	 # spill
	call _operand_sym
	pushq %r12
	pushq %rax
	pushq $1611333643
	call _insn_new
	addq $24,%rsp
	leaq 8(%r13),%rdi
	movq %rax,%rsi
	call _insn_append
	movq %r14,%rdi
	movl $10,%esi
	movq %rbx,%rdx
	call _block_add_successor
	movq %r13,%rdi
	movl $10,%esi
	movq %rbx,%rdx
	call _block_add_successor
	movq -8(%rbp),%r10	 # spill
	movq 24(%r10),%rdi
	xorl %esi,%esi
	call _gen
	movq -8(%rbp),%r10	 # spill
	movq %rax,24(%r10)
	movq -8(%rbp),%r10	 # spill
	movl (%r10),%esi
	cmpl $184549413,%esi
	jnz L268
L267:
	movq %rax,%rdi
	movq %r14,%rsi
	movq %r15,%rdx
	call _gen_branch
	jmp L269
L268:
	movq %rax,%rdi
	movq %r15,%rsi
	movq %r13,%rdx
	call _gen_branch
L269:
	movq %r15,_current_block(%rip)
	movq -8(%rbp),%r10	 # spill
	movq 32(%r10),%rdi
	xorl %esi,%esi
	call _gen
	movq -8(%rbp),%r10	 # spill
	movq %rax,32(%r10)
	movq %rax,%rdi
	movq %r14,%rsi
	movq %r13,%rdx
	call _gen_branch
	movq %rbx,_current_block(%rip)
	movq -8(%rbp),%rdi	 # spill
	call _tree_free
	movq -16(%rbp),%rdi	 # spill
	call _tree_sym
L266:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L274:
_gen_quest:
L276:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L293:
	movq %rdi,%r15
	movq 8(%r15),%rsi
	movq (%rsi),%rsi
	testq $1,%rsi
	jz L280
L279:
	xorl %r14d,%r14d
	jmp L281
L280:
	leaq 8(%r15),%rdi
	call _symbol_temp
	movq %rax,%r14
L281:
	call _block_new
	movq %rax,%r12
	call _block_new
	movq %rax,%rbx
	call _block_new
	movq %rax,%r13
	movq 24(%r15),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,24(%r15)
	movq %rax,%rdi
	movq %r12,%rsi
	movq %rbx,%rdx
	call _gen_branch
	movq %r12,_current_block(%rip)
	movq 32(%r15),%rsi
	movq 24(%rsi),%rdi
	xorl %esi,%esi
	call _gen
	movq 32(%r15),%rsi
	movq %rax,24(%rsi)
	cmpq $0,%r14
	jz L284
L282:
	movq 32(%r15),%rsi
	movq 24(%rsi),%rdi
	call _operand_leaf
	movq %rax,%r12
	movq %r14,%rdi
	call _operand_sym
	pushq %r12
	pushq %rax
	pushq $1611333643
	call _insn_new
	addq $24,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
L284:
	movq _current_block(%rip),%rdi
	movl $10,%esi
	movq %r13,%rdx
	call _block_add_successor
	movq %rbx,_current_block(%rip)
	movq 32(%r15),%rsi
	movq 32(%rsi),%rdi
	xorl %esi,%esi
	call _gen
	movq 32(%r15),%rsi
	movq %rax,32(%rsi)
	cmpq $0,%r14
	jz L287
L285:
	movq 32(%r15),%rsi
	movq 32(%rsi),%rdi
	call _operand_leaf
	movq %rax,%rbx
	movq %r14,%rdi
	call _operand_sym
	pushq %rbx
	pushq %rax
	pushq $1611333643
	call _insn_new
	addq $24,%rsp
	movq _current_block(%rip),%rsi
	leaq 8(%rsi),%rdi
	movq %rax,%rsi
	call _insn_append
L287:
	movq _current_block(%rip),%rdi
	movl $10,%esi
	movq %r13,%rdx
	call _block_add_successor
	movq %r13,_current_block(%rip)
	movq %r15,%rdi
	call _tree_free
	cmpq $0,%r14
	jz L289
L288:
	movq %r14,%rdi
	call _tree_sym
	jmp L278
L289:
	call _tree_v
L278:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L295:
_gen_comma:
L297:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L302:
	movq %rdi,%rbx
	movq 24(%rbx),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,24(%rbx)
	movq %rbx,%rdi
	call _tree_commute
	movq %rbx,%rdi
	call _tree_chop_binary
	movq %rax,%rdi
	xorl %esi,%esi
	call _gen
L299:
	popq %rbx
	popq %rbp
	ret
L304:
_gen_call:
L306:
	pushq %rbp
	movq %rsp,%rbp
	subq $40,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L359:
	movq %rdi,%r13
	leaq -24(%rbp),%rsi
	movq $0,-24(%rbp)
	movq %rsi,-16(%rbp)
	movl $2,-8(%rbp)
	movq 24(%r13),%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,%rsi
	movq %rsi,24(%r13)
	movq 8(%rsi),%rsi
	movq (%rsi),%rdi
	testq $32768,%rdi
	jz L316
L322:
	movq 16(%rsi),%rsi
	movq (%rsi),%rdi
	testq $16384,%rdi
	jz L316
L318:
	movq (%rsi),%rsi
	movq $-9223372036854775808,%rdi
	testq %rdi,%rsi
	jz L316
L315:
	movl $539230216,%esi
	jmp L317
L316:
	movl $539230215,%esi
L317:
	movl %esi,%r15d
	movq 8(%r13),%rsi
	movq (%rsi),%rsi
	testq $1,%rsi
	jz L327
L326:
	call _tree_v
	movq %rax,%rsi
	movq %rsi,-32(%rbp)	 # spill
	jmp L328
L327:
	leaq 8(%r13),%rsi
	movq %rsi,%rdi
	call _symbol_temp
	movq %rax,%rsi
	movq %rsi,-40(%rbp)	 # spill
	movq %rsi,%rdi
	call _tree_sym
	movq %rax,%rsi
	movq %rsi,-32(%rbp)	 # spill
L328:
	movq 8(%r13),%rsi
	movq (%rsi),%rsi
	testq $65536,%rsi
	jz L338
L329:
	movq -40(%rbp),%rdi	 # spill
	call _tree_sym
	movq %rax,%rsi
	movq %rsi,%rdi
	call _tree_addrof
	movq %rax,%rsi
	movq 32(%r13),%rax
	leaq 48(%rsi),%rdi
	movq %rax,48(%rsi)
	cmpq $0,%rax
	jz L336
L335:
	movq 32(%r13),%rax
	movq %rdi,56(%rax)
	jmp L337
L336:
	movq %rdi,40(%r13)
L337:
	leaq 32(%r13),%rdi
	movq %rsi,32(%r13)
	movq %rdi,56(%rsi)
L338:
	movq 40(%r13),%rsi
	movq 8(%rsi),%rsi
	movq (%rsi),%rbx
	cmpq $0,%rbx
	jz L340
L341:
	movq 48(%rbx),%rsi
	cmpq $0,%rsi
	jz L345
L344:
	movq 56(%rbx),%rdi
	movq %rdi,56(%rsi)
	jmp L346
L345:
	movq 56(%rbx),%rsi
	movq %rsi,40(%r13)
L346:
	movq 48(%rbx),%rsi
	movq 56(%rbx),%rdi
	movq %rsi,(%rdi)
	movq 8(%rbx),%rsi
	movq (%rsi),%rsi
	testq $65536,%rsi
	jz L348
L347:
	leaq 8(%rbx),%rsi
	movq %rsi,%rdi
	xorl %esi,%esi
	call _type_sizeof
	movq %rax,%r14
	movq %rbx,%rdi
	call _tree_addrof
	movq %rax,%rsi
	movq %rsi,%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,%rbx
	movq %rbx,%r12
	movq _target(%rip),%rsi
	movq 16(%rsi),%rsi
	movq %rsi,%rdi
	movq %r14,%rsi
	xorl %edx,%edx
	call _operand_i
	movq %rax,%r14
	movq %rbx,%rdi
	call _operand_leaf
	movq %rax,%rsi
	pushq %r14
	pushq %rsi
	pushq $807665673
	call _insn_new
	addq $24,%rsp
	movq %rax,%rsi
	leaq -24(%rbp),%rdi
	call _insn_prepend
	jmp L349
L348:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _gen
	movq %rax,%rsi
	movq %rsi,%r12
	movq %rsi,%rdi
	call _operand_leaf
	movq %rax,%rsi
	pushq %rsi
	pushq %r15
	call _insn_new
	addq $16,%rsp
	movq %rax,%rsi
	leaq -24(%rbp),%rdi
	call _insn_prepend
L349:
	movq %r12,%rdi
	call _tree_free
	jmp L338
L340:
	movq 8(%r13),%rsi
	movq (%rsi),%rsi
	testq $1,%rsi
	jnz L350
L353:
	testq $65536,%rsi
	jz L351
L350:
	movq 24(%r13),%rdi
	call _operand_leaf
	movq %rax,%rsi
	pushq %rsi
	pushq $0
	pushq $1730150406
	call _insn_new
	addq $24,%rsp
	movq %rax,%rsi
	movq %rsi,%rbx
	movq _current_block(%rip),%rdi
	addq $8,%rdi
	call _insn_append
	jmp L352
L351:
	movq 24(%r13),%rdi
	call _operand_leaf
	movq %rax,%rbx
	movq -40(%rbp),%rdi	 # spill
	call _operand_sym
	movq %rax,%rsi
	pushq %rbx
	pushq %rsi
	pushq $1730150406
	call _insn_new
	addq $24,%rsp
	movq %rax,%rsi
	movq %rsi,%rbx
	movq _current_block(%rip),%rdi
	addq $8,%rdi
	call _insn_append
L352:
	leaq -24(%rbp),%rax
	movq _current_block(%rip),%rsi
	addq $8,%rsi
	movq %rsi,%rdi
	movq %rbx,%rsi
	movq %rax,%rdx
	call _insns_insert_before
	movq %r13,%rdi
	call _tree_free
	movq -32(%rbp),%rax	 # spill
L308:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L361:
_gen:
L362:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L517:
	movl %esi,%r12d
	movq %rdi,%rbx
	testl $1,%r12d
	jz L367
L365:
	movq %rbx,%rdi
	call _tree_rewrite_volatile
	movq %rax,%rdi
	call _tree_opt
	movq %rax,%rdi
	call _tree_simplify
	movq %rax,%rbx
	movl _debug_flag_e(%rip),%esi
	cmpl $0,%esi
	jz L367
L368:
	movq %rax,%rdi
	xorl %esi,%esi
	call _tree_debug
L367:
	movl (%rbx),%esi
	cmpl $184549413,%esi
	jz L429
	jb L438
L478:
	cmpl $637534242,%esi
	jae L516
L479:
	cmpl $402654223,%esi
	jz L419
	jb L480
L489:
	cmpl $549455648,%esi
	jz L405
	jb L490
L495:
	cmpl $549455908,%esi
	jz L405
	jnz L373
L490:
	cmpl $549454359,%esi
	jz L405
	ja L373
L491:
	cmpl $549453845,%esi
	jz L405
	jnz L373
L480:
	cmpl $276825113,%esi
	jz L405
	jb L481
L486:
	cmpl $402653966,%esi
	jz L419
	jnz L373
L481:
	cmpl $268436265,%esi
	jz L419
	ja L373
L482:
	cmpl $201326602,%esi
	jnz L373
L407:
	movq %rbx,%rdi
	call _gen_asg
	movq %rax,%rbx
	jmp L373
L516:
	cmpl $637534243,%esi
	jbe L426
L498:
	cmpl $1073741830,%esi
	jz L375
	jb L499
L508:
	cmpl $1082130441,%esi
	jz L394
	ja L373
L509:
	cmpl $1082130439,%esi
	jnz L373
L394:
	movq %rbx,%rdi
	call _gen_unary
	movq %rax,%rbx
	jmp L373
L499:
	cmpl $1073741828,%esi
	jz L379
	jb L500
L505:
	cmpl $1073741829,%esi
	jnz L373
L381:
	movq %rbx,%rdi
	xorl %esi,%esi
	call _gen_fetch
	movq %rax,%rbx
	jmp L373
L500:
	cmpl $1073741827,%esi
	jz L387
	ja L373
L501:
	cmpl $817890072,%esi
	jz L405
	jnz L373
L387:
	movq %rbx,%rdi
	call _gen_call
	movq %rax,%rbx
	jmp L373
L379:
	movq %rbx,%rdi
	call _gen_cast
	movq %rax,%rbx
	jmp L373
L375:
	movq %rbx,%rdi
	call _gen_addrof
	movq %rax,%rbx
	jmp L373
L438:
	cmpl $134217996,%esi
	jz L419
	jb L439
L459:
	cmpl $134219294,%esi
	jz L405
	jb L460
L469:
	cmpl $134220045,%esi
	jz L419
	jb L470
L475:
	cmpl $184549409,%esi
	jz L429
	jnz L373
L470:
	cmpl $134219795,%esi
	jz L419
	ja L373
L471:
	cmpl $134219538,%esi
	jz L419
	jnz L373
L460:
	cmpl $134219035,%esi
	jz L405
	jb L461
L466:
	cmpl $134219280,%esi
	jz L419
	jnz L373
L461:
	cmpl $134219025,%esi
	jz L419
	ja L373
L462:
	cmpl $134218251,%esi
	jz L419
	jnz L373
L439:
	cmpl $2342,%esi
	jz L405
	jb L440
L449:
	cmpl $33554463,%esi
	jz L426
	jb L450
L456:
	cmpl $134217748,%esi
	jz L419
	jnz L373
L450:
	cmpl $33554460,%esi
	jae L455
L451:
	cmpl $33554458,%esi
	jz L426
	jnz L373
L455:
	cmpl $33554461,%esi
	ja L373
L426:
	movq %rbx,%rdi
	call _gen_rel
	movq %rax,%rbx
	jmp L373
L440:
	cmpl $43,%esi
	jz L377
	jb L441
L446:
	cmpl $278,%esi
	jz L405
	jnz L373
L441:
	cmpl $42,%esi
	jz L385
	ja L373
L442:
	cmpl $39,%esi
	jnz L373
L383:
	movq %rbx,%rdi
	call _gen_quest
	movq %rax,%rbx
	jmp L373
L385:
	movq %rbx,%rdi
	call _gen_comma
	movq %rax,%rbx
	jmp L373
L377:
	movq %rbx,%rdi
	call _gen_blkasg
	movq %rax,%rbx
	jmp L373
L405:
	movq %rbx,%rdi
	call _gen_binary
	movq %rax,%rbx
	jmp L373
L419:
	movq %rbx,%rdi
	call _gen_compound
	movq %rax,%rbx
	jmp L373
L429:
	movq %rbx,%rdi
	call _gen_log
	movq %rax,%rbx
L373:
	testl $2,%r12d
	jz L432
L431:
	movq %rbx,%rdi
	call _tree_free
	xorl %eax,%eax
	jmp L364
L432:
	movq %rbx,%rax
L364:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L519:
.globl _symbol_temp
.globl _block_always_successor
.globl _block_add_successor
.globl _gen_args
.globl _scope_walk_args
.globl _target
.globl _insn_prepend
.globl _insn_append
.globl _tree_rewrite_volatile
.globl _tree_commute
.globl _insns_insert_before
.globl _tree_v
.globl _cast_narrow
.globl _block_new
.globl _insn_new
.globl _tree_opt
.globl _tree_free
.globl _block_split_edge
.globl _symbol_storage
.globl _debug_flag_e
.globl _tree_addrof
.globl _operand_leaf
.globl _type_sizeof
.globl _tree_debug
.globl _operand_reg
.globl _symbol_reg
.globl _symbol_temp_reg
.globl _gen_branch
.globl _tree_simplify
.globl _tree_chop_binary
.globl _operand_i
.globl _field_mask
.globl _current_block
.globl _entry_block
.globl _tree_sym
.globl _operand_sym
.globl _gen
