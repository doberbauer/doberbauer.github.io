Recently I was playing [Sniper Ghost Warrior Contracts 2](https://en.wikipedia.org/wiki/Sniper_Ghost_Warrior_Contracts_2) (a great game that suffers from an absurdly long title) and I was intrigued by the mask overlay ability. It's a mask worn by the player that can be toggled on at any time to detect and display points of interest in the game world. A point of interest could be an anti personnel landmine, an enemy target, a climbing point, a robotic turret, CCTV camera, etc.
I thought this was pretty cool and I love the idea of a futuristic soldier having access to oodles of data about their environment and tactical situation. 

What got me really fired up was when I started thinking about how all these systems might work in the real world. Individually, every feature the mask offers is possible. After a bit of research into this I came across Microsoft's ["Integrated Visual Augmentation System" or "IVAS"](https://en.wikipedia.org/wiki/Integrated_Visual_Augmentation_System). It's a military grade version of Microsoft's [HoloLens 2](https://en.wikipedia.org/wiki/HoloLens_2) with thermal and low-light cameras in what appears to be a stereo configuration. 

That's cool, but rather than trying to figure out what IVAS does and how it does it, it'll be more fun to dream up my own version. This is a purely "if I had an unlimited budget and engineering/development staff" kind of thought experiment. The thermal camera alone would be prohibitively expensive and the kinds of computation I'll discuss would likely require a more powerful computer than would be feasibly portable, not to mention myriad other engineering challenges insurmountable to the hobbyist maker. With the disclaimers out of the way let's get started.

# Night vision
Being able to see in low light situations provides a major tactical advantage to soldiers. 

[Military NVGs](https://en.wikipedia.org/wiki/Night-vision_device#Generation_3+_(GEN_III_OMNI_I%E2%80%93IX)) are bulky long cylinders and they greatly restrict peripheral vision. Also if you look at a bright light source the goggles become overexposed and temporarily blind the user.

Using ultra high ISO speeds and cutting edge image denoising algorithms it should be possible to capture video with a wide angle lens and a very sensitive sensor, and then denoise each frame.
Robust automatic exposure adjustments should mitigate overexposure. A PID loop with a high sampling rate would probably do nicely. 
[Papers with Code - Video Denoising ](https://paperswithcode.com/task/video-denoising)
[Real time denoising](https://github.com/m-tassano/fastdvdnet)
[Sensitive camera](https://www.arducam.com/product/arducam-cmos-mt9j001-1-2-3-inch-10mp-monochrome-camera-module/)

# Thermal vision
No cool military mask would be complete without thermal vision. Being able to see warm bodies is a [huge tactical advantage](https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fpossibility.teledyneimaging.com%2Fwp-content%2Fuploads%2F2017%2F03%2Fthermal.png&f=1&nofb=1&ipt=78c2393c9e54082f6baf5a78fa35de165d5c13c4b9b134ec4d35ccf277dee7d5&ipo=images), especially when an adversary is wearing camouflage. Thermal vision is also useful for forensic analysis such as how recently a vehicle was driven based on its engine temperature. Thermal cameras are widely available but they're not cheap.

# Area mapping
Area mapping could be useful in a number of ways. Supposing you have intel on the area ahead of time (satellite or aerial LIDAR, or synthetic aperture radar) you could map the area again and highlight aspects that have changed. Viewing the deltas could reveal new fortifications, or indicate which areas are most patrolled. Area mapping could also give data on terrain passage difficulty. In [Death Stranding](https://en.wikipedia.org/wiki/Death_Stranding), the [Odradek](https://deathstranding.fandom.com/wiki/Odradek) was a device used to map the area around the main character. Among other things, this would show you how hard it would be to get your character around [certain environmental obstacles](https://jeuxpourtous.org/wp-content/uploads/2019/11/Death-Stranding-Odradek-Scanner-Comment-utiliser.jpg). For [soldiers carrying heavy gear](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2F2.bp.blogspot.com%2F-IZR1Jm4rxfU%2FWEjCS7HHAdI%2FAAAAAAAAFuk%2F9oS6JXK1EvkRsD-oSOSwRV-YPvogE8G2gCLcB%2Fs1600%2Fheavy-pack-1-5-official.jpg&f=1&nofb=1&ipt=ed8f83032bfa345ab3598cd179a9a28d0015a06cbd4d0ce62f7e991e867aa0df&ipo=images), being able to see the easiest path could preserve stamina for when it's life or death.

How would area mapping work for a soldier? There are a few ways of getting area geometry. Lidar and stereo cameras are some of the most common. Each has their own limitations though.

# Path finding overlay
- Overlay waypoints, compass, maps, tracking data, detected/classified objects and output from other systems

# Compass
This one's pretty simple and very do-able with inexpensive tech. Knowing your cardinal directions gives more situational awareness. A GPS or magnetometer is all that's needed to get your orientation relative to the magnetic north pole. From there it's just a matter of displaying that data as a compass on the heads up display. 

# Threat detection
Numerous video games feature the ability to tag and track enemies as they move around, [even behind obstacles](https://i.imgur.com/y54fWNf.png), and it's no wonder because it's extremely useful. Being able to see the real-time positions of enemies can level the playing field for a single soldier going up against a horde of adversaries. With platforms like DARPA's [ARGUS-IS](https://en.wikipedia.org/wiki/ARGUS-IS) it might be possible to tag and track numerous individuals in real-time, but that system requires an air asset to loiter overhead and carry the sensor suite. Until reality can catch up to science-fiction and video games, the individual soldier operating without air asset support will have to settle for less. Less in this case is a convolutional neural net trained on human shapes - head, torso, arms, legs. There are numerous pretrained models for human detection in Tensorflow, PyTorch, OpenCV and more. 
The neural net would scan each frame for human-shaped things, draw bounding boxes for each detected object and the confidence level. The user would have the ability to change the confidence level threshold themselves so filter out false positives. This is the detector portion and it would be followed by a classifier portion to determine if the detected human is a threat.
The image within the bounding box defined by the detector would be send downstream to a classifier trained on humans with guns, knives, explosives, camouflage, etc. There could be a checkpoint after which the classifier model could be trained on the camo patterns and gear profile of your adversary. There's now an [open source model](https://www.iterate.ai/solutions/Open-Source-Threat-Detection) trained to do just that.

There could also be a final pass to detect for landmines, IEDs and unexploded ordinance.
All of these models could be evaluated to determine the minimum number of parameters where the model's accuracy is unaffected. This could reduce the time it takes to check images against the model. 

# Motion amplification 
Here's a fun one - [Eulerian motion and color amplification](https://people.csail.mit.edu/mrub/evm/#code). 
The process amplifies miniscule motion so it's more apparent to the human eye. It can do the same task for color instead of motion, exaggerating changes in color so they're more apparent to us. 
Motion amplification is used to inspect things like engines and industrial machines to see if they're improperly mounted. A sentry could use motion amplification as an early warning system – exaggerating the motion of an approaching person, drone or vehicle.
Color amplification could be used in spy scenarios to indicate if someone's lying. The amplified colors could be used to measure heartrate and you could see an elevation in heart rate and redness intensity when someone's lying. 

# Weapon detection and classification
The mask could also include weapon detection and classification features. Say your mask has just alerted you that a person has been detected and classified as a threat. The weapon classification system kicks in and attempts to classify the weapon. Is it a rifle, a pistol, a rocket launcher? If the threat is 150 meters away a pistol won't be as much of a threat as a precision rifle. Once the class of weapon has been determined it could attempt to classify the model of the weapon. At 1000 meters a 7.62mm precision rifle won't be as much of a threat as a 12.7mm anti-material rifle. Stats about the weapon's muzzle velocity, effective range, caliber, and standard magazine size could be pulled from a database and displayed to the user.

# Laser range finding
Whether you're sniping, firing a rifle, calling in air support or popping off a round from an under-barrel M203 grenade launcher, knowing the range to your target is important. 
The visor could have a laser range finder mounted on it to provide range information for whatever the user is looking at. It would be mounted on a small rotational turret to allow it to swivel and acquire range data for any selected target within the visor's field of view. That way it's not restricted to only providing the range for something the user's head is pointed at. 

# Audio amplification
Imagine you're a solo sharpshooter in hostile territory. You're prone and looking down on your target. Your field of vision is what's in front of you and what's in your scope. Your sense of hearing is the only thing that connects you to your immediate surroundings. Now imagine your fancy high tech mask could amplify sounds nearby, making you more aware of what's going on around you. It could isolate frequencies common to footsteps and drop the volume on other frequencies. It could even denoise the audio signal so it's not raising the noise floor as it amplifies the audio signal.

# Battery pack
Between the computation, driving the screen and powering the fans to keep everything cool, this thing's going to be drawing some power. It's going to need a battery with a lot of juice, but there are some other requirements in addition to capacity. Soldiers have enough to carry as it is so the battery will need to be lightweight. If a soldier has to choose whether to pack the mask or extra armor plating, ammunition or water, chances are they'll choose those mission critical pieces of gear over the mask. The battery also needs to be ruggedized to withstand impacts and rough handling. I'm not saying it needs to be able to withstand a direct bullet strike or even shrapnel, but it should be able to withstand being knocked against the hull of an APC, the impact of a parachute drop, and otherwise being violently thrown around during combat. 
The battery's chemistry should also be tolerant of extreme temperatures. If it can't withstand the desert heat or the arctic cold then that's a problem. 
While this may sound fantastical, it would be nice if the battery could be recharged by the movement of the soldier. Mechanical energy from the soldiers movements could be translated into electrochemical energy to recharge the cells and keep the system alive. According to one paper, heel strikes, knee and ankle movements generate the most energy and would be suitable targets for biomechanical energy harvesting.
[Energy dense battery at extreme temp](https://www.pnas.org/doi/full/10.1073/pnas.2200392119)
[Biomechanical Energy Harvesting](https://jneuroengrehab.biomedcentral.com/articles/10.1186/1743-0003-8-22)

# Flashlight
A simple but useful addition. A small array of high intensity LEDs would work great here. Could be modified to strobe and disorient adversaries and be detachable to misdirect enemy fire toward the off-body flashlight.

# Wi-Fi vision
Researchers discovered that [wifi signals are disrupted by human bodies]((https://news.mit.edu/2018/artificial-intelligence-senses-people-through-walls-0612). Not only that, but they recently were able to [accurately estimate poses](https://vpnoverview.com/news/wifi-routers-used-to-produce-3d-images-of-humans/) of people even behind walls. The setup requires that you have access to the wifi transmitter as well as a receiver on the other side, so it's not exactly the [bat vision seen in The Dark Knight](https://i.imgur.com/0axPjrf.gif). Still, in a hostage situation a HRT team could position a wifi transmitter on one side of a building and a receiver on the other and get a sense of people's positions inside. 

# Laser microphone
If we didn't qualify for Mi6's Q branch we certainly will now. A [laser microphone](https://hackaday.com/2010/09/25/laser-mic-makes-eavesdropping-remarkably-simple/) fires a laser beam at a target and measures tiny oscillations in the reflected light produced when sound vibrates nearby surfaces. The oscillations in the light are converted into an audio signal so you can hear what's going on a kilometer away. All that's really required is a laser, a sensitive camera and the software to drive the calculations. It's probably even possible to do all of this just with circuits.
# Video Streaming and Recording
Police officers in the U.S. are required to wear body cameras to make officers accountable for their actions and reduce corruption. We already have a suite of cameras, why not record that video to a solid state memory module on board? 
While we're at it we could provide the video as a feed for commanders to view and ~~micromanage~~ coordinate soldier's movements.
