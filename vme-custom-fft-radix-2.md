# A custom implementation of an FFT Radix-2 DIF using the PSP Virtual Mobile Engine

A shape, or a game element that reacts to sound, a boss, an NPC, or entire gameplay that adapts to the way you play, the resolution of certain puzzles, the automatic adjustment of the sound envelope of background music, voice detection, adaptive-resolution raycasting, etc.

The list could go on much longer, and it rests on a single principle: extracting exploitable frequency characteristics from a signal in order to dynamically adapt our algorithms. FFTs (Fast Fourier Transformations).

But here's the thing, FFTs can be expensive on our small machines. However, on our PSPs, it is now possible to use a compute unit that until recently was a complete black box, and it can help us with that.

This unit performs operations across multiple data streams, making it competitive with specialized SIMD processors. The difference from a classic SIMD, however, lies in the nature of the parallelism. On a SIMD, the same instructions are executed in parallel across multiple pieces of data. Here, each PE (processing element) handles a stream in which the data is processed in a chain, one after another, somewhat like a compute pipeline dedicated to that stream, and it does so quickly. Having multiple PEs then makes it possible to achieve parallelism across the computation of different streams, with each PE processing its own stream in parallel with the others.

This unit is the Virtual Mobile Engine, and a first custom implementation of a Radix-2 DIF FFT has been built on it.

## Among its main parts, we find:

**The handling of reordering according to the stride of each butterfly.** This is managed via the local DMAC, which has an AGU extension with two counters for handling sliding windows according to a given stride. An external counter jumps according to the stride to position the window, while an internal counter reads within that window. It's this combination that produces the correct reordering. It also has the ability to loop in ring buffer mode according to a defined maximum size.

**At each stage/butterfly**, the high and low branches are first streamed to the VME through a first context and go through the butterfly's first two operations, ADD and SUB.

Then, at the output of this first context, the low branch's data is streamed to a second context to be computed against the twiddles. This second context also handles reading the twiddles corresponding to the current stage, since the VME too has, internally, AGUs that allow for the readjustment and organization of data. The twiddles go through a complex compute pipeline (real, imaginary) and produce four distinct streams. It is also within this same second context that, at the end of processing, the four streams are computed once more: an ADD on the real parts and a SUB on the imaginary parts, of the low branch's output.

The data coming out of each butterfly stage passes back through the DMAC before being transferred to the next stage. It's a bit technical, I'll grant you that, and even more so as a whole. I'll leave you the GitHub link containing this first custom implementation: <https://github.com/mcidclan/psp-virtual-mobile-engine-ext/tree/main/fft-dif-16-point-radix-2>

## Applications

FFTs can be used in pattern analysis, in certain parts of terrain generation, and many other cases, and their cost becomes reasonable thanks to this type of hardware running in parallel with the main processor.

## About the Virtual Mobile Engine

The Virtual Mobile Engine is accessible from the Media Engine, which acts as the host processor. It is a CGRA-type compute unit. The idea behind these code samples is to shed more light on this hardware, to understand its capabilities, its limits, and the opportunities it offers us. It also serves as documentation and as technical reference material for preservation.

Additionally to reversing part of the Media Engine's core functions, what I do, and what I've done to obtain this understanding of the hardware, is to send data to its registers and observe how they respond and what they return. I take notes, and after a certain amount of testing and experimentation, I start to identify reproducible behavior and see new paths to explore. I establish hypotheses and use them to figure out what is still missing and/or what was initially misinterpreted.

After a while, and much trial and error, an understanding of what the hardware is capable of starts to take shape.

*m-c/d*

