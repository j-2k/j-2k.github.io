---
title: "🌱 Shell Texturing"
layout: post
date: 2023-11-21 07:45
tag: shaders
image: /assets/shellcombined.png
headerImage: true
projects: true
#hidden: true # don't count this post in blog pagination
description: "Shell Texturing Grass"
category: project
author: juma
externalLink: false
usemathjax: true
---

This post will mainly be about my YouTube video where I go over shell texturing.

(Video link will go here)

I will also write a YouTube script here, even though it's probably a good idea to just keep the script private, I don't really care,
so I will make it public.

---

## Part 0.5 - What is Shell Texturing?
Simply put it's an optimized technique for rendering certain types of geometries that might be difficult to render due to them requiring high amounts of polygons/triangles. For example, instead of having 100 blades of grass or hair with real geometry, it will require triangles for every single "strand", the issue with this is that for each "strand" will contain 10 triangles and that 10 triangles spanned throughout having a 100 blades (100 * 10 = 1000 tris already) can be exponentially expensive.

However, since computers are turbo-fast nowadays there are ways to get good real physics & graphics when it comes to hair, fur, & grass, etc. But, discussing that is out of the scope of this video.

---

## Part 1 - Managing The Shell Textures
First, a C# script to manage the shell textures (I CALLED THEM "SHEETS" IN MY CASE!). This ShellTextureManager script needs to manage all of the following functions & it's good to keep in mind this data is being sent to the GPU/Shell Texture Shader.

*Initial Parameters*
- Height (Manage height & spreading the sheets across the height)
- Density (density of the sheets / the amount of layers)
- Color (Shell texture color)

*Post-Initial Phase Parameters*
- Thickness (Change the thickness of the blocky "grass" shape)
- RNG Control (Max & Min, a way to control the RNG but also gives a similar effect to thickness)

These are the parameters & functions we need to have in our shell manager to control the shell texture effect & play around with the values to find something nice. More importantly, now we will tackle each of the functions above & talk about how to do them, prepare yourself for some math.

### Height
Initially, I hard coded this to test with quads but soon enough I realized I shouldn't continue with this approach since if I change the mesh it will just look like spheres stacked on top of each other with the same scale & etc. That's not what we want, we need to protrude the mesh along its normal to this we simply do the following.

V vector = for every vertex on the object  
N vector = for every vertex normal  
D scalar = distance/height  (SCALAR OFFSET)  
i decimal = normalized shell texture index  (ACEROLA CALLES THIS THE HEIGHT BUT IM CALLING IT THE "NORMALIZED INDEX")  

$$\vec{V} = \vec{N} \times {D} \times i$$

***VERY IMPORTANT! THESE MATH FORMULAS SHOWN ARE NOT TO BE SCALED 1:1 IN CODE, IT IS SHOWING THE FORMULA IN A SIMPLE MATTER. IN ACTUAL CODE YOU HAVE TO ADD THIS VECTOR OFFSET, THIS RULE WILL CONTINUE FOR THIS WHOLE POST/VIDEO.**

Notes: none.

### Density



---

## Part 2 - UV & Noise

---

## Part 3 - Blocky Grass

---

## Part 4 - Thickness

---

## Part 5 - Windy Grass

---

## Part 6 - Sphere Displacement

---


