---
title: Building a Rasa Chatbot to Perform Natural Language Queries
author: ~
date: '2018-03-15'
slug: building-a-rasa-chatbot
categories: ['python','nlp','machine-learning']
tags: ['python', 'nlp', 'machine-learning', 'chatbot']
---

<a name="top"></a>

In this post I'll be sharing a stateless chat bot built with [Rasa](https://rasa.com/).  The bot has been trained to perform natural language queries against the [iTunes Charts](https://www.apple.com/itunes/charts/free-apps/) to retrieve app rank data.  A preview of the bot's capabilities can be seen in a small [Dash](https://plot.ly/products/dash/) app that appears in the gif below.

All the code used in the project can be found in [this github repo](https://github.com/AdamSpannbauer/app_rasa_chat_bot).

*****

![Chat bot demo](https://github.com/AdamSpannbauer/app_rasa_chat_bot/blob/master/readme/dash_demo2.gif?raw=true){: .center-image width="50%" }

## The Process

### Some about Rasa


![Rasa Logo](https://github.com/RasaHQ.png){: .center-image width="100px" }

Rasa is a powerful open source framework for building conversational & independent chatbots.  The cornerstone concepts for making chatbots are intent classification, named entity recognition (NER), and state management.  In this project we'll leverage intent and NER, but the app rank bot will be stateless for simplicity.  In a real production chat bot, managing state is a must for your users to have a good experience.

Rasa has great [documentation](https://nlu.rasa.ai/), so we won't go too in depth on general Rasa usage.  However, I'll share a high level overview of the steps taken to build the app rank bot, and we'll go into detail when it doesn't overlap with the docs.

### Gathering Training Data

Like any machine learning project, we need to start off with some training data.  This project was trained on the [Free](https://www.apple.com/itunes/charts/free-apps/), [Top Grossing](https://www.apple.com/itunes/charts/top-grossing-apps/), and [Paid](https://www.apple.com/itunes/charts/paid-apps/) categories for apps on the [iTunes Charts](https://www.apple.com/itunes/charts/).  To gather this data a [python script](https://github.com/AdamSpannbauer/app_rasa_chat_bot/blob/master/utils/downloader.py) using BeautifulSoup was written.  The [results](https://github.com/AdamSpannbauer/app_rasa_chat_bot/blob/master/data/app_chart_data.csv) were stored as a csv that contains columns for the app's name, genre, rank, and chart.  These 4 points of data will be our entity types in our NER model. 

![Itunes Charts](https://github.com/AdamSpannbauer/app_rasa_chat_bot/blob/master/readme/itunes_charts.png?raw=true){: .center-image width="75%" }

Now let's build up data for training our intent classifier.  Rasa provides some [example data](https://github.com/RasaHQ/rasa_nlu/blob/master/data/examples/rasa/demo-rasa.json) in their tutorial for building ['A simple restaurant search bot'](https://nlu.rasa.ai/tutorial.html#section-tutorial).  In the spirit of being ~~lazy~~ resourceful, let's use this example data has a starting point.  The provided data has some example phrases for the intents of `greet`, `affirm`, `restaurant_search`, & `goodbye`.  We'll keep all these, but change the labels for `restaurant_search` to `None` (I additionally added some random sentences from [Wikipedia](https://www.wikipedia.org/) to the `None` class).  The resulting generic dataset can be found [here](https://github.com/AdamSpannbauer/app_rasa_chat_bot/blob/master/data/generic_rasa_train_data.json).

So we have some generic intent classes, but we don't have any training data for the intent of searching our app chart data.  Rasa suggests to build up your actual training data by [pretending to be the bot yourself](https://medium.com/rasa-blog/put-on-your-robot-costume-and-be-the-minimum-viable-bot-yourself-3e48a5a59308); this way you can collect real data from your users.  Since this is just a small effort I decided to go a less involved route.  Generic template sentences were made up like the ones below.

```python
#example template sentences for intent 'app_rank_search'
['show me the number {rank} app on the {chart} chart',
'what rank is {app}',
'what {genre} apps are popular']
```

After generating a number of these sentences, a bootstrap style approach was used to generate data.  Both the phrases and entities were randomly sampled to fill in the `{blanks}`.  This sampling process was run a couple thousand times to ensure representation of entities in varied phrases.  This process generates data that's seemingly realistic; however, in a real world scenario the strategy of gathering data directly from your users would be ideal.

The full script used to generate [our final training data](https://github.com/AdamSpannbauer/app_rasa_chat_bot/blob/master/data/app_train_data.json) can be seen [here](https://github.com/AdamSpannbauer/app_rasa_chat_bot/blob/master/gen_training_data.py).

### Training

With the training data squared away we can run the model training process.  This step is really easy thanks to the work put in by the Rasa team.  The full script used to train the model is shown in the below code chunk.

```python
from rasa_nlu.converters import load_data
from rasa_nlu.config import RasaNLUConfig
from rasa_nlu.model import Trainer
#read in training data
training_data = load_data('data/app_train_data.json')
#define pipeline and specify config
args = {'pipeline': 'spacy_sklearn'}
config = RasaNLUConfig(cmdline_args = args)
trainer = Trainer(config)
#train model
interpreter = trainer.train(training_data)
#save model
model_directory = trainer.persist('./rasa_model')
```

### Testing Rasa NLU Model

*Note: As is the theme with a lot of my side projects, my focus is on learning a new technology as opposed to building a fully optimized production system.  So staying in line with that theme, the 'testing' is not as rigorous as it should be in a 'real' application.*

Before we add in bot-like responses, we can examine the output of our Rasa NLU model.  Example code of how to do this can be seen below.

```python
import json
from rasa_nlu.model import Interpreter
from rasa_nlu.config import RasaNLUConfig
# read in model
args = {'pipeline': 'spacy_sklearn'}
config = RasaNLUConfig(cmdline_args = args)
model_path = 'path/to/model/dir'
interpreter = Interpreter.load(model_path, config)
#call model on an example sentence
example = u'what is the number 1 games app'
parsed = interpreter.parse(example)
#print entities parsed
print(json.dumps(parsed['entities'],indent=2))
#OUTPUT
#  [
#    {
#      "start": 19,
#      "extractor": "ner_crf",
#      "end": 20,
#      "value": "1",
#      "entity": "numrank"
#    },
#    {
#      "extractor": "ner_crf",
#      "end": 26,
#      "processors": [
#        "ner_synonyms"
#      ],
#      "value": "games",
#      "entity": "genre",
#      "start": 21
#    }
#  ]
#print top intent class prediction
print(json.dumps(parsed['intent'],indent=2))
#OUTPUT
#  {
#    "confidence": 0.9971578029365911, 
#    "name": "app_rank_search"
#  }
```

The output of calling our model is a dictionary that contains the keys: `[u'entities', u'intent', 'text', u'intent_ranking']`.  The contents of the `'entities'` and `'intent'` items can be seen in the above chunk.  The `'text'` item stores the input phrase that we passed to the interpreter.  The `'intent_ranking'` item stores a list of objects with structure identical to the `'intent'` item; this list is a sorted and shows the confidence reported for each intent class.

### Giving the Bot a Voice

To interact with the bot we'll write a respond function.  The full 'bot' module used in this project can be found [here](https://github.com/AdamSpannbauer/app_rasa_chat_bot/blob/master/utils/bot.py).  Our bot leverages the `'entities'` and `'intent'` output of our Rasa model.  If the intent is `greet` or `goodbye` then our bot will respond with a random greeting/closing.  If the intent is our custom `app_rank_search` class, then we use the parsed entities to filter our app rank data.  Once we have this filtered data we can use it to fill in blanks in response templates (such as the ones shown below).

```python
#example query reporting template responses
['{app} is a {genre} app ranked {rank} on the {chart} chart',
'number {rank} on the {chart} chart is the {genre} app {app}']
```

### Interacting with the Bot

I wrote up 2 interfaces that allow us to intereact with our respond function.  

The 1st interface was an excuse to write my first [Plotly Dash](https://plot.ly/products/dash/) application.  It uses similar logic to the command line interface that we'll see next, but the output is much \(prettier\).  The full Dash application code can be seen [here](https://github.com/AdamSpannbauer/app_rasa_chat_bot/blob/master/dash_demo_app.py).  The resulting application is shown in the gif at the [top of this post](#top).

The other bot interface is from the command line.  The bulk of the code used for this interface can be seen below (the full script can be seen [here](https://github.com/AdamSpannbauer/app_rasa_chat_bot/blob/master/live_test_rasa.py)).  We infinetly loop prompting for user input and passing it to the custom `respond` function.  This `respond` function accepts the user input and applies our trained Rasa interpreter to the text.  We also supply a path to the app chart data for our respond function to query.  Example output from the command line interface can be seen below the code chunk.

```python
#import our custom bot module
import utils.bot
print("TYPE 'exit' TO LEAVE CHAT\n\n")
while True:
  #prompt and read in user input
  user_input = unicode(raw_input("USER: "))
  #exit if prompted by user
  if user_input.lower() == 'exit':
    break
  #generate response with custom respond function
  response = utils.bot.respond(
                #text to respond to
                user_input, 
                #rasa nlu model object
                interpreter,
                #path to scraped app data
                app_data_path)
  #print response
  print('BOT: {}\n'.format('\n'.join(response)))
```

![Chat bot CLI demo](https://github.com/AdamSpannbauer/app_rasa_chat_bot/blob/master/readme/example.gif?raw=true){: .center-image width="80%" }
