---
title: "My First Raytracing Project"
layout: post
date: 2023-04-04 08:19
tag: graphics
image: /assets/images/jumapfptest.jpeg
headerImage: true
projects: true
#hidden: true # don't count this post in blog pagination
description: "This is a simple and minimalist template for Jekyll for those who likes to eat noodles."
category: project
author: juma
externalLink: false
---

Documenting all my learnings from creating my first raytracer/pathtracer & essentially anything new that I find out. Interested in this raytracing repository? [Click here to see it on my github](https://github.com/j-2k/Raytracing).

> :warning: Do becareful taking everything as granted, I am human so there might be some mistakes.

This documentation is targeted towards people with all ranges of experience. I try to make it as dummy as possible but sometimes we just have to get technical so I hope it all makes sense with just a little thinking. 

---

# Content

***Click on any section below to go there immediately in this page***

1. [UV Coordinates](#UV_Image) (the Hello World for a raytracer)
2. [Rendering My First Sphere](#Rendering_Sphere)
3. [Visualize Raytracing in Unity](#Visualize_Raytracing) (Small detour to see how a raytracer "looks" like)

to be continued...


---
<a name="UV_Image"></a>

## UV Coordinates
***The Hello World Stage***

<div class="side-by-side">
    <div class="toleft">
        <img class="image" src="/assets/raytracingproj/uvimage.png" alt="UV Image">
        <figcaption class="caption">UV Coordinates Shader Example</figcaption>
    </div>

    <div class="toright">
        <p>
        What is a UV? <br>
        UV Image or UV Texture Mapping can be interpreted as a basis for coordinates for every point in a 3D model, <strong><mark>it is essentially an instruction to tell a texture HOW it should WRAP AROUND a 3D object.</mark></strong> depending on how it was modelled (unwrapped), these "instructions" are created in the process of creating a 3D model, this term is called "Texture Unwrapping". however I don't specifically care about models in this context all I want is the coordinates in the context of creating a shader, & this is going off topic so back to coordinates.<br>
        <br>
        What are UV Coordinates used for in shaders? <br>
        UV coordinates are used in creating all types of shaders from awesome looking shader effects & also mapping such as normal mapping & displacement mapping. It's literally the basis for every shader as we use its coordinates to specify for instructions to happen at certain places.<br>
        Simply put looking at the image on the left you can see something happening in each x/y coordinate or in this case U is the X coordinate and V is the Y coordinate. The more it goes in a certain direction the color gets stronger, X is our red channel and Y is our green channel, combining the both will give you a yellow color as seen in the top right of the image.<br>
        <br>
        </p>
    </div>
</div>

<div class="side-by-side">
    <div class="toleft">
    {% highlight c++ %}

    //Renderer Class with Render Image Function
    void Renderer::Render(){
        for (uint32_t y = 0; y < m_FinalImage->GetHeight(); y++){
            for (uint32_t x = 0; x < m_FinalImage->GetWidth(); x++){
                glm::vec2 coord = {
			(float)x / (float)m_FinalImage->GetWidth(),
			(float)y / (float)m_FinalImage->GetHeight()
			};

                m_ImageData[x + y * m_FinalImage->GetWidth()] = PerPixel(coord);
            }
        }
    m_FinalImage->SetData(m_ImageData);
    }

    //Renderer Class with PerPixel Function (Acting Fragment Shader)
    uint32_t Renderer::PerPixel(glm::vec2 fragCoord){
	    uint8_t r = (uint8_t)(fragCoord.x * 255.0f);
	    uint8_t g = (uint8_t)(fragCoord.y * 255.0f);

	    return 0xff000000 | (g << 8) | r;
    }
    {% endhighlight %}
    <figcaption class="caption">Code for visualizing UV coordinates</figcaption>

    </div>

    <div class="toright">
        <p>
        How do we generate such a Image?<br>
        Here we are using a Vulkan Image to generate this for us, but the concept is the same in all instances. We first take the height & width of our image, then we loop over every single pixel in the image space and divide the current coordinates dimension with its respective image height/width.
        <br>
        Once we have a coordinate we send it in a PerPixel function to give us a color THATS IT dont overthink this part. More info on this function it acts as a stage in the popular graphics pipeline called "Fragment Shader". We didn't implement a real fragment shader though hence the word "act" I used previously... This is a image remember? :)<br>
        <br>
        Diving into the PerPixel Function things start to explode in my brain literally. I didn't understand a thing before & im not sure if I fully understand till now since im self learning everything here. But I think I know what is going on & im going to try to explain in simple terms hopefully. <br>
        <a href="https://en.wikipedia.org/wiki/RGBA_color_model" target="_blank">I found this on wikipedia while writing this next section its going to be crazy useful, click it to check it out, it covers how colors & bits work, all low level technical stuff. You can skip to the next part below if its boring</a>.
        </p>
    </div>
</div>

<div class="side-by-side">
    <div class="toleft">
    <center><strong>
    <h2>Bits & 
    <span style="color:cyan">C</span><span style="color:#F800FF">o</span><span style="color:yellow">l</span><span style="color:lightgreen">o</span><span style="color:#FFA500">r</span><span style="color:#FF00A1">s</span>!</h2>
    Bits are either 1 or 0<br>
    1 Byte is 8 Bits<br>
    1 Hex Digit is 4 Bits<br>
    uint32_t holds 32 Bits<br>
    uint8_t holds 8 Bits<br>
    <mark>RGBA is a color format that holds 32 Bits</mark><br>
    </strong></center>
    <figcaption class="caption">Some important things to know</figcaption>
    <br>
    <br>
    <img class="image" src="/assets/raytracingproj/HexRGBAbits.png" width="800" height="60" alt="Hex RGBA Bits Image">
    <figcaption class="caption">RGBA32 Memory Channel</figcaption>
    <figcaption class="caption"><mark>Bytes stored in memory in a little endian machine are in the order of ABGR</mark></figcaption>
    </div>

    <div class="toright">
        <p>
        First of all WHAT is our image?<br>
         Well I didn't show this previously but we defined our image format to be in RGBA (this was done via Vulkan so its helping a lot here) but the question still lies what is RGBA? Here I went into a mini downward spiral of curiosity since I didn't learn or do much low level programming, I wanted to know exactly what in the magic is going on specifically in the line below. Now continue on the left.
        </p>
         {% highlight c++ %}return 0xff000000 | (g << 8) | r; {% endhighlight %}
        <p>
         Looking at the beauty of this format on the left everything made sense to me now. RGBA is 32 bits each channel holds 8 bits (32 bits ÷ 4 color channels) which is also 2 HEX digits. Ever wondered in a game why when you change the color of your crosshair the max is 255? the answer is 8 bits, uint8_t is a unsigned integer (only positive) datatype that can hold a value of exactly up to 255.
        </p>
    </div>
</div>

{% highlight c++ %} return 0xff000000 | (g << 8) | r; // | is the or operator // << is the bitshifting operator{% endhighlight %}
So what is going on here? every GROUPED 2 HEX DIGITS after "0x" is a color channel except that everything here is actually reversed instead of RGBA its ABGR (LOOK AT THE HIGHLIGHTED TEXT UNDER THE RGBA32 MEMORY IMAGE ON THE LEFT). So again every GROUPED 2 HEX DIGITS after "0x" make a color channel "return 0xff000000" ff is our alpha the next 2 zeros are our blue the next 2 are green the next 2 are red. Now here comes the [bitwise or operators](https://en.wikipedia.org/wiki/Bitwise_operation){:target="_blank"} r is being directly inserted into the memory block at the start so it immediately takes up 8 bits r is a variable of uint8_t, then g << 8 does the same thing except the << signales it to SHIFT 8 bits to the left and now is occupying the space AFTER the red channel, we then leave the blue channel empty to get the classic UV coordinate image of red & green. If you are curious on how the or operator inserts im going to quickly explain below using bits. Long story short, ORing 2 bits together will give you 0 if both bits are 0, in the case one of the 2 bits are 1 the answer is 1.<br>
[Click here to open a new tab to a wiki page of bitwise operations if you want to learn more.](https://en.wikipedia.org/wiki/Bitwise_operation){:target="_blank"}
{% highlight binary %}//Example from https://en.wikipedia.org/wiki/Bitwise_operation
   0010 (decimal 2)
OR 1000 (decimal 8)
 = 1010 (decimal 10){% endhighlight %}
Finally, with this explanation done we can now go back to the code & return this memory block that is a uint32_t which is what the function returns. This color then gets placed into the imagedata or PIXEL whenever it gets called.

I loved this segment so much because of the low level part since I never was REALLY taught this properly & I learned this all by myself & am really happy with my progress even though its small. Hope to see you in the next section where its all just Mathematics✨

---

## Rendering My First Sphere <a name="Rendering_Sphere"></a>

---

## Visualize Raytracing in Unity <a name="Visualize_Raytracing"></a>

---



[Click here to go back to my home page](https://j-2k.github.io).
