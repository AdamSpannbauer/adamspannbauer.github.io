---
title: Finding and Using Images' Dominant Colors using Python & OpenCV
author: ~
date: 2018-03-02
slug: app-icon-dominant-colors
categories: ['cv','python','opencv','machine-learning']
tags: ['cv','python','opencv','machine-learning','data-viz']
---


This post is about finding an image's *dominant color*. To illustrate
this concept we'll be working with app icons from the Apple App Store.
Why do we care about an images dominant color? A couple of uses we'll go
over are sorting images in a collage and [automatically making
appropriate color selections in plots](#plot).

![](https://raw.githubusercontent.com/AdamSpannbauer/iphone_app_icon/master/readme/sorted_app_icons.jpg){: .center-image width="95%" }

# Downloading the App Icons

Before we can play around with app icons we need to have some images of
the icons. To do this I wrote a little scraping script using
[BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/). I won't
go into detail on the scraping, but if you're interested you can check
out the code
[here](https://github.com/AdamSpannbauer/iphone_app_icon/blob/master/download_top_chart_icons.py).

# Extracting Dominant Colors

## Theory

*A note on color before we start: Images are typically stored in the
[RGB colorspace](https://en.wikipedia.org/wiki/RGB_color_space), but the
[HSV colorspace](https://en.wikipedia.org/wiki/HSV_color_space) relates
more to how we perceive color. Because of this feature of HSV we'll be
working with it throughout this post.*

So how do you find the dominant color in an image? A good first guess of
how to do it might be to take the average color of all the pixels in the
image. However, unless our image is all one color, an average will end
up with a result that doesn't resemble our image at all. We can see this
illustrated in the example with the [Stack
Jump](https://itunes.apple.com/us/app/stack-jump/id1263426717?mt=8) icon
below (the average color of the icon is displayed immediately to the
right of the original icon).

![](https://raw.githubusercontent.com/AdamSpannbauer/iphone_app_icon/master/readme/dom_color_k_1.jpg){: .center-image }

So if the average doesn't work well then what does? It turns out a good
strategy of finding dominant color involves
[*k*-means](https://en.wikipedia.org/wiki/K-means_clustering). If we
choose the right value of *k* then the centroid of the largest cluster
will be a pretty good representation of the image's dominant color.

Let's revisist our Stack Jump example. We can see that the icon is
really only made up of 4 colors: green, pink, white, and black. So
choosing a *k* of 4 makes a lot of sense for this case. Below we see
that this strategy performs way better than the using the average color.

![](https://raw.githubusercontent.com/AdamSpannbauer/iphone_app_icon/master/readme/dom_color_k_3.jpg){: .center-image }

## Code

Let's jump into some python code to perform this *k*-means dominant
color extraction. I won't be adding too much commentary since the
docstring takes care of most of what I'd say about the function.

One thing to note is that the function doesn't convert the image to the
HSV colorspace. So if you want the output to be HSV then the input image
will need to already be converted before calling the function.

```python
from sklearn.cluster import KMeans
from collections import Counter
import cv2 #for resizing image

def get_dominant_color(image, k=4, image_processing_size = None):
    """
    takes an image as input
    returns the dominant color of the image as a list
    
    dominant color is found by running k means on the 
    pixels & returning the centroid of the largest cluster

    processing time is sped up by working with a smaller image; 
    this resizing can be done with the image_processing_size param 
    which takes a tuple of image dims as input

    >>> get_dominant_color(my_image, k=4, image_processing_size = (25, 25))
    [56.2423442, 34.0834233, 70.1234123]
    """
    #resize image if new dims provided
    if image_processing_size is not None:
        image = cv2.resize(image, image_processing_size, 
                            interpolation = cv2.INTER_AREA)
    
    #reshape the image to be a list of pixels
    image = image.reshape((image.shape[0] * image.shape[1], 3))

    #cluster and assign labels to the pixels 
    clt = KMeans(n_clusters = k)
    labels = clt.fit_predict(image)

    #count labels to find most popular
    label_counts = Counter(labels)

    #subset out most popular centroid
    dominant_color = clt.cluster_centers_[label_counts.most_common(1)[0][0]]

    return list(dominant_color)
```

I've also written a script to test out this function. The script can be
found
[here](https://github.com/AdamSpannbauer/iphone_app_icon/blob/master/demo_dominant_color.py).
With the script we can use the command line to test out what effect *k*
has on the dominant color of our image of interest. Below is an example
of how to call the script in the context of the [github
repo](https://github.com/AdamSpannbauer/iphone_app_icon), and example
output of using the script on the
[Florence](https://itunes.apple.com/us/app/florence/id1297430468?mt=8)
app icon.

```bash
python -i icons/paid-apps_florence.jpg -k 3
```

<p align='center'>
  <table width="500" border="0" cellpadding="5" align='center'>
  <tr>
  <td align="center" valign="center">
  <img src="https://raw.githubusercontent.com/AdamSpannbauer/iphone_app_icon/master/readme/dom_color2_k_1.jpg"/>
  <br/>
  *k* = 1 (average color)
  </td>
  <td align="center" valign="center">
  <img src="https://raw.githubusercontent.com/AdamSpannbauer/iphone_app_icon/master/readme/dom_color2_k_2.jpg"/>
  <br/>
  *k* = 3
  </td>
  </tr>
</table>
</p>

# Applications of Dominant Color

One possible application of dominant color is for use in sorting images.
An example of doing this with the app icon data can be seen at the [top
of this post](#top). To do this I used the `get_dominant_color` function
and then sorted the images by the hue component of HSV. The full script
used to create the output can be found
[here](https://github.com/AdamSpannbauer/iphone_app_icon/blob/master/sort_icons_by_color.py).

Another possible application of this dominant color extraction is in
data viz. Let's say we want to generate line plots for our sample of
apps. We could use a default color palette, but it might add to our
users' experience if our line colors matched the colors of the app icons
they're familiar with. Using dominant color extraction we can assign
appropriate colors for use in our plot automatically.

<div>
    <a href="https://plot.ly/~spannbaueradam/3/?share_key=ynUb1EA752bQxvRMWwK1GY" target="_blank" title="Plot 3" style="display: block; text-align: center;"><img src="https://plot.ly/~spannbaueradam/3.png?share_key=ynUb1EA752bQxvRMWwK1GY" alt="Plot 3" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="spannbaueradam:3" sharekey-plotly="ynUb1EA752bQxvRMWwK1GY" src="https://plot.ly/embed.js" async></script>
</div>

*Note the plot data is a random walk, it doesn't actually relate to any
app metric (on purpose).*

The plot above is generated with [Plotly](https://plot.ly/) and [this
python
script](https://github.com/AdamSpannbauer/iphone_app_icon/blob/master/dominant_color_plot.py).

You might have noticed the plot avoids the issue of apps having
similiar/identical dominant colors. A strategy of using secondary colors
and/or adjusting colors to be disimilar could be possible strategies to
deal with this issue. At this point I haven't put in the time to
implement these ideas.
