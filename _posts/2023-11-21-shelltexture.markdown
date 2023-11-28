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

## Part 0 - Disclaimer
In this video I will probably make some mistakes since I'm learning a new topic, feel free to correct me in the comments about anything I say wrong.

This video's main objective is to not showcase shell texturing but to go over what I did & learned in the #acerolafurrychallenge held by Acerola. This video will contain some math & shader terms that I might not explain from the ground up so I apologize in advance if something didn't make sense (insert Acerola video & advise to watch it before watching my video), im trying my best to keep this video short.

I also want to emphasize that I tried doing this challenge to the best of my ability by just learning from the video instead of straight jumping into Acerola's source code & copying code, I tried to do everything at the start by myself & only when I got stuck/finished the whole project I checked his code (it's similar anyway since I based mine off his video explanation).

Before I begin on what I did, I will try quickly explaining shell texturing. 

---

## Part 0.5 - What is Shell Texturing?
Simply put it's an optimized technique for rendering certain types of geometries that might be difficult to render due to them requiring high amounts of polygons/triangles where instead of using "real" geometry you fake the effect with shells (or meshes). 

For example, 100 blades of grass or hair with real geometry, will require triangles for every single "strand", the issue with this is that let's say each "strand" will contain 10 triangles and that 10 triangles spanned throughout having a 100 blades (100 * 10 = 1000 tris already, 100 blades is nothing btw) can be potentially expensive, on top of all the physics calcuations in the vertex shader & etc.

However, since computers are turbo-fast nowadays there are ways to get good real physics & graphics when it comes to hair, fur, & grass, etc. But, discussing that is out of the scope of this video.

#### How does Shell Texturing solve this issue?
Shell texturing uses a technique where we duplicate the base mesh of an object on top of each other while killing certain pixels on the mesh to give off a shape of exactly what we want. For example, grass & fur/hair can all be done using this method, as you can see in these pictures.

Since we duplicate the base mesh a couple of times (show examples of this on-screen from #acerolafurrychallenge) to create an effect we want like fur, we don't create many triangles (assuming you are doing it right) like a geometry grass shader for example. However, this method is still prone to optimization issues such as overdraw, if you'd like to learn about overdraw just go watch Acerola's video where he covers it.

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

### Density
Since the Script manages the density this part is simple, in my case I have an array that stores all the shells & if I change the number of layers I have it just adjusts from the upper bound of the array, example if I want more shells I add, else if I need less I delete. These both start from the upper bound of the array in both cases. After adjusting the density you need to run the height function to fix all the new changes. Moving on to Part 2!  

Notes: none.

---

## Part 2 - UV & Noise
Let's talk about randomness, since we want to create grass, grass heights are often random in the real world, so to replicate a similar setting we first need a RNG for this shader, how do we do this? NOISE. we can do this by just stealing a hashing function from Shadertoy, but in all seriousness, we just need a decently uniform RNG, which a hashing function can do pretty well.  

I'm in no way a cryptographic/randomness specialized guy, but I have used hashing functions many times, however I don't exactly understand what happens inside them, all I know is that in a hashing function, we just insert a seed & that seed will be moved/bit shifted in the function so much that a random number will come out of it ranging from 0 - 1, kinda all you need to know, not important to understand what happens inside that much.  

{% highlight c++ %}
float hash11(float p)       //hash11 I stole from shadertoy (1 input scalar 1 output scalar)
            {               //p gets inserted as our seed
                p = frac(p * .1031);
                p *= p + 33.33;
                p *= p + p;
                return frac(p);
            }
{% endhighlight %}

Since we now have a black & white shell we need to address why it's all just 1 color, that's because we need to resize this UV map & it's only a size of 0 - 1, to be larger and to do that we just multiply the size by 100, now we have a 100x100 "blocks" that are random greyscaled colors ranging from 0 - 1.

Here I came across another problem where instead of using an uint, I used a float since I never weirdly used an int type in shaders I had a spam of noise on my quad because it's all in decimals so I never got the blocky/floor clamped numbers you would get using an uint instead of float. I was stuck on this even though I knew the issue was it being decimals while trying to understand why my noise value wasn't magically floored. Insert typical programmer brain fart moment.  

---

## Part 3 - Blocky Grass
Now we have randomness which means we can start to simulate grass, our way of doing this is to simply compare 2 values, the random value and the height value of the shell (both range from 0 - 1).

---

## Part 4 - Thickness

---

## Part 5 - Windy Grass

---

## Part 6 - Sphere Displacement

---


