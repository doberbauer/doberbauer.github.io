Title: Perlin Noise Topographic Map Generator
Date: 2024-01-01 12:00
Category: Art


I've always loved [topographic maps](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fi.pinimg.com%2Foriginals%2F13%2F8c%2F83%2F138c83d3c62e3c3b5cdc413704bc6ce5.jpg&f=1&nofb=1&ipt=8fa67a08008a5a888c1789d03ab113c00c8bb66c3cf9eb6f49bc47aa65127e6d&ipo=images). They're so useful but so rarely used in everyday life. I recently had some time on my hands and decided to whip up some code to generate some procedural topographic maps. 


![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1705962508-0.png)

![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1705962551-0.png)



# Generating the noise texture
I knew from my tinkering with Blender and Photoshop that there's a pretty solid cloud/height map noise generation method. A little searching revealed it's called [Perlin Noise](https://en.wikipedia.org/wiki/Perlin_noise) and it's commonly used to procedurally generate terrain in computer graphics. 

I was coding all this on my phone via Pydroid 3 and I didn't want to install some random perlin noise package from pip. I wanted a simple straightforward numpy implementation. 
I did a quick search for "Python Perlin noise numpy" and found [this](https://saturncloud.io/blog/producing-2d-perlin-noise-with-numpy-a-comprehensive-guide-for-data-scientists/) article with code. 

It did a nice job of generating the texture so I moved on to drawing topo lines.

# Generating interval lines
It may not be the *best* way to do this, but it's the way that made sense to me at the time. 
1. Use the code from the article to generate a noise matrix. 
2. Get the values of the highest peek and the lowest valley
3. Slice through the terrain from bottom to top at set intervals
4. Use binary dilation to find the boundary of the slice
5. Append the boundaries to a boundaries matrix

This just yields the boundaries as a mask. 
`noise[mask] = 1` overlays the mask onto the original array. 

# Improvements
I didn't like how this looked so I went a step further.
The raw noise texture has very smooth gradients and in the final image with the topo lines it just looked too fluffy and soft. 
I wrote another method that does the following:
1. Iteratively slice through the noise matrix from bottom to top
2. Mask the noise matrix with each slice
3. Set all the values for pixels within the mask to the same value (the index of the slice)
4. Output the resulting noise matrix.

Now instead of [smooth gradients](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Frmarcus.info%2Fblog%2Fassets%2Fperlin%2Fraw%2Foctaves.png&f=1&nofb=1&ipt=23f9b35dd60ce793ca61e3b142f2c4138cf57bac8b42d94418bc529e0cdafffd&ipo=images) I had nice [flat steps](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fi.etsystatic.com%2F9147400%2Fr%2Fil%2F48cc55%2F2068811700%2Fil_fullxfull.2068811700_aie8.jpg&f=1&nofb=1&ipt=d0005b62a7f991f54af6d378be311bcbffc6a4d1f827931329aef8b43f47d768&ipo=images) that were more appealing to my eye. 

[Perlin github gist](https://gist.github.com/hyperific/6d0565ce85a052839fc8e95aa109fbe9)