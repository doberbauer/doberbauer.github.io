I wrote a quick Python script that recreates an image using thousands of randomly placed squares. The square color is determined by taking the mean color value of the original image that's covered by the square. Square size is randomly determined between a max and a min. To ensure there aren't any gaps between boxes the number of boxes is determined mathematically as the midpoint between the number of boxes needed if all of them were the max size and if all of them are the min size. 

Unsurprisingly, wrecktangles acts as a form of *incredibly* lossy compression. Increasing the min and max box sizes increases the compression ratio.


Original (6.75 MB)
![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1706045452-0.jpg)
Wrecktangle min 5, max 20 (4.30 MB)
![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1706054626-0.png)
Wrecktangle min 10, max 75 (1.47 MB)
![Hyperific's blog](https://bear-images.sfo2.cdn.digitaloceanspaces.com/hyperific-1706054557-0.png)

For fun I wrote the script using OpenCV and again using PIL.
## [OpenCV version](https://gist.github.com/hyperific/a624df6cb12640ed20b0cc228dc4dbc7)
``` Python
import cv2
import numpy as np
def wreck(path,r1,r2):
    img = cv2.imread(path)
    out = np.zeros(img.shape,dtype=np.uint8)
    w = img.shape[1]
    h = img.shape[0]
    for b in range(int(((w*h / r2) + (w*h / r1)) /2)):
       boxsize = np.random.randint(r1,r2)
       x1 = np.random.randint(0,w-boxsize)
       x2 = x1+boxsize
       y1 = np.random.randint(0,h-boxsize)
       y2 = y1+ boxsize
       mean = (np.mean(img[y1:y2,x1:x2,0]), np.mean(img[y1:y2,x1:x2,1]), np.mean(img[y1:y2,x1:x2,2]))
       out = cv2.rectangle(out, (x1, y1), (x2, y2), mean, -1)
    cv2.imwrite("out.png", out)
```

## [PIL version](https://gist.github.com/hyperific/6b8544f8e4bb0ad6af6f78c2e4420fb8)
``` Python
from PIL import Image, ImageDraw
import numpy as np
def wreck(path,r1=5, r2=21):
    img = np.array(Image.open(path))
    out = Image.new('RGB', (img.shape[1],img.shape[0]))
    draw = ImageDraw.Draw(out) # Init ImageDraw
    w = img.shape[1]
    h = img.shape[0]
    for b in range(int(((w*h / r2) + (w*h / r1)) /2)):
       boxsize = np.random.randint(r1,r2)
       x1 = np.random.randint(0,w-boxsize)
       x2 = x1+boxsize
       y1 = np.random.randint(0,h-boxsize)
       y2 = y1+ boxsize
       mean = (int(np.mean(img[y1:y2,x1:x2,0])), int(np.mean(img[y1:y2,x1:x2,1])), int(np.mean(img[y1:y2,x1:x2,2])))
       draw.rectangle(((x1, y1), (x2, y2)), fill=mean)
    out.save("out.png")

```