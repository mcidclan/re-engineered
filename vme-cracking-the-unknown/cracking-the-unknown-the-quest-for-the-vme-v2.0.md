<sub>v2.0</sub>

# Cracking the Unknown: The Quest for the Virtual Mobile Engine (VME)

Hi, I'm m-cid, a hobbyist developer from France who enjoys giving old hardware like the PSP a second life through software by pushing it beyond what it was thought capable of.

In January 2026 I started the 'psp-media-engine-cracking-the-unknown' project where the main purpose was to figure out how the Sony Virtual Mobile Engine 2 works. It is a piece of hardware integrated into the PSP, known to be the core unit for processing audio. However, basically no public technical resources existed on that subject. So, I first had to go through reverse engineering to list the available hardware registers related to it. Then, most of the work was done through trial and error relying on feedback from the device itself. In that Quest, I had to formulate many hypotheses in order to validate or invalidate them, then build new hypotheses until things became clearer.

See the 'psp-media-engine-cracking-the-unknown' project on GitHub for a glimpse into the investigation's history.

Apparently, Sony had released a bit of information about this hardware's specifications which let us understand that the VME was a sort of reconfigurable vector engine chip, coarse grain heterogeneous type capable of being reconfigured on the fly. By searching around, I found that this was actually quite close to what a small academic CGRA could have been. So, I decided to explore that possibility and dig into how CGRAs work and are structured in theory.

After some dump sessions I discovered a sort of bitstream with recognizable patterns present in the Media Engine's eDRAM. At the time I had no idea what it really was or why it was structured that way, especially since most of what I extracted from the eDRAM turned out to be incomplete and were more about default configurations for the hardware.

Over time I progressively managed to get feedback from the VME with data being processed over the scratchpad shared between it and its host processor, the Media Engine. The first feedback actually came during a session where I was sending random data to it: the VME started running from a bad configuration where it never returned correctly. This feedback appeared since I was bypassing the finish sync status checks of the VME, which let the Media Engine continue running in parallel with it. It wasn't obvious to get there since everything was a little bit foggy in terms of understanding, but it was fun. Progress was mostly a matter of time as I slowly got familiar with the hardware, something that, like anything else, you end up understanding better the more time and energy you invest in it.

In the end, other elements such as the Address Generator Units, the Interconnect, the Functional Units, Multiplexers, and the Context Memory Config, which I first called bitstream, along with the notion of the datapath itself, started to be confirmed as entities close to what I read and understood from CGRAs and became clearer to me. The takeaway is that understanding how a basic VME pipeline works, based on what I discovered and with the understanding I gathered, isn't that hard in the end.

## Beyond Audio: A Quest for the VME's Additional Purposes

Some people might have thought the Virtual Mobile Engine was designed just for audio processing, but that appeared not to be true, at least for the VME2, which is the one integrated into the PSPs. Indeed, this version of the VME clearly has functional opcodes designed for other things like decoding and image processing. For example, we have "sum of absolute differences," which enables motion estimation, "XOR parity," which is useful for integrity checks, "64-bit binary rotation," and "absolute difference / absolute sum," which can be used for edge detection.

Those are operations that do not appear to have a real direct purpose in pure audio processing and could serve totally different purposes. Additionally, the VME obviously has generic ALUs (including logical operations) which allow us to process any sort of data.

The fact that the VME appears to not only be designed for audio and could be used for other processing tasks is not a hypothetical claim. Indeed, sample code making use of it for fields like image processing, AI neural networks, vector transformations, and FFT implementation, which are used in many fields, has already seen the light of day. If you want to take advantage of it in your homebrew games, apps, or game ports, you just need to keep in mind that it does have its own constraints and limitations.

The Media Engine communicates with the VME through 4 known ways.

The main one is via a memory range dedicated to the memory context configuration, accessible through a number of hardware registers exposed as MMIO. The known range is from 0x440f8000 to 0x440f81a4, inclusive. This interface exposes the memory context configuration and allows the host CPU to dynamically update the VME datapath.

On the other hand, there is a second way to send a configuration to the VME, this time by fully setting up a new memory context saved into RAM or local eDRAM, which will then be sent via the dedicated DMAC to the memory context configuration of the VME.

We then have other paths of communication with the VME, this time for data rather than configuration. This is done through a scratchpad shared between the ME and the VME. It has the particularity that each data word is a 24-bit two's complement value packed into a 32-bit word, where the 8 higher bits are sign-extended from bit 23. The main one is through the MMIO interface of the scratchpad, based at 0x440ff000, which allows the host CPU to directly send or retrieve data to and from the VME.

Transfers to and from the scratchpad can also be done using the local DMAC, where the external memory can be either RAM or local ME eDRAM. The DMAC additionally offers the ability to unpack 32-bit words from host memory into 16-bit significant values for the VME, and conversely pack 16-bit values back into 32-bit words when returning data to host memory.

It is worth noting that the scratchpad format puts a real constraint to consider when implementing a solution to be handled by the VME. Depending on what you want to achieve, you'll have to pick the right input and output format, since the VME operates in fixed point. That said, the range of applications is huge, and you don't have to limit yourself to classical implementations. You don't have to try to directly port your scalar or SIMD implementation onto it. Instead, you need to think about how the VME could handle it on its own.

## A Multi-Stream Engine: The Quest for Parallelism

The VME works as a multi-stream engine where each conceptual Processing Element (PE) is composed of 2 physical functional units (FUs) that can each have their own Address Generation Unit (AGU) and can run in series or in parallel. This means we can have 8 functional units processing 8 pairs of streams in series or in parallel. Each of them has a multiplexer with two selectors that route the back and front buffers to be processed together according to the operation set on the related FU.

According to early specifications that we could find on the internet, the VME is clocked at a max clock frequency of 166MHz with a throughput of 5 Giga Ops/sec, giving us a number of around 30 operations per cycle, which is non-negligible. Thanks to the VME's capabilities, the Media Engine now has more room to handle tasks in parallel with the PSP's main CPU.

We now have some useful sample code that can help with implementing VME pipelines. As proof of this, these samples have already been used as reference to integrate the VME into homebrew projects, such as ports of N64 games like Ocarina of Time and, more recently, Star Fox 64, for example. So I invite you to take a look at them by following the GitHub link below:

https://github.com/mcidclan/psp-virtual-mobile-engine-ext

You can even think about joining us with new ideas to integrate the VME into homebrew projects.

Additionally, to give you more visibility, here's a simplified view of how data moves through the VME datapath, from host memory to the functional units and back:

```
                    HOST MEMORY
                   /            \
                  /              \
           via DMAC          via MMIO
                  \              /
                   \            /
              CONTEXT MEMORY CONFIG
                        |
                        v
           ADDRESS GENERATOR + DATAPATH
                        |
                        v
        +-------------------------------+
        |  front 0   front 1   ...      |
        |     \         \               |
        |      \         \              |
        |  back 0   back 1   ...        |
        |     \         \               |
        |      \         \              |
        |      +-----------+            |
        |      |    MUX    |  ...       |
        |      +-----------+            |
        |            |                  |
        |            v                  |
        |      +----------+             |
        |      |    FU    |  ...        |
        |      |   Acc    |             |
        |      +----------+             |
        |            |                  |
        +------------|------------------+
                     v
              staging result 0, 1, ...
                     |
                     v
                STAGING MEMORY
                     ^
                     |
                     v
                  SCRATCHPAD
                     |
                     v
                  HOST MEMORY
```

Thanks for reading, m-cid.
