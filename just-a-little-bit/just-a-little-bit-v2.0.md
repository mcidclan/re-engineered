# Just a Little "Bit"

*version 2.0 - 21 Nov 2026*

## Hey Hi! I'm Not AI

Dear readers, I have to speak to you in a specific style, so as not to be confused with an AI. Some will understand, others will ask: why? It is just me, thinking out loud under the stars.

Let me try:

If we're taking only one bit, or just a little bit from the vastness of the sky, do you think we'll be able to encode, program or write anything with it? If not, why not use a wider range of bits?

Hmm, that will be an exercise in style.

## Streaming Bits

We actually need some bytes... or words? Imagine them flowing one by one. Try to visualize this: one stream, two streams! Side by side, moving to supply an engine capable of blending them. Do you know what the VME is? I already told you about it, don't you... doesn't it remind you?

If not, please take a look at [The Quest for the VME](https://github.com/mcidclan/re-engineered/blob/main/vme-cracking-the-unknown/cracking-the-unknown-the-quest-for-the-vme-v2.0.md)  

So, can we do it with one bit? Maybe not. Two bits? Maybe not. How many? Four bits? A nibble, not a byte. But we could demonstrate an addition relying only on logic. That should be fine. Addition? No, wait, we'll do better: a multiplication. Let's get into the action, making exotic operations, on this reconfigurable unit, now usable after cracking the unknown.

The words are many, waiting for the word to get through the PEs. Those Processing Elements, which will be named P0, P1, P2, P3... Some would ask what I am talking about, others would say, please tell them about it! In any case, I hope y'all are ready to enter into a new experiment. Because, we will now inspired by FPGAs, use the VME to describe a circuit using only its logical capabilities.

Wait wait wait!

That's... a bit too much. But is that okay? I mean, am I enough of a non-AI? Or do I sound a little bit high?

## Designing the Bits

Back to the subject.

There is that bit, an annoying bit which should be managed. What do we call it again? Ah yes, the carry bit! Not that one? Then Carry, out! The other one, and carry-in? Breathe in. So no. Yes, but no native arithmetic operations, even though the VME supports most of them. It cannot divide? Shift it to a Q-machine and get a different vibe, or if you can, just shift. But for now, it's time for a challenging exercise in which we'll care only about a few logicals.

Just taking a small pause, catching my breath... before getting back into the loop, because we need a loop... yeah we need a loop!

Not yet!

Hold your pens and take some notes. Indeed, you should know that VMEG has a doc: something we can refer to when we're blocked. And... it does explain, how to loop through and around a block. I think I just dropped a quote before the boat was set afloat. Hope this is not a flop, at least you can't call it a slop!

To clarify, VMEG is a language that makes VME kernels easier to engage. More details on it further down the page. The VME has 4 structural Processing Elements, each containing 2 physical Functional Units. It would suit our use case perfectly well, indeed. In fact, it is exactly what we need, but I realize this will be way too long. So... hey! Forget about the circuit and delay the multiplication!

Take a look at what we're focusing on:

```
  0011
+ 1010
= 1101
```

You can't see it? Please enter and take a seat. If we keep our eyes on the board, we do see the streams, the one on the top, and the other below it. And then those 4 bits emerge from the Algebraic. The XOR is obvious, until Carry comes in... falling from nowhere, or out of the XORing. Now, we do need a thing to bring this annoying bit in!

But again, we're not using the Addition because it would make things way less challenging. AI would use addition, because it doesn't feel what a challenge is. We need to AND the operands, and SHIFT to carry out. Keep the carry-in bits in hand, XOR again, and repeat until we get zero at the end.

Here is the main stage of the pipeline:

```
        0011
    XOR 1010
      = 1001
```

```
        0011
    AND 1010
      = 0010
    SHL 0010
      = 0100
```

Then we need to repeat it until the AND gives naught, the XOR, for its part, will give what we sought. We're on 4 bits, so no need to fret, 4 rounds at most and we're all set! 

VMEGC is part of the VME2 Next-Level Toolkit. It allows us to easily declare and compile kernels to be processed over the unit. The toolkit also provides an emulator! You don't necessarily need a PSP to become an experimenter or investigator... so, stay away from the door! Take a chair, or a place on the floor... you can just write the kernel, and ask the emulator to process it with its operators.

As an example, this is how our pipeline could have been represented:

```
.kernel logicAdd direct {
  Staging s0, s1, s2, s3, s4, s5;
  .p0 {
    s0 = top0 ^ top1;
    s1 = (top0 & top1) << 1;
  }
  .p1 {
    s2 = s0 ^ s1;
    s3 = (s0 & s1) << 1;
  }
  .p2 {
    s4 = s2 ^ s3;
    s5 = (s2 & s3) << 1;
  }
  .p3 {
    .spad = s4 ^ s5;
  }
}
```

## Crafting with Bits

This is called VMEG, the same one we spoke about too early. It is then transpiled into C or Cpp, by... VMEGC. Amazing, isn't it? I wanted you to open your editors, not... 'Claude as editor', to write the code above, but...

Unfortunately, the VME doesn't shift over pure logical units, whatever the previous example said. We would need an operator in between, or a post process before the stream is seen, so that the result can show its head, but we'll use a VMEG flow instead. A flow replaces the PEs through a VMEG kernel when using pipeline mode. This mode automatically manages the PEs behind, and allows us to chain multiple operations one after the other.

VMEG also provides a for loop that folds expressions, the transpiler then unfolds, to rebuild the right instructions:

```
.kernel logicAdd pipeline {

  Int count = 1;
  
  .flow {
    .count = count - 1;
    Slot carry[4];
    
    .slot = top0 ^ top1;
    carry[0] = (top0 & top1) << 1;

    for i in 0..2 {
      carry[i + 1] = (slot & carry[i]) << 1;
      .slot = slot ^ carry[i];
    }

    .slot = slot ^ carry[3];
  }
}

```

This code shows how flexibility was brought to VMEGC, which here takes advantage of the VME by automatically organizing communication between the PEs and the functional units behind the scenes.

You'll tell me, okay, we're using the VME, but is that all? A logical addition, and what else after all? Actually it brings new ideas to explore, and I think Homebrew is not just about porting games, or writing another music player, and whatever. But it's also about having fun, and giving life to things that others might not imagine possible.

Do I take all of this too seriously? Hmm... maybe. Just a little bit.

## Thinkering a Little Bit

This article is a sort of techno-poetic essay, a way to say that most of the contributors who remain on the scene give their spare time, driven by their motivation and the fun they get out of it, or so I hope. And that even without AI, a human can write something that gets mistaken for it, because the line is thin these days... don't you see? But let me clarify that I'm not fighting AI, and I'm not fighting those who are against it either. I think that as long as the human behind the work can still express themselves, while letting AI carry the tasks we have mastered or that save time... we can grow and learn more as humans.

m-cid.

Oh yeah, thanks for reading! And I forgot to say that the toolkit has recently been updated!

- We now have 8-PE kernel support, using a hybrid approach that relies on some of the hardware particularities. Two groups of 4 parallel PEs are chained in series, with the inputs and outputs of all 8 PEs still reserved independently. This chaining takes advantage of the config memory shadow register and of the interconnect splitting mode.
- Additionally, we now have the new kernel type called "pipeline", which takes streams as input and processes them from top to bottom. It is what we used for writing our previous Kernel, and here, as you can see, there is no concept of PEs. The AGUs are automatically configured by VMEG. The pipeline accepts a flow of operations going downward with practically no limit.

I invite you to take a look at the full sample code related to this article, as well as the vme2-next-level toolkit project and the cracking-the-unknown project to get a broader view of the subject:

[Sample Code + Toolkit](https://github.com/Th6uD1nk/vme2-next-level/tree/main/samples/c/logic-add)  
[Cracking the Unknown](https://github.com/mcidclan/psp-media-engine-cracking-the-unknown)  

The end.
