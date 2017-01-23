---
layout: post
title: "Sampling from shifted Gompertz distribution"
subtitle: using Accept-Reject method 
bigimg: /img/math.jpg
output:
  html_document:
    toc: no
  md_document:
    pandoc_args: --latexmathml
    variant: markdown_github+tex_math_dollars+autolink_bare_uris+ascii_identifiers
tags: R
---



## Shifted Gompertz distribution

Shifted Gompertz distribution is useful distribution which can be used to describe time needed for adopting new innovation within the market. Recent studies showed that it outperforms *Bass* model of diffusion in some cases[^1]. 

Its *pdf* is given by 

$$p(t | b, \eta) = b\exp(-bt) \exp(-\eta e^{-bt})[1 + \eta(1 - \exp(-bt))]$$

Below we show what happens if we increase $\eta$ parameter (inverse of propensity to adopt) for different values of $b$ **appeal of innovation**.
Higher $\eta$ parameter is set more density mode is shifted away from $0$.

![plot of chunk unnamed-chunk-1]({{ site.url }}/img/2016-12-30-diffusion-part1-unnamed-chunk-1-1.png)

This distribution do not have closed form solutions for moments. If we want to calculate them and also simulate data for model validation we need to be able to sample from it.
Luckily there is a simple way of producing samples from arbitrary density:  

**Fundamental Theorem of Simulation**.  
*Simulating*

$$ X \sim f(x) $$

*is equivalent to simulating*

$$ (X, U) \sim \mathbb{U}\{(x, u) : 0 < u < f(x)\} \ \ \ \ \blacksquare $$

Direct corollary introduces convenient way of simulating from target distribution:

**Accept-Reject algorithm.**  
*Let $X \sim f(x)$ and let $g(x)$ be a density function that satisfies $f(x) \leq M g(x)$ for some constant $M \geq 1$ and $h:=Mg$. 
Then, to simulate  $X \sim f$ it is sufficient to generate pair*

$$ Y \sim g \ \ \ \ and \ \ \ \  (U|Y = y) \sim \mathbb{U}[0, h(y)], $$

*until $0 < u < f(x).$ $\ \ \ \ \ \ \blacksquare$*  

Function $h$ is sometimes called *envelope* function. More tight the inequality the more efficient sampling will be.

## Simulating from Shifted Gumbel distribution

Let's assume for a moment that our density has bounded support.
Below we plot an example of using **Accept-Reject** method for sampling from target density with simple constant envelope function:


![plot of chunk unnamed-chunk-2]({{ site.url }}/img/2016-12-30-diffusion-part1-unnamed-chunk-2-1.png)

This way of simulating distribution works but is not particularly efficient as more than 75% of sample is rejected and that's with assumption that the support of target density is in $(0, 40)$. If we would extend support of the target density to further away from $0$ we would see increasing drop of sampler efficiency.

We can easily improve our sampling method by noticing the following inequality:

$$\forall_{t>0} \ \ p[t|b,\eta] \le (1 + \eta) b\exp(-bt) = h(t)$$

This will provide us nice majorization function for the tail of shifted Gompertz distribution by scaled exponential density. 

In order to make it more tight around mode of our target density we can define improved envelope function as:

$$h'(t) =  1_{[0, t^*]} * m(b, \eta) + 1_{(t^*, +\infty)} * (1 + \eta) * b \exp(-bt)$$


where $t^*$ is a point where envelope function reaches maximum of shifted Gompertz density and $m$ is a value of the density at this point.

Of course $h'$ is not a density itself. Since it is integrable we can recover density $g$ as

$$ g(t| b, \eta) := \frac{h'(t|b, \eta)}{\int_{0}^{\infty} h'(s|b, \eta) \mathrm{d}s} = \frac{1}{M(b, \eta)} h'(t|b,\eta)$$

$$ = \frac{1}{M(b, \eta)} \left\{ 1_{[0, t^*]} * m(b, \eta) + 1_{(t^*, +\infty)} * (1 + \eta) * b \exp(-bt) \right\} $$

$$ = \frac{m(b,\eta) t^*}{M(b,\eta)} * \frac{1}{t^*}1_{[0, t^*]}  + \frac{m(b,\eta) e^{-bt^*}(1+\eta)}{M(b,\eta)} * 1_{(t^*, +\infty)} b \exp(-b(t - t^*))$$

$$ = p_1(b,\eta) * \mathbb{U}[0, t^*] + p_2(b, \eta) * \mathrm{Exp}(b)|_{t^*}^{\infty} $$

which is mixture of uniform and truncated exponential distribution.

### Algorithm pseudocode

    select component with probabilities p1(b, eta) and p2(b, eta)
    Repeat
      sample y from selected density component
      sample u from U[0, h'(y|b, eta)]
    Until u < p(y|b, eta)

### Sampling results


![plot of chunk unnamed-chunk-3]({{ site.url }}/img/2016-12-30-diffusion-part1-unnamed-chunk-3-1.png)
    
### Kolmogorov Smirnov test

It's straightforward to test whether we sample from correct distributions by comparing empirical cumulative distribution function with exact one. 





```

	One-sample Kolmogorov-Smirnov test

data:  s
D = 0.0018236, p-value = 0.8936
alternative hypothesis: two-sided
```

Perfect! Sampled data distribution is indistinguishable from theoretical density. We can also inspect sampling results visually:

![plot of chunk unnamed-chunk-6]({{ site.url }}/img/2016-12-30-diffusion-part1-unnamed-chunk-6-1.png)

## Summary

We created from scratch new sampler for shifted Gompertz distribution :) and it's fairly efficient: for one milion of sampled values it takes latter algorithm only 0.131 of a second. 

Code for this post can be found here:
<https://github.com/jakubglinka/posts/tree/master/diffusion_part1>

[^1]: Bauckhage, Christian; Kersting, Kristian (2014). "Strong Regularities in Growth and Decline of Popularity of Social Media Services"
