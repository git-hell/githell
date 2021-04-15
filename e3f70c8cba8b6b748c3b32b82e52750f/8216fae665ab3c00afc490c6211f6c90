.text
_execvp:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	movq _environ(%rip),%rdx
	call _execvpe
L3:
	popq %rbp
	ret
L8:
.globl _execvpe
.globl _execvp
.globl _environ
