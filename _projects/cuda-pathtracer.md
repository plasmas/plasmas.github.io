---
layout: page
title: CUDA Pathtracer
description: Monte Carlo pathTracer written in CUDA, C++
img: /assets/img/projects/cuda-pathtracer/showcase.png
importance: 1
category: C++
related_publications: false
---

[Source](https://github.com/plasmas/Project3-CUDA-Path-Tracer)

_Tested on: Windows 11, i5-11600K @ 3.91GHz 32GB, RTX 4090 24GB (Personal Desktop)_

# Overview

<div align="center">
<img
  src='/assets/img/projects/cuda-pathtracer/showcase.png'
  style="width: 100%; height: auto;"
/>
<p>Showcase Scene</p>
</div>

A CUDA-based path tracer based on Monte Carlo sampling.

This path tracer harnesses the parallel processing power of modern NVIDIA GPUs using CUDA to simulate the transport of light in a 3D scene. The Monte Carlo approach allows for accurate simulations of complex lighting phenomena, such as global illumination, caustics, and multiple reflections.

---

## Feature List

- **Optimized Performance**: By leveraging the CUDA platform, the path tracer significantly boosts ray tracing speeds, distributing ray computations across numerous GPU cores. Additional optimizations include:

  - Stream compaction and material sorting to minimize warp divergence.
  - Caching primary rays and first intersections to eliminate redundant calculations.

- **Versatile Surface Shading**: The path tracer supports a diverse range of surfaces, encompassing:

  - Ideal diffusive (Lambertian) surfaces.
  - Perfect and imperfect specular surfaces.
  - Refractive materials.

- **Multiple Importance Sampling (MIS)**: For shading imperfect specular surfaces, MIS is judiciously utilized to mitigate fireflies and reduce variance.

- **Advanced Antialiasing**: Stochastic sampling with sub-pixel space jittering enhances antialiasing, ensuring smooth and polished edge renderings.

- **Authentic Depth-of-Field**: The path tracer offers a physically-based depth-of-field, allowing for user-specified aperture diameters and focal lengths, enhanced with random aperture jittering.

- **Specialized Camera Effects**:

  - **Fish-Eye Camera**: Create wide-angle, distorted visual effects characteristic of a fish-eye lens.
  - **Panorama Camera**: Capture sweeping, wide-angle views perfect for panoramic visuals.

- **Dynamic Motion Blur**: Introducing motion into static scenes, the `VELOC` tag lets users assign velocity vectors to objects. Combined with adjustable shutter time, this feature vividly captures the effects of motion blur.

---

## Path Tracing Pipeline

Our path tracer meticulously constructs a detailed rendering of a scene by iterating over rays projected from the camera, accumulating the final colors of each ray. The iterative process can be delineated into the following phases:

1. **Primary Ray Generation**:

   - In this phase, rays emanating from the camera are synthesized in a parallelized manner.
   - A caching system is in place that, when activated, preserves primary rays and their initial intersections for reuse in subsequent iterations.
   - In scenarios where caching is disabled, advanced features such as antialiasing, motion blur, and other intricate camera effects are operational.

2. **Intersection Computation**:

   - A parallelized computation is carried out to determine each ray's intersection against the entirety of the scene.
   - The system is equipped to handle some of primitives, like cubes and spheres.

3. **Shading & Ray Scattering**:

   - Drawing from each ray's properties and its intersection data, the ray's color is refined using a selection of BSDFs. Furthermore, a direction is charted for the succeeding ray in the next iteration.

4. **Partial Color Accumulation**:

   - Rays that either intersect with a light source or reach their bounce threshold have their colors aggregated.
   - Stream compaction is utilized to remove these rays post color collection.

5. **Termination Assessment**:
   - The iteration is concluded if no active rays remain. If active rays persist, the process returns to the second phase.

---

# Features

---

## 1. BSDFs for Basic Materials

The path tracer has support for basic BSDFs, including pure diffusive (Lambertian) surfaces, and pure specular (mirror-like) surfaces.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-pathtracer/pure_diffusive.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/projects/cuda-pathtracer/pure_specular.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: Pure Diffusive (5000 iters, 8 depth). Right: Pure Specular (5000 iters, 8 depth).
</div>

---

## 2. Active Path Stream Compaction

To minimize wrap divergence, terminated paths are removed at each bounce. This is done by first collect colors of terminated paths and then stream compaction which checks the remaining number of bounces for each ray. Stream compaction is implemented by `thrust::remove_if` function.

To quantify the benefits of stream compaction, the number of active paths are recorded in every bounce, within one iteration. The original cornell box (open) and a closed cornell box are used to compare the benefits between open systems and close systems.

<div align="center">
<img
  src='/assets/img/projects/cuda-pathtracer/stream_compaction.svg'
  style="width: 100%; height: auto;"
/>
<p>Stream Compaction Results</p>
</div>

The graphs shows that for both open and closed boxes, the number of remaining active paths decrease as the number of bounces increases, but the decrease is more obvious in the open box. This is expected as open box allows paths to escape the box to enter the void, but not for closed boxes. Open box also has a steady decrease of active paths, which is due to paths hitting light sources and terminating prematurely. Therefore we can say that for this specific path tracer, the benefits of stream compaction increases for more closed system and also fore systems with more light sources.

Compared to no stream compaction - launching 640,000 thread for 8 time, - stream compaction eliminates over 51.5% of number of threads launched in an open box, and only about 3.8% in a closed box. These threads correspond to inactive paths, and would cause an unnecessary overhead.

---

## 3. Material Sorting

Also to minimize wrap divergence, paths and their intersections are sorted acccording to the material type they hit in each iteration, so that each wrap will contain more threads that will access the same material info. `thust::sequence` is used to generate a sequence of indices, and then sorted with `thrust::sort` according to intersection material ids. Finally, paths and intersections are shuffled according to the sorted indices.

To understand the outcome of this optimization, we measure average time to generate a frame, using the original Cornell box with only 4 material types.

<div align="center">
<img
  src='/assets/img/projects/cuda-pathtracer/material_sorting.svg'
  style="width: 100%; height: auto;"
/>
<p>Material Sorting</p>
</div>

Without material sorting, it takes about 3.0ms per frame. And with sorting, the time comes to 7.7ms. It is obvious that for this specific scene, sorting materials actually has negative impact on performance. Even though sorting materials indeed enhance memory coarsening, the sorting overhead dwarfs the benefit of memory coarsening, especially when the number of materials is low.

---

## 4. Primary Ray & First Interaction Caching

In situations where there is no randomization in the generation of first ray and the objects are static, primary rays are always the same, as well as their first intersections with the scene. Therefore, we can cache the primary rays at the beginning of the first iteration, in the `pathtraceInit()` method. When the camera moves, the init function will be called again, and the cache will be updated.

<div align="center">
<img
  src='/assets/img/projects/cuda-pathtracer/caching.svg'
  style="width: 100%; height: auto;"
/>
<p>Primary Ray Caching</p>
</div>

From the testing result under different maximum bounce limit, it seems that the performance difference is a constant factor, and does not increase with the bounce limit. This is expected because the only computation eliminated is the first bounce of each iteration, and subsequent bounces are the same. On my GPU, the performance gain of caching is on average 0.28ms.

---

## 5. Refraction

The path tracer also supports ideal refraction, combined with Frensel's effect, which is emulated by Schlick's approximation.

<div align="center">
<img
  src='/assets/img/projects/cuda-pathtracer/refraction.png'
  style="width: 100%; height: auto;"
/>
<p>Refraction</p>
</div>

This implementation only accounts for ideal reflections and refraction, so images refracted within spheres are sharp and not blurred.

---

## 6. Imperfect Specular Surfaces with MIS

To achieve a good mixture of diffusion and specular shading, MIS weights are used to lower the variance and promote convergence. To account for imperfect specular shading, rays leaving the surface are sampled in a lobe centering around the ideal reflect direction. The results resembles a good Phong shading result.

<div align="center">
<img
  src='/assets/img/projects/cuda-pathtracer/imperfect_specular.png'
  style="width: 100%; height: auto;"
/>
<p>Imperfect Specular Reflection</p>
</div>

The use of MIS weights, which are computed by PDFs for both sampling techniques, lowers the variance and appearance of fireflies drastically. On the contrary, if a 50/50 chance of refraction/reflection chance is chosen, the variance will be so high that the final image will be full of fireflies.

---

## 7. Stochastic Antialiasing

To achieve smoother edges, antialiasing is accomplished by jittering primary rays within the range of a single pixel. Since randomization is introduced, primary ray caching is no longer feasible.

Here is a side-by-side comparison of edges:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-pathtracer/w_antialiasing.png" title="With Antialiasing" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-pathtracer/wo_antialiasing.png" title="Without Antialiasing" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: With Antialiasing (5000 iters, 8 depth). Right: Without Antialiasing (5000 iters, 8 depth).
</div>

---

## 8. Motion Blur

In the scene file, a `VELOC` `vec3` tag can be added to each object to specify the velocity vector of its movement.

To achieve a sense of movement, a time is randomly chosen from 0 to a specified shutter time, and objects will be updated to their respective position at that specific time, achieving a sampling at that time.

To further show the acceleration of the object, time is sampled more often at the end of the shutter time, so the object is more solid at the end position of its movement.

<div align="center">
<img
  src='/assets/img/projects/cuda-pathtracer/motion_blue.png'
  style="width: 100%; height: auto;"
/>
<p>Motion Blur</p>
</div>

The sphere is specified a velocity of `[4, 0, 0]`, therefore moving to the right. It is obvious that the sphere is more solid on the right, which is the endpoint of its movement.

---

## 9. Physically-Based Depth-of-Field

Rather than using a traditional pin-hole camera, the origin of primary rays are instead generated within a small aperture space. The aperture diameter as well as the focal distance can then be specified.

Focal distance is used to control the focal point and which objects should be sharp and clear:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-pathtracer/focal_left.png" title="Focal Length 12.38" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-pathtracer/focal_right.png" title="Focal Length 8.08" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: Focal Length 12.38. Right: Focal Length 8.08.
</div>

Aperture diameter is used to control the blur degree. Larger aperture means more blur at the same distance from the focal plain:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-pathtracer/aperture_1.png" title="Aperture Diameter 1.0" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-pathtracer/aperture_5.png" title="Aperture Diameter 5.0" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: Aperture Diameter 1.0. Right: Aperture Diameter 5.0.
</div>

---

## 10. Alternative Camera Types

Fish-eye camera and Panorama camera are also available for this path tracer, by blending primary rays in a spherical or cylindrical coordination.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-pathtracer/fisheye.png" title="Fish Eye Camera" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-pathtracer/panorama.png" title="Panorama Camera" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: Fish Eye Camera. Right: Panorama Camera.
</div>

---

# Reference

1. [GPU Gems 3: Importance-based Sampling](https://developer.nvidia.com/gpugems/gpugems3/part-iii-rendering/chapter-20-gpu-based-importance-sampling)
