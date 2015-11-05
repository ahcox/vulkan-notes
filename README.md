
# Vulkan Articles and Videos

Time-ordered notes on Vulkan articles and videos, with the newest on top.

## Vulkan: High efficiency on mobile <a id="vulkan_High_efficiency_on_mobile_video"></a>
> November 5th, 2015

This is both a video webinar and an accompanying blog post. 
* https://www.youtube.com/watch?v=4exq7Pb0XRo
* http://blog.imgtec.com/powervr/vulkan-high-efficiency-on-mobile

### Notes on the video
#### Intro
* No new information.
* CPU efficiency is the subject of the current talk, not GPU efficiency.
* Sputtering frame
  * Driver overhead often the cause. 
  * Low maximum nmber of draw calls in GLES.

#### Thermal Budgets on a SOC
* Desktop CPUs and get hot and need cooling, but mobile SOCs cannot get hot.
* Vulkan reduces CPU workloads, even for homescreens and other simple apps.
* Less CPU gives headroom for more GPU work in a given power budget.

#### Power and Battery Life
* Mobiles need batteries.
* More work = more current.
* Less work = longer battery life.

#### How Vulkan Achieves Better Efficiency
* Not magic bullet. Naive port can be slower than DX11 / OpenGL.
* [Aras Pranckevičius](http://aras-p.info/blog/) (Unity) found it hard to add next gen API support [_I have heard this too from DX12 game ports - Andy_].
* Rewards can be great.
* No error checking or validation in Vulkan.
  * Errors are only useful to debug.
  * Debug happens ahead of time.
  * Old way in GL is like shipping a debug build of your game.
  * Vulkan has a concept of API layers.   
  * Tooling layers handle all error checking on top of core API.
  * Because a layer doesn't have to happen in a release build, it can at the limit be incredibly slow, equivalent to running an app in Valgrind [_Say for an overnight stress test - Andy_].
* Avoids Hazard tracking and synchronization
  * In the old way, the API view of an object can be modified while the GPU is using it, and driver allocates and copies, and fences and syncs to make that work. This overhead is always present to some extent even if the app is not modifying in-use resources, just in case it is.
  * In the new Vulkan way, it is all up to the app to track hazards.
* **Pipeline Objects**
  * Current GL way: blending state of GL and other _state_ gets turned into shader code which is patched into user shaders at runtime.
    * Driver uses extra memory to cache compiled combinations of code generated for this supposedly fixed state and user shaders.
  * New Vulkan way bakes almost every piece of state up-front in pipeline state object.
    * So drivers can compile that down to the most efficient form possible ahead of time.
  * Curent GLES apps often have state objects.
    * Most apps should be able to supply this ahead of time instead of in the middle of a render.
* **Command Buffer** reuse
  * Commands recorded ahead and driver gets to analyse that buffer and optimise it once.  
  * State can't change, resources (textures, uniforms, ...) can.
  * Demo (interior scene from GDC)
    * Only 2 command buffers used (+ one for UI maybe).
      * Identical but using different uniform buffer bound for camera [_Why two identical? Is this for double buffering? -Andy_]
* **Multithreading**
  * Multiple cores at low frequency more efficient than one fast core. Vulkan enables this.
  * Vk has better scaling than GLES: allows multiple cores to do work on API.
  * In Gnomes demo.
      * GLES maxing one core out, and bouncing thread between two cores to spread the heat.
      * Vulkan speading the laod over all cores.
  * More in next talk on threading.
* **GPU Efficiency**
  * Vulkan is more explicit.
  * Vulkan allows earlier optimisation.
  * Final talk will go into mapping Vulkan concepts to Imagination hardware.

#### Conclusion
  * Significant overhead reduction.
  * Gnome demo reduced workload to 33%, and total system load went down to 50%.
  * Cache hits and misses and other details also thought about by Khronos WG when designing Vulkan.
  * Difficult to get going (Aras did not have a good time).
  * Rewarding in the end though and you will see gains. 
  * Questions via [twitter](https://twitter.com/tobskihectov) welcome.

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
[Disclosure of RenderPass: [Graham Sellers, AMD](https://onedrive.live.com/redir?resid=2053FCB7E3D99729!124&authkey=!AN2LmaEK6rJB4yQ&ithint=file%2cpptx); [Piers Daniell, NVIDIA](https://www.youtube.com/watch?v=NqensKmmRfE&t=29m15s)]

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

    // Setup framebuffer attachments with a DONT_BACK_THIS_WITH_MEMORY flag set on some.
    // ...
    VkRenderPassBegin(cmdBuffer);
    // Draw things
    // ...
    VkRenderPassEnd(cmdBuffer);
    // Driver goes off to bin some trianges and draw them.
 
I am looking forward to seeing some good information come out about this design,
and finding out what I am missing.
Hopefully it all hangs together and makes perfect sense.


_[Link here](http://ahcox.com/vulkan/vulkan-notes/)._
