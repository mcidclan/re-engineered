In this article, we will see how the Virtual Mobile Engine (VME) can be used to generate moving particles on its own, directly to the frame buffer.  

![Screenshot](./vme-particles.png)

This experiment uses the new VME2 Next-Level Toolkit, which is available here:  
[VME2 Next-Level Toolkit](https://github.com/Th6uD1nk/vme2-next-level)

The toolkit is a project that introduces a language specially designed to describe and integrate VME kernels into your .C or .CPP statements, built to produce the code executed on the Media Engine.

Indeed, the VME is hosted by the Media Engine, which in our particular case is reduced to managing the load and transfer between the memory and the VME. All the logic is left to the VME itself. The main CPU, on its side, just sets up the frame buffer in a shared memory area and runs a loop to keep the application running until HOME is pressed.

This was done to demonstrate that the VME can serve as the piece of hardware acting as the main processing unit in a homebrew project.

The idea here is to draw and move particles over a buffer to be displayed on the screen. 16 particles are drawn, each one moving along its own straight line at its own speed. When a particle leaves the screen, it comes back at the start of the same line and travels along it again.

To make this possible, the VME has to rely on several kernels that describe the logic it has to process. Together, those kernels produce a pipeline where each stage has its own role and contributes to the construction of the final image to be displayed.

Here is a simplified description of the pipeline by kernel name and role:
- 'Advance' increments the phase of each particle.
- 'Place' calculates the position of each particle.
- 'ProfileX' determines how much each particle contributes to the image horizontally (across the columns).
- 'MakeDy' calculates how much each particle contributes to the image vertically. Together with 'ProfileX', it determines how each particle's colour spreads across the image based on its position.
- 'LoadResident' loads the previous data of the particles into the scratchpad, split across 4 main areas (top0 to top3), with 4 particles in each area.
- Renderers: 'RenderFirst', 'RenderMiddle' and 'RenderLast'. These final three kernels calculate the brightness contribution of each particle and add it to the existing pixel values. They process two rows in parallel, accumulating the contributions of all 16 particles before moving on to the next two rows. (Note that this part could be improved.)

Here is a demonstration of the result:  
[![Particles Demo](https://img.youtube.com/vi/Ufj0jaM75Go/hqdefault.jpg)](https://youtu.be/Ufj0jaM75Go)


For more technical information, you can take a look at the source and refer to the README if you wish: [LINK]
[VMEG Particles](https://github.com/Th6uD1nk/vmeg-samples/tree/main/vmegc-rgb-particles)

The possibilities are enormous, since a kernel executed on the VME doesn't necessarily need to be waited on by the ME, which gives us more units to offload tasks to in our homebrew projects.

Thanks for reading,

m-c/d
