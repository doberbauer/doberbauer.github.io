The [Brake Safe](https://hyperific.bearblog.dev/brakesafe/) project I left unfinished last year is kind of my white whale. I think of it all the time. "If I'd just had this sensor" or "if I'd just used that algorithm". I'll be driving down the road and think to myself, "If I only finished that project I'd know if I was within safe braking distance of that car."

This post won't be my absolution. Not even close. But it's a step in the right direction. In this post I'm going to be reinventing the wheel in another way (because the safe braking distance problem has certainly been solved by the creators of self driving cars...). 

In this post I'm going to try my darndest to estimate my car's speed via optical flow. That's right, as in the optical flow [used in your mouse](https://en.m.wikipedia.org/wiki/Optical_flow#Optical_flow_sensor). Except instead of determining the direction and magnitude of motion in 2D (like your mouse does) I'll be determining the direction and magnitude of motion *into* the image frame. 

The [Lucas Kanade method](https://en.m.wikipedia.org/wiki/Lucas%E2%80%93Kanade_method) is a point tracking algorithm that should serve my purposes quite well. We get a good trackable point in the frame and LK can track it into the next frame and beyond. Better still it works in real time. 

Next we need a method to get trackable points or "features". These are usually corners, but sufficiently contrasty regions seem to be preferred too. I tried [Shi Tomasi corner detection](https://en.m.wikipedia.org/wiki/Corner_detection#The_Harris_&_Stephens_/_Shi%E2%80%93Tomasi_corner_detection_algorithms) but it didn't consistently yield enough points for me to work with. [Scale invariant feature transform (SIFT)](https://en.m.wikipedia.org/wiki/Scale-invariant_feature_transform) served me well recently in another project but it's not great for real time feature extraction. I ended up trying [FAST](https://en.m.wikipedia.org/wiki/Features_from_accelerated_segment_test), [ORB](https://en.m.wikipedia.org/wiki/Oriented_FAST_and_rotated_BRIEF) and [SURF](https://en.m.wikipedia.org/wiki/Speeded_up_robust_features) and determined that FAST was the best mixture of performance and number of points detected.

With realtime feature extraction using FAST and realtime tracking using Lucas Kanade I had what I needed to generate a [vector field](https://en.m.wikipedia.org/wiki/Vector_field). 

Here's how I set it up. Every even numbered frame I ask FAST to find trackable features. Every odd frame I ask Lucas Kanade to track the features from the previous frame. With that I have two equal sized arrays of X and Y coordinates. Put them together and you get a field of vectors. The vector from Point A to Point B has a direction and a magnitude. If the point is a good trackable point it should be radiating out from the center of motion. The center of motion is the point from which all the good points should be radiating, and is essentially where the car is moving. If you play flight simulator games you'll have seen this before. It's not the direction your plane is *pointing*, it's the direction the car is *moving toward*. 

Trouble is, there are rogue points. Maybe the camera is picking up a reflection of the dash off the windshield, or a truck is passing by on the opposite side of the road. Worse yet - the camera could be picking up your windshield wipers! These bad vectors have angles that don't quite fit the vectors around them - they're going off in crazy directions and would certainly throw off the speed estimation calculation.

And that's just the angles. What about the magnitudes? Objects closer to the vehicle will pass by more quickly than objects further away. Suppose we track a sign in the distance. Between frames 1 and 2 it only moves by a few pixels. Between 2 and 3 it moves a few more. By frames 200 and 201 it's absolutely ripping across the screen at a rate of 200 pixels per frame. 

I use several mechanisms to filter bad tracks.
First, `calcOpticalFlowPyrLK` returns `status` where, according to the documentation, "each element of the vector is set to 1 if the flow for the corresponding features has been found, otherwise, it is set to 0." So we can use that to quickly remove any point with a status of 0. 
`calcOpticalFlowPyrLK` also returns `err` where "each element of the vector is set to an error for the corresponding feature, type of the error measure can be set in flags parameter; if the flow wasn't found then the error is not defined (use the status parameter to find such cases)." If I get the mean error I can then create a logical map of the errors that are less than or equal to that mean. 

I used `np.logical_and` and get the intersection of the aforementioned logical arrays and use that to filter all the points.

Now that we have points with `status==1` and with `error <= mean(error)` we need to compute some angles. Every vector *should* be radiating out from the middle of the image. If a vector's angle is different than the angle to the middle of the image then it shouldn't be included - it's too deviant. First I calculate the angle between new points and old points. Then I compute the angle between new points and the center of the image (not the true center, more upper middle) and take the absolute value of the differences between angles. If the difference is greater than π/2 then the vector is culled. Larger angles make the filter more permissive and noisy.

Now that I have good vectors what's next? Now it's time to compute the distance. Distance is `np.sqrt((x2-x1),(y2-y1))` or more simply `np.linalg.norm(vector_array)`

But as mentioned previously, all vectors are not created equal. The distance from the center matters. We can't ignore the vectors at the center just because they've only moved a few pixels just as we shouldn't inflate the importance of vectors at the edge just because their magnitude is so great. We can help matters by weighting distances by their distance to the center. If a vector is close to the center it should have its distance increased proportion to that distance. If a vector is far away from the center the opposite should be true.
`vector_magnitudes * 1/np.log(abs(dist_to_center))` should work. 

To check the accuracy of my estimation I'll need to know the [ground truth](https://en.m.wikipedia.org/wiki/Ground_truth) speed of my vehicle. I used an OBD2 Bluetooth diagnostic tool and [Car Scanner](https://play.google.com/store/apps/details?id=com.ovz.carscanner) to get this data.

