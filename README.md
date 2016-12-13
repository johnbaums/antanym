[![Travis-CI Build Status](https://travis-ci.org/AustralianAntarcticDataCentre/antanym.svg?branch=master)](https://travis-ci.org/AustralianAntarcticDataCentre/antanym)
[![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/AustralianAntarcticDataCentre/antanym?branch=master&svg=true)](https://ci.appveyor.com/project/AustralianAntarcticDataCentre/antanym)
# antanym
This R package provides easy access to Antarctic geographic place name information. Currently it uses the Composite Gazetteer of Antarctica (but may be expanded to other sources, such an subantarctic gazetteers, at a later date).

The SCAR Composite Gazetteer of Antarctica (CGA) was begun in 1992 and consists of approximately 37,000 names corresponding to 19,000 distinct features. These place names have been submitted by the national names committees from 22 countries. Since 2008, Italy and Australia jointly have managed the CGA, the former taking care of the editing, the latter maintaining database and website. The SCAR Standing Committee on Antarctic Geographic Information (SCAGI) coordinates the project.

Because Antarctica does not fall under the sovereignty of any one nation, there is no single naming authority responsible for place names. In general, individual countries have administrative bodies that are responsible for their national policy on, and authorisation and use of, Antarctic names. The CGA includes the names of features south of 60° S, including terrestrial and undersea or under-ice. It is a compilation of all Antarctic names that have been submitted by representatives of national gazetteers, and so there may be multiple names associated with a given feature. Consider using the `an_preferred()` function for resolving a single name per feature.

For more information, see the [CGA home page](http://data.aad.gov.au/aadc/gaz/scar/).

References
----------

Composite Gazetteer of Antarctica, Scientific Committee on Antarctic Research. GCMD Metadata (http://gcmd.nasa.gov/records/SCAR_Gazetteer.html)



Installing
----------

``` r
install.packages("devtools")
library(devtools)
install_github("AustralianAntarcticDataCentre/antanym")
```

Usage
-----

``` r
library(antanym)
g <- an_read()

## islands within 20km of 100E, 66S
an_filter(an_near(g,c(100,-66),20),feature_type="Island")

## one name per feature
## names starting with "Sm", preferring the Polish name where there is one
an_preferred(an_filter(g,"^Sm"),origin_country="Poland")

## ask for suggested names to show on a given map
suggested <- an_suggest(g,map_extent=c(60,90,-70,-65),map_dimensions=c(80,80))

head(suggested,10) ## the 10 best names purely by score
an_thin(suggested,10) ## the 10 best names considering both score and spatial coverage


## similar calls, using dplyr
library(dplyr)
g %>% an_near(c(100,-66),20) %>% an_filter(feature_type="Island")
g %>% an_filter("^Sm") %>% an_preferred(origin_country="Poland")
```


Demos
-----

### [Simple leaflet app](https://australianantarcticdatacentre.github.io/antanym-demo/leaflet.html)

Source code:

``` r
library(antanym)
library(dplyr)
library(leaflet)
g <- an_read()

## find single name per feature, preferring United Kingdom
##  names where available, and only rows with valid locations
temp <- g %>% an_preferred("United Kingdom") %>%
  filter(!is.na(longitude) & !is.na(latitude))

## replace NAs with empty strings in narrative
temp$narrative[is.na(temp$narrative)] <- ""

## formatted popup HTML
popup <- sprintf("<h1>%s</h1><p><strong>Country of origin:</strong> %s<br />
  <strong>Longitude:</strong> %g<br /><strong>Latitude:</strong> %g<br />
  <a href=\"https://data.aad.gov.au/aadc/gaz/scar/display_name.cfm?gaz_id=%d\">
    Link to SCAR gazetteer</a></p>",temp$place_name,temp$country_name,
  temp$longitude,temp$latitude,temp$gaz_id)

m <- leaflet() %>%
  addProviderTiles("Esri.WorldImagery") %>%
  addMarkers(lng=temp$longitude,lat=temp$latitude,group="placenames",
    clusterOptions = markerClusterOptions(),popup=popup,
    label=temp$place_name,labelOptions=labelOptions(textOnly=TRUE))
```
