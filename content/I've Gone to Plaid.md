Title: I've Gone to Plaid
Date: 2023-09-20 09:36:20
Category: Art

This is an exercise I did to keep myself busy while processing data. 

It's a procedural plaid pattern generator written in Python. It produces a Gingham, Buffalo or plaid pattern with procedural colors and design elements.

[Just show me the code](https://github.com/hyperific/gone_to_plaid)

![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1706130144-0.png)

# Gingham and Buffalo
Buffalo plaid is characterized by a colored background (usually red) and black crisscrossing lines, forming solid squares where they intersect.
![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1705965396-0.png)

Gingham is similar except the background is always white and the lines are variable colors (usually red).
![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1705965482-0.png)

I'm not a strict disciple of the church of "Don't Repeat Yourself" so the functions for gingham and buffalo plaid look pretty much the same except for the color generation handling. 

I could have made buffalo and gingham using intersecting lines or rectangles but instead of opted to make the pattern out of squares.
To make squares I used PIL ImageDraw's `rectangle()` function.
It needs a tuple `shape`, and an RGB color tuple `fill`. Shape is (x1,y1,x2,y2), or top left x coordinate, top left y coordinate, bottom right x coordinate, bottom right y coordinate. 

First I get 20 linearly spaced values from 0 to the width of the image. These will act as the X and Y coordinates of each rectangle's top left corner.
Since the spacing is variable I also grab the second value of the spacing array and save it so I can calculate the bottom right corner. 
I loop through rows first, then columns, so the pattern is build from left to right and top to bottom. The shape becomes `(spacing[col],spacing[row],spacing[col]+dif,spacing[row]+dif)`
For row 1 column 1 with a difference in spacing of 50 pixels the shape would be `(0,0,0+50,0+50)`.

Now let's talk colors. Every time you call the gingham or buffalo function it generates the pattern with a unique randomly chosen color. I opted for HSL colors since they're intuitive and it's easy to convert them to RGB later.
First, a base hue is chosen as a random angle between 0 and 359 degrees of the color wheel. From this base hue all the colors of the pattern will be derived. Buffalo colors differ only by luminance, gingham patterns differ only by saturation. As the loop proceeds through rows and columns colors are chosen based on whether the column and row numbers are even or odd.

# Plaid
Plaid is where things go off-script. There are *loads* of different plaid variations.

![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1705965550-1.jpg)

![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1705965550-0.png)

Unlike buffalo and gingham, here I *did* decide to use intersecting lines.
I'll try my best to keep this from getting too convoluted but maybe grab another cup of coffee just in case.

My plaid patterns are made up of three basic elements: 
- Thick lines
- Medium lines
- Thin lines
There are also two optional elements:
- Boxes at intersections
- Additional thin horizontal or vertical line

The number, thickness, color and presence of some lines is randomized. 

As for colors, first the saturation for most of the colors is determined randomly.
The background and box hues are determined separately as values between 0 and 359. Then there's a possibility of having other colors be analogous or complementary by making an array `degree_diff = [30,-30,180,-180]`, randomly indexing into it and hen subtracting that value from other hues.
Once the HSV colors are determined they're converted to RGB. Thin lines have randomly determined luminance values, but their hue and saturation are static. 

There are some randomly determined offsets for lines and boxes and whatnot and those end up trickling down and influencing the positions of sub-elements.
Once all the elements are drawn we end up with 1/4 of the final image. I figured why bother with the extra hassle when I could just flip and mirror the image? It's the same information just transformed a bit. 

At the end there's a coin flip to determine if thin diagonal lines with be drawn over the top of the image to create a fabric-like texture.



# Code
``` Python
from PIL import Image, ImageDraw, ImageOps
import numpy as np
import colorsys
import uuid
import argparse

def main(mode):
    if mode == 'plaid':
        _plaid()
    elif mode == 'buffalo':
        _buffalo()
    elif mode == 'gingham':
        _gingham()


def _gingham(w=1000,h=1000):
    # Gingham is defined by a white background and evenly spaced colored lines. 
    # Horizontal and vertical lines have different saturation values
    # Where lines intersect the color is darker still
    # In this implementation I've chosen to use rectangles instead of lines.

    image = Image.new("RGBA", (w,h), "white")
    draw = ImageDraw.Draw(image)
    

    spacing = np.linspace(0,w,20)
    dif = spacing[1] #This is the difference in spacing between each value of spacing
    
    # The hue is generated randomly as a value between 0 and 359 (the color wheel being 360 degrees)
    # This value is divided by 100 since colorsys represents hsv as percentages
    hue = np.random.randint(0,359)/100 
    
    
    # Colors for background, and the 3 different colored boxes are generated
    color1 = _hsv2rgb(hue,0,1)
    color2 = _hsv2rgb(hue,0.5,1)
    color3 = _hsv2rgb(hue,0.25,1)
    color4 = _hsv2rgb(hue,0.75,1)

    # Rows and columns are looped through. Colors are alternated based on whether the row
    # index is even or odd.

    for row in range(len(spacing)):
        if row%2 == 0:
            colors = [color1,color2]
        else:
            colors = [color3,color4]
        for col in range(len(spacing)):
            shape = (spacing[col],spacing[row],spacing[col]+dif,spacing[row]+dif)
            draw.rectangle(shape,fill=colors[col%2])

    # It's randomly decided whether to additionally draw diagonal lines across the whole image
    # These diagonal lines are usually present in some fashion in plaid patterns
    if np.random.randint(2):
        dlines = _diagonal_lines(w*4,h*4,color="white",n=500)
        image.paste(dlines,(0,0),dlines)
    image.show()
    _save_pattern(image)
    

def _buffalo(w=1000,h=1000):
    # Buffalo plaid is similar to gingham. It's defined by a colored background and
    # black/gray lines. Where the lines intersect they're usually darker.
    # Like gingham, horizontal and vertical lines typically have different intensities.
    # In this implementation I've chosen to use rectangles instead of lines.

    image = Image.new("RGBA", (w,h), "white")
    draw = ImageDraw.Draw(image)
    
    spacing = np.linspace(0,w,20)
    dif = spacing[1] #This is the difference in spacing between each value of spacing
    
    # The hue is generated randomly as a value between 0 and 359 (the color wheel being 360 degrees)
    # This value is divided by 100 since colorsys represents hsv as percentages
    hue = np.random.randint(0,359)/100
    
    # Colors for background, and the 3 different colored boxes are generated
    color1 = _hsv2rgb(hue,1,1)
    color2 = _hsv2rgb(hue,1,0.5)
    color3 = _hsv2rgb(hue,1,0.25)
    color4 = _hsv2rgb(hue,1,0)

    # Rows and columns are looped through. Colors are alternated based on whether the row
    # index is even or odd.

    for row in range(len(spacing)):
        if row%2 == 0:
            colors = [color1,color2]
        else:
            colors = [color3,color4]
        for col in range(len(spacing)):
            shape = (spacing[col],spacing[row],spacing[col]+dif,spacing[row]+dif)
            draw.rectangle(shape,fill=colors[col%2])

    # It's randomly decided whether to additionally draw diagonal lines across the whole image
    # These diagonal lines are usually present in some fashion in plaid patterns   
    if np.random.randint(2):
        dlines = _diagonal_lines(w*4,h*4,color=color1,n=500)
        image.paste(dlines,(0,0),dlines)
    image.show()
    _save_pattern(image)

def _plaid():
    # Plaid patterns are diverse. This implementation uses rules which generate only
    # a subset of the plaid patterns that exist in the wild. 
    
    # Background and major element colors are generated randomly
    # All other elements are procedurally generated to be harmoneous
    # with major element colors. They can be complementary or analogous.

    # Height and width here describe the initial tile dimensions
    # This tile will be copied 3 times, resulting in an image whose
    # dimensions are (w*4,h*4)
    w = 250 
    h = 250


    sat = np.random.random() # Random saturation value between 0 and 1

    #Background and box hues are randomly generated integers between 0 and 359 (colorwheel being 360 degrees)
    bkg_hue = np.random.randint(0,359)
    box_hue = np.random.randint(0,359)
    
    # This is a list of possible colorwheel degree differences
    # 30 and -30 are analogous colors
    # 180 and -180 are complementary colors
    degree_diff = [30,-30,180,-180]

    # Thick line hues are defined by the box hue minus a randomly decided degree difference
    thick_hue = box_hue-degree_diff[np.random.randint(4)]
    
    # RGB colors are calculated from the above HSV.
    bkg_color = _hsv2rgb(bkg_hue/100,sat,0.8)
    box_color = _hsv2rgb(box_hue/100,sat,0.5)
    thick_color = _hsv2rgb(thick_hue/100,sat,0.60)
    
    # Thin lines can be 95%, 60% and 0.5% bright
    # Brightness is randomly picked from the list.
    thin_vals = [0.95,0.60,0.05]
    thin_color = _hsv2rgb(thick_hue/100,0.15,thin_vals[np.random.randint(2)])
    
    img = Image.new("RGBA",(w,h),bkg_color)
    draw = ImageDraw.Draw(img)
    
    line_offset = np.random.randint(25,50)
    line_offset -= line_offset % 5
    
    box_offset = np.random.randint(20,40)
    if box_offset %2 !=0:
        box_offset +=1

    vline = (line_offset+box_offset/2,0,line_offset+box_offset/2,h)
    draw.line(vline,fill=thick_color,width=box_offset+1)

    hline = (0,line_offset+box_offset/2,w,line_offset+box_offset/2)
    draw.line(hline,fill=thick_color,width=box_offset+1)
    
    if np.random.randint(2):
        vline1 = (line_offset-box_offset,0,line_offset-box_offset,h)
        draw.line(vline1,fill=thick_color,width=int(box_offset/4+1))
        
        vline2 = (line_offset+box_offset*2,0,line_offset+box_offset*2,h)
        draw.line(vline2,fill=thick_color,width=int(box_offset/4+1))

        hline1 = (0,line_offset-box_offset,w,line_offset-box_offset)
        draw.line(hline1,fill=thick_color,width=int(box_offset/4+1))
        
        hline2 = (0,line_offset+box_offset*2,w,line_offset+box_offset*2)
        draw.line(hline2,fill=thick_color,width=int(box_offset/4+1))
    
    # Coinflip whether to draw boxes at line intersections
    if np.random.randint(2):
        box = (line_offset,line_offset,line_offset+box_offset,line_offset+box_offset)
        draw.rectangle(box,fill=box_color)

    # Coinflip whether to draw an additional line
    # Coinflip whether the line is drawn horizontal or vertical
    if np.random.randint(2):
        direction = np.random.randint(2)
        if direction:
            extra_line = (0,line_offset*2+box_offset*2,w,line_offset*2+box_offset*2)
        else:
            extra_line = (line_offset*2+box_offset*2,0,line_offset*2+box_offset*2,h)
        draw.line(extra_line,fill=thick_color,width=(int(box_offset/4)))

    # Additional thin horizontal and vertical lines are drawn
    # The number of lines is randomly chosen between 3 and 5
    # There's an additional coinflip to add additional spacing to the line offsets
    num_lines = np.random.randint(3,6)
    if num_lines %2 !=0:
        num_lines +=1  
    spacing = np.round(np.linspace(0-box_offset/2,box_offset+box_offset/2,num_lines)).astype(int)
    if np.random.randint(2):
        line_offset *=4
    for l in range(num_lines):
        thin_vline = (line_offset+spacing[l],0,line_offset+spacing[l],h)
        draw.line(thin_vline,fill=thin_color,width=int(spacing[1]/3))
        
        thin_hline = (0,line_offset+spacing[l],w,line_offset+spacing[l])
        draw.line(thin_hline,fill=thin_color,width=int(spacing[1]/3))
        

    flipped = ImageOps.flip(img)
    img2 = Image.new("RGB",(w,h*2))
    img2.paste(img,(0,0))
    img2.paste(flipped,(0,h))
    
    mirrored = ImageOps.mirror(img2)
    img3 = Image.new("RGB",(w*2,h*2))
    img3.paste(img2,(0,0))
    img3.paste(mirrored,(w,0))
    
    tile = Image.new("RGB",(w*4,h*4))
    tile.paste(img3,(0,0))
    tile.paste(img3,(w*2,0))
    tile.paste(img3,(0,h*2))
    tile.paste(img3,(w*2,h*2))
    
    # Coinflip whether to additionally draw diagonal lines across the whole image
    # These diagonal lines are usually present in some fashion in plaid patterns 
    if np.random.randint(2):
        dlines = _diagonal_lines(w*4,h*4,color=bkg_color,n=250)
        tile.paste(dlines,(0,0),dlines)
    tile.show()
    _save_pattern(tile)
    
    

def _hsv2rgb(h,s,v):
    # Utility function that wraps colorsys.hsv_to_rgb
    return tuple(round(i * 255) for i in colorsys.hsv_to_rgb(h,s,v))

def _diagonal_lines(w,h,n=40,color=(255,255,255)):
    # Utility function to draw diagonal lines present in plaid patterns
    xspacing = np.linspace(0,w,n)
    yspacing = np.linspace(0,h,n)
    img = Image.new("RGBA",(w,h),(0,0,0,0))
    draw = ImageDraw.Draw(img)
    
    for i in range(n):
        shape = (xspacing[i],0,w,h-yspacing[i])
        draw.line(shape,fill=color,width=1)
        shape = (0,yspacing[i],w-xspacing[i],h)
        draw.line(shape,fill=color,width=1)
    return img

def _save_pattern(image):
    # Utility function to generate a random unique filename and save the image
    
    # Generate a random UUID (Universally Unique Identifier)
    random_uuid = uuid.uuid4()

    # Create a random file name by converting the UUID to a string
    random_file_name = str(random_uuid)

    # You can add a file extension if needed
    random_file_name_with_extension = random_file_name + '.png'
    image.save(random_file_name_with_extension)
    
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        prog='Gone to Plaid',
        description='A plaid pattern generator.')
    parser.add_argument(
    '--mode',
    '-m',
    type=str,
    choices=['buffalo', 'gingham','plaid'],
    help='Pattern type to generate.',
    default='plaid'
    )
    
    args = parser.parse_args()
    main(args.mode)
```