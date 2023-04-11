---
title: "My First Raytracing Project"
layout: post
date: 2023-04-04 08:19
tag: graphics
image: /assets/images/jumapfptest.jpeg
headerImage: true
projects: true
#hidden: true # don't count this post in blog pagination
description: "Raytracing Project Documentation"
category: project
author: juma
externalLink: false
---


**Documenting all my learnings from creating my first raytracer/pathtracer & essentially anything new that I find out. Interested in this raytracing repository?** [Click here to see it on my github](https://github.com/j-2k/Raytracing).

> :warning: **Do becareful taking everything as granted, I am human so there might be some mistakes.**

**This documentation is targeted towards people with all ranges of experience. I try to make it as dummy as possible but sometimes we just have to get technical so I hope it all makes sense with just a little thinking. All my learning resources are included in the github link above if you are interested in learning from the same resources I learned from.**


---

# Content

***Click on any section below to go there immediately in this page***

1. [UV Coordinates](#UV_Image) (the Hello World for a raytracer)
2. [Mathematics Required](#Math_1) (Circle (2D) Intersection explanation)
3. [Rendering My First Sphere](#Rendering_Sphere) 
4. [Visualize Raytracing in Unity](#Visualize_Raytracing) (Small detour to see how a raytracer "looks" like)

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
<a name="Math_1"></a>

## Mathematics for Circle Intersection

***Line VS Ray***
<div class="side-by-side">
    <div class="toleft">
    <center>
        <strong>Line</strong><br>
        <br>
        <iframe src="https://www.desmos.com/calculator/yu0jkumuek?embed" width="500" height="300" style="border: 1px solid #ccc" frameborder=0></iframe>
        <figcaption class="caption">Demos Example of the line y = mx+b.<br>
        Where m the slope is 2 & b the offset is 5.</figcaption>
        <br>
        <strong>Ray</strong><br>
        <br>
        <iframe src="https://www.desmos.com/calculator/vzccpddhzp?embed" width="500" height="300" style="border: 1px solid #ccc" frameborder=0></iframe>
        <figcaption class="caption">Demos Example of the "ray" with origin (0,0) & direction of (5,5).<br>
        Both graphs are interactive click the lines to see its coordinates.</figcaption>
        <br>
        <br>
        <strong>
        <mark>Important Notes</mark><br>
        Vector2 refers to a vector of 2 components 𝑿 𝒀<br>
        Vector3 refers to a vector of 3 components 𝑿 𝒀 𝒁<br>
        Circle Formula is (𝒙 - 𝒂)<sup>2</sup> + (𝒚 - 𝒃)<sup>2</sup> = 𝒓<sup>2</sup><br>
        (𝒂 , 𝒃) is the origin coordinate for the circle <br>
        𝒓 is the radius
        </strong>
    </center>

    </div>

    <div class="toright">
        <p>
        Coming from Unity the two are easy to differentiate, however, some people might have different definitions for this, & it's okay just use my definitions for now. Also for this section, everything will be in 2D & can be easily extended to 3D by just introducing an additional Z coordinate.<br>
        <br>
        A line is exactly what you think it is, just a long object that extends infinitely. In the context of a game engine you can tell a line to go from position X to position Y, & all it needs is 2 positions & it will draw a line.<br>
        <br>
        A ray however is slightly different this time a ray dosent want 2 positions rather it wants a <mark>STARTING POSITION & A DIRECTION</mark>. Imagine this ray in your head you have the starting position set to the origin & you specify the direction to be (5,5) where would it go?<br>
        <br>
        Looking at the image below the first graph, that is exactly what you would get, even though I said its a quote on quote ray under the graph its important to know that in order to differentiate between a line and a ray is that at the end of a ray you would place a arrowhead to show where the ray is pointing towards its direction. I cant show this in desmos so I placed some lines to fake an arrowhead look. <mark>Dont forget a ray contains a origin & a direction!</mark><br>
        <br>
        Quickly, the formula for the ray on the left is the following: <br>
        𝑷<sub>𝒙𝒚</sub> = 𝒂<sub>𝒙𝒚</sub> + 𝒃<sub>𝒙𝒚</sub>𝑻<br>
        𝑷 for any point along the line<br>
        𝒂 = origin<br>
        𝒃 = direction<br>
        𝑻 = direction scaler<br>
        Substitution: 𝑷<sub>𝒙𝒚</sub> = (0,0) + (5,5)(1), However its also good to know that doing things this way is "bad" we should just normalize the direction vector and just multiply it by the direction scaler. Which looks like this.<br>
        𝑷<sub>𝒙𝒚</sub> = (0,0) + (1,1)(5). With this done it should be known that if you move the origin to lets say (5,5) the whole ray moves to (5,5) & will still point towards the positive 𝒙 𝒚 quadrant (top right or 1,1) with a direction scaler of 5,5 (the tip will be at 10,10).
        
        

        </p>
    </div>
</div>

### Circles & Square Root Magic

<div class="side-by-side">
    <div class="toleft">
    <strong><center>Circle Formula is (𝒙 - 𝒂)<sup>2</sup> + (𝒚 - 𝒃)<sup>2</sup> = 𝒓<sup>2</sup><br></center></strong><br>
    <iframe src="https://www.desmos.com/calculator/6otgrghcfp?embed" width="500" height="500" style="border: 1px solid #ccc" frameborder=0></iframe>
    <figcaption class="caption">Circle with radius 2 & orgin (0,0) & line y = x <br>
    Solved Circle => y = ±sqrt(4 - 𝒙<sup>2</sup>)<br></figcaption><br>
    </div>

    <div class="toright">
        <p>
        Looking at the circle formula there is something that happens in the back that isnt too obvious, but I will make it clear here. Imagine a circle with radius 2 @ origin 0 now substituting that all in the circle formula & solving for y will give you this.<br>
        => (𝒙 - 0)<sup>2</sup> + (𝒚 - 0)<sup>2</sup> = 2<sup>2</sup><br>
        => 𝒙<sup>2</sup> + 𝒚<sup>2</sup> = 4<br>
        y = sqrt(4 - 𝒙<sup>2</sup>)<br>
        <br>
        If you plug the final answer of y into desmos or a graphing calculator you will get ONLY the blue line of the circle as you can see on the left. In order to complete the circle from just a semi-circle you need to add a minus / turn the sqrt into a negative to get the other half of the circle to be under. flipped semi circle is y = <mark>-</mark>sqrt(4 - 𝒙<sup>2</sup>)<br>
        <br>
        <strong>Magical Square Roots & The Second Semi Circle</strong><br>
        What is the square root of 4? its 2 why? because 2 squared is 4. Now what if the 2 was -2, what is -2 squared? also 4. With this in mind now you know that square roots actually can have multiple solutions, its good to show the ± symbol before a sqrt to show that it has multiple solutions.
        </p>
    </div>
</div>

### Rays & Circle Intersection

Now we need to find the circle intersection, you can just mouse over the graph and actually find the intersection but we need to find it with a formula & use it in our program. Lets imagine the black line to be a ray that intersects our circle but additionally imagine MULTIPLE rays coming from the bottom left and going towards the sphere in all directions. Here I'll just make something in desmos to show what I mean real quickly.
<div class="side-by-side">
    <div class="toleft">
    <p>
    Looking on the right is literally something really close to our raytracer we are blasting so many rays from the bottom left which we can assume in our case is the camera & shooting it forward towards all directions from a origin. Now ofcourse there are much more rays in our program this is just for showcasing. You can also see that some rays hit & go through the circle having 2 INTERSECTIONS mean while some are out of bounds of the circle and didnt manage to hit the circle while some lines just graze a singular point on the circle.<br>
    <br>
    So with these intersections in mind we continue to how we find these points. Let take the ray going through the origin, the formula for that ray would be:<br> 
    𝑷𝒙𝒚 = (-3,-3) + (1,1)𝑻 ///(𝑻 is infinite in the graph but its ok),<br>
    now we know the ray formula lets bring the circle formula back its: <br>
    (𝒙 - 𝒂)<sup>2</sup> + (𝒚 - 𝒃)<sup>2</sup> = 𝒓<sup>2</sup> simplify it with origin 0 & move r to the left<br>
    𝒙<sup>2</sup> + 𝒚<sup>2</sup> - 𝒓<sup>2</sup> = 0<br>
    <br>
    This is actually enough now with these 2 formulas:<br>
    <mark>𝑷<sub>𝒙𝒚</sub> = 𝒂<sub>𝒙𝒚</sub> + 𝒃<sub>𝒙𝒚</sub>𝑻</mark><br> (substitued in it would be 𝑷𝒙𝒚 = (-3,-3) + (1,1)𝑻)<br>
    <mark>𝒙<sup>2</sup> + 𝒚<sup>2</sup> - 𝒓<sup>2</sup> = 0</mark><br>
    Lets substitute the first equation into the second, this might get a bit heavy so try to tag along.
    </p>
    </div>

    
    <div class="toright">
    <iframe src="https://www.desmos.com/calculator/qghgx0uanr?embed" width="500" height="500" style="border: 1px solid #ccc" frameborder=0></iframe>
    </div>
</div>

(𝒂<sub>𝒙</sub> + 𝒃<sub>𝒙</sub> 𝑻)<sup>2</sup> + (𝒂<sub>𝒚</sub> + 𝒃<sub>𝒚</sub> 𝑻)<sup>2</sup> - 4 = 0 Which after substitution looks like (-3 + 1𝑻)<sup>2</sup>+(-3 + 1𝑻)<sup>2</sup>- 4 = 0<br>
The key here is to now solve for 𝑻, im not going to go step by step for solving it since its massive & my web dosn't currently have mathjax implemented. <br>
but at the end when you solve for 𝑻 you get a nice small equation: 2𝑻<sup>2</sup> - 12𝑻 + 14 = 0 what does it look like?<br>
A Quadratic equation so how do we solve this? We use the [Quadratic Formula](https://en.wikipedia.org/wiki/Quadratic_formula){:target="_blank"} <br>
However, in our case we can just use the discriminant which is everything inside the sqrt, (b<sup>2</sup> - 4ac)<br>
The coefficients are as follows: a = 2, b = -12, c = 14. Substitute them in the discriminant we get 144 - 112 = 32.<br>
One thing the answer of the discriminant tells us is actually how many solutions there are depending on the answer, in our case we got 32 which is above 0 which means there is 2 solutions (2 INTERSECTIONS). Now the discriminant is dependent on if its above 0 then it means there are 2 solutions if its 0 then that means it has 1 solution if its under 0 it means it has 0 solutions. This is literally what we will be using to find out if we intersect a sphere or not.<br>
Another Example with magic square roots below to show the effect of the discriminant:<br>
<strong>sqrt(4) is 2 or -2 // 2 solutions<br>
sqrt(0) is 0 which ±0 is the same // 1 solution<br>
sqrt(-x) is 𝒊 which any negative number under a square root has no real solutions.<br>
</strong>
<center>
<img class="image" src="/assets/raytracingproj/rootsphere.jpg"  alt="Hex RGBA Bits Image">
<figcaption class="caption">Ray-sphere intersection results</figcaption>
<figcaption class="caption">Taken from popular raytracing in 1 weekend book</figcaption>
</center>
This is actually just enough to know if our ray hit something or not. But if you want to find the exact intersection points you have to substitute all coefficients in the quadratic formula. First in our discriminant we know we have 2 solutions so that means our answer after substitution & solving should give us 2 answers which is true, the answers are 4.4142136~ & 1.5857864~. These numbers are what 𝑻 is equal to so what do we do now? Sub them in the ray equation which will yield you 2 equations -3 + 1(1.5857864) => -1.414~ & -3 + 1(4.4142136) => 1.414~ & since the x & y is equal the coordinates are (-1.414 , -1.414) & (1.414 , 1.414), that is all we can also tell the negative version is the one closer to us or in other words the first intersection since our origin was (-3 , -3).<br>
<strong>Check the graph above and mouse over the line intersecting the circle at the origin and check its intersection points to see for yourself.</strong>

---
<a name="Rendering_Sphere"></a>

## Rendering My First Sphere

***ALMOST EVERYTHING HERE IS BASED ON THE PREVIOUS PART I HIGHLY SUGGEST UNDERSTANDING THE MATH BEFORE PROCEEDING***

---

## Visualize Raytracing in Unity <a name="Visualize_Raytracing"></a>

***I tried to visualize my raytracer in Unity, here's my take on it***

---


[Click here to go back to my home page](https://j-2k.github.io).
