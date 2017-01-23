---
layout: post
title: "Applying diffusion theory to Google Trends"
subtitle: on example of Candy Crush Saga adoption
bigimg: /img/math.jpg
output:
  html_document:
    toc: no
  md_document:
    pandoc_args: --latexmathml
    variant: markdown_github+tex_math_dollars+autolink_bare_uris+ascii_identifiers
tags: R
---



Since I was recently researching different diffussion models I thought it would be nice to see whether they are applicable to popularity of information researched online.

I could choose any topic but I instantly remembered time when Candy Crash Saga was very popular in Poland. I wondered whether it was similar in other countries.

For those not familiar with the topic. Candy Crush Saga is a mobile game based on connect-4 principle. 

<img src="{{ site.url }}/img/Candy_Crush_logo.png" style="width: 200px;"/>

<https://en.wikipedia.org/wiki/Candy_Crush_Saga>

## Getting data from Google Trends

I used [gtrendsR](https://cran.r-project.org/web/packages/gtrendsR/index.html) package to retrieve relative number of searches of CCS over time.

Below is example how we can retrieve term popularity for given keyword and country:


```r
  library(gtrendsR)
  usr <- "user@gmail.com"
  psw <- "pwd"            
  gconnect(usr, psw)
  
  # "/m/0rytj3p" is a hashcode for CCS mobile game
  ccs_trend <- gtrends(query = "/m/0rytj3p", 
                         start_date = "2012-04-01",
                         geo = c("US", "PL"))
```

There are important notes that I need to make when it comes to retrieve data via this Google Trends API:

 - there is an user quota something about 500 requests a day
 - you can issue only 10 requests per minute
 - it is relative keyword popularity within country
 - requested data is always normalized to max = 100 (I kept US as a reference category)
 - if there is not enough data in any of the selected countries it will return error
 - if you want to use more complex search term like *CCS mobile game* you need to manually select it in Google Trends and copy the hash code of the term from embedding script - [details](https://support.google.com/trends/answer/4365538?hl=en)
 
I must admit overall this was more challenging than expected but as long as you keep all those limitations in mind everything should be fine. At the end of the day I ended up with weekly data from 61 countries.

Below I plotted randomly selected countries to see what different popularity patterns we can observe:

![plot of chunk unnamed-chunk-2]({{ site.url }}/img/2017-01-23-diffusion-part2-unnamed-chunk-2-1.png)

## Diffusion Theory

Diffusion theory explains how information expands through connected systems via three parameters:

 - **p - appeal of innovation**
 - **q - propensity to immitate**
 - **m - ultimate potential**

Specific models have different dependencies on those parameters or assume their heterogeneity in population under study. Here I will use *shifted Gompertz* distribution but other choices are possible (*Bass, Gamma Shifted Gompertz, Weibull*). 
Examples of the evolution curves based on SG distribution may be found [here](http://www.jakubglinka.com/2016-12-30-diffusion-part1/).

### Shifted Gompertz diffusion model

We assume that adoption of the app is following shifted Gompertz distribution with pdf:

$$f(t|b, \eta) = b e^{-bx}e^{-\eta e^{-bx}}[1 + \eta(1 - e^{-bx})], \ \ \ \mathrm{for} \ \ t \ge 0$$

where $p = be^{-\eta}$ and $q = b - p$.

Let $F$ denote cumulative density function of SG and since we have weekly aggregated data we will assume that $t$ is measured in weeks. Then term popularity in week $t$ is equal to:

$$ N_t = m * (F(t| b, \eta) - F(t - 1 | b, \eta)) $$

Usually diffusion models are estimated using non-linear least squares technique[^1] which is equivalent to

$$ S_t = m * (F(t| b, \eta) - F(t - 1 | b, \eta)) + \epsilon, \ \ \ \mathrm{where} \ \ \epsilon \sim \mathrm{N}[0, \sigma] $$

where $S_t$ denotes observed term popularity in week $t$.

Below is my Stan implementation of this model:


```stan

data {
  int<lower = 2> T;                         // number of observed periods
  real<lower = 0> S[T];                     // share of users adoptions
}

parameters {
  real<lower=0> b;                          // appeal of innovation
  real<lower=0> eta;                        // rate of adoption
  real<lower=0> sigma;
  real<lower=0> m;
}

transformed parameters {
  vector[T] share;
  share = sg_weekly_share(T, m, b, eta);    // N_t
}

model {
      S ~ normal(share, sigma);
}

generated quantities {
  vector[T] pred_s;
  real p;
  real q;
  
  pred_s = share;
  p = b * exp(- eta);
  q = b - p;
}
```



### Cross-country comparison

Below I plotted all the countries with respect to the $p$ and $q$. 
Size of the label is proportional to the log of keyword popularity $m$.

![plot of chunk unnamed-chunk-5]({{ site.url }}/img/2017-01-23-diffusion-part2-unnamed-chunk-5-1.png)

I grouped countries with respect to speed and dynamics of adoption. 
Below I plotted selected countries from the respective groups.
Red line is theoretical diffusion curve of fitted SG model. I plotted additionally 95% confidence interval.

### Early Adopters

![plot of chunk unnamed-chunk-6]({{ site.url }}/img/2017-01-23-diffusion-part2-unnamed-chunk-6-1.png)

Sweden had highest $p$ parameter which means that it adopted CCS fastest along with Great Britain, Netherlands and Greece. 

### Late boom

![plot of chunk unnamed-chunk-7]({{ site.url }}/img/2017-01-23-diffusion-part2-unnamed-chunk-7-1.png)

Taiwan had very steep adoption rate which according to this article exceeded makers expectations:  
<http://www.chinapost.com.tw/taiwan/business/2013/09/17/389125/Candy-Crushs.htm>  
Similar reports can be found for Hong Kong.

### Late Majority

![plot of chunk unnamed-chunk-8]({{ site.url }}/img/2017-01-23-diffusion-part2-unnamed-chunk-8-1.png)

So in the end I was surprised to see that Poland was rather late adopter of CCS compared to other countries!

## Summary

In this post we analysed Google Trends data using diffusion theory which summarizes nicely popularity evolution of Candy Crush Saga searches between 14 November 2012 and 20 October 2014 and lets us to summarize differences across the countries.

Code for this post can be found here:
<https://github.com/jakubglinka/posts/tree/master/diffusion_part2>

[^1]: The Impact of Heterogeneity and Ill-Conditioning on Diffusion Model Parameter Estimates Albert C. Bemmaor, Janghyuk Lee

