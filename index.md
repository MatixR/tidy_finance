--- 
title: "Tidy Finance with R"
author: 
  - Christoph Scheuch, wikifolio Financial Technologies 
  - Stefan Voigt, University of Copenhagen and Danish Finance Institute
  - Patrick Weiss, Vienna University of Economics and Business
date: "2022-02-28"
site: bookdown::bookdown_site
output: bookdown::bs4_book
documentclass: book
bibliography: [book.bib]
biblio-style: apalike
link-citations: yes
github-repo: voigtstefan/tidy_finance
url: https://www.tidy-finance.org
cover-image: cover.jpg
description: |
  An open-source textbook for empirical finance applications with R. 
---

# Preface {.unnumbered}

[![Buy hardcover version](cover.jpg){.cover width="250"}]()
This website is the online version of *Tidy Finance with R*, a book currently under development and intended for eventual print release. The book is the result of a joint effort of [Christoph Scheuch](https://christophscheuch.github.io/), [Stefan Voigt](https://voigtstefan.me/), and [Patrick Weiss](https://sites.google.com/view/patrick-weiss). 

We are grateful for any kind of feedback on *every* aspect of the book. So please get in touch with us via [contact@tidy-finance.org](mailto:contact@tidy-finance.org) if you spot typos, discover any issues that deserve more attention, or if you have suggestions for additional chapters and sections. Additionally, let us know if you found the text helpful. We look forward to hearing from you!

## Why does this book exist? {.unnumbered}

Financial economics is a vibrant area of research, a central part of all businesses activities, and at least implicitly relevant for our everyday life. Despite its relevance for our society and a vast number of empirical studies of financial phenomenons, one quickly learns that the actual implementation is typically rather opaque. 
As graduate students, we were particularly surprised by the lack of public code for seminal papers or even textbooks on key concepts of financial economics. The lack of transparent code not only leads to numerous replication efforts (and their failures), but it also constitutes a waste of resources on problems that have already been solved by countless others in secrecy.

This book aims to lift the curtain on reproducible finance by providing a fully transparent code base for many common financial applications. We hope to inspire others to share their code publicly and take part in our journey towards more reproducible research in the future. 

## Who should read this book? {.unnumbered}

We write this book for three audiences:

* Students who want to acquire the basic tools required to conduct financial research ranging from undergrad to graduate level. The structure of the book is kept simple enough such that the material is sufficient for self-study purposes.  
* Instructors who look for materials to teach in empirical finance courses. We provide plenty of examples and (hopefully) intuitive explanations which can easily be adjusted or expanded.   
* Data analysts or statisticians who work on issues pertaining to financial data and need practical tools to do so. 

## What will you learn? {.unnumbered}

The book is currently divided into 5 parts:

* Chapter 1 introduces you to important concepts around which our approach to Tidy Finance revolves. 
* Chapter 2 provides tools to organize your data and prepare the most common data sets used in financial research: CRSP and Compustat.
* Chapters 3-7 deal with key concepts of empirical asset pricing such as beta estimation, portfolio sorts, and performance analysis. 
* Chapters 8-9 apply machine learning methods to problems in factor selection and option pricing. 
* Chapters 10-11 provide approaches for parametric, constrained portfolio optimization, and backtesting procedures.  

The number of chapters and covered content is subject to change as we will introduce additional material in the near future. 

## What won’t you learn? {.unnumbered}

This book is about empirical work. While we assume only basic knowledge in statistics and econometrics, we do not provide detailed treatments of the underlying theoretical models or methods applied in this book. Instead, you find references to the seminal academic work in journal articles and to more detailed treatments. 
We believe that our comparative advantage is to provide a thorough implementation of portfolio sorts, backtesting procedures, machine learning methods, or other related topics in empirical finance and enrich these implementations with discussions of the needy-greedy choices you face while conducting empirical analyses. We hence refrain from deriving theoretical models or discussing the statistical properties of well-established tools.

## Why R? {.unnumbered}

We believe that R is among best choices for a programming language in the area of finance. Some of our favorite features include:

- R is free and open source, so you can use it in academic and professional contexts.
- A diverse and active online community works on a broad range of tools.
- A massive set of actively maintained packages for all kinds of applications exists, e.g., data manipulation, visualization, machine learning, etc.
- Powerful tools for communication, e.g., Rmarkdown and shiny, are redily available.
- RStudio is one of the best development environments for interactive data analysis.
- Strong foundation of functional programming are provided.
- Smooth integration with other programming languages, e.g., SQL, Python, C, C++, Fortran, etc.

For more information, we refer to @Wickham2019.

## Why tidy? {.unnumbered}

As you start working with data, you quickly realize that you spend a lot of time reading, cleaning, and transforming your data. In fact, it is often said that more than 80% of data analysis is spent on preparing data. By *tidying data*, we want to structure data sets to facilitate further analyses. As @Wickham2014 puts it: 

>[T]idy datasets are all alike, but every messy dataset is messy in its own way. Tidy datasets provide a standardized way to link the structure of a dataset (its physical layout) with its semantics (its meaning). 

In its essence, tidy data follows these three principles:

1. Every column is a variable.
2. Every row is an observation.
3. Every cell is a single value.

Throughout this book, we try to follow these principles as best as we can. If you want to learn more about tidy data principles in an informal manner, we refer you to [this vignette](https://cran.r-project.org/web/packages/tidyr/vignettes/tidy-data.html).

In addition to the data layer, there are also tidy coding principles outlined in [the tidy tools manifesto](https://tidyverse.tidyverse.org/articles/manifesto.html) that we try to follow: 

1. Reuse existing data structures.
2. Compose simple functions with the pipe.
3. Embrace functional programming.
4. Design for humans.

In particular, we heavily draw on a set of packages called the [`tidyverse`](https://tidyverse.tidyverse.org/index.html) [@Wickham2019]. The `tidyverse` is a consistent set of packages for all data analysis tasks, ranging from importing and wrangling to visualizing and modeling data with the same grammar. In addition to explicit tidy principles, the `tidyverse` has further benefits: (i) if you master one package, it is easier to master others, and (ii) the core packages are developed and maintained by the Public Benefit Company RStudio, Inc. 

## Prerequisites {.unnumbered}

Before we continue, make sure you have all the software you need for this book:

- [Install R and RStudio](https://rstudio-education.github.io/hopr/starting.html#starting). To get a walk-through of the installation for every major operating system, follow the steps outlined [in this summary](https://rstudio-education.github.io/hopr/starting.html#starthng). The whole process should be done in a few clicks. If you wonder about the difference: R is an open-source language and environment for statistical computing and graphics, free to download and use. While R runs the computations, RStudio is an integrated development environment that provides an interface by adding many convenient features and tools. We suggest doing all the coding in RStudio.
- Open RStudio and [install the `tidyverse`](https://tidyverse.tidyverse.org/). Not sure how it works? You find helpful information on how to install packages in this [brief summary](https://rstudio-education.github.io/hopr/packages2.html). 

If you are new to R, we recommend starting with the following sources:

- A very gentle and good introduction into the workings of R can be found in the form of the [weighted dice project](https://rstudio-education.github.io/hopr/project-1-weighted-dice.html). Once you are done setting up R on your machine, try to follow the instructions in this project.
- The main book on the `tidyverse` is available online and for free: [R for Data Science](https://r4ds.had.co.nz/introduction.html) by Hadley Wickham and Garrett Grolemund explains the majority of the tools we use in our book. 

## About the authors {.unnumbered}

We met at the [Vienna Graduate School of Finance](https://www.vgsf.ac.at/) from which each of us graduated with a different focus, but a shared passion: coding with R. We continue to sharpen our R skills as part of our current occupations:

* [Christoph Scheuch](https://christophscheuch.github.io/) is the Director of Product at the social trading platform [wikifolio.com](https://www.wikifolio.com/) where he is responsible for product planning, execution and monitoring. He also manages a team of data scientists to analyze user behavior and develop new products.
* [Stefan Voigt](https://voigtstefan.me/) is an Assistant Professor of Finance at the [Department of Economics at the University in Copenhagen](https://www.economics.ku.dk/) and a research fellow at the [Danish Finance Institute](https://danishfinanceinstitute.dk/). His research focuses on blockchain technology, high-frequency trading and financial econometrics. Stefan teaches parts of this book in his courses on empirical finance.
* [Patrick Weiss](https://sites.google.com/view/patrick-weiss) is a Post-Doc at the [Vienna University of Economics and Business](https://www.wu.ac.at/en/). His research centers around the intersection between asset pricing and corporate finance. 

## License {.unnumbered}

This book is licensed to you under [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).

The code samples in this book are licensed under [Creative Commons CC0 1.0 Universal (CC0 1.0), i.e. public domain](https://creativecommons.org/publicdomain/zero/1.0/).


<!-- ## Acknowledgements {.unnumbered} -->

<!-- ... -->

## Colophon {.unnumbered}

This book was written in RStudio using bookdown. The website is hosted with github pages and automatically updated after every commit. The complete source is [available from GitHub](www.github.com/voigtstefan/tidy_finance). We generated all plots in this book using `ggplot2` and its classic dark-on-light theme (`theme_bw()`). 

This version of the book was built with R version 4.1.2 (2021-11-01) and the following packages:

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> Package </th>
   <th style="text-align:left;"> Version </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> alabama </td>
   <td style="text-align:left;"> 2015.3-1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bookdown </td>
   <td style="text-align:left;"> 0.24.4 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> broom </td>
   <td style="text-align:left;"> 0.7.12 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dbplyr </td>
   <td style="text-align:left;"> 2.1.1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frenchdata </td>
   <td style="text-align:left;"> 0.2.0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> furrr </td>
   <td style="text-align:left;"> 0.2.3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ggplot2 </td>
   <td style="text-align:left;"> 3.3.5 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> glmnet </td>
   <td style="text-align:left;"> 4.1-3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> googledrive </td>
   <td style="text-align:left;"> 2.0.0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hardhat </td>
   <td style="text-align:left;"> 0.2.0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> jsonlite </td>
   <td style="text-align:left;"> 1.7.3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kableExtra </td>
   <td style="text-align:left;"> 1.3.4 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> keras </td>
   <td style="text-align:left;"> 2.8.0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> knitr </td>
   <td style="text-align:left;"> 1.37 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lmtest </td>
   <td style="text-align:left;"> 0.9-39 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lubridate </td>
   <td style="text-align:left;"> 1.8.0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> quadprog </td>
   <td style="text-align:left;"> 1.5-8 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> readxl </td>
   <td style="text-align:left;"> 1.3.1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> renv </td>
   <td style="text-align:left;"> 0.15.2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rlang </td>
   <td style="text-align:left;"> 1.0.1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rmarkdown </td>
   <td style="text-align:left;"> 2.11 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> RPostgres </td>
   <td style="text-align:left;"> 1.4.3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> RSQLite </td>
   <td style="text-align:left;"> 2.2.9 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sandwich </td>
   <td style="text-align:left;"> 3.0-1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> scales </td>
   <td style="text-align:left;"> 1.1.1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> slider </td>
   <td style="text-align:left;"> 0.2.2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tidymodels </td>
   <td style="text-align:left;"> 0.1.4 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tidyquant </td>
   <td style="text-align:left;"> 1.0.3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tidyverse </td>
   <td style="text-align:left;"> 1.3.1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> timetk </td>
   <td style="text-align:left;"> 2.7.0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wesanderson </td>
   <td style="text-align:left;"> 0.3.6 </td>
  </tr>
</tbody>
</table>
