For [Genuary 5](https://genuary.art/prompts#jan5) the prompt is *"In the style of Vera Molnár (1924-2023)."*

[Vera Molnár](https://en.wikipedia.org/wiki/Vera_Moln%C3%A1r) was a pioneer of computer generated artwork. I decided to make my #genuary5 submission as a recreation of [1% de Desordre – Bleu + Rouge (1979)](https://backend.artreview.com/wp-content/uploads/2022/06/1-per-Disorder-%E2%80%93-Blue-and-Red-1230x1135.jpg), with an altered color scheme. As it turns out, what I made was actually closer to [1% de désordre (1976)](https://www.sothebys.com/en/buy/auction/2022/natively-digital-1-3-generative-art/1-de-desordre-1-of-disorder). In keeping with the style of the original, my final image is a quadriptych. I also made it into an animated gif, looping over 10 variations. I'm calling it "Désordre bronzé et noir" or "tan and black mess".

![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1706210382-0.png)

![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1706210382-1.gif)

Here's how it works.

- First set the value for the height and width of the image (it's always 1:1).
- Next set the number of columns and rows you want. Yes it's hard coded, get over it. 
- Set the background and line colors. Here I use tan for the background and a dark gray color for the lines.
- `sqsz` is the square size, which is the size in pixels of each grid cell. I probably should have named it `cellsz`...
- `spacing` is the square root of the square size. When the squares are drawn there will be this much spacing between the square and the edge of the cell. `2 * spacing` is technically the true spacing. 
- `inners` is the number of possible internal squares given `sqsz` and `spacing`. 
- `cellstarts` is the starting (upper left) X and Y coordinates for each square.
- `cellends` is the ending (lower right) X and Y coords

The outermost loop runs 10 times, generating 10 image variations. You can set it to whatever you want. I wanted ten for my animated gif.
In the inner loop the PIL `Image` and `ImageDraw` objects are initialized and variations are calculated. I set things up to avoid a bunch of nested `if` statements. Not that performance was a critical factor for me, I just wanted to challenge myself to make this with as few `if` statements as possible. 

- `widths` is an array of line widths that's iterated through as each column and row is built out. All widths start out as `int(spacing/2)` but 3 width values are randomly chosen to be thinner or thicker.
- `mult` is another array that's iterated through. When the outer box is drawn its width is multiplied by `mult`. The randomly chosen boxes whose widths are different will have their outer width set to zero, effectively erasing them. 
- `submult` is similar to `mult` except it only affects the inner squares, and only the cell with the thinnest lines.

Once all these parameters are set the row and column loops begin and the boxes are drawn. 
The last step just adds a bit of trim all the way around by pasting the image onto one that's slightly bigger.

```Python
import numpy as np
from PIL import Image, ImageDraw

dim = 2000 # Height and width of the image
ncell = 10 # Number of grid columns and rows

bkg = (210,208,193) #Background color
line = (74,72,64) # Line color

sqsz = int(dim/ncell) # Size in pixels of each grid cell
spacing = int(np.sqrt(sqsz))
inners = int(sqsz / spacing / 4)
cellstarts = np.arange(0+spacing,dim+spacing,sqsz)
cellends = np.arange(sqsz-spacing,dim+sqsz-spacing,sqsz)
for i in range(10):
    image = Image.new('RGB', (dim,dim),color=bkg) #Image init
    draw = ImageDraw.Draw(image) # Init ImageDraw
    
    widths = np.ones((ncell,ncell),dtype=int) * int(spacing/2) 
    spx = np.random.choice(range(ncell),3,replace=False)
    spy = np.random.choice(range(ncell),3,replace=False)
    
    widths[spx[0],spy[0]] = int(spacing/4)
    widths[spx[1],spy[1]] = int(spacing/3)
    widths[spx[2],spy[2]] = int(spacing)
    
    mult = np.ones((ncell,ncell),dtype=int) #Multiplies the special cells widths by 0
    mult[spx,spy] = 0
    
    submult = np.ones((ncell,ncell),dtype=int)
    submult[spx[0],spy[0]] = 2

    for rw in range(len(cellstarts)):
        for col in range(len(cellstarts)):
            width = widths[rw,col]
            m = mult[rw,col]
            draw.rounded_rectangle([cellstarts[col],cellstarts[rw],
                            cellends[col],cellends[rw]],
                           fill=None,outline=line,width=width*m,radius=6)
            for inner in range(submult[rw,col],inners):
                subsp = int(spacing*2*inner+1)
                draw.rounded_rectangle([cellstarts[col]+subsp,cellstarts[rw]+subsp,
                               cellends[col]-subsp,cellends[rw]-subsp],
                               fill=None,outline=line,width=width,radius=6)
    bigger_image = Image.new('RGB',(dim+sqsz,dim+sqsz),color=bkg)
    bigger_image.paste(image,(int(sqsz/2),int(sqsz/2)))
    bigger_image.save(f"{i}.jpg")

```