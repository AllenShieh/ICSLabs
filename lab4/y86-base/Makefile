CC=gcc
CFLAGS=-Wall -O2
YAS=./y86asm-base
YIS=./y86sim-base

APPFILES = abs-asum-cmov.sim abs-asum-jmp.sim asum.sim asumr.sim cjr.sim j-cc.sim poptest.sim pushquestion.sim pushtest.sim prog1.sim prog2.sim prog3.sim prog4.sim prog5.sim prog6.sim prog7.sim prog8.sim prog9.sim prog10.sim ret-hazard.sim

INSFILES = halt.sim nop.sim rrmovl.sim cmovle.sim cmovl.sim cmove.sim cmovne.sim cmovge.sim cmovg.sim irmovl.sim rmmovl.sim mrmovl.sim addl.sim subl.sim andl.sim xorl.sim jmp.sim jle.sim jl.sim je.sim jne.sim jge.sim jg.sim call.sim ret.sim pushl.sim popl.sim byte.sim word.sim long.sim pos.sim align.sim

all: sim

# These are implicit rules for making .bin and .yo files from .ys files.
# E.g., make sum.bin or make sum.yo
.SUFFIXES: .ys .sim
.ys.sim:
	$(YAS) $*.ys
	$(YIS) $*.bin > $*.sim

sim: $(INSFILES) $(APPFILES)

clean:
	rm -f *.bin *.sim *.sim.base *~
