# (PART\*) Getting started {.unnumbered}

# Introduction to Tidy Finance

The main aim of this chapter is to familiarize yourself with the `tidyverse`. We start by downloading and visualizing stock data before we move to a simple portfolio choice problem. These examples introduce you to our approach of *Tidy Finance*.

## Working with stock market data

At the start of each session, we load the required packages. 
Throughout the entire book we always use the package `tidyverse`.
In this chapter we load the convenient `tidyquant` package to download price data. 
You typically have to install a package once before you can load it. In case you have not done this yet, call `install.packages("tidyquant")`. 
If you have trouble using `tidyquant`, check out the [documentation](https://cran.r-project.org/web/packages/tidyquant/vignettes/TQ01-core-functions-in-tidyquant.html#yahoo-finance). 


```r
library(tidyverse)
library(tidyquant)
```

We first download daily prices for one stock market ticker, e.g., *AAPL*, directly from the data provider Yahoo!Finance. 
To download the data, you can use the command `tq_get`. If you do not know how to use it, make sure you read the help file by calling `?tq_get`. 
We especially recommend taking a look at the documentation's examples section. 


```r
prices <- tq_get("AAPL", get = "stock.prices")
```

```
## Warning: x = 'AAPL', get = 'stock.prices': Error in new.session(): Could not establish session after 5 attempts.
```

```r
prices
```

```
## [1] NA
```

`tq_get` downloads stock market data from Yahoo!Finance if you do not specify another data source. The function returns a tibble with eight quite self-explanatory columns: *symbol*, *date*, the market prices at the *open, high, low* and *close*, the daily *volume* (in number of traded shares), and the *adjusted* price in USD. The adjusted prices are corrected for anything that might affect the stock price after the market closes, e.g., stock splits and dividends. These actions do affect the quoted prices, but they have no direct impact on the investors who hold the stock.  

Next, we use `ggplot2` to visualize the time series of adjusted prices. 



































