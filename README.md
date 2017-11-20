---
title: "Introduction to the istacr R-package"
author: "José Manuel Cazorla-Artiles, Christian González-Martel"
date: "2017-11-27"
---

# Introduction


The Canary Islands Statistics Institute (Instituto Canario de Estadítica, ISTAC)^[<http://www.gobiernodecanarias.org/istac/>] is the central organ of the autonomous statistical system and official research center of the Government of the Canary Islands, created and regulated by Law 1/1991 of 28 January, Statistics of the Autonomous Community of the Canary Islands (CAC), and among others, assigns the following functions:

* Providing statistical information: The ISTAC has among its objectives to provide, with technical and professional independence, statistical information of interest to the CAC, taking into account the fragmentation of the territory and its singularities and complying with the principles established in the Code of Good Practices of the European Statistics.

* Coordinate public statistical activity: The ISTAC is the body responsible for promoting, managing and coordinating the public statistical activity of the CAC, assuming the exercise of the statutory competence provided for in Article 30, paragraph 23, of the Statute of Autonomy of the Canary Islands . 

To help provide access to this rich source of information, ISTAC themselves, provide a well structured API^[<https://es.slideshare.net/ISTAC/guia-de-uso-api-de-acceso-a-istac-base>]. While this API is very useful for integration into web services and other high-level applications, it becomes quickly overwhelming for researchers who have neither the time nor the expertise to develop software to interface with the API. This leaves the researcher to rely on manual bulk downloads of spreadsheets of the data they are interested in. This too is can quickly become overwhelming, as the work is manual, time consuming, and not easily reproducible. The goal of the `istacr` R-package is to provide a bridge between these alternatives and allow researchers to focus on their research questions and not the question of accessing the data. The `istacr` R-package allows researchers to quickly search and download the data of their particular interest in a programmatic and reproducible fashion; this facilitates a seamless integration into their workflow and allows analysis to be quickly rerun on different areas of interest and with realtime access to the latest available data.

### Highlighted features of the `istacr` R-package: 

- Access to all data available in the API
- Support for searching and downloading data
- Ability to return `POSIXct` dates for easy integration into plotting and time-series analysis techniques
- Support for Most Recent Value queries
- Support for `grep` style searching for data descriptions and names

# Getting Started

The first step would be searching for the data you are interested in. `istac_search()` provides `grep` style searching of all available indicators from the ISTAC API and returns the indicator information that matches your query.


## Finding available data with `cache`

For performance and ease of use, a cached version of useful information is provided with the `istacr` R-package. This data is called `cache` and provides a snapshot of available islands, indicators, and other relevant information. `cache` is by default the the source from which `istac_search()` and `istac()` uses to find matching information. The structure of `cache` is as follows
```{r}
library(istacr)

str(cache, max.level = 1)
```

## Search available data with `wbsearch()`

`istac_search()` searches through the `cache` data frame to find indicators that match a search pattern. An example of the structure of this data frame is below
```{r, echo=FALSE, results='asis'}
knitr::kable(head(istacr::cache[4310:4311, ]))
```

By default the search is done over the `titulo` field and returns all the columns of the matching rows. The `ID` values are inputs into `istac()`, the function for downloading the data. To return the key columns `ID` and `titulo` for the `cache` data frame, you can set `extra = TRUE`.
```{r}
library(istacr)

busqueda <- istac_search(pattern = "parado")
head(busqueda)

```

Other fields can be searched by simply changing the `fields` parameter. For example
```{r}
library(istacr)

EPA_busqueda <- istac_search(pattern = "Encuesta de Población Activa", fields = "encuesta")
head(EPA_busqueda)

```

Regular expressions are also supported.
```{r}
library(istacr)

# 'pobreza' OR 'parados' OR 'trabajador'
popatr_busqueda <- istac_search(pattern = "pobreza|parados|trabajador")

head(popatr_busqueda)

```

## Downloading data with `istac()`

Once you have found the set of indicators that you would like to explore further, the next step is downloading the data with `istac()`. The following examples are meant to highlight the different ways in which `istac()` can be used and demonstrate the major optional parameters.

The default value for the `islas` parameter is a special value of `all` which as you might expect, returns data on the selected `ID` for every available country or region, if it is aviable.
```{r}
library(istacr)

pop_data <- istac(istac_table = "dem.pob.exp.res.40")

head(pop_data)
```

If you are interested in only some subset of islands you can pass along the specific island to the `island` parameter. The islands that can be passed to the `islas` parameter correspond to `all`,`Canarias`,`Lanzarote`,`Fuerteventura`,`Gran Canaria`,`Tenerife`,`La Gomera`,`La Palma` or `El Hierro`.
```{r}
library(istacr)

# Population, total
# country values: iso3c, iso2c, regionID, adminID, incomeID
pop_data <- istac(islas = c("Lanzarote","Fuerteventura"), istac_table = "dem.pob.exp.res.40")

head(pop_data)
```

### Using `POSIXct`

Setting the parameter `POSIXct = TRUE` gives you the posibility to work with dates. You can set a `startdate` and `enddate`.

```{r}
library(istacr)

# islas values: all,Canarias,Lanzarote,Fuerteventura,Gran Canaria,Tenerife,La Gomera,La Palma or El Hierro
pop_data <- istac(islas = c("Lanzarote","Fuerteventura"), istac_table = "dem.pob.exp.res.40")

head(pop_data)
```


### Using `POSIXct = TRUE`
The default format for the `Periodo` or `Años` column is not conducive to sorting or plotting, especially when downloading sub annual data, such as monthly or quarterly data. To address this, if `TRUE`, the `POSIXct` parameter adds the additional columns `fecha` and `periodicidad`. `fecha` converts the default date into a `POSIXct`. `periodicidad` denotes the time resolution that the date represents. This option requires the use of the package `lubridate (>= 1.5.0)`. If `POSIXct = TRUE` and `lubridate (>= 1.5.0)` is not available, a `warning` is produced and the option is ignored.

`startdate` and `enddate` must be in the format `YYYY`.
 
 

```{r}
library(istacr)

pop_data <- istac(istac_table = "dem.pob.exp.res.40", POSIXct = TRUE, startdate = 2010, enddate = 2016)

head(pop_data)
```

The `POSIXct = TRUE` option makes plotting and sorting dates much easier.
```{r, fig.height = 4, fig.width = 7.5}
library(istacr)
library(ggplot2)

pop_data <- istac(islas = "Canarias", istac_table = "dem.pob.exp.res.40", POSIXct = TRUE, startdate = 2010, enddate = 2016)


ggplot(pop_data, aes(x = fecha, y = valor, colour = Nacionalidades)) +
  geom_line(size = 1) +
  labs(title = "Población según sexos y nacionalidades. Islas de Canarias y años", x = "Fecha", y = "Habitantes") +
  theme(legend.position="bottom") +
  facet_wrap(~ Sexos)
```


### Using `mrv`
If you do not know the latest date an indicator you are interested in is available  you can use the `mrv` instead of `startdate` and `enddate`. `mrv` stands for most recent value and takes a `integer` corresponding to the number of most recent values you wish to return
```{r}
library(istacr)

pop_data <- istac(istac_table = 'dem.pob.exp.res.40', POSIXct = TRUE, mrv = 1)

head(pop_data)
```

You can increase this value and it will return no more than the `mrv` value. However, if `mrv` is greater than the number of available data it will return all data instead.

### Using `freq`
If the data has several granularity, you can select son eof them with the parameter `freq`. Possible values are:
`anual`,`semestral`,`trimestral`,`mensual`,`quincenal`,`semanal`. If the granularity selected is not aviable all granularity will be showed.

```{r}
library(istacr)

soc_data <- istac(istac_table = 'soc.sal.est.ser.1625', POSIXct = TRUE, freq = "semestral")

head(soc_data)
```



## `startdate`, `enddate`, `mrv`, `freq` with `POSIXct = FALSE`  
If you make a query with `istac()` and `POSIXct = FALSE` the `startdate`, `enddate`, `mrv`, `freq` are ignored and the function will launch a warning.

```{r}
library(istacr)

soc_data <- istac(istac_table = 'soc.sal.est.ser.1625', POSIXct = FALSE, freq = "semestral")

head(soc_data)
```
