---
title: "lexRankr & Twitter: find a user's most representative tweets"
author: "Adam Spannbauer"
date: 2017-03-09T21:14:05-05:00
categories: ["R"]
tags: ["R","lexrank", "twitter", "nlp", "tidy"]
---

# Packages Used

```r
library(lexRankr)
library(tidyverse)
library(stringr)
library(httr)
library(jsonlite)
```

In this post we'll get tweets from twitter using the twitter API and then analyze the tweets using [lexRankr](https://github.com/AdamSpannbauer/lexRankr) in order to find a user's most representative tweets.  LexRankr is an R implementation of the [Lexrank Algorithm](https://www.cs.cmu.edu/afs/cs/project/jair/pub/volume22/erkan04a-html/erkan04a.html) which can be used for extractive text summarization.  

If you don't care about interacting with the twitter api you can **[jump to the lexrank analysis](#lexrank-analysis)**.
  
# Get user tweets
  
Before we can analyze tweets we'll need some tweets to analyze.  We'll be using [Twitter's API](https://dev.twitter.com/overview/api), and you'll need to set up an account to get all the keys needed for the api.  The credentials needed for the api are: consumer key, consumer secret, token, and token secret. 
Once we have the keys, we'll set them as environment variables to use in for the rest of the code in this post.  Below is how to set up your credentials to use the twitter api in this vignette.

```r
## set api tokens/keys/secrets as environment vars
# Sys.setenv(cons_key     = 'my_cons_key')
# Sys.setenv(cons_secret  = 'my_cons_sec')
# Sys.setenv(token        = 'my_token')
# Sys.setenv(token_secret = 'my_token_sec')

#sign oauth
auth <- httr::oauth_app("twitter",
                        key=Sys.getenv("cons_key"),
                        secret=Sys.getenv("cons_secret"))
sig  <- httr::sign_oauth1.0(auth, 
                            token=Sys.getenv("token"), 
                            token_secret=Sys.getenv("token_secret"))
```

Now that we have our credentials set up, we'll use a custom function to get a user's tweets.  The function definition is a little lengthy to be in the middle of a blog post, but the code for the function `get_timeline_df` can be seen [here](https://gist.github.com/AdamSpannbauer/a5f0bf7f815fca7035828466911cacf9).  The function takes a user's twitter handle, the number of tweets to get from the api, and the credentials we just set up; it returns a dataframe with the columns `created_at, favorite_count, retweet_count, text`.

We can now use our function to gather a user's tweets.  For an example lets use one of the most famous twitter accounts as of late: ['@realDonaldTrump'](https://twitter.com/realDonaldTrump). 
```r
tweets_df <- get_timeline_df("realDonaldTrump", 600, sig) %>% 
  #clean out newlines for display
  mutate(text = str_replace_all(text, "\n", " "))

tweets_df %>% 
  mutate(Date=strptime(created_at, 
                       format="%a %b %d %H:%M:%S +0000 %Y") %>% 
           str_sub(end=10)) %>% 
  arrange(desc(Date)) %>% 
  head(n=3) %>% 
  select(`Tweet Text`=text, Date) %>% 
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">text</th>
<th style="text-align: left;">created_at</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Yes, it is true - Carlos Slim, the great businessman from Mexico, called me about getting together for a meeting. We met, HE IS A GREAT GUY!</td>
<td style="text-align: left;">Tue Dec 20 20:27:57 +0000 2016</td>
</tr>
<tr class="even">
<td style="text-align: left;">especially how to get people, even with an unlimited budget, out to vote in the vital swing states ( and more). They focused on wrong states</td>
<td style="text-align: left;">Tue Dec 20 13:09:18 +0000 2016</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Bill Clinton stated that I called him after the election. Wrong, he called me (with a very nice congratulations). He “doesn’t know much” …</td>
<td style="text-align: left;">Tue Dec 20 13:03:59 +0000 2016</td>
</tr>
</tbody>
</table>

# Lexrank Analysis

We now have a dataframe that contains a column of tweets.  This column of tweets will be the subject of the rest of the analysis.  With the data in this format, we only need to call the `bind_lexrank` function to apply the lexrank algorithm to the tweets.  The function will add a column of lexrank scores.  The higher the lexrank score the more representative the tweet is of the tweets that we downloaded.

*note: typically one would parse documents into sentences before applying lexrank (*`?unnest_sentences`*); however we will equate tweets to sentences for this analysis*
  
```r
tweets_df %>% 
  bind_lexrank(text, id, level="sentences") %>% 
  arrange(desc(lexrank)) %>% 
  head(n=5) %>% 
  select(text, lexrank) %>% 
  knitr::kable(caption = "Most Representative @realDonaldTrump Tweets")
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">text</th>
<th style="text-align: right;">lexrank</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">MAKE AMERICA GREAT AGAIN!</td>
<td style="text-align: right;">0.0087551</td>
</tr>
<tr class="even">
<td style="text-align: left;">Well, the New Year begins. We will, together, MAKE AMERICA GREAT AGAIN!</td>
<td style="text-align: right;">0.0085258</td>
</tr>
<tr class="odd">
<td style="text-align: left;">HAPPY PRESIDENTS DAY - MAKE AMERICA GREAT AGAIN!</td>
<td style="text-align: right;">0.0082361</td>
</tr>
<tr class="even">
<td style="text-align: left;">Happy Thanksgiving to everyone. We will, together, MAKE AMERICA GREAT AGAIN!</td>
<td style="text-align: right;">0.0060486</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Hopefully, all supporters, and those who want to MAKE AMERICA GREAT AGAIN, will go to D.C. on January 20th. It will be a GREAT SHOW!</td>
<td style="text-align: right;">0.0059713</td>
</tr>
</tbody>
</table>

# Repeating tweetRank analysis for other users

With our `get_timeline_df` function we can easily repeat this analysis for other users.  Below we repeat the whole analysis in a single magrittr pipeline.

```r
get_timeline_df("dog_rates", 600, sig) %>% 
  mutate(text = str_replace_all(text, "\n", " ")) %>% 
  bind_lexrank(text, id, level="sentences") %>% 
  arrange(desc(lexrank)) %>% 
  head(n=5) %>% 
  select(text, lexrank) %>% 
  knitr::kable(caption = "Most Representative @dog_rates Tweets")
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">text</th>
<th style="text-align: right;">lexrank</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;"><span class="citation" data-cites="Lin_Manuel">@Lin_Manuel</span> good day good dog</td>
<td style="text-align: right;">0.0167123</td>
</tr>
<tr class="even">
<td style="text-align: left;">Please keep loving</td>
<td style="text-align: right;">0.0099864</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Here we h*ckin go</td>
<td style="text-align: right;">0.0085708</td>
</tr>
<tr class="even">
<td style="text-align: left;">Last day to get anything from our Valentine’s Collection by Valentine’s Day! Shop: <a href="https://t.co/MXljGLH3qY" class="uri">https://t.co/MXljGLH3qY</a> <a href="https://t.co/qFBCMytKMB" class="uri">https://t.co/qFBCMytKMB</a></td>
<td style="text-align: right;">0.0077583</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Even if I tried (which I would never), I’d last like 17 seconds</td>
<td style="text-align: right;">0.0073899</td>
</tr>
<tr class="even">
<td style="text-align: left;"></td>
<td style="text-align: right;"></td>
</tr>
</tbody>
</table>
