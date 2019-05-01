agricultural\_data
================
owen flanagan
30 April 2019

-   [Introduction](#introduction)
-   [Data loading and tidying](#data-loading-and-tidying)
    -   [Loading](#loading)
    -   [Reshaping](#reshaping)
    -   [Removing non countries](#removing-non-countries)

Introduction
============

This work is a study of some interesting agricultural indicators I came accross while working on some analysis of economic freedom. I believe that explaining techniques, their use, and justifying them is an important part of any data scientists workflow so I will aim to explain my reasoning and justification - practice makes perfect.

The data I will be using in this post were sourced from the World Bank. A full list of indicators can be found here: <https://data.worldbank.org/indicator>

Of these indicators I picked out a subset which I thought would be interesting together,

<https://data.worldbank.org/indicator/AG.LND.IRIG.AG.ZS?view=chart> Agricultural irrigated land % of total agricultural land

<https://data.worldbank.org/indicator/AG.LND.TRAC.ZS?view=chart> Agricultural machienry, tractors per 100 sq. km of arable land more economic freedom, more machines "Quote some of the invisible hand here"

<https://data.worldbank.org/indicator/AG.YLD.CREL.KG?view=chart> Cereal yield (kg per hectare) interesting to look at wrt tractora and irrigation and fertilizer might be interesting to build some models predicting cereal yield as target - glm and rf with feature importance

<https://data.worldbank.org/indicator/AG.CON.FERT.ZS?view=chart> Fertilizer consumption (kilograms per hectare of arable land)

Taken together we have a dataset which contains: irrigation fertilizer mechanisation and resultant cereal yield

This seems like a reasonable application of modelling and I will apply both linear models and tree ensembles to this data set to see what level of predictive accuracy I can get.

(the countries dataset from world bank might include some other interesting data such as wealth, lat and longitude and similar)

Data loading and tidying
========================

The data we are planning on using is provided by the world bank. The world bank does provide an API which we could query, but there is an R package available called wbstats which provides easy access to the data we want. Information on using that wbstats package can be found here:

<https://cran.r-project.org/web/packages/wbstats/vignettes/Using_the_wbstats_package.html>

Loading
-------

``` r
countries <- wb_cachelist$countries %>% as.tbl()
countries <- countries %>% 
  select(-c(capital,iso2c,region_iso2c,regionID,adminID,admin_iso2c,incomeID,income_iso2c,lendingID,lending_iso2c,lending,admin)) %>% 
  mutate(long = as.numeric(long), lat = as.numeric(lat))
countries %>% 
  head() %>% 
  kable()
```

<table>
<thead>
<tr>
<th style="text-align:left;">
iso3c
</th>
<th style="text-align:left;">
country
</th>
<th style="text-align:right;">
long
</th>
<th style="text-align:right;">
lat
</th>
<th style="text-align:left;">
region
</th>
<th style="text-align:left;">
income
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
ABW
</td>
<td style="text-align:left;">
Aruba
</td>
<td style="text-align:right;">
-70.0167
</td>
<td style="text-align:right;">
12.51670
</td>
<td style="text-align:left;">
Latin America & Caribbean
</td>
<td style="text-align:left;">
High income
</td>
</tr>
<tr>
<td style="text-align:left;">
AFG
</td>
<td style="text-align:left;">
Afghanistan
</td>
<td style="text-align:right;">
69.1761
</td>
<td style="text-align:right;">
34.52280
</td>
<td style="text-align:left;">
South Asia
</td>
<td style="text-align:left;">
Low income
</td>
</tr>
<tr>
<td style="text-align:left;">
AFR
</td>
<td style="text-align:left;">
Africa
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:left;">
Aggregates
</td>
<td style="text-align:left;">
Aggregates
</td>
</tr>
<tr>
<td style="text-align:left;">
AGO
</td>
<td style="text-align:left;">
Angola
</td>
<td style="text-align:right;">
13.2420
</td>
<td style="text-align:right;">
-8.81155
</td>
<td style="text-align:left;">
Sub-Saharan Africa
</td>
<td style="text-align:left;">
Lower middle income
</td>
</tr>
<tr>
<td style="text-align:left;">
ALB
</td>
<td style="text-align:left;">
Albania
</td>
<td style="text-align:right;">
19.8172
</td>
<td style="text-align:right;">
41.33170
</td>
<td style="text-align:left;">
Europe & Central Asia
</td>
<td style="text-align:left;">
Upper middle income
</td>
</tr>
<tr>
<td style="text-align:left;">
AND
</td>
<td style="text-align:left;">
Andorra
</td>
<td style="text-align:right;">
1.5218
</td>
<td style="text-align:right;">
42.50750
</td>
<td style="text-align:left;">
Europe & Central Asia
</td>
<td style="text-align:left;">
High income
</td>
</tr>
</tbody>
</table>
``` r
irrigation <- wb(indicator = "AG.LND.IRIG.AG.ZS") %>% #download the specified indicator from the World Bank API
  as.tbl() %>% #convert the dataframe to a tibble for better console printing in interactive analysis (limits default number of rows printed to a reasonable number and prints var type of each column beneath row name)
  mutate(date = as.numeric(date)) #convert the date column to a numeric value

irrigation %>% 
  head()%>% #return the first 6 rows of the irrigation tibble as a new tibble
  kable() #output the new tibble as an html friendly table
```

<table>
<thead>
<tr>
<th style="text-align:left;">
iso3c
</th>
<th style="text-align:right;">
date
</th>
<th style="text-align:right;">
value
</th>
<th style="text-align:left;">
indicatorID
</th>
<th style="text-align:left;">
indicator
</th>
<th style="text-align:left;">
iso2c
</th>
<th style="text-align:left;">
country
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
AFG
</td>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
6.481140
</td>
<td style="text-align:left;">
AG.LND.IRIG.AG.ZS
</td>
<td style="text-align:left;">
Agricultural irrigated land (% of total agricultural land)
</td>
<td style="text-align:left;">
AF
</td>
<td style="text-align:left;">
Afghanistan
</td>
</tr>
<tr>
<td style="text-align:left;">
AFG
</td>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
5.710894
</td>
<td style="text-align:left;">
AG.LND.IRIG.AG.ZS
</td>
<td style="text-align:left;">
Agricultural irrigated land (% of total agricultural land)
</td>
<td style="text-align:left;">
AF
</td>
<td style="text-align:left;">
Afghanistan
</td>
</tr>
<tr>
<td style="text-align:left;">
AFG
</td>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
5.742548
</td>
<td style="text-align:left;">
AG.LND.IRIG.AG.ZS
</td>
<td style="text-align:left;">
Agricultural irrigated land (% of total agricultural land)
</td>
<td style="text-align:left;">
AF
</td>
<td style="text-align:left;">
Afghanistan
</td>
</tr>
<tr>
<td style="text-align:left;">
AFG
</td>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
5.518333
</td>
<td style="text-align:left;">
AG.LND.IRIG.AG.ZS
</td>
<td style="text-align:left;">
Agricultural irrigated land (% of total agricultural land)
</td>
<td style="text-align:left;">
AF
</td>
<td style="text-align:left;">
Afghanistan
</td>
</tr>
<tr>
<td style="text-align:left;">
AFG
</td>
<td style="text-align:right;">
2012
</td>
<td style="text-align:right;">
5.465576
</td>
<td style="text-align:left;">
AG.LND.IRIG.AG.ZS
</td>
<td style="text-align:left;">
Agricultural irrigated land (% of total agricultural land)
</td>
<td style="text-align:left;">
AF
</td>
<td style="text-align:left;">
Afghanistan
</td>
</tr>
<tr>
<td style="text-align:left;">
AFG
</td>
<td style="text-align:right;">
2011
</td>
<td style="text-align:right;">
5.391717
</td>
<td style="text-align:left;">
AG.LND.IRIG.AG.ZS
</td>
<td style="text-align:left;">
Agricultural irrigated land (% of total agricultural land)
</td>
<td style="text-align:left;">
AF
</td>
<td style="text-align:left;">
Afghanistan
</td>
</tr>
</tbody>
</table>
``` r
tractors <- wb(indicator = "AG.LND.TRAC.ZS") %>% 
  as.tbl() %>% 
  mutate(date = as.numeric(date))
tractors %>% head() %>% kable()
```

<table>
<thead>
<tr>
<th style="text-align:left;">
iso3c
</th>
<th style="text-align:right;">
date
</th>
<th style="text-align:right;">
value
</th>
<th style="text-align:left;">
indicatorID
</th>
<th style="text-align:left;">
indicator
</th>
<th style="text-align:left;">
iso2c
</th>
<th style="text-align:left;">
country
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
2000
</td>
<td style="text-align:right;">
153.9730
</td>
<td style="text-align:left;">
AG.LND.TRAC.ZS
</td>
<td style="text-align:left;">
Agricultural machinery, tractors per 100 sq. km of arable land
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
1999
</td>
<td style="text-align:right;">
126.8138
</td>
<td style="text-align:left;">
AG.LND.TRAC.ZS
</td>
<td style="text-align:left;">
Agricultural machinery, tractors per 100 sq. km of arable land
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
1998
</td>
<td style="text-align:right;">
115.2138
</td>
<td style="text-align:left;">
AG.LND.TRAC.ZS
</td>
<td style="text-align:left;">
Agricultural machinery, tractors per 100 sq. km of arable land
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
1997
</td>
<td style="text-align:right;">
113.5813
</td>
<td style="text-align:left;">
AG.LND.TRAC.ZS
</td>
<td style="text-align:left;">
Agricultural machinery, tractors per 100 sq. km of arable land
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
1996
</td>
<td style="text-align:right;">
111.9251
</td>
<td style="text-align:left;">
AG.LND.TRAC.ZS
</td>
<td style="text-align:left;">
Agricultural machinery, tractors per 100 sq. km of arable land
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
1995
</td>
<td style="text-align:right;">
112.2665
</td>
<td style="text-align:left;">
AG.LND.TRAC.ZS
</td>
<td style="text-align:left;">
Agricultural machinery, tractors per 100 sq. km of arable land
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
</tbody>
</table>
``` r
fertilizer <- wb(indicator = "AG.CON.FERT.ZS") %>% 
  as.tbl()%>% 
  mutate(date = as.numeric(date))
fertilizer  %>% head() %>% kable()
```

<table>
<thead>
<tr>
<th style="text-align:left;">
iso3c
</th>
<th style="text-align:right;">
date
</th>
<th style="text-align:right;">
value
</th>
<th style="text-align:left;">
indicatorID
</th>
<th style="text-align:left;">
indicator
</th>
<th style="text-align:left;">
iso2c
</th>
<th style="text-align:left;">
country
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
68.35913
</td>
<td style="text-align:left;">
AG.CON.FERT.ZS
</td>
<td style="text-align:left;">
Fertilizer consumption (kilograms per hectare of arable land)
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
73.25786
</td>
<td style="text-align:left;">
AG.CON.FERT.ZS
</td>
<td style="text-align:left;">
Fertilizer consumption (kilograms per hectare of arable land)
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
68.16071
</td>
<td style="text-align:left;">
AG.CON.FERT.ZS
</td>
<td style="text-align:left;">
Fertilizer consumption (kilograms per hectare of arable land)
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
62.39705
</td>
<td style="text-align:left;">
AG.CON.FERT.ZS
</td>
<td style="text-align:left;">
Fertilizer consumption (kilograms per hectare of arable land)
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
2012
</td>
<td style="text-align:right;">
64.09569
</td>
<td style="text-align:left;">
AG.CON.FERT.ZS
</td>
<td style="text-align:left;">
Fertilizer consumption (kilograms per hectare of arable land)
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
2011
</td>
<td style="text-align:right;">
104.89946
</td>
<td style="text-align:left;">
AG.CON.FERT.ZS
</td>
<td style="text-align:left;">
Fertilizer consumption (kilograms per hectare of arable land)
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
</tbody>
</table>
``` r
cereal_yield <- wb(indicator = "AG.YLD.CREL.KG") %>% 
  as.tbl() %>% 
  mutate(date = as.numeric(date)) 
cereal_yield %>% head() %>% kable()
```

<table>
<thead>
<tr>
<th style="text-align:left;">
iso3c
</th>
<th style="text-align:right;">
date
</th>
<th style="text-align:right;">
value
</th>
<th style="text-align:left;">
indicatorID
</th>
<th style="text-align:left;">
indicator
</th>
<th style="text-align:left;">
iso2c
</th>
<th style="text-align:left;">
country
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
2017
</td>
<td style="text-align:right;">
2023.717
</td>
<td style="text-align:left;">
AG.YLD.CREL.KG
</td>
<td style="text-align:left;">
Cereal yield (kg per hectare)
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
1744.945
</td>
<td style="text-align:left;">
AG.YLD.CREL.KG
</td>
<td style="text-align:left;">
Cereal yield (kg per hectare)
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
2119.373
</td>
<td style="text-align:left;">
AG.YLD.CREL.KG
</td>
<td style="text-align:left;">
Cereal yield (kg per hectare)
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
1795.737
</td>
<td style="text-align:left;">
AG.YLD.CREL.KG
</td>
<td style="text-align:left;">
Cereal yield (kg per hectare)
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
1966.458
</td>
<td style="text-align:left;">
AG.YLD.CREL.KG
</td>
<td style="text-align:left;">
Cereal yield (kg per hectare)
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:right;">
2012
</td>
<td style="text-align:right;">
2031.372
</td>
<td style="text-align:left;">
AG.YLD.CREL.KG
</td>
<td style="text-align:left;">
Cereal yield (kg per hectare)
</td>
<td style="text-align:left;">
1A
</td>
<td style="text-align:left;">
Arab World
</td>
</tr>
</tbody>
</table>
Some notes on what we can see so far:

-   We essentially have a number of data tables with the format \[Country,Year,Indicator,Value\] with some of these repeated in different formats. Needs reshaping.
-   We don't really have a very good name column at the moment. Need to make.
-   The country column for several of these datasets has a value "Arab World". As far as I am aware, that's not a country. Needs cleaning.

Reshaping
---------

``` r
irrigation_red <- irrigation %>% 
  select(iso3c, country, date, value) %>% 
  rename(irrigation_percent_of_land = value)

irrigation_red
```

    ## # A tibble: 856 x 4
    ##    iso3c country      date irrigation_percent_of_land
    ##    <chr> <chr>       <dbl>                      <dbl>
    ##  1 AFG   Afghanistan  2016                       6.48
    ##  2 AFG   Afghanistan  2015                       5.71
    ##  3 AFG   Afghanistan  2014                       5.74
    ##  4 AFG   Afghanistan  2013                       5.52
    ##  5 AFG   Afghanistan  2012                       5.47
    ##  6 AFG   Afghanistan  2011                       5.39
    ##  7 AFG   Afghanistan  2010                       5.00
    ##  8 AFG   Afghanistan  2009                       4.84
    ##  9 AFG   Afghanistan  2008                       5.78
    ## 10 AFG   Afghanistan  2007                       5.94
    ## # ... with 846 more rows

``` r
tractors_red <- tractors %>% 
  select(iso3c, country, date, value) %>% 
  rename(tractors_per_100sqkm = value)

tractors_red
```

    ## # A tibble: 7,444 x 4
    ##    iso3c country     date tractors_per_100sqkm
    ##    <chr> <chr>      <dbl>                <dbl>
    ##  1 ARB   Arab World  2000                154. 
    ##  2 ARB   Arab World  1999                127. 
    ##  3 ARB   Arab World  1998                115. 
    ##  4 ARB   Arab World  1997                114. 
    ##  5 ARB   Arab World  1996                112. 
    ##  6 ARB   Arab World  1995                112. 
    ##  7 ARB   Arab World  1994                103. 
    ##  8 ARB   Arab World  1993                101. 
    ##  9 ARB   Arab World  1992                 96.2
    ## 10 ARB   Arab World  1991                 97.4
    ## # ... with 7,434 more rows

``` r
fertilizer_red <- fertilizer %>% 
  select(iso3c, country, date, value) %>% 
  rename(fertilizer_kg_per_hectare = value)
fertilizer_red
```

    ## # A tibble: 2,985 x 4
    ##    iso3c country     date fertilizer_kg_per_hectare
    ##    <chr> <chr>      <dbl>                     <dbl>
    ##  1 ARB   Arab World  2016                      68.4
    ##  2 ARB   Arab World  2015                      73.3
    ##  3 ARB   Arab World  2014                      68.2
    ##  4 ARB   Arab World  2013                      62.4
    ##  5 ARB   Arab World  2012                      64.1
    ##  6 ARB   Arab World  2011                     105. 
    ##  7 ARB   Arab World  2010                      92.2
    ##  8 ARB   Arab World  2009                      82.5
    ##  9 ARB   Arab World  2008                     101. 
    ## 10 ARB   Arab World  2007                      92.0
    ## # ... with 2,975 more rows

``` r
cereal_yield_red <- cereal_yield %>% 
  select(iso3c, country, date, value) %>% 
  rename(cereal_yield_kg_per_heactare = value)
cereal_yield_red
```

    ## # A tibble: 11,781 x 4
    ##    iso3c country     date cereal_yield_kg_per_heactare
    ##    <chr> <chr>      <dbl>                        <dbl>
    ##  1 ARB   Arab World  2017                        2024.
    ##  2 ARB   Arab World  2016                        1745.
    ##  3 ARB   Arab World  2015                        2119.
    ##  4 ARB   Arab World  2014                        1796.
    ##  5 ARB   Arab World  2013                        1966.
    ##  6 ARB   Arab World  2012                        2031.
    ##  7 ARB   Arab World  2011                        2523.
    ##  8 ARB   Arab World  2010                        2241.
    ##  9 ARB   Arab World  2009                        2615.
    ## 10 ARB   Arab World  2008                        2296.
    ## # ... with 11,771 more rows

Removing non countries
----------------------

``` r
countries %>% left_join(irrigation_red %>% select(-country),by="iso3c") %>% 
  left_join(tractors_red %>% select(-country),by=c("iso3c","date")) %>% 
  left_join(fertilizer_red %>% select(-country),by=c("iso3c","date")) %>% 
  left_join(cereal_yield_red %>% select(-country),by=c("iso3c","date")) %>% head() %>% kable()
```

<table>
<thead>
<tr>
<th style="text-align:left;">
iso3c
</th>
<th style="text-align:left;">
country
</th>
<th style="text-align:right;">
long
</th>
<th style="text-align:right;">
lat
</th>
<th style="text-align:left;">
region
</th>
<th style="text-align:left;">
income
</th>
<th style="text-align:right;">
date
</th>
<th style="text-align:right;">
irrigation\_percent\_of\_land
</th>
<th style="text-align:right;">
tractors\_per\_100sqkm
</th>
<th style="text-align:right;">
fertilizer\_kg\_per\_hectare
</th>
<th style="text-align:right;">
cereal\_yield\_kg\_per\_heactare
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
ABW
</td>
<td style="text-align:left;">
Aruba
</td>
<td style="text-align:right;">
-70.0167
</td>
<td style="text-align:right;">
12.5167
</td>
<td style="text-align:left;">
Latin America & Caribbean
</td>
<td style="text-align:left;">
High income
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
AFG
</td>
<td style="text-align:left;">
Afghanistan
</td>
<td style="text-align:right;">
69.1761
</td>
<td style="text-align:right;">
34.5228
</td>
<td style="text-align:left;">
South Asia
</td>
<td style="text-align:left;">
Low income
</td>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
6.481140
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
12.18230
</td>
<td style="text-align:right;">
1981.6
</td>
</tr>
<tr>
<td style="text-align:left;">
AFG
</td>
<td style="text-align:left;">
Afghanistan
</td>
<td style="text-align:right;">
69.1761
</td>
<td style="text-align:right;">
34.5228
</td>
<td style="text-align:left;">
South Asia
</td>
<td style="text-align:left;">
Low income
</td>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
5.710894
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
12.12582
</td>
<td style="text-align:right;">
2133.0
</td>
</tr>
<tr>
<td style="text-align:left;">
AFG
</td>
<td style="text-align:left;">
Afghanistan
</td>
<td style="text-align:right;">
69.1761
</td>
<td style="text-align:right;">
34.5228
</td>
<td style="text-align:left;">
South Asia
</td>
<td style="text-align:left;">
Low income
</td>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
5.742548
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
12.11646
</td>
<td style="text-align:right;">
2017.5
</td>
</tr>
<tr>
<td style="text-align:left;">
AFG
</td>
<td style="text-align:left;">
Afghanistan
</td>
<td style="text-align:right;">
69.1761
</td>
<td style="text-align:right;">
34.5228
</td>
<td style="text-align:left;">
South Asia
</td>
<td style="text-align:left;">
Low income
</td>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
5.518333
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
14.87930
</td>
<td style="text-align:right;">
2048.5
</td>
</tr>
<tr>
<td style="text-align:left;">
AFG
</td>
<td style="text-align:left;">
Afghanistan
</td>
<td style="text-align:right;">
69.1761
</td>
<td style="text-align:right;">
34.5228
</td>
<td style="text-align:left;">
South Asia
</td>
<td style="text-align:left;">
Low income
</td>
<td style="text-align:right;">
2012
</td>
<td style="text-align:right;">
5.465576
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
28.06565
</td>
<td style="text-align:right;">
2029.6
</td>
</tr>
</tbody>
</table>
