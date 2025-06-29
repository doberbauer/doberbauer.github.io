In season 3 Episode 7 of Lost, "Not in Portland", there's a scene in which young Karl is being subjected to a [brainwashing video](https://youtu.be/W-bVFovGMIU?t=138). Bizarre images flash before his eyes while loud drum and bass music pounds away over loud speakers. [Here's](https://youtu.be/yp024U-sH1A) the video he was watching in case your brain needs washing.
We see [something similar](https://youtu.be/I4xis2xWI5U&t=4m8s) on an episode of House MD in season 6 entitled "Black Hole". In this case it's not torture but part of a "cognitive pattern recognition" test to map the young woman's brain so they can see what she's thinking. 

I thought it'd be cool to make something like this, but I honestly didn't want to invest a lot of time for a flight of fancy. As such I turned to ChatGPT to do the heavy lifting. A few clarifying prompts later and after some slight edits on my end I got this

```Python
import moviepy.editor as mp

def extract_clips(input_file, output_file, num_clips, clip_duration = 0.3):
    # Load the input video
    video = mp.VideoFileClip(input_file)

    # Calculate the total duration of the input video in seconds
    total_duration = video.duration

    # Calculate the interval between each clip in seconds
    interval = total_duration / num_clips

    # Create a list to store the extracted video and audio clips
    video_clips = []
    audio_clips = []

    # Extract the desired number of clips
    for i in range(num_clips):
        # Calculate the start and end times for the current clip
        start_time = i * interval
        end_time = start_time + clip_duration

        # Extract the video clip from the input video
        video_clip = video.subclip(start_time, end_time)

        # Extract the audio clip from the input video
        audio_clip = video_clip.audio

        # Add the video and audio clips to the lists
        video_clips.append(video_clip)
        audio_clips.append(audio_clip)

    # Concatenate the video clips into a single output video
    final_video = mp.concatenate_videoclips(video_clips)

    # Concatenate the audio clips into a single output audio
    final_audio = mp.concatenate_audioclips(audio_clips)

    # Set the audio of the final video to the concatenated audio
    final_video = final_video.set_audio(final_audio)

    # Write the final output video to a file
    final_video.write_videofile(output_file)
```

I snagged a video from [VHS Vault](https://archive.org/details/vhsvault) and gave it a try. [Here's the result](https://youtu.be/5ohxA7YwRAU). All I need to do now is add some throbbing DnB over top of it. Maybe [Cartridge - End of the World](https://youtu.be/e59Mo7AhMhg?t=37) or [Limewax - Everything](https://youtu.be/bwhxKYVDTBU?t=67).

I'm just posting this as a thought experiment and for entertainment purposes. Please don't use this for sketchy unethical reasons including but not limited to torture or brainwashing.