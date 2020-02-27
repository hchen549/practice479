Practice Exam
=============

This practice exam asks you to do several code wrangling tasks that we
have done in class so far.

Clone this repo into Rstudio and fill in the necessary code. Then,
commit and push to github. Finally, turn in a link to canvas.

    ## -- Attaching packages ---------------------------------------------------- tidyverse 1.3.0 --

    ## √ ggplot2 3.2.1     √ purrr   0.3.3
    ## √ tibble  2.1.3     √ dplyr   0.8.3
    ## √ tidyr   1.0.0     √ stringr 1.4.0
    ## √ readr   1.3.1     √ forcats 0.4.0

    ## -- Conflicts ------------------------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

    ## Warning: package 'nycflights13' was built under R version 3.6.2

Make a plot with three facets, one for each airport in the weather data.
The x-axis should be the day of the year (1:365) and the y-axis should
be the mean temperature recorded on that day, at that airport.

    library(lubridate)

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

    new_weather <- weather %>% mutate(day_of_year = yday(time_hour))
    weather %>% mutate(day_of_year = yday(time_hour)) %>%
      group_by(day_of_year) %>%
      summarise(mean_temperature = mean(temp)) %>%
      left_join(new_weather) %>%
      ggplot(aes(x = day_of_year, mean_temperature)) +
      geom_line() +
      facet_grid(~origin)

    ## Joining, by = "day_of_year"

![](README_files/figure-markdown_strict/unnamed-chunk-2-1.png)

Make a non-tidy matrix of that data where each row is an airport and
each column is a day of the year.

    #new_weather %>%
     # pivot_wider(names_from = day_of_year, values_from = origin)

    ?pivot_wider

    ## starting httpd help server ... done

For each (airport, day) contruct a tidy data set of the airport’s
“performance” as the proportion of flights that departed less than an
hour late.

    weather

    ## # A tibble: 26,115 x 15
    ##    origin  year month   day  hour  temp  dewp humid wind_dir wind_speed
    ##    <chr>  <int> <int> <int> <int> <dbl> <dbl> <dbl>    <dbl>      <dbl>
    ##  1 EWR     2013     1     1     1  39.0  26.1  59.4      270      10.4 
    ##  2 EWR     2013     1     1     2  39.0  27.0  61.6      250       8.06
    ##  3 EWR     2013     1     1     3  39.0  28.0  64.4      240      11.5 
    ##  4 EWR     2013     1     1     4  39.9  28.0  62.2      250      12.7 
    ##  5 EWR     2013     1     1     5  39.0  28.0  64.4      260      12.7 
    ##  6 EWR     2013     1     1     6  37.9  28.0  67.2      240      11.5 
    ##  7 EWR     2013     1     1     7  39.0  28.0  64.4      240      15.0 
    ##  8 EWR     2013     1     1     8  39.9  28.0  62.2      250      10.4 
    ##  9 EWR     2013     1     1     9  39.9  28.0  62.2      260      15.0 
    ## 10 EWR     2013     1     1    10  41    28.0  59.6      260      13.8 
    ## # ... with 26,105 more rows, and 5 more variables: wind_gust <dbl>,
    ## #   precip <dbl>, pressure <dbl>, visib <dbl>, time_hour <dttm>

    flights %>%
      mutate(date = ymd(paste(year,"-", month,"-" ,day))) %>%
      mutate(dep_late = dep_delay >=60) %>%
      group_by(origin, date) %>%
      summarise(proportion = mean(dep_late, na.rm = TRUE))

    ## # A tibble: 1,095 x 3
    ## # Groups:   origin [3]
    ##    origin date       proportion
    ##    <chr>  <date>          <dbl>
    ##  1 EWR    2013-01-01     0.0822
    ##  2 EWR    2013-01-02     0.163 
    ##  3 EWR    2013-01-03     0.0210
    ##  4 EWR    2013-01-04     0.0653
    ##  5 EWR    2013-01-05     0.0338
    ##  6 EWR    2013-01-06     0.05  
    ##  7 EWR    2013-01-07     0.0789
    ##  8 EWR    2013-01-08     0.0181
    ##  9 EWR    2013-01-09     0.0239
    ## 10 EWR    2013-01-10     0.0204
    ## # ... with 1,085 more rows

Construct a tidy data set to that give weather summaries for each
(airport, day). Use the total precipitation, minimum visibility, maximum
wind\_gust, and average wind\_speed.

    new_weather %>%
      group_by(origin, day) %>%
      summarise(total_prec = sum(precip), min_vis = min(visib), max_wind = max(wind_gust, na.rm = TRUE), avg_wind_speed = mean(wind_speed, na.rm = TRUE))

    ## # A tibble: 93 x 6
    ## # Groups:   origin [3]
    ##    origin   day total_prec min_vis max_wind avg_wind_speed
    ##    <chr>  <int>      <dbl>   <dbl>    <dbl>          <dbl>
    ##  1 EWR        1       1.91    1.5      33.4           9.47
    ##  2 EWR        2       1.25    2        33.4           8.63
    ##  3 EWR        3       1.57    3        33.4           9.09
    ##  4 EWR        4       0       4        32.2          10.0 
    ##  5 EWR        5       0.08    0.25     31.1           7.72
    ##  6 EWR        6       0.94    0.25     47.2           9.32
    ##  7 EWR        7       4.46    2        40.3          10.00
    ##  8 EWR        8       3.63    0.5      33.4          10.1 
    ##  9 EWR        9       2.4     0.5      36.8           9.14
    ## 10 EWR       10       1.72    0.25     43.7           8.90
    ## # ... with 83 more rows

    table(new_weather$origin)

    ## 
    ##  EWR  JFK  LGA 
    ## 8703 8706 8706

Construct a linear model to predict the performance of each
(airport,day) using the weather summaries and a “fixed effect” for each
airport. Display the summaries.

Repeat the above, but only for EWR. Obviously, exclude the fixed effect
for each airport.
