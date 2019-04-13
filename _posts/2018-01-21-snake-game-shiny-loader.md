---
title: Snake Game Shiny Loader
author: ~
date: '2018-01-21'
slug: snake-game-shiny-loader
categories: ['r','shiny']
tags: ['r', 'shiny', 'game']
---

This post is to share the [🐍`snakeLoadR`🐍](https://github.com/AdamSpannbauer/snakeLoadR) R package.  The package adds the snake game as a loading screen tied to a specific output in a shiny app.

![](/assets/2018/01/snake_load.gif){: .center-image width="95%" }

[This repo](https://github.com/AdamSpannbauer/shiny_snake_loader) has the code used to make the app in the gif.


****

This loader package is more of a novelity than it is anything useful, but it was a fun little project.  If you're interested in using the package yourself you can install it using:

```r
devtools::install_github("AdamSpannbauer/snakeLoadR")
```

To use the snake loader in a shiny app, you need to include the `snakeLoadR::snake_loader` function in your UI and provide an `outputId` that the loader will be tied to.  The below chunk shows a minimal shiny app using the snake loader.

```r
library(shiny)
library(snakeLoadR)

shinyApp(
  shinyUI(
    fluidPage(
      fluidRow(
        column(width=10, offset=1, algin="left",
               actionButton("my_button", "Start Fake 30 Second Job"),
               uiOutput("my_output")
        )
      ),
      snakeLoadR::snake_loader(outputId = "my_output",
                               header = "Play Snake while you wait!",
                               controls = TRUE)
    )
  ),
  shinyServer(function(input, output) {
    output$my_output <- renderUI({
      if(input$my_button != 0) Sys.sleep(30)
      HTML(paste0("<h3>Fake job completed <code>", input$my_button,"</code> times!</h3>"))
    })
  })
)
```
