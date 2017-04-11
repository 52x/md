title: R Markdown Review
date: 2015-10-01 23:38:08
categories: Codage|编程
tags: [R, rmarkdown, knitr, markdown]
---

![knitr](/img/knit-logo.png)

Quoting the introduction on RStudio official site:

> R Markdown is an authoring format that enables easy creation of dynamic documents, presentations, and reports from R. It combines the core syntax of markdown (an easy-to-write plain text format) with embedded R code chunks that are run so their output can be included in the final document. R Markdown documents are fully reproducible (they can be automatically regenerated whenever underlying R code or data changes).

<!--more-->
When you use `RMarkdown` in R, you need to install this package manually. If you use RStudio, `RMarkdown` and `knitr` are pre-installed.
There is a `knit` button on top of the editor window which helps you to convert `.Rmd` file to specified format.

Frequent Used Syntax
--------------------

Per below you could find some frequent used syntax:

-   `#` for **header** - level 1, one more `#` leads to one level down
-   `*` or number like `1`, `2`, etc. at the beginning of a sentence for creating a **list**, unordered or ordered.
-   `*<blabla>*` for **italic**, `**<blabla>**` for **bold**, `_<blabla>_` for **underscores**, `~~<blabla>~~` for **strikethrough**.
	* *italic*
	* **bold**
	* _underscore_
	* ~~strikethrough~~
-   A pair of `` `<your code>` `` for quoting **a smaill piece of inline code**, if you embed a piece of code in `` `r ` ``, R Markdown will run the code in between automatically and replace it with the result.
-   A string of 3 backticks following `` {r} `` to start a **chunk of code** indicating the chunk of code is programmed by R. To end this chunk, you just need to add another 3 backticks.
	````
	```{r}
	<your code>
	```
	````
-   To create a **block quotation**, each line preceded by a `>` character and a space. (The `>` need not start at the left margin, but it should not be indented more than three spaces.)
-   `[website_name](http://...)` set a **hyperlink**
-   Use `$$ <equition> $$` to display **equations** in a new line and `$ <equition> $` for inline equation. You can use all of the [*standard latex math
    symbols*](https://en.wikibooks.org/wiki/LaTeX/Mathematics) to create attractive equations.
-   Three or more asterisks `*****` or dashes `-----` are used as **horizontal rule** or **page break**.
-   To create a **table**:

          First Header  | Second Header
          --------------|--------------
          Content Cell  | Content Cell
          Content Cell  | Content Cell

You could check the link for more [*basic syntax*](http://rmarkdown.rstudio.com/authoring_basics.html).

Practical Syntax
----------------

### Chunk Options

A chunk option is inserted right beside `{r <option>}`. 
For instance, `{r message = FALSE}` means all messages generated in the chunk is suppressed. Along with `message`, there are also `warning` and `error` which control the info delivery from the chunk. Besides, there are 3 options controlling code processing.

| option          | code display | code processing | result display |
|-----------------|--------------|-----------------|----------------|
| `echo=FALSE`    | X            | O               | O              |
| `eval=FALSE`    | O            | X               | X              |
| `results='hide'`| O            | O               | X              |

### Reference Label

An interesting feature available in knitr is the labeling of code snippets. The code chunk below would be assigned the label `summary_mtcars_disp`:

````
    ```{r summary_mtcars_disp, results = 'hide'}
    summary(mtcars$disp)
    ```
````

However, because the results option is equal to hide, no output is shown. This is what appears in the output document:

    summary(mtcars$disp)

What purpose do these labels serve? `knitr` provides the option `ref.label` to refer to previously defined and labeled code chunks. If used correctly, `knitr` will copy the code of the chunk you referred to and repeat it in the current code chunk. This feature enables you to separate R code and R output in the output document, without code duplication. Let's continue the example; the following code chunk:

````
    ```{r ref.label='summary_mtcars_disp', echo = FALSE}
    ```
````

produces the output you would expect:

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##    71.1   120.8   196.3   230.7   326.0   472.0

Notice that the `echo` option was explicitly set to `FALSE`, suppressing the R code that generated the output. There is a myriad of options available for code chunks, you can discover them at [*Yihui Xie's website*](http://yihui.name/knitr/options/)

### Alternative Output Formats

#### Basic Options

When you create a `.Rmd` file, RStudio automatically generates a YAML field like:

    ---
    title: "This is a title"
    author: "I am the author"
    date: "October 1, 2015"
    output: html_document
    ---

The first 3 lines generates the title author and date in the output `html` document. There are 3 default output format: `html_document`, `pdf_document` and `word_document`. Besides, you can also set it to be `md_document`. The latter output method is very helpful when you want to display `.Rmd` in a static blog. (WordPress, Hexo, etc.)

Notice that to visualize data in a pdf document, you will have to use the `ggplot2` package as an alternative to the `ggvis` package. This is for a reason: the `ggvis` package creates graphs that are HTML objects. These graphs are useful for HTML documents, but cannot be included in a pdf document without intermediary steps.

You can also export your file as a slideshow by changing the output field to one of `beamer_presentation`, `ioslides_presentation` and `slidy_presentation`.

which creates a slidy HTML slideshow. R Markdown will start a new slide at each first or second level header in your document. You can insert additional slide breaks with markdown's horizontal rule syntax:

    ***

#### Specify knitr And pandoc Options

Each R Markdown output template is a collection of `knitr` and `pandoc` options. You can customize your output by overwriting the default options that come with the template. For example, the YAML header below overwrites the default code highlight style of the pdf\_document template to create a document that uses the `zenburn` style:

    ---
    title: "Demo"
    output:
      pdf_document:
        highlight: zenburn
    ---

The YAML header below overwrites the default bootstrap CSS theme of the `html_document` template.

    ---
    title: "Demo"
    output:
      html_document:
        theme: spacelab
    ---

Pay close attention to the indentation of the options inside the YAML header; if you do not do this correctly, `pandoc` will not correctly understand your specifications. As an example, notice the difference between only specifying the output document to be HTML:

    ---
    output: html_document
    ---

and specifying an HTML output document with a different theme:

    ---
    output:
      html_document:
        theme: spacelab
    ---

You can learn more about popular options to overwrite in the [R Markdown Reference
Guide](https://www.rstudio.com/wp-content/uploads/2015/03/rmarkdown-reference.pdf).

#### Brand the Reports with Style Sheets

In the last exercise, we showed a way to change the CSS style of your HTML output: you can set the theme option of `html_document` to one of `default`, `cerulean`, `journal`, `flatly`, `readable`, `spacelab`, `united`, or `cosmo`. (Try it out). But what if you want to customize your CSS in more specific ways? You can do this by writing a `.css` file for your report and saving it in the same directory as the `.Rmd` file. To have your report use the CSS, set the `css` option of `html_document` to the file name, like this

    ---
    title: "Demo"
    output:
      html_document:
        css: styles.css
    ---

Custom CSS is an easy way to add branding to your reports.

#### Interactive Reports

`shiny` is an R package that uses R to build interactive web apps such as data explorers and dashboards. You can add `shiny` components to an R Markdown file to make an interactive document. You can also use R Markdown to create reports that use interactive `ggvis` graphics. `ggvis` relies on the `shiny` framework to create interactivity.

When you do this, you must ensure that you add runtime: shiny to the file's YAML header. You need to use an HTML output format (like `html_document`, `ioslides_presentation`, or `slidy_presentation`).

To learn more about interactivity with `Shiny`, `ggvis` and R, visit [*Shiny in RStudio*](shiny.rstudio.com).

-----------------------------------
## Reference:
[Reporting with R Markdown](https://www.datacamp.com/courses/reporting-with-r-markdown)
