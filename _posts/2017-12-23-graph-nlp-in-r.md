---
title: Plotting Christmas Carols in R
author: ~
date: '2017-12-24'
slug: graph-nlp-in-r
categories: ['R','NLP','graph','data viz','spacy']
tags: ['R','NLP','graph','data viz','spacy']
---

<script src="/assets/rmarkdown-libs/htmlwidgets/htmlwidgets.js"></script>
<link href="/assets/rmarkdown-libs/vis/vis.css" rel="stylesheet" />
<script src="/assets/rmarkdown-libs/vis/vis.min.js"></script>
<script src="/assets/rmarkdown-libs/visNetwork-binding/visNetwork.js"></script>

In this post we'll be playing with [`spacyr`](https://github.com/kbenoit/spacyr) & [`visNetwork`](http://datastorm-open.github.io/visNetwork/) to parse and plot the lyrics of the Christmas Carol 'Santa Claus is Coming to Town'.

<hr>

The `spacyr` package is a wrapper around the [`spaCy`](https://spacy.io/) python module for NLP.  The parsing capabilities of `spaCy` are pretty advanced and they allow us to easily get dependency relationships between the tokens in our documents.  One way to view the output of these parsing capabilities is to try them out interactively with [displacy](https://demos.explosion.ai/displacy/).  Instead of using displacy, we'll make our own viz using `visNetwork`.

Before we can perform the analysis we'll need some text data.  Tis the season, so we'll scrape the lyrics of the carol 'Santa Claus is Coming to Town'.  In the below chunk we scrape the lyrics to the carol.

```r
#load all the packages used in this analysis
library(spacyr)
library(visNetwork)
library(xml2)
library(rvest)
library(data.table)

#scrape 'Santa Claus is Coming to Town'
page_html = read_html("http://www.metrolyrics.com/santa-claus-is-coming-to-town-lyrics-christmas-carols.html")
#extract text of story
page_txt  = html_text(html_nodes(page_html, ".verse"))
#clean up white space
page_txt = gsub("\\s+", " ", page_txt)
```

Now that we have all of the lyrics we can move onto parsing the text with `spacyr`.  Once we have our environment set up this parse step is a pretty simple task that is accomplished with 2 lines of code in the below chunk.  The rest of the code in the chunk does some cleaning up around the parsed pronouns and punctuation.

*Note: `spacyr` sits on top of python & spacy; so you'll have to have those 2 set up before spacyr will work*

```r
#start background py process with spacy en model loaded
spacyr::spacy_initialize()

#perform spacy parse (returns df)
dt_spacy_parse = spacyr::spacy_parse(page_txt, tag = TRUE,dependency = TRUE)
#convert to datatable
data.table::setDT(dt_spacy_parse)
#replace pronoun lemmas with the token
dt_spacy_parse[lemma == "-PRON-", lemma := tolower(token)]
#remove punctuation
dt_spacy_parse = dt_spacy_parse[pos != "PUNCT",]
```

### Plotting a Single Sentence

With the parsed lyrics we can move on to plotting the carol's structure using `visNetwork`.  Let's start by plotting a single sentence.  We'll use the title lyric of the song as our sentence of interest.  

In the code chunk below we use a custom function, `prep_nodes_and_edges`, to convert the `spacyr` parse output into the `nodes` & `edges` data.frames that we need to plot with `visNetwork`.  This function's definition can be seen at the bottom of this post.  After calling the custom prep function, all that's left is to do the plotting, and inspect our output.

Plotting sentences like this or using [displacy](https://demos.explosion.ai/displacy/) can help give insight into how a particular parser is working.

```r
#select sentence to plot (selected "Santa Claus is coming to town")
sentence_i = dt_spacy_parse[doc_id=="text2" & sentence_id==3]
#re combine words into sentence for plot title
sentence_full = paste(sentence_i[,token], collapse=" ")

#process spacy parse output for plotting with visNetwork
plot_data = prep_nodes_and_edges(sentence_i, label="token")
nodes = plot_data$nodes
edges = plot_data$edges

#plot sentence in heirarchical layout
visNetwork(nodes, edges, main = sentence_full) %>% 
  visNodes(shape="ellipse") %>% 
  visInteraction(dragNodes = FALSE, 
                 dragView = FALSE, 
                 zoomView = FALSE) %>% 
  visHierarchicalLayout() %>% 
  visLegend(zoom = FALSE)
```

![](/assets/2017/12/parsed_sentence_network.png){: .center-image width="80%" }

### Plotting the Full Document

Using our custom `prep_nodes_and_edges` function, it's a small jump to go from plotting single sentences to plotting a full document or corpus.  To do this we'll just leave out the step of filtering down to a single sentence.

Another difference is that we won't use the hierarchical layout provided by `visNetwork` so we'll get a more randomized structure in our viz.  (In just a little bit we'll color it to make it look more like a snowflake).

This viz by itself adds little insight.  We can key in on our largest nodes to see which words are most common, but it's hard to see structure in this viz.  A possible way to gain more insight would be push our parse results into a graph database like [Neo4j](https://neo4j.com/) and use a query language like [cypher](https://neo4j.com/developer/cypher-query-language/) to try and get to more meaningful output.

```r
#process spacy parse output for plotting with visNetwork
plot_data = prep_nodes_and_edges(dt_spacy_parse, label="lemma")
nodes = plot_data$nodes
edges = plot_data$edges
edges = edges[from != to]

#plot whole document
visNetwork(nodes, edges) %>% 
  visIgraphLayout()
```

<div id="htmlwidget-2" style="width:672px;height:480px;" class="visNetwork html-widget"></div>
<script type="application/json" data-for="htmlwidget-2">{"x":{"nodes":{"id":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45],"label":["you","well","watch","out","not","cry","better","pout","i","be","tell","why","santa","claus","come","to","town","he","have","make","a","list","and","check","it","twice","go","find","who","naughty","nice","see","when","sleep","know","awake","if","bad","or","good","so","for","goodness","sake","o"],"group":["PRON","ADV","VERB","PART","ADV","VERB","PROPN","VERB","PRON","VERB","VERB","ADV","PROPN","PROPN","VERB","ADP","NOUN","PRON","VERB","VERB","DET","NOUN","CCONJ","VERB","PRON","ADV","VERB","VERB","NOUN","ADJ","ADJ","VERB","ADV","VERB","VERB","ADJ","ADP","ADJ","CCONJ","ADJ","ADV","ADP","NOUN","NOUN","INTJ"],"size":[5,5,10,5,5,10,5,10,5,5,5,5,25,25,5,5,15,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,10,5,5,5,5,5],"x":[-0.503073300524289,-0.974411821114205,-0.824139417287104,-1,-0.786831838396144,-0.775045102861232,-0.978887411820122,-0.781385335521058,-0.742477209820146,-0.146860997797119,-0.475231078946604,-0.672838565890827,-0.543927313991671,-0.493676777503157,-0.44319029136851,-0.707986766907934,-0.947689892735448,0.0109205555882661,0.297548381360948,0.132601357925836,-0.15921183305039,-0.0212616968896064,0.468631657297508,0.312182002738557,0.54387127229987,0.283027098102021,-0.297014175646098,-0.649567538400154,0.627765646531356,0.623732532184506,0.896306657029122,-0.262411686720852,-0.00214353742631568,-0.237776549528484,-0.0107524859016386,-0.195950260361064,0.233520093897442,0.350116528967443,0.673118018563666,0.184681417324672,-0.0175911876547783,0.0503165801944385,0.448832889569635,0.225971921530838,1],"y":[-0.00982601640096004,0.283731777113992,0.124981952550091,0.0885065306675448,0.615710596657659,0.329581829550505,0.493565561304403,0.821380868042201,-0.426048265599758,-0.314026880974367,-0.285865167433899,-0.626645869362399,-0.831281429776961,-0.664099664731346,-0.444408365983734,-0.262963361272765,-0.298808472796976,0.235571837748786,0.0824465189208881,0.484043188178169,1,0.792010558107051,0.452472289275535,0.761192995527141,0.834755503804657,0.986754750057961,0.447282418238401,0.151108611874824,-0.0156064980829623,0.239404241323742,0.158472830865672,0.169261794782441,-0.196539488935707,-0.071310893708167,-0.0196944868360716,-0.669163735330128,-0.287273231497118,-0.466015081163282,-0.527429834581652,-0.50369875775104,-0.571066572800129,-0.74353708875343,-1,-0.971850251028371,0.450672976591417]},"edges":{"to":[38,38,10,10,10,10,10,10,10,10,10,10,24,24,14,14,14,14,14,15,15,15,15,15,15,15,15,15,15,15,15,15,15,15,6,6,6,6,6,6,6,6,28,28,42,27,27,19,19,35,35,35,35,22,20,20,20,20,20,30,30,8,8,44,32,32,32,34,34,34,11,11,11,11,11,11,11,11,16,16,16,3,3,3,3,3,3,3],"from":[40,39,36,38,42,40,19,37,41,33,1,1,25,26,13,13,13,13,13,10,10,10,10,10,14,14,14,14,14,16,16,16,12,12,7,7,5,5,2,2,1,1,4,16,44,28,20,30,29,10,10,18,18,21,23,24,19,18,22,23,31,5,5,43,18,34,1,10,33,1,10,10,15,15,9,9,1,1,17,17,17,6,4,4,2,2,1,1],"label":["conj","cc","acomp","acomp","prep","acomp","aux","mark","advmod","advmod","nsubj","nsubj","dobj","advmod","compound","compound","compound","compound","compound","aux","aux","aux","aux","aux","nsubj","nsubj","nsubj","nsubj","nsubj","prep","prep","prep","advmod","advmod","advmod","advmod","neg","neg","advmod","advmod","nsubj","nsubj","prt","aux","pobj","xcomp","ccomp","acomp","nsubj","advcl","ccomp","nsubj","nsubj","det","cc","conj","aux","nsubj","dobj","cc","conj","neg","neg","compound","nsubj","advcl","dobj","aux","advmod","nsubj","aux","aux","ccomp","ccomp","nsubj","nsubj","dobj","dobj","pobj","pobj","pobj","ccomp","prt","prt","advmod","advmod","nsubj","nsubj"]},"nodesToDataframe":true,"edgesToDataframe":true,"options":{"width":"100%","height":"100%","nodes":{"shape":"dot","physics":false},"manipulation":{"enabled":false},"edges":{"smooth":false},"physics":{"stabilization":false}},"groups":["PRON","ADV","VERB","PART","PROPN","ADP","NOUN","DET","CCONJ","ADJ","INTJ"],"width":null,"height":null,"idselection":{"enabled":false},"byselection":{"enabled":false},"main":null,"submain":null,"footer":null,"background":"rgba(0, 0, 0, 0)","igraphlayout":{"type":"square"}},"evals":[],"jsHooks":[]}</script>

If we want our carol graph to look more like a snowflake we can add a color column to our nodes `data.frame` & re-plot.

```r
nodes[, color := "lightblue"]

#plot whole document
visNetwork(nodes, edges) %>% 
  visIgraphLayout()
```

<div id="htmlwidget-3" style="width:672px;height:480px;" class="visNetwork html-widget"></div>
<script type="application/json" data-for="htmlwidget-3">{"x":{"nodes":{"id":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45],"label":["you","well","watch","out","not","cry","better","pout","i","be","tell","why","santa","claus","come","to","town","he","have","make","a","list","and","check","it","twice","go","find","who","naughty","nice","see","when","sleep","know","awake","if","bad","or","good","so","for","goodness","sake","o"],"group":["PRON","ADV","VERB","PART","ADV","VERB","PROPN","VERB","PRON","VERB","VERB","ADV","PROPN","PROPN","VERB","ADP","NOUN","PRON","VERB","VERB","DET","NOUN","CCONJ","VERB","PRON","ADV","VERB","VERB","NOUN","ADJ","ADJ","VERB","ADV","VERB","VERB","ADJ","ADP","ADJ","CCONJ","ADJ","ADV","ADP","NOUN","NOUN","INTJ"],"size":[5,5,10,5,5,10,5,10,5,5,5,5,25,25,5,5,15,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,10,5,5,5,5,5],"color":["lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue","lightblue"],"x":[-0.406997741194532,-0.828506933287726,-0.647510179256486,-0.640969463051876,-0.924438075209942,-0.776826845985658,-1,-0.975588353787003,-0.475805025628737,0.00950388547269032,-0.279756028486015,-0.0395817548035083,-0.42878649897815,-0.304400604064697,-0.151227094146622,-0.225197093168332,-0.242707282250002,0.175934583482307,0.391960012670138,0.244527719512905,0.498393352353324,0.391892854637003,0.532419835954278,0.0874558712564553,0.0852718845643381,-0.134275787155247,-0.0920061214988726,-0.404407880681409,0.692337076165161,0.667214168810666,0.917100952081051,-0.095379914069544,-0.188570335649042,-0.217629450803951,0.192836577206286,0.330583229209654,0.180182230264825,0.284805263760779,0.478734600708794,0.120151718186494,-0.0519303589041504,0.486696228156284,1,0.798528024000645,0.902742901314416],"y":[-0.329346731037925,-0.288250951826517,-0.145870404348929,0.159639258600777,0.119783855148691,-0.149679232457532,-0.156918415805316,0.331173524592039,-0.659723129004133,-0.442516084381877,-0.44836555685429,-0.00938255682665978,0.00482181678237237,-0.0911779942516898,-0.195080868995164,0.118020357945141,0.316756994037876,-0.0140084562827127,0.00440061770106981,0.404127456936708,0.87096183372323,0.677447855463552,0.418188306712293,0.767592583784342,1,0.927017095251227,0.539684343546947,0.404369099397648,-0.0365106456754857,0.219006572166182,0.234665249348246,-0.31260973924197,-0.752996670880449,-0.562813022891386,-0.248324727511852,-0.561863686008157,-0.656923430187999,-0.814650000675991,-1,-0.820628283532193,-0.85162200473787,-0.468380544485355,-0.40677602954361,-0.47829981744036,0.620495757542973]},"edges":{"to":[38,38,10,10,10,10,10,10,10,10,10,10,24,24,14,14,14,14,14,15,15,15,15,15,15,15,15,15,15,15,15,15,15,15,6,6,6,6,6,6,6,6,28,28,42,27,27,19,19,35,35,35,35,22,20,20,20,20,20,30,30,8,8,44,32,32,32,34,34,34,11,11,11,11,11,11,11,11,16,16,16,3,3,3,3,3,3,3],"from":[40,39,36,38,42,40,19,37,41,33,1,1,25,26,13,13,13,13,13,10,10,10,10,10,14,14,14,14,14,16,16,16,12,12,7,7,5,5,2,2,1,1,4,16,44,28,20,30,29,10,10,18,18,21,23,24,19,18,22,23,31,5,5,43,18,34,1,10,33,1,10,10,15,15,9,9,1,1,17,17,17,6,4,4,2,2,1,1],"label":["conj","cc","acomp","acomp","prep","acomp","aux","mark","advmod","advmod","nsubj","nsubj","dobj","advmod","compound","compound","compound","compound","compound","aux","aux","aux","aux","aux","nsubj","nsubj","nsubj","nsubj","nsubj","prep","prep","prep","advmod","advmod","advmod","advmod","neg","neg","advmod","advmod","nsubj","nsubj","prt","aux","pobj","xcomp","ccomp","acomp","nsubj","advcl","ccomp","nsubj","nsubj","det","cc","conj","aux","nsubj","dobj","cc","conj","neg","neg","compound","nsubj","advcl","dobj","aux","advmod","nsubj","aux","aux","ccomp","ccomp","nsubj","nsubj","dobj","dobj","pobj","pobj","pobj","ccomp","prt","prt","advmod","advmod","nsubj","nsubj"]},"nodesToDataframe":true,"edgesToDataframe":true,"options":{"width":"100%","height":"100%","nodes":{"shape":"dot","physics":false},"manipulation":{"enabled":false},"edges":{"smooth":false},"physics":{"stabilization":false}},"groups":["PRON","ADV","VERB","PART","PROPN","ADP","NOUN","DET","CCONJ","ADJ","INTJ"],"width":null,"height":null,"idselection":{"enabled":false},"byselection":{"enabled":false},"main":null,"submain":null,"footer":null,"background":"rgba(0, 0, 0, 0)","igraphlayout":{"type":"square"}},"evals":[],"jsHooks":[]}</script>

Happy Holidays!

<hr>

### Helper functions for preparing parse data for plotting

```r
#function to get mode of vector
stat_mode = function(x) {
  ux = unique(x)
  ux[which.max(tabulate(match(x, ux)))]
}

#function to prep nodes and edges dataframes
prep_nodes_and_edges = function(parse_df, label="lemma", size_scale=5,minimize_stopwords=TRUE) {
  dt_spacy_parse = data.table(parse_df)
  #create nodes df in format preffered by visNetwork
  nodes = dt_spacy_parse[,c(label, "pos"), with=FALSE]
  nodes = nodes[, .(mode_pos = stat_mode(pos),
                    N = .N), by=label]
  nodes[, id := 1:.N]
  setnames(nodes, c("label","group","size","id"))
  setcolorder(nodes, c("id","label","group","size"))
  
  if (minimize_stopwords) {
      # set stopwords to size 1
    nodes[label %in% tm::stopwords('SMART'), size := 1]
  }
  
  nodes[,size := size*size_scale]
  
  #create edges df in format preffered by visNetwork
  edges = dt_spacy_parse[,c("doc_id", "sentence_id", "token_id", label, "head_token_id", "dep_rel"), with=FALSE]
  edges = merge(edges, edges, 
                by.x=c("doc_id", "sentence_id","head_token_id"),
                by.y=c("doc_id", "sentence_id","token_id"))
  edges = edges[,c(paste(label, c("x","y"),sep="."), "dep_rel.x"), with=FALSE]
  data.table::setnames(edges, names(edges), c("from","to", "label"))
  
  #join in ids from nodes df
  edges = merge(edges, nodes[,.(id, label)], by.x="from", by.y="label")
  edges[,from := id]
  edges[,id := NULL]
  edges = merge(edges, nodes[,.(id, label)], by.x="to", by.y="label")
  edges[,to := id]
  edges[,id := NULL]
  
  list(nodes=nodes, edges=edges)
}
```
