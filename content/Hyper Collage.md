Title: Hyper Collage
Date: 2024-01-01 12:00
Category: Art

The NFT boom of 2021 opened my eyes to a whole world of generative art. While some projects were more about the memes than the art, some really neat things were being done with p5js, Processing, even Turtles. 
They used programming and math to make beautiful procedural art and it actually looked like *art*. I liked the idea of doing something like that but the projects I worked on were more like paper dolls than abstract modern art. 
Those projects ultimately flopped and by the next year the bottom had completely fallen out of NFTs. Nonetheless I still loved the idea of designing a script that generates art. 

At the time I was getting interested in web scraping a had come across the icrawler Python package. I had the idea of scraping a bunch of images and making them into a collage. It was to be a daily series where the images were pulled from the day's top articles. I'd use icrawler to download images, PIL to arrange them randomly and a instabot to upload them to Instagram. The former and latter parts didn't work quite as I'd planned but overall it worked so I'd call it a success.

```Terminal
pip install icrawler
pip install pillow
pip install instabot
```

```Python
from datetime import date, datetime, timedelta
from icrawler.builtin import GoogleImageCrawler
import os
import time
from PIL import Image
import random as rnd
from instabot import Bot
import sys
# Parameters

today = str(date.today())

today2 = datetime.now().strftime(r"%B %d, %Y")
classes = [today, today2]
number=100
out_size = (2160, 2160)
out_folder = 'Downloads/'

for c in classes:
    googlecrawler = GoogleImageCrawler(storage={'root_dir':out_folder})
      googlecrawler.crawl(keyword=c,filters=None,max_num=number,offset=0)

imgs = os.listdir(out_folder)
out = Image.new(mode="RGB", size=out_size, color=(0,0,0))

for i in range(1, len(imgs)):

    pWide= int(out_size[0]/len(imgs)*10)

    img = Image.open(out_folder + imgs[i])

    wper =(pWide/float(img.size[0]))

    hsize = int((float(img.size[1] * float(wper))))

    img = img.resize((pWide,hsize), Image.ANTIALIAS)

    posX = rnd.randrange(0,int(out_size[0]-img.size[0]))

    posY = rnd.randrange(0,int(out_size[1]-img.size[1]))

    out.paste(img, (posX, posY))

out.save(today + '.jpg', 'JPEG')
```

I left out the instabot portion because every time it attempted to upload Instagram would block the action and send me an alert that someone was breaking into my account.

The search classes for the day's top images could have been improved for sure. I could have used the Google News API to pull headlines from top stories and use those as icrawler search classes instead of variations of the day's date. 


# Results
**2022-02-01**
![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1705964577-0.jpg)
**2022-02-05**
![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1705964577-1.jpg)
**2022-02-09**
![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1705964577-2.jpg)


