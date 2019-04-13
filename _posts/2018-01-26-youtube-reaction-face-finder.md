---
title: YouTube Reaction Face Finder
author: ~
date: '2018-01-26'
slug: youtube-reaction-face-finder
categories: ['python','keras','opencv','cv']
tags: ['cv','python','keras','deep learning','computer vision']
---

This post is to share [a side project on extracting 'reaction faces' from YouTube videos](https://github.com/AdamSpannbauer/youtube_reaction_face).

*****

# Example Output

Output from video: ['PLOTCON 2016: Hadley Wickham, New open viz in R'](https://www.youtube.com/watch?v=cU0-NrUxRw4).

<table width="500" border="0" cellpadding="5" align="center">
<tr>
<td align="center" valign="center">
<img src="/assets/2018/01/neutral.jpg" height="100px" />
<br />
Neutral
</td>
<td align="center" valign="center">
<img src="/assets/2018/01/scared.jpg" height="100px" />
<br />
Scared
</td>
<td align="center" valign="center">
<img src="/assets/2018/01/happy.jpg" height="100px" />
<br />
Happy
</td>
</tr>
</table>

# Usage

The code requires `opencv` & `keras` (model trained with TensorFlow backend).  To use the reaction face finder:

1. Clone the repo at [AdamSpannbauer/youtube_reaction_face](https://github.com/AdamSpannbauer/youtube_reaction_face)
2. Launch a terminal window and `cd` into the newly cloned repo's directory
3. Use the below command format to run the program
4. Wait for the output to appear in the specified output directory


![](/assets/2018/01/yt_processing_load_bar.gif){: .center-image width="95%" }

```bash
#command format
python youtube_react_face.py -y YOUTUBEURL -o OUTPUT [-m MAXFRAMES]

#example command
python youtube_react_face.py --youtubeURL https://www.youtube.com/watch?v=tVb0g0-JCfk --output output
```

## Arguments:

```bash
#required
-y YOUTUBEURL, --youtubeURL YOUTUBEURL
                 url for youtube video to find reaction faces in
-o OUTPUT, --output OUTPUT
             dir to output reactions to
                        
#optional
-m MAXFRAMES, --maxFrames MAXFRAMES
                stop processing video frames after this many (default 5000)
```

# Disclaimer

Before classifying the emotion of a face the program has to find the face.  This project currently uses a Haar cascade face detector.  This style of detection is great for speed, but is prone to false positives.  So every now and then the output will show a scared tie or an angry shirt.

# Sources

Huge thanks to [Adrian](https://twitter.com/PyImageSearch) at [PyImageSearch](https://www.pyimagesearch.com/) for his book, [Deep Learning for Computer Vision with Python](https://www.pyimagesearch.com/deep-learning-computer-vision-python-book/).  Adrian and the book gave me all the tools needed for building the emotion model.  Since I followed along with the book so closely for creating the model, I don't feel that it would be appropriate to share the code used.
