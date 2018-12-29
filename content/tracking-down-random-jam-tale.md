----
Title: Tracking down random jam
Date: 2018-12-28 11:51
Modified: 2018-12-26 12:02
Category: tale
Tags: first, 386bsd, tale
Slug: tracking-down-random-jam-tale
Authors: William Jolitz
Summary: How a random hang in the 386bsd kernel was found, and ways to diagnose that were used. First tale.


Took me months to track down an irritating hang. The exact same kernel would work perfectly, then not. On certain hardware/VM's it would never fail, on others it would never work, and on some it would be "in between", sort of like a certain children's story (Goldilocks and the Three Bears).

If you want to avoid "TL;DR", put this bit of assembler in locore.s before swtch() relies on interrupts to wake up the next process:

<pre><code class="c">
	cli  /* begin reset 8259's */
	movb $0x11, %al
	outb %al, $0x20
	movb $0x20, %al
	outb %al, $0x21
	movb $0x04, %al
	outb %al, $0x21
	movb $0x01, %al
	outb %al, $0x21
	movb $0xff, %al
	outb %al, $0x21
	movb $0x02, %al
	outb %al, $0x20

	movb $0x11, %al
	outb %al, $0xa0
	movb $0x28, %al
	outb %al, $0xa1
	movb $0x02, %al
	outb %al, $0xa1
	movb $0x01, %al
	outb %al, $0xa1
	movb $0xff, %al
	outb %al, $0xa1
	movb $0x02, %al
	outb %al, $0xa0

	call _wdredo
	sti
</code></pre>

The quick answer was that the 8259 PIC's (X86 original interrupt control unit) state was getting trashed. So by finding this out and forcing reinitilization (plus forcing a "redo" of a potentially "lost" disk interrupt), the random hangs didn't happen. (Left out for brevity are the necessary 'write posts' and 'read delays' for marginal chipsets that required bus setup and hold "recovery" time to properly function).

The longer answer is the more interesting part - how do you track down such, where do you look, what are you up against, and how to get to the bottom of why this happens, so that this is correctly mitigated.

The 8259 is a cranky design - hidden registers you can't read only write, certain operations only in a specific order, many implementations with different flaws. With the earliest version of 386bsd, the only "recipe" that was stable was one that DOS (and later Windows) used, and often was suboptimal. Which didn't matter because early PC's functioned erratically due to cost evasion - "busy waiting" and no DMA common.

Didn't have hypervisors and X86 VM's then, or the memory to allow them (2MB 16Mhz ISA motherboard). But now that I do, checking the 8259's internal state showed it was set improperly when jammed, otherwise fine. So blindly resetting it when needed like above made it work.

Modifying the VM's code itself let me find where the program counter was when the change occurred. It was on recovery from stray interrupts, another misfeature of the 8259. If an interrupt is signalled but appears to be withdrawn (not there) when handled, it's considered a stray interrupt, which can happen before or after or not with a true handled interrupt (had they afforded a tiny stack in the 8259 to record state, but this was back when every single gate was minimized so as to increase the number of dice per wafer).

This explains why it was inconsistent in presentation - the hang/no hang was idiosyncratic, depending on timing. With the VM emulator, switching between cpu types would allow it to work, then not, because the effective "sampling" would be different, just like with different hardware chipsets/speed/delays.

How far down the "rathole" do you go with a open source system to support architecture flaws? Clearly enough to allow it to be useful, but at some point it obfuscates both clarity of the code as well as performance. Current Spectre/Meltdown/other bugs tickle this for modern systems to a much greater degree, as they reach into and distort things that are otherwise clear, and will have to be tediously phased out in time.

So why did this take months? Because it wasn't clear what the problem was at first. One casts about with many theories before becoming desperate enough to go "low level" for hours in debugging to capture the above mentioned event's "cold case".

With the corrected code in place, the above patch removed, things work as they should. Now one goes through all the collateral code/notes and sees how all handing of interrupts is being done, to spot other surprises.



