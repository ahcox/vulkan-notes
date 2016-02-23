
# Vulkan Articles and Videos

Time-ordered notes on Vulkan articles and videos, with the newest on top.

## Vulkan: Scaling to multiple threads video <a id="vulkan_Scaling_to_multiple_threads_video"></a>
> November 19th, 2015

* [Watch on YouTube](https://www.youtube.com/watch?v=s3ub6iVThro&t=2m35s)

### Notes on the video

#### Introduction

* Tobias, in IMG dev tech for 5 years.
* No new information in this talk.
* Subject will be about scaling to multiple threads.

#### GPU Waiting on the CPU?

* Talked last time about how API inefficiencies can slow things down on CPU.
* Another side to that...
* Multi-core CPUs
  * GLES is stuck with a single thread.
  * In a 4 core CPU, GLES only uses 1 at a time.
  * Stop-gap solutions for special cases like background loading resources and post processing effects dont really scale.

#### Making use of all the cores

* Why do you want to se all the cores?
  * If you can scale workloads over all cores, you can get more efficiency.
  * More efficient for power to run many cores at a low clock rate than one at a high clock rate.
      * Better battery life
      * Less heat.
  * Want linear progression of scaling without spending all time synching.
* **Gnome Demo**
  * Uses 2D uniform grid of square cells ("tiles").
  * Each cell is frustum culled and streamed as they come on-screen.
      * In GLES a cell just produces a batch of drawcalls.
      * In Vulkan they are individual command buffers.
  * In zoomed-out view a lot of these cells are created and destroyed each frame.
      * On order of four hundred thousand draw calls each frame.
        * ~250k of these are temporally coherent: reused from previous frame.
        * ~150k are brand new.
        * In GLES there is one core maxed-out.
        * When we speed up the movement of the camera so there is lots
          of dymanic buffer creation,
          if we only had one core,
          Vullkan would get stuck like GLES does maxing out one core.
          Because Vulkan lets command buffers be created on multiple cores,
          the FPS stays fairly high. 

#### Vulkan Mechanisms for scaling
* Nothing magical, just good solid engineering.
* Vulkan gives lots of mechanisms for scaling.
* Not doing magic in background on different threads.
* Vulkan puts you in charge of what threads do what work.
* **No Global State**
  * GLES required thread local storage.
     * Lookup in TLS on every function call. Lots of cache misses and indirections.
  * Vulkan requires getting an instance object to start to talk to it.
     * There is no implicit context: all state related to a call is in the arguments to it.
* **External Synchronization**
  * GLES makes all functions safe from all threads.
     * Adds lots of locks internally to make this work.
     * Locks can be expensive on some platforms/architectures even if not causing a wait. 
  * Vulkan does not lock on modification of internal state.
     * E.g., if you wanted to reset a command buffer, you could cause all sorts of problems if it was in use. Up to you to sync.
  * Upshot: no locking in driver, you rarely lock, better scaling, better perf.
* **Multithreaded command generation**
  * GLES:
     * No separation of work generation and submission.
     * As submission is on single thread, generation gets stuck on one thread too.
  * Vulkan:
     * Command buffers allow work generation on multiple threads ahead of time.
     * Submission on one thread is cheap.
     * Submission onto command queue: not accessible to multiple threads.
     * We expose as many queues as the GPU has frontends for work submission (in theory).
* **Command buffers require memory**
  * Involves dynamic allocation as buffer size is not known ahead.
  * Command buffer reset allows reuse of underlying memory.
     * If buffer grows second time used, there is still dynamic alloc.
  * During development of spec, after a while examples that did command buffer reset were using too much memory as the high water mark in each command buffer would rise individually.
     * Solution = command pools.
        * Group command buffers together.
        * That pool's memory will grow to max requirement  on one thread, but each individual buffer stays tight.

#### Conclusion
  * We thought hard about scaling.

#### Questions
  * Certification / Conformance:
    * Not ratified, no conformance tests, no spec, but plan to have tests.
    * Implementors cannot claim to support API without passing conformance testing.
    * There will hopefully bit quite a bit of that.


## Vulkan: High efficiency on mobile <a id="vulkan_High_efficiency_on_mobile_video"></a>
> November 5th, 2015

This is both a video webinar and an accompanying blog post.

* [https://www.youtube.com/watch?v=4exq7Pb0XRo](https://www.youtube.com/watch?v=4exq7Pb0XRo)
* [http://blog.imgtec.com/powervr/vulkan-high-efficiency-on-mobile](http://blog.imgtec.com/powervr/vulkan-high-efficiency-on-mobile)

### Notes on the video
#### Introduction
* No new information.
* CPU efficiency is the subject of the current talk, not GPU efficiency.
* Stuttering frame
  * Driver overhead often the cause. 
  * Low maximum number of draw calls in GLES.

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
      * Identical but using different uniform buffer bound for camera [_Clarification from Tobias: two identical command buffers are for double buffering: I guess while one is being slurped by the GPU, the other can have its camera matrix uniform updated safely. -Andy_]
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

[http://blog.imgtec.com/powervr/gnomes-per-second-in-vulkan-and-opengl-es](http://blog.imgtec.com/powervr/gnomes-per-second-in-vulkan-and-opengl-es)

This article describes Imagination Technology's PowerVR Vulkan demo
for SIGGRAPH 2015.
A [video](https://youtu.be/P_I8an8jXuM) exists that show the demo but with no explanation.

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
* [PowerVR Native SDK](https://github.com/powervr-graphics/Native_SDK)


_[Link here](http://ahcox.com/vulkan/vulkan-notes/)._
