---
title: Building an Image Search Engine with Pretrained ResNet50 from Keras
author: ~
date: 2018-03-04
slug: using-keras-to-build-an-image-search-engine
categories: ['cv','python','opencv','machine-learning','deep-learning']
tags: ['cv','python','opencv','machine-learning']
---

In this post we'll be using the pretrained
[ResNet50](https://keras.io/applications/#resnet50)
[ImageNet](http://www.image-net.org/) weights shipped with
[Keras](https://keras.io) as a foundation for building a small image
search engine. In the below image we can see some sample output from our
final product.

![](https://raw.githubusercontent.com/AdamSpannbauer/iphone_app_icon/master/readme/togo_imnet_search_shadow.png){: .center-image width="75%" }

As in my [last post](/2018/03/02/app-icon-dominant-colors/) we'll be
working with app icons that we're gathered by this [scrape
script](https://github.com/AdamSpannbauer/iphone_app_icon/blob/master/download_top_chart_icons.py).
All the images we'll be using can be found
[here](https://github.com/AdamSpannbauer/iphone_app_icon/tree/master/icons).

# Image Search Engine Overview

![](https://raw.githubusercontent.com/AdamSpannbauer/iphone_app_icon/master/readme/search_engine_flow.png){: .center-image width="95%" }

The above diagram shows a high level flow for not only our image search
engine, but for search engines in general.

You'll notice the foundation for searching is built around having
quantitative features to compare the query against the potential
results. The engine we're about to build will generate these features
using the pretrained [ResNet50](https://keras.io/applications/#resnet50)
[ImageNet](http://www.image-net.org/) weights shipped with
[Keras](https://keras.io).

# Extracting Features with ResNet50

So how do we extract features with the ResNet50 model? Turns out its
pretty simple thanks to the amazing work of the Keras developers.

By default, the pretrained model will classify the images we throw at
it. The translation into the ImageNet classes is done by the
fully-connected layer at the 'top' of the network. In our case we want
don't want this top classification layer, so we set
`include_top = False` and voila, we now have features coming out of the
network. We'll also set `pooling='avg'` to flatten our feature output
into 1 dimension. The full definition of our feature extractor is shown
below.

```python
from keras.applications.resnet50 import ResNet50

model = ResNet50(weights='imagenet', 
                 include_top=False, 
                 pooling='avg')
```

With that model definition we're well on our way to builing a feature DB
for us to search against. The rest of the script to build out a database
of features is iterating over our images, calling `model.predict()` to
extract featrures, and writing the output to file. Since we're working
with a relatively small set of images (300) we're going to write to a
`.csv`. If we were working with larger data there would come a point
where looking into other storage methods could be beneficial (eg
[hdf5](https://en.wikipedia.org/wiki/Hierarchical_Data_Format) or a
traditional database technology).

The full script that was used to generate features for our search engine
can be found
[here](https://github.com/AdamSpannbauer/iphone_app_icon/blob/master/create_imagenet_features.py).
We can call the script from the command line by providing a path to our
directory of images and the path to our output `.csv`.

```bash
python create_imagenet_features.py --dataset icons --output imagenet_features.csv
```

# Performing a Search

To perform a search we need to build a framework to accept a query
image, extract features, compare these features to the images in our
feature DB, and return the images with the most similar features. The
first two steps are repeats from buiding up our feature DB. The next 2
steps will have us stepping into new territory.

To be able to compare our features we'll need to decide on a
distance/similarity metric. Potential metrics could be [Euclidean
distance](https://en.wikipedia.org/wiki/Euclidean_distance), [cosine
distance](https://en.wikipedia.org/wiki/Cosine_similarity), and [*Χ*^2^
(chi squared) distance](https://en.wikipedia.org/wiki/Chi-squared_test).
In our search we're going to be using *Χ*^2^ (there was no quantitative
testing to arive at this decision, but it qualitatively produces good
results with our icon data). The below code chunk shows how we can
implement *Χ*^2^ in python.

```python
import numpy as np

# function to compute chi square dist
def chi2_distance(histA, histB, eps=1e-10):
    d = 0.5 * np.sum(((histA - histB) ** 2) / (histA + histB + eps))
    return d
```

The rest of [our search
script](https://github.com/AdamSpannbauer/iphone_app_icon/blob/master/imagenet_search.py)
handles applying our distance metric exhaustively to our query and each
image in our database. Again, we're taking advantage of how small our
data is; if we had larger data this exhaustive search would be painfully
slow and we'd want to take some steps to speed up our queries with some
additional preprocessing of our feature DB. Once we've calculated the
distance between our query and all possible results we'll do some
sorting and return the top N results.

To call [the search
script](https://github.com/AdamSpannbauer/iphone_app_icon/blob/master/imagenet_search.py)
you can use the below line from your terminal. We just need to specify
the paths to our query image and the feature DB we're querying against.

```bash
python imagenet_search.py --query search.jpg --featuresPath imagenet_features.csv
```

# Results

So we now have an image search engine built up. Let's take a look at
some example results.

### Searching with App Icons

The first output we'll look at is searching with app icons that exist in
our feature DB. Since we already have features for these images, our
search script looks them up in the database instead of calling the
ResNet50 to extract them.

Our first example result is using the [Papa's Burgeria To
Go!](https://itunes.apple.com/us/app/papas-burgeria-to-go/id600626116?mt=8)
app icon. There are a few other 'To Go!' apps in our dataset, and if our
search is performing well we should see these come up in our results. As
you can see below, our engine performed exceptionally well on this
search.

Our output includes the icon used as our query with a distance of 0.00.
This is an uninteresting result, but it serves as confirmation that our
distance metric is doing what it's supposed to. The other aspect in our
output to note is our distance metric displayed at the top of each
result. The relative differences in these distances are more important
to note than the actual values.

![](https://raw.githubusercontent.com/AdamSpannbauer/iphone_app_icon/master/readme/togo_imnet_search_shadow.png){: .center-image width="75%" }

Below are some more example results of searching with an image already
included in our feature DB.

![](https://raw.githubusercontent.com/AdamSpannbauer/iphone_app_icon/master/readme/slots_imnet_search_shadow.png){: .center-image width="40%" }
  
![](https://raw.githubusercontent.com/AdamSpannbauer/iphone_app_icon/master/readme/candycrush_imnet_search_shadow.png){: .center-image width="40%" }

### Searching with an Outside Image

Let's say we're big fans of [Assassin's
Creed](https://en.wikipedia.org/wiki/Assassin%27s_Creed) and want to
perform an image search for an app suggestion. To do this we can screen
shot our favorite hooded assassin and query our app icons.

Our top result is [Assassin's Creed
Identity](https://itunes.apple.com/us/app/assassins-creed-identity/id880971164?mt=8),
which seems to be as good of a result as we could have hoped for. The
rest of our results aren't as relevant, but note that we see a steep
increase in distance after our first result.

![](https://raw.githubusercontent.com/AdamSpannbauer/iphone_app_icon/master/readme/assassin_creed_search.png){: .center-image width="75%" }
