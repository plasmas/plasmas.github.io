---
layout: page
title: CUDA Denoiser
description: Denoiser for Monte Carlo Path Tracer using CUDA, C++
img: /assets/img/projects/cuda-denoiser/thumbnail.png
importance: 1
category: C++
related_publications: false
---

[Source](https://github.com/plasmas/Project4-CUDA-Denoiser)

_Tested on: Windows 11, i5-11600K @ 3.91GHz 32GB, RTX 4090 24GB (Personal Desktop)_

---

# Overview

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/original.png" title="Original (10 iters, 8 depth)" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/denoised_65x65.png" title="Denoised (filter size 65x65)" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Original (10 iters, 8 depth). Right: Denoised (filter size 65x65).
</div>

A CUDA-based A-Trous denoiser for path tracers.

In real-time rendering, even modern machines don't have the capability to perform an sufficient number of path tracing iterations to achieve a photo-realistic image. Within a few iterations, the image generated can have a lot of noise, resulting in high variance. Therefore, denoising is need to smooth the image. A naive Gaussian blur can indeed smooth the image, but will blur across edges. Also, using a large Gaussian kernel is expensive when performing convolution. Therefore, we use the A-Trous wavelet filter with an edge-stopping function.

This denoiser implements the method by [Dammertz et al.](https://jo.dreggn.org/home/2010_atrous.pdf), with a few tweaks.

---

# Performance Analysis

## Filter Sizes

The time cost of denoising is dependent on how many iterations are performed. We record the time cost by computing the denoised image, at different number of iterations requested.

To unify the benchmarks, without explicit mention, all tests are performed using [Cornell Ceiling Light](https://github.com/plasmas/Project4-CUDA-Denoiser/tree/base-code/scenes/cornell_ceiling_light.txt) scene, at the default resolution and camera position. A total of 10 path trace iterations are performed, with a depth of 8 each. Denoising parameters are default as `color_phi = 3.740`, `normal_phi = 0.285`, `position_phi = 2.642`. Image resolution is by default `800x800`.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/time_filtersize.svg" title="Time vs. Filter Size" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Time vs. Filter Size
</div>

As seen in the plot, the denoise time cost increases with the filter size. This is expected because increased filter size generally means more denoise iterations. And since the computation cost of each iteration stays generally the same, more iterations mean more time cost.

Since at iteration $$i$$, the side length of the expanded filter is $$4 \times 2^i + 1$$, only filter sizes that is on threasholds are chosen.

The denoise results at each filter size is shown below:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/filter5.png" title="5x5 Filter" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/filter9.png" title="9x9 Filter" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/filter17.png" title="17x17 Filter" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: 5x5 Filter; Middle: 9x9 Filter; Right: 17x17 Filter.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/filter33.png" title="33x33 Filter" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/filter65.png" title="65x65 Filter" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/filter129.png" title="129x129 Filter" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: 33x33 Filter; Middle: 65x65 Filter; Right: 129x129 Filter.
</div>

<div class="row" style="width: 34%">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/filter267.png" title="257x257 Filter" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: 257x257 Filter.
</div>

We can see that as the filter size increases, the denoising benefits have diminishing returns, and sometimes erase local details. For this scene, denoising achieves best result when applying 65x65 filters, producing a smooth image while retaining details like shadows. If there are too few iterations, the noise is not sufficiently removed. However, if there are too more iterations, details like shadows begin to disappear because larger filters tend to blend in colors further away, and hence colors become overly simplified.

## Resolution

Aside from filter size (number of iterations), the resolution of the input image also affects how much time is taken to perform denoising. Here we perform denoising on several square resolutions (from `800x800` up to `4800x4800`) and examine the time cost with respect to the number of pixels in image.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/time_resolution.svg" title="Time vs. Filter Size" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Time Cost with Respect to Number of Pixels
</div>

It is clear that the time cost for denoising scales linearly with the number of pixels. This is also expected because the amount of computation for each pixel stays nearly constant, hence the total time cost scales with the number of pixels.

## Smoothing Benefits

To understand how denoising produces better images with fewer iterations, we first generate a ray trace ground-truth by performing 10k iterations, for the maximum quality. Then we compare the image generated by 10 iterations with denoising. Parts of difference are highlighted below:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/iter10_no_denoise.png" title="Original (10 iters, 8 depth)" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/iter10_denoise.png" title="Denoised (filter size 65x65)" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Original (10 iters, 8 depth); Right: Denoised (filter size 65x65).
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/diff_no_denoise.png" title="Origin Diff" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/diff_denoise.png" title="Denoised Diff" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Origin Diff; Right: Denoised Diff.
</div>

We can see that by applying denoising, the difference between the ground-truth and the image with 10 iterations is reduced rapidly. The only exceptions lie on the boundaries of colors and sharp object intersections. We can see that for homogeneous surfaces that have uniform colors, this denoising technique does a good job.

## Material Types

To compare the effect of denoising on different material surfaces, we examine perfect specular surfaces and purely diffusive surfaces.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/iter10_no_denoise.png" title="Specular Original (10 iters, 8 depth)" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/iter10_denoise.png" title="Specular Denoised (filter size 65x65)" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Specular Original (10 iters, 8 depth); Right: Specular Denoised (filter size 65x65).
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/diffuse_no_denoise.png" title="Diffusive Original (10 iters, 8 depth)" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/diffuse_denoised.png" title="Diffusive Denoised (filter size 65x65)" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Diffusive Original (10 iters, 8 depth); Right: Diffusive Denoised (filter size 65x65).
</div>

We can see that the denoiser performs much better on diffuse surfaces than on specular surfaces. On specular surfaces, particularly, the colors are smeared and blend together. This is expected because the color of pure specular surfaces rely solely on the color they reflect. This color, however, can have little relation to the surface normals or positions. Therefore, the denoiser cannot leverage the full potential of normals or positions to identify edges, therefore resulting in a bad reconstruction.

## Scene Differences

We also test the denoiser's effectiveness on different scenes. In this case, we test both [`cornell_ceiling_light`](https://github.com/plasmas/Project4-CUDA-Denoiser/tree/base-code/scenes/cornell_ceiling_light.txt) and the original [`cornell`](https://github.com/plasmas/Project4-CUDA-Denoiser/tree/base-code/scenes/cornell.txt) scene.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/iter10_no_denoise.png" title="Cornell Ceiling Light Original (10 iters, 8 depth)" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/iter10_denoise.png" title="Cornell Ceiling Light Denoised (filter size 65x65)" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Cornell Ceiling Light Original (10 iters, 8 depth); Right: Cornell Ceiling Light Denoised (filter size 65x65).
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/cornell_no_denoise.png" title="Cornell Original (10 iters, 8 depth)" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cuda-denoiser/cornell_denoise.png" title="Cornell Denoised (filter size 65x65)" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Cornell Original (10 iters, 8 depth); Right: Cornell Denoised (filter size 65x65).
</div>

We can see that with a smaller light in the original `cornell` box, with the same 10 iterations, the denoiser performed poorly and didn't remove a noticible amount of noise, while making the image darker. This is because a smaller light means less valid samples and we can obtain less information with the same number of iterations. As a result, the amount of noise increases, and the denoiser cannot effectively remove noise.
