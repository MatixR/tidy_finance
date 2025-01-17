# (PART\*) Getting started {.unnumbered}

# Introduction to Tidy Finance

The main aim of this chapter is to familiarize yourself with the `tidyverse`. We start by downloading and visualizing stock data before moving to a simple portfolio choice problem. These examples introduce you to our approach of *Tidy Finance*.

## Working with stock market data

At the start of each session, we load the required packages. 
Throughout the entire book, we always use the package `tidyverse`.
In this chapter, we also load the convenient `tidyquant` package to download price data. 
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
prices <- tq_get("AAPL", get = "stock.prices", from = "2000-01-01", to = "2022-03-30")
prices
```

```
## # A tibble: 5,596 x 8
##   symbol date        open  high   low close    volume adjusted
##   <chr>  <date>     <dbl> <dbl> <dbl> <dbl>     <dbl>    <dbl>
## 1 AAPL   2000-01-03 0.936 1.00  0.908 0.999 535796800    0.856
## 2 AAPL   2000-01-04 0.967 0.988 0.903 0.915 512377600    0.784
## 3 AAPL   2000-01-05 0.926 0.987 0.920 0.929 778321600    0.795
## 4 AAPL   2000-01-06 0.948 0.955 0.848 0.848 767972800    0.726
## 5 AAPL   2000-01-07 0.862 0.902 0.853 0.888 460734400    0.761
## # ... with 5,591 more rows
```

`tq_get` downloads stock market data from Yahoo!Finance if you do not specify another data source. The function returns a tibble with eight quite self-explanatory columns: *symbol*, *date*, the market prices at the *open, high, low* and *close*, the daily *volume* (in number of traded shares), and the *adjusted* price in USD. The adjusted prices are corrected for anything that might affect the stock price after the market closes, e.g., stock splits and dividends. These actions affect the quoted prices, but they have no direct impact on the investors who hold the stock.  

Next, we use `ggplot2` to visualize the time series of adjusted prices. 


```r
prices %>%
  ggplot(aes(x = date, y = adjusted)) +
  geom_line() +
  labs(
    x = NULL, 
    y = NULL,
    title = "AAPL stock prices",
    subtitle = "Prices in USD, adjusted for dividend payments and stock splits"
  )
```

<img src="10_introduction_files/figure-html/unnamed-chunk-3-1.png" width="672" style="display: block; margin: auto;" />

Instead of analyzing prices, we compute daily returns defined as $(p_t - p_{t-1}) / p_{t-1}$ where $p_t$ is the adjusted day $t$ price. The function `lag` computes the previous value in a vector. 


```r
returns <- prices %>%
  arrange(date) %>%
  mutate(ret = (adjusted - lag(adjusted)) / lag(adjusted)) %>%
  select(symbol, date, ret)
returns
```

```
## # A tibble: 5,596 x 3
##   symbol date           ret
##   <chr>  <date>       <dbl>
## 1 AAPL   2000-01-03 NA     
## 2 AAPL   2000-01-04 -0.0843
## 3 AAPL   2000-01-05  0.0146
## 4 AAPL   2000-01-06 -0.0865
## 5 AAPL   2000-01-07  0.0474
## # ... with 5,591 more rows
```

The resulting tibble contains three columns where the last contains the daily returns. Note that the first entry naturally contains `NA` because there is no previous price. Additionally, the computations require that the time series is ordered by date. 
Otherwise, `lag` would be meaningless. 

For the upcoming examples, we remove missing values as these would require separate treatment when computing, e.g., sample averages. In general, however, make sure you understand why `NA` values occur and carefully examine if you can simply get rid of these observations. 


```r
returns <- returns %>%
  drop_na(ret)
```

Next, we visualize the distribution of daily returns in a histogram. For convenience, we multiply the returns by 100 to get returns in percent for the visualizations. Additionally, we also add a dashed red line that indicates the 5\% quantile of the daily returns to the histogram, which is a (crude) proxy for the worst return of the stock with a probability of at least 5\%. 


```r
quantile_05 <- quantile(returns %>% pull(ret) * 100, 0.05)

returns %>%
  ggplot(aes(x = ret * 100)) +
  geom_histogram(bins = 100) +
  geom_vline(aes(xintercept = quantile_05),
    color = "red",
    linetype = "dashed"
  ) +
  labs(
    x = NULL, 
    y = NULL,
    title = "Distribution of daily AAPL returns (in percent)",
    subtitle = "The dotted vertical line indicates the historical 5% quantile"
  )
```

<img src="10_introduction_files/figure-html/unnamed-chunk-6-1.png" width="672" style="display: block; margin: auto;" />

Here, `bins = 100` determines the number of bins and hence implicitly the width of the bins. 
Before proceeding, make sure you understand how to use the geom `geom_vline()` to add a dotted red line that indicates the 5\% quantile of the daily returns. 
A typical task before proceeding with *any* data is to compute summary statistics for the main variables of interest. 


```r
returns %>%
  mutate(ret = ret * 100) %>%
  summarize(across(
    ret,
    list(
      daily_mean = mean,
      daily_sd = sd,
      daily_min = min,
      daily_max = max
    )
  ))
```

```
## # A tibble: 1 x 4
##   ret_daily_mean ret_daily_sd ret_daily_min ret_daily_max
##            <dbl>        <dbl>         <dbl>         <dbl>
## 1          0.129         2.52         -51.9          13.9
```

We see that the maximum *daily* return was around 13.905 percent.  
You can also compute these summary statistics for each year by imposing `group_by(year = year(date))`, where the call `year(date)` computes the year.


```r
returns %>%
  mutate(ret = ret * 100) %>%
  group_by(year = year(date)) %>%
  summarize(across(
    ret,
    list(
      daily_mean = mean,
      daily_sd = sd,
      daily_min = min,
      daily_max = max
    ),
    .names = "{.fn}"
  ))
```

```
## # A tibble: 23 x 5
##    year daily_mean daily_sd daily_min daily_max
##   <dbl>      <dbl>    <dbl>     <dbl>     <dbl>
## 1  2000     -0.346     5.49    -51.9      13.7 
## 2  2001      0.233     3.93    -17.2      12.9 
## 3  2002     -0.121     3.05    -15.0       8.46
## 4  2003      0.186     2.34     -8.14     11.3 
## 5  2004      0.470     2.55     -5.58     13.2 
## # ... with 18 more rows
```

In case you wonder: the additional argument `.names = "{.fn}"` in `across()` determines how to name the output columns. The specification is rather flexible and allows almost arbitrary column names, which can be useful for reporting.

## Scaling up the analysis

As a next step, we generalize the code from before such that all the computations can handle an arbitrary vector of tickers (e.g., all constituents of an index). Following tidy principles, it is quite easy to download the data, plot the price time series, and tabulate the summary statistics for an arbitrary number of assets.

This is where the `tidyverse` magic starts: tidy data makes it extremely easy to generalize the computations from before to as many assets you like. The following code takes any vector of tickers, e.g., `ticker <- c("AAPL", "MMM", "BA")`, and automates the download as well as the plot of the price time series. In the end, we create the table of summary statistics for an arbitrary number of assets. We perform the analysis with data from all current constituents of the [Dow Jones Industrial Average index](https://en.wikipedia.org/wiki/Dow_Jones_Industrial_Average). 


```r
ticker <- tq_index("DOW") 
index_prices <- tq_get(ticker,
  get = "stock.prices",
  from = "2000-01-01",
  to = "2022-03-30"
) %>%
  filter(symbol != "DOW") # Exclude the index itself
```

The resulting file contains 159099 daily observations for in total 29 different corporations. The figure below illustrates the time series of downloaded *adjusted* prices for each of the constituents of the Dow Jones index. Make sure you understand every single line of code! (What are the arguments of `aes()`? Which alternative geoms could you use to visualize the time series? Hint: if you do not know the answers try to change the code to see what difference your intervention causes). 


```r
index_prices %>%
  ggplot(aes(
    x = date,
    y = adjusted,
    color = symbol
  )) +
  geom_line() +
  labs(
    x = NULL,
    y = NULL,
    color = NULL,
    title = "DOW index stock prices",
    subtitle = "Prices in USD, adjusted for dividend payments and stock splits"
  ) +
  theme(legend.position = "none")
```

<img src="10_introduction_files/figure-html/unnamed-chunk-9-1.png" width="672" style="display: block; margin: auto;" />

Do you notice the small differences relative to the code we used before? `tq_get(ticker)` returns a tibble for several symbols as well. All we need to do to illustrate all tickers simultaneously is to include `color = symbol` in the `ggplot2` aesthetics. In this way, we generate a separate line for each ticker. Of course, there are simply too many lines on this graph to properly identify the individual stocks, but it illustrates the point well.

The same holds for stock returns. Before computing the returns, we use `group_by(symbol)` such that the `mutate` command is performed for each symbol individually. The same logic applies to the computation of summary statistics: `group_by(symbol)` is the key to aggregating the time series into ticker-specific variables of interest. 


```r
all_returns <- index_prices %>%
  group_by(symbol) %>%
  mutate(ret = adjusted / lag(adjusted) - 1) %>%
  select(symbol, date, ret) %>%
  drop_na(ret)

all_returns %>%
  mutate(ret = ret * 100) %>%
  group_by(symbol) %>%
  summarize(across(
    ret,
    list(
      daily_mean = mean,
      daily_sd = sd,
      daily_min = min,
      daily_max = max
    ),
        .names = "{.fn}"
  ))
```

```
## # A tibble: 29 x 5
##   symbol daily_mean daily_sd daily_min daily_max
##   <chr>       <dbl>    <dbl>     <dbl>     <dbl>
## 1 AMGN       0.0484     1.98     -13.4      15.1
## 2 AXP        0.0573     2.30     -17.6      21.9
## 3 BA         0.0603     2.20     -23.8      24.3
## 4 CAT        0.0707     2.03     -14.5      14.7
## 5 CRM        0.124      2.68     -27.1      26.0
## # ... with 24 more rows
```

Note that you are now also equipped with all tools to download price data for *each* ticker listed in the S&P 500 index with the same number of lines of code. Just use `ticker <- tq_index("SP500")`, which provides you with a tibble that contains each symbol that is (currently) part of the S&P 500. However, don't try this if you are not prepared to wait for a couple of minutes because this is quite some data to download!

## Other forms of data aggregation 

Of course, aggregation across other variables than `symbol` can make sense as well. For instance, suppose you are interested in answering the question: are days with high aggregate trading volume likely followed by days with high aggregate trading volume? To provide some initial analysis on this question, we take the downloaded prices and compute aggregate daily trading volume for all Dow Jones constituents in USD. Recall that the column *volume* is denoted in the number of traded shares. Thus, we multiply the trading volume with the daily closing price to get a proxy for the aggregate trading volume in USD. Scaling by `1e9` denotes daily trading volume in billion USD.  


```r
volume <- index_prices %>%
  mutate(volume_usd = volume * close / 1e9) %>%
  group_by(date) %>%
  summarize(volume = sum(volume_usd))

volume %>%
  ggplot(aes(x = date, y = volume)) +
  geom_line() +
  labs(
    x = NULL, y = NULL,
    title = "Aggregate daily trading volume (billion USD)"
  ) 
```

<img src="10_introduction_files/figure-html/unnamed-chunk-11-1.png" width="672" style="display: block; margin: auto;" />

One way to illustrate the persistence of trading volume would be to plot volume on day $t$ against volume on day $t-1$ as in the example below. We add a 45°-line to indicate a hypothetical one-to-one relation by `geom_abline`, addressing potential differences in the axes' scales.


```r
volume %>%
  ggplot(aes(x = lag(volume), y = volume)) +
  geom_point() +
  geom_abline(aes(intercept = 0, slope = 1),
    color = "red",
    linetype = "dotted"
  ) +
  labs(
    x = "Previous day aggregate trading volume (billion USD)",
    y = "Aggregate trading volume (billion USD)",
    title = "Persistence of trading volume"
  )
```

```
## Warning: Removed 1 rows containing missing values (geom_point).
```

<img src="10_introduction_files/figure-html/unnamed-chunk-12-1.png" width="672" style="display: block; margin: auto;" />

Do you understand where the warning `## Warning: Removed 1 rows containing missing values (geom_point).` comes from and what it means? Purely eye-balling reveals that days with high trading volume are often followed by similarly high trading volume days.

## Portfolio choice problems

In the previous part, we show how to download stock market data and inspect it with graphs and summary statistics. Now, we move to a typical question in Finance, namely, how to optimally allocate wealth across different assets. The standard framework for optimal portfolio selection considers investors that prefer higher future returns but dislike future return volatility (defined as the square root of the return variance): the *mean-variance investor*. 

An essential tool to evaluate portfolios in the mean-variance context is the *efficient frontier*, the set of portfolios which satisfy the condition that no other portfolio exists with a higher expected return but with the same volatility (i.e., the risk). We compute and visualize the efficient frontier for several stocks. 
First, we extract each asset's *monthly* returns. In order to keep things simple, we work with a balanced panel and exclude tickers for which we do not observe a price on every single trading day since 2000.


```r
index_prices <- index_prices %>%
  group_by(symbol) %>%
  mutate(n = n()) %>%
  ungroup() %>%
  filter(n == max(n)) %>%
  select(-n)

returns <- index_prices %>%
  mutate(month = floor_date(date, "month")) %>%
  group_by(symbol, month) %>%
  summarize(price = last(adjusted), .groups = "drop_last") %>%
  mutate(ret = price / lag(price) - 1) %>%
  drop_na(ret) %>%
  select(-price)
```

Next, we transform the returns from a tidy tibble into a $(T \times N)$ matrix with one column for each of the $N$ tickers to compute the covariance matrix $\Sigma$ and also the expected return vector $\mu$. We achieve this by using `pivot_wider()` with the new column names from the column `symbol` and setting the values to `ret`.
We compute the vector of sample average returns and the sample variance-covariance matrix, which we consider as proxies for the parameters of future returns. 


```r
returns_matrix <- returns %>%
  pivot_wider(
    names_from = symbol,
    values_from = ret
  ) %>%
  select(-month)

sigma <- cov(returns_matrix)
mu <- colMeans(returns_matrix)
```

Then, we compute the minimum variance portfolio weights $\omega_\text{mvp}$ as well as the expected return $\omega_\text{mvp}'\mu$ and volatility $\sqrt{\omega_\text{mvp}'\Sigma\omega_\text{mvp}}$ of this portfolio. Recall that the minimum variance portfolio is the vector of portfolio weights that are the solution to 
$$\omega_\text{mvp} = \arg\min w'\Sigma w \text{ s.t. } \sum\limits_{i=1}^Nw_i = 1.$$
It is easy to show analytically, that $\omega_\text{mvp} = \frac{\Sigma^{-1}\iota}{\iota'\Sigma^{-1}\iota}$ where $\iota$ is a vector of ones.


```r
N <- ncol(returns_matrix)
iota <- rep(1, N)
mvp_weights <- solve(sigma) %*% iota
mvp_weights <- mvp_weights / sum(mvp_weights)

tibble(expected_ret = t(mvp_weights) %*% mu, volatility = sqrt(t(mvp_weights) %*% sigma %*% mvp_weights))
```

```
## # A tibble: 1 x 2
##   expected_ret[,1] volatility[,1]
##              <dbl>          <dbl>
## 1          0.00839         0.0314
```

Note that the *monthly* volatility of the minimum variance portfolio is of the same order of magnitude as the *daily* standard deviation of the individual components. Thus, the diversification benefits in terms of risk reduction are tremendous!

Next, we set out to find the weights for a portfolio that achieves three times the expected return of the minimum variance portfolio. However, mean-variance investors are not interested in any portfolio that achieves the required return but rather in the efficient portfolio, i.e., the portfolio with the lowest standard deviation. 
If you wonder where the solution $\omega_\text{eff}$ comes from: The efficient portfolio is chosen by an investor who aims to achieve minimum variance *given a minimum acceptable expected return* $\bar{\mu}$. Hence, their objective function is to choose $\omega_\text{eff}$ as the solution to
$$\omega_\text{eff}(\bar{\mu}) = \arg\min w'\Sigma w \text{ s.t. } w'\iota = 1 \text{ and } \omega'\mu \geq \bar{\mu}.$$
The code below implements the analytic solution to this optimization problem for a benchmark return $\bar\mu$ which we set to 3 times the expected return of the minimum variance portfolio. We encourage you to verify that it is correct. 


```r
mu_bar <- 3 * t(mvp_weights) %*% mu

C <- as.numeric(t(iota) %*% solve(sigma) %*% iota)
D <- as.numeric(t(iota) %*% solve(sigma) %*% mu)
E <- as.numeric(t(mu) %*% solve(sigma) %*% mu)

lambda_tilde <- as.numeric(2 * (mu_bar - D / C) / (E - D^2 / C))
efp_weights <- mvp_weights + lambda_tilde / 2 * (solve(sigma) %*% mu - D / C * solve(sigma) %*% iota)
```

## The efficient frontier 

The two mutual fund separation theorem states that as soon as we have two efficient portfolios (such as the minimum variance portfolio and the efficient portfolio for another required level of expected returns like above), we can characterize the entire efficient frontier by combining these two portfolios. The code below implements the construction of the *efficient frontier*, which characterizes the highest expected return achievable at each level of risk. To understand the code better, make sure to familiarize yourself with the inner workings of the `for` loop.


```r
c <- seq(from = -0.4, to = 1.9, by = 0.01)
res <- tibble(
  c = c,
  mu = NA,
  sd = NA
)

for (i in seq_along(c)) {
  w <- (1 - c[i]) * mvp_weights + (c[i]) * efp_weights
  res$mu[i] <- 12 * 100 * t(w) %*% mu
  res$sd[i] <- 12 * sqrt(100) * sqrt(t(w) %*% sigma %*% w)
}
```

Finally, it is simple to visualize the efficient frontier alongside the two efficient portfolios within one, powerful figure using `ggplot2`. We also add the individual stocks in the same call. 

```r
res %>%
  ggplot(aes(x = sd, y = mu)) +
  geom_point() +
  geom_point( # locate the minimum variance and efficient portfolio
    data = res %>% filter(c %in% c(0, 1)),
    color = "red",
    size = 4
  ) +
  geom_point( # locate the individual assets
    data = tibble(mu = 12 * 100 * mu, sd = 12 * 10 * sqrt(diag(sigma))),
    aes(y = mu, x = sd), color = "blue", size = 1
  ) +
  labs(
    x = "Annualized standard deviation (in percent)",
    y = "Annualized expected return (in percent)",
    title = "Dow Jones asset returns and efficient frontier",
    subtitle = "Red dots indicate the location of the minimum variance and efficient tangency portfolio"
  )
```

<img src="10_introduction_files/figure-html/unnamed-chunk-18-1.png" width="672" style="display: block; margin: auto;" />
The black line indicates the efficient frontier: the set of portfolios a mean-variance efficient investor would choose from. Compare the performance relative to the individual assets (the blue dots) - it should become clear that diversifying yields massive performance gains (at least as long as we take the parameters $\Sigma$ and $\mu$ as given).

## Exercises

1. Download daily prices for another stock market ticker of your choice from Yahoo!Finance with `tq_get` from the `tidyquant` package. Plot two time series of the ticker's un-adjusted and adjusted closing prices. Explain the differences.
1. Compute daily net returns for the asset and visualize the distribution of daily returns in a histogram. Also, use `geom_vline()` to add a dashed red line that indicates the 5% quantile of the daily returns within the histogram. Compute summary statistics (mean, standard deviation, minimum and maximum) for the daily returns
1. Take your code from before and generalize it such that you can perform all the computations for an arbitrary vector of tickers (e.g., `ticker <- c("AAPL", "MMM", "BA")`). Automate the download, the plot of the price time series, and create a table of return summary statistics for this arbitrary number of assets.
1. Consider the research question: Are days with high aggregate trading volume often also days with large absolute price changes? Find an appropriate visualization to analyze the question.
1. Compute monthly returns from the downloaded stock market prices. Compute the vector of historical average returns and the sample variance-covariance matrix. After you compute the minimum variance portfolio weights and the portfolio volatility and average returns, visualize the mean-variance efficient frontier. Choose one of your assets and identify the portfolio which yields the same historical volatility but achieves the highest possible average return.
1. In the portfolio choice analysis, we restricted our sample to all assets that were trading on every single day since 2000. How is such a decision a problem when you want to infer future expected portfolio performance from the results?
1. The efficient frontier characterizes the portfolios with the highest expected return for different levels of risk, i.e., standard deviation. Identify the portfolio with the highest expected return per standard deviation. Hint: the ratio of expected return to standard deviation is an important concept in Finance. 
