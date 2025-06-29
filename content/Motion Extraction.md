Title: Motion Extraction
Date: 2024-01-01 12:00
Category: Exploration

The excellent ["Motion Extraction" video by Posy](https://www.youtube.com/watch?v=NSS6yAMZF78) demonstrates how to extract motion from video.

## How does this work?
Posy describes this much better than I can so if you haven't watched his video go do that.

[Take me to the code](https://gist.github.com/hyperific/c6724a2f35f4ba1af50d2de043962509)
or
[Try on Google Colab](https://colab.research.google.com/drive/1hoIDW7fqt8QtWe-Mgb2Sr49CEehy75aG)

[Example (YouTube)](https://youtu.be/L-2cr1kdzMU)

Here's the step-by-step breakdown.

1. Take a stabilized video clip (could be stabilized with a tripod or with fancy stabilization tools)
2. Duplicate the clip
3. Offset the duplicate by some number of frames
4. Invert the duplicate
5. Set the inverted duplicate and the original clip to 50% opacity
6. Combine the clips


I chose to use OpenCV to do this so things work slightly different.
As far as I know OpenCV only allows you to operate on frames of a video, not whole clips. I'm looking into doing a MoviePy implementation but I can't figure out the compositing bit. If anyone knows how to composite clips together in MoviePy please get in touch.

Here's how I did it.

1. Get the number of frames in the video
2. Loop through each frame, grabbing that frame and the next frame in the series defined by the offset. If the offset is 1 then we get frames 1 and 2. If it's 5 then we get frames 1 and 5.
3. Invert the offset frame using `cv2.bitwise_not`. If a pixel value in the input is 0 it'll be inverted to 255. Likewise if a pixel is 255 it'll be inverted to 0.
4. Combine the frames using `cv2.addWeighted`. Conveniently this allows you to set the opacity for each clip.
5. Write the composited frame to the output video container.





```Python
import cv2
import os

# User parameters
video_path = r"" #Where your video lives
offset = 12 #How many frames difference
mode = 1

#Define the capture target
cap = cv2.VideoCapture(video_path)

# Capture properties
frame_count = int(cap.get(cv2.CAP_PROP_FRAME_COUNT))
frame_width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
frame_height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
fps = cap.get(cv2.CAP_PROP_FPS)

#File handling junk
directory = os.path.dirname(video_path)
filename = os.path.splitext(os.path.basename(video_path))[0]
new_filename = f"{filename}_motion_extracted_mode{mode}_offset{offset}.avi"
output_file = os.path.join(directory, new_filename)

#Output file definition
fourcc = cv2.VideoWriter_fourcc(*'XVID')
out = cv2.VideoWriter(output_file, fourcc, fps, (frame_width, frame_height))

if mode == 1:
#Must subtract the offset from the framecount or code below will try to index
# frames that don't exist.
    for i in range(0,frame_count-offset):
        cap.set(cv2.CAP_PROP_POS_FRAMES, i) #Get the initial frame
        ret, frame_a = cap.read()
        if not ret:
            continue
        
        cap.set(cv2.CAP_PROP_POS_FRAMES, i+offset) # Get the next frame in the series
        ret, frame_b = cap.read()
        if not ret:
            continue
        
        inverted = cv2.bitwise_not(frame_b)
        composite = cv2.addWeighted(frame_a, 0.5, inverted, 0.5, 0)
        
        out.write(composite)
        
elif mode == 2: #Compare every frame to "offset" frame
    cap.set(cv2.CAP_PROP_POS_FRAMES, offset) # Get the next frame in the series
    ret, start_frame = cap.read()
    for i in range(1,frame_count):
        cap.set(cv2.CAP_PROP_POS_FRAMES, i)
        ret, frame = cap.read()
        if ret:
            inverted = cv2.bitwise_not(frame)
            composite = cv2.addWeighted(start_frame, 0.5, inverted, 0.5, 0)
        
            out.write(composite)
cap.release()
out.release()

```