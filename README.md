
# Vulkan Articles and Videos

Time-ordered notes on Vulkan articles and videos, with the newest on top.

## Gnomes per second in Vulkan and OpenGL ES
> August 10th, 2015

http://blog.imgtec.com/powervr/gnomes-per-second-in-vulkan-and-opengl-es

This article describes Imagination Technology's PowerVR Vulkan demo
for SIGGRAPH 2015.
A video shows the demo but with no explanation: https://youtu.be/P_I8an8jXuM

### Summary
* Android on Intel Nexus Player, PowerVR G6430 GPU
* Raw, naive API calls, no instancing, as though all geometry, shader and textures were unique.
* Divides scene into a uniform grid and builds a command buffer per cell.
* Reuses command buffers, destroying them as the corresponding gid cells exit the frustum and regenerating them as they come back on screen.
* Validation of command buffers is thus amortised over many frames of rendering.
* Nearing half a million draw calls per second without instancing.
* Mode generating command buffers in parallel across four cores
  * 80 command buffers of 45 draw calls each.
  * GPU bound so CPU could be down-clocked to save power.
* Vulkan RenderPass objects are used to exploit pixel local storage.

### Related Links
* PowerVR Native SDK: https://github.com/powervr-graphics/Native_SDK

# Notes on API as it has Been Publicly Disclosed

## RenderPass
[ [Disclosure of RenderPass](https://onedrive.live.com/redir?resid=2053FCB7E3D99729!124&authkey=!AN2LmaEK6rJB4yQ&ithint=file%2cpptx) ]

I don't think the real motivation for this feature has been articulated yet in any of the videos, blogs, and slide decks made available [2015-11-02].
The business of using on-chip memory for gbuffers could just as easily have been achieved with a flag on the object representing the framebuffer attachments saying `DONT_BACK_THIS_WITH_MEMORY`.
A sequence of draw commands would then have had access to this on-chip memory without any work by the app. 
An advantage I don't remember being mentioned but can see,
is that it allows a tile-based GPU's driver to start issuing work to the GPU at the earliest possible moment, without having to wait until it has seen a whole frame of draw commands.
By using a RenderPass to tell the driver the scope of all work that will touch a given framebuffer,
the driver has a natural trigger for the jobs like binning that a tiler needs to do.
This _could_ lower latency to photons, and _could_ reduce the chance of the GPU idling,
but then again, these advantages could just as easily have been achieved with a pair of commands
to start and end rendering to a target explicitly, without all the subpass business.

    VkRenderPassBegin(cmdBuffer);
    // Draw things
    VkRenderPassEnd(cmdBuffer);
    // Driver goes off to bin some trianges and draw them.
 
I am looking forward to seeing some good information come out about this design,
and finding out what I am missing.
Hopefully it all hangs together and makes perfect sense.


_[Link here](http://ahcox.com/vulkan/vulkan-notes/)._
