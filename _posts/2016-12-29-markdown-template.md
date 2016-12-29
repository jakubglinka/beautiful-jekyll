---
layout: post
title: "Markdown Template of post"
subtitle: This was added manualy
bigimg: /img/math.jpg
output:
  html_document:
    toc: no
  md_document:
    pandoc_args: --latexmathml
    variant: markdown_github+tex_math_dollars+autolink_bare_uris+ascii_identifiers
---



## GitHub Documents

This is an R Markdown format used for publishing markdown documents to GitHub. When you click the **Knit** button all R code chunks are run and a markdown file (.md) suitable for publishing to GitHub is generated.

## Including Code

You can include R code in the document as follows:


```r
summary(cars)
```

```
##      speed           dist       
##  Min.   : 4.0   Min.   :  2.00  
##  1st Qu.:12.0   1st Qu.: 26.00  
##  Median :15.0   Median : 36.00  
##  Mean   :15.4   Mean   : 42.98  
##  3rd Qu.:19.0   3rd Qu.: 56.00  
##  Max.   :25.0   Max.   :120.00
```

## Including Plots

You can also embed plots, for example:

![plot of chunk pressure]({{ site.url }}/img/2016-12-29-markdown-template-pressure-1.png)

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.

## Including Formulas

$$ \sum_{i = 1}^n \frac{1}{i} = \infty $$

Obviously since $\int_{\mathbb{R}} \frac{1}{x} dx = + \infty$ as in [^1]

## Including code

    if (a > 3) {
      moveShip(5 * gravity, DOWN);
    }


[^1]: Abramovitz and Steingun
