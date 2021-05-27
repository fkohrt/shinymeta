# shinymeta

<!-- badges: start -->
[![R build status](https://github.com/rstudio/shinymeta/workflows/R-CMD-check/badge.svg)](https://github.com/rstudio/shinymeta/actions)
[![CRAN status](https://www.r-pkg.org/badges/version/shinymeta)](https://cran.r-project.org/)
[![Lifecycle: experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://lifecycle.r-lib.org/articles/stages.html)
<!-- badges: end -->

The **shinymeta** R package provides tools for capturing logic in a Shiny app and exposing it as code that can be run outside of Shiny (e.g., from an R console). It also provides tools for bundling both the code and results to the end user.

## Breaking changes in shinymeta 0.2.0

On August 6, 2019, we introduced a major syntax change into shinymeta, that is not compatible with the previous syntax. If you're here after watching @jcheng5's [useR2019 talk](https://www.youtube.com/watch?v=5KByRC6eqC8) or reading @zappingseb's [blog post](https://towardsdatascience.com/shinymeta-a-revolution-for-reproducibility-bfda6b329f68), please be aware that the `!!` operator has been replaced with a `..()` function. See [this page](https://github.com/rstudio/shinymeta/wiki/Syntax-changes-for-shinymeta-0.2.0) for details.

## Installation

**shinymeta** is not yet on CRAN, but you can install it via the **remotes** package:

```r
remotes::install_github("rstudio/shinymeta")
```

## Generating code with shinymeta

In short, **shinymeta** provides counterparts to Shiny's reactive building blocks (e.g., `reactive()` -> `metaReactive()`, `observe()` -> `metaObserve()`, `render()` -> `metaRender()`) that have the added ability to capture and expose the logic needed to recreate their current state *outside of the Shiny runtime*. Once the appropriate logic has been captured by these meta-counterparts and reactive reads (e.g., `input$var`, `var()`) have been marked by a special `..()` operator, then `expandChain()` can produce code to replicate the state of a particular meta-counterpart (e.g., `output$Summary`).

```r
library(shiny)
library(shinymeta)

ui <- fluidPage(
  selectInput("var", label = "Choose a variable", choices = names(cars)),
  verbatimTextOutput("Summary"),
  verbatimTextOutput("code")
)

server <- function(input, output) {
  var <- metaReactive({
    cars[[..(input$var)]]
  })
  output$Summary <- metaRender(renderPrint, {
    summary(..(var()))
  })
  output$code <- renderPrint({
    expandChain(output$Summary())
  })
}

shinyApp(ui, server)
```

<div align="center">
  <img src="https://i.imgur.com/5gNquPE.gif" width="67%" />
</div>

For more details, explanation, and overview **shinymeta** features, see the article on [code generation](https://rstudio.github.io/shinymeta/articles/code-generation.html) as well as [code distribution](https://rstudio.github.io/shinymeta/articles/code-distribution.html).

## A motivating example

Below is a screen-recording of a Shiny app which allows you to obtain CRAN download statistics for any combination of CRAN packages, look at those downloads using different aggregations, and produce a Rmd-based report _with code to reproduce the visualization_. This Shiny app is different from most in that it generates R code to reproduce what the user sees in the Shiny app (i.e., notice how the generated report reflects the user's input).

<img src="https://i.imgur.com/DJCCRiP.gif" />

We hope this example helps illustrate and inspire several reasons why you might want to generate standalone R code that mimics logic in your Shiny app:

* **Automation**: Shiny apps often use data that changes over time: stock quotes, sensor readings, centralized databases, etc. By providing users with reproducible R code, you enable them to take that logic into other workflows, such as creating a [periodic R Markdown email using RStudio Connect](https://docs.rstudio.com/connect/1.7.4/user/r-markdown-schedule.html) (e.g., the R code generated by the Shiny app above has been tailored so that anyone can re-run the code to acquire the latest download statistics).

* **Transparency**: By generating code that exposes the core logic of your **shiny** app, you make things more transparent for yourself and others, which has numerous benefits:

    * **Reassurance**: In the domain of data analysis, it's usually enough hard to know exactly how a result is derived, and be confident that it's 100% correct. Unfortunately, wrapping your analysis code in a user interface like Shiny make the situation even worse: the analysis code is now embedded inside a larger system making the overall logic even more error prone and difficult to fully understand. Export the core logic of your analysis can help reassure others (and yourself!) that your work is correct.
    
    * **Education**: For example, in the classroom, a student might use a Shiny app interactively to gain intuitive understanding of a statistical concept, then use the code to learn how to use the corresponding function from their own R scripts.
    
    * **Enabling**: Shiny is great for enabling others to interface with an R script you've written, but what if your users wish to explore things that your interface doesn't allow for? By exposing the core logic of your app, you make it easier for motivated users to modify and build upon your work in ways you never thought about.
    
    * **Documentation**: This one is especially relevant for exploratory analysis apps that allow you to derive insight from a dataset. A great example is the ANOVA example app in the [case studies vignette](case-studies.html#ANOVA), where you can upload a dataset, run an ANOVA analysis, then download a report with all ANOVA results as well as the code to reproduce it.

* **Permanence**: Using a Shiny app can have an ephemeral feeling to it; what happens in the future if the server goes down, or the app's features change? With a reproducible report, your users can download a more permanent artifact that can be saved locally.
    

## Acknowledgements

Many people projects provided motivation, inspiration, and ideas that have lead to **shinymeta**. Thanks especially to Adrian Waddell for inspiring the over-overarching metaprogramming approach and Doug Kelkhoff for his work on **scriptgloss**. Also thanks to Eric Nantz and Xiao Ni for providing feedback and insight on potential applications. We'd also like to acknowledge and highlight other work towards this same goal such as <http://intro-stats.com> (Eric Hare and Andee Kaplan) and <https://radiant-rstats.github.io/docs/> (Vincent Nijs).
