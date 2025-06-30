Title: Multi Image Steganography
Date: 2023-06-22 20:24:32
Category: Exploration



I have been wanting to do a steganography project of my own since I picked up my first programming language a few years ago.

Where most steganography projects I've come across aim to secretly embed the entire message in a single image, I wanted to create a method that would encode a message across multiple images. You would need to have the entire collection of images to assemble the complete message.
I also knew I wanted to do direct pixel manipulation rather than modifying metadata or something like that. 
I also wanted to encode as much information as possible using the fewest pixels so the message would be difficult to detect.

Each pixel of a JPG image file has 3 channels: red, green and blue. Each channel can have a value ranging from 0 to 255. 
There are 128 standard unicode characters plus extra non-standard characters as well. 

Given that the message spans multiple images, it's important to also include an index denoting the order the information hidden in any particular image fits with information from others. 

How many pixels will we need per image?

- There are three color channels per pixel (RGB)
- We need a sort index + n characters encoded as integers 
- Typical word length is 5 letters 
- Typical sentence length is 15-20 words. 

I decided that two pixels would be sufficient. The first pixel would carry the sort index and two characters. The second pixel would carry 3 characters.
If the message length in characters was not divisible by 5 then the remaining characters would be filled with whitespace.

I'm using the first and last pixels in the image, but really any two pixels will do. Choosing pixels away from the edge would prevent loss of encoded message content via cropping.

Unicode characters are converted to integers with `ord()`. 

The decoder converts integers to Unicode characters using `chr()`.

## The Code
```Python
def encode(imagesFolder, message, outDir):
   import numpy as np 
   from PIL import Image 
   import os
   import math 
   imagePaths = os.listdir(imagesFolder) 
   imgCount =0
   locA = (0,0)
   for i in np.arange(0,len(message),5):
   	if i+5>len(message):
   		chunk = message[i:len(message)]
   		d = i+5-len(message)
   		for a in range(d):
   			chunk=chunk + ' '
   	else:
   		chunk = message[i:i+5]
   	pixA = (imgCount,ord(chunk[0]),ord(chunk[1]))
   	pixB = (ord(chunk[2]),ord(chunk[3]),ord(chunk[4]))
   	img = Image.open(imagesFolder + '/'+imagePaths[imgCount])
   	w,h= img.size
   	locB = (w-1,h-1)
   	img.putpixel(locA,pixA)
   	img.putpixel(locB,pixB)
   	img.save(imagesFolder +'/' + outDir + '/' + str(imgCount)+'.png')
   	imgCount += 1
def decode(imagesFolder):
	import os
	from PIL import Image
	import numpy as np
	locA=(0,0)
	imagePaths = os.listdir(imagesFolder)
	imgIndices = []
	chunks =[]
	for i in range(len(imagePaths)):
		img = Image.open(imagesFolder+'/'+imagePaths[i])
		w,h = img.size
		locB = (w-1,h-1)
		pixA = img.getpixel(locA)
		pixB = img.getpixel(locB)
		imgIndices.append(pixA[0])
		chunk = chr(pixA[1])+chr(pixA[2])+chr(pixB[0])+chr(pixB[1])+chr(pixB[2])
		chunks.append(chunk)
	imgIndices=np.array(imgIndices)
	chunks=np.array(chunks)
	sort = np.argsort(imgIndices)
	message = "".join(chunks[sort])
	return message
```
## Pros 
- Only modifying two pixels so the file size should stay very similar to the original carrier. 
## Cons
- Cropping the image may remove one or both of the message-carrying pixels.
- Compression may alter one or both of the message-carrying pixels.
- The pixel's color may be obviously different than pixels around it, tipping off the keen observer to the presence of hidden content.
- Long messages would require more images or more message-carrying pixels per image

[Github link to code](https://github.com/hyperific/steganography-tools/blob/main/multi-image.py)