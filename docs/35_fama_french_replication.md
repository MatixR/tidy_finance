# Bivariate Sorts: Replication of Fama & French Factors
output: html_document
---



The Fama and French three-factor model (see CITATION) is a cornerstone of asset pricing. On top of the market factor represented by the traditional CAPM beta, the model includes the factors size and value. We did introduce both factors in the previous section, and their definition remains the same. Size is the SMB factor (small-minus-big) that is long small firms and short large firms. The value factor is HML (high-minus-low) and is long in high book-to-market firms and short the low book-to-market counterparts. While we demonstrated the main idea of portfolio sorts, we also want to show how to replicate these significant factors. However, we do not aim at a perfect replication but want to show the main ideas.

The current section relies on this set of packages. 

```r
library(tidyverse)
library(RSQLite)
library(lubridate)
library(sandwich)
library(lmtest)
```

## Databases

We use the same data sources CRSP and compustat, as we need exactly the same variables to compute the size and value factors in the way Fama and French do it. Hence, there is nothing new below.


```r
# Load data from database
tidy_finance <- dbConnect(SQLite(), "data/tidy_finance.sqlite", extended_types = TRUE)

crsp_monthly <- tbl(tidy_finance, "crsp_monthly") %>% 
  collect()

factors_ff_monthly <- tbl(tidy_finance, "factors_ff_monthly") %>% 
  collect()

compustat <- tbl(tidy_finance, "compustat") %>% 
  collect()

# Keep only relevant data
data_ff <- crsp_monthly %>% 
  left_join(factors_ff_monthly, by = "month") %>% 
  select(permno, gvkey, month, ret_excess, mkt_excess, mktcap, mktcap_lag, exchange) %>% 
  drop_na()

# Compustat data
be <- compustat %>%
  select(gvkey, datadate, be) %>% 
  drop_na()
```

## Data Preparation

Yet when we start merging our data set for computing the premiums, there are a few differences to the previous sections. First, Fama and French form their portfolios in June of year $t$, whereby the returns of July are the first monthly return for the respective portfolio. For firm size, they consequently use the market capitalization recorded for June. It is then held constant until June of year $t+1$.

Second, Fama and French also have a different protocol for computing the book-to-market ratio. They use market equity as of the end of year $t - 1$ and the book equity reported in year $t-1$, i.e., the `datadate` is within the last year. Hence, the book-to-market ratio can be based on accounting information that is up to 18 months old. Market equity also does not necessarily reflect the same time point as book equity.

To implement all these time lags, we again employ the temporary `sorting_date`-column. Notice that when we combine the information, we want to have a single observation per year and stock since we are only interested in computing the breakpoints held constant for the entire year. We ensure this by a call of `distinct()` at the end of the chunk below.


```r
# Market equity
ff_me <- data_ff %>%
  filter(month(month) == 6) %>%
  mutate(sorting_date = month %m+% months(1)) %>%
  select(permno, sorting_date, ff_me = mktcap)

# Book-to-market
## December market equity
ff_me_dec <- data_ff %>%
  filter(month(month) == 12) %>%
  mutate(sorting_date = ymd(paste0(year(month) + 1, "07-01)"))) %>%
  select(permno, gvkey, sorting_date, bm_me = mktcap)

## Book to market
ff_bm <- be %>%
  # be from year before
  mutate(sorting_date = ymd(paste0(year(datadate) + 1, "07-01"))) %>%
  select(gvkey, sorting_date, bm_be = be) %>%
  drop_na() %>%
  inner_join(ff_me_dec, by = c("gvkey", "sorting_date")) %>%
  mutate(ff_bm = bm_be/bm_me) %>%
  select(permno, sorting_date, ff_bm)

# Combine factors
ff_variables <- ff_me %>% 
  inner_join(ff_bm, by = c("permno", "sorting_date")) %>%
  drop_na() %>%
  distinct(permno, sorting_date, .keep_all = TRUE)
```

## Breakpoints

Before we attach the sorting variables to our returns, we compute the breakpoints separately. Matching everything before computing the breakpoints would be possible but less efficient. Regarding the other choice variables, Fama and French rely on NYSE-specific breakpoints, they form two portfolios in the size dimension at the median and three portfolios in the dimension of book-to-market at the 30%- and 70%-percentiles, and they use independent sorts. Additionally, we perform an `inner_join()`  with our return data to ensure that we only use traded stocks when computing the breakpoints as a first step. 


```r
ff_breakpoints <- ff_variables %>%
  inner_join(data_ff, by = c("permno" = "permno", "sorting_date" = "month")) %>%
  filter(exchange == "NYSE") %>%
  group_by(sorting_date) %>%
  summarise(ff_me_50 = median(ff_me),
            ff_bm_30 = quantile(ff_bm, 0.3, names = FALSE),
            ff_bm_70 = quantile(ff_bm, 0.7, names = FALSE))
```

Next, we merge the June breakpoints to the return data. To implement this step, we create a new column `sorting_date` in our return data by setting the date to sort on to July of $t-1$ if the month is June (of year $t$) or earlier or to July of year $t$ if the month is July or later. Doing so ensures that we form portfolios based on the information assigned to June of year $t$. In principle, we will still form portfolios on a monthly basis, but this is only incorrect to a few stocks that drop out during the year.


```r
ff_portfolios <- data_ff %>%
  mutate(sorting_date = case_when(month(month) <= 6 ~ ymd(paste0(year(month) - 1, "0701")),
                                  month(month) >= 7 ~ ymd(paste0(year(month), "0701")))) %>%
  inner_join(ff_variables, by = c("permno", "sorting_date")) %>%
  left_join(ff_breakpoints, by = "sorting_date")
```

## Fama and French Factor Returns

We can now use our returns alongside the breakpoints to assign the portfolios. We do so slightly differently by using the `case_when()`-function. We assign the portfolios by adding numbers that result in the respective portfolio. The base portfolio of small growth stocks gets the value of 11. We add 10 for large firms and 1 and 2 for higher book-to-market ratios. In the end, we add the letter p to indicate that these are portfolio numbers. Overall, this procedure is potentially less intuitive but suitable for this exercise given the low number of portfolios. 

After forming the monthly value-weighted return per portfolio, we use the function `pivot_wider()`. This function is useful in various scenarios. In our case, we use it to transform the data containing portfolio-months in each row into a data frame with one row for each month and columns that indicate the portfolio returns. The argument `id_cols` defines how to identify the new observations uniquely, `names_from` indicates which column of the old structure defines the new set's column names, and `values_from` gives the column that contains the data to be stored in the created columns. 

Finally, we can then compute the equal-weighted monthly returns of the size and value factors. The size premium results from going long the three small portfolios (i.e., portfolios 11, 12, and 13) and shorting large portfolios (i.e., portfolios 21, 22, and 23). The value premium is achieved by investing in the two high book-to-market portfolios (i.e., portfolios 13 and 23) and shorting the growth portfolios (i.e., portfolios 11 and 21).


```r
ff_factors <- ff_portfolios %>%
  mutate(portfolio = 10,
         portfolio = case_when(ff_me < ff_me_50 ~ portfolio,
                               ff_me >= ff_me_50 ~ portfolio + 10),
         portfolio = case_when(ff_bm < ff_bm_30 ~ portfolio + 1,
                               ff_bm >= ff_bm_30 & ff_bm < ff_bm_70 ~ portfolio + 2,
                               ff_bm >= ff_bm_70 ~ portfolio + 3),
         portfolio = paste0("p", as.character(portfolio))) %>%
  group_by(portfolio, month) %>%
  summarise(ret = weighted.mean(ret_excess, mktcap_lag), .groups = "drop") %>%
  pivot_wider(id_cols = month, names_from = portfolio, values_from = ret) %>%
  mutate(ff_SMB = (p11 + p12 + p13) / 3 - (p21 + p22 + p23) / 3,
         ff_HML = (p13 + p23) / 2 - (p11 + p21) / 2) %>%
  select(month, ff_SMB, ff_HML)
```

## Correlation Tests

In the last step, we replicated the size and value premiums following the procedure outlined by Fama and French. However, we did not follow their procedure strictly. The final question is then; how close did we get? We answer this question by looking at the two time-series estimates in a correlation analysis using the function `cor.test()` and a test of means using the function `t.test()`.

We can see that we reject the null hypothesis for both premiums in the correlation tests, i.e., the hypothesis that the time series are uncorrelated is rejected. Moreover, we cannot reject the hypothesis that the means of the Fama and French factors and our replication results are different. Hence, we can conclude that we did a relatively good job in replicating their premiums.


```r
test <- factors_ff_monthly %>% 
  inner_join(ff_factors, by = "month") %>%
  mutate(ff_SMB = round(ff_SMB, 4),
         ff_HML = round(ff_HML, 4))

cor.test(test$ff_SMB, test$smb) 
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  test$ff_SMB and test$smb
## t = 216.1, df = 712, p-value < 2.2e-16
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  0.9912746 0.9934900
## sample estimates:
##      cor 
## 0.992463
```

```r
cor.test(test$ff_HML, test$hml)
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  test$ff_HML and test$hml
## t = 129.46, df = 712, p-value < 2.2e-16
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  0.9761907 0.9822018
## sample estimates:
##       cor 
## 0.9794122
```

```r
t.test(test$ff_SMB, test$smb) 
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  test$ff_SMB and test$smb
## t = 0.084818, df = 1426, p-value = 0.9324
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -0.002996805  0.003267673
## sample estimates:
##   mean of x   mean of y 
## 0.001945378 0.001809944
```

```r
t.test(test$ff_HML, test$hml)
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  test$ff_HML and test$hml
## t = -0.11467, df = 1425.4, p-value = 0.9087
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -0.003147035  0.002799416
## sample estimates:
##   mean of x   mean of y 
## 0.002485714 0.002659524
```