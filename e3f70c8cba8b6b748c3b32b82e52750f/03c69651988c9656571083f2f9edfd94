.data
.align 8
_powtab0:
	.quad 0x3ff0000000000000
	.quad 0x3fb999999999999a
	.quad 0x3f847ae147ae147b
	.quad 0x3f50624dd2f1a9fc
	.quad 0x3f1a36e2eb1c432d
	.quad 0x3ee4f8b588e368f1
	.quad 0x3eb0c6f7a0b5ed8d
	.quad 0x3e7ad7f29abcaf48
	.quad 0x3e45798ee2308c3a
	.quad 0x3e112e0be826d695
	.quad 0x3ddb7cdfd9d7bdbb
	.quad 0x3da5fd7fe1796495
	.quad 0x3d719799812dea11
	.quad 0x3d3c25c268497682
	.quad 0x3d06849b86a12b9b
	.quad 0x3cd203af9ee75616
.align 8
_powtab1:
	.quad 0x3ff0000000000000
	.quad 0x4024000000000000
	.quad 0x4059000000000000
	.quad 0x408f400000000000
	.quad 0x40c3880000000000
	.quad 0x40f86a0000000000
	.quad 0x412e848000000000
	.quad 0x416312d000000000
	.quad 0x4197d78400000000
	.quad 0x41cdcd6500000000
	.quad 0x4202a05f20000000
	.quad 0x42374876e8000000
	.quad 0x426d1a94a2000000
	.quad 0x42a2309ce5400000
	.quad 0x42d6bcc41e900000
	.quad 0x430c6bf526340000
.align 8
_powtab2:
	.quad 0x4341c37937e08000
	.quad 0x4693b8b5b5056e17
	.quad 0x49e5e531a0a1c873
	.quad 0x4d384f03e93ff9f5
	.quad 0x508afcef51f0fb5f
	.quad 0x53ddf67562d8b363
	.quad 0x5730a1f5b8132466
	.quad 0x5a827748f9301d32
	.quad 0x5dd4805738b51a75
	.quad 0x6126c2d4256ffcc3
	.quad 0x647945145230b378
	.quad 0x67cc0e1ef1a724eb
	.quad 0x6b1f25c186a6f04c
	.quad 0x6e714a52dffc6799
	.quad 0x71c33234de7ad7e3
	.quad 0x75154fdd7f73bf3c
	.quad 0x7867a93a2954f3b8
	.quad 0x7bba44df832b8d46
	.quad 0x7f0d2a1be4048f90
.text
.align 8
L31:
	.quad 0x0
___pow10:
L4:
	pushq %rbp
	movq %rsp,%rbp
L5:
	cmpl $0,%edi
	jge L8
L7:
	negl %edi
	cmpl $16,%edi
	jge L11
L10:
	movslq %edi,%rsi
	movsd _powtab0(,%rsi,8),%xmm0
	jmp L6
L11:
	cmpl $307,%edi
	jg L15
L14:
	movl %edi,%esi
	andl $15,%esi
	movslq %esi,%rsi
	movsd _powtab0(,%rsi,8),%xmm0
	sarl $4,%edi
	leal -1(%rdi),%esi
	movslq %esi,%rsi
	movsd _powtab2(,%rsi,8),%xmm1
	divsd %xmm1,%xmm0
	jmp L6
L15:
	movsd L31(%rip),%xmm0
	jmp L6
L8:
	cmpl $16,%edi
	jge L20
L19:
	movslq %edi,%rsi
	movsd _powtab1(,%rsi,8),%xmm0
	jmp L6
L20:
	cmpl $308,%edi
	jg L24
L23:
	movl %edi,%esi
	andl $15,%esi
	movslq %esi,%rsi
	movsd _powtab1(,%rsi,8),%xmm0
	sarl $4,%edi
	leal -1(%rdi),%esi
	movslq %esi,%rsi
	movsd _powtab2(,%rsi,8),%xmm1
	mulsd %xmm1,%xmm0
	jmp L6
L24:
	movsd ___huge_val(%rip),%xmm0
L6:
	popq %rbp
	ret
L32:
.globl ___pow10
.globl ___huge_val
