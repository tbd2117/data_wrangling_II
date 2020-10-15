Reading\_data\_from\_the\_web
================
10/15/2020

### Scrape a table from url

I want the first table from [this
page](http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm)

read in the html

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"
drug_use_html = read_html(url)

drug_use_html
```

    ## {html_document}
    ## <html lang="en">
    ## [1] <head>\n<link rel="P3Pv1" href="http://www.samhsa.gov/w3c/p3p.xml">\n<tit ...
    ## [2] <body>\r\n\r\n<noscript>\r\n<p>Your browser's Javascript is off. Hyperlin ...

extract the table(s): focus on the first one

``` r
drug_use_html %>%
  html_nodes(css = "table")
```

    ## {xml_nodeset (15)}
    ##  [1] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ##  [2] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ##  [3] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ##  [4] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ##  [5] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ##  [6] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ##  [7] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ##  [8] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ##  [9] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ## [10] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ## [11] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ## [12] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ## [13] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ## [14] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...
    ## [15] <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100 ...

## focus on the first one

``` r
table_marj = 
  drug_use_html %>% 
  html_nodes(css = "table") %>% 
  first() %>%
  html_table() %>% 
  slice(-1) %>%  #this is to remove the firs row in every column (that contains text in this case)
  as_tibble() # make it nicer
```

## LEARNING ASSESSMENT

``` r
table_cost_living_nyc =
  read_html("https://www.bestplaces.net/cost_of_living/city/new_york/new_york") %>% 
  html_nodes(css = "table") %>% 
  .[[1]]%>% 
  html_table(header=TRUE)
```

### CSS selectors

IMDB url on starwars movies from
[here](https://www.imdb.com/list/ls070150896/)

``` r
swm_html = 
  read_html("https://www.imdb.com/list/ls070150896/")
```

For each element, I’ll use the CSS selector in html\_nodes() to extract
the relevant HTML code, and convert it to text. Then I can combine these
into a data frame.

``` r
title_vec = 
  swm_html %>%
  html_nodes(".lister-item-header a") %>%
  html_text()

gross_rev_vec = 
  swm_html %>%
  html_nodes(".text-small:nth-child(7) span:nth-child(5)") %>%
  html_text()

runtime_vec = 
  swm_html %>%
  html_nodes(".runtime") %>%
  html_text()

swm_df = 
  tibble(
    title = title_vec,
    rev = gross_rev_vec,
    runtime = runtime_vec)
```

### Using an API

``` r
nyc_water = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.csv") %>% 
  content("parsed")
```

    ## Parsed with column specification:
    ## cols(
    ##   year = col_double(),
    ##   new_york_city_population = col_double(),
    ##   nyc_consumption_million_gallons_per_day = col_double(),
    ##   per_capita_gallons_per_person_per_day = col_double()
    ## )

``` r
brfss_smart2010 = 
  GET("https://chronicdata.cdc.gov/resource/acme-vg9e.csv",
      query = list("$limit" = 5000)) %>% 
  content("parsed")
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character(),
    ##   year = col_double(),
    ##   sample_size = col_double(),
    ##   data_value = col_double(),
    ##   confidence_limit_low = col_double(),
    ##   confidence_limit_high = col_double(),
    ##   display_order = col_double(),
    ##   locationid = col_logical()
    ## )

    ## See spec(...) for full column specifications.

``` r
poke = 
  GET("http://pokeapi.co/api/v2/pokemon/1") %>%
  content()

poke$name
```

    ## [1] "bulbasaur"

To build a Pokemon dataset for analysis, you’d need to distill the data
returned from the API into a useful format; iterate across all pokemon;
and combine the results.

For both of the API examples we saw today, it wouldn’t be terrible to
just download the CSV, document where it came from carefully, and move
on. APIs are more helpful when the full dataset is complex and you only
need pieces, or when the data are updated regularly.
