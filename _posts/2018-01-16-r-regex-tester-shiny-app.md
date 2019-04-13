---
title: R Regex Tester Shiny App
author: ~
date: '2018-01-16'
slug: r-regex-tester-shiny-app
categories: ['r','shiny','regex','nlp']
tags: ['r','shiny','regex','nlp']
---

This post is to announce a shiny app I've written to test regular expressions in an R environment.

If you're not interested in reading any commentary on the motivation & features, here's a [link to the app on shinyapps.io](https://spannbaueradam.shinyapps.io/r_regex_tester/).  If you use the app and have any issues please report them to [the app's github repo](https://github.com/AdamSpannbauer/r_regex_tester_app) (also please leave a ★ if you like the app!)

![](/assets/2018/01/regex_full_screenshot.png){: .center-image width="90%" }

****

# Why?

This is definitely [not the first](https://pbs.twimg.com/media/C-j5TEGUMAA9hnT.jpg) online regex tester, and it's not the most fully featured.  The main reasons for developing this were: 

  1.  Testing in an R environment (I would always have to [double my backslashes](https://xkcd.com/1638/) after working out a regex in another tester)
  2. A fun/challenging side project that involved shiny
  3. And of course to have the chance to help somebody out if they find the app useful

# Features

## Options

* Use the common options used across the R [Pattern Matching and Replacement](https://stat.ethz.ch/R-manual/R-devel/library/base/html/grep.html) family of functions.


![](/assets/2018/01/regex_app_options.png){: .center-image width="80%" }

* The other 2 options concerning backslashes allow you to write an R flavored regex.  
    * For example, if you want to match a literal period with a regex you'll type "\\\\." (as if you were writing the regex in R).  
    * If you don't like this behavior, and you'd rather type half of the slashes needed to make the regex functional in R, you can select the "Auto Escape Backslashes" option for "Pattern" and then use "\\." to match literal periods in the app.

## Input

* There are 2 text inputs:
    1. __Matching Pattern__: type the regular expression or fixed pattern here that you want to use to match against your text.
    2. __Test String__: type the text that you want your Matching Pattern to search through


![](/assets/2018/01/regex_app_input.png){: .center-image width="80%" }

## Results

* After the pattern and test string are entered we see 2 different versions of the resulting pattern matching:
    1. The test string is shown with the matches/capture groups highlighted where they appear in the text
        * As noted in the UI, currently nested capture group highlighting isn't supported.  If our matching pattern was "t(e(s))(t)" the highlighting wouldn't display correctly.
    2. The second output is a bulleted list of the matches and capture groups found in our test string.  In the screen shot below we see we matched 2 instances of "test", and each of these matches display below them the contents of the 2 capture groups we included in our regex pattern.

![](/assets/2018/01/regex_app_results.png){: .center-image width="80%" }

## Regex Explanation

* There's additionally a collapsable panel that will do it's best to break down your regex and explain the components.  As noted in the UI these explanations are provided by [rick.measham.id.au](http://rick.measham.id.au/paste/explain)
* The screen shot below shows the explanation for our regex: "t(es)(t)"

![](/assets/2018/01/regex_app_explain.png){: .center-image width="80%" }

## Helping Documentation

![](/assets/2018/01/regex_app_navbar.png){: .center-image width="85%" }

* The app includes some documentation for using regular expressions in R.  The two including pieces of helping documentaion are:
    1. The [RStudio](https://www.rstudio.com/) Regex Cheatsheet
    2. The base R documentation on regex (this is what would appear if you ran the command `?regex`)

# Extra

Shiny's bootstrap roots allow apps to transition between desktop and mobile pretty seamlessly.  The app's mobile experience isn't terrible, so you can use it for all your regex-ing fun on the go! (I won't ask why) 

![](/assets/2018/01/regex_app_mobile.jpg){: .center-image width="60%" }

