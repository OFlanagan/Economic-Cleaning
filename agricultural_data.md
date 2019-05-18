agricultural\_data
================
owen flanagan
30 April 2019

-   [Introduction](#introduction)
-   [Data loading and tidying](#data-loading-and-tidying)
    -   [Loading](#loading)
    -   [Reshaping](#reshaping)
    -   [Removing non countries](#removing-non-countries)
    -   [Returning to the histograms](#returning-to-the-histograms)

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
  rename(IrrigationPercentOfLand = value)

irrigation_red %>% head() %>% kable()
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
date
</th>
<th style="text-align:right;">
IrrigationPercentOfLand
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
AFG
</td>
<td style="text-align:left;">
Afghanistan
</td>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
6.481140
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
2015
</td>
<td style="text-align:right;">
5.710894
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
2014
</td>
<td style="text-align:right;">
5.742548
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
2013
</td>
<td style="text-align:right;">
5.518333
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
2012
</td>
<td style="text-align:right;">
5.465576
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
2011
</td>
<td style="text-align:right;">
5.391717
</td>
</tr>
</tbody>
</table>
``` r
tractors_red <- tractors %>% 
  select(iso3c, country, date, value) %>% 
  rename(TractorsPer100Sqkm = value)

tractors_red %>% head() %>% kable()
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
date
</th>
<th style="text-align:right;">
TractorsPer100Sqkm
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
2000
</td>
<td style="text-align:right;">
153.9730
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
1999
</td>
<td style="text-align:right;">
126.8138
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
1998
</td>
<td style="text-align:right;">
115.2138
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
1997
</td>
<td style="text-align:right;">
113.5813
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
1996
</td>
<td style="text-align:right;">
111.9251
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
1995
</td>
<td style="text-align:right;">
112.2665
</td>
</tr>
</tbody>
</table>
``` r
fertilizer_red <- fertilizer %>% 
  select(iso3c, country, date, value) %>% 
  rename(FertilizerKgPerHectare = value)
fertilizer_red %>% head() %>% kable()
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
date
</th>
<th style="text-align:right;">
FertilizerKgPerHectare
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
68.35913
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
73.25786
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
68.16071
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
62.39705
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
2012
</td>
<td style="text-align:right;">
64.09569
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
2011
</td>
<td style="text-align:right;">
104.89946
</td>
</tr>
</tbody>
</table>
``` r
cereal_yield_red <- cereal_yield %>% 
  select(iso3c, country, date, value) %>% 
  rename(CerealYieldKgPerHectare = value)
cereal_yield_red %>% head() %>% kable()
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
date
</th>
<th style="text-align:right;">
CerealYieldKgPerHectare
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
2017
</td>
<td style="text-align:right;">
2023.717
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
1744.945
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
2119.373
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
1795.737
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
1966.458
</td>
</tr>
<tr>
<td style="text-align:left;">
ARB
</td>
<td style="text-align:left;">
Arab World
</td>
<td style="text-align:right;">
2012
</td>
<td style="text-align:right;">
2031.372
</td>
</tr>
</tbody>
</table>
Removing non countries
----------------------

The easiest way to remove the non-countries from the data set is to join onto the countries dataframe.

``` r
df <- countries %>%
    left_join(irrigation_red %>% select(-country),by="iso3c") %>% 
    left_join(tractors_red %>% select(-country),by=c("iso3c","date")) %>% 
    left_join(fertilizer_red %>% select(-country),by=c("iso3c","date")) %>% 
    left_join(cereal_yield_red %>% select(-country),by=c("iso3c","date"))
```

``` r
df %>% head() %>% kable()
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
IrrigationPercentOfLand
</th>
<th style="text-align:right;">
TractorsPer100Sqkm
</th>
<th style="text-align:right;">
FertilizerKgPerHectare
</th>
<th style="text-align:right;">
CerealYieldKgPerHectare
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
``` r
df %>% summary()
```

    ##     iso3c             country               long               lat        
    ##  Length:1040        Length:1040        Min.   :-175.216   Min.   :-41.29  
    ##  Class :character   Class :character   1st Qu.:   7.434   1st Qu.: 13.70  
    ##  Mode  :character   Mode  :character   Median :  26.098   Median : 34.52  
    ##                                        Mean   :  25.569   Mean   : 26.66  
    ##                                        3rd Qu.:  51.445   3rd Qu.: 44.45  
    ##                                        Max.   : 179.090   Max.   : 64.18  
    ##                                        NA's   :117        NA's   :117     
    ##     region             income               date     
    ##  Length:1040        Length:1040        Min.   :2001  
    ##  Class :character   Class :character   1st Qu.:2005  
    ##  Mode  :character   Mode  :character   Median :2008  
    ##                                        Mean   :2008  
    ##                                        3rd Qu.:2012  
    ##                                        Max.   :2016  
    ##                                        NA's   :184   
    ##  IrrigationPercentOfLand TractorsPer100Sqkm FertilizerKgPerHectare
    ##  Min.   : 0.000          Min.   :   1.864   Min.   :   0.3081     
    ##  1st Qu.: 1.232          1st Qu.: 106.260   1st Qu.:  37.5067     
    ##  Median : 5.266          Median : 163.511   Median : 100.2493     
    ##  Mean   :10.076          Mean   : 534.825   Mean   : 154.9184     
    ##  3rd Qu.:13.376          3rd Qu.: 541.363   3rd Qu.: 180.8365     
    ##  Max.   :59.711          Max.   :6438.452   Max.   :2304.6079     
    ##  NA's   :184             NA's   :874        NA's   :272           
    ##  CerealYieldKgPerHectare
    ##  Min.   :  176.3        
    ##  1st Qu.: 1999.1        
    ##  Median : 2944.8        
    ##  Mean   : 3400.2        
    ##  3rd Qu.: 4333.9        
    ##  Max.   :28130.1        
    ##  NA's   :209

We can see we have many NAs in our dataset. It seems that a lot of rows are missing longitude and lattitude data, and more are missing date. My guess would be that a lot of rows are missing values in a lot of columns and these are for countries like Aruba. We should start by looking at the columns where the data is missing from the fewest rows. This will allow us to potentially reduce the Na's from most rows.

``` r
df %>% filter(is.na(long)) %>% head() %>%  kable()
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
IrrigationPercentOfLand
</th>
<th style="text-align:right;">
TractorsPer100Sqkm
</th>
<th style="text-align:right;">
FertilizerKgPerHectare
</th>
<th style="text-align:right;">
CerealYieldKgPerHectare
</th>
</tr>
</thead>
<tbody>
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
ANR
</td>
<td style="text-align:left;">
Andean Region
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
ARB
</td>
<td style="text-align:left;">
Arab World
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
BEA
</td>
<td style="text-align:left;">
East Asia & Pacific (IBRD-only countries)
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
BEC
</td>
<td style="text-align:left;">
Europe & Central Asia (IBRD-only countries)
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
BHI
</td>
<td style="text-align:left;">
IBRD countries classified as high income
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
</tbody>
</table>
When we do this we see that these are indeed rows for non-countries - we don't care about them.

``` r
df <- df %>% filter(!is.na(long))
df %>% summary()
```

    ##     iso3c             country               long               lat        
    ##  Length:923         Length:923         Min.   :-175.216   Min.   :-41.29  
    ##  Class :character   Class :character   1st Qu.:   7.434   1st Qu.: 13.70  
    ##  Mode  :character   Mode  :character   Median :  26.098   Median : 34.52  
    ##                                        Mean   :  25.569   Mean   : 26.66  
    ##                                        3rd Qu.:  51.445   3rd Qu.: 44.45  
    ##                                        Max.   : 179.090   Max.   : 64.18  
    ##                                                                           
    ##     region             income               date     
    ##  Length:923         Length:923         Min.   :2001  
    ##  Class :character   Class :character   1st Qu.:2005  
    ##  Mode  :character   Mode  :character   Median :2008  
    ##                                        Mean   :2008  
    ##                                        3rd Qu.:2012  
    ##                                        Max.   :2016  
    ##                                        NA's   :93    
    ##  IrrigationPercentOfLand TractorsPer100Sqkm FertilizerKgPerHectare
    ##  Min.   : 0.000          Min.   :   1.864   Min.   :   0.3081     
    ##  1st Qu.: 1.199          1st Qu.: 106.260   1st Qu.:  37.5067     
    ##  Median : 5.372          Median : 163.511   Median : 100.2493     
    ##  Mean   :10.229          Mean   : 534.825   Mean   : 154.9184     
    ##  3rd Qu.:13.621          3rd Qu.: 541.363   3rd Qu.: 180.8365     
    ##  Max.   :59.711          Max.   :6438.452   Max.   :2304.6079     
    ##  NA's   :93              NA's   :757        NA's   :155           
    ##  CerealYieldKgPerHectare
    ##  Min.   :  176.3        
    ##  1st Qu.: 2017.0        
    ##  Median : 2969.9        
    ##  Mean   : 3423.3        
    ##  3rd Qu.: 4351.7        
    ##  Max.   :28130.1        
    ##  NA's   :104

Working off the same strategy of investigating the columns with the least rows missing we can look at date.

``` r
df %>% filter(is.na(date)) %>% head() %>%  kable()
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
IrrigationPercentOfLand
</th>
<th style="text-align:right;">
TractorsPer100Sqkm
</th>
<th style="text-align:right;">
FertilizerKgPerHectare
</th>
<th style="text-align:right;">
CerealYieldKgPerHectare
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
ASM
</td>
<td style="text-align:left;">
American Samoa
</td>
<td style="text-align:right;">
-170.6910
</td>
<td style="text-align:right;">
-14.28460
</td>
<td style="text-align:left;">
East Asia & Pacific
</td>
<td style="text-align:left;">
Upper middle income
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
ATG
</td>
<td style="text-align:left;">
Antigua and Barbuda
</td>
<td style="text-align:right;">
-61.8456
</td>
<td style="text-align:right;">
17.11750
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
BDI
</td>
<td style="text-align:left;">
Burundi
</td>
<td style="text-align:right;">
29.3639
</td>
<td style="text-align:right;">
-3.37840
</td>
<td style="text-align:left;">
Sub-Saharan Africa
</td>
<td style="text-align:left;">
Low income
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
</tbody>
</table>
These rows are for countries that we may actually be interested in. It may be that there is simply not data available for these coutnries, or it may be something else. This deserves investigation.

``` r
countries_with_one_row <- irrigation_red %>% 
  #group dataframe by country name
  group_by(country) %>% 
  #count the number of rows - this is the number of years of data
  summarise(n=n()) %>% 
  #filter to only have countries with a single row
  filter(n == 1)

irrigation_red %>% 
  #filter the original data set to only contain the rows for countries with single rows
  filter(country %in% countries_with_one_row$country) %>%
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
date
</th>
<th style="text-align:right;">
IrrigationPercentOfLand
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
BHR
</td>
<td style="text-align:left;">
Bahrain
</td>
<td style="text-align:right;">
2001
</td>
<td style="text-align:right;">
43.478262
</td>
</tr>
<tr>
<td style="text-align:left;">
CPV
</td>
<td style="text-align:left;">
Cabo Verde
</td>
<td style="text-align:right;">
2004
</td>
<td style="text-align:right;">
4.640000
</td>
</tr>
<tr>
<td style="text-align:left;">
CHL
</td>
<td style="text-align:left;">
Chile
</td>
<td style="text-align:right;">
2007
</td>
<td style="text-align:right;">
6.953979
</td>
</tr>
<tr>
<td style="text-align:left;">
SWZ
</td>
<td style="text-align:left;">
Eswatini
</td>
<td style="text-align:right;">
2002
</td>
<td style="text-align:right;">
3.663399
</td>
</tr>
<tr>
<td style="text-align:left;">
GRD
</td>
<td style="text-align:left;">
Grenada
</td>
<td style="text-align:right;">
2008
</td>
<td style="text-align:right;">
2.857143
</td>
</tr>
<tr>
<td style="text-align:left;">
GTM
</td>
<td style="text-align:left;">
Guatemala
</td>
<td style="text-align:right;">
2003
</td>
<td style="text-align:right;">
6.163112
</td>
</tr>
</tbody>
</table>
When we do this analysis, we see that the issue is that the dataset does not have data for all years for all countries. We can get a better idea of this by making a histogram.

``` r
irrigation_red %>% 
  #group dataframe by country name
  group_by(country) %>% 
  #count the number of rows - this is the number of years of data
  summarise(n=n()) %>% 
  ggplot(aes(n)) + geom_histogram() + 
  ggtitle("Number of years data per country in irrigation dataset")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](agricultural_data_files/figure-markdown_github/unnamed-chunk-17-1.png) We can see straight away that there are only 13 countries in our data set which span the whole range. At this stage we could ask if the countries with less than the full range are a continuous range (i.e. 2005-2008), or a number of sparse points accross the 16 years. Before digging deeper into that minor question, it would be better to investigate this for the other data sets.

``` r
tractors_red %>% 
  #group dataframe by country name
  group_by(country) %>% 
  #count the number of rows - this is the number of years of data
  summarise(n=n()) %>% 
  ggplot(aes(n)) + geom_histogram() +
  ggtitle("Number of years data per country in tractors dataset")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](agricultural_data_files/figure-markdown_github/unnamed-chunk-18-1.png)

``` r
fertilizer_red %>% 
  #group dataframe by country name
  group_by(country) %>% 
  #count the number of rows - this is the number of years of data
  summarise(n=n()) %>% 
  ggplot(aes(n)) + geom_histogram() + 
  ggtitle("Number of years data per country in fertilizer dataset")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](agricultural_data_files/figure-markdown_github/unnamed-chunk-19-1.png)

``` r
cereal_yield_red %>% 
  #group dataframe by country name
  group_by(country) %>% 
  #count the number of rows - this is the number of years of data
  summarise(n=n()) %>% 
  ggplot(aes(n)) + geom_histogram() +
  ggtitle("Number of years data per country in cereal yield dataset")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](agricultural_data_files/figure-markdown_github/unnamed-chunk-20-1.png)

It is interesting that cereal\_yield and tractors have a max number of rows per country of approximately 50 while fertilzier and irrigation ahve approx 16.

``` r
cereal_yield_red %>% 
  group_by(country) %>% 
  summarise(n=n()) %>% 
  arrange(desc(n))
```

    ## # A tibble: 227 x 2
    ##    country          n
    ##    <chr>        <int>
    ##  1 Afghanistan     57
    ##  2 Albania         57
    ##  3 Algeria         57
    ##  4 Angola          57
    ##  5 Arab World      57
    ##  6 Argentina       57
    ##  7 Australia       57
    ##  8 Austria         57
    ##  9 Bahamas, The    57
    ## 10 Bangladesh      57
    ## # ... with 217 more rows

Afghanistan has a 57 rows so we might as well pick that. An easy way of getting an idea of the full range of the data is a quick plot.

``` r
cereal_yield_red %>% filter(country=="Afghanistan") %>% 
  ggplot(aes(x=date,y=CerealYieldKgPerHectare)) + 
  geom_point() + 
  geom_line() +
  ggtitle("Cereal Yield in Afghanistan overlaid with key events")+
  #Monarchy deposed
  geom_vline(xintercept =  1973,color="sky blue") +
  #start of Soviet Afghan War
  geom_vline(xintercept =  1979,color="red") +
  #end of Soviet Afghan War
  geom_vline(xintercept = 1989,color="red") +
  #Start of civil war
  geom_vline(xintercept = 1992,color="dark green") +
  #end of civil war and start of Taliban rule
  geom_vline(xintercept = 1996,color="dark green") +
  #Start of Operation Enduring Freedom
  geom_vline(xintercept = 2001,color="blue") 
```

![](agricultural_data_files/figure-markdown_github/unnamed-chunk-22-1.png)

This gives us what is quite an interesting graph for a region that has suffered decades of turmoil. This provides us with a story that can provide some motivation for this work. What is driving these changes in cereal yields?

\*\*\* NOTE - SHOULD PUT THIS GRAPH AT THE START TO TRY AND PROVIDE A SOURCE OF MOTIVATION FOR THE STORY \*\*\*

A quick bit of research on wikipedia allowed me to add lines roughly seperating the different periods of recent Afghan history.

-   Afghanistan was ruled by monarchy until the last king Mohammed Zahir Shah was overthrown by his cousin Mohammed Daoud Khan in 1973.

-   In 1978 Afghanistan, the Saur Revolution led to an unpopular, aggresively modernising, socialist government backed by the USSR military.

-   By 1989 the USSR had given up on its campaign in the region. From 1992-1996 there was a civil war which ended with the Taliban taking control.

-   In 2001 the USA and it's allies launched Operation Enduring Freedom as a response to the September 11 terrorist attacks and the beginning of a massive campaign of rebuilding and investing in the region.

As we have the other data sets we can have a quick look at the fertilizer yield for Afghanistan.

``` r
fertilizer_red %>% 
  filter(country=="Afghanistan") %>% 
  ggplot(aes(x=date,y=FertilizerKgPerHectare)) + 
  geom_point() + 
  geom_line() +
  ggtitle("FertilizerPerHectare in Afghanistan")
```

![](agricultural_data_files/figure-markdown_github/unnamed-chunk-23-1.png)

``` r
tractors_red %>% filter(country == "Afghanistan") %>% 
  ggplot(aes(x=date,y=TractorsPer100Sqkm)) +
  geom_point()+
  geom_line()+
  ggtitle("Tractor usage in Afghanistan")
```

![](agricultural_data_files/figure-markdown_github/unnamed-chunk-24-1.png)

``` r
irrigation_red %>%
  filter(country=="Afghanistan") %>% 
  ggplot(aes(x=date,y=IrrigationPercentOfLand)) +
  geom_point() +
  geom_line()+
  ggtitle("Irrigation in Afghanistan")
```

![](agricultural_data_files/figure-markdown_github/unnamed-chunk-25-1.png)

As individual plots, these do not tell much of a story. We should combine them all together.

``` r
afghan_cereal <- cereal_yield_red %>% 
  filter(country=="Afghanistan")
afghan_irrigation <- irrigation_red %>%
  filter(country=="Afghanistan")
afghan_fertilizer <- fertilizer_red %>% 
  filter(country=="Afghanistan")
afghan_tractors <- tractors_red %>% 
  filter(country=="Afghanistan")

afghan_combined <- afghan_cereal %>% 
  left_join(afghan_fertilizer) %>% 
  left_join(afghan_irrigation) %>% 
  left_join(afghan_tractors)
```

    ## Joining, by = c("iso3c", "country", "date")
    ## Joining, by = c("iso3c", "country", "date")
    ## Joining, by = c("iso3c", "country", "date")

``` r
afghan_combined %>% 
  select(-c(iso3c,country)) %>% 
  gather(key,value,-date) %>% 
  ggplot(aes(x=date,y=value)) +
  facet_wrap(facets=vars(key),ncol=1, scales="free_y")+
  geom_point() +
  geom_line()
```

    ## Warning: Removed 100 rows containing missing values (geom_point).

    ## Warning: Removed 17 rows containing missing values (geom_path).

![](agricultural_data_files/figure-markdown_github/unnamed-chunk-26-1.png)

When taken together we can see that these datasets do not have a significant period of overlap for Afghanistan.

Returning to the histograms
---------------------------

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](agricultural_data_files/figure-markdown_github/unnamed-chunk-27-1.png)

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](agricultural_data_files/figure-markdown_github/unnamed-chunk-28-1.png)

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](agricultural_data_files/figure-markdown_github/unnamed-chunk-29-1.png)

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](agricultural_data_files/figure-markdown_github/unnamed-chunk-30-1.png)

When we examine each of these histograms in turn we see that the cereal yield and fertilizer datasets mostly consist of countries with data points for all countries in the years they cover. The fertilizer and tractor datasets cover a far shorter range of time than the cereal yield - approx 15 years vs 50 years. If we want to study the relationship of these variables we are only going to be able to do it for the 15 years where both data sets exist. This same logic applies for our other datasets (tractors and irrigation).
