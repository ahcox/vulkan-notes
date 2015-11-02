Time-ordered notes on Vulkan articles and videos, with the newest on top.

# Gnomes per second in Vulkan and OpenGL ES
>>> August 10th, 2015

http://blog.imgtec.com/powervr/gnomes-per-second-in-vulkan-and-opengl-es
This article describes Imagination Technology's PowerVR Vulkan demo
for SIGGRAPH 2015.
A video shows the demo but with no explanation: https://youtu.be/P_I8an8jXuM

## Summary
* Android on Intel Nexus Player, PowerVR G6430 GPU
* Raw, naive API calls, no instancing, as though all geometry, shader and textures were unique.
* Devides scene into a grid and builds a command buffer per cell.
* Reuses command buffers and regenerates them as they come on screen.
* Validation of command buffers is thus amortised over many frames of rendering.
* Nearing half a million draw calls per second without instancing.
* Mode generating command buffers in parallel across four cores
  * 80 comamnd buffers of 45 draw calls.
  * GPU bound so CPU could be down-clocked to save power.
* Render passes are used to exploint pixel local storage.

## Related Links
* PowerVR Native SDK: https://github.com/powervr-graphics/Native_SDK


_Copyright Andrew h. Cox, 2015. All rights reserved worldwide._
