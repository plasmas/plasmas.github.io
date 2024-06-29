---
layout: page
title: Mini-Minecraft
description: A simple Minecraft clone written in C++ and OpenGL.
img: /assets/img/placeholder.jpg
importance: 6
category: C++
related_publications: false
---

[Source](https://github.com/plasmas/Mini-Minecraft)

_Per course policy, access to code is only granted upon request._

<!-- https://youtu.be/khf67kJt4hQ -->
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include video.liquid path="https://www.youtube-nocookie.com/embed/khf67kJt4hQ" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## Highlights

- Implemented a procedural terrain generation system using Perlin noise, featuring multiple biomes and a complex cave system.
- Developed a multithreaded chunk generation and loading system using Qt, ensuring smooth gameplay and efficient resource management.
- Created visually appealing effects, including animated textures, a realistic day-night cycle, water waves, distance fog, and dynamic grass coloring.
- Engineered robust player movement and interaction mechanics, including precise collision detection, block breaking, and placing functionality.

---

## Procedural Terrain Generation

### Biome System

The terrain is procedurally generated using a simple Perlin noise algorithm. The terrain is generated in chunks, and the chunks are loaded and unloaded as the player moves around the world. There are 4 types of biomes in the world:

- mountain (stone blocks with snow on top if the height is over 190, using 2d fbm noise function)
- snow land (dirt blocks with snow on top using 2d Perlin noise)
- grassland (dirt blocks with grass on top using smooth 2d Perlin noise, when the temperature is low, the grass is yellow)
- desert (sand blocks using 2d Perlin noise with small amplitude).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/placeholder.jpg" title="Mountain & Snow land biome" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/placeholder.jpg" title="Grassland biome" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/placeholder.jpg" title="Dessert biome" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Mountain & Snow land biome; Middle: Grassland biome; Right: Dessert biome.
</div>

Also, there is a cave system in this world, 3D perlin noise functions are used with large grid size to generate caves beneath the terrain's surface, and if the height is lower enough, the cave will be filled with lava.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/placeholder.jpg" title="Lava Cave System" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Lava Cave system
</div>

### Multithreaded Generation and Chunk Loading

We used multithreading primitives in `Qt` to load and generate chunks in parallel. This allows for a smooth experience when moving around the world. `QRunnable`, `QThreadPool` and `QMutex` are used for this purpose. The following classes are implemented:

- BlockTypeWorker: Extends QRunnable, which is responsible for creating chunks of a terrain zone, and will be used in initializing terrain and expand terrain. The generated chunks will be added to a vector and will be used in creating VBO data later at each tick.

- VBOWorker: Responsible for creates the VBO data for a given chunk and pushed to a vector which will be passed to GPU in main thread.

In general, each chunk's type is computed once in a worker thread, and persists for the lifecycle of the program. The VBO data, however, is generated based on the player's position and is updated every frame. Chunks which are not visible to the player are offloaded from the GPU, and chunks which are visible are generated in worker threads and loaded into the GPU by the main thread.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/placeholder.jpg" title="Fast Terrain Generation & Loading" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/placeholder.jpg" title="Terrain Offloading" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Fast Terrain Generation & Loading; Right: Terrain Offloading.
</div>

---

## Visual Effects

### Animated Textures

Animated textures were developed to enhance the visual appeal of the game environment. UV offsets were coded into the static face array, allowing for precise texture mapping on block faces. A unique approach was taken to animate textures, where a flag in the VBO data indicates the need for animation, and a time variable in the shader controls the animation pace. This resulted in a distinctive, stepwise movement of water and other textures, adding a dynamic element to the game world.

### Day-Night Cycle

The day-night cycle was implemented to add realism and depth to the game's atmosphere. An increment timer calculates the sun's direction, and the color of the sky is interpolated based on the angle between a ray cast and the sun's direction. This creates a smooth transition between day and night, dynamically changing the sky's color and enhancing the immersive experience of the game.

### Water Waves

To simulate the natural movement of water, a **sine function** was used to distort the water surface, creating realistic water waves. Height field sampling was employed to accurately determine the normals for the distorted surface, allowing for effective application of **Blinn-Phong shading**. This technique made the changes in the water's surface more visible, significantly improving the game's visual quality.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/placeholder.jpg" title="Animated Texture of Lava" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/placeholder.jpg" title="Day-Night Cycle" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/placeholder.jpg" title="Water Waves" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Animated Texture of Lava; Middle: Day-Night Cycle; Right: Water Waves.
</div>

### Distance Fog

Distance fog was introduced to add depth to the game world and improve rendering performance by obscuring distant objects. The distance between each block and the player's camera is calculated, and the block's color is interpolated with a foggy white based on that distance using the smoothstep function. This effect not only enhances the visual appeal of the game but also contributes to a more efficient rendering of the game environment.

### Procedural Grass Coloring

Procedural grass coloring was added to diversify the environment, making it more vibrant and realistic. A new block type for grey grass was introduced, and the color of the grass was adjusted in the fragment shader to appear darker and yellowish in colder areas. This dynamic color variation responds to the simulated temperature conditions in the game world, adding depth and realism to the grassland biome and enhancing the overall immersive experience.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/placeholder.jpg" title="Distance Fog" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/placeholder.jpg" title="Procedural Grass Coloring" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Distance Fog; Right: Procedural Grass Coloring.
</div>

---

## Player Movement and Interaction

### Controls

In the `tick` function, the `InputBundle` is processed to update the player's acceleration and velocity. A drag force is applied to the player to simulate air resistance.

### Collision Detection

In each tick, compute the displacement of the player and **grid march** from all 12 vertices of the player hit box. Compute the minimum distance the player can move in all 3 directions. Then move the player with the minimum distance (minus a very small offset).

### Block Breaking and Placing

On mouse button click, cast a ray of length 3 from camera position. To delete a block, if the ray hits a block, set the block type to `EMPTY`. To add a block, check the ray hit point on the block to determine the position to add the new block. Then set the new block type to `STONE`.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/placeholder.jpg" title="Collision Detection" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/placeholder.jpg" title="Block Breaking & Placing" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Collision Detection; Right: Block Breaking & Placing.
</div>
