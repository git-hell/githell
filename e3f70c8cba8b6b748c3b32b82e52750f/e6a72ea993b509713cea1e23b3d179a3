.data
.align 1
_tzdstdef:
	.byte 49,46,49,46,52,58,45,49
	.byte 46,49,46,49,48,58,50,58
	.byte 54,48,46,46,46,46,46,46
	.byte 0
.align 1
_daynames:
	.byte 83,117,110,77,111,110,84,117
	.byte 101,87,101,100,84,104,117,70
	.byte 114,105,83,97,116,0
.align 1
_dpm:
	.byte 31
	.byte 28
	.byte 31
	.byte 30
	.byte 31
	.byte 30
	.byte 31
	.byte 31
	.byte 30
	.byte 31
	.byte 30
	.byte 31
.align 4
_dstadjust:
	.int 3600
.align 1
_dsthour:
	.byte 2
.align 1
_dsttimes:
	.byte 1
	.byte 0
	.byte 3
	.byte 255
	.byte 0
	.byte 9
.align 1
_months:
	.byte 74,97,110,70,101,98,77,97
	.byte 114,65,112,114,77,97,121,74
	.byte 117,110,74,117,108,65,117,103
	.byte 83,101,112,79,99,116,78,111
	.byte 118,68,101,99,0
.align 1
_timestr:
	.byte 65,65,65,32,65,65,65,32
	.byte 68,68,32,68,68,58,68,68
	.byte 58,68,68,32,68,68,68,68
	.byte 10,0
.align 1
_tz0:
	.byte 71,77,84,0,0,0,0,0
	.byte 0,0,0,0,0,0,0,0
	.byte 0,0,0,0,0,0,0,0
	.byte 0,0,0,0,0,0,0,0
.align 1
_tz1:
	.byte 0,0,0,0,0,0,0,0
	.byte 0,0,0,0,0,0,0,0
	.byte 0,0,0,0,0,0,0,0
	.byte 0,0,0,0,0,0,0,0
.align 8
_tzname:
	.quad _tz0
	.quad _tz1
.text
_isleap:
L13:
	pushq %rbp
	movq %rsp,%rbp
L14:
	movl $4000,%esi
	movl %edi,%eax
	cltd
	idivl %esi
	cmpl $0,%edx
	jnz L18
L16:
	xorl %eax,%eax
	jmp L15
L18:
	movl $400,%esi
	movl %edi,%eax
	cltd
	idivl %esi
	cmpl $0,%edx
	jz L21
L20:
	movl $100,%esi
	movl %edi,%eax
	cltd
	idivl %esi
	cmpl $0,%edx
	jz L22
L24:
	movl $4,%esi
	movl %edi,%eax
	cltd
	idivl %esi
	cmpl $0,%edx
	jnz L22
L21:
	movl $1,%eax
	jmp L15
L22:
	xorl %eax,%eax
L15:
	popq %rbp
	ret
L32:
_nthday:
L34:
	pushq %rbp
	movq %rsp,%rbp
L35:
	movzbl (%rdi),%esi
	movl %esi,%ecx
	cmpl $0,%esi
	jnz L39
L37:
	movzbl 1(%rdi),%eax
	jmp L36
L39:
	movl _tm+12(%rip),%eax
	movl _tm+24(%rip),%edx
	subl %edx,%eax
	movzbl 1(%rdi),%edi
	addl %edi,%eax
	cmpl $0,%esi
	jle L58
L44:
	cmpl $0,%eax
	jle L47
L45:
	addl $-7,%eax
	jmp L44
L47:
	addl $7,%eax
	addl $-1,%ecx
	cmpl $0,%ecx
	jle L36
	jg L47
L58:
	movl _tm+16(%rip),%esi
	movslq %esi,%rsi
L50:
	movzbl _dpm(%rsi),%edi
	cmpl %edi,%eax
	jge L53
L51:
	addl $7,%eax
	jmp L50
L53:
	addl $-7,%eax
	addl $1,%ecx
	cmpl $0,%ecx
	jl L53
L36:
	popq %rbp
	ret
L61:
_isdaylight:
L63:
	pushq %rbp
	movq %rsp,%rbp
L64:
	movq _tzname+8(%rip),%rsi
	movzbl (%rsi),%esi
	cmpl $0,%esi
	jnz L68
L66:
	xorl %eax,%eax
	jmp L65
L68:
	movl _tm+16(%rip),%esi
	movzbl _dsttimes+2(%rip),%edi
	movzbl _dsttimes+5(%rip),%eax
	cmpl %eax,%edi
	jg L73
L77:
	cmpl %edi,%esi
	jl L70
L81:
	cmpl %eax,%esi
	jg L70
L73:
	cmpl %eax,%edi
	jle L71
L85:
	cmpl %edi,%esi
	jge L71
L89:
	cmpl %eax,%esi
	jle L71
L70:
	xorl %eax,%eax
	jmp L65
L71:
	cmpl %edi,%esi
	jnz L95
L94:
	movq $_dsttimes,%rdi
	call _nthday
	movl _tm+12(%rip),%esi
	cmpl %eax,%esi
	jz L99
L97:
	cmpl %eax,%esi
	setg %sil
	movzbl %sil,%eax
	jmp L65
L99:
	movl _tm+8(%rip),%esi
	movzbl _dsthour(%rip),%edi
	cmpl %edi,%esi
	setge %sil
	movzbl %sil,%eax
	jmp L65
L95:
	cmpl %eax,%esi
	jnz L103
L102:
	movq $_dsttimes+3,%rdi
	call _nthday
	movl _tm+12(%rip),%esi
	cmpl %eax,%esi
	jz L107
L105:
	cmpl %eax,%esi
	setl %sil
	movzbl %sil,%eax
	jmp L65
L107:
	movl _tm+8(%rip),%esi
	movzbl _dsthour(%rip),%edi
	addl $-1,%edi
	cmpl %edi,%esi
	setl %sil
	movzbl %sil,%eax
	jmp L65
L103:
	movl $1,%eax
L65:
	popq %rbp
	ret
L114:
_setdst:
L116:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L181:
	movq %rdi,%rbx
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L121
L119:
	movq %rbx,%rdi
	call _atoi
	movb %al,_dsttimes(%rip)
L122:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L124
L125:
	movzbl (%rbx),%esi
	addq $1,%rbx
	cmpl $46,%esi
	jnz L122
L124:
	movq %rbx,%rdi
	call _atoi
	leal -1(%rax),%esi
	movb %sil,_dsttimes+1(%rip)
L129:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L131
L132:
	movzbl (%rbx),%esi
	addq $1,%rbx
	cmpl $46,%esi
	jnz L129
L131:
	movq %rbx,%rdi
	call _atoi
	leal -1(%rax),%esi
	movb %sil,_dsttimes+2(%rip)
L136:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L121
L139:
	movzbl (%rbx),%esi
	addq $1,%rbx
	cmpl $58,%esi
	jnz L136
L121:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L145
L143:
	movq %rbx,%rdi
	call _atoi
	movb %al,_dsttimes+3(%rip)
L146:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L148
L149:
	movzbl (%rbx),%esi
	addq $1,%rbx
	cmpl $46,%esi
	jnz L146
L148:
	movq %rbx,%rdi
	call _atoi
	leal -1(%rax),%esi
	movb %sil,_dsttimes+4(%rip)
L153:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L155
L156:
	movzbl (%rbx),%esi
	addq $1,%rbx
	cmpl $46,%esi
	jnz L153
L155:
	movq %rbx,%rdi
	call _atoi
	leal -1(%rax),%esi
	movb %sil,_dsttimes+5(%rip)
L160:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L145
L163:
	movzbl (%rbx),%esi
	addq $1,%rbx
	cmpl $58,%esi
	jnz L160
L145:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L169
L167:
	movq %rbx,%rdi
	call _atoi
	movb %al,_dsthour(%rip)
L170:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L169
L173:
	movzbl (%rbx),%esi
	addq $1,%rbx
	cmpl $58,%esi
	jnz L170
L169:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L118
L177:
	movq %rbx,%rdi
	call _atoi
	imull $60,%eax
	movl %eax,_dstadjust(%rip)
L118:
	popq %rbx
	popq %rbp
	ret
L183:
.data
.align 4
L187:
	.int 0
.text
_tzset:
L184:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L185:
	movl L187(%rip),%esi
	cmpl $0,%esi
	jnz L186
L190:
	movl $1,L187(%rip)
	movq $L195,%rdi
	call _getenv
	movq %rax,%rbx
	cmpq $0,%rax
	jz L186
L194:
	movq $0,_timezone(%rip)
	movq _tzname(%rip),%rsi
L197:
	movzbl (%rbx),%edi
	cmpl $0,%edi
	jz L199
L204:
	cmpl $58,%edi
	jz L199
L200:
	movq _tzname(%rip),%rdi
	addq $31,%rdi
	cmpq %rdi,%rsi
	jae L199
L198:
	movzbl (%rbx),%edi
	addq $1,%rbx
	movb %dil,(%rsi)
	addq $1,%rsi
	jmp L197
L199:
	movb $0,(%rsi)
L208:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L210
L211:
	movzbl (%rbx),%esi
	addq $1,%rbx
	cmpl $58,%esi
	jnz L208
L210:
	movq %rbx,%rdi
	call _atoi
	movslq %eax,%rsi
	imulq $60,%rsi
	movq %rsi,_timezone(%rip)
L215:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L217
L218:
	movzbl (%rbx),%esi
	addq $1,%rbx
	cmpl $58,%esi
	jnz L215
L217:
	movq _tzname+8(%rip),%rsi
L222:
	movzbl (%rbx),%edi
	cmpl $0,%edi
	jz L224
L229:
	cmpl $58,%edi
	jz L224
L225:
	movq _tzname+8(%rip),%rdi
	addq $31,%rdi
	cmpq %rdi,%rsi
	jae L224
L223:
	movzbl (%rbx),%edi
	addq $1,%rbx
	movb %dil,(%rsi)
	addq $1,%rsi
	jmp L222
L224:
	movb $0,(%rsi)
L233:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L235
L236:
	movzbl (%rbx),%esi
	addq $1,%rbx
	cmpl $58,%esi
	jnz L233
L235:
	movq _tzname+8(%rip),%rsi
	movzbl (%rsi),%esi
	cmpl $0,%esi
	jz L186
L242:
	movq $_tzdstdef,%rdi
	call _setdst
	movq %rbx,%rdi
	call _setdst
L186:
	popq %rbx
	popq %rbp
	ret
L247:
_gmtime:
L248:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L249:
	movq (%rdi),%rdi
	movq %rdi,%rsi
	cmpq $0,%rdi
	jge L253
L251:
	xorl %esi,%esi
L253:
	movl $86400,%edi
	movq %rsi,%rax
	cqto
	idivq %rdi
	movq %rax,%rdi
	movl %edi,%ebx
	movl $86400,%ecx
	movq %rsi,%rax
	cqto
	idivq %rcx
	movq %rdx,%rsi
	movl $3600,%ecx
	movq %rsi,%rax
	cqto
	idivq %rcx
	movl %eax,_tm+8(%rip)
	movl $3600,%ecx
	movq %rsi,%rax
	cqto
	idivq %rcx
	movq %rdx,%rsi
	movl $60,%ecx
	movq %rsi,%rax
	cqto
	idivq %rcx
	movl %eax,_tm+4(%rip)
	movl $60,%ecx
	movq %rsi,%rax
	cqto
	idivq %rcx
	movl %edx,_tm(%rip)
	leal 4(%rdi),%eax
	movl $7,%esi
	xorl %edx,%edx
	divl %esi
	movl %edx,%r12d
	movl $1970,%r13d
L255:
	movl %r13d,%edi
	call _isleap
	cmpl $0,%eax
	jz L259
L258:
	movl $366,%esi
	jmp L260
L259:
	movl $365,%esi
L260:
	cmpl %esi,%ebx
	jae L263
L257:
	leal -1900(%r13),%esi
	movl %esi,_tm+20(%rip)
	movl %ebx,_tm+28(%rip)
	movl %r12d,_tm+24(%rip)
	movl %r13d,%edi
	call _isleap
	cmpl $0,%eax
	jz L266
L265:
	movb $29,_dpm+1(%rip)
	jmp L267
L266:
	movb $28,_dpm+1(%rip)
L267:
	movq $_dpm,%rdi
L268:
	cmpq $_dpm+12,%rdi
	jae L271
L272:
	movzbl (%rdi),%esi
	cmpl %esi,%ebx
	jb L271
L269:
	subl %esi,%ebx
	addq $1,%rdi
	jmp L268
L271:
	subq $_dpm,%rdi
	movl %edi,_tm+16(%rip)
	leal 1(%rbx),%esi
	movl %esi,_tm+12(%rip)
	movq $_tm,%rax
L250:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L263:
	addl $1,%r13d
	subl %esi,%ebx
	jmp L255
L280:
_localtime:
L281:
	pushq %rbp
	movq %rsp,%rbp
	subq $8,%rsp
	pushq %rbx
	pushq %r12
L289:
	movq %rdi,%r12
	call _tzset
	movq (%r12),%rsi
	movq _timezone(%rip),%rdi
	subq %rdi,%rsi
	leaq -8(%rbp),%rbx
	movq %rsi,-8(%rbp)
	movq %rbx,%rdi
	call _gmtime
	call _isdaylight
	cmpl $0,%eax
	jz L285
L284:
	movq (%r12),%rsi
	movq _timezone(%rip),%rdi
	movl _dstadjust(%rip),%eax
	movslq %eax,%rax
	subq %rdi,%rsi
	addq %rax,%rsi
	movq %rsi,-8(%rbp)
	movq %rbx,%rdi
	call _gmtime
	movl $1,_tm+32(%rip)
	jmp L286
L285:
	movl $0,_tm+32(%rip)
L286:
	movq $_tm,%rax
L283:
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L291:
_asctime:
L292:
	pushq %rbp
	movq %rsp,%rbp
L293:
	movl 24(%rdi),%esi
	movl %esi,%eax
	cmpl $7,%esi
	jb L297
L295:
	xorl %eax,%eax
L297:
	imull $3,%eax
	movl %eax,%esi
	movzbl _daynames(%rsi),%eax
	movb %al,_timestr(%rip)
	movzbl _daynames+1(%rsi),%eax
	movb %al,_timestr+1(%rip)
	movzbl _daynames+2(%rsi),%esi
	movb %sil,_timestr+2(%rip)
	movb $32,_timestr+3(%rip)
	movl 16(%rdi),%esi
	movl %esi,%eax
	cmpl $12,%esi
	jb L300
L298:
	xorl %eax,%eax
L300:
	imull $3,%eax
	movl %eax,%esi
	movzbl _months(%rsi),%eax
	movb %al,_timestr+4(%rip)
	movzbl _months+1(%rsi),%eax
	movb %al,_timestr+5(%rip)
	movzbl _months+2(%rsi),%esi
	movb %sil,_timestr+6(%rip)
	movb $32,_timestr+7(%rip)
	movl 12(%rdi),%esi
	cmpl $10,%esi
	jb L302
L301:
	movl $10,%ecx
	movl %esi,%eax
	xorl %edx,%edx
	divl %ecx
	addl $48,%eax
	movb %al,_timestr+8(%rip)
	jmp L303
L302:
	movb $32,_timestr+8(%rip)
L303:
	movl $10,%ecx
	movl %esi,%eax
	xorl %edx,%edx
	divl %ecx
	leal 48(%rdx),%esi
	movb %sil,_timestr+9(%rip)
	movb $32,_timestr+10(%rip)
	movl 8(%rdi),%esi
	movl $10,%ecx
	movl %esi,%eax
	xorl %edx,%edx
	divl %ecx
	addl $48,%eax
	movb %al,_timestr+11(%rip)
	movl $10,%ecx
	movl %esi,%eax
	xorl %edx,%edx
	divl %ecx
	leal 48(%rdx),%esi
	movb %sil,_timestr+12(%rip)
	movb $58,_timestr+13(%rip)
	movl 4(%rdi),%esi
	movl $10,%ecx
	movl %esi,%eax
	xorl %edx,%edx
	divl %ecx
	addl $48,%eax
	movb %al,_timestr+14(%rip)
	movl $10,%ecx
	movl %esi,%eax
	xorl %edx,%edx
	divl %ecx
	leal 48(%rdx),%esi
	movb %sil,_timestr+15(%rip)
	movb $58,_timestr+16(%rip)
	movl (%rdi),%esi
	movl $10,%ecx
	movl %esi,%eax
	xorl %edx,%edx
	divl %ecx
	addl $48,%eax
	movb %al,_timestr+17(%rip)
	movl $10,%ecx
	movl %esi,%eax
	xorl %edx,%edx
	divl %ecx
	leal 48(%rdx),%esi
	movb %sil,_timestr+18(%rip)
	movb $32,_timestr+19(%rip)
	movl 20(%rdi),%esi
	addl $1900,%esi
	movl $1000,%edi
	movl %esi,%eax
	xorl %edx,%edx
	divl %edi
	leal 48(%rax),%edi
	movb %dil,_timestr+20(%rip)
	movl $1000,%edi
	movl %esi,%eax
	xorl %edx,%edx
	divl %edi
	movl %edx,%esi
	movl $100,%edi
	movl %esi,%eax
	xorl %edx,%edx
	divl %edi
	leal 48(%rax),%edi
	movb %dil,_timestr+21(%rip)
	movl $100,%edi
	movl %esi,%eax
	xorl %edx,%edx
	divl %edi
	movl %edx,%esi
	movl $10,%edi
	movl %esi,%eax
	xorl %edx,%edx
	divl %edi
	leal 48(%rax),%edi
	movb %dil,_timestr+22(%rip)
	movl $10,%edi
	movl %esi,%eax
	xorl %edx,%edx
	divl %edi
	leal 48(%rdx),%esi
	movb %sil,_timestr+23(%rip)
	movb $10,_timestr+24(%rip)
	movb $0,_timestr+25(%rip)
	movq $_timestr,%rax
L294:
	popq %rbp
	ret
L308:
_ctime:
L309:
	pushq %rbp
	movq %rsp,%rbp
L310:
	call _localtime
	movq %rax,%rdi
	call _asctime
L311:
	popq %rbp
	ret
L316:
L195:
	.byte 84,73,77,69,90,79,78,69
	.byte 0
.globl _tzset
.comm _timezone, 8, 8
.globl _timezone
.local _tm
.comm _tm, 36, 4
.globl _tzname
.globl _gmtime
.globl _localtime
.globl _ctime
.globl _asctime
.globl _getenv
.globl _atoi
