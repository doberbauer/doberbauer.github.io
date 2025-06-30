Title: Mean Shift Rotoscoping Part 2
Date: 2023-08-07 00:00:00
Category: Art

In Part 1 of Mean Shift Rotoscoping I showed how OpenCV's Mean Shift Filter can be used to produce a rotoscoping effect. At the end of that post I guessed that Amazon's *Undone* used this technique as part of their rotoscoping pipeline. I have since found *numerous* [articles](https://www.indiewire.com/awards/industry/undone-amazon-prime-video-rotoscope-1202238213/) and [interviews](https://www.youtube.com/watch?v=J9sYE9tIwTE) explaining how the creative teams behind the show achieved its signature visual style. Mean shift filtering wasn't mentioned. 

That being said, I think it's an interesting approach and I wanted to improve upon it. Mean Shift does a great job of smoothing out fine detail while preserving coarse detail, but I wanted to add some details back in to really lock in that rotoscoped style. The artists behind *Undone* clearly put a lot of emphasis on character's [facial expressions](https://deadline.com/wp-content/uploads/2019/06/screen-shot-2019-06-07-at-10.28.54-am.png?w=1024), using thick dark lines to define facial features much in the way [comic artists do](https://d1466nnw0ex81e.cloudfront.net/n_iv/600/651895.jpg). I figured I could achieve something similar with a little computer vision. For examples to follow I used [this](https://www.pexels.com/video/a-woman-standing-by-the-riverside-smiling-for-the-camera-3253738/) video from Pexels.

## Face Recognition 
I was searching GitHub for a good Eulerian video magnification Python implementation (for an upcoming project / blog post) and I stumbled upon [Face Recognition](https://github.com/ageitgey/face_recognition/tree/master), a lovely Python package for detecting and manipulating faces in video and images. The `face_landmarks` method caught my eye as it could recognize facial landmarks like eyes, nose, chin, mouth, eye brows, etc. It looked pretty dang cool so thought I'd gave it a whirl. Face Recognition requires Mac or Linux and unfortunately I had neither, nor did I have the patience to follow Face Recognition's  unofficial Windows installation guide. Instead I opted to set it up in a Google Colab notebook. I threw my existing mean shift script in, sprinkled in the code to detect and draw facial landmarks and let 'er rip. 90 minutes of free Google GPU compute time later and the results were... unimpressive. Don't get me wrong, Face Recognition was great: it was able to identify face landmarks and I could render them out to a mean shift rotoscoped frame. What didn't impressive me was how the landmark lines didn't flow very well. They looked blocky and obviously artificial. Each facial feature was essentially a list of points and the method I was using to draw them simply connected the dots iteratively. This was not ideal. If I wanted to even come close to the visual style of *Undone* I'd need to do better.

[See for yourself](https://youtu.be/K1OG2nO3hrg)

Here's the Google Colab notebook so you can try it yourself. 

[AutoRotoscope (Face Recognition - Blocky)](https://colab.research.google.com/drive/1mZdg8yb8EHEddoHiwENXHqrFHQYkqVIX?usp=sharing)


## Facial Recognition + NURBS

So I had a list of points but they were just being connected one after the next with lines. But with so few points, connecting them in this fashion looked like the 2D equivalent of a [low polycount character from a PS1 era game](https://imgur.io/89p2Gzx?r). What I needed was more points. 
After much head-scratching and foolish attempts to generate and interpolate along splines and bezier curves, I was eventually reminded of NURBS. Non-Uniform Rational B-Splines or NURBS (I like NURBS better — it's like the [Ermahgerd Gersberms](https://knowyourmeme.com/memes/ermahgerd) girl [saying 'noobs'](https://chat.openai.com/share/50675628-1a3d-4524-b991-699dd552855b)) are great for representing surfaces in 2D and 3D space. I could build a NURBS curve from sparse points and then get however many points I wanted interpolated along the surface. With enough points the eye won't even see the individual straight lines, it'll just see a nice organic curve.
A quick search introduced me to [geomdl](https://github.com/orbingol/NURBS-Python/tree/8ae8b127eb0b130a25a6c81e98e90f319733bca0), a terrific implementation of NURBS for Python. Implementing it was a breeze and it only took a few lines to create and interpolate along a NURBS curve. The results were an improvement from the raw `facial_landmarks` output. 

[See for yourself](https://youtu.be/hbS4ogziZ4g)

Here's another Google Colab notebook if you'd like to try it out. 

[AutoRotoscope (Face Recognition - NURBS)](https://colab.research.google.com/drive/1yTU9Yra7AcvOawIVahuXFhebvpTh-xVZ?usp=sharing)

## Facepalm 🤦‍♂️
I was still unsatisfied though. The NURBS curve improved how the facial landmarks were drawn, but they still just didn't look right. The effect looked forced and lacked artfulness. On top of that, when Face Recognition failed to detect a face the effect would disappear. Perhaps equally uncanny was when it misidentified facial landmarks and lines would be drawn where they didn't belong.
This was really not the application `facial_landmarks` was meant for so I decided to look for other solutions.
Then an obvious one occurred to me - why not just do edge detection? Use the original frame before rotoscoping, detect edges in it and overlay those edges onto the rotoscoped frame. It'd work regardless of there being a face in the frame and I imagined it'd be computationally much quicker to do.
Honestly when that thought occurred to me I was shocked how much time I sunk into Face Recognition when such an obvious solution was right in front of me. On the plus side, I learned about some cool packages and even though drawing facial landmarks didn't look great I did get it working in the end. 
Anyway, back to edge detection!
I looked like OpenCV's [Canny Edge Detection](https://docs.opencv.org/3.4/da/d22/tutorial_py_canny.html) would do the trick and hey — it was already in OpenCV which I was using for everything else! One less dependency! And a nice bonus for me — I could run it on my Windows machine again. The edges that come out of Canny are white and the background is black, but a quick bitwise_not operation to invert them would take care of that.
They were also quite thin and sometimes difficult to see, so I used OpenCV's [`dilate`](https://docs.opencv.org/3.4/db/df6/tutorial_erosion_dilatation.html) function to thicken them up a bit. `dilate` works on white pixels so this had to happen before inverting. The results, in my opinion, looked a **hell** of a lot better than anything that came out of Face Recognition and certainly better than mean shift alone. [See for yourself](https://youtu.be/QIekVx3svRw).
I exposed the Canny Edge Detection thresholds and dilation kernel size to give more control over which edges are detected and how much they're thickened. 

You can play with it [here](https://colab.research.google.com/drive/1sKbdY6mbhxxu_fx7QwwQhg1e6TuRsTdD?usp=sharing). 