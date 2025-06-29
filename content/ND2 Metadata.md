# ND2 Metadata


This morning I was faced with the task of opening 114 microscope image files, opening up the image properties and jotting down the image width and height. 
To make matters worse, the program to open those files can only be used if you are

A. physically at your computer and have a HASP key plugged in

or 

B. using Google Chrome's remote desktop extension, which is painfully slow and sometimes just stops working for no apparent reason

I have been dabbling with the [nd2](https://pypi.org/project/nd2/) package to automate some other tedious tasks like grabbing excitation wavelengths, channel names, image dimensions and other bits of metadata. I wrote a little script using this gem of a package to fetch the metadata from every file, index into it, fetch the properties I want and append them to a list.

``` Python
paths = [ <all_the_paths> ] 
width = []
height = []
for path in paths:
    f = nd2.ND2File(path) # load image
    meta = f.unstructured_metadata()
    width.append(meta[""ImageAttributesLV""][""SLxImageAttributes""][""Width""])
    height.append(meta[""ImageAttributesLV""][""SLxImageAttributes""][""Height""])
```

It took a few minutes to write the script.
It took roughly 5 seconds to run.

Just for fun I timed how long it took to open a single file in the image viewer program, open up the image properties window and jot down the metadata. It took 21.26 seconds from start to finish, mostly due to the dimensions property not being selectable in the program so I couldn't just copy/paste I had to type it out. Assuming I was working at a constant rate it would have taken me 39.9 minutes to get the dimensions for all 114 files.

The time I saved has been repurposed to write this post. 