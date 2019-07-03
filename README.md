# shinymeta

The **shinymeta** R package provides tools for capturing logic in a Shiny app and exposing it as code that can be run outside of Shiny (e.g., from an R console). It also provides tools for bundling both the code and results to the end user.

## Installation

**shinymeta** is not yet on CRAN, but you can install it via the **remotes** package:

```r
remotes::install_github("rstudio/shinymeta")
```

## Motivating example

Below is a screen-recording of a Shiny app which allows you to obtain CRAN download statistics for any combination of CRAN packages, look at those downloads using different aggregations, and produce a Rmd-based report with code to reproduce the visualization. This Shiny app is different from most in that it generates R code to reproduce what the user sees in the Shiny app (i.e., notice how the generated report reflects the user's input).

<img src="https://rstudio.github.io/shinymeta/articles/cranview-intro.gif" />

We hope this example helps to illustrate several reasons why you might want to generate standalone R code that mimics logic in your Shiny app:

* **Automation**: Shiny apps often use data that changes over time: stock quotes, sensor readings, centralized databases, etc. By providing users with reproducible R code, you enable them to take that logic into other workflows, such as creating a [periodic R Markdown email using RStudio Connect](https://docs.rstudio.com/connect/1.7.4/user/r-markdown-schedule.html) (e.g., the R code generated by the Shiny app above has been tailored so that anyone can re-run the code to acquire the latest download statistics).

* **Transparency**: By generating code that exposes the core logic of your **shiny** app, you make things more transparent for yourself and others, which has numerous benefits:

    * **Reassurance**: In the domain of data analysis, it's usually enough hard to know exactly how a result is derived, and be confident that it's 100% correct. Unfortunately, wrapping your analysis code in a user interface like Shiny make the situation even worse: the analysis code is now embedded inside a larger system making the overall logic even more error prone and difficult to fully understand. Export the core logic of your analysis can help reassure others (and yourself!) that your work is correct.
    
    * **Education**: For example, in the classroom, a student might use a Shiny app interactively to gain intuitive understanding of a statistical concept, then use the code to learn how to use the corresponding function from their own R scripts.
    
    * **Enabling**: Shiny is great for enabling others to interface with an R script you've written, but what if your users wish to explore things that your interface doesn't allow for? By exposing the core logic of your app, you make it easier for motivated users to modify and build upon your work in ways you never thought about.
    
    * **Documentation**: This one is especially relevant for exploratory analysis apps that allow you to derive insight from a dataset. A great example is the ANOVA example app in the [case studies vignette](04-case-studies.html#ANOVA), where you can upload a dataset, run an ANOVA analysis, then download a report with all ANOVA results as well as the code to reproduce it.

* **Permanence**: Using a Shiny app can have an ephemeral feeling to it; what happens in the future if the server goes down, or the app's features change? With a reproducible report, your users can download a more permanent artifact that can be saved locally.
    

## Generating code with shinymeta

In overly simplified terms, **shinymeta** provides variants of **shiny**'s reactive execution contexts (e.g., `reactive()` -> `metaReactive()`, `observe()` -> `metaObserve()`, `render()` -> `metaRender()`).  When called with **shinymeta**'s meta mode enabled, these variants return [quasi-quoted](https://adv-r.hadley.nz/quasiquotation.html) expressions, providing the **shiny** app author a flexible way to construct code expressions without duplication of the app's core logic. Here's a very basic example:

```r
library(shiny)
library(shinymeta)

ui <- fluidPage(
  sliderInput("n", "Number of samples", min = 1, value = 30, max = 100),
  plotOutput("p"),
  verbatimTextOutput("code")
)

server <- function(input, output) {
  output$p <- metaRender(renderPlot, {
    hist(rnorm(!!input$n))
  })
  output$code <- renderPrint({
    withMetaMode(output$p())
  })
}

shinyApp(ui, server)
```

Generating code with **shinymeta** that is correct and clean is not necessarily straight-forward. It requires a good grasp of **shiny** and metaprogramming. To learn more, read the article on [code generation](http://rstudio.github.io/shinymeta/articles/01-code-generation.html).

## Compared to alternatives

Combining interactivity and reproducibility is generally a very hard problem in computing. Shiny provides a great foundation for producing interactive artifacts, but reproducing those artifacts in another context is difficult. We do see **shinymeta** as a step forward for the reproducibility problem, but it is by no means *the solution*. As with most things, there are tradeoffs and other approaches you may want to consider. We think there are roughly three tradeoff categories:

 1. **Quality of output:** Does the generated code look clean, like a human would write it; or is it awkwardly structured and/or contain random boilerplate?
2. **Ease of implementation** How hard is it to add code generation to your app? This includes learning and maintenance of the codebase.
3. **Generality:** Can code generation features be added to _any_ Shiny app, or only apps that follow certain conventions? Does it work as well for large, complex apps as for small ones?

Furthermore, there are roughly three different approaches one could take, each scoring differently on the tradeoff categories (more stars means better):

### Approach 1: Copy and paste

> Quality of output: \*\*\*\*\*
>
> Ease of implementation: \*\* (but highly variable)
>
> Generality: \*\*

The simplest approach is to simply duplicate your logic. You have an `app.R` file that contains your Shiny app, and a separate `script.R` or `report.Rmd` file that contains the same logic in linear form (i.e. minus any of the structure that Shiny imposes). The `script.R` or `report.Rmd` contains placeholders that will be replaced by input values selected by the user.

This approach is easy to understand, but it means that you are stuck maintaining a parallel codebase that you must manually keep in sync. Over the long term, this can be a source of not only tedium but also bugs. On the plus side, because you're maintaining `script.R`/`report.Rmd` by hand, the level of code quality is limited only by your own skill.

### Approach 2: Mechanical transformation (scriptgloss)

> Quality of output: \*\*
>
> Ease of implementation: \*\*\*\*
>
> Generality: \*\*
 
This approach automatically transforms your reactives, outputs, and observers into linearized code, using predefined algorithms/heuristics. You as the app author are neither required nor able to influence the code generation process very much. The nice thing about this is how little effort it is -- you can add [scriptgloss](https://github.com/dgkf/scriptgloss) to your app in a couple of minutes!

The price you pay for all this automation is that the generated code looks pretty unnatural, with some Shiny-related wires sticking out. Plus, there are several common situations that will lead to the script not working; then the onus is on either the app author to gain a deeper understanding of scriptgloss and restructure the app to suit, or on the user to take the slightly broken code and fix it.

### Approach 3: Metaprogramming (shinymeta)

> Quality of output: \*\*\*\*
>
> Ease of implementation: \*\*
>
> Generality: \*\*\*\*\*

Of these three techniques, metaprogramming is by far the most conceptually challenging to get started with. In exchange for climbing the considerable learning curve, you get much more control over how the code output is generated. We also believe (but can't yet prove) that this approach can scale to larger, more complex apps, including ones that use Shiny modules.


## Acknowledgements

Adrian Waddell (especially), Eric Nantz, Xiao Ni, Doug Keklhoff

intRo?
https://github.com/gammarama/intRo/blob/papers/jcgs-paper-2016/paper-original.pdf
https://github.com/hadley/15-student-papers/blob/master/student-papers.pdf
