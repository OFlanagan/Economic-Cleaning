Economic Cleaning
================

  - [initial work](#initial-work)

## initial work

This project is an excercise in data cleaning and data exploration. This
project will investigate some economic data.

This project was inspired by an economic freedom index dataset I found
on Kaggle Datasets.
<https://www.kaggle.com/lewisduncan93/the-economic-freedom-index/version/1#>

I quite like freedom and it would be interesting, and nice, to identify
the relationships between these economic freedom indicators and other
important things like economic performance, human well being,
corruption, politics, wars, natural disasters, environmental impact etc.

The idea of freedom can be a little contentious. Many of my readers will
be living in countries under some form of liberalism, an ideology which
focusses on the rights of the individual rather than the goals of the
collective.

However, most people are not in favour of complete freedom - very few
people think that we should live in some sort of anarchic mad max
situation where the strong take what they can and property belongs to
those who are strong enough to take and hold it.

There are also restrictions on freedoms around what consenting
individuals can do with each other. e.g. same sex relations.

Beyond these rights to be free from harm from others, nearly all
countries have laws controlling what an individual can do to themselves
- laws against killing oneself, laws against changing ones state of
conciousness with drug use.

The type of freedom I will be investigating in this study is economic
freedom, the freedom of individuals to pursue commercial activities.

NB - for this study we are going to assume that these metrics are in
some way true measures of economic freedom. The first thing we should do
is read the dataset and inspect it.

``` r
df <- read_csv("economic_freedom_index2019_data.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character(),
    ##   CountryID = col_double()
    ## )

    ## See spec(...) for full column specifications.

``` r
kable(df %>% head())
```

<table>

<thead>

<tr>

<th style="text-align:right;">

CountryID

</th>

<th style="text-align:left;">

Country Name

</th>

<th style="text-align:left;">

WEBNAME

</th>

<th style="text-align:left;">

Region

</th>

<th style="text-align:left;">

World Rank

</th>

<th style="text-align:left;">

Region Rank

</th>

<th style="text-align:left;">

2019 Score

</th>

<th style="text-align:left;">

Property Rights

</th>

<th style="text-align:left;">

Judical Effectiveness

</th>

<th style="text-align:left;">

Government Integrity

</th>

<th style="text-align:left;">

Tax Burden

</th>

<th style="text-align:left;">

Gov’t Spending

</th>

<th style="text-align:left;">

Fiscal Health

</th>

<th style="text-align:left;">

Business Freedom

</th>

<th style="text-align:left;">

Labor Freedom

</th>

<th style="text-align:left;">

Monetary Freedom

</th>

<th style="text-align:left;">

Trade Freedom

</th>

<th style="text-align:left;">

Investment Freedom

</th>

<th style="text-align:left;">

Financial Freedom

</th>

<th style="text-align:left;">

Tariff Rate (%)

</th>

<th style="text-align:left;">

Income Tax Rate (%)

</th>

<th style="text-align:left;">

Corporate Tax Rate (%)

</th>

<th style="text-align:left;">

Tax Burden % of GDP

</th>

<th style="text-align:left;">

Gov’t Expenditure % of GDP

</th>

<th style="text-align:left;">

Country

</th>

<th style="text-align:left;">

Population (Millions)

</th>

<th style="text-align:left;">

GDP (Billions, PPP)

</th>

<th style="text-align:left;">

GDP Growth Rate (%)

</th>

<th style="text-align:left;">

5 Year GDP Growth Rate (%)

</th>

<th style="text-align:left;">

GDP per Capita (PPP)

</th>

<th style="text-align:left;">

Unemployment (%)

</th>

<th style="text-align:left;">

Inflation (%)

</th>

<th style="text-align:left;">

FDI Inflow (Millions)

</th>

<th style="text-align:left;">

Public Debt (% of GDP)

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:right;">

1

</td>

<td style="text-align:left;">

Afghanistan

</td>

<td style="text-align:left;">

Afghanistan

</td>

<td style="text-align:left;">

Asia-Pacific

</td>

<td style="text-align:left;">

152

</td>

<td style="text-align:left;">

39

</td>

<td style="text-align:left;">

51.5

</td>

<td style="text-align:left;">

19.6

</td>

<td style="text-align:left;">

29.6

</td>

<td style="text-align:left;">

25.2

</td>

<td style="text-align:left;">

91.7

</td>

<td style="text-align:left;">

80.3

</td>

<td style="text-align:left;">

99.3

</td>

<td style="text-align:left;">

49.2

</td>

<td style="text-align:left;">

60.4

</td>

<td style="text-align:left;">

76.7

</td>

<td style="text-align:left;">

66.0

</td>

<td style="text-align:left;">

10

</td>

<td style="text-align:left;">

10

</td>

<td style="text-align:left;">

7.0

</td>

<td style="text-align:left;">

20.0

</td>

<td style="text-align:left;">

20.0

</td>

<td style="text-align:left;">

5.0

</td>

<td style="text-align:left;">

25.6

</td>

<td style="text-align:left;">

Afghanistan

</td>

<td style="text-align:left;">

35.5

</td>

<td style="text-align:left;">

$69.6

</td>

<td style="text-align:left;">

2.5

</td>

<td style="text-align:left;">

2.9

</td>

<td style="text-align:left;">

$1,958

</td>

<td style="text-align:left;">

8.8

</td>

<td style="text-align:left;">

5.0

</td>

<td style="text-align:left;">

53.9

</td>

<td style="text-align:left;">

7.3

</td>

</tr>

<tr>

<td style="text-align:right;">

2

</td>

<td style="text-align:left;">

Albania

</td>

<td style="text-align:left;">

Albania

</td>

<td style="text-align:left;">

Europe

</td>

<td style="text-align:left;">

52

</td>

<td style="text-align:left;">

27

</td>

<td style="text-align:left;">

66.5

</td>

<td style="text-align:left;">

54.8

</td>

<td style="text-align:left;">

30.6

</td>

<td style="text-align:left;">

40.4

</td>

<td style="text-align:left;">

86.3

</td>

<td style="text-align:left;">

73.9

</td>

<td style="text-align:left;">

80.6

</td>

<td style="text-align:left;">

69.3

</td>

<td style="text-align:left;">

52.7

</td>

<td style="text-align:left;">

81.5

</td>

<td style="text-align:left;">

87.8

</td>

<td style="text-align:left;">

70

</td>

<td style="text-align:left;">

70

</td>

<td style="text-align:left;">

1.1

</td>

<td style="text-align:left;">

23.0

</td>

<td style="text-align:left;">

15.0

</td>

<td style="text-align:left;">

24.9

</td>

<td style="text-align:left;">

29.5

</td>

<td style="text-align:left;">

Albania

</td>

<td style="text-align:left;">

2.9

</td>

<td style="text-align:left;">

$36.0

</td>

<td style="text-align:left;">

3.9

</td>

<td style="text-align:left;">

2.5

</td>

<td style="text-align:left;">

$12,507

</td>

<td style="text-align:left;">

13.9

</td>

<td style="text-align:left;">

2.0

</td>

<td style="text-align:left;">

1,119.1

</td>

<td style="text-align:left;">

71.2

</td>

</tr>

<tr>

<td style="text-align:right;">

3

</td>

<td style="text-align:left;">

Algeria

</td>

<td style="text-align:left;">

Algeria

</td>

<td style="text-align:left;">

Middle East and North Africa

</td>

<td style="text-align:left;">

171

</td>

<td style="text-align:left;">

14

</td>

<td style="text-align:left;">

46.2

</td>

<td style="text-align:left;">

31.6

</td>

<td style="text-align:left;">

36.2

</td>

<td style="text-align:left;">

28.9

</td>

<td style="text-align:left;">

76.4

</td>

<td style="text-align:left;">

48.7

</td>

<td style="text-align:left;">

18.7

</td>

<td style="text-align:left;">

61.6

</td>

<td style="text-align:left;">

49.9

</td>

<td style="text-align:left;">

74.9

</td>

<td style="text-align:left;">

67.4

</td>

<td style="text-align:left;">

30

</td>

<td style="text-align:left;">

30

</td>

<td style="text-align:left;">

8.8

</td>

<td style="text-align:left;">

35.0

</td>

<td style="text-align:left;">

23.0

</td>

<td style="text-align:left;">

24.5

</td>

<td style="text-align:left;">

41.4

</td>

<td style="text-align:left;">

Algeria

</td>

<td style="text-align:left;">

41.5

</td>

<td style="text-align:left;">

$632.9

</td>

<td style="text-align:left;">

2.0

</td>

<td style="text-align:left;">

3.1

</td>

<td style="text-align:left;">

$15,237

</td>

<td style="text-align:left;">

10.0

</td>

<td style="text-align:left;">

5.6

</td>

<td style="text-align:left;">

1,203.0

</td>

<td style="text-align:left;">

25.8

</td>

</tr>

<tr>

<td style="text-align:right;">

4

</td>

<td style="text-align:left;">

Angola

</td>

<td style="text-align:left;">

Angola

</td>

<td style="text-align:left;">

Sub-Saharan Africa

</td>

<td style="text-align:left;">

156

</td>

<td style="text-align:left;">

33

</td>

<td style="text-align:left;">

50.6

</td>

<td style="text-align:left;">

35.9

</td>

<td style="text-align:left;">

26.6

</td>

<td style="text-align:left;">

20.5

</td>

<td style="text-align:left;">

83.9

</td>

<td style="text-align:left;">

80.7

</td>

<td style="text-align:left;">

58.2

</td>

<td style="text-align:left;">

55.7

</td>

<td style="text-align:left;">

58.8

</td>

<td style="text-align:left;">

55.4

</td>

<td style="text-align:left;">

61.2

</td>

<td style="text-align:left;">

30

</td>

<td style="text-align:left;">

40

</td>

<td style="text-align:left;">

9.4

</td>

<td style="text-align:left;">

17.0

</td>

<td style="text-align:left;">

30.0

</td>

<td style="text-align:left;">

20.6

</td>

<td style="text-align:left;">

25.3

</td>

<td style="text-align:left;">

Angola

</td>

<td style="text-align:left;">

28.2

</td>

<td style="text-align:left;">

$190.3

</td>

<td style="text-align:left;">

0.7

</td>

<td style="text-align:left;">

2.9

</td>

<td style="text-align:left;">

$6,753

</td>

<td style="text-align:left;">

8.2

</td>

<td style="text-align:left;">

31.7

</td>

<td style="text-align:left;">

\-2,254.5

</td>

<td style="text-align:left;">

65.3

</td>

</tr>

<tr>

<td style="text-align:right;">

5

</td>

<td style="text-align:left;">

Argentina

</td>

<td style="text-align:left;">

Argentina

</td>

<td style="text-align:left;">

Americas

</td>

<td style="text-align:left;">

148

</td>

<td style="text-align:left;">

26

</td>

<td style="text-align:left;">

52.2

</td>

<td style="text-align:left;">

47.8

</td>

<td style="text-align:left;">

44.5

</td>

<td style="text-align:left;">

33.5

</td>

<td style="text-align:left;">

69.3

</td>

<td style="text-align:left;">

49.5

</td>

<td style="text-align:left;">

33.0

</td>

<td style="text-align:left;">

56.4

</td>

<td style="text-align:left;">

46.9

</td>

<td style="text-align:left;">

60.2

</td>

<td style="text-align:left;">

70.0

</td>

<td style="text-align:left;">

55

</td>

<td style="text-align:left;">

60

</td>

<td style="text-align:left;">

7.5

</td>

<td style="text-align:left;">

35.0

</td>

<td style="text-align:left;">

30.0

</td>

<td style="text-align:left;">

30.8

</td>

<td style="text-align:left;">

41.0

</td>

<td style="text-align:left;">

Argentina

</td>

<td style="text-align:left;">

44.1

</td>

<td style="text-align:left;">

$920.2

</td>

<td style="text-align:left;">

2.9

</td>

<td style="text-align:left;">

0.7

</td>

<td style="text-align:left;">

$20,876

</td>

<td style="text-align:left;">

8.7

</td>

<td style="text-align:left;">

25.7

</td>

<td style="text-align:left;">

11,857.0

</td>

<td style="text-align:left;">

52.6

</td>

</tr>

<tr>

<td style="text-align:right;">

6

</td>

<td style="text-align:left;">

Armenia

</td>

<td style="text-align:left;">

Armenia

</td>

<td style="text-align:left;">

Europe

</td>

<td style="text-align:left;">

47

</td>

<td style="text-align:left;">

24

</td>

<td style="text-align:left;">

67.7

</td>

<td style="text-align:left;">

57.2

</td>

<td style="text-align:left;">

46.3

</td>

<td style="text-align:left;">

38.6

</td>

<td style="text-align:left;">

84.7

</td>

<td style="text-align:left;">

79.0

</td>

<td style="text-align:left;">

53.0

</td>

<td style="text-align:left;">

78.3

</td>

<td style="text-align:left;">

71.4

</td>

<td style="text-align:left;">

77.8

</td>

<td style="text-align:left;">

80.8

</td>

<td style="text-align:left;">

75

</td>

<td style="text-align:left;">

70

</td>

<td style="text-align:left;">

2.1

</td>

<td style="text-align:left;">

26.0

</td>

<td style="text-align:left;">

20.0

</td>

<td style="text-align:left;">

21.3

</td>

<td style="text-align:left;">

26.4

</td>

<td style="text-align:left;">

Armenia

</td>

<td style="text-align:left;">

3.0

</td>

<td style="text-align:left;">

$28.3

</td>

<td style="text-align:left;">

7.5

</td>

<td style="text-align:left;">

3.6

</td>

<td style="text-align:left;">

$9,456

</td>

<td style="text-align:left;">

18.2

</td>

<td style="text-align:left;">

0.9

</td>

<td style="text-align:left;">

245.7

</td>

<td style="text-align:left;">

53.5

</td>

</tr>

</tbody>

</table>

This does seem interesting, we have quite a few columns, with one row
for each country. Many of these columns have been read in incorrectly.
However, this dataset only contained data for 2019. I looked at the
source website and it appears that the data is only available year by
year <https://www.heritage.org/> This is not a problem, I can simply
download the files for each year and then merge them together.

``` r
name_list <- list.files("economic_freedom/")

extract_economic_freedom_year <- function(file){
  #take a file name in the economic_freedom folder
  #read the csv into memory
  #make sure all columns except for name are treated as numeric
  read_csv(paste0("economic_freedom/",file),
           na="N/A") %>% 
  mutate_at(vars(-name),as.numeric)
}

data_list <- name_list %>% map(extract_economic_freedom_year)
economic_freedom <- data_list %>% bind_rows()
#spaces in column names causes trouble so we will replace these with underscores
#we will be joining this data set with other datasets so it makes sense to 
#add a label indicating the source dataset here.
names(economic_freedom) <- str_c("efi_",names(economic_freedom) %>% str_replace(" ","_"))
kable(economic_freedom %>% head())
```

<table>

<thead>

<tr>

<th style="text-align:left;">

efi\_name

</th>

<th style="text-align:right;">

efi\_index\_year

</th>

<th style="text-align:right;">

efi\_overall\_score

</th>

<th style="text-align:right;">

efi\_property\_rights

</th>

<th style="text-align:right;">

efi\_government\_integrity

</th>

<th style="text-align:right;">

efi\_judicial\_effectiveness

</th>

<th style="text-align:right;">

efi\_tax\_burden

</th>

<th style="text-align:right;">

efi\_government\_spending

</th>

<th style="text-align:right;">

efi\_fiscal\_health

</th>

<th style="text-align:right;">

efi\_business\_freedom

</th>

<th style="text-align:right;">

efi\_labor\_freedom

</th>

<th style="text-align:right;">

efi\_monetary\_freedom

</th>

<th style="text-align:right;">

efi\_trade\_freedom

</th>

<th style="text-align:right;">

efi\_investment\_freedom

</th>

<th style="text-align:right;">

efi\_financial\_freedom

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

Afghanistan

</td>

<td style="text-align:right;">

1995

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

Albania

</td>

<td style="text-align:right;">

1995

</td>

<td style="text-align:right;">

49.7

</td>

<td style="text-align:right;">

50

</td>

<td style="text-align:right;">

10

</td>

<td style="text-align:right;">

NA

</td>

<td style="text-align:right;">

81.7

</td>

<td style="text-align:right;">

34.3

</td>

<td style="text-align:right;">

NA

</td>

<td style="text-align:right;">

70

</td>

<td style="text-align:right;">

NA

</td>

<td style="text-align:right;">

22.1

</td>

<td style="text-align:right;">

59.0

</td>

<td style="text-align:right;">

70

</td>

<td style="text-align:right;">

50

</td>

</tr>

<tr>

<td style="text-align:left;">

Algeria

</td>

<td style="text-align:right;">

1995

</td>

<td style="text-align:right;">

55.7

</td>

<td style="text-align:right;">

50

</td>

<td style="text-align:right;">

50

</td>

<td style="text-align:right;">

NA

</td>

<td style="text-align:right;">

48.8

</td>

<td style="text-align:right;">

69.5

</td>

<td style="text-align:right;">

NA

</td>

<td style="text-align:right;">

70

</td>

<td style="text-align:right;">

NA

</td>

<td style="text-align:right;">

59.2

</td>

<td style="text-align:right;">

54.2

</td>

<td style="text-align:right;">

50

</td>

<td style="text-align:right;">

50

</td>

</tr>

<tr>

<td style="text-align:left;">

Angola

</td>

<td style="text-align:right;">

1995

</td>

<td style="text-align:right;">

27.4

</td>

<td style="text-align:right;">

30

</td>

<td style="text-align:right;">

30

</td>

<td style="text-align:right;">

NA

</td>

<td style="text-align:right;">

61.6

</td>

<td style="text-align:right;">

0.0

</td>

<td style="text-align:right;">

NA

</td>

<td style="text-align:right;">

40

</td>

<td style="text-align:right;">

NA

</td>

<td style="text-align:right;">

0.0

</td>

<td style="text-align:right;">

25.0

</td>

<td style="text-align:right;">

30

</td>

<td style="text-align:right;">

30

</td>

</tr>

<tr>

<td style="text-align:left;">

Argentina

</td>

<td style="text-align:right;">

1995

</td>

<td style="text-align:right;">

68.0

</td>

<td style="text-align:right;">

70

</td>

<td style="text-align:right;">

50

</td>

<td style="text-align:right;">

NA

</td>

<td style="text-align:right;">

80.7

</td>

<td style="text-align:right;">

86.6

</td>

<td style="text-align:right;">

NA

</td>

<td style="text-align:right;">

85

</td>

<td style="text-align:right;">

NA

</td>

<td style="text-align:right;">

61.1

</td>

<td style="text-align:right;">

58.4

</td>

<td style="text-align:right;">

70

</td>

<td style="text-align:right;">

50

</td>

</tr>

<tr>

<td style="text-align:left;">

Armenia

</td>

<td style="text-align:right;">

1995

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

We now have a reasonable dataset and can perform some initial plots. The
most basic plot we can make is a line plot of our time series.

``` r
economic_freedom %>% 
  ggplot(aes(x=efi_index_year,y=efi_overall_score,color=efi_name)) + 
  geom_line() +
  theme(legend.position = "none") +
  ggtitle("Overall Economic Freedom Scores of all countries since 1995")
```

    ## Warning: Removed 455 rows containing missing values (geom_path).

![](project_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

This plot is quite crowded and we have had to leave off the key
indicating which colors are which. There are a few interesting points we
can gather from this: \* some countries start with high freedom and
maintain steady high freedom \* some countries start with low freedom
and maintain steady low freedom \* some countries do not have data
spanning the full range. We can easily filter out a few of these
countries and identify them. These outliers may be interesting.

``` r
economic_freedom %>% 
  filter(efi_overall_score > 85 | efi_overall_score < 20) %>% 
  ggplot(aes(x=efi_index_year,y=efi_overall_score,color=efi_name)) + 
  geom_line() +
  ggtitle("Overall Economic Freedom Scores of outlier countries since 1995")
```

![](project_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

That simple filter was very helpful. We see that Singapore and Hong Kong
have had long term high economic freedom, and that North Korea has had
long term low economic freedom. It is commonly known that Singapore and
Hong Kong are quite wealthy, while North Korea is not as wealthy but
these are outliers and only a few data points. We also see that data
about Iraq ceases to be available after 2002, this makes sense as the US
and its allies invaded Iraq in 2003. The method used to isolate the Iraq
data was quite simplistic so we should check that the data does indeed
end for Iraq, the alternative is that there could be a gap and then
significantly higher data later on.

``` r
economic_freedom %>% 
  filter(efi_name=="Iraq") %>% 
  select(efi_overall_score) %>% 
  summary()
```

    ##  efi_overall_score
    ##  Min.   :15.60    
    ##  1st Qu.:17.20    
    ##  Median :17.20    
    ##  Mean   :16.97    
    ##  3rd Qu.:17.20    
    ##  Max.   :17.20    
    ##  NA's   :18

There are some interesting questions we can ask: linear model to
identify general trends calculate variance to see increases and
decreases how does this measure correlate with important things like
wealth and health

We can also investigate the other components of the EFI dataset in a
similar way.

\#Dataset \#2 - World bank data

The economic freedom data is quite interesting, but it is quite limited
in what it tells us. The data is a set of calculated measures which were
calculated with the aim of quantifying economic freedom. The goal of
this work is to explore the impact of economic freedom.

``` r
str(wb_cachelist, max.level = 1)
```

    ## List of 7
    ##  $ countries  :'data.frame': 304 obs. of  18 variables:
    ##  $ indicators :'data.frame': 16978 obs. of  7 variables:
    ##  $ sources    :'data.frame': 43 obs. of  8 variables:
    ##  $ datacatalog:'data.frame': 238 obs. of  29 variables:
    ##  $ topics     :'data.frame': 21 obs. of  3 variables:
    ##  $ income     :'data.frame': 7 obs. of  3 variables:
    ##  $ lending    :'data.frame': 4 obs. of  3 variables:

There are 7 dataframes available in the wbstats data list.

``` r
kable(wb_cachelist$countries %>% head())
```

<table>

<thead>

<tr>

<th style="text-align:left;">

iso3c

</th>

<th style="text-align:left;">

iso2c

</th>

<th style="text-align:left;">

country

</th>

<th style="text-align:left;">

capital

</th>

<th style="text-align:left;">

long

</th>

<th style="text-align:left;">

lat

</th>

<th style="text-align:left;">

regionID

</th>

<th style="text-align:left;">

region\_iso2c

</th>

<th style="text-align:left;">

region

</th>

<th style="text-align:left;">

adminID

</th>

<th style="text-align:left;">

admin\_iso2c

</th>

<th style="text-align:left;">

admin

</th>

<th style="text-align:left;">

incomeID

</th>

<th style="text-align:left;">

income\_iso2c

</th>

<th style="text-align:left;">

income

</th>

<th style="text-align:left;">

lendingID

</th>

<th style="text-align:left;">

lending\_iso2c

</th>

<th style="text-align:left;">

lending

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

ABW

</td>

<td style="text-align:left;">

AW

</td>

<td style="text-align:left;">

Aruba

</td>

<td style="text-align:left;">

Oranjestad

</td>

<td style="text-align:left;">

\-70.0167

</td>

<td style="text-align:left;">

12.5167

</td>

<td style="text-align:left;">

LCN

</td>

<td style="text-align:left;">

ZJ

</td>

<td style="text-align:left;">

Latin America & Caribbean

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

HIC

</td>

<td style="text-align:left;">

XD

</td>

<td style="text-align:left;">

High income

</td>

<td style="text-align:left;">

LNX

</td>

<td style="text-align:left;">

XX

</td>

<td style="text-align:left;">

Not classified

</td>

</tr>

<tr>

<td style="text-align:left;">

AFG

</td>

<td style="text-align:left;">

AF

</td>

<td style="text-align:left;">

Afghanistan

</td>

<td style="text-align:left;">

Kabul

</td>

<td style="text-align:left;">

69.1761

</td>

<td style="text-align:left;">

34.5228

</td>

<td style="text-align:left;">

SAS

</td>

<td style="text-align:left;">

8S

</td>

<td style="text-align:left;">

South Asia

</td>

<td style="text-align:left;">

SAS

</td>

<td style="text-align:left;">

8S

</td>

<td style="text-align:left;">

South Asia

</td>

<td style="text-align:left;">

LIC

</td>

<td style="text-align:left;">

XM

</td>

<td style="text-align:left;">

Low income

</td>

<td style="text-align:left;">

IDX

</td>

<td style="text-align:left;">

XI

</td>

<td style="text-align:left;">

IDA

</td>

</tr>

<tr>

<td style="text-align:left;">

AFR

</td>

<td style="text-align:left;">

A9

</td>

<td style="text-align:left;">

Africa

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

Aggregates

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

Aggregates

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

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

AO

</td>

<td style="text-align:left;">

Angola

</td>

<td style="text-align:left;">

Luanda

</td>

<td style="text-align:left;">

13.242

</td>

<td style="text-align:left;">

\-8.81155

</td>

<td style="text-align:left;">

SSF

</td>

<td style="text-align:left;">

ZG

</td>

<td style="text-align:left;">

Sub-Saharan Africa

</td>

<td style="text-align:left;">

SSA

</td>

<td style="text-align:left;">

ZF

</td>

<td style="text-align:left;">

Sub-Saharan Africa (excluding high income)

</td>

<td style="text-align:left;">

LMC

</td>

<td style="text-align:left;">

XN

</td>

<td style="text-align:left;">

Lower middle income

</td>

<td style="text-align:left;">

IBD

</td>

<td style="text-align:left;">

XF

</td>

<td style="text-align:left;">

IBRD

</td>

</tr>

<tr>

<td style="text-align:left;">

ALB

</td>

<td style="text-align:left;">

AL

</td>

<td style="text-align:left;">

Albania

</td>

<td style="text-align:left;">

Tirane

</td>

<td style="text-align:left;">

19.8172

</td>

<td style="text-align:left;">

41.3317

</td>

<td style="text-align:left;">

ECS

</td>

<td style="text-align:left;">

Z7

</td>

<td style="text-align:left;">

Europe & Central Asia

</td>

<td style="text-align:left;">

ECA

</td>

<td style="text-align:left;">

7E

</td>

<td style="text-align:left;">

Europe & Central Asia (excluding high income)

</td>

<td style="text-align:left;">

UMC

</td>

<td style="text-align:left;">

XT

</td>

<td style="text-align:left;">

Upper middle income

</td>

<td style="text-align:left;">

IBD

</td>

<td style="text-align:left;">

XF

</td>

<td style="text-align:left;">

IBRD

</td>

</tr>

<tr>

<td style="text-align:left;">

AND

</td>

<td style="text-align:left;">

AD

</td>

<td style="text-align:left;">

Andorra

</td>

<td style="text-align:left;">

Andorra la Vella

</td>

<td style="text-align:left;">

1.5218

</td>

<td style="text-align:left;">

42.5075

</td>

<td style="text-align:left;">

ECS

</td>

<td style="text-align:left;">

Z7

</td>

<td style="text-align:left;">

Europe & Central Asia

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

HIC

</td>

<td style="text-align:left;">

XD

</td>

<td style="text-align:left;">

High income

</td>

<td style="text-align:left;">

LNX

</td>

<td style="text-align:left;">

XX

</td>

<td style="text-align:left;">

Not classified

</td>

</tr>

</tbody>

</table>

This countries dataset includes a lot of detail which we can join onto
our EFI dataset.

``` r
kable(wb_cachelist$indicators %>% head(1))
```

<table>

<thead>

<tr>

<th style="text-align:left;">

indicatorID

</th>

<th style="text-align:left;">

indicator

</th>

<th style="text-align:left;">

unit

</th>

<th style="text-align:left;">

indicatorDesc

</th>

<th style="text-align:left;">

sourceOrg

</th>

<th style="text-align:left;">

sourceID

</th>

<th style="text-align:left;">

source

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

ZINC

</td>

<td style="text-align:left;">

Zinc, cents/kg, current$

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

Zinc (LME), high grade, minimum 99.95% purity, settlement price
beginning April 1990; previously special high grade, minimum 99.995%,
cash prices

</td>

<td style="text-align:left;">

Platts Metals Week, Engineering and Mining Journal; Thomson Reuters
Datastream; World Bank.

</td>

<td style="text-align:left;">

21

</td>

<td style="text-align:left;">

Global Economic Monitor Commodities

</td>

</tr>

</tbody>

</table>

The indicators dataframe appears to be a list of all indicators
available with descriptions. We will need to do a little more research
before we can pick some interesting indicators as there are literally
thousands of indicators including zinc prices.

``` r
kable(wb_cachelist$sources %>% head())
```

<table>

<thead>

<tr>

<th style="text-align:left;">

sourceID

</th>

<th style="text-align:left;">

lastUpdated

</th>

<th style="text-align:left;">

source

</th>

<th style="text-align:left;">

sourceAbbr

</th>

<th style="text-align:left;">

sourceDesc

</th>

<th style="text-align:left;">

sourceURL

</th>

<th style="text-align:left;">

dataAvail

</th>

<th style="text-align:left;">

metadataAvail

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

11

</td>

<td style="text-align:left;">

2013-02-22

</td>

<td style="text-align:left;">

Africa Development Indicators

</td>

<td style="text-align:left;">

ADI

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

Y

</td>

<td style="text-align:left;">

Y

</td>

</tr>

<tr>

<td style="text-align:left;">

36

</td>

<td style="text-align:left;">

2017-11-06

</td>

<td style="text-align:left;">

Statistical Capacity Indicators

</td>

<td style="text-align:left;">

BBS

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

Y

</td>

<td style="text-align:left;">

NA

</td>

</tr>

<tr>

<td style="text-align:left;">

31

</td>

<td style="text-align:left;">

2017-07-18

</td>

<td style="text-align:left;">

Country Policy and Institutional Assessment

</td>

<td style="text-align:left;">

CPI

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

Y

</td>

<td style="text-align:left;">

Y

</td>

</tr>

<tr>

<td style="text-align:left;">

41

</td>

<td style="text-align:left;">

2015-05-22

</td>

<td style="text-align:left;">

Country Partnership Strategy for India (FY2013 - 17)

</td>

<td style="text-align:left;">

CPS

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

Y

</td>

<td style="text-align:left;">

N

</td>

</tr>

<tr>

<td style="text-align:left;">

1

</td>

<td style="text-align:left;">

2017-01-03

</td>

<td style="text-align:left;">

Doing Business

</td>

<td style="text-align:left;">

DBS

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

Y

</td>

<td style="text-align:left;">

Y

</td>

</tr>

<tr>

<td style="text-align:left;">

30

</td>

<td style="text-align:left;">

2016-03-31

</td>

<td style="text-align:left;">

Exporter Dynamics Database – Indicators at Country-Year Level

</td>

<td style="text-align:left;">

ED1

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

Y

</td>

<td style="text-align:left;">

N

</td>

</tr>

</tbody>

</table>

``` r
kable(wb_cachelist$datacatalog %>% head(1))
```

<table>

<thead>

<tr>

<th style="text-align:left;">

source

</th>

<th style="text-align:left;">

sourceAbbr

</th>

<th style="text-align:left;">

sourceDesc

</th>

<th style="text-align:left;">

url

</th>

<th style="text-align:left;">

type

</th>

<th style="text-align:left;">

langSupport

</th>

<th style="text-align:left;">

periodicity

</th>

<th style="text-align:left;">

econCoverage

</th>

<th style="text-align:left;">

granularity

</th>

<th style="text-align:left;">

numEcons

</th>

<th style="text-align:left;">

topics

</th>

<th style="text-align:left;">

updateFreq

</th>

<th style="text-align:left;">

updateSched

</th>

<th style="text-align:left;">

lastRevision

</th>

<th style="text-align:left;">

contactInfo

</th>

<th style="text-align:left;">

accessOpt

</th>

<th style="text-align:left;">

bulkDownload

</th>

<th style="text-align:left;">

cite

</th>

<th style="text-align:left;">

detailURL

</th>

<th style="text-align:left;">

popularity

</th>

<th style="text-align:left;">

coverage

</th>

<th style="text-align:left;">

api

</th>

<th style="text-align:left;">

apiURL

</th>

<th style="text-align:left;">

SourceID

</th>

<th style="text-align:left;">

dataNotes

</th>

<th style="text-align:left;">

mobileApp

</th>

<th style="text-align:left;">

geoCoverage

</th>

<th style="text-align:left;">

sourceURL

</th>

<th style="text-align:left;">

apiLocation

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

World Development Indicators

</td>

<td style="text-align:left;">

WDI

</td>

<td style="text-align:left;">

The primary World Bank collection of development indicators, compiled
from officially-recognized international sources. It presents the most
current and accurate global development data available, and includes
national, regional and global
estimates.

</td>

<td style="text-align:left;">

<http://databank.worldbank.org/data/views/variableSelection/selectvariables.aspx?source=world-development-indicators>

</td>

<td style="text-align:left;">

Time series

</td>

<td style="text-align:left;">

English, Spanish, French, Arabic, Chinese

</td>

<td style="text-align:left;">

Annual

</td>

<td style="text-align:left;">

WLD, EAP, ECA, LAC, MNA, SAS, SSA, HIC, LMY, IBRD, IDA

</td>

<td style="text-align:left;">

National, Regional

</td>

<td style="text-align:left;">

217

</td>

<td style="text-align:left;">

Agriculture & Rural Development, Aid Effectiveness, Climate Change,
Economy & Growth, Education, Energy & Mining, Environment, External
Debt, Financial Sector, Gender, Health, Infrastructure, Labor & Social
Protection, Poverty, Private Sector, Public Sector, Science &
Technology, Social Development, Trade, Urban Development

</td>

<td style="text-align:left;">

Quarterly

</td>

<td style="text-align:left;">

April, July, September, December

</td>

<td style="text-align:left;">

15-Sep-2017

</td>

<td style="text-align:left;">

<data@worldbank.org>

</td>

<td style="text-align:left;">

API, Bulk download, Query tool

</td>

<td style="text-align:left;">

WDI (Excel)-ZIP (59
MB)=<http://databank.worldbank.org/data/download/WDI_excel.zip=excel;WDI>
(CSV)-ZIP (57
MB)=<http://databank.worldbank.org/data/download/WDI_csv.zip=csv;Information>
about WDI revisions (Excel) (912
KB)=<http://databank.worldbank.org/data/download/WDIrevisions.xls=excel>

</td>

<td style="text-align:left;">

World Development Indicators, The World Bank

</td>

<td style="text-align:left;">

<http://data.worldbank.org/data-catalog/world-development-indicators>

</td>

<td style="text-align:left;">

3765

</td>

<td style="text-align:left;">

1960 - 2016

</td>

<td style="text-align:left;">

1

</td>

<td style="text-align:left;">

<http://data.worldbank.org/developers>

</td>

<td style="text-align:left;">

2

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA

</td>

</tr>

</tbody>

</table>

``` r
kable(wb_cachelist$topics %>% head(3))
```

<table>

<thead>

<tr>

<th style="text-align:left;">

topicID

</th>

<th style="text-align:left;">

topic

</th>

<th style="text-align:left;">

topicDesc

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

1

</td>

<td style="text-align:left;">

Agriculture & Rural Development

</td>

<td style="text-align:left;">

For the 70 percent of the world’s poor who live in rural areas,
agriculture is the main source of income and employment. But depletion
and degradation of land and water pose serious challenges to producing
enough food and other agricultural products to sustain livelihoods here
and meet the needs of urban populations. Data presented here include
measures of agricultural inputs, outputs, and productivity compiled by
the UN’s Food and Agriculture Organization.

</td>

</tr>

<tr>

<td style="text-align:left;">

2

</td>

<td style="text-align:left;">

Aid Effectiveness

</td>

<td style="text-align:left;">

Aid effectiveness is the impact that aid has in reducing poverty and
inequality, increasing growth, building capacity, and accelerating
achievement of the Millennium Development Goals set by the international
community. Indicators here cover aid received as well as progress in
reducing poverty and improving education, health, and other measures of
human welfare.

</td>

</tr>

<tr>

<td style="text-align:left;">

3

</td>

<td style="text-align:left;">

Economy & Growth

</td>

<td style="text-align:left;">

Economic growth is central to economic development. When national income
grows, real people benefit. While there is no known formula for
stimulating economic growth, data can help policy-makers better
understand their countries’ economic situations and guide any work
toward improvement. Data here covers measures of economic growth, such
as gross domestic product (GDP) and gross national income (GNI). It also
includes indicators representing factors known to be relevant to
economic growth, such as capital stock, employment, investment, savings,
consumption, government spending, imports, and exports.

</td>

</tr>

</tbody>

</table>

``` r
kable(wb_cachelist$income %>% head())
```

<table>

<thead>

<tr>

<th style="text-align:left;">

incomeID

</th>

<th style="text-align:left;">

iso2c

</th>

<th style="text-align:left;">

income

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

HIC

</td>

<td style="text-align:left;">

XD

</td>

<td style="text-align:left;">

High income

</td>

</tr>

<tr>

<td style="text-align:left;">

INX

</td>

<td style="text-align:left;">

XY

</td>

<td style="text-align:left;">

Not classified

</td>

</tr>

<tr>

<td style="text-align:left;">

LIC

</td>

<td style="text-align:left;">

XM

</td>

<td style="text-align:left;">

Low income

</td>

</tr>

<tr>

<td style="text-align:left;">

LMC

</td>

<td style="text-align:left;">

XN

</td>

<td style="text-align:left;">

Lower middle income

</td>

</tr>

<tr>

<td style="text-align:left;">

LMY

</td>

<td style="text-align:left;">

XO

</td>

<td style="text-align:left;">

Low & middle income

</td>

</tr>

<tr>

<td style="text-align:left;">

MIC

</td>

<td style="text-align:left;">

XP

</td>

<td style="text-align:left;">

Middle income

</td>

</tr>

</tbody>

</table>

``` r
kable(wb_cachelist$lending %>% head())
```

<table>

<thead>

<tr>

<th style="text-align:left;">

lendingID

</th>

<th style="text-align:left;">

iso2c

</th>

<th style="text-align:left;">

lending

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

IBD

</td>

<td style="text-align:left;">

XF

</td>

<td style="text-align:left;">

IBRD

</td>

</tr>

<tr>

<td style="text-align:left;">

IDB

</td>

<td style="text-align:left;">

XH

</td>

<td style="text-align:left;">

Blend

</td>

</tr>

<tr>

<td style="text-align:left;">

IDX

</td>

<td style="text-align:left;">

XI

</td>

<td style="text-align:left;">

IDA

</td>

</tr>

<tr>

<td style="text-align:left;">

LNX

</td>

<td style="text-align:left;">

XX

</td>

<td style="text-align:left;">

Not classified

</td>

</tr>

</tbody>

</table>
