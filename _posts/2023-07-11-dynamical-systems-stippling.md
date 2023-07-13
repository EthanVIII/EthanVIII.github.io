---
layout: post
title: Creating stippled images using dynamical systems
description: I implemented an algorithm in Processing that produces organic-looking dot pattern images from images. This process uses dynamical particle systems to produce interesting outputs.
---

This process caught my eye as it produces some organic stippled images using a chaotic deterministic system.

Using a portrait image of a friend (Juliette), I generated the below image using this process. Bright pixels in the image are depicted as larger dots and dark pixels smaller dots, which creates different apparent shades when put against a black background. Hence calling it "stippling" is actually a bit of a misnomer as traditional stippling relies on different densities of dots to depict shade instead of different sizes.

![A black and white dot pattern image of a portrait of a woman](/assets/stippling_output.png)

## The stippling algorithm
This algorithm simply takes an image and converts a proportion of pixels into differently sized particles based on brightness. It then puts these objects into a dynamical system where it keeps track of position, velocity, and acceleration. At each timestep the particles interact by pushing or pulling on each other based on their size. 

For this image I had particles within a certain radius repel each other. The distance between two particles would be linearly transformed into the repelling "force" on both particles. The final acceleration of each particle at each timestep would simply be the sum of force vectors on that particle. The particles also had inertia, where the force applied is dependent on the particle's size.

This created a pleasing effect where the edges of an object would tend to "shed" into empty space and smaller particles (darker pixels) would drift away faster. This gives the stippled images hazy edges and a more organic feel.

This process was inspired by Robert Hodgin's work. Unfortunately, his writeup is no longer available on his website.

## Image pre-processing
I found that sparse or mostly black source images worked best for the look I wanted. This reduced the number of particles and allowed them to spread out. The subject would also be the main focus of the resulting stippled image. To get this effect, I removed the background from the source photo below.

![A portrait photo of a woman](/assets/stippling_source.jpg)

I also converted this image into greyscale using the luminosity method, which helped with choosing the final particle size.

## Luminosity to particle size
To get our final particle size, define a function $f$ that maps from brightness (0 to 255) to a positive radius of the particle. The only constraint is that $f(0)=0$ so that black pixels do not make particles, and $f$ probably should be monotonic increasing. 

After some trial and error, I actually used an odd function that would often return negative radii. While it contradicts what i mentioned previously, it seemed to work for this image. This function was:

$$f(x)=\frac{1}{\log(1.05x)}$$

where $x$ is the brightness of the pixel from 0 to 255. 

This did generate a few artifacts in the final output, such as the abnormally large particles on the wristwatch buckle. If I did this again, I would normalise the input and pick a function from there.

## Picking the timestep
Finally, I ran the simulation to get the final effect. This part isn't particularly remarkable. I generated a series of images of the system from $t=0$ to $t=30$.

![Animation of a dynamical particle system created to represent a portrait.](/assets/stippling_anim.gif)

I picked a frame that was the best (personally) and called that the final product!








