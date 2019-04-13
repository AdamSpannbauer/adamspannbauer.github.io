---
title: rPackedBar on CRAN
author: ~
date: '2018-02-03'
slug: rpackedbar-on-cran
categories: ['r','data-viz','shiny']
tags: ['r','data-viz','shiny']
---

This post is to announce [`rPackedBar`](https://github.com/AdamSpannbauer/rPackedBar)'s release to CRAN and to share a [shiny app to visualize twitter interactions using a packed barchart](https://spannbaueradam.shinyapps.io/rPackedBarDemo/).

_This post and the package has been updated due to feedback from [Xan Gregg](https://twitter.com/xangregg)._

![](/assets/2018/02/packbar_twitter_app_screenshot.png){: .center-image width="95%" }

# About Packed Bars

If you want to read more about the ideas behind packed bars, I'd suggest to read [this introduction]((https://community.jmp.com/t5/JMP-Blog/Introducing-packed-bars-a-new-chart-form/ba-p/39972)) by the chart's creator, [Xan Gregg](https://twitter.com/xangregg).  Also in the linked article, he provides an annotated example that serves as a pretty good intro by itself (shown below).

![](/assets/2018/02/annotated_packed_bar.png){: .center-image width="95%" }

# Using Packed Bars in R

## Installation

```r
#install from CRAN
install.packages('rPackedBar')

#install dev version
devtools::install_github('AdamSpannbauer/rPackedBar')
```

## Usage

```r
library(data.table)
library(rPackedBar)

#use sample data from treemap package
data(GNI2014, package = 'treemap')
data.table::setDT(GNI2014)

#summarize data to plot
my_input_data = GNI2014[, sum(population), by=country]
#make label shorter to fit
my_input_data[country=="United States", country := "USA"]

#packed bar with default settings
p = plotly_packed_bar(my_input_data,
                      label_column = 'country',
                      value_column = 'V1')

plotly::config(p, displayModeBar = FALSE)
```

<div>
    <a href="https://plot.ly/~spannbaueradam/5/?share_key=LwqOd3f9Lp9QgY3pMlg5ur" target="_blank" title="rpackedbar_example" style="display: block; text-align: center;"><img src="https://plot.ly/~spannbaueradam/5.png?share_key=LwqOd3f9Lp9QgY3pMlg5ur" alt="rpackedbar_example" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="spannbaueradam:5" sharekey-plotly="LwqOd3f9Lp9QgY3pMlg5ur" src="https://plot.ly/embed.js" async></script>
</div>


# [rPackedBar Demo App on shinyapps.io](https://spannbaueradam.shinyapps.io/rPackedBarDemo/)

If you want to see some more example usage of the `rPackedBar` package without having to write any code you can check out [a toy app](https://spannbaueradam.shinyapps.io/rPackedBarDemo/) I published on [shinyapps.io](shinyapps.io).  The app grabs some tweets for a given user and uses packed bars to plot tweet interactions (as measured by retweets and favorites).  You can see example output in the screen shot at the top of this post.   

_Note: The app uses the functions `packedBarOutput` & `renderPackedBar`.  At the time of writing this post, these functions are not included in the version of the package on CRAN.  They are included in the dev version that can be installed using: `devtools::install_github('AdamSpannbauer/rPackedBar')`_

# Update to Post & Package

The updates to the package are only available in the dev version at the time of writing this.

<p align="center"><blockquote class="twitter-tweet tw-align-center" data-cards="hidden" data-lang="en"><p lang="en" dir="ltr">Thanks again for the constructive feedback.<br><br>☑ Left align primary bar labels<br>☑ Lighter grays in secondary bars<br> ☐  Auto-determine label width to avoid unlabeled primary bars while avoiding labels spilling over bar (non issue in interactive use though) <a href="https://t.co/eroaqnCo6a">pic.twitter.com/eroaqnCo6a</a></p>&mdash; Adam Spannbauer (@ASpannbauer) <a href="https://twitter.com/ASpannbauer/status/960330930341675008?ref_src=twsrc%5Etfw">February 5, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></p>
