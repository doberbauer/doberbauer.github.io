Title: Mean Shift Rotoscoping Part 1
Date: 2024-01-01 12:00
Category: Art

My wife and I watched a few episodes of *Undone* the other night on Prime Video. What struck me immediately, other than Rosa Salazar's authentic performance, was the [visual style](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Ftvseriesfinale.com%2Fwp-content%2Fuploads%2F2019%2F11%2Fundone.jpg&f=1&nofb=1&ipt=c9411f208ec31e40c62835bffc52498768b1fef52a265f580896b565bc3ff151&ipo=images). I've seen rotoscoped films before like [Waking Life](https://en.wikipedia.org/wiki/Waking_Life) and [A Scanner Darkly](https://en.wikipedia.org/wiki/A_Scanner_Darkly_(film)) (both Richard Linklater films), but this was noticeably different. When you rotoscope entirely by hand there are noticeable [frame-to-frame inconsistencies](https://youtu.be/xX10vQEa56E) in texture, line direction, stroke length and width, etc that make the whole frame [feel alive](https://youtu.be/hkjDUERgCQw?t=26) (look at this guy's shirt for example). *Undone* [didn't have those](https://youtu.be/6uWCNHQgfnc?t=10). I wondered how they could have achieved this and immediately suspected they had some computational help. I've been working with a lot of clustering algorithms for work and recently stumbled upon Mean Shift clustering / segmentation. I wondered if they used something like that to achieve the aforementioned effect in *Undone*. 

You're forgiven if you're unfamiliar with mean shift clustering. I only came across it recently myself. Imagine the input dataset as a landscape with peaks and valleys, where the height at each point represents the density of data points. The mean shift algorithm operates by placing a kernel at each data point and iteratively moving it towards the direction of steepest ascent, which corresponds to regions of higher density. As the kernel moves, it is influenced by the gradient of the density function, causing it to shift more significantly in regions with a higher concentration of nearby data points. When applied to image data you end up with this cool color blending effect that looks a bit like someone put a cartoon filter on reality.

I wanted to put this to the test so I did a little searching. I found that OpenCV has a mean shift filter and it does what I want. Per my usual ~~laziness~~ resourcefulness when it comes to flights of fancy, I turned to ChatGPT to do the work. Some embellishments have been made...

> Me: "ChatGPT, noble and benevolent AI overlord, please produce Python code to use OpenCV meanshift filtering on video"

> ChatGPT: "Certainly my kind and loyal underling, my favorite among the fleshlings! Here's an example of Python code that uses OpenCV's mean shift filtering on a video:"

``` Python
import cv2

def mean_shift_filtering(video_path):
    # Open the video file
    cap = cv2.VideoCapture(video_path)

    # Get the video properties
    frame_width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
    frame_height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
    fps = cap.get(cv2.CAP_PROP_FPS)

    # Create a VideoWriter object to save the filtered video
    output_file = 'filtered_video.avi'
    fourcc = cv2.VideoWriter_fourcc(*'XVID')
    out = cv2.VideoWriter(output_file, fourcc, fps, (frame_width, frame_height))

    # Loop through each frame of the video
    while cap.isOpened():
        ret, frame = cap.read()

        if not ret:
            break

        # Perform mean shift filtering
        shifted_frame = cv2.pyrMeanShiftFiltering(frame, sp=15, sr=30)

        # Display the filtered frame
        cv2.imshow('Filtered Frame', shifted_frame)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

        # Write the filtered frame to the output video file
        out.write(shifted_frame)

    # Release the video capture and writer objects
    cap.release()
    out.release()

    # Close all OpenCV windows
    cv2.destroyAllWindows()

    print("Mean shift filtering complete. Saved filtered video as", output_file)
```

I tried it on some stock video clips
- Pexels Hot air balloons [Original](https://www.pexels.com/video/couple-walking-down-the-road-towards-the-hot-air-balloon-festival-3018542/) | [Mean Shift Rotoscoped](https://youtu.be/HVqM6kZnpxQ)
- Pexels Cloudy skyline [Original](https://www.pexels.com/video/buildings-with-a-foggy-background-in-timelapse-mode-2015875/) | [Mean Shift Rotoscoped](https://youtu.be/uV73n_-wVRA)
- Pexels Lovey dovey couple [Original](https://www.pexels.com/video/a-couple-in-a-public-display-of-affection-3191106/) | [Mean Shift Rotoscoped](https://youtu.be/aE6Hzj72z8g)
- Timelapse I shot at Lake Tahoe [Original](https://youtu.be/lSzWJM8Xlls) | [Mean Shift Rotoscoped](https://youtu.be/e0vifOglOcw)

*Undone* **certainly** used multiple techniques to achieve their particular look, including 3D modeling, classical rotoscoping, etc. Based on what I've come up with I would bet that their visual processing pipeline involves mean shift filtering. If nothing else it's the first pass upon which artists would fill in details. 