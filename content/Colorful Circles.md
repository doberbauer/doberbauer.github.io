Title: Colorful Circles
Date: 2024-01-01 12:00
Category: Art

January 1st marked the first day of [Genuary](https://genuary.art/)! 
Genuary is an annual celebration of [generative art](https://en.wikipedia.org/wiki/Generative_art). If you're a regular reader of this blog you'll know that I love generative art. I'm not very good at it, but you can't get better at a thing unless you practice and that's what I want to get out of my art projects in the month of ~~January~~ Genuary. 

The prompt for Genuary1 is *"Particles, lots of them."*

As such here is my script that randomly placed randomly sized circles with randomly picked colors from a color map. 

[View on Github](https://gist.github.com/hyperific/d6ec03e6d7c3284517a883290aab4b74)

![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1706084533-0.jpg)

![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1706134692-1.jpg)

![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1706134692-0.jpg)


I wrote it on my phone using [Pydroid3](https://play.google.com/store/apps/details?id=ru.iiec.pydroid3) and I sized the output so I could use it as my phone's background.

``` Python
from PIL import Image, ImageDraw
import numpy as np
import matplotlib.pyplot as plt

#Parameters
color_map = "viridis"
ncolors = 3
width = 720
height = 1600
circles = 1000

width *= 2
height *= 2

cmap = plt.get_cmap(color_map) #Colormap init

colors = np.array([cmap(i) for i in np.linspace(0,1,ncolors)])[:,0:3]
colors = np.array([(int(r * 255), int(g*255),int(b*255)) for r, g, b in colors])


image = Image.new('RGB', (width, height)) #Image init
draw = ImageDraw.Draw(image) # Init ImageDraw


for circle in range(circles):
    ulx = np.random.randint(0, width)
    uly = np.random.randint(0, height)
    random_offset = np.random.randint(5,100)
    lrx = ulx + random_offset
    lry = uly + random_offset
    color = tuple(colors[np.random.randint(0,ncolors),:])
    if np.random.randint(0,2):
        draw.ellipse((ulx, uly, lrx, lry), fill = color, outline = color)
    else:
        draw.ellipse((ulx, uly, lrx, lry), fill = None, outline = color)
    # Upper left coords, then lower right coords

image = image.resize((width // 2, height // 2), resample=Image.Resampling.LANCZOS)

image.save("out.png")
```