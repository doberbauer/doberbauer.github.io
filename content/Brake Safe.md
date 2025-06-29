Title: Brake Safe
Date: 2024-01-01 12:00
Category: Project

In this post I want to talk about a project I started but couldn't finish. I figure if I write about it I can get some insights into why it failed and prevent myself from making the same mistakes or, better yet, fail more effectively. 
 
# Brake Safe
Maintaining safe braking distance is very important and where I live I see it being ignored and intruded upon *every* time I drive. Cars driving only a few feet behind the car in front of them, going 75 mph down the highway. 
I am a goody goody safe driver and it has paid off so far based on my non-existent accident record. 
I always want to be safer though and that's what lead me to this project idea.
My car isn't brand new, it's actually over 10 years old at this point, so it doesn't have the shiny new safety features like lane detection and correction, blind spot indicators or even a backup camera. I figured I could upgrade my safety features myself and learn some things along the way. 

My idea was to calculate the safe braking distance from my speed and then determine if I'm within that distance from the car in front of me. 

I knew going in that there'd be some significant constraints.

# Design Decisions
1. Cheap. While there's something to be said for learning all the things I needed to learn to accomplish my goal, it just didn't make sense to spend more on this than it would cost to buy something like it on Amazon.
2. Small. I didn't want my dashboard to be crowded up by a bunch of electronics. If I was going to build this I was going to build it small and unobtrusive. 
3. Reasonably accurate. This thing needed to work better than my eye but I wasn't kidding myself into thinking it needed to be accurate enough for the automotive industry or an autonomous vehicle or something. 
4. Computationally light-weight. Per #1, I was doing this on the cheap and per #2 it needed to be small so I locked myself into the Raspberry Pi ecosystem early on. Whatever was driving the computational side of things needed to be light enough that a humble Pi could manage it.

There were other constraints too but these are the core design tenets. 

## Calculating Safe Braking Distance
How do you calculate safe braking distance? Well it turns out it's more complicated than you might expect. As you may have guessed, the distance increases as your speed increases. What about the road conditions? Is it raining, snowing or a clear sunny day? Are you on an incline, decline, or flat area? How good are your brakes? How good are your tires? How much does your car weigh? What's your reaction time like? Are you rested and alert? 
All these factors play a role in the overall equation, and even then there's some margin for error. 
For my purposes (not going into an autonomous vehicle, not going into any kind of product assuring people's safety) I just need a rule of thumb that gets me in the ballpark. Fortunately there are a few rules of thumb for safe braking distance. The one I've chosen is (speed\10) * 3 + (speed\10)**2

## Speed
First thing's first, I need to determine my vehicle's speed.

### GPS
GPS is one way I could get speed. Get my coordinates at time 1, wait a second or 2 and get my coordinates at time 2. Calculate the distance traveled and it's simple rate = distance/time. I could get fancy and calculate the [great circle distance](https://en.wikipedia.org/wiki/Great-circle_distance) but that's overkill for this sampling period. Of the three possible solutions presented here, this one has probably the most significant drawback - if there's no GPS signal then I can't calculate my speed and the whole thing falls apart. 

### Optical flow? 
[Optical flow](https://en.wikipedia.org/wiki/Optical_flow) is "the pattern of apparent motion of objects, surfaces, and edges in a visual scene caused by the relative motion between an observer and a scene.". There's an optical flow sensor in your computer's mouse that tells the cursor which direction to move and how fast. While it might be possible to get a rough speed from optical flow, the rate at which things move on the camera depends on their distance to the camera. For instance if I was driving down a narrow tunnel the tunnel walls would be closer to me and the apparent optical flow rate would be quite fast. Conversely if I was on a salt flat where things are very far from me the apparent optical flow rate would be slow. I suppose if the camera was pointed at the ground then it wouldn't matter. It was fun to think about anyway. 

### OBD-II interface
Modern cars have computers and report vehicle statistics via OBD-II interface. OBD-II also provides voltage on [pin 16](https://pinoutguide.com/CarElectronics/car_obd2_pinout.shtml) so we might even be able to use that to power the Pi. It's too bad I was an idiot and failed to look up the voltage from pin 16 before attempting to power my Pi from it... 12V != 5V. In retrospect OBD-II was the best of the 3 options and I should have stuck with it. I was admittedly pretty disheartened from murdering a brand new Raspberry Pi because of sheer stupidity (and in the peek of the Raspberry Pi shortage no less). If you can solder up a new power regulator to a Raspberry Pi 3A+ please get in touch with me and I'll send you the damn thing for free.


I found a [cheap USB GPS](https://www.amazon.com/gp/product/B00NWEEWW8/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1) and decided to go that route.

## Distance
I needed to be able to measure the distance between my car and whatever is in front of me to determine if I was within safe braking distance. There are all kinds of ways to measure distance and they all have tradeoffs. The variables to balance here are accuracy, cost, range and practicality which is sort of a composite variable of "how hideous is the going to be on my car" and "how many advanced degrees will I need to get to make it work". Of these variables, accuracy is weighted the least, with cost, range and practicality being the driving factors. The range and accuracy assessments here are based on admittedly pretty surface-level research.

- RADAR: accurate, long range, expensive, impractical
- LiDAR: accurate, medium range, expensive, impractical
- Time of flight: somewhat accurate, very short range, inexpensive, practical
- Ultrasonic: somewhat accurate (probably less so on a moving vehicle), short range, inexpensive, practical
- Stereo cameras: pretty accurate assuming I can keep the cameras at precisely the same distance, medium range, inexpensive, practical
- Optical geometry: not very accurate, medium range, inexpensive, practical

I weighed all the variables and ultimately decided that optical geometry was the best fit for me. [This answer](https://photo.stackexchange.com/questions/12434/how-do-i-calculate-the-distance-of-an-object-in-a-photo#12437) on the Photography Stack Exchange does a great job of explaining the ins and outs of it. 
>  "...the ratio of the size of the object on the sensor and the size of the object in real life is the same as the ratio between the focal length and distance to the object.
> To work out the size of the object on the sensor, work out it's height in pixels, divide by the image height in pixels and multiply by the physical height of the sensor.

In my case I'm swapping out height for width. I'd detect the car in front of me and measure the detection bounding box width. [Federal regulations](https://ops.fhwa.dot.gov/freight/publications/size_regs_final_rpt/) restrict vehicles to a maximum width. Of course, below this maximum, vehicle widths cover a broad range and they vary by vehicle class. All this to say that we're going to experience inaccuracies in measured distance when a car's width differs from the width we're setting in the equation. 

## Vehicle Detection
"But wait...you're glossing over the whole car detection thing!"

To get the bounding box I have to actually detect the car.
There were a couple of options here but I decided to use a Haar cascade.
I won't blame you if you saw "Haar cascade" and thought "How *old* is this article?" Haar cascades are old hat in the brave new world of machine learning and computer vision. **But** they're computationally inexpensive compared to the alternatives and we're running this on a Raspberry Pi...
There are decent vehicle detection Haar cascade models out there so I went with one of the off-the-shelf models and called it a day.

In retrospect I could have used icrawler to scrape together a training set and then used [Teachable Machine](https://teachablemachine.withgoogle.com/) to train a Tensorflow Lite model. Tensorflow Lite would be lightweight enough to run on a Pi Zero, but I wasn't aware of this possibility at the time I was working on all this.

## Heat mitigation
The plan was for this thing to sit on my dash, in full sunlight, and point out at the road ahead of me. The computation alone was going to make it hot, so between that, the direct heating from the sun and the heat radiating up from the dash it was going to get **HOT**. I was going to need some way of ditching excess heat. I knew this would come at the cost of one of my design tenets (small), but there's no way of getting around it.
I looked at heat sinks, blower fans, and Peltier-effect thermoelectric cooling and decided that a conventional heat sink and fan combo would be sufficient. I used [this](https://www.amazon.com/gp/product/B07Y46G1W6/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1) tiny 30x30x4mm blower fan on Amazon. I'll get more into why I went with this fan later. 

## Weather
This was sort of a fun add-on to potentially increase the accuracy of the braking distance calculation by adding in a weather component. If it was clear and sunny then the weather multiplier would be 1. As conditions worsened though the weather multiplier would increase, increasing the estimated braking distance. 

# Code
I wanted to treat the components like daemons and let the master control script request information as it was needed. I didn't want the system bound up while waiting for a GPS signal, so this just seemed like a good idea. There are probably significantly better ways of doing this. This is just the way I went with.
I found [PyZMQ](https://zeromq.org/languages/python/) which "looks like an embeddable networking library but acts like a concurrency framework". 

I wrote several variations of the code, but I never got all the systems to work together the way I expected. I've chosen not to share the code here because it wouldn't make much sense in its current state. I'll make the code available on a per-request basis if anyone's interested.

## Distance Estimation
I'd use OpenCV's CascadeClassifier to detect cars and draw bounding boxes. For each frame I cropped the image to a region of interest that was only 100 pixels high in the hope that this would improve computation speeds. 
If a car was detected within the region of interest its bounding box would be used to estimate distance:
``` distance = (known_size * focal_length)/width ```
If that distance was within safe braking distance then the user would get positive feedback. If not then they're get negative feedback.

## Getting GPS Coordinates
I'd establish a serial connection with the USB GPS device and use pynmea2 to parse the GPS signal. I'd then send the coordinates and GPS timestamp to a zmq socket so the information would be available upon request. 



## Weather conditions
I planned to use OpenWeatherMap's API to access the weather for whatever zipcode I'm in. OpenWeatherMap's usage guidelines recommend that you only query their servers so many times per hour. I figured that unless I was driving at breakneck speeds the weather from place to place isn't likely to change much in 10 minutes, so that's how often I'd allow the system to update the weather multiplier. 

# What I learned
While I never managed to build a device that could do everything I set out to do, I learned a lot in the process. Thinking through some of the design challenges exposed me to a lot of things I'd otherwise be ignorant of. For example prior to this project I didn't know about depth-sensing stereo cameras, and the pros and cons of using them. I didn't know how to receive and parse GPS signals, or how to use geopy's geoencoders to get a zipcode from GPS coordinates. I wouldn't have known about Peltier coolers if I hadn't been looking for ways to mitigate heat. 
The project ultimately failed, but I consider it a successful failure because of the body of knowledge I took away from it. 