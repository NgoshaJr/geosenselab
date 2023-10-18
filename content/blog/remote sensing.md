---
title: "Download and Analyse Satellite Data with R"
date: 2023-04-01T16:09:23+02:00
draft: false
tags: [sensing, erddap, MODIS]
categories: [remote sensing]
---

This post shows the power and versatility of R packages for analyzing earth observation data, specifically in the context of studying sea surface temperature variabilities and chlorophyll-a in the southern part of the West Indian Ocean in Tanzania. By utilizing packages such as rerddap, tidyverse, and lubridate, researchers and analysts can efficiently access and manipulate large amounts of data, enabling them to uncover valuable insights and trends. These tools provide a streamlined workflow, allowing for a more effective and accurate analysis of complex datasets. Overall, this post serves as a reminder of the incredible potential that R packages have in advancing our understanding of the Earth's oceans and the critical role they play in the larger ecosystem.

## Load the required R packages

```{r}
library(rerddap)
library(lubridate)
library(tidyverse)
library(rmarkdown)
```

## Search and Explore Satellite data from ERDDAP Server

```{r}
sst_data = ed_search(query = 'MODIS SST')
sst_data
```

## Extract and Download SST data using Dataset ID

```{r}
info('jplMURSST41mday')
```

```{r}
<ERDDAP info> jplMURSST41mday 
 Base URL: https://upwell.pfeg.noaa.gov/erddap 
 Dataset Type: griddap 
 Dimensions (range):  
     time: (2002-06-16T00:00:00Z, 2023-03-16T00:00:00Z) 
     latitude: (-89.99, 89.99) 
     longitude: (-179.99, 180.0) 
 Variables:  
     mask: 
     nobs: 
     sst: 
         Units: degree_C 
```

## Download and arrange data in tabular form

```{r}
sst = griddap("jplMURSST41mday", 
                      latitude =  c(-11,-9),
                      longitude = c(39.75, 41),
                      time = c("2022-03-16", "2023-03-16"), 
                      fmt = "csv")
sst
```
## Convert data into readable format

```{r}
sst = sst %>% 
  mutate(time = lubridate::as_date(time))

## Visualize the head and tail of the dataset
sst %>% FSA::headtail() %>% as_tibble() 
```

## Plot

```{r}
mycolor <-c("#053061", "#2166ac", "#4393c3", "#92c5de", 
"#d1e5f0", "#f7f7f7", "#fddbc7", "#f4a582", "#d6604d",
"#b2182b", "#67001f")
sst %>%
  #filter(time <= "2021-03-16") %>%
  ggplot(aes(x = longitude, y = latitude)) +
  geom_tile(aes(fill = sst)) +
  scale_fill_gradientn(colours = mycolor, trans = scales::log10_trans()) +
  facet_wrap
  (~ time)+ggtitle('The sea surface temperature variability in Tanzania')
```

{{<image  caption="Fig.1: Sea Surface Temperature Valiabilities along the West Indian Ocean Waters" src="img/sst_wio.png" >}}

## Search and Explore Satellite data from ERDDAP Server

```{r}
chla_data = ed_search(query = 'MODIS AQU CHlorophyll global')
chla_data
```

## Extract and Download Chla data using Dataset ID

```{r}
info('erdMH1chlamday')
```

```{r}
<ERDDAP info> erdMH1chlamday 
 Base URL: https://upwell.pfeg.noaa.gov/erddap 
 Dataset Type: griddap 
 Dimensions (range):  
     time: (2003-01-16T00:00:00Z, 2022-05-16T00:00:00Z) 
     latitude: (-89.97917, 89.97916) 
     longitude: (-179.9792, 179.9792) 
 Variables:  
     chlorophyll: 
         Units: mg m-3 
         Units: degree_C 
```

## Download and arrange data in tabular form

```{r}
chla = griddap("erdMH1chlamday", 
                      latitude =  c(-11,-9),
                      longitude = c(39.75, 41),
                      time = c("2021-03-16", "2022-03-16"), 
                      fmt = "csv")
chla
```

## Convert data into readable format

```{r}
chla = chla %>% 
  mutate(time = lubridate::as_date(time))

## Visualize the head and tail of the dataset
chla %>% FSA::headtail() %>% as_tibble() 
```

## Plot

```{r}
# create a custom color palette
mycolor <-c('#FF6666', '#FFCC66', '#C6FF66', '#66FF66', '#66FFCC', '#66C2FF', '#6666FF', '#CC66FF', '#FF66CC')

chla %>%
  #filter(time <= "2021-03-16") %>%
  ggplot(aes(x = longitude, y = latitude)) +
  geom_tile(aes(fill = chlorophyll)) +
  scale_fill_gradientn(colors = mycolor, trans = log10_trans(), na.value = "transparent") +
  facet_wrap(~ time) +
  ggtitle('The Chlorophyll Concentrations in Tanzania')

```

{{<image  caption="Fig.2: Chlorophyll-a Concentration Valiabilities along the West Indian Ocean Waters" src="img/chla_wio.png" >}}
