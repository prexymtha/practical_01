Practical_01: API exercise
================
2026-04-23

# Data

Data is collected using the FRED API. The GINI coefficient - an
indicator of inequality - is obtained for Chile and Argentina from 1987
to 2024. Further, the USD prices of commodities (copper, zinc, and
aluminum) per metric ton are collected.

# Installation, data collection, and tidying

Install (if necessary) and load fredr, pacman, and tidyverse.

``` r
if(!require ( "pacman" , quietly = TRUE ) ) {
   install.packages("pacman")
   library(pacman)
   }
if(!require ( "fredr" , quietly = TRUE ) ) {
  install.packages("fredr")
  library(fredr)
  }
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.2.0     ✔ readr     2.1.6
    ## ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ## ✔ ggplot2   4.0.2     ✔ tibble    3.3.1
    ## ✔ lubridate 1.9.5     ✔ tidyr     1.3.2
    ## ✔ purrr     1.2.1     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

# Data collection

Use the FRED API key and indicator names to source the time series.

``` r
# Set your FRED API key - obtain from https://fredaccount.stlouisfed.org/apikey
fredr_set_key("INSERT OWN API KEY") # I removed your personal API for security reasons , see comments sent via email."

# Fetch Chile and Argentina Gini Coefficients

chile_gini <- fredr(series_id = "SIPOVGINICHL" , observation_start = as.Date("1987-01-01"))

argentina_gini <- fredr(series_id = "SIPOVGINIARG" , observation_start = as.Date("1987-01-01"))

# Fetch commodity prices

copper_price <- fredr(series_id = "PCOPPUSDM" , observation_start = as.Date("1992-01-01"))

aluminium_price <- fredr(series_id = "PALUMUSDM" , observation_start = as.Date("1992-01-01"))

zinc_price <- fredr(series_id = "PZINCUSDM" , observation_start = as.Date("1992-01-01"))
```

# Data cleaning and tidying

Columns meaningfully renamed, the Chile and Argentina dataframes are
merged into an “Inequality” dataframe, and commodity prices are merged
into a “Commodity Price” dataframe. The dataframes are then tidied by
removing unnecessary columns and by using pivot_longer to make new
“country” and “commodity” columns in the respective dataframes.

``` r
# Tidy the data for the first plot

arg_gini <- argentina_gini %>% 
  rename(
    argentina = value
  )

chil_gini <- chile_gini %>% 
  rename(
    chile = value
  )

# Merge Chile and Argentina and select relevant columns
merge01 <- arg_gini %>% 
  left_join(
    chil_gini,
    by = "date"
  ) %>% 
  select(date, argentina, chile)

# Pivot longer
inequality <- merge01 %>% 
  pivot_longer(
    cols = argentina:chile,
     names_to = "country",
     values_to = "gini"
  )

# Tidy the data for Plot 2

# Rename "value" column to commodity
copper_price <- copper_price %>% 
  rename(
    copper_price = value
  )

aluminium_price <- aluminium_price %>% 
  rename(
    aluminium_price = value
  )

zinc_price <- zinc_price %>% 
  rename(
    zinc_price = value
  )

# Merge commodity prices

merge02 <- copper_price %>% 
  left_join(
    aluminium_price,
    by = "date"
  ) %>% 
  select(date, copper_price, aluminium_price)

merge03 <- merge02 %>% 
  left_join(
    zinc_price,
    by = "date"
  ) %>% 
  select(date, copper_price, aluminium_price, zinc_price)

# Pivot longer
commodity_prices <- merge03 %>% 
  pivot_longer(
    cols = copper_price:zinc_price,
    names_to = "commodity",
    values_to = "price"
  )
```

# Plots

ggplot is used to visualise the data from the cleaned and tidied
dataframes.

## Plot 1: Chile and Argentina Gini Coefficients

``` r
# Plot 1: Chile and Argentina Gini Coefficients
ggplot(inequality,
       aes(x = date, y = gini, colour = country)) +
  geom_point() +
  geom_smooth() +
  labs( title = "Chile and Argentina Gini Coeffecients", y = "Gini", x = "Year")
```

    ## `geom_smooth()` using method = 'loess' and formula = 'y ~ x'

    ## Warning: Removed 25 rows containing non-finite outside the scale range
    ## (`stat_smooth()`).

    ## Warning: Removed 25 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

![](README_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

This plot explores the movement of Chile and Argentina’s respective Gini
coefficients since 1987. This annual figure indicates the level of
inequality present in an economy. Chile a much higher level of
inequality compared to Argentina, but over the years the two have
seemingly converged.

## Plot 2: Commodity Prices

``` r
#Plot 2: Commodity Prices
ggplot(commodity_prices,
       aes(x = date, y = price, colour = commodity)) +
  geom_line() +
  labs( title = "Commodity Prices", y = "USD per Metric Ton", x = "Year")
```

![](README_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

The second plot explores the movements in monthly commodity prices since
1992. Until the mid-2000s, copper, zinc, and aluminium were in close
range of each other. Copper has since diverged in price and around three
times the price of aluminium and zinc in the age of the green transition
and growing demand for copper.
