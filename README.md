R Notebook
================

# Spectral variance across the western United States

## Setup Python environment

I’m using the environment provided by the Earth Data Science class.
<https://www.earthdatascience.org/workshops/setup-earth-analytics-python/>

I’m running python from Rstudio. This chunk tells rstudio how to find
that environment.

``` r
library(reticulate)
#library(usethis)

#use_python("/Users/ty/opt/miniconda3/bin/python")
#use_virtualenv("~/myenv")
#use_condaenv(condaenv = "r-nlp", conda = "/Users/ty/opt/miniconda3/envs/earth-analytics-python/opt/miniconda3")
#use_virtualenv("/Users/ty/opt/miniconda3/envs/earth-analytics-python/bin/python3")
py_config()
```

    ## python:         /Users/ty/opt/miniconda3/envs/earth-analytics-python/bin/python3.8
    ## libpython:      /Users/ty/opt/miniconda3/envs/earth-analytics-python/lib/libpython3.8.dylib
    ## pythonhome:     /Users/ty/opt/miniconda3/envs/earth-analytics-python:/Users/ty/opt/miniconda3/envs/earth-analytics-python
    ## version:        3.8.12 | packaged by conda-forge | (default, Oct 12 2021, 21:50:38)  [Clang 11.1.0 ]
    ## numpy:          /Users/ty/opt/miniconda3/envs/earth-analytics-python/lib/python3.8/site-packages/numpy
    ## numpy_version:  1.22.1
    ## 
    ## NOTE: Python version was forced by RETICULATE_PYTHON

``` r
#edit_r_environ()
```

Load python packages

``` python
import pandas as pd
import geopandas
import matplotlib.pyplot as plt
import numpy as np
import plotnine
```

    ## /Library/Frameworks/R.framework/Versions/4.1/Resources/library/reticulate/python/rpytools/loader.py:39: FutureWarning: pandas.Int64Index is deprecated and will be removed from pandas in a future version. Use pandas.Index with the appropriate dtype instead.

``` python
import contextily as cx
import requests
import json
```

## Filter metadata to decide what you’re going to try to download.

A small csv version of the neon metadate exists on the NEON website.
Download this file and load it into python.
<https://www.neonscience.org/resources/learning-hub/tutorials/neon-api-01-introduction-requests>

``` python
metadata = pd.read_csv('data/NEON_Field_Site_Metadata_20220224.csv')
print(metadata.field_site_id)
```

    ## 0     ABBY
    ## 1     ARIK
    ## 2     BARC
    ## 3     BARR
    ## 4     BART
    ##       ... 
    ## 76    WALK
    ## 77    WLOU
    ## 78    WOOD
    ## 79    WREF
    ## 80    YELL
    ## Name: field_site_id, Length: 81, dtype: object

## R code to translate into python code

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──

    ## ✓ ggplot2 3.3.5     ✓ purrr   0.3.4
    ## ✓ tibble  3.1.6     ✓ dplyr   1.0.8
    ## ✓ tidyr   1.2.0     ✓ stringr 1.4.0
    ## ✓ readr   2.0.2     ✓ forcats 0.5.1

    ## Warning: package 'tidyr' was built under R version 4.1.2

    ## Warning: package 'dplyr' was built under R version 4.1.2

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(sf)
```

    ## Warning: package 'sf' was built under R version 4.1.2

    ## Linking to GEOS 3.9.1, GDAL 3.4.0, PROJ 8.1.1; sf_use_s2() is TRUE

``` r
library(maps)
```

    ## 
    ## Attaching package: 'maps'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     map

Steps: 1) Read in the metadata csv 2) Spatially project those metadata
so we can do spatial filters 3) Calculate a bounding box around the
available NEON sites 4) Download a basemap and make sure it has the same
spatial projection as your data.

``` r
metadata_read = read.csv('data/NEON_Field_Site_Metadata_20220224.csv')

metatdata <- st_as_sf(metadata_read, coords = c("field_longitude", "field_latitude"), crs=4326)

bb <- st_bbox(metatdata)

usa <- st_as_sf(map_data("usa"), coords = c( "long", "lat"), crs=4326)
```

Plot the NEON sites

``` r
ggplot() +
  geom_sf(data = usa, col="white") +
  geom_sf(data=metatdata,aes( color=field_avg_number_of_green_days), size=5) +
  coord_sf(xlim = c(bb[1], bb[3]), ylim = c(bb[2], bb[4]))
```

![](prototyping-notebook_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

Filter your data to only include terrestrial sites

``` r
terrestrial_sites <- metatdata %>% 
  filter(field_site_type == "Core Terrestrial" | field_site_type == "Gradient Terrestrial" ) 

as.data.frame(terrestrial_sites)
```

    ##    field_domain_id field_site_id
    ## 1              D16          ABBY
    ## 2              D18          BARR
    ## 3              D01          BART
    ## 4              D02          BLAN
    ## 5              D19          BONA
    ## 6              D11          CLBJ
    ## 7              D10          CPER
    ## 8              D09          DCFS
    ## 9              D19          DEJU
    ## 10             D08          DELA
    ## 11             D03          DSNY
    ## 12             D07          GRSM
    ## 13             D04          GUAN
    ## 14             D01          HARV
    ## 15             D19          HEAL
    ## 16             D03          JERC
    ## 17             D14          JORN
    ## 18             D06          KONA
    ## 19             D06          KONZ
    ## 20             D04          LAJA
    ## 21             D08          LENO
    ## 22             D07          MLBS
    ## 23             D13          MOAB
    ## 24             D13          NIWO
    ## 25             D09          NOGP
    ## 26             D11          OAES
    ## 27             D15          ONAQ
    ## 28             D07          ORNL
    ## 29             D03          OSBS
    ## 30             D20          PUUM
    ## 31             D10          RMNP
    ## 32             D02          SCBI
    ## 33             D02          SERC
    ## 34             D17          SJER
    ## 35             D17          SOAP
    ## 36             D14          SRER
    ## 37             D05          STEI
    ## 38             D10          STER
    ## 39             D08          TALL
    ## 40             D17          TEAK
    ## 41             D18          TOOL
    ## 42             D05          TREE
    ## 43             D06          UKFS
    ## 44             D05          UNDE
    ## 45             D09          WOOD
    ## 46             D16          WREF
    ## 47             D12          YELL
    ##                                                field_site_name
    ## 1                                               Abby Road NEON
    ## 2                                               Utqiaġvik NEON
    ## 3                            Bartlett Experimental Forest NEON
    ## 4                                Blandy Experimental Farm NEON
    ## 5                 Caribou-Poker Creeks Research Watershed NEON
    ## 6                    Lyndon B. Johnson National Grassland NEON
    ## 7                       Central Plains Experimental Range NEON
    ## 8                                Dakota Coteau Field Site NEON
    ## 9                                          Delta Junction NEON
    ## 10                                              Dead Lake NEON
    ## 11                             Disney Wilderness Preserve NEON
    ## 12                    Great Smoky Mountains National Park NEON
    ## 13                                         Guanica Forest NEON
    ## 14                     Harvard Forest & Quabbin Watershed NEON
    ## 15                                                  Healy NEON
    ## 16                           The Jones Center At Ichauway NEON
    ## 17                             Jornada Experimental Range NEON
    ## 18                            Konza Prairie Agroecosystem NEON
    ## 19                       Konza Prairie Biological Station NEON
    ## 20                             Lajas Experimental Station NEON
    ## 21                                         Lenoir Landing NEON
    ## 22                       Mountain Lake Biological Station NEON
    ## 23                                                   Moab NEON
    ## 24                                            Niwot Ridge NEON
    ## 25              Northern Great Plains Research Laboratory NEON
    ## 26                   Marvin Klemme Range Research Station NEON
    ## 27                                                 Onaqui NEON
    ## 28                                              Oak Ridge NEON
    ## 29                      Ordway-Swisher Biological Station NEON
    ## 30                     Pu'u Maka'ala Natural Area Reserve NEON
    ## 31                                        Rocky Mountains NEON
    ## 32             Smithsonian Conservation Biology Institute NEON
    ## 33              Smithsonian Environmental Research Center NEON
    ## 34                         San Joaquin Experimental Range NEON
    ## 35                                        Soaproot Saddle NEON
    ## 36                          Santa Rita Experimental Range NEON
    ## 37                               Steigerwaldt-Chequamegon NEON
    ## 38                                         North Sterling NEON
    ## 39                              Talladega National Forest NEON
    ## 40                                        Lower Teakettle NEON
    ## 41                                   Toolik Field Station NEON
    ## 42                                              Treehaven NEON
    ## 43                     University of Kansas Field Station NEON
    ## 44 University of Notre Dame Environmental Research Center NEON
    ## 45                    Chase Lake National Wildlife Refuge NEON
    ## 46                         Wind River Experimental Forest NEON
    ## 47                              Yellowstone National Park NEON
    ##         field_site_type field_site_subtype field_colocated_site
    ## 1  Gradient Terrestrial                                        
    ## 2  Gradient Terrestrial                                        
    ## 3  Gradient Terrestrial                                        
    ## 4  Gradient Terrestrial                                    LEWI
    ## 5      Core Terrestrial                                    CARI
    ## 6      Core Terrestrial                                    PRIN
    ## 7      Core Terrestrial                                        
    ## 8  Gradient Terrestrial                                    PRLA
    ## 9  Gradient Terrestrial                                        
    ## 10 Gradient Terrestrial                                    BLWA
    ## 11 Gradient Terrestrial                                        
    ## 12 Gradient Terrestrial                                    LECO
    ## 13     Core Terrestrial                                        
    ## 14     Core Terrestrial                                    HOPB
    ## 15 Gradient Terrestrial                                        
    ## 16 Gradient Terrestrial                                    FLNT
    ## 17 Gradient Terrestrial                                        
    ## 18 Gradient Terrestrial                                    KING
    ## 19     Core Terrestrial                                        
    ## 20 Gradient Terrestrial                                        
    ## 21 Gradient Terrestrial                                    TOMB
    ## 22 Gradient Terrestrial                                        
    ## 23 Gradient Terrestrial                                        
    ## 24     Core Terrestrial                                    COMO
    ## 25 Gradient Terrestrial                                        
    ## 26 Gradient Terrestrial                                        
    ## 27     Core Terrestrial                                        
    ## 28     Core Terrestrial                                    WALK
    ## 29     Core Terrestrial                               BARC|SUGG
    ## 30     Core Terrestrial                                        
    ## 31 Gradient Terrestrial                                        
    ## 32     Core Terrestrial                                    POSE
    ## 33 Gradient Terrestrial                                        
    ## 34     Core Terrestrial                                        
    ## 35 Gradient Terrestrial                                        
    ## 36     Core Terrestrial                                        
    ## 37 Gradient Terrestrial                                        
    ## 38 Gradient Terrestrial                                        
    ## 39     Core Terrestrial                                    MAYF
    ## 40 Gradient Terrestrial                                        
    ## 41     Core Terrestrial                               TOOK|OKSR
    ## 42 Gradient Terrestrial                                        
    ## 43 Gradient Terrestrial                                        
    ## 44     Core Terrestrial                                        
    ## 45     Core Terrestrial                                    PRPO
    ## 46     Core Terrestrial                                        
    ## 47     Core Terrestrial                               BLDE|YELL
    ##                                                                            field_site_host
    ## 1                                               Washington Department of Natural Resources
    ## 2                                                            Ukpeagvik Inupiat Corporation
    ## 3                                             Northern Research Station, US Forest Service
    ## 4                                                      University of Virginia|Private Land
    ## 5                                                                     University of Alaska
    ## 6                                                                        US Forest Service
    ## 7                                                       USDA-Agricultural Research Service
    ## 8                                                                  North Dakota Land Trust
    ## 9                                                                Bureau of Land Management
    ## 10                                                              US Army Corps of Engineers
    ## 11                                                                  The Nature Conservancy
    ## 12                                                                   National Park Service
    ## 13 State Forest, Dept. of Natural and Environmental Resources, Commonwealth of Puerto Rico
    ## 14                Harvard University|Massachusetts DCR Division of Water Supply Protection
    ## 15                                                  Alaska Department of Natural Resources
    ## 16                                                        Robert Woodruff Foundation, Inc.
    ## 17                                                      USDA-Agricultural Research Service
    ## 18                                                                 Kansas State University
    ## 19                                         The Nature Conservancy, Kansas State University
    ## 20                                                               University of Puerto Rico
    ## 21                                 US Army Corps of Engineers|US Fish and Wildlife Service
    ## 22                  University of Virginia|George Washington and Jefferson National Forest
    ## 23                                                               Bureau of Land Management
    ## 24                                       US Forest Service, University of Colorado-Boulder
    ## 25                                                      USDA Agricultural Research Service
    ## 26                                                               Oklahoma State University
    ## 27                                                               Bureau of Land Management
    ## 28                                                                    Department of Energy
    ## 29                                                        University of Florida Foundation
    ## 30                                  Hawaii Forest Department of Land and Natural Resources
    ## 31                                                               Roosevelt National Forest
    ## 32                                                                 Smithsonian Institution
    ## 33                                                                 Smithsonian Institution
    ## 34                                                                       US Forest Service
    ## 35                                                                       US Forest Service
    ## 36                                                                   University of Arizona
    ## 37                                             Private|Chequamegon Nicolet National Forest
    ## 38                                                                                 Private
    ## 39                                                               Talladega National Forest
    ## 40                                                                       US Forest Service
    ## 41                                                               Bureau of Land Management
    ## 42                                                   University of Wisconsin-Stevens Point
    ## 43                                                                    University of Kansas
    ## 44                                                                University of Notre Dame
    ## 45                                                            US Fish and Wildlife Service
    ## 46                                   Pacific Northwest Research Station, US Forest Service
    ## 47                                                                   National Park Service
    ##                                                                                                                                                                                   field_site_url
    ## 1                                                                                                                                                                        https://www.dnr.wa.gov/
    ## 2                                                                                                      http://www.north-slope.org/departments/planning-community-services/applications-and-forms
    ## 3                                                                                                                                            https://www.nrs.fs.fed.us/ef/locations/nh/bartlett/
    ## 4                                                                                                                                                           http://blandy.virginia.edu/research|
    ## 5                                                                                                                                                                       https://www.uaf.edu/uaf/
    ## 6                                                                                                                https://www.fs.usda.gov/detail/texas/about-forest/districts/?cid=fswdev3_008440
    ## 7  https://www.ars.usda.gov/plains-area/fort-collins-co/center-for-agricultural-resources-research/rangeland-resources-systems-research/docs/rrsr/central-plains-experimental-research-location/
    ## 8                                                                                                                                                                       https://www.land.nd.gov/
    ## 9                                                                                                                                       https://www.blm.gov/office/eastern-interior-field-office
    ## 10                                                                                                                                                               https://www.sam.usace.army.mil/
    ## 11                                                                                       https://www.nature.org/en-us/get-involved/how-to-help/places-we-protect/the-disney-wilderness-preserve/
    ## 12                                                                                                                                                                    https://irma.nps.gov/rprs/
    ## 13                                                                                                                                                                                              
    ## 14                                                                              https://harvardforest.fas.harvard.edu/conducting-research|https://www.mass.gov/service-details/watershed-permits
    ## 15                                                                                                                                                                        http://dnr.alaska.gov/
    ## 16                                                                                                                                                             http://www.jonesctr.org/research/
    ## 17                                                                                                                                                                     https://jornada.nmsu.edu/
    ## 18                                                                                                                             https://kpbs.konza.k-state.edu/scientist-information/permits.html
    ## 19                                                                                                                             https://kpbs.konza.k-state.edu/scientist-information/permits.html
    ## 20                                                                                                                                                                                              
    ## 21                                                                                                                           https://www.sam.usace.army.mil/|https://www.fws.gov/refuge/choctaw/
    ## 22                                                                                                                                 https://mlbs.virginia.edu/resapp|https://www.fs.usda.gov/gwj/
    ## 23                                                                                                                                                  https://www.blm.gov/office/moab-field-office
    ## 24                                                                                                                                                                                              
    ## 25                                                                                                                                         https://www.ars.usda.gov/plains-area/mandan-nd/ngprl/
    ## 26                                                                                                                                                                      http://oaes.okstate.edu/
    ## 27                                                                                                                                             https://www.blm.gov/office/salt-lake-field-office
    ## 28                                                                                                                                                             https://www.ornl.gov/division/esd
    ## 29                                                                                                                                               https://ordway-swisher.ufl.edu/ResearchUse.aspx
    ## 30                                                                                                                                              https://dlnr.hawaii.gov/ecosystems/nars/permits/
    ## 31                                                                                                                                                                   https://www.fs.usda.gov/arp
    ## 32                                                                                                                                        https://www.si.edu/unit/conservation-biology-institute
    ## 33                                                                                                                                                                          https://serc.si.edu/
    ## 34                                                                                                                                                     https://www.fs.fed.us/psw/ef/san_joaquin/
    ## 35                                                                                                                                                               https://www.fs.usda.gov/sierra/
    ## 36                                                                              https://taac.arizona.edu/sites/taac.arizona.edu/files/SRER%20Range%20Use%20Agreement%202014%20revised%5B1%5D.pdf
    ## 37                                                                                                                                                                |https://www.fs.usda.gov/cnnf/
    ## 38                                                                                                                                                                                              
    ## 39                                                                                                             https://www.fs.usda.gov/detail/alabama/about-forest/districts/?cid=fsbdev3_002555
    ## 40                                                                                                                                                               https://www.fs.usda.gov/sierra/
    ## 41                                                                                                                                              https://toolik.alaska.edu/user_guide/permits.php
    ## 42                                                                                                                                https://www.uwsp.edu/cnr-ap/treehaven/Pages/Research-Form.aspx
    ## 43                                                                                                                                                             https://biosurvey.ku.edu/research
    ## 44                                                                                                                                     https://underc.nd.edu/underc-west/research-opportunities/
    ## 45                                                                                                                                                        https://www.fws.gov/refuge/chase_lake/
    ## 46                                                                                                                                                                  https://www.fs.usda.gov/pnw/
    ## 47                                                                                                                                                                    https://irma.nps.gov/rprs/
    ##        field_nonneon_research_allowed
    ## 1                             Limited
    ## 2                                 Yes
    ## 3                                 Yes
    ## 4  Yes (Tower and TIS) / Very Limited
    ## 5                                 Yes
    ## 6                        Very Limited
    ## 7                                 Yes
    ## 8                                 Yes
    ## 9                                 Yes
    ## 10                            Limited
    ## 11                                Yes
    ## 12                                Yes
    ## 13                            Limited
    ## 14                      Yes / Limited
    ## 15                                Yes
    ## 16                                Yes
    ## 17                                Yes
    ## 18                                Yes
    ## 19                                Yes
    ## 20                            Limited
    ## 21                       Limited /Yes
    ## 22                      Yes / Limited
    ## 23                                Yes
    ## 24                                Yes
    ## 25                                Yes
    ## 26                            Limited
    ## 27                            Limited
    ## 28                       Very Limited
    ## 29                                Yes
    ## 30                                Yes
    ## 31                                Yes
    ## 32                                Yes
    ## 33                                Yes
    ## 34                                Yes
    ## 35                                Yes
    ## 36                                Yes
    ## 37                 Very Limited / Yes
    ## 38                       Very Limited
    ## 39                                Yes
    ## 40                                Yes
    ## 41                                Yes
    ## 42                                Yes
    ## 43                                Yes
    ## 44                                Yes
    ## 45                                Yes
    ## 46                                Yes
    ## 47                            Limited
    ##                                                                                                                                                                                                                                                                                                                                                                                                                                                       field_access_details
    ## 1                                                                                                                                                                                                                                                                                                                                                                                    Reseachers should coordinate with the site manager and submit a site research permit.
    ## 2                                                                                                                                                                                                                                                                                                                                                                   Reseachers should coordinate with the site manager and submit appropriate site research-access permit.
    ## 3                                                                                                                                                                                                                                  Bartlett Experimental Forest has very limited staff and the USFS personnel are NOT available to support external researchers. Permitting requests for research activities should be submitted at least 6 months prior to planned start.
    ## 4                                                                                                                                                                                                                                                                                                                                                    Research activity at the NEON tower area via University of Virginia. / Very limited available via private land owner.
    ## 5                                                                                                                                                                                                                                                                                                    NEON site is colocated with Boreal LTER. Please allow extra lead time (3-6 months prior to planned start) for coordination requirements; coordinate with site manager
    ## 6                                                                                                                                                                                                                                                                                                                  Researchers should coordinate directly with the US Forest Service for permitting and approval. Site host is very limited to external research requests.
    ## 7                                                                                                                                                                                                                                                                                                                                                                                                                      Reseachers should coordinate with the site manager.
    ## 8                                                                                                                                                                                                                                                                                                                                                                                                                      Reseachers should coordinate with the site manager.
    ## 9                                                                                                                                                                                                                                                                                                                                                                                                                      Reseachers should coordinate with the site manager.
    ## 10                                                                                                                                                                                                                                                                                               This area is primarily intended to provide recreational opportunities and is also classified as a hunting area. As such, access is very limited but potentially possible.
    ## 11                                                                                                                                                                                                                                                                                                                                           New research requests may be restricted to limited portions of the preserve - coordination required with preserve management.
    ## 12                                                                                                                                                                                                                                                                                                                                        The National Park Service is open to additional research activities taking place in this area. Apply via IRMA Permitting portal.
    ## 13                                                                                                                                             It is highly recommended that prospective researchers make contact well in advance of any planned work (> 6 months) as there are multiple steps and a high level of coordination that must take place to gain permission and access to this area. All requests will be evaluated on a case-by-case basis by the site hosts.
    ## 14                                                                Field site managed by Harvard University - open to additional research via the Harvard Experimental Forest website. Many other colocated research projects within NEON area. / Field site located in the Quabbin Resevoir - Massachusetts DCR Division of Water Supply Protection . This property is very limited to additional research - researchers should take necessary coordinations with the DCR.
    ## 15                                                                                                                                                                                                                                                                                                                                                                                                                     Reseachers should coordinate with the site manager.
    ## 16                                                                                                                                                                                                                                                                                                                                                                                 This area is a very active research community and will required a site research permit.
    ## 17                                                                                                                                                                                            JORN encourages research activities that do not conflict with on-going activities. Please apply directly with site host and USDA for a site research permit.  All requests are reviewed for impacts to cultural resources, endangered species and other research activities.
    ## 18                                                                                                                                                                                                                       KSU - Konza Prairie Biological Station (KSU) welcomes and encourages research use that fits their mission and is compatible with their abilities as a host. Please plan on at more than two weeks advance notice to request and plan site access.
    ## 19                                                                                                                                                                                                  The Nature Conservancy and Konza Prairie Biological Station (KSU) welcomes and encourages research use that fits their mission and is compatible with their abilities as a host. Please plan on at more than two weeks advance notice to request and plan site access.
    ## 20                                                                                                                    The site host actively encourages research at this site, however all requests will be evaluated on a case-by-case basis. Please note that due to the high density of research taking place in this area, it is critical to coordinate closely with the site host to ensure no conflicts with ongoing research.; > 6months prior to planned schedule.
    ## 21                                                                                                    This area is primarily intended to provide recreational opportunities and is also classified as a hunting area. As such, access is very limited but potentially possible. / The primary purpose of the refuge is to provide wood duck brood habitat and serve as a protected wintering area for waterfowl. Researchers should coordinate directly with site manager.
    ## 22                                                                                                                                                                                                                                                                  Mountain Lake Biological Station is open to additional research activities taking place in this area. / Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 23                                                                                                                                                                                                                                                                                                                                                                                                                     Reseachers should coordinate with the site manager.
    ## 24                                                                                                                                                                                                                                                                                                                                                                                   Reseachers should coordinate with the site manager and submit a site research permit.
    ## 25                                                                                                                                                                                                                                                                                                                                        This site host welcomes and encourages additional research activities related to the USDA Agricultural Research Service mission.
    ## 26                                                                                                                                                                                                                                                                                                                                                                                                                     Reseachers should coordinate with the site manager.
    ## 27                                                                                                                                                                                                                                                                                                                                                                                   Reseachers should coordinate with the site manager and submit a site research permit.
    ## 28                                                                                                                                                                                                                                                                                                      There is currently no system in place to authorize access for additional research activities to this federally secure area. Coordinate directly with site manager.
    ## 29                                                                                                                                                                                                                                                                                                                                                                                 This area is a very active research community and will required a site research permit.
    ## 30                                                                                                                                                                                                                                                                                                                                                                      Researchers should coordinate with site manager and apply for a site research permit (3-6 months).
    ## 31                                                                                                                                                                                                                                                                                                                                                                          Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 32           Significant portions of the site are off-limits or have restricted access due to captive animal activities. Annual hunting activities may render this area completely inaccessible for short periods of time (1 or 2 days) over the course of the season. Most research activities require approval from a land-use committee; and stays longer than a month may require the researcher to apply for contractor credentials and submit to a background check.
    ## 33 Some areas of the site have pre-existing experiments and a high density of markers, equipment, sensors etc. Care should be taken to coordinate and avoid impacting on-going work at the site. All researchers must be accompanied while on-site by at least one individual with an approved Visiting Scientist Agreement. Annual hunting activities may render this area completely inaccessible for short periods of time (1 or 2 days) over the course of the season.
    ## 34                                                                                                                                                                                                                                                                                                                                                                          Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 35                                                                                                                                                                                                                                                                                                                                                                          Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 36                                                                                                                                                                                     SRER encourages research activities that do not conflict with on-going research. Researchers should apply directly for a site research permit. All proposed actions will be reviewed for potential impacts to cultural resources, endangered species and other research activities.
    ## 37                                                                                                                                                                                                                                                    Very limited available via private land owner. / NEON sampling area of this field site is located in Chequamegon Nicolet National Forest. Additional research may take place in this area with site host permission.
    ## 38                                                                                                                                                                                                                                                                                                                                                                                                                          Very limited available via private land owner.
    ## 39                                                                                                                                                                                                                                                                                                                                                                          Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 40                                                                                                                                                                                                                                                                                                                                                                          Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 41                                                                                                                                                                                                                                                                                                                                                          Reseachers should coordinate with the site manager and submit a site research permit via Toolik Field Station.
    ## 42                                                                                                                                                                                                                                                                                                                                  Treehaven encourages research projects. Research activities may be limited based on other uses of this property and on-going projects.
    ## 43                                                                                                                                                                                                                                                     This site host welcomes and encourages research use that fits their mission and is compatible with their abilities as a host. Please plan on at more than two weeks advance notice to request and plan site access.
    ## 44                                                                                                                                                                                                                                                       UNDERC encourages research; subject to evaluation based on potential for scientific advancement, impact on the site, and consistency and compatibility with other designated, non-research, uses of the property.
    ## 45                                                                                                                                                                                                                                                                                                                                                      This site host welcomes and encourages additional research activities related to the US Fish And Wildlife mission.
    ## 46                                                                                                                                                                                                                                                                                                                        Reseachers should coordinate with the site manager and submit a site research permit. Various on-going research projects to be aware of at site.
    ## 47                                                                                                                                                                                                                                                               The National Park Service is open to additional research activities taking place in this area. Apply via IRMA Permitting portal. NEON research area is limited to additional research due to sensitivity.
    ##    field_neon_field_operations_office field_geodetic_datum field_utm_northing
    ## 1          Domain 16 Support Facility                WGS84            5067870
    ## 2       Domain 18/19 Support Facility                WGS84            7910572
    ## 3          Domain 01 Support Facility                WGS84            4881512
    ## 4          Domain 02 Support Facility                WGS84            4327546
    ## 5       Domain 18/19 Support Facility                WGS84            7225713
    ## 6          Domain 11 Support Facility                WGS84            3696682
    ## 7       Domain 10/13 Support Facility                WGS84            4518311
    ## 8          Domain 09 Support Facility                WGS84            5223133
    ## 9       Domain 18/19 Support Facility                WGS84            7084367
    ## 10         Domain 08 Support Facility                WGS84            3524155
    ## 11         Domain 03 Support Facility                WGS84            3111131
    ## 12         Domain 07 Support Facility                WGS84            3952335
    ## 13         Domain 04 Support Facility                WGS84            1988112
    ## 14         Domain 01 Support Facility                WGS84            4713265
    ## 15      Domain 18/19 Support Facility                WGS84            7085048
    ## 16         Domain 03 Support Facility                WGS84            3453956
    ## 17         Domain 14 Support Facility                WGS84            3607409
    ## 18         Domain 06 Support Facility                WGS84            4331745
    ## 19         Domain 06 Support Facility                WGS84            4330787
    ## 20         Domain 04 Support Facility                WGS84            1993594
    ## 21         Domain 08 Support Facility                WGS84            3524827
    ## 22         Domain 07 Support Facility                WGS84            4136943
    ## 23      Domain 10/13 Support Facility                WGS84            4234597
    ## 24      Domain 10/13 Support Facility                WGS84            4433941
    ## 25         Domain 09 Support Facility                WGS84            5181356
    ## 26         Domain 11 Support Facility                WGS84            3918579
    ## 27         Domain 15 Support Facility                WGS84            4448480
    ## 28         Domain 07 Support Facility                WGS84            3983383
    ## 29         Domain 03 Support Facility                WGS84            3284767
    ## 30         Domain 20 Support Facility                WGS84            2163674
    ## 31      Domain 10/13 Support Facility                WGS84            4458524
    ## 32         Domain 02 Support Facility                WGS84            4308784
    ## 33         Domain 02 Support Facility                WGS84            4305735
    ## 34         Domain 17 Support Facility                WGS84            4110434
    ## 35         Domain 17 Support Facility                WGS84            4100967
    ## 36         Domain 14 Support Facility                WGS84            3530547
    ## 37         Domain 05 Support Facility                WGS84            5042744
    ## 38      Domain 10/13 Support Facility                WGS84            4480891
    ## 39         Domain 08 Support Facility                WGS84            3645864
    ## 40         Domain 17 Support Facility                WGS84            4097400
    ## 41      Domain 18/19 Support Facility                WGS84            7618428
    ## 42         Domain 05 Support Facility                WGS84            5041048
    ## 43         Domain 06 Support Facility                WGS84            4323550
    ## 44         Domain 05 Support Facility                WGS84            5123163
    ## 45         Domain 09 Support Facility                WGS84            5219442
    ## 46         Domain 16 Support Facility                WGS84            5074637
    ## 47         Domain 12 Support Facility                WGS84            4977886
    ##    field_utm_easting field_utm_zone    field_site_county field_site_state
    ## 1           552075.5            10N                Clark               WA
    ## 2           585246.1             4N          North Slope               AK
    ## 3           316812.2            19N              Carroll               NH
    ## 4           753379.5            17N               Clarke               VA
    ## 5           476436.7             6N Fairbanks North Star               AK
    ## 6           632982.2            14N                 Wise               TX
    ## 7           521454.8            13N                 Weld               CO
    ## 8           491923.5            14N             Stutsman               ND
    ## 9           561330.1             6N  Southeast Fairbanks               AK
    ## 10          389259.0            16N               Greene               AL
    ## 11          457161.1            17N              Osceola               FL
    ## 12          273599.9            17N               Sevier               TN
    ## 13          725706.1            19N    Guanica Municipio               PR
    ## 14          732183.2            18N            Worcester               MA
    ## 15          391276.6             6N               Denali               AK
    ## 16          741205.7            16N                Baker               GA
    ## 17          327074.6            13N             Dona Ana               NM
    ## 18          706389.5            14N                Riley               KS
    ## 19          710729.8            14N                Riley               KS
    ## 20          703591.1            19N      Lajas Municipio               PR
    ## 21          390139.6            16N              Choctaw               AL
    ## 22          542067.6            17N                Giles               VA
    ## 23          641031.5            12N             San Juan               UT
    ## 24          450328.3            13N              Boulder               CO
    ## 25          353761.6            14N               Morton               ND
    ## 26          494662.3            14N              Washita               OK
    ## 27          376339.7            12N               Tooele               UT
    ## 28          745052.4            16N             Anderson               TN
    ## 29          403886.4            17N               Putnam               FL
    ## 30          256868.7             5N               Hawaii               HI
    ## 31          453588.4            13N              Larimer               CO
    ## 32          748090.6            17N               Warren               VA
    ## 33          364703.7            18N         Anne Arundel               MD
    ## 34          257214.0            11N               Madera               CA
    ## 35          298792.7            11N               Fresno               CA
    ## 36          515553.9            12N                 Pima               AZ
    ## 37          297968.7            16N              Lincoln               WI
    ## 38          667085.0            13N                Logan               CO
    ## 39          463241.7            16N                 Bibb               AL
    ## 40          321515.7            11N               Fresno               CA
    ## 41          403757.1             6N          North Slope               AK
    ## 42          297965.3            16N              Lincoln               WI
    ## 43          310276.3            15N            Jefferson               KS
    ## 44          304367.0            16N              Gogebic               MI
    ## 45          481694.7            14N             Stutsman               ND
    ## 46          581417.8            10N             Skamania               WA
    ## 47          536352.2            12N                 Park               WY
    ##    field_site_country field_mean_elevation_m field_minimum_elevation_m
    ## 1                 USA                    365                        NA
    ## 2                 USA                      4                         0
    ## 3                 USA                    274                       230
    ## 4                 USA                    183                       119
    ## 5                 USA                    230                       212
    ## 6                 USA                    272                       258
    ## 7                 USA                   1654                      1601
    ## 8                 USA                    575                       494
    ## 9                 USA                    517                       444
    ## 10                USA                     25                        21
    ## 11                USA                     20                        15
    ## 12                USA                    575                       426
    ## 13                USA                    125                         8
    ## 14                USA                    348                       160
    ## 15                USA                    677                       588
    ## 16                USA                     47                        29
    ## 17                USA                   1324                      1316
    ## 18                USA                    323                       312
    ## 19                USA                    414                       333
    ## 20                USA                     16                        13
    ## 21                USA                     13                        NA
    ## 22                USA                   1170                       752
    ## 23                USA                   1799                      1722
    ## 24                USA                   3490                      2975
    ## 25                USA                    589                       517
    ## 26                USA                    519                       487
    ## 27                USA                   1662                      1599
    ## 28                USA                    344                       230
    ## 29                USA                     46                        22
    ## 30                USA                   1685                      1034
    ## 31                USA                   2742                      2454
    ## 32                USA                    352                       251
    ## 33                USA                     33                         0
    ## 34                USA                    400                       252
    ## 35                USA                   1210                      1044
    ## 36                USA                    997                       897
    ## 37                USA                    476                       466
    ## 38                USA                   1365                      1352
    ## 39                USA                    166                        63
    ## 40                USA                   2149                      2086
    ## 41                USA                    832                       704
    ## 42                USA                    467                       447
    ## 43                USA                    322                       272
    ## 44                USA                    521                       501
    ## 45                USA                    591                       569
    ## 46                USA                    351                       300
    ## 47                USA                   2133                      1847
    ##    field_maximum_elevation_m field_mean_annual_temperature_C
    ## 1                        708                            10.0
    ## 2                         13                           -12.0
    ## 3                        655                             6.2
    ## 4                        193                            12.1
    ## 5                        728                            -3.0
    ## 6                        343                            17.5
    ## 7                       1686                             8.6
    ## 8                        592                             4.9
    ## 9                        581                            -3.0
    ## 10                        41                            17.6
    ## 11                        22                            22.5
    ## 12                      1978                            13.1
    ## 13                       209                            23.0
    ## 14                       415                             7.4
    ## 15                       717                            -1.3
    ## 16                        55                            19.2
    ## 17                      1333                            15.7
    ## 18                       359                            12.7
    ## 19                       443                            12.4
    ## 20                        41                            25.0
    ## 21                        NA                            18.1
    ## 22                      1320                             8.8
    ## 23                      2039                            10.1
    ## 24                      3583                             0.3
    ## 25                       592                             5.9
    ## 26                       531                            15.5
    ## 27                      2061                             9.0
    ## 28                       356                            14.4
    ## 29                        49                            20.9
    ## 30                      1877                            12.7
    ## 31                      3042                             2.9
    ## 32                       571                            11.6
    ## 33                        43                            13.6
    ## 34                       502                            16.4
    ## 35                      1373                            13.4
    ## 36                      1461                            19.3
    ## 37                       536                             4.8
    ## 38                      1371                             9.7
    ## 39                       179                            17.2
    ## 40                      2734                             8.0
    ## 41                       980                            -9.0
    ## 42                       482                             4.8
    ## 43                       332                            12.7
    ## 44                       538                             4.3
    ## 45                       606                             4.9
    ## 46                      1008                             9.2
    ## 47                      2244                             3.4
    ##    field_mean_annual_precipitation_mm field_dominant_wind_direction
    ## 1                                2451                              
    ## 2                                 105                             E
    ## 3                                1325                             S
    ## 4                                 983                            NW
    ## 5                                 262                             W
    ## 6                                 926                             S
    ## 7                                 344                            NW
    ## 8                                 490                            NW
    ## 9                                 305                             E
    ## 10                               1372                            SW
    ## 11                               1216                             E
    ## 12                               1375                            NW
    ## 13                                840                             S
    ## 14                               1199                            NW
    ## 15                                385                            NW
    ## 16                               1308                            NE
    ## 17                                271                            SW
    ## 18                                850                           SSW
    ## 19                                870                           SSW
    ## 20                                830                           SSE
    ## 21                               1386                             N
    ## 22                               1227                           WNW
    ## 23                                319                           SSW
    ## 24                               1005                           WNW
    ## 25                                457                           NNE
    ## 26                                779                            SE
    ## 27                                288                            SW
    ## 28                               1340                           WNW
    ## 29                               1302                            NE
    ## 30                               2657                              
    ## 31                                731                              
    ## 32                               1126                           SSE
    ## 33                               1075                           NNW
    ## 34                                540                            NW
    ## 35                                900                            NW
    ## 36                                346                           ESE
    ## 37                                797                           WSW
    ## 38                                433                            NW
    ## 39                               1383                             N
    ## 40                               1223                            NW
    ## 41                                316                             S
    ## 42                                797                           WSW
    ## 43                                990                            SE
    ## 44                                802                           SSW
    ## 45                                494                            NW
    ## 46                               2225                            NW
    ## 47                                493                           SSE
    ##              field_mean_canopy_height_m
    ## 1  34m (in surrounding non-logged area)
    ## 2                                   0.3
    ## 3                                    23
    ## 4                                     1
    ## 5                                     8
    ## 6                                    13
    ## 7                                   0.4
    ## 8                                   0.4
    ## 9                                    10
    ## 10                                   30
    ## 11                                  1.5
    ## 12                                   30
    ## 13                                   10
    ## 14                                   26
    ## 15                                  0.3
    ## 16                                   27
    ## 17                                  0.4
    ## 18                                     
    ## 19                                  1.5
    ## 20                                  0.4
    ## 21                                   35
    ## 22                                   18
    ## 23                                  0.2
    ## 24                                  0.2
    ## 25                                  0.4
    ## 26                                    1
    ## 27                                  1.2
    ## 28                                   28
    ## 29                                   23
    ## 30                                   20
    ## 31                                   19
    ## 32                                   35
    ## 33                                   38
    ## 34                                   21
    ## 35                                   32
    ## 36                                    2
    ## 37 20m (in surounding non-logged area) 
    ## 38        varies based on crops planted
    ## 39                                   25
    ## 40                                   35
    ## 41                                  0.3
    ## 42                                   23
    ## 43                                   19
    ## 44                                   24
    ## 45                                    1
    ## 46                                   50
    ## 47                                   14
    ##                                        field_dominant_nlcd_classes
    ## 1                Evergreen Forest|Grassland/Herbaceous|Shrub/Scrub
    ## 2                                     Emergent Herbaceous Wetlands
    ## 3                   Deciduous Forest|Evergreen Forest|Mixed Forest
    ## 4                                     Deciduous Forest|Pasture/Hay
    ## 5    Deciduous Forest|Evergreen Forest|Mixed Forest|Woody Wetlands
    ## 6                            Deciduous Forest|Grassland/Herbaceous
    ## 7                                             Grassland/Herbaceous
    ## 8                                             Grassland/Herbaceous
    ## 9                      Evergreen Forest|Shrub/Scrub|Woody Wetlands
    ## 10                                 Evergreen Forest|Woody Wetlands
    ## 11                                      Pasture/Hay|Woody Wetlands
    ## 12                               Deciduous Forest|Evergreen Forest
    ## 13                                                Evergreen Forest
    ## 14   Deciduous Forest|Evergreen Forest|Mixed Forest|Woody Wetlands
    ## 15                        Dwarf Scrub|Evergreen Forest|Shrub/Scrub
    ## 16 Cultivated Crops|Deciduous Forest|Evergreen Forest|Mixed Forest
    ## 17                                                     Shrub/Scrub
    ## 18                                                Cultivated Crops
    ## 19                           Deciduous Forest|Grassland/Herbaceous
    ## 20               Cultivated Crops|Grassland/Herbaceous|Pasture/Hay
    ## 21                                 Deciduous Forest|Woody Wetlands
    ## 22                                                Deciduous Forest
    ## 23                                    Evergreen Forest|Shrub/Scrub
    ## 24                           Evergreen Forest|Grassland/Herbaceous
    ## 25                                            Grassland/Herbaceous
    ## 26                                Grassland/Herbaceous|Shrub/Scrub
    ## 27                                    Evergreen Forest|Shrub/Scrub
    ## 28                   Deciduous Forest|Evergreen Forest|Pasture/Hay
    ## 29    Emergent Herbaceous Wetlands|Evergreen Forest|Woody Wetlands
    ## 30                                                Evergreen Forest
    ## 31                                                Evergreen Forest
    ## 32                   Deciduous Forest|Evergreen Forest|Pasture/Hay
    ## 33                               Cultivated Crops|Deciduous Forest
    ## 34               Evergreen Forest|Grassland/Herbaceous|Shrub/Scrub
    ## 35                                    Evergreen Forest|Shrub/Scrub
    ## 36                                                     Shrub/Scrub
    ## 37                    Deciduous Forest|Mixed Forest|Woody Wetlands
    ## 38                                                Cultivated Crops
    ## 39                  Deciduous Forest|Evergreen Forest|Mixed Forest
    ## 40                                    Evergreen Forest|Shrub/Scrub
    ## 41                                         Dwarf Scrub|Shrub/Scrub
    ## 42   Deciduous Forest|Evergreen Forest|Mixed Forest|Woody Wetlands
    ## 43                                    Deciduous Forest|Pasture/Hay
    ## 44                    Deciduous Forest|Mixed Forest|Woody Wetlands
    ## 45               Emergent Herbaceous Wetlands|Grassland/Herbaceous
    ## 46                                                Evergreen Forest
    ## 47               Evergreen Forest|Grassland/Herbaceous|Shrub/Scrub
    ##                                                                                                                 field_domint_plant_species
    ## 1                                        Gaultheria shallon (salal)|Pseudotsuga menziesii (douglas fir)|Pteridium aquilinum (bracken fern)
    ## 2                        Carex aquatilis (water sedge)|Dupontia fisheri (Fisher's tundragrass)|Petasites frigidus (Arctic sweet coltsfoot)
    ## 3                                            Fagus grandifolia (American beech)|Tsuga canadensis (Eastern hemlock)|Acer rubrum (Red maple)
    ## 4                           Rhamnus davurica (dahurian buckthorn)|Lonicera maackii (amur honeysuckle)|Juglans nigra (eastern black walnut)
    ## 5                                            Picea mariana (black spruce)|Populus tremuloides (quaking aspen)|Alnus viridis (green alder) 
    ## 6                                                                          Quercus stellata (post oak)|Quercus marilandica (blackjack oak)
    ## 7                 Bouteloua gracilis (Blue grama)|Hesperostipa comata (Needle-and-thread grass)|Thelesperma filifolium (Stiff greenthread)
    ## 8                         Elaeagnus commutata (slilverberry)|Poa pratensis (Kentucky bluegrass)|Crataegus chrysocarpa (fireberry hawthorn)
    ## 9                      Picea mariana (black spruce)|Vaccinium vitis-idaea (lingonberry)|Betula glandulosa / Betula  (American dwarf birch)
    ## 10                                                         Quercus nigra (Water oak)|Celtis laevigata (Sugarberry)|Acer rubrum (Red maple)
    ## 11                          Andropogon virginicus (broomsedge bluestem)|Aristida beyrichia (wiregrass)|Axonopus furcatus (big carpetgrass)
    ## 12                                  Liriodendron tulipifera (tulip tree/yellow poplar)|Lindera benzoin (spicebush)|Acer rubrum (red maple)
    ## 13                                              Gymnanthes lucida (oysterwood)|Croton lucidus (firebush)|Eugenia foetida (Spanish stopper)
    ## 14                                      Osmunda cinnamomea (Cinmon fern)|Quercus rubra (Northern red oak)|Tsuga cadensis (Eastern hemlock)
    ## 15                      Betula glandulosa / Betula  (American dwarf birch)|Picea glauca (white spruce)|Ledum palustre (marsh Labrador tea)
    ## 16                                     Aristida beyrichiana (wiregrass)|Quercus falcata (southern red oak)|Pinus palustris (longleaf pine)
    ## 17                                Prosopis glandulosa (honey mesquite)|Yucca elata (soaptree yucca)|Bouteloua eriopoda (black grama grass)
    ## 18                                                                                              Agriculture field, crops vary year to year
    ## 19                      Cornus drummondii (Roughleaf Dogwood)|Schizachyrium scoparium (Little Bluestem)|Andropogon gerardii (Big Bluestem)
    ## 20                                  Pithecellobium dulce (pinzán)|Cynodon nlemfuensis (African bermudagrass)|Urochloa maxima (guineagrass)
    ## 21                                      Liquidambar styraciflua (sweetgum)|Carpinus carolinia (American hornbeam)|Ilex decidua (possumhaw)
    ## 22                                                       Acer rubrum (red maple)|Quercus alba (northern red oak)|Quercus rubra (white oak)
    ## 23                                  Atriplex canescens (fourwing saltbush)|Ephedra viridis (Mormon tea)|Bouteloua gracilis (blue grama)  .
    ## 24                                     Abies lasiocarpa (subapline fir)|Picea engelmannii (Engelmann spruce)|Vaccinium sp. (blueberry sp.)
    ## 25               Elaeagnus angustifolia (Russian olive)|Poa pratensis (Kentucky bluegrass)|Symphoricarpos occidentalis (Western snowberry)
    ## 26                           Bouteloua dactyloides (buffalo grass)|Bouteloua gracilis (blue grama)|Bouteloua curtipendula (sideoats grama)
    ## 27                      Artemisia tridentata (big sagebrush)|Ceratocephala testiculata (curveseed butterwort)|Bromus tectorum (cheatgrass)
    ## 28                                                         Acer rubrum (red maple)|Nyssa sylvatica (sour gum)|Quercus monta (chestnut oak)
    ## 29                                       Quercus laevis (turkey oak)|Pinus palustris (longleaf pine)|Aristida beyrichia (Beyrich threeawn)
    ## 30                                              Metrosideros polymorpha (ʻŌhiʻa)|Cibotium glaucum (hapu‘u)|Styphelia tameiameiae (pūkiawe)
    ## 31                                    Pinus contorta (Lodgepole pine)|Abies lasiocarpa (Subalpine fir)|Pseudotsuga menziesii (Douglas-fir)
    ## 32                         Liriodendron tulipifera (tulip tree)|Microstegium vimineum (Japanese stiltgrass)|Fraxinus americana (white ash)
    ## 33                   Liriodendron tulipifera (tulip poplar)|Fagus grandifolia (American Beech)|Liquidambar styraciflua (American sweetgum)
    ## 34                                           Quercus douglasii (blue oak)|Erodium botrys (broadleaf filaree)|Bromus diandrus (great brome)
    ## 35 Arctostaphylos viscida ssp. Mariposa (mariposa manzanita)|Chamaebatia foliolosa (mountain misery)|Quercus chrysolepis (canyon live oak)
    ## 36                               Larrea tridentata (creosote bush)|Prosopis veluti (velvet mesquite)|Parkinsonia florida (blue palo verde)
    ## 37                                                 Populus tremuloides (quaking aspen)|Abies balsamea (balsam fir)|Acer rubrum (red maple)
    ## 38                                                                                              Agriculture field, crops vary year to year
    ## 39                                  Quercus montana (Chestnut oak)|Liriodendron tulipifera (tulip tree)|Cornus florida (flowering dogwood)
    ## 40                                                    Abies magnifica (red fir)|Abies concolor (white fir)|Pinus contorta (lodgepole pine)
    ## 41                                  Eriophorum vaginatum (tussock cottongrass)|Carex sp. (true sedges)|Vaccinium vitis-idaea (lingonberry)
    ## 42                                                          Acer saccharum (sugar maple)|Acer rubrum (red maple)|Alnus incana (gray alder)
    ## 43                           Fraxinus americana (white ash)|Symphoricarpos orbiculatus (coralberry)|Celtis occidentalis (common hackberry)
    ## 44                                                        Acer saccharum (sugar maple)|Abies balsamea (balsam fir)|Acer rubrum (red maple)
    ## 45                                            Poa pratensis (Kentucky bluegrass)|Elymus repens (couch grass)|Bromus inermis (smooth brome)
    ## 46                                     Gaultheria shallon (salal)|Psuedotsuga menziesii (douglas fir)|Tsuga heterophylla (western hemlock)
    ## 47                                Artemisia tridentata (big sagebrush)|Pseudotsuga menziesii (douglas fir)|Pinus contorta (lodgepole pine)
    ##    field_usgs_huc field_watershed_name field_watershed_size_km2
    ## 1                                                            NA
    ## 2                                                            NA
    ## 3                                                            NA
    ## 4                                                            NA
    ## 5                                                            NA
    ## 6                                                            NA
    ## 7                                                            NA
    ## 8                                                            NA
    ## 9                                                            NA
    ## 10                                                           NA
    ## 11                                                           NA
    ## 12                                                           NA
    ## 13                                                           NA
    ## 14                                                           NA
    ## 15                                                           NA
    ## 16                                                           NA
    ## 17                                                           NA
    ## 18                                                           NA
    ## 19                                                           NA
    ## 20                                                           NA
    ## 21                                                           NA
    ## 22                                                           NA
    ## 23                                                           NA
    ## 24                                                           NA
    ## 25                                                           NA
    ## 26                                                           NA
    ## 27                                                           NA
    ## 28                                                           NA
    ## 29                                                           NA
    ## 30                                                           NA
    ## 31                                                           NA
    ## 32                                                           NA
    ## 33                                                           NA
    ## 34                                                           NA
    ## 35                                                           NA
    ## 36                                                           NA
    ## 37                                                           NA
    ## 38                                                           NA
    ## 39                                                           NA
    ## 40                                                           NA
    ## 41                                                           NA
    ## 42                                                           NA
    ## 43                                                           NA
    ## 44                                                           NA
    ## 45                                                           NA
    ## 46                                                           NA
    ## 47                                                           NA
    ##    field_lake_depth_mean_m field_lake_depth_max_m field_tower_height_m
    ## 1                       NA                     NA                   19
    ## 2                       NA                     NA                    9
    ## 3                       NA                     NA                   35
    ## 4                       NA                     NA                    8
    ## 5                       NA                     NA                   19
    ## 6                       NA                     NA                   22
    ## 7                       NA                     NA                    9
    ## 8                       NA                     NA                    8
    ## 9                       NA                     NA                   22
    ## 10                      NA                     NA                   42
    ## 11                      NA                     NA                    8
    ## 12                      NA                     NA                   45
    ## 13                      NA                     NA                   23
    ## 14                      NA                     NA                   39
    ## 15                      NA                     NA                    9
    ## 16                      NA                     NA                   42
    ## 17                      NA                     NA                    8
    ## 18                      NA                     NA                    8
    ## 19                      NA                     NA                    8
    ## 20                      NA                     NA                    8
    ## 21                      NA                     NA                   47
    ## 22                      NA                     NA                   29
    ## 23                      NA                     NA                    8
    ## 24                      NA                     NA                    8
    ## 25                      NA                     NA                    8
    ## 26                      NA                     NA                    8
    ## 27                      NA                     NA                    8
    ## 28                      NA                     NA                   39
    ## 29                      NA                     NA                   35
    ## 30                      NA                     NA                   32
    ## 31                      NA                     NA                   25
    ## 32                      NA                     NA                   52
    ## 33                      NA                     NA                   62
    ## 34                      NA                     NA                   39
    ## 35                      NA                     NA                   52
    ## 36                      NA                     NA                    8
    ## 37                      NA                     NA                   22
    ## 38                      NA                     NA                    8
    ## 39                      NA                     NA                   35
    ## 40                      NA                     NA                   59
    ## 41                      NA                     NA                    9
    ## 42                      NA                     NA                   36
    ## 43                      NA                     NA                   35
    ## 44                      NA                     NA                   39
    ## 45                      NA                     NA                    8
    ## 46                      NA                     NA                   74
    ## 47                      NA                     NA                   18
    ##          field_usgs_geology_unit
    ## 1                      WAEOv1b;0
    ## 2                 A003.BW002_114
    ## 3                       NHJc1b;0
    ## 4                      VAOCAco;0
    ## 5  A003.LG002_114|E191.LG002_930
    ## 6                         TXKa;0
    ## 7                         COKl;0
    ## 8                        NDQct;0
    ## 9                 A003.MH002_140
    ## 10                      ALHalt;0
    ## 11                      FLPSHu;0
    ## 12                     TNpCAs;15
    ## 13                           Tjd
    ## 14               MADht;0|MAOps;0
    ## 15                B006.HE002_570
    ## 16                       GAEOo;1
    ## 17                       NMQTs;0
    ## 18                      KSQal3;0
    ## 19                        KSPc;0
    ## 20                            Qa
    ## 21                      ALHalt;0
    ## 22                       VADSz;0
    ## 23                        UTQe;0
    ## 24                        COXb;0
    ## 25                       NDQol;0
    ## 26                       OKPdy;0
    ## 27                        UTQa;0
    ## 28            TNOnc;10|TNCAcr;10
    ## 29                       FLPOc;0
    ## 30                       Qk4|Qk2
    ## 31                        COYg;0
    ## 32                      VACAZc;0
    ## 33                        MDTn;5
    ## 34                     CAgrMZ3;0
    ## 35                     CAgrMZ3;0
    ## 36                         AZQ;0
    ## 37               WIXmi;0|WIXmv;0
    ## 38                        COTo;0
    ## 39                ALKg;1|ALKck;1
    ## 40                        CAQg;0
    ## 41                A003.PS002_400
    ## 42                       WIXmv;0
    ## 43                       KSQgd;0
    ## 44                       MIAgn;0
    ## 45                      NDQcrh;0
    ## 46                        WAQv;0
    ## 47                 WYQb;0|WYTs;0
    ##                                                                         field_megapit_soil_family
    ## 1                                                                                                
    ## 2                                                                                                
    ## 3                                               Coarse-loamy, isotic, frigid.  Aquic Haplorthods.
    ## 4                                                Fine, mixed, subactive, mesic. Ultic Hapludalfs.
    ## 5                                Coarse-silty, mixed, superactive, subgelic.  Typic Histoturbels.
    ## 6                                                                                                
    ## 7                                    Fine, loamy, mixed, superactive, mesic.  Aridic Argiustolls.
    ## 8                                     Fine, loamy, mixed, superactive, frigid. Typic Haplustolls.
    ## 9                                          Coarse, loamy, mixed, superactive. Typic Haplocryepts.
    ## 10                                            Fine, mixed, semiactive, thermic. Aquic Paleudults.
    ## 11                                               Sandy, siliceous, hyperthermic.  Aeric Alaquods.
    ## 12                                              Loamy, skeletal, isotic, mesic.  Typic Humudepts.
    ## 13                                Coarse, loamy, carbonatic, isohyperthermic. Typic Haplocalcids.
    ## 14 Coarse, loamy over sandy or sandy, skeletal, mixed, semiactive, frigid.  Oxyaquic Dystrudepts.
    ## 15                                                                                               
    ## 16                                               Loamy, kaolinitic, thermic.  Arenic Kandiudults.
    ## 17                               Coarse, loamy, mixed, superactive, thermicc. Typic Petrocalcids.
    ## 18                                             Fine, smectitic, mesic.  Pachic Vertic Argiudolls.
    ## 19                                            Fine, smectitic, mesic. Pachic Udertic Argiustolls.
    ## 20                                 Fine, mixed, superactive, isohyperthermic.  Sodic Haplusterts.
    ## 21                                        Fine, mixed, active, acid, thermic.  Vertic Epiaquepts.
    ## 22                                                 The dominant wind direction is west-northwest.
    ## 23                                  Coarse, loamy, mixed, superactive, mesic. Ustic Haplocalcids.
    ## 24                                         Coarse, loamy, mixed, superactive. Typic Haplocryolls.
    ## 25                                    Fine, loamy, mixed, superactive, frigid. Typic Argiustolls.
    ## 26                                                                                               
    ## 27                                    Fine, loamy, mixed, superactive, mesic. Xeric Haplocalcids.
    ## 28                                                   Fine, kaolinitic, thermic. Typic Paleudults.
    ## 29                                                                                               
    ## 30                                                                                               
    ## 31                                       Loamy, skeletal, mixed, superactive. Ustic Haplocryolls.
    ## 32                                       Loamy, skeletal, mixed, active, mesic. Ultic Hapludalfs.
    ## 33                                           Fine, loamy, mixed, active, mesic. Aquic Hapludults.
    ## 34                                                                                               
    ## 35                                     Fine, loamy, mixed, semiactive, mesic. Ultic Haploxeralfs.
    ## 36                                                                                               
    ## 37                                    Coarse, loamy, mixed, superactive, frigid. Alfic Epiaquods.
    ## 38                                    Fine, silty, mixed, superactive, mesic. Pachic Argiustolls.
    ## 39                                  Fine, loamy, siliceous, subactive, thermic. Typic Hapludults.
    ## 40                                                                                               
    ## 41                                                                                               
    ## 42                                  Coarse, loamy, mixed, superactive, frigid. Alfic Haplorthods.
    ## 43                                                     Fine, smectitic, mesic. Pachic Argiudolls.
    ## 44                                  Coarse, loamy, mixed, superactive, frigid. Argic Fragiaquods.
    ## 45   Coarse, loamy, over-sandy or sandy, skeletal, mixed, superactive, frigid. Typic Haplustolls.
    ## 46                                                     Medial, amorphic, mesic. Typic Hapludands.
    ## 47                                                                                               
    ##           field_soil_subgroup field_avg_number_of_green_days
    ## 1             Andic Humudepts                            190
    ## 2          Typic Histoturbels                             45
    ## 3           Aquic Haplorthods                            180
    ## 4            Ultic Hapludalfs                            235
    ## 5          Typic Histoturbels                            115
    ## 6            Udic Paleustalfs                            260
    ## 7          Aridic Argiustolls                            260
    ## 8           Typic Haplustolls                            170
    ## 9          Typic Haplocryepts                            120
    ## 10           Aquic Paleudults                            270
    ## 11             Aeric Alaquods                            260
    ## 12            Typic Humudepts                            220
    ## 13         Typic Haplocalcids                            365
    ## 14       Oxyaquic Dystrudepts                            210
    ## 15         Lithic Haplustepts                            110
    ## 16         Arenic Kandiudults                            220
    ## 17         Typic Petrocalcids                            260
    ## 18   Pachic Vertic Argiudolls                            210
    ## 19 Pachic Udertic Argiustolls                            210
    ## 20          Sodic Haplusterts                            365
    ## 21          Vertic Epiaquepts                            265
    ## 22                Fluvaquents                            200
    ## 23         Ustic Haplocalcids                            215
    ## 24         Typic Haplocryolls                            130
    ## 25          Typic Argiustolls                            175
    ## 26         Lithic Haplustepts                            240
    ## 27         Xeric Haplocalcids                            205
    ## 28           Typic Paleudults                            225
    ## 29    Typic Quartzipsam-ments                            245
    ## 30           Typic Hapludands                            365
    ## 31         Ustic Haplocryolls                            165
    ## 32           Ultic Hapludalfs                            245
    ## 33           Aquic Hapludults                            245
    ## 34    Psammentic Haploxerolls                            250
    ## 35         Ultic Haploxeralfs                            200
    ## 36        Typic Torrifluvents                            247
    ## 37            Alfic Epiaquods                            170
    ## 38         Pachic Argiustolls                            180
    ## 39           Typic Hapludults                            255
    ## 40         Pachic Humixerepts                            180
    ## 41         Typic Histoturbels                             80
    ## 42          Alfic Haplorthods                            170
    ## 43          Pachic Argiudolls                            255
    ## 44          Argic Fragiaquods                            165
    ## 45          Typic Haplustolls                            170
    ## 46           Typic Hapludands                            175
    ## 47         Pachic Argiustolls                            144
    ##    field_avg_grean_increase_doy field_avg_green_max_doy
    ## 1                           110                     165
    ## 2                           175                     195
    ## 3                           120                     170
    ## 4                            75                     150
    ## 5                           135                     180
    ## 6                        60|215                 135|230
    ## 7                            90                     165
    ## 8                           120                     180
    ## 9                           130                     170
    ## 10                           60                     135
    ## 11                           60                     140
    ## 12                           90                     155
    ## 13                                                     
    ## 14                          110                     160
    ## 15                          135                     180
    ## 16                           90                     175
    ## 17                           80                     185
    ## 18                           90                     160
    ## 19                           90                     160
    ## 20                                                     
    ## 21                           70                     145
    ## 22                          110                     160
    ## 23                           85                     165
    ## 24                          140                     190
    ## 25                          115                     170
    ## 26                       70|233                 135|270
    ## 27                           75                     130
    ## 28                           90                     140
    ## 29                           70                     150
    ## 30                                                     
    ## 31                          120                     180
    ## 32                           85                     150
    ## 33                           80                     155
    ## 34                          270                      65
    ## 35                           90                     155
    ## 36                       87|186                 139|215
    ## 37                          120                     165
    ## 38                           90                     150
    ## 39                           75                     135
    ## 40                          120                     180
    ## 41                          160                     185
    ## 42                          120                     165
    ## 43                           75                     160
    ## 44                          120                     170
    ## 45                          120                     180
    ## 46                          115                     165
    ## 47                          125                     169
    ##    field_avg_green_decrease_doy field_avg_green_min_doy
    ## 1                           205                     300
    ## 2                           210                     220
    ## 3                           220                     300
    ## 4                           210                     310
    ## 5                           210                     250
    ## 6                       175|265                 295|320
    ## 7                           210                     350
    ## 8                           205                     290
    ## 9                           210                     250
    ## 10                          205                     330
    ## 11                          190                     320
    ## 12                          215                     310
    ## 13                                                     
    ## 14                          220                     300
    ## 15                          210                     245
    ## 16                          220                     310
    ## 17                          245                     340
    ## 18                          210                     300
    ## 19                          210                     300
    ## 20                                                     
    ## 21                          200                     335
    ## 22                          220                     310
    ## 23                          225                     300
    ## 24                          220                     270
    ## 25                          200                     290
    ## 26                      165|290                 225|310
    ## 27                          170                     280
    ## 28                          210                     315
    ## 29                          190                     315
    ## 30                                                     
    ## 31                          210                     285
    ## 32                          220                     320
    ## 33                          220                     325
    ## 34                           95                     155
    ## 35                          185                     290
    ## 36                      189|259                 219|334
    ## 37                          215                     290
    ## 38                          190                     270
    ## 39                          195                     330
    ## 40                          205                     300
    ## 41                          205                     240
    ## 42                          215                     290
    ## 43                          210                     330
    ## 44                          215                     285
    ## 45                          210                     290
    ## 46                          210                     290
    ## 47                          193                     269
    ##                                                                                                                                                   field_phenocams
    ## 1  [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D16.ABBY.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D16.ABBY.DP1.00042)
    ## 2  [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D18.BARR.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D18.BARR.DP1.00042)
    ## 3  [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D01.BART.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D01.BART.DP1.00042)
    ## 4  [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D02.BLAN.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D02.BLAN.DP1.00042)
    ## 5  [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D19.BONA.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D19.BONA.DP1.00042)
    ## 6  [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D11.CLBJ.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D11.CLBJ.DP1.00042)
    ## 7  [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.CPER.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.CPER.DP1.00042)
    ## 8  [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D09.DCFS.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D09.DCFS.DP1.00042)
    ## 9  [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D19.DEJU.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D19.DEJU.DP1.00042)
    ## 10 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D08.DELA.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D08.DELA.DP1.00042)
    ## 11 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D03.DSNY.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D03.DSNY.DP1.00042)
    ## 12 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D07.GRSM.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D07.GRSM.DP1.00042)
    ## 13 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D04.GUAN.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D04.GUAN.DP1.00042)
    ## 14 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D01.HARV.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D01.HARV.DP1.00042)
    ## 15 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D19.HEAL.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D19.HEAL.DP1.00042)
    ## 16 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D03.JERC.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D03.JERC.DP1.00042)
    ## 17 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D14.JORN.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D14.JORN.DP1.00042)
    ## 18 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D06.KONA.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D06.KONA.DP1.00042)
    ## 19 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D06.KONZ.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D06.KONZ.DP1.00042)
    ## 20 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D04.LAJA.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D04.LAJA.DP1.00042)
    ## 21 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D08.LENO.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D08.LENO.DP1.00042)
    ## 22 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D07.MLBS.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D07.MLBS.DP1.00042)
    ## 23 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D13.MOAB.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D13.MOAB.DP1.00042)
    ## 24 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D13.NIWO.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D13.NIWO.DP1.00042)
    ## 25 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D09.NOGP.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D09.NOGP.DP1.00042)
    ## 26 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D11.OAES.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D11.OAES.DP1.00042)
    ## 27 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D15.ONAQ.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D15.ONAQ.DP1.00042)
    ## 28 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D07.ORNL.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D07.ORNL.DP1.00042)
    ## 29 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D03.OSBS.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D03.OSBS.DP1.00042)
    ## 30 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D20.PUUM.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D20.PUUM.DP1.00042)
    ## 31 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.RMNP.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.RMNP.DP1.00042)
    ## 32 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D02.SCBI.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D02.SCBI.DP1.00042)
    ## 33 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D02.SERC.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D02.SERC.DP1.00042)
    ## 34 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.SJER.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.SJER.DP1.00042)
    ## 35 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.SOAP.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.SOAP.DP1.00042)
    ## 36 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D14.SRER.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D14.SRER.DP1.00042)
    ## 37 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D05.STEI.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D05.STEI.DP1.00042)
    ## 38 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.STER.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.STER.DP1.00042)
    ## 39 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D08.TALL.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D08.TALL.DP1.00042)
    ## 40 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.TEAK.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.TEAK.DP1.00042)
    ## 41 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D18.TOOL.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D18.TOOL.DP1.00042)
    ## 42 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D05.TREE.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D05.TREE.DP1.00042)
    ## 43 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D06.UKFS.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D06.UKFS.DP1.00042)
    ## 44 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D05.UNDE.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D05.UNDE.DP1.00042)
    ## 45 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D09.WOOD.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D09.WOOD.DP1.00042)
    ## 46 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D16.WREF.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D16.WREF.DP1.00042)
    ## 47 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D12.YELL.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D12.YELL.DP1.00042)
    ##    field_number_tower_levels                   geometry
    ## 1                          5 POINT (-122.3303 45.76244)
    ## 2                          4 POINT (-156.6194 71.28241)
    ## 3                          6 POINT (-71.28737 44.06389)
    ## 4                          4  POINT (-78.04179 39.0337)
    ## 5                          5 POINT (-147.5026 65.15401)
    ## 6                          5    POINT (-97.57 33.40123)
    ## 7                          4 POINT (-104.7456 40.81554)
    ## 8                          4 POINT (-99.10656 47.16165)
    ## 9                          5 POINT (-145.7514 63.88112)
    ## 10                         6 POINT (-87.80388 32.54173)
    ## 11                         4 POINT (-81.43619 28.12505)
    ## 12                         6 POINT (-83.50195 35.68896)
    ## 13                         5  POINT (-66.8687 17.96955)
    ## 14                         6 POINT (-72.17265 42.53691)
    ## 15                         4  POINT (-149.2133 63.8758)
    ## 16                         6 POINT (-84.46862 31.19484)
    ## 17                         4 POINT (-106.8425 32.59069)
    ## 18                         4 POINT (-96.61293 39.11045)
    ## 19                         4 POINT (-96.56307 39.10077)
    ## 20                         4 POINT (-67.07689 18.02126)
    ## 21                         6 POINT (-88.16118 31.85386)
    ## 22                         6 POINT (-80.52485 37.37831)
    ## 23                         4 POINT (-109.3883 38.24828)
    ## 24                         4 POINT (-105.5824 40.05425)
    ## 25                         4 POINT (-100.9154 46.76972)
    ## 26                         4  POINT (-99.05878 35.4106)
    ## 27                         4  POINT (-112.4524 40.1776)
    ## 28                         6 POINT (-84.28259 35.96413)
    ## 29                         6 POINT (-81.99343 29.68928)
    ## 30                         6 POINT (-155.3173 19.55309)
    ## 31                         5   POINT (-105.546 40.2759)
    ## 32                         6 POINT (-78.13949 38.89292)
    ## 33                         6 POINT (-76.56001 38.89013)
    ## 34                         6 POINT (-119.7323 37.10878)
    ## 35                         6 POINT (-119.2622 37.03337)
    ## 36                         4 POINT (-110.8355 31.91068)
    ## 37                         6 POINT (-89.58637 45.50894)
    ## 38                         4 POINT (-103.0293 40.46189)
    ## 39                         5 POINT (-87.39326 32.95047)
    ## 40                         7  POINT (-119.006 37.00583)
    ## 41                         4 POINT (-149.3705 68.66109)
    ## 42                         6 POINT (-89.58571 45.49369)
    ## 43                         6 POINT (-95.19215 39.04043)
    ## 44                         6 POINT (-89.53725 46.23391)
    ## 45                         4  POINT (-99.24133 47.1282)
    ## 46                         8 POINT (-121.9519 45.82049)
    ## 47                         5 POINT (-110.5391 44.95348)

Plot your filtered data.

``` r
ggplot() +
  geom_sf(data = usa, col="white") +
  geom_sf(data=terrestrial_sites,aes( color=field_avg_number_of_green_days), size=5) +
  coord_sf(xlim = c(bb[1], bb[3]), ylim = c(bb[2], bb[4]))
```

![](prototyping-notebook_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

Filter your data with a bounding box to only include sites in the “west”

``` r
box <- c(xmin = -130, ymin = 25, xmax = -100, ymax = 50)
west_us_terrestrial_neon_sites <- metatdata %>% 
  filter(field_site_type == "Core Terrestrial" | field_site_type == "Gradient Terrestrial" ) %>%
  st_crop(box)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
as.data.frame(west_us_terrestrial_neon_sites)
```

    ##    field_domain_id field_site_id                                field_site_name
    ## 1              D16          ABBY                                 Abby Road NEON
    ## 7              D10          CPER         Central Plains Experimental Range NEON
    ## 17             D14          JORN                Jornada Experimental Range NEON
    ## 23             D13          MOAB                                      Moab NEON
    ## 24             D13          NIWO                               Niwot Ridge NEON
    ## 25             D09          NOGP Northern Great Plains Research Laboratory NEON
    ## 27             D15          ONAQ                                    Onaqui NEON
    ## 31             D10          RMNP                           Rocky Mountains NEON
    ## 34             D17          SJER            San Joaquin Experimental Range NEON
    ## 35             D17          SOAP                           Soaproot Saddle NEON
    ## 36             D14          SRER             Santa Rita Experimental Range NEON
    ## 38             D10          STER                            North Sterling NEON
    ## 40             D17          TEAK                           Lower Teakettle NEON
    ## 46             D16          WREF            Wind River Experimental Forest NEON
    ## 47             D12          YELL                 Yellowstone National Park NEON
    ##         field_site_type field_site_subtype field_colocated_site
    ## 1  Gradient Terrestrial                                        
    ## 7      Core Terrestrial                                        
    ## 17 Gradient Terrestrial                                        
    ## 23 Gradient Terrestrial                                        
    ## 24     Core Terrestrial                                    COMO
    ## 25 Gradient Terrestrial                                        
    ## 27     Core Terrestrial                                        
    ## 31 Gradient Terrestrial                                        
    ## 34     Core Terrestrial                                        
    ## 35 Gradient Terrestrial                                        
    ## 36     Core Terrestrial                                        
    ## 38 Gradient Terrestrial                                        
    ## 40 Gradient Terrestrial                                        
    ## 46     Core Terrestrial                                        
    ## 47     Core Terrestrial                               BLDE|YELL
    ##                                          field_site_host
    ## 1             Washington Department of Natural Resources
    ## 7                     USDA-Agricultural Research Service
    ## 17                    USDA-Agricultural Research Service
    ## 23                             Bureau of Land Management
    ## 24     US Forest Service, University of Colorado-Boulder
    ## 25                    USDA Agricultural Research Service
    ## 27                             Bureau of Land Management
    ## 31                             Roosevelt National Forest
    ## 34                                     US Forest Service
    ## 35                                     US Forest Service
    ## 36                                 University of Arizona
    ## 38                                               Private
    ## 40                                     US Forest Service
    ## 46 Pacific Northwest Research Station, US Forest Service
    ## 47                                 National Park Service
    ##                                                                                                                                                                                   field_site_url
    ## 1                                                                                                                                                                        https://www.dnr.wa.gov/
    ## 7  https://www.ars.usda.gov/plains-area/fort-collins-co/center-for-agricultural-resources-research/rangeland-resources-systems-research/docs/rrsr/central-plains-experimental-research-location/
    ## 17                                                                                                                                                                     https://jornada.nmsu.edu/
    ## 23                                                                                                                                                  https://www.blm.gov/office/moab-field-office
    ## 24                                                                                                                                                                                              
    ## 25                                                                                                                                         https://www.ars.usda.gov/plains-area/mandan-nd/ngprl/
    ## 27                                                                                                                                             https://www.blm.gov/office/salt-lake-field-office
    ## 31                                                                                                                                                                   https://www.fs.usda.gov/arp
    ## 34                                                                                                                                                     https://www.fs.fed.us/psw/ef/san_joaquin/
    ## 35                                                                                                                                                               https://www.fs.usda.gov/sierra/
    ## 36                                                                              https://taac.arizona.edu/sites/taac.arizona.edu/files/SRER%20Range%20Use%20Agreement%202014%20revised%5B1%5D.pdf
    ## 38                                                                                                                                                                                              
    ## 40                                                                                                                                                               https://www.fs.usda.gov/sierra/
    ## 46                                                                                                                                                                  https://www.fs.usda.gov/pnw/
    ## 47                                                                                                                                                                    https://irma.nps.gov/rprs/
    ##    field_nonneon_research_allowed
    ## 1                         Limited
    ## 7                             Yes
    ## 17                            Yes
    ## 23                            Yes
    ## 24                            Yes
    ## 25                            Yes
    ## 27                        Limited
    ## 31                            Yes
    ## 34                            Yes
    ## 35                            Yes
    ## 36                            Yes
    ## 38                   Very Limited
    ## 40                            Yes
    ## 46                            Yes
    ## 47                        Limited
    ##                                                                                                                                                                                                                                                                   field_access_details
    ## 1                                                                                                                                                                                                Reseachers should coordinate with the site manager and submit a site research permit.
    ## 7                                                                                                                                                                                                                                  Reseachers should coordinate with the site manager.
    ## 17        JORN encourages research activities that do not conflict with on-going activities. Please apply directly with site host and USDA for a site research permit.  All requests are reviewed for impacts to cultural resources, endangered species and other research activities.
    ## 23                                                                                                                                                                                                                                 Reseachers should coordinate with the site manager.
    ## 24                                                                                                                                                                                               Reseachers should coordinate with the site manager and submit a site research permit.
    ## 25                                                                                                                                                    This site host welcomes and encourages additional research activities related to the USDA Agricultural Research Service mission.
    ## 27                                                                                                                                                                                               Reseachers should coordinate with the site manager and submit a site research permit.
    ## 31                                                                                                                                                                                      Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 34                                                                                                                                                                                      Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 35                                                                                                                                                                                      Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 36 SRER encourages research activities that do not conflict with on-going research. Researchers should apply directly for a site research permit. All proposed actions will be reviewed for potential impacts to cultural resources, endangered species and other research activities.
    ## 38                                                                                                                                                                                                                                      Very limited available via private land owner.
    ## 40                                                                                                                                                                                      Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 46                                                                                                                                    Reseachers should coordinate with the site manager and submit a site research permit. Various on-going research projects to be aware of at site.
    ## 47                                                                           The National Park Service is open to additional research activities taking place in this area. Apply via IRMA Permitting portal. NEON research area is limited to additional research due to sensitivity.
    ##    field_neon_field_operations_office field_geodetic_datum field_utm_northing
    ## 1          Domain 16 Support Facility                WGS84            5067870
    ## 7       Domain 10/13 Support Facility                WGS84            4518311
    ## 17         Domain 14 Support Facility                WGS84            3607409
    ## 23      Domain 10/13 Support Facility                WGS84            4234597
    ## 24      Domain 10/13 Support Facility                WGS84            4433941
    ## 25         Domain 09 Support Facility                WGS84            5181356
    ## 27         Domain 15 Support Facility                WGS84            4448480
    ## 31      Domain 10/13 Support Facility                WGS84            4458524
    ## 34         Domain 17 Support Facility                WGS84            4110434
    ## 35         Domain 17 Support Facility                WGS84            4100967
    ## 36         Domain 14 Support Facility                WGS84            3530547
    ## 38      Domain 10/13 Support Facility                WGS84            4480891
    ## 40         Domain 17 Support Facility                WGS84            4097400
    ## 46         Domain 16 Support Facility                WGS84            5074637
    ## 47         Domain 12 Support Facility                WGS84            4977886
    ##    field_utm_easting field_utm_zone field_site_county field_site_state
    ## 1           552075.5            10N             Clark               WA
    ## 7           521454.8            13N              Weld               CO
    ## 17          327074.6            13N          Dona Ana               NM
    ## 23          641031.5            12N          San Juan               UT
    ## 24          450328.3            13N           Boulder               CO
    ## 25          353761.6            14N            Morton               ND
    ## 27          376339.7            12N            Tooele               UT
    ## 31          453588.4            13N           Larimer               CO
    ## 34          257214.0            11N            Madera               CA
    ## 35          298792.7            11N            Fresno               CA
    ## 36          515553.9            12N              Pima               AZ
    ## 38          667085.0            13N             Logan               CO
    ## 40          321515.7            11N            Fresno               CA
    ## 46          581417.8            10N          Skamania               WA
    ## 47          536352.2            12N              Park               WY
    ##    field_site_country field_mean_elevation_m field_minimum_elevation_m
    ## 1                 USA                    365                        NA
    ## 7                 USA                   1654                      1601
    ## 17                USA                   1324                      1316
    ## 23                USA                   1799                      1722
    ## 24                USA                   3490                      2975
    ## 25                USA                    589                       517
    ## 27                USA                   1662                      1599
    ## 31                USA                   2742                      2454
    ## 34                USA                    400                       252
    ## 35                USA                   1210                      1044
    ## 36                USA                    997                       897
    ## 38                USA                   1365                      1352
    ## 40                USA                   2149                      2086
    ## 46                USA                    351                       300
    ## 47                USA                   2133                      1847
    ##    field_maximum_elevation_m field_mean_annual_temperature_C
    ## 1                        708                            10.0
    ## 7                       1686                             8.6
    ## 17                      1333                            15.7
    ## 23                      2039                            10.1
    ## 24                      3583                             0.3
    ## 25                       592                             5.9
    ## 27                      2061                             9.0
    ## 31                      3042                             2.9
    ## 34                       502                            16.4
    ## 35                      1373                            13.4
    ## 36                      1461                            19.3
    ## 38                      1371                             9.7
    ## 40                      2734                             8.0
    ## 46                      1008                             9.2
    ## 47                      2244                             3.4
    ##    field_mean_annual_precipitation_mm field_dominant_wind_direction
    ## 1                                2451                              
    ## 7                                 344                            NW
    ## 17                                271                            SW
    ## 23                                319                           SSW
    ## 24                               1005                           WNW
    ## 25                                457                           NNE
    ## 27                                288                            SW
    ## 31                                731                              
    ## 34                                540                            NW
    ## 35                                900                            NW
    ## 36                                346                           ESE
    ## 38                                433                            NW
    ## 40                               1223                            NW
    ## 46                               2225                            NW
    ## 47                                493                           SSE
    ##              field_mean_canopy_height_m
    ## 1  34m (in surrounding non-logged area)
    ## 7                                   0.4
    ## 17                                  0.4
    ## 23                                  0.2
    ## 24                                  0.2
    ## 25                                  0.4
    ## 27                                  1.2
    ## 31                                   19
    ## 34                                   21
    ## 35                                   32
    ## 36                                    2
    ## 38        varies based on crops planted
    ## 40                                   35
    ## 46                                   50
    ## 47                                   14
    ##                          field_dominant_nlcd_classes
    ## 1  Evergreen Forest|Grassland/Herbaceous|Shrub/Scrub
    ## 7                               Grassland/Herbaceous
    ## 17                                       Shrub/Scrub
    ## 23                      Evergreen Forest|Shrub/Scrub
    ## 24             Evergreen Forest|Grassland/Herbaceous
    ## 25                              Grassland/Herbaceous
    ## 27                      Evergreen Forest|Shrub/Scrub
    ## 31                                  Evergreen Forest
    ## 34 Evergreen Forest|Grassland/Herbaceous|Shrub/Scrub
    ## 35                      Evergreen Forest|Shrub/Scrub
    ## 36                                       Shrub/Scrub
    ## 38                                  Cultivated Crops
    ## 40                      Evergreen Forest|Shrub/Scrub
    ## 46                                  Evergreen Forest
    ## 47 Evergreen Forest|Grassland/Herbaceous|Shrub/Scrub
    ##                                                                                                                 field_domint_plant_species
    ## 1                                        Gaultheria shallon (salal)|Pseudotsuga menziesii (douglas fir)|Pteridium aquilinum (bracken fern)
    ## 7                 Bouteloua gracilis (Blue grama)|Hesperostipa comata (Needle-and-thread grass)|Thelesperma filifolium (Stiff greenthread)
    ## 17                                Prosopis glandulosa (honey mesquite)|Yucca elata (soaptree yucca)|Bouteloua eriopoda (black grama grass)
    ## 23                                  Atriplex canescens (fourwing saltbush)|Ephedra viridis (Mormon tea)|Bouteloua gracilis (blue grama)  .
    ## 24                                     Abies lasiocarpa (subapline fir)|Picea engelmannii (Engelmann spruce)|Vaccinium sp. (blueberry sp.)
    ## 25               Elaeagnus angustifolia (Russian olive)|Poa pratensis (Kentucky bluegrass)|Symphoricarpos occidentalis (Western snowberry)
    ## 27                      Artemisia tridentata (big sagebrush)|Ceratocephala testiculata (curveseed butterwort)|Bromus tectorum (cheatgrass)
    ## 31                                    Pinus contorta (Lodgepole pine)|Abies lasiocarpa (Subalpine fir)|Pseudotsuga menziesii (Douglas-fir)
    ## 34                                           Quercus douglasii (blue oak)|Erodium botrys (broadleaf filaree)|Bromus diandrus (great brome)
    ## 35 Arctostaphylos viscida ssp. Mariposa (mariposa manzanita)|Chamaebatia foliolosa (mountain misery)|Quercus chrysolepis (canyon live oak)
    ## 36                               Larrea tridentata (creosote bush)|Prosopis veluti (velvet mesquite)|Parkinsonia florida (blue palo verde)
    ## 38                                                                                              Agriculture field, crops vary year to year
    ## 40                                                    Abies magnifica (red fir)|Abies concolor (white fir)|Pinus contorta (lodgepole pine)
    ## 46                                     Gaultheria shallon (salal)|Psuedotsuga menziesii (douglas fir)|Tsuga heterophylla (western hemlock)
    ## 47                                Artemisia tridentata (big sagebrush)|Pseudotsuga menziesii (douglas fir)|Pinus contorta (lodgepole pine)
    ##    field_usgs_huc field_watershed_name field_watershed_size_km2
    ## 1                                                            NA
    ## 7                                                            NA
    ## 17                                                           NA
    ## 23                                                           NA
    ## 24                                                           NA
    ## 25                                                           NA
    ## 27                                                           NA
    ## 31                                                           NA
    ## 34                                                           NA
    ## 35                                                           NA
    ## 36                                                           NA
    ## 38                                                           NA
    ## 40                                                           NA
    ## 46                                                           NA
    ## 47                                                           NA
    ##    field_lake_depth_mean_m field_lake_depth_max_m field_tower_height_m
    ## 1                       NA                     NA                   19
    ## 7                       NA                     NA                    9
    ## 17                      NA                     NA                    8
    ## 23                      NA                     NA                    8
    ## 24                      NA                     NA                    8
    ## 25                      NA                     NA                    8
    ## 27                      NA                     NA                    8
    ## 31                      NA                     NA                   25
    ## 34                      NA                     NA                   39
    ## 35                      NA                     NA                   52
    ## 36                      NA                     NA                    8
    ## 38                      NA                     NA                    8
    ## 40                      NA                     NA                   59
    ## 46                      NA                     NA                   74
    ## 47                      NA                     NA                   18
    ##    field_usgs_geology_unit
    ## 1                WAEOv1b;0
    ## 7                   COKl;0
    ## 17                 NMQTs;0
    ## 23                  UTQe;0
    ## 24                  COXb;0
    ## 25                 NDQol;0
    ## 27                  UTQa;0
    ## 31                  COYg;0
    ## 34               CAgrMZ3;0
    ## 35               CAgrMZ3;0
    ## 36                   AZQ;0
    ## 38                  COTo;0
    ## 40                  CAQg;0
    ## 46                  WAQv;0
    ## 47           WYQb;0|WYTs;0
    ##                                           field_megapit_soil_family
    ## 1                                                                  
    ## 7      Fine, loamy, mixed, superactive, mesic.  Aridic Argiustolls.
    ## 17 Coarse, loamy, mixed, superactive, thermicc. Typic Petrocalcids.
    ## 23    Coarse, loamy, mixed, superactive, mesic. Ustic Haplocalcids.
    ## 24           Coarse, loamy, mixed, superactive. Typic Haplocryolls.
    ## 25      Fine, loamy, mixed, superactive, frigid. Typic Argiustolls.
    ## 27      Fine, loamy, mixed, superactive, mesic. Xeric Haplocalcids.
    ## 31         Loamy, skeletal, mixed, superactive. Ustic Haplocryolls.
    ## 34                                                                 
    ## 35       Fine, loamy, mixed, semiactive, mesic. Ultic Haploxeralfs.
    ## 36                                                                 
    ## 38      Fine, silty, mixed, superactive, mesic. Pachic Argiustolls.
    ## 40                                                                 
    ## 46                       Medial, amorphic, mesic. Typic Hapludands.
    ## 47                                                                 
    ##        field_soil_subgroup field_avg_number_of_green_days
    ## 1          Andic Humudepts                            190
    ## 7       Aridic Argiustolls                            260
    ## 17      Typic Petrocalcids                            260
    ## 23      Ustic Haplocalcids                            215
    ## 24      Typic Haplocryolls                            130
    ## 25       Typic Argiustolls                            175
    ## 27      Xeric Haplocalcids                            205
    ## 31      Ustic Haplocryolls                            165
    ## 34 Psammentic Haploxerolls                            250
    ## 35      Ultic Haploxeralfs                            200
    ## 36     Typic Torrifluvents                            247
    ## 38      Pachic Argiustolls                            180
    ## 40      Pachic Humixerepts                            180
    ## 46        Typic Hapludands                            175
    ## 47      Pachic Argiustolls                            144
    ##    field_avg_grean_increase_doy field_avg_green_max_doy
    ## 1                           110                     165
    ## 7                            90                     165
    ## 17                           80                     185
    ## 23                           85                     165
    ## 24                          140                     190
    ## 25                          115                     170
    ## 27                           75                     130
    ## 31                          120                     180
    ## 34                          270                      65
    ## 35                           90                     155
    ## 36                       87|186                 139|215
    ## 38                           90                     150
    ## 40                          120                     180
    ## 46                          115                     165
    ## 47                          125                     169
    ##    field_avg_green_decrease_doy field_avg_green_min_doy
    ## 1                           205                     300
    ## 7                           210                     350
    ## 17                          245                     340
    ## 23                          225                     300
    ## 24                          220                     270
    ## 25                          200                     290
    ## 27                          170                     280
    ## 31                          210                     285
    ## 34                           95                     155
    ## 35                          185                     290
    ## 36                      189|259                 219|334
    ## 38                          190                     270
    ## 40                          205                     300
    ## 46                          210                     290
    ## 47                          193                     269
    ##                                                                                                                                                   field_phenocams
    ## 1  [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D16.ABBY.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D16.ABBY.DP1.00042)
    ## 7  [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.CPER.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.CPER.DP1.00042)
    ## 17 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D14.JORN.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D14.JORN.DP1.00042)
    ## 23 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D13.MOAB.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D13.MOAB.DP1.00042)
    ## 24 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D13.NIWO.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D13.NIWO.DP1.00042)
    ## 25 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D09.NOGP.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D09.NOGP.DP1.00042)
    ## 27 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D15.ONAQ.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D15.ONAQ.DP1.00042)
    ## 31 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.RMNP.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.RMNP.DP1.00042)
    ## 34 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.SJER.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.SJER.DP1.00042)
    ## 35 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.SOAP.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.SOAP.DP1.00042)
    ## 36 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D14.SRER.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D14.SRER.DP1.00042)
    ## 38 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.STER.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.STER.DP1.00042)
    ## 40 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.TEAK.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.TEAK.DP1.00042)
    ## 46 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D16.WREF.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D16.WREF.DP1.00042)
    ## 47 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D12.YELL.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D12.YELL.DP1.00042)
    ##    field_number_tower_levels                   geometry
    ## 1                          5 POINT (-122.3303 45.76244)
    ## 7                          4 POINT (-104.7456 40.81554)
    ## 17                         4 POINT (-106.8425 32.59069)
    ## 23                         4 POINT (-109.3883 38.24828)
    ## 24                         4 POINT (-105.5824 40.05425)
    ## 25                         4 POINT (-100.9154 46.76972)
    ## 27                         4  POINT (-112.4524 40.1776)
    ## 31                         5   POINT (-105.546 40.2759)
    ## 34                         6 POINT (-119.7323 37.10878)
    ## 35                         6 POINT (-119.2622 37.03337)
    ## 36                         4 POINT (-110.8355 31.91068)
    ## 38                         4 POINT (-103.0293 40.46189)
    ## 40                         7  POINT (-119.006 37.00583)
    ## 46                         8 POINT (-121.9519 45.82049)
    ## 47                         5 POINT (-110.5391 44.95348)

Plot your western points to confirm that you have properly filtered
them.

``` r
ggplot() +
  geom_sf(data = usa, col="white") +
  geom_sf(data=west_us_terrestrial_neon_sites,aes( color=field_avg_number_of_green_days), size=2) +
  coord_sf(xlim = c(box[1], box[3]), ylim = c(box[2], box[4]))
```

![](prototyping-notebook_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

Filter the western NEON sites to only include sites with Evergreen trees
present.

``` r
west_us_terrestrial_Evergreen_neon_sites <- metatdata %>% 
  filter(field_site_type == "Core Terrestrial" | field_site_type == "Gradient Terrestrial" ) %>%
  filter(field_dominant_nlcd_classes !=  "Grassland/Herbaceous" & field_dominant_nlcd_classes !=  "Shrub/Scrub" & field_dominant_nlcd_classes != "Cultivated Crops") %>%
  st_crop(box)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
as.data.frame(west_us_terrestrial_Evergreen_neon_sites)
```

    ##    field_domain_id field_site_id                     field_site_name
    ## 1              D16          ABBY                      Abby Road NEON
    ## 19             D13          MOAB                           Moab NEON
    ## 20             D13          NIWO                    Niwot Ridge NEON
    ## 22             D15          ONAQ                         Onaqui NEON
    ## 26             D10          RMNP                Rocky Mountains NEON
    ## 29             D17          SJER San Joaquin Experimental Range NEON
    ## 30             D17          SOAP                Soaproot Saddle NEON
    ## 33             D17          TEAK                Lower Teakettle NEON
    ## 39             D16          WREF Wind River Experimental Forest NEON
    ## 40             D12          YELL      Yellowstone National Park NEON
    ##         field_site_type field_site_subtype field_colocated_site
    ## 1  Gradient Terrestrial                                        
    ## 19 Gradient Terrestrial                                        
    ## 20     Core Terrestrial                                    COMO
    ## 22     Core Terrestrial                                        
    ## 26 Gradient Terrestrial                                        
    ## 29     Core Terrestrial                                        
    ## 30 Gradient Terrestrial                                        
    ## 33 Gradient Terrestrial                                        
    ## 39     Core Terrestrial                                        
    ## 40     Core Terrestrial                               BLDE|YELL
    ##                                          field_site_host
    ## 1             Washington Department of Natural Resources
    ## 19                             Bureau of Land Management
    ## 20     US Forest Service, University of Colorado-Boulder
    ## 22                             Bureau of Land Management
    ## 26                             Roosevelt National Forest
    ## 29                                     US Forest Service
    ## 30                                     US Forest Service
    ## 33                                     US Forest Service
    ## 39 Pacific Northwest Research Station, US Forest Service
    ## 40                                 National Park Service
    ##                                       field_site_url
    ## 1                            https://www.dnr.wa.gov/
    ## 19      https://www.blm.gov/office/moab-field-office
    ## 20                                                  
    ## 22 https://www.blm.gov/office/salt-lake-field-office
    ## 26                       https://www.fs.usda.gov/arp
    ## 29         https://www.fs.fed.us/psw/ef/san_joaquin/
    ## 30                   https://www.fs.usda.gov/sierra/
    ## 33                   https://www.fs.usda.gov/sierra/
    ## 39                      https://www.fs.usda.gov/pnw/
    ## 40                        https://irma.nps.gov/rprs/
    ##    field_nonneon_research_allowed
    ## 1                         Limited
    ## 19                            Yes
    ## 20                            Yes
    ## 22                        Limited
    ## 26                            Yes
    ## 29                            Yes
    ## 30                            Yes
    ## 33                            Yes
    ## 39                            Yes
    ## 40                        Limited
    ##                                                                                                                                                                                         field_access_details
    ## 1                                                                                                                      Reseachers should coordinate with the site manager and submit a site research permit.
    ## 19                                                                                                                                                       Reseachers should coordinate with the site manager.
    ## 20                                                                                                                     Reseachers should coordinate with the site manager and submit a site research permit.
    ## 22                                                                                                                     Reseachers should coordinate with the site manager and submit a site research permit.
    ## 26                                                                                                            Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 29                                                                                                            Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 30                                                                                                            Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 33                                                                                                            Researchers should coordinate directly with the US Forest Service for permitting and approval.
    ## 39                                                          Reseachers should coordinate with the site manager and submit a site research permit. Various on-going research projects to be aware of at site.
    ## 40 The National Park Service is open to additional research activities taking place in this area. Apply via IRMA Permitting portal. NEON research area is limited to additional research due to sensitivity.
    ##    field_neon_field_operations_office field_geodetic_datum field_utm_northing
    ## 1          Domain 16 Support Facility                WGS84            5067870
    ## 19      Domain 10/13 Support Facility                WGS84            4234597
    ## 20      Domain 10/13 Support Facility                WGS84            4433941
    ## 22         Domain 15 Support Facility                WGS84            4448480
    ## 26      Domain 10/13 Support Facility                WGS84            4458524
    ## 29         Domain 17 Support Facility                WGS84            4110434
    ## 30         Domain 17 Support Facility                WGS84            4100967
    ## 33         Domain 17 Support Facility                WGS84            4097400
    ## 39         Domain 16 Support Facility                WGS84            5074637
    ## 40         Domain 12 Support Facility                WGS84            4977886
    ##    field_utm_easting field_utm_zone field_site_county field_site_state
    ## 1           552075.5            10N             Clark               WA
    ## 19          641031.5            12N          San Juan               UT
    ## 20          450328.3            13N           Boulder               CO
    ## 22          376339.7            12N            Tooele               UT
    ## 26          453588.4            13N           Larimer               CO
    ## 29          257214.0            11N            Madera               CA
    ## 30          298792.7            11N            Fresno               CA
    ## 33          321515.7            11N            Fresno               CA
    ## 39          581417.8            10N          Skamania               WA
    ## 40          536352.2            12N              Park               WY
    ##    field_site_country field_mean_elevation_m field_minimum_elevation_m
    ## 1                 USA                    365                        NA
    ## 19                USA                   1799                      1722
    ## 20                USA                   3490                      2975
    ## 22                USA                   1662                      1599
    ## 26                USA                   2742                      2454
    ## 29                USA                    400                       252
    ## 30                USA                   1210                      1044
    ## 33                USA                   2149                      2086
    ## 39                USA                    351                       300
    ## 40                USA                   2133                      1847
    ##    field_maximum_elevation_m field_mean_annual_temperature_C
    ## 1                        708                            10.0
    ## 19                      2039                            10.1
    ## 20                      3583                             0.3
    ## 22                      2061                             9.0
    ## 26                      3042                             2.9
    ## 29                       502                            16.4
    ## 30                      1373                            13.4
    ## 33                      2734                             8.0
    ## 39                      1008                             9.2
    ## 40                      2244                             3.4
    ##    field_mean_annual_precipitation_mm field_dominant_wind_direction
    ## 1                                2451                              
    ## 19                                319                           SSW
    ## 20                               1005                           WNW
    ## 22                                288                            SW
    ## 26                                731                              
    ## 29                                540                            NW
    ## 30                                900                            NW
    ## 33                               1223                            NW
    ## 39                               2225                            NW
    ## 40                                493                           SSE
    ##              field_mean_canopy_height_m
    ## 1  34m (in surrounding non-logged area)
    ## 19                                  0.2
    ## 20                                  0.2
    ## 22                                  1.2
    ## 26                                   19
    ## 29                                   21
    ## 30                                   32
    ## 33                                   35
    ## 39                                   50
    ## 40                                   14
    ##                          field_dominant_nlcd_classes
    ## 1  Evergreen Forest|Grassland/Herbaceous|Shrub/Scrub
    ## 19                      Evergreen Forest|Shrub/Scrub
    ## 20             Evergreen Forest|Grassland/Herbaceous
    ## 22                      Evergreen Forest|Shrub/Scrub
    ## 26                                  Evergreen Forest
    ## 29 Evergreen Forest|Grassland/Herbaceous|Shrub/Scrub
    ## 30                      Evergreen Forest|Shrub/Scrub
    ## 33                      Evergreen Forest|Shrub/Scrub
    ## 39                                  Evergreen Forest
    ## 40 Evergreen Forest|Grassland/Herbaceous|Shrub/Scrub
    ##                                                                                                                 field_domint_plant_species
    ## 1                                        Gaultheria shallon (salal)|Pseudotsuga menziesii (douglas fir)|Pteridium aquilinum (bracken fern)
    ## 19                                  Atriplex canescens (fourwing saltbush)|Ephedra viridis (Mormon tea)|Bouteloua gracilis (blue grama)  .
    ## 20                                     Abies lasiocarpa (subapline fir)|Picea engelmannii (Engelmann spruce)|Vaccinium sp. (blueberry sp.)
    ## 22                      Artemisia tridentata (big sagebrush)|Ceratocephala testiculata (curveseed butterwort)|Bromus tectorum (cheatgrass)
    ## 26                                    Pinus contorta (Lodgepole pine)|Abies lasiocarpa (Subalpine fir)|Pseudotsuga menziesii (Douglas-fir)
    ## 29                                           Quercus douglasii (blue oak)|Erodium botrys (broadleaf filaree)|Bromus diandrus (great brome)
    ## 30 Arctostaphylos viscida ssp. Mariposa (mariposa manzanita)|Chamaebatia foliolosa (mountain misery)|Quercus chrysolepis (canyon live oak)
    ## 33                                                    Abies magnifica (red fir)|Abies concolor (white fir)|Pinus contorta (lodgepole pine)
    ## 39                                     Gaultheria shallon (salal)|Psuedotsuga menziesii (douglas fir)|Tsuga heterophylla (western hemlock)
    ## 40                                Artemisia tridentata (big sagebrush)|Pseudotsuga menziesii (douglas fir)|Pinus contorta (lodgepole pine)
    ##    field_usgs_huc field_watershed_name field_watershed_size_km2
    ## 1                                                            NA
    ## 19                                                           NA
    ## 20                                                           NA
    ## 22                                                           NA
    ## 26                                                           NA
    ## 29                                                           NA
    ## 30                                                           NA
    ## 33                                                           NA
    ## 39                                                           NA
    ## 40                                                           NA
    ##    field_lake_depth_mean_m field_lake_depth_max_m field_tower_height_m
    ## 1                       NA                     NA                   19
    ## 19                      NA                     NA                    8
    ## 20                      NA                     NA                    8
    ## 22                      NA                     NA                    8
    ## 26                      NA                     NA                   25
    ## 29                      NA                     NA                   39
    ## 30                      NA                     NA                   52
    ## 33                      NA                     NA                   59
    ## 39                      NA                     NA                   74
    ## 40                      NA                     NA                   18
    ##    field_usgs_geology_unit
    ## 1                WAEOv1b;0
    ## 19                  UTQe;0
    ## 20                  COXb;0
    ## 22                  UTQa;0
    ## 26                  COYg;0
    ## 29               CAgrMZ3;0
    ## 30               CAgrMZ3;0
    ## 33                  CAQg;0
    ## 39                  WAQv;0
    ## 40           WYQb;0|WYTs;0
    ##                                        field_megapit_soil_family
    ## 1                                                               
    ## 19 Coarse, loamy, mixed, superactive, mesic. Ustic Haplocalcids.
    ## 20        Coarse, loamy, mixed, superactive. Typic Haplocryolls.
    ## 22   Fine, loamy, mixed, superactive, mesic. Xeric Haplocalcids.
    ## 26      Loamy, skeletal, mixed, superactive. Ustic Haplocryolls.
    ## 29                                                              
    ## 30    Fine, loamy, mixed, semiactive, mesic. Ultic Haploxeralfs.
    ## 33                                                              
    ## 39                    Medial, amorphic, mesic. Typic Hapludands.
    ## 40                                                              
    ##        field_soil_subgroup field_avg_number_of_green_days
    ## 1          Andic Humudepts                            190
    ## 19      Ustic Haplocalcids                            215
    ## 20      Typic Haplocryolls                            130
    ## 22      Xeric Haplocalcids                            205
    ## 26      Ustic Haplocryolls                            165
    ## 29 Psammentic Haploxerolls                            250
    ## 30      Ultic Haploxeralfs                            200
    ## 33      Pachic Humixerepts                            180
    ## 39        Typic Hapludands                            175
    ## 40      Pachic Argiustolls                            144
    ##    field_avg_grean_increase_doy field_avg_green_max_doy
    ## 1                           110                     165
    ## 19                           85                     165
    ## 20                          140                     190
    ## 22                           75                     130
    ## 26                          120                     180
    ## 29                          270                      65
    ## 30                           90                     155
    ## 33                          120                     180
    ## 39                          115                     165
    ## 40                          125                     169
    ##    field_avg_green_decrease_doy field_avg_green_min_doy
    ## 1                           205                     300
    ## 19                          225                     300
    ## 20                          220                     270
    ## 22                          170                     280
    ## 26                          210                     285
    ## 29                           95                     155
    ## 30                          185                     290
    ## 33                          205                     300
    ## 39                          210                     290
    ## 40                          193                     269
    ##                                                                                                                                                   field_phenocams
    ## 1  [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D16.ABBY.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D16.ABBY.DP1.00042)
    ## 19 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D13.MOAB.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D13.MOAB.DP1.00042)
    ## 20 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D13.NIWO.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D13.NIWO.DP1.00042)
    ## 22 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D15.ONAQ.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D15.ONAQ.DP1.00042)
    ## 26 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.RMNP.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D10.RMNP.DP1.00042)
    ## 29 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.SJER.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.SJER.DP1.00042)
    ## 30 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.SOAP.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.SOAP.DP1.00042)
    ## 33 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.TEAK.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D17.TEAK.DP1.00042)
    ## 39 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D16.WREF.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D16.WREF.DP1.00042)
    ## 40 [Tower top](https://phenocam.sr.unh.edu/webcam/sites/NEON.D12.YELL.DP1.00033)|[Tower bottom](https://phenocam.sr.unh.edu/webcam/sites/NEON.D12.YELL.DP1.00042)
    ##    field_number_tower_levels                   geometry
    ## 1                          5 POINT (-122.3303 45.76244)
    ## 19                         4 POINT (-109.3883 38.24828)
    ## 20                         4 POINT (-105.5824 40.05425)
    ## 22                         4  POINT (-112.4524 40.1776)
    ## 26                         5   POINT (-105.546 40.2759)
    ## 29                         6 POINT (-119.7323 37.10878)
    ## 30                         6 POINT (-119.2622 37.03337)
    ## 33                         7  POINT (-119.006 37.00583)
    ## 39                         8 POINT (-121.9519 45.82049)
    ## 40                         5 POINT (-110.5391 44.95348)

Plot your Evergreen filter to confirm that your filter worked properly.

``` r
ggplot() +
  geom_sf(data = usa, col="white") +
  geom_sf(data=west_us_terrestrial_Evergreen_neon_sites,aes( color=field_mean_canopy_height_m), size=2) +
  coord_sf(xlim = c(box[1], box[3]), ylim = c(box[2], box[4]))
```

![](prototyping-notebook_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

## Learn how to use an API

Select one of the sitecodes from your filtered metadataset above and
enter it below. Follow the tutorial offered by NEON to explore the data
structure and organizational scheme for neon api data.

<https://www.neonscience.org/resources/learning-hub/tutorials/neon-api-01-introduction-requests>

``` python
SERVER = 'http://data.neonscience.org/api/v0/'
SITECODE = 'NIWO'

site_request = requests.get(SERVER+'sites/'+SITECODE)

#Convert to Python JSON object
site_json = site_request.json()
site_json.keys()
```

    ## dict_keys(['data'])

``` python
site_json['data'].keys()
```

    ## dict_keys(['siteCode', 'siteName', 'siteDescription', 'siteType', 'siteLatitude', 'siteLongitude', 'stateCode', 'stateName', 'domainCode', 'domainName', 'deimsId', 'releases', 'dataProducts'])

``` python
site_json
```

    ## {'data': {'siteCode': 'NIWO', 'siteName': 'Niwot Ridge NEON', 'siteDescription': 'Niwot Ridge NEON', 'siteType': 'CORE', 'siteLatitude': 40.05425, 'siteLongitude': -105.58237, 'stateCode': 'CO', 'stateName': 'Colorado', 'domainCode': 'D13', 'domainName': 'Southern Rockies and Colorado Plateau', 'deimsId': 'https://deims.org/bd3322ed-0ae8-4da5-beb5-1bd5c9205fa1', 'releases': [{'release': 'RELEASE-2021', 'generationDate': '2021-01-23T02:30:02Z', 'url': 'https://data.neonscience.org/api/v0/releases/RELEASE-2021'}, {'release': 'RELEASE-2022', 'generationDate': '2022-01-20T17:39:46Z', 'url': 'https://data.neonscience.org/api/v0/releases/RELEASE-2022'}], 'dataProducts': [{'dataProductCode': 'DP1.00001.001', 'dataProductTitle': '2D wind speed and direction', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00001.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00002.001', 'dataProductTitle': 'Single aspirated air temperature', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00002.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00003.001', 'dataProductTitle': 'Triple aspirated air temperature', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00003.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00004.001', 'dataProductTitle': 'Barometric pressure', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00004.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00005.001', 'dataProductTitle': 'IR biological temperature', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00005.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00006.001', 'dataProductTitle': 'Precipitation', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00006.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00013.001', 'dataProductTitle': 'Wet deposition chemical analysis', 'availableMonths': ['2018-11', '2018-12', '2019-01', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-12', '2020-01', '2020-03', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00013.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2018-11', '2018-12', '2019-01', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-12', '2020-01', '2020-03', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12']}]}, {'dataProductCode': 'DP1.00014.001', 'dataProductTitle': 'Shortwave radiation (direct and diffuse pyranometer)', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00014.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00017.001', 'dataProductTitle': 'Dust and particulate size distribution', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00017.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00022.001', 'dataProductTitle': 'Shortwave radiation (primary pyranometer)', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00022.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00023.001', 'dataProductTitle': 'Shortwave and longwave radiation (net radiometer)', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00023.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00024.001', 'dataProductTitle': 'Photosynthetically active radiation (PAR)', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00024.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00033.001', 'dataProductTitle': 'Phenology images', 'availableMonths': ['2017-04', '2017-05', '2017-06', '2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01', '2022-02', '2022-03'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2017-04', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2017-05', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2022-01', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2022-02', 'https://data.neonscience.org/api/v0/data/DP1.00033.001/NIWO/2022-03'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2017-04', '2017-05', '2017-06', '2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01', '2022-02', '2022-03']}]}, {'dataProductCode': 'DP1.00038.001', 'dataProductTitle': 'Stable isotopes in precipitation', 'availableMonths': ['2018-11', '2018-12', '2019-01', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-12', '2020-01', '2020-03', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00038.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2018-11', '2018-12', '2019-01', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-12', '2020-01', '2020-03', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12']}]}, {'dataProductCode': 'DP1.00040.001', 'dataProductTitle': 'Soil heat flux plate', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00040.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00041.001', 'dataProductTitle': 'Soil temperature', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00041.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00042.001', 'dataProductTitle': 'Snow depth and understory phenology images', 'availableMonths': ['2017-03', '2017-04', '2017-05', '2017-06', '2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01', '2022-02', '2022-03'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2017-03', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2017-04', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2017-05', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2022-01', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2022-02', 'https://data.neonscience.org/api/v0/data/DP1.00042.001/NIWO/2022-03'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2017-03', '2017-04', '2017-05', '2017-06', '2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01', '2022-02', '2022-03']}]}, {'dataProductCode': 'DP1.00043.001', 'dataProductTitle': 'Spectral sun photometer - calibrated sky radiances', 'availableMonths': ['2017-11', '2017-12', '2018-01', '2018-02', '2018-08', '2018-09', '2018-11', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00043.001/NIWO/2021-03'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2017-11', '2017-12', '2018-01', '2018-02', '2018-08', '2018-09', '2018-11', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03']}]}, {'dataProductCode': 'DP1.00066.001', 'dataProductTitle': 'Photosynthetically active radiation (quantum line)', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00066.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00094.001', 'dataProductTitle': 'Soil water content and water salinity', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00094.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00095.001', 'dataProductTitle': 'Soil CO2 concentration', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00095.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00096.001', 'dataProductTitle': 'Soil physical and chemical properties, Megapit', 'availableMonths': ['2014-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00096.001/NIWO/2014-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2014-08']}]}, {'dataProductCode': 'DP1.00098.001', 'dataProductTitle': 'Relative humidity', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00098.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP1.00101.001', 'dataProductTitle': 'Particulate mass', 'availableMonths': ['2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-12', '2020-01', '2020-03', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP1.00101.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-12', '2020-01', '2020-03', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12']}]}, {'dataProductCode': 'DP1.10003.001', 'dataProductTitle': 'Breeding landbird point counts', 'availableMonths': ['2015-07', '2017-07', '2018-07', '2019-07', '2020-07', '2021-07'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10003.001/NIWO/2015-07', 'https://data.neonscience.org/api/v0/data/DP1.10003.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10003.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10003.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10003.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.10003.001/NIWO/2021-07'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07']}, {'release': 'RELEASE-2022', 'availableMonths': ['2015-07', '2017-07', '2018-07', '2019-07', '2020-07']}]}, {'dataProductCode': 'DP1.10010.001', 'dataProductTitle': 'Coarse downed wood log survey', 'availableMonths': ['2016-09', '2016-10', '2019-09', '2019-10'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10010.001/NIWO/2016-09', 'https://data.neonscience.org/api/v0/data/DP1.10010.001/NIWO/2016-10', 'https://data.neonscience.org/api/v0/data/DP1.10010.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.10010.001/NIWO/2019-10'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2016-09', '2016-10', '2019-09', '2019-10']}]}, {'dataProductCode': 'DP1.10014.001', 'dataProductTitle': 'Coarse downed wood bulk density sampling', 'availableMonths': ['2019-10', '2020-07', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10014.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.10014.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.10014.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2019-10', '2020-07', '2020-08']}]}, {'dataProductCode': 'DP1.10017.001', 'dataProductTitle': 'Digital hemispheric photos of plot vegetation', 'availableMonths': ['2016-06', '2016-07', '2016-08', '2016-09', '2017-06', '2017-07', '2017-08', '2017-09', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2020-06', '2020-07', '2020-08', '2020-09', '2021-06', '2021-07', '2021-08', '2021-09'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2016-06', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2016-07', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2016-09', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.10017.001/NIWO/2021-09'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-06', '2021-07', '2021-08', '2021-09']}, {'release': 'RELEASE-2022', 'availableMonths': ['2016-06', '2016-07', '2016-08', '2016-09', '2017-06', '2017-07', '2017-08', '2017-09', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2020-06', '2020-07', '2020-08', '2020-09']}]}, {'dataProductCode': 'DP1.10020.001', 'dataProductTitle': 'Ground beetle sequences DNA barcode', 'availableMonths': ['2015-07', '2015-08', '2015-09', '2016-06', '2016-07', '2016-08', '2016-09', '2017-06', '2017-07', '2017-08', '2017-09', '2018-06', '2018-07', '2018-08', '2019-06', '2019-07', '2019-08', '2019-09'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2015-07', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2015-08', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2015-09', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2016-06', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2016-07', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2016-09', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10020.001/NIWO/2019-09'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2015-07', '2015-08', '2015-09', '2016-06', '2016-07', '2016-08', '2016-09', '2017-06', '2017-07', '2017-08', '2017-09', '2018-06', '2018-07', '2018-08', '2019-06', '2019-07', '2019-08', '2019-09']}]}, {'dataProductCode': 'DP1.10022.001', 'dataProductTitle': 'Ground beetles sampled from pitfall traps', 'availableMonths': ['2015-07', '2015-08', '2015-09', '2016-06', '2016-07', '2016-08', '2016-09', '2017-06', '2017-07', '2017-08', '2017-09', '2018-06', '2018-07', '2018-08', '2019-06', '2019-07', '2019-08', '2019-09', '2020-06', '2020-07', '2020-08', '2021-06', '2021-07', '2021-08', '2021-09'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2015-07', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2015-08', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2015-09', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2016-06', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2016-07', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2016-09', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.10022.001/NIWO/2021-09'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2020-06', '2020-07', '2020-08', '2021-06', '2021-07', '2021-08', '2021-09']}, {'release': 'RELEASE-2022', 'availableMonths': ['2015-07', '2015-08', '2015-09', '2016-06', '2016-07', '2016-08', '2016-09', '2017-06', '2017-07', '2017-08', '2017-09', '2018-06', '2018-07', '2018-08', '2019-06', '2019-07', '2019-08', '2019-09']}]}, {'dataProductCode': 'DP1.10023.001', 'dataProductTitle': 'Herbaceous clip harvest', 'availableMonths': ['2015-08', '2016-08', '2017-08', '2018-08', '2019-08', '2020-08', '2021-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10023.001/NIWO/2015-08', 'https://data.neonscience.org/api/v0/data/DP1.10023.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10023.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10023.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10023.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10023.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.10023.001/NIWO/2021-08'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-08']}, {'release': 'RELEASE-2022', 'availableMonths': ['2015-08', '2016-08', '2017-08', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP1.10026.001', 'dataProductTitle': 'Plant foliar traits', 'availableMonths': ['2019-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10026.001/NIWO/2019-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2019-08']}]}, {'dataProductCode': 'DP1.10033.001', 'dataProductTitle': 'Litterfall and fine woody debris production and chemistry', 'availableMonths': ['2016-06', '2016-07', '2016-08', '2016-09', '2016-10', '2016-11', '2017-07', '2017-08', '2017-10', '2017-11', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2020-07', '2020-08', '2020-09', '2020-11', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2016-06', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2016-07', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2016-09', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2016-10', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2016-11', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.10033.001/NIWO/2021-10'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-06', '2021-07', '2021-08', '2021-09', '2021-10']}, {'release': 'RELEASE-2022', 'availableMonths': ['2016-06', '2016-07', '2016-08', '2016-09', '2016-10', '2016-11', '2017-07', '2017-08', '2017-10', '2017-11', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2020-07', '2020-08', '2020-09', '2020-11']}]}, {'dataProductCode': 'DP1.10038.001', 'dataProductTitle': 'Mosquito sequences DNA barcode', 'availableMonths': ['2017-06', '2017-07', '2017-08', '2018-06', '2018-07', '2019-06', '2019-07', '2019-08', '2020-06', '2020-07'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10038.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.10038.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10038.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10038.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10038.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10038.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.10038.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10038.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10038.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.10038.001/NIWO/2020-07'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-06', '2017-07', '2017-08', '2018-06', '2018-07', '2019-06', '2019-07', '2019-08', '2020-06', '2020-07']}]}, {'dataProductCode': 'DP1.10041.001', 'dataProductTitle': 'Mosquito pathogen status', 'availableMonths': ['2017-06', '2017-07', '2017-08', '2018-06', '2018-07', '2018-08', '2019-07', '2019-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10041.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.10041.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10041.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10041.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10041.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10041.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10041.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10041.001/NIWO/2019-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-06', '2017-07', '2017-08', '2018-06', '2018-07', '2018-08', '2019-07', '2019-08']}]}, {'dataProductCode': 'DP1.10043.001', 'dataProductTitle': 'Mosquitoes sampled from CO2 traps', 'availableMonths': ['2017-05', '2017-06', '2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2017-05', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP1.10043.001/NIWO/2021-12'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-05', '2017-06', '2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12']}]}, {'dataProductCode': 'DP1.10047.001', 'dataProductTitle': 'Soil physical and chemical properties, distributed initial characterization', 'availableMonths': ['2018-06', '2018-07'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10047.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10047.001/NIWO/2018-07'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2018-06', '2018-07']}]}, {'dataProductCode': 'DP1.10055.001', 'dataProductTitle': 'Plant phenology observations', 'availableMonths': ['2015-07', '2015-08', '2015-09', '2015-10', '2016-07', '2016-08', '2016-09', '2016-10', '2016-11', '2017-05', '2017-06', '2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2021-04', '2021-07'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2015-07', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2015-08', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2015-09', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2015-10', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2016-07', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2016-09', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2016-10', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2016-11', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2017-05', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP1.10055.001/NIWO/2021-07'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-04', '2021-07']}, {'release': 'RELEASE-2022', 'availableMonths': ['2015-07', '2015-08', '2015-09', '2015-10', '2016-07', '2016-08', '2016-09', '2016-10', '2016-11', '2017-05', '2017-06', '2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11']}]}, {'dataProductCode': 'DP1.10058.001', 'dataProductTitle': 'Plant presence and percent cover', 'availableMonths': ['2015-07', '2015-08', '2016-07', '2016-08', '2017-07', '2017-08', '2017-09', '2018-06', '2018-07', '2018-08', '2019-07', '2019-08', '2020-07', '2020-08', '2021-06', '2021-07', '2021-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2015-07', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2015-08', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2016-07', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.10058.001/NIWO/2021-08'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-06', '2021-07', '2021-08']}, {'release': 'RELEASE-2022', 'availableMonths': ['2015-07', '2015-08', '2016-07', '2016-08', '2017-07', '2017-08', '2017-09', '2018-06', '2018-07', '2018-08', '2019-07', '2019-08', '2020-07', '2020-08']}]}, {'dataProductCode': 'DP1.10064.001', 'dataProductTitle': 'Rodent pathogen status, hantavirus', 'availableMonths': ['2018-06', '2018-07', '2018-08', '2019-07', '2019-08', '2019-09', '2019-10'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10064.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10064.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10064.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10064.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10064.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10064.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.10064.001/NIWO/2019-10'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2018-06', '2018-07', '2018-08', '2019-07', '2019-08', '2019-09', '2019-10']}]}, {'dataProductCode': 'DP1.10066.001', 'dataProductTitle': 'Root biomass and chemistry, Megapit', 'availableMonths': ['2014-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10066.001/NIWO/2014-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2014-08']}]}, {'dataProductCode': 'DP1.10067.001', 'dataProductTitle': 'Root biomass and chemistry, periodic', 'availableMonths': ['2019-08', '2019-09'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10067.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10067.001/NIWO/2019-09'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2019-08', '2019-09']}]}, {'dataProductCode': 'DP1.10072.001', 'dataProductTitle': 'Small mammal box trapping', 'availableMonths': ['2015-06', '2015-07', '2015-08', '2015-09', '2015-10', '2016-07', '2016-08', '2016-09', '2017-06', '2017-07', '2017-09', '2018-06', '2018-07', '2018-08', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2021-06', '2021-07', '2021-08', '2021-09'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2015-06', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2015-07', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2015-08', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2015-09', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2015-10', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2016-07', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2016-09', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP1.10072.001/NIWO/2021-09'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-06', '2021-07', '2021-08', '2021-09']}, {'release': 'RELEASE-2022', 'availableMonths': ['2015-06', '2015-07', '2015-08', '2015-09', '2015-10', '2016-07', '2016-08', '2016-09', '2017-06', '2017-07', '2017-09', '2018-06', '2018-07', '2018-08', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10']}]}, {'dataProductCode': 'DP1.10076.001', 'dataProductTitle': 'Small mammal sequences DNA barcode', 'availableMonths': ['2016-07', '2016-08', '2016-09', '2017-06', '2017-07', '2017-09', '2018-06', '2018-07', '2018-08', '2019-07', '2019-08', '2019-09', '2020-07', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2016-07', 'https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2016-09', 'https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.10076.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2016-07', '2016-08', '2016-09', '2017-06', '2017-07', '2017-09', '2018-06', '2018-07', '2018-08', '2019-07', '2019-08', '2019-09', '2020-07', '2020-08']}]}, {'dataProductCode': 'DP1.10081.001', 'dataProductTitle': 'Soil microbe community composition', 'availableMonths': ['2016-06', '2016-08', '2016-10', '2017-06', '2017-07', '2017-08', '2017-09', '2018-06', '2018-07', '2018-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10081.001/NIWO/2016-06', 'https://data.neonscience.org/api/v0/data/DP1.10081.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10081.001/NIWO/2016-10', 'https://data.neonscience.org/api/v0/data/DP1.10081.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.10081.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10081.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10081.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.10081.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10081.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10081.001/NIWO/2018-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2016-06', '2016-08', '2016-10', '2017-06', '2017-07', '2017-08', '2017-09', '2018-06', '2018-07', '2018-08']}]}, {'dataProductCode': 'DP1.10086.001', 'dataProductTitle': 'Soil physical and chemical properties, periodic', 'availableMonths': ['2015-07', '2016-06', '2016-08', '2016-10', '2017-06', '2017-07', '2017-08', '2017-09', '2018-06', '2018-07', '2018-08', '2018-09', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2020-06', '2020-07', '2020-08', '2020-09', '2021-06', '2021-07', '2021-09'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2015-07', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2016-06', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2016-10', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.10086.001/NIWO/2021-09'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-06', '2021-07', '2021-09']}, {'release': 'RELEASE-2022', 'availableMonths': ['2015-07', '2016-06', '2016-08', '2016-10', '2017-06', '2017-07', '2017-08', '2017-09', '2018-06', '2018-07', '2018-08', '2018-09', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2020-06', '2020-07', '2020-08', '2020-09']}]}, {'dataProductCode': 'DP1.10093.001', 'dataProductTitle': 'Ticks sampled using drag cloths', 'availableMonths': ['2016-06', '2016-08', '2016-09', '2016-10', '2017-07', '2017-08', '2018-06', '2018-07', '2018-08', '2019-07', '2019-08', '2020-06', '2020-07', '2020-08', '2021-06', '2021-07', '2021-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2016-06', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2016-09', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2016-10', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP1.10093.001/NIWO/2021-08'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-06', '2021-07', '2021-08']}, {'release': 'RELEASE-2022', 'availableMonths': ['2016-06', '2016-08', '2016-09', '2016-10', '2017-07', '2017-08', '2018-06', '2018-07', '2018-08', '2019-07', '2019-08', '2020-06', '2020-07', '2020-08']}]}, {'dataProductCode': 'DP1.10098.001', 'dataProductTitle': 'Vegetation structure', 'availableMonths': ['2015-06', '2015-07', '2015-08', '2015-09', '2016-08', '2016-09', '2016-10', '2019-09', '2019-10', '2020-09', '2020-10', '2021-09', '2021-10'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10098.001/NIWO/2015-06', 'https://data.neonscience.org/api/v0/data/DP1.10098.001/NIWO/2015-07', 'https://data.neonscience.org/api/v0/data/DP1.10098.001/NIWO/2015-08', 'https://data.neonscience.org/api/v0/data/DP1.10098.001/NIWO/2015-09', 'https://data.neonscience.org/api/v0/data/DP1.10098.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10098.001/NIWO/2016-09', 'https://data.neonscience.org/api/v0/data/DP1.10098.001/NIWO/2016-10', 'https://data.neonscience.org/api/v0/data/DP1.10098.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.10098.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP1.10098.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP1.10098.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP1.10098.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP1.10098.001/NIWO/2021-10'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-09', '2021-10']}, {'release': 'RELEASE-2022', 'availableMonths': ['2015-06', '2015-07', '2015-08', '2015-09', '2016-08', '2016-09', '2016-10', '2019-09', '2019-10', '2020-09', '2020-10']}]}, {'dataProductCode': 'DP1.10104.001', 'dataProductTitle': 'Soil microbe biomass', 'availableMonths': ['2019-06', '2019-07', '2019-08', '2019-09', '2020-07', '2020-08', '2020-09'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10104.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.10104.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10104.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10104.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP1.10104.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP1.10104.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP1.10104.001/NIWO/2020-09'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2019-06', '2019-07', '2019-08', '2019-09', '2020-07', '2020-08', '2020-09']}]}, {'dataProductCode': 'DP1.10107.001', 'dataProductTitle': 'Soil microbe metagenome sequences', 'availableMonths': ['2016-08', '2017-07', '2017-08', '2018-07', '2018-08', '2019-07', '2019-08', '2020-07'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10107.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10107.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10107.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10107.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10107.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10107.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP1.10107.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.10107.001/NIWO/2020-07'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2020-07']}, {'release': 'RELEASE-2022', 'availableMonths': ['2016-08', '2017-07', '2017-08', '2018-07', '2018-08', '2019-07', '2019-08']}]}, {'dataProductCode': 'DP1.10108.001', 'dataProductTitle': 'Soil microbe marker gene sequences', 'availableMonths': ['2016-06', '2016-08', '2016-10', '2017-06', '2017-07', '2017-08', '2017-09', '2018-06', '2018-07', '2018-08', '2018-09', '2019-06', '2019-07'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10108.001/NIWO/2016-06', 'https://data.neonscience.org/api/v0/data/DP1.10108.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10108.001/NIWO/2016-10', 'https://data.neonscience.org/api/v0/data/DP1.10108.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.10108.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10108.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10108.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.10108.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP1.10108.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP1.10108.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.10108.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP1.10108.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP1.10108.001/NIWO/2019-07'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2019-06', '2019-07']}, {'release': 'RELEASE-2022', 'availableMonths': ['2016-06', '2016-08', '2016-10', '2017-06', '2017-07', '2017-08', '2017-09', '2018-06', '2018-07', '2018-08', '2018-09']}]}, {'dataProductCode': 'DP1.10109.001', 'dataProductTitle': 'Soil microbe group abundances', 'availableMonths': ['2016-06', '2016-08', '2016-10', '2017-06', '2017-07', '2017-08', '2017-09'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10109.001/NIWO/2016-06', 'https://data.neonscience.org/api/v0/data/DP1.10109.001/NIWO/2016-08', 'https://data.neonscience.org/api/v0/data/DP1.10109.001/NIWO/2016-10', 'https://data.neonscience.org/api/v0/data/DP1.10109.001/NIWO/2017-06', 'https://data.neonscience.org/api/v0/data/DP1.10109.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP1.10109.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP1.10109.001/NIWO/2017-09'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2016-06', '2016-08', '2016-10', '2017-06', '2017-07', '2017-08', '2017-09']}]}, {'dataProductCode': 'DP1.10111.001', 'dataProductTitle': 'Site management and event reporting', 'availableMonths': ['2015-05'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.10111.001/NIWO/2015-05'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2015-05']}]}, {'dataProductCode': 'DP1.30001.001', 'dataProductTitle': 'LiDAR slant range waveform', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.30001.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.30001.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.30001.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.30001.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP1.30003.001', 'dataProductTitle': 'Discrete return LiDAR point cloud', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.30003.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.30003.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.30003.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.30003.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP1.30006.001', 'dataProductTitle': 'Spectrometer orthorectified surface directional reflectance - flightline', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.30006.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.30006.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.30006.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.30006.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP1.30008.001', 'dataProductTitle': 'Spectrometer orthrorectified at-sensor radiance - flightline', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.30008.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.30008.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.30008.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.30008.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP1.30010.001', 'dataProductTitle': 'High-resolution orthorectified camera imagery', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP1.30010.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP1.30010.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP1.30010.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP1.30010.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP2.30011.001', 'dataProductTitle': 'Albedo - spectrometer - flightline', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP2.30011.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP2.30011.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP2.30011.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP2.30011.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP2.30012.001', 'dataProductTitle': 'LAI - spectrometer - flightline', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP2.30012.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP2.30012.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP2.30012.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP2.30012.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP2.30014.001', 'dataProductTitle': 'fPAR - spectrometer - flightline', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP2.30014.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP2.30014.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP2.30014.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP2.30014.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP2.30019.001', 'dataProductTitle': 'Canopy water indices - flightline', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP2.30019.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP2.30019.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP2.30019.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP2.30019.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP2.30026.001', 'dataProductTitle': 'Vegetation indices - spectrometer - flightline', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP2.30026.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP2.30026.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP2.30026.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP2.30026.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP3.30006.001', 'dataProductTitle': 'Spectrometer orthorectified surface directional reflectance - mosaic', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP3.30006.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP3.30006.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP3.30006.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP3.30006.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP3.30010.001', 'dataProductTitle': 'High-resolution orthorectified camera imagery mosaic', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP3.30010.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP3.30010.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP3.30010.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP3.30010.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP3.30011.001', 'dataProductTitle': 'Albedo - spectrometer - mosaic', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP3.30011.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP3.30011.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP3.30011.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP3.30011.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP3.30012.001', 'dataProductTitle': 'LAI - spectrometer - mosaic', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP3.30012.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP3.30012.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP3.30012.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP3.30012.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP3.30014.001', 'dataProductTitle': 'fPAR - spectrometer - mosaic', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP3.30014.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP3.30014.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP3.30014.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP3.30014.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP3.30015.001', 'dataProductTitle': 'Ecosystem structure', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP3.30015.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP3.30015.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP3.30015.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP3.30015.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP3.30019.001', 'dataProductTitle': 'Canopy water indices - mosaic', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP3.30019.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP3.30019.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP3.30019.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP3.30019.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP3.30024.001', 'dataProductTitle': 'Elevation - LiDAR', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP3.30024.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP3.30024.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP3.30024.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP3.30024.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP3.30025.001', 'dataProductTitle': 'Slope and Aspect - LiDAR', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP3.30025.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP3.30025.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP3.30025.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP3.30025.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP3.30026.001', 'dataProductTitle': 'Vegetation indices - spectrometer - mosaic', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP3.30026.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP3.30026.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP3.30026.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP3.30026.001/NIWO/2020-08'], 'availableReleases': [{'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2018-08', '2019-08', '2020-08']}]}, {'dataProductCode': 'DP4.00001.001', 'dataProductTitle': 'Summary weather statistics', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2017-07', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2017-08', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP4.00001.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-07', '2017-08', '2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}, {'dataProductCode': 'DP4.00200.001', 'dataProductTitle': 'Bundled data products - eddy covariance', 'availableMonths': ['2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06', '2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01'], 'availableDataUrls': ['https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2017-10', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2017-11', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2017-12', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2018-01', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2018-02', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2018-03', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2018-04', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2018-05', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2018-06', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2018-07', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2018-09', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2018-10', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2018-11', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2018-12', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2019-01', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2019-02', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2019-03', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2019-04', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2019-05', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2019-06', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2019-07', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2019-09', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2019-10', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2019-11', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2019-12', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2020-01', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2020-02', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2020-03', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2020-04', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2020-05', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2020-06', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2020-07', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2020-08', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2020-09', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2020-10', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2020-11', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2020-12', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2021-01', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2021-02', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2021-03', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2021-04', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2021-05', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2021-06', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2021-07', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2021-08', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2021-09', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2021-10', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2021-11', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2021-12', 'https://data.neonscience.org/api/v0/data/DP4.00200.001/NIWO/2022-01'], 'availableReleases': [{'release': 'PROVISIONAL', 'availableMonths': ['2021-07', '2021-08', '2021-09', '2021-10', '2021-11', '2021-12', '2022-01']}, {'release': 'RELEASE-2022', 'availableMonths': ['2017-09', '2017-10', '2017-11', '2017-12', '2018-01', '2018-02', '2018-03', '2018-04', '2018-05', '2018-06', '2018-07', '2018-08', '2018-09', '2018-10', '2018-11', '2018-12', '2019-01', '2019-02', '2019-03', '2019-04', '2019-05', '2019-06', '2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03', '2020-04', '2020-05', '2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11', '2020-12', '2021-01', '2021-02', '2021-03', '2021-04', '2021-05', '2021-06']}]}]}}

``` python
site_json['data'].keys()
```

    ## dict_keys(['siteCode', 'siteName', 'siteDescription', 'siteType', 'siteLatitude', 'siteLongitude', 'stateCode', 'stateName', 'domainCode', 'domainName', 'deimsId', 'releases', 'dataProducts'])

``` python
for product in site_json['data']['dataProducts']:
    print(product['dataProductCode'],product['dataProductTitle'])
    
```

    ## DP1.00001.001 2D wind speed and direction
    ## DP1.00002.001 Single aspirated air temperature
    ## DP1.00003.001 Triple aspirated air temperature
    ## DP1.00004.001 Barometric pressure
    ## DP1.00005.001 IR biological temperature
    ## DP1.00006.001 Precipitation
    ## DP1.00013.001 Wet deposition chemical analysis
    ## DP1.00014.001 Shortwave radiation (direct and diffuse pyranometer)
    ## DP1.00017.001 Dust and particulate size distribution
    ## DP1.00022.001 Shortwave radiation (primary pyranometer)
    ## DP1.00023.001 Shortwave and longwave radiation (net radiometer)
    ## DP1.00024.001 Photosynthetically active radiation (PAR)
    ## DP1.00033.001 Phenology images
    ## DP1.00038.001 Stable isotopes in precipitation
    ## DP1.00040.001 Soil heat flux plate
    ## DP1.00041.001 Soil temperature
    ## DP1.00042.001 Snow depth and understory phenology images
    ## DP1.00043.001 Spectral sun photometer - calibrated sky radiances
    ## DP1.00066.001 Photosynthetically active radiation (quantum line)
    ## DP1.00094.001 Soil water content and water salinity
    ## DP1.00095.001 Soil CO2 concentration
    ## DP1.00096.001 Soil physical and chemical properties, Megapit
    ## DP1.00098.001 Relative humidity
    ## DP1.00101.001 Particulate mass
    ## DP1.10003.001 Breeding landbird point counts
    ## DP1.10010.001 Coarse downed wood log survey
    ## DP1.10014.001 Coarse downed wood bulk density sampling
    ## DP1.10017.001 Digital hemispheric photos of plot vegetation
    ## DP1.10020.001 Ground beetle sequences DNA barcode
    ## DP1.10022.001 Ground beetles sampled from pitfall traps
    ## DP1.10023.001 Herbaceous clip harvest
    ## DP1.10026.001 Plant foliar traits
    ## DP1.10033.001 Litterfall and fine woody debris production and chemistry
    ## DP1.10038.001 Mosquito sequences DNA barcode
    ## DP1.10041.001 Mosquito pathogen status
    ## DP1.10043.001 Mosquitoes sampled from CO2 traps
    ## DP1.10047.001 Soil physical and chemical properties, distributed initial characterization
    ## DP1.10055.001 Plant phenology observations
    ## DP1.10058.001 Plant presence and percent cover
    ## DP1.10064.001 Rodent pathogen status, hantavirus
    ## DP1.10066.001 Root biomass and chemistry, Megapit
    ## DP1.10067.001 Root biomass and chemistry, periodic
    ## DP1.10072.001 Small mammal box trapping
    ## DP1.10076.001 Small mammal sequences DNA barcode
    ## DP1.10081.001 Soil microbe community composition
    ## DP1.10086.001 Soil physical and chemical properties, periodic
    ## DP1.10093.001 Ticks sampled using drag cloths
    ## DP1.10098.001 Vegetation structure
    ## DP1.10104.001 Soil microbe biomass
    ## DP1.10107.001 Soil microbe metagenome sequences
    ## DP1.10108.001 Soil microbe marker gene sequences
    ## DP1.10109.001 Soil microbe group abundances
    ## DP1.10111.001 Site management and event reporting
    ## DP1.30001.001 LiDAR slant range waveform
    ## DP1.30003.001 Discrete return LiDAR point cloud
    ## DP1.30006.001 Spectrometer orthorectified surface directional reflectance - flightline
    ## DP1.30008.001 Spectrometer orthrorectified at-sensor radiance - flightline
    ## DP1.30010.001 High-resolution orthorectified camera imagery
    ## DP2.30011.001 Albedo - spectrometer - flightline
    ## DP2.30012.001 LAI - spectrometer - flightline
    ## DP2.30014.001 fPAR - spectrometer - flightline
    ## DP2.30019.001 Canopy water indices - flightline
    ## DP2.30026.001 Vegetation indices - spectrometer - flightline
    ## DP3.30006.001 Spectrometer orthorectified surface directional reflectance - mosaic
    ## DP3.30010.001 High-resolution orthorectified camera imagery mosaic
    ## DP3.30011.001 Albedo - spectrometer - mosaic
    ## DP3.30012.001 LAI - spectrometer - mosaic
    ## DP3.30014.001 fPAR - spectrometer - mosaic
    ## DP3.30015.001 Ecosystem structure
    ## DP3.30019.001 Canopy water indices - mosaic
    ## DP3.30024.001 Elevation - LiDAR
    ## DP3.30025.001 Slope and Aspect - LiDAR
    ## DP3.30026.001 Vegetation indices - spectrometer - mosaic
    ## DP4.00001.001 Summary weather statistics
    ## DP4.00200.001 Bundled data products - eddy covariance

``` python
PRODUCTCODE = 'DP2.30011.001'
```

``` python
#Get available months of Breeding Landbird Count data products for TEAK site
#Loop through the 'dataProducts' list items (each one a dict) at the site
for product in site_json['data']['dataProducts']: 
    if(product['dataProductCode'] == PRODUCTCODE): #If a list item's 'dataProductCode' dict element equals the product code string,
        print('Available Months: ',product['availableMonths']) #print the available months and URLs
        print('URLs for each Month: ', product['availableDataUrls'])
```

    ## Available Months:  ['2017-09', '2018-08', '2019-08', '2020-08']
    ## URLs for each Month:  ['https://data.neonscience.org/api/v0/data/DP2.30011.001/NIWO/2017-09', 'https://data.neonscience.org/api/v0/data/DP2.30011.001/NIWO/2018-08', 'https://data.neonscience.org/api/v0/data/DP2.30011.001/NIWO/2019-08', 'https://data.neonscience.org/api/v0/data/DP2.30011.001/NIWO/2020-08']

``` python
#Make Request
data_request = requests.get(SERVER+'data/'+PRODUCTCODE+'/'+SITECODE+'/'+'2020-08')
data_json = data_request.json()

print(data_json)
```

    ## {'data': {'productCode': 'DP2.30011.001', 'siteCode': 'NIWO', 'month': '2020-08', 'release': 'RELEASE-2022', 'packages': [], 'files': [{'name': 'NEON_D13_NIWO_DP2_20200801_160156_ALBD.tif', 'size': 38642604, 'md5': None, 'crc32': '29bc1828', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_160156_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=06ac7546a78d5379b5dc05cb2cdbf1f390f973c4fa3655457bfe4520558c4371'}, {'name': 'NEON_D13_NIWO_DP2_20200720_174713_ALBD.tif', 'size': 28805904, 'md5': None, 'crc32': 'e3da049a', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_174713_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=19c74756a09a68c992d0d7b84914bddbc96aee49869dc29bd3577dd87e8eb213'}, {'name': 'NEON_D13_NIWO_DP2_20200807_154502_ALBD.tif', 'size': 57664080, 'md5': None, 'crc32': '64c29b25', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_154502_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=5678f5fa59296556d5d8b9423ab2fbd8ef31a17822869d3601fb46079defb2fd'}, {'name': '20200807_172100_hsi_kml_0000.kml', 'size': 86436, 'md5': None, 'crc32': 'de1c3aa8', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_172100_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=1effd8694038a79c2238b8c9ea1a10195038f4b9430d6e7701f5bcfb0f736835'}, {'name': '20200801_153959_hsi_kml_0000.kml', 'size': 96369, 'md5': None, 'crc32': '6ac41d0b', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_153959_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=5de7fae4dbfb1086b88146a18278e6ee309d9f574bbc9cd4316750f9d409263f'}, {'name': '20200720_173638_hsi_kml_0000.kml', 'size': 64811, 'md5': None, 'crc32': '5b4fe6dd', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_173638_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=234c7556d550fd5f96e968d8d34d24555ecce33d2486bce080cdf60a7f5f589f'}, {'name': 'NEON_D13_NIWO_DP2_20200720_154719_ALBD.tif', 'size': 43899972, 'md5': None, 'crc32': '9281b3b3', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_154719_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=2f197c17f820a484e04ffc7f0696ff42535ded440f912100afacae1495d7957b'}, {'name': '20200720_171552_hsi_kml_0000.kml', 'size': 65285, 'md5': None, 'crc32': '59d6bc99', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_171552_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=c0ff2137ce2e378d67973a4726014168d687995679f844ccc3c80871cc00e397'}, {'name': '20200807_164808_hsi_kml_0000.kml', 'size': 86916, 'md5': None, 'crc32': 'cbc002c0', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_164808_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=7318a4f849757d2540867ff495be88a8db1ede6546a67bb0b0102e36c6bb4c81'}, {'name': 'NEON_D13_NIWO_DP2_20200731_154336_ALBD.tif', 'size': 39327772, 'md5': None, 'crc32': '48811ba8', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020073114/NEON_D13_NIWO_DP2_20200731_154336_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=bd74c2253305aa544f3654564341f3ce85331f7275df418c32ce670ceaeb91f4'}, {'name': 'NEON_D13_NIWO_DP2_20200807_162134_ALBD.tif', 'size': 39824460, 'md5': None, 'crc32': 'da2319ca', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_162134_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=cd1c3678a97f89f3b664a66134b470eb7819e11dbe6475182aff12e07f248aab'}, {'name': '20200720_174713_hsi_kml_0000.kml', 'size': 57367, 'md5': None, 'crc32': 'e9c3255e', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_174713_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=498f8b4def621e55cda5aedb78415012513d0f1f2a949825b43ea56d04ce9128'}, {'name': '20200720_154719_hsi_kml_0000.kml', 'size': 99635, 'md5': None, 'crc32': 'a6d9b08a', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_154719_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=2ff989ec3fcf1c22a67d38cffaab462b77f5d77e6d6555ae13f78628365b68c5'}, {'name': 'NEON.D13.NIWO.DP2.30011.001.readme.20220302T205717Z.txt', 'size': 11097, 'md5': 'a70e6306cb0d644131ea81acbd210cc2', 'crc32': None, 'url': 'https://neon-prod-pub-1.s3.data.neonscience.org/release/tag/RELEASE-2022A/NEON.DOM.SITE.DP2.30011.001/NIWO/20200801T000000--20200901T000000/basic/NEON.D13.NIWO.DP2.30011.001.readme.20220302T205717Z.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=d68268504cc6fafaad1d073573727017147a17348dc388c3747fce7465194466'}, {'name': '20200720_165434_hsi_kml_0000.kml', 'size': 57583, 'md5': None, 'crc32': '598bd72e', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_165434_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=b44a3696436d76a95b00df8b2bcf74417f2114f4d586b63024a4c9abf67df936'}, {'name': 'NEON_D13_NIWO_DP2_20200720_175213_ALBD.tif', 'size': 28402140, 'md5': None, 'crc32': '11d1e68b', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_175213_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=323139c2904827be885e0f6ea5163a1aa7a42a9098a216dd67b20cdcae67c2c6'}, {'name': 'NEON_D13_NIWO_DP2_20200801_153959_ALBD.tif', 'size': 36437144, 'md5': None, 'crc32': '253a1107', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_153959_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=20bb41da360db8467ff27edcf1c7cdefa0597ab0d2514b56473435937a83cc69'}, {'name': '20200807_155314_hsi_kml_0000.kml', 'size': 86727, 'md5': None, 'crc32': '0f733a96', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_155314_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=6baae473256a0cdd82eadd29512e0d5737aece537cfd4ee90147af6613d67598'}, {'name': '20200720_161640_hsi_kml_0000.kml', 'size': 103688, 'md5': None, 'crc32': '1492b156', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_161640_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=68f639bf594273b49f84e4a51c09ae9ef62d4c4fec7fe04f0d0fb82745fba63d'}, {'name': '20200807_160118_hsi_kml_0000.kml', 'size': 89464, 'md5': None, 'crc32': 'ae83ed04', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_160118_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=466e7d1b87cf594b8f156f4f1548a6e71e7ec37ba6a4d9dedcfe54196c318d0b'}, {'name': '20200807_162134_hsi_kml_0000.kml', 'size': 86142, 'md5': None, 'crc32': '08ae1a9f', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_162134_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=8556053606de2b4cd4b552ef32efd5c19e47721531f73d39a6c46b375e0b214a'}, {'name': '20200720_172208_hsi_kml_0000.kml', 'size': 60804, 'md5': None, 'crc32': '9b30f65f', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_172208_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=5f6f8e4ae11f011b0df875fc21ac0e14f92b68ec3d8fae31a5985e87159f1353'}, {'name': '20200807_170802_hsi_kml_0000.kml', 'size': 82406, 'md5': None, 'crc32': '17d4aa4f', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_170802_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=dbd7b3bb4c37bf33c6091b73fa12f038e69e554e359416893db0f2cf796eb060'}, {'name': 'NEON_D13_NIWO_DP2_20200720_173121_ALBD.tif', 'size': 21724904, 'md5': None, 'crc32': '747cab6e', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_173121_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=14c97600a2628f0052e7ca4c2e2b474e23fa2c75b9af07d5a9f04484aa9d16d0'}, {'name': 'NEON_D13_NIWO_DP2_20200720_180514_ALBD.tif', 'size': 63612396, 'md5': None, 'crc32': '90ed6075', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_180514_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=79f6062316683f0ad6f25f2e5c9fe3c2675d319b279ea2514b03857b64e581db'}, {'name': 'NEON_D13_NIWO_DP2_20200720_172208_ALBD.tif', 'size': 27488556, 'md5': None, 'crc32': '95f2f06f', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_172208_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=74083f2699c61c123d5ae67fef4c62b21c8cd0c50e0eb73b4a6b8a4e4e8b787e'}, {'name': '20200801_152930_hsi_kml_0000.kml', 'size': 81447, 'md5': None, 'crc32': '948af222', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_152930_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=7b038cf7242f1e11c8468dad3f8993dffc9c3efe1175e61eb75bc790d5b4d83f'}, {'name': '20200720_173121_hsi_kml_0000.kml', 'size': 68313, 'md5': None, 'crc32': '5c4068a1', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_173121_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=bb2e05f7d569f0ed10cc9c71853369023a795a36433a9895f3beafa7a0033a76'}, {'name': 'NEON_D13_NIWO_DP2_20200807_164154_ALBD.tif', 'size': 43818724, 'md5': None, 'crc32': 'c63cb4f0', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_164154_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=5c3d256c22baf62e66b11a84445d6514aae23f3bec10ac39554de6b8d257e820'}, {'name': 'NEON_D13_NIWO_DP2_20200801_163340_ALBD.tif', 'size': 45453708, 'md5': None, 'crc32': '0d036c36', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_163340_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=10510c5dbc329caa454957bb38badaba72e52aff7e4b16ff6a2796c46f1a8cec'}, {'name': 'NEON_D13_NIWO_DP2_20200807_165500_ALBD.tif', 'size': 41225052, 'md5': None, 'crc32': '9d3c9b23', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_165500_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=148541cd256cd58328f1499653ce804f2c5b1496aa3b74a8c19c870f22020baf'}, {'name': 'NEON_D13_NIWO_DP2_20200807_162829_ALBD.tif', 'size': 45339260, 'md5': None, 'crc32': '877d4bc3', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_162829_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f38f907c39f81e8a7eee5809002bdf232025ef4f13f56e119cd4255d6d28e51a'}, {'name': 'NEON_D13_NIWO_DP2_20200807_164808_ALBD.tif', 'size': 37719276, 'md5': None, 'crc32': 'fde41c25', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_164808_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=19a389a0074959a9eeedc0c902faaab016d2777cbff3c2e5ce96022e5ad5487a'}, {'name': '20200807_160712_hsi_kml_0000.kml', 'size': 88457, 'md5': None, 'crc32': '15a16625', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_160712_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f72356d7e60e6d08c5892b70ce3e10cc58544950ba7849fc2809d6ae1e5426ae'}, {'name': '2020_NIWO_4_NIWO_L3_spectrometer_processing.pdf', 'size': 10756903, 'md5': None, 'crc32': '508637ea', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/2020_NIWO_4_NIWO_L3_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=94e54e16959f043c9de8ab6cd0c5b6e120505a60698347278a292f2bd5a9ed83'}, {'name': '20200807_165500_hsi_kml_0000.kml', 'size': 86905, 'md5': None, 'crc32': 'bd5efef0', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_165500_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=31536a3a211a08719c4cde993391cb6678bd08eb12c0e20e1e97e02f760b8828'}, {'name': '20200801_155424_hsi_kml_0000.kml', 'size': 95980, 'md5': None, 'crc32': '6826f0e5', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_155424_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=4863c49cbce6f6017de2e22a5d41e44ee1f4c6299efd06139d7802d995c92129'}, {'name': '20200720_175809_hsi_kml_0000.kml', 'size': 91955, 'md5': None, 'crc32': '0115faa7', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_175809_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=73b1eefa9a931ec9396b9408cfbe278324339f8d43a28adb448ccadf8ec62be4'}, {'name': 'NEON_D13_NIWO_DP2_20200807_170802_ALBD.tif', 'size': 41780604, 'md5': None, 'crc32': '8513245f', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_170802_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=96b88cc9593c07566208ff953f5fc112471b58a6d78c71463736b0a6aca05601'}, {'name': 'NEON_D13_NIWO_DP2_20200801_162711_ALBD.tif', 'size': 46640088, 'md5': None, 'crc32': '9e9af92e', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_162711_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=4216e84017603bf995f53cd84545712bcef7907c37081e0791709d8fc95d8dd4'}, {'name': 'NIWO_2020072014_L1_spectrometer_processing.pdf', 'size': 1878105, 'md5': None, 'crc32': '1b2cd1e1', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/NIWO_2020072014_L1_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=b23a2e78b1e4a26452f63096e80cbc5cd128185590b60cf6041041de33713a72'}, {'name': '2020_NIWO_4_2020073114_P2C1_L2_spectrometer_processing.pdf', 'size': 5288295, 'md5': None, 'crc32': '4d2d5604', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/2020_NIWO_4_2020073114_P2C1_L2_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=e831345c73edea0699f6e5989278d96a467bba8d936f113e531bd4182b3f3fc4'}, {'name': 'NEON_D13_NIWO_DP2_20200807_163444_ALBD.tif', 'size': 47069132, 'md5': None, 'crc32': '07b39945', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_163444_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=8074b7aa7101d2520b32ff018772b7341e81cfb7e19939b3214a05278238d085'}, {'name': 'NEON_D13_NIWO_DP2_20200801_152930_ALBD.tif', 'size': 46156596, 'md5': None, 'crc32': '6b5823a1', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_152930_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a7aba67a05e94696a53255ffd7bdff9b06816b7388916c8d24ca8261f71f1514'}, {'name': '20200720_163955_hsi_kml_0000.kml', 'size': 94161, 'md5': None, 'crc32': '6bcda0c7', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_163955_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=13ae1a9ca09f3935155420de9a963b9929e23ea84c318266aa6f6ee3f83e582e'}, {'name': '20200720_165934_hsi_kml_0000.kml', 'size': 57290, 'md5': None, 'crc32': 'c7272a7e', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_165934_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=b84f3d5a4aa5175b8ebe6eeb0c03129dd4fb45ca31b2a458290ebf2f4adbb499'}, {'name': 'NEON_D13_NIWO_DP2_20200720_181654_ALBD.tif', 'size': 59266476, 'md5': None, 'crc32': 'f2ab3328', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_181654_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=aed1a68600441a392d43186eecd0dfaf245c415d0b55980e9eb339a734f37c38'}, {'name': '2020_NIWO_4_2020072014_P2C1_L2_spectrometer_processing.pdf', 'size': 7630749, 'md5': None, 'crc32': 'fb18a236', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/2020_NIWO_4_2020072014_P2C1_L2_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=aa82a9e1372b92770bde10426665ae06681fbb7f4fcdf04ed7215adc98f6326d'}, {'name': 'NEON_D13_NIWO_DP2_20200731_155024_ALBD.tif', 'size': 46022612, 'md5': None, 'crc32': 'f1f232e9', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020073114/NEON_D13_NIWO_DP2_20200731_155024_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=955991d9759eda5421b086df015ea11587f3b79b7bb341b0352975a57028ac14'}, {'name': 'NEON_D13_NIWO_DP2_20200720_161640_ALBD.tif', 'size': 44265996, 'md5': None, 'crc32': '1e6539a8', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_161640_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=4e2aedb47162996b784173c819c7f5ccd3d0c07c46b1851019734b421ff7fd2f'}, {'name': 'NEON_D13_NIWO_DP2_20200731_151902_ALBD.tif', 'size': 61566796, 'md5': None, 'crc32': '44fef28d', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020073114/NEON_D13_NIWO_DP2_20200731_151902_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=ecef8be215022c6bf621b5c00c50dc7cf2a1197b2fc852f62ec060d421421f8c'}, {'name': 'NEON_D13_NIWO_DP2_20200807_161351_ALBD.tif', 'size': 48779984, 'md5': None, 'crc32': 'ea61f89e', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_161351_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=272e76d52e806169398b9d8f259bc32475331cb1641788bc71942cc7e4c7d0fa'}, {'name': '20200720_175213_hsi_kml_0000.kml', 'size': 67545, 'md5': None, 'crc32': '2f50ff15', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_175213_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a911e7b69a8b33da58dc38ee8c8d9c2ce7f0063528b26c06f78f5b3ec0b8d556'}, {'name': '20200720_181208_hsi_kml_0000.kml', 'size': 47537, 'md5': None, 'crc32': '6259bd59', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_181208_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=38a4256dc6e5edcae6a270d11a24471b65bee7f8180543fd74fcab629f902870'}, {'name': '20200807_163444_hsi_kml_0000.kml', 'size': 86263, 'md5': None, 'crc32': 'ab70059e', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_163444_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f31e4f0115477b5545421532d2aa0ab5fad6768482858b6d22f86f23d59c6ff6'}, {'name': 'NIWO_2020080114_L1_spectrometer_processing.pdf', 'size': 1908264, 'md5': None, 'crc32': '7210a099', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/NIWO_2020080114_L1_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=22cb1ab21d4b8160d0ab86c2adeba46a0cab55320c848dcacc1b263ecb0d4c14'}, {'name': '20200720_181654_hsi_kml_0000.kml', 'size': 109293, 'md5': None, 'crc32': '918390bd', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_181654_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=c5f5d8764cdc1b3270abbdc09c755e3cc5daa6521761c8e75578c5451930fddf'}, {'name': 'NEON_D13_NIWO_DP2_20200801_155424_ALBD.tif', 'size': 38847116, 'md5': None, 'crc32': '526a2459', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_155424_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=8c43fd36c127ec761253738ef81172f384a3ca8d85921b1029e34158d3d9c67d'}, {'name': 'NIWO_2020073114_L1_spectrometer_processing.pdf', 'size': 1845176, 'md5': None, 'crc32': 'e9c01aac', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/NIWO_2020073114_L1_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=8ee873f54c4fb91525f7b37f91a948fa2b305baa660e3ac3295f28b30145116c'}, {'name': 'NEON_D13_NIWO_DP2_20200720_164656_ALBD.tif', 'size': 25740444, 'md5': None, 'crc32': '5d35dcdc', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_164656_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=36d2ea76b3603f42ebce5d21260817590db68266c5ec6d146b4a4edc144bab89'}, {'name': 'NEON_D13_NIWO_DP2_20200720_160107_ALBD.tif', 'size': 40368396, 'md5': None, 'crc32': '1c23ca26', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_160107_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=15a201252bd0be3350dea392694ff85e32edc626ae527eadee0d857165957f69'}, {'name': 'NEON_D13_NIWO_DP2_20200807_170127_ALBD.tif', 'size': 43046296, 'md5': None, 'crc32': '83e5c5de', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_170127_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=163aefb2915450aed65656a8fc5ed6fb00a9ebed9d925b0a576f9c196f90be7f'}, {'name': 'NEON_D13_NIWO_DP2_20200801_162052_ALBD.tif', 'size': 46864744, 'md5': None, 'crc32': '036ba1a5', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_162052_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=b0ddc8be869ef48c63ffcf06d5512a7d17c8fcaa62266948cf476908a1a83935'}, {'name': '20200807_162829_hsi_kml_0000.kml', 'size': 89038, 'md5': None, 'crc32': 'a0edd785', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_162829_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=0e22ff591a85f35a07f8a8647280115263b0dc504be566024ae371e4f9f7d5c3'}, {'name': '20200720_180514_hsi_kml_0000.kml', 'size': 110205, 'md5': None, 'crc32': 'ff71fc5d', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_180514_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a2fc3208800ed7bbe9ddd5a175c257876fec80267febdbca75a9fa1e9675eb81'}, {'name': 'NEON_D13_NIWO_DP2_20200720_171113_ALBD.tif', 'size': 27175960, 'md5': None, 'crc32': '07b8277e', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_171113_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=31bccbf4111dda9d8359282c5ded8c53f203a0ddd04bb5d4d4bffd7bc1b45330'}, {'name': 'NEON_D13_NIWO_DP2_20200731_155646_ALBD.tif', 'size': 37108680, 'md5': None, 'crc32': '773c8948', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020073114/NEON_D13_NIWO_DP2_20200731_155646_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=eaa543f0d6f399ff884b47f03c98bc1b3f361ba2aafcac02a5b4e2f3f0bd7297'}, {'name': '20200731_151902_hsi_kml_0000.kml', 'size': 99179, 'md5': None, 'crc32': 'c4e83492', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200731_151902_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=747a22ac2dd3c9e8ac5de0d7d5ee257a4d77179f708adb635680a64692331f9e'}, {'name': '20200801_162711_hsi_kml_0000.kml', 'size': 94102, 'md5': None, 'crc32': '1422751c', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_162711_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f52eb8ca2f1ef5e79ef36670292377cc9802436b9fceda5e07011ca07837fbd4'}, {'name': '20200731_154336_hsi_kml_0000.kml', 'size': 109300, 'md5': None, 'crc32': '5c52aea3', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200731_154336_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=64079ae8b6fdc19479bb5c1435254a2270cee9c22e3a53f3ca1302380e499a20'}, {'name': '20200807_171424_hsi_kml_0000.kml', 'size': 81434, 'md5': None, 'crc32': '9ed62eb8', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_171424_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=8b053519adb5d76449de5048f0ff205b3ce504d13fefab053db1d4e8a383072a'}, {'name': 'NEON_D13_NIWO_DP2_20200720_163955_ALBD.tif', 'size': 42262236, 'md5': None, 'crc32': '5f83fdae', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_163955_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=41ff285eb6a890e61926b3ca5c9c17a6bdc743489df124872a9f8c0393aaa3b5'}, {'name': '20200720_163210_hsi_kml_0000.kml', 'size': 105799, 'md5': None, 'crc32': '0af7718e', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_163210_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=fab950f931faf3ac77412a56f664b8fc80eafcb60d6dd1a9001818cc9d01015e'}, {'name': '20200807_154502_hsi_kml_0000.kml', 'size': 72216, 'md5': None, 'crc32': 'f955614a', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_154502_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=d6ca15990c55eda4ab5b15410046e8461ffe1f8fe284aed05298a12c6639951d'}, {'name': '20200720_160107_hsi_kml_0000.kml', 'size': 106434, 'md5': None, 'crc32': '2858b1b9', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_160107_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=aa2d46612e963778502c63afd827e334fdadd52aae1ac06f33edb28927fd0a18'}, {'name': '20200801_162052_hsi_kml_0000.kml', 'size': 93345, 'md5': None, 'crc32': 'c7e6ece0', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_162052_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=685e566966a6612ac25fd9f47048eddecc45f50f8fd05eaefd1359cd217ef1dd'}, {'name': '20200720_174206_hsi_kml_0000.kml', 'size': 61724, 'md5': None, 'crc32': '2e26de6d', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_174206_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=9ed597ed6d3748274c98e9e9791d0128b9ae8f789f7b0a7ed32451ac3fabc25f'}, {'name': '20200807_170127_hsi_kml_0000.kml', 'size': 83951, 'md5': None, 'crc32': '0e33d07c', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_170127_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=b755de3501da7b22f734538367d31ea1ce3a9229c95c9d44800bf7a893d284a8'}, {'name': 'NEON_D13_NIWO_DP2_20200720_163210_ALBD.tif', 'size': 42240108, 'md5': None, 'crc32': 'd0e8285e', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_163210_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=11a19690696383acf33d5ce55584404c82e7da5983dbeb062bb56cd78ea21139'}, {'name': 'NEON_D13_NIWO_DP2_20200720_171552_ALBD.tif', 'size': 29202540, 'md5': None, 'crc32': 'ff31ee58', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_171552_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=8a437c99fe8d3f39fbf8b492d7fa31a604f358c6bcc575f0c68ae6dbc71596e5'}, {'name': '20200801_163340_hsi_kml_0000.kml', 'size': 95590, 'md5': None, 'crc32': 'ef59e0ab', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_163340_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f889e7deef2db680b94b901ebe766389c0e65e7ea792386feb906d2b52eb06fa'}, {'name': 'NIWO_2020080714_L1_spectrometer_processing.pdf', 'size': 1877999, 'md5': None, 'crc32': '7e69a3c1', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/NIWO_2020080714_L1_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=509b0bb4698435a3e29bce3c58de6468d51ac538213188e3f7b2813678466798'}, {'name': '20200801_154719_hsi_kml_0000.kml', 'size': 98791, 'md5': None, 'crc32': '271fb78a', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_154719_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=b5b1ffb62635dd44604a1d25e4b9d74693f813d36b59866b3651be6071200bb8'}, {'name': 'NEON_D13_NIWO_DP2_20200720_155401_ALBD.tif', 'size': 48346876, 'md5': None, 'crc32': 'cac96623', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_155401_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=9870586468e178d3f63a2fc4377b0f748313b884ecf8645a516c73051f1be820'}, {'name': 'NEON_D13_NIWO_DP2_20200720_172628_ALBD.tif', 'size': 26367660, 'md5': None, 'crc32': '681579d1', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_172628_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=bd01a2109fa79fcc9bda142ba50363e360c0ea48635b49d7c57a4fa42ce2b02d'}, {'name': '20200720_171113_hsi_kml_0000.kml', 'size': 69914, 'md5': None, 'crc32': '2ecba22e', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_171113_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=13eea8615de51246c933dad1b5aee243e1046a9d2a46b249f86a2651bfd545b0'}, {'name': 'NEON_D13_NIWO_DP2_20200801_160814_ALBD.tif', 'size': 41934156, 'md5': None, 'crc32': 'cb447f1b', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_160814_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=989e6142084164ec2a234545dc38bb0cee9594c7331d8abd83e02644e5237707'}, {'name': '20200731_155646_hsi_kml_0000.kml', 'size': 109728, 'md5': None, 'crc32': 'f26eaaeb', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200731_155646_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=2b5400ac7bd3da1a0e7810b680d7bcbaf0c91fc1cb28c049e3d1cf1c327b55f5'}, {'name': '20200720_162438_hsi_kml_0000.kml', 'size': 99368, 'md5': None, 'crc32': '760937e5', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_162438_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a51432d7fca8c25c3520bc2e401d54b794acc74ee73d537c0f4d618d40a169be'}, {'name': 'NEON_D13_NIWO_DP2_20200731_153721_ALBD.tif', 'size': 38072628, 'md5': None, 'crc32': 'e88ccd5f', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020073114/NEON_D13_NIWO_DP2_20200731_153721_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=20f843568d001f9b30053dfe1dd731d77c62f0f0c81d155fb61b56975e745fbd'}, {'name': 'NEON_D13_NIWO_DP2_20200801_161441_ALBD.tif', 'size': 46276740, 'md5': None, 'crc32': 'b96086b9', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_161441_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=25e0908160dcd290faefbc6ad0369d6d206fa38dcb02b9057d2369b2562c02a8'}, {'name': 'NEON_D13_NIWO_DP2_20200720_173638_ALBD.tif', 'size': 28996344, 'md5': None, 'crc32': '52ebc3e1', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_173638_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=88d098870d454343b0e6ea385fd3c761a950ccf81e61712e287afbea39afb641'}, {'name': '2020_NIWO_4_2020080114_P2C1_L2_spectrometer_processing.pdf', 'size': 7040620, 'md5': None, 'crc32': 'd6a77ebc', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/2020_NIWO_4_2020080114_P2C1_L2_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a13de3ac0bf0223ce8a10ae745aec9f47382d115e9a52959553d358f4e247b58'}, {'name': 'NEON_D13_NIWO_DP2_20200807_160118_ALBD.tif', 'size': 52812980, 'md5': None, 'crc32': 'c18c7f41', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_160118_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=012774a63c748669121aae89561de7f2f00b1a70cb95ba3e08b352e20012f65a'}, {'name': '20200731_153721_hsi_kml_0000.kml', 'size': 91416, 'md5': None, 'crc32': '7763d03b', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200731_153721_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=88503bb3e3b724ce23e171ba59ed0f9c7383924699c5806a47bbff6bf6a42ed1'}, {'name': 'NEON_D13_NIWO_DP2_20200720_162438_ALBD.tif', 'size': 50897652, 'md5': None, 'crc32': 'c31cee40', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_162438_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a2de8e771c02e45cbde335bb238217da175e4b7e687947e54fd07cc47a581227'}, {'name': '20200720_155401_hsi_kml_0000.kml', 'size': 91761, 'md5': None, 'crc32': '8ee36152', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_155401_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=35d24fdcf37a758645a9eaae2f6b4f331f9528e0cd6dc962ada0f1f1c709175f'}, {'name': '20200720_170523_hsi_kml_0000.kml', 'size': 55008, 'md5': None, 'crc32': '1d8c11de', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_170523_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=912f6e1ddeb2dbc19c148d6c56111dc89634c4fefc430d3fe266137584464c45'}, {'name': 'NEON_D13_NIWO_DP2_20200720_160855_ALBD.tif', 'size': 39287724, 'md5': None, 'crc32': '11a19050', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_160855_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=9e86a319cc4765e162acd3785f1ab593956a1977ab8700bd6d1f7265c8f37411'}, {'name': 'NEON_D13_NIWO_DP2_20200807_155314_ALBD.tif', 'size': 46807296, 'md5': None, 'crc32': '02e08db4', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_155314_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=cb88d962daf1ab367b59c949a36a8fb8c1d5a148586d9935ef9f8ebd9c7583fa'}, {'name': 'NEON_D13_NIWO_DP2_20200720_181208_ALBD.tif', 'size': 17270364, 'md5': None, 'crc32': '852671de', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_181208_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=36459a0f9d09d8eadcec7c7e9c760a3d0aded7e0e20bceab4e79acd35824d1bc'}, {'name': '20200801_160156_hsi_kml_0000.kml', 'size': 101356, 'md5': None, 'crc32': '71e4ab2e', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_160156_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f2350bf3adc2b82f8342f432dab136c1d42bdc325301d9f8929e3c88b5f9cb4b'}, {'name': 'NEON_D13_NIWO_DP2_20200720_154017_ALBD.tif', 'size': 48905196, 'md5': None, 'crc32': '29c31e4d', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_154017_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=547890fd7da4ca659dea5a2e0fdb2199ec6c895ffeb8bdaa287974cd65a55e30'}, {'name': '2020_NIWO_4_2020080714_P2C1_L2_spectrometer_processing.pdf', 'size': 8549613, 'md5': None, 'crc32': 'd0d5b7e0', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/2020_NIWO_4_2020080714_P2C1_L2_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f1a50d6c3fd7116c05ca44e2722b7e2c159ce859797a76ca28adb3e721efdbd9'}, {'name': 'NEON_D13_NIWO_DP2_20200720_165934_ALBD.tif', 'size': 25473820, 'md5': None, 'crc32': '93b540f6', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_165934_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=e4e3b667d45df19a420d32648ffa3c1a5b6eefc99cdecd15f9b49ce158d59109'}, {'name': 'NEON_D13_NIWO_DP2_20200720_175809_ALBD.tif', 'size': 46057708, 'md5': None, 'crc32': 'be88f2dd', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_175809_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=4efa2c78c704e086628d40de644781413e1b80971752874ceebbb4b567126e12'}, {'name': '20200807_161351_hsi_kml_0000.kml', 'size': 86314, 'md5': None, 'crc32': '7e073854', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_161351_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=c495fd930d458df488396bfcbbf6bafe1dc6c32771843dea02a76fcec916a98a'}, {'name': '20200807_164154_hsi_kml_0000.kml', 'size': 91130, 'md5': None, 'crc32': 'a0385e7e', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_164154_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=0a5f414ee308b506e4a3c909c12c05c7408e574ca747c35276476475c7e317b0'}, {'name': 'NEON_D13_NIWO_DP2_20200720_170523_ALBD.tif', 'size': 26911436, 'md5': None, 'crc32': '8ce1f0c1', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_170523_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=ccc8bc65d3fd9ecb744d1368d5852b65b863859100a18e9dda7f071040cea8ce'}, {'name': 'NEON_D13_NIWO_DP2_20200720_165434_ALBD.tif', 'size': 31206476, 'md5': None, 'crc32': '61546397', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_165434_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=50acfc629a1ac096529028b1a567897dd7375bbff22794e0f64f709af064fb0a'}, {'name': '20200720_164656_hsi_kml_0000.kml', 'size': 65526, 'md5': None, 'crc32': '85290801', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_164656_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=1b3ca0832efdadd84267f6569d04ed30bd9b4202ba5d69ed9b08af5aafd64559'}, {'name': '20200720_160855_hsi_kml_0000.kml', 'size': 104058, 'md5': None, 'crc32': 'b74d9038', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_160855_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=ed2350095922e631b97132c24e659c25efac8513b7f9b8e2acb5f82d27270bf0'}, {'name': 'NEON_D13_NIWO_DP2_20200807_172100_ALBD.tif', 'size': 43666956, 'md5': None, 'crc32': 'ff979f92', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_172100_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=7d2c2709fa4c2788f6a87f20c57faad15e92293118cf20271bfd780811fd9e4e'}, {'name': 'NEON_D13_NIWO_DP2_20200801_154719_ALBD.tif', 'size': 42130636, 'md5': None, 'crc32': '22cb8329', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_154719_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=2842e570a57e4df0bcd80fdfe844481a45a62fb9e9a7e0ad7633153a16918017'}, {'name': '20200720_172628_hsi_kml_0000.kml', 'size': 57392, 'md5': None, 'crc32': '540355d4', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_172628_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=564d0e5c5b8646d1e121385794a08347c0b57e270ad681610b89a24aaf2a2f12'}, {'name': '20200720_154017_hsi_kml_0000.kml', 'size': 96453, 'md5': None, 'crc32': 'cabba58f', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_154017_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=bde4e8781930393da84b0746fee4a9b9854a4f5f35532863998f12d0c1a51d60'}, {'name': 'NEON_D13_NIWO_DP2_20200807_160712_ALBD.tif', 'size': 50044236, 'md5': None, 'crc32': '770836d1', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_160712_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a67b42081da565f332fdee2588f7767638f36e84ce9a92fa0901492984a8713d'}, {'name': 'NEON_D13_NIWO_DP2_20200720_174206_ALBD.tif', 'size': 23881984, 'md5': None, 'crc32': 'f896242f', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_174206_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=0539c24ca289a615308c1a6fb32b8e29a6266d2fd8cfdb1197ffada9dc76d0c9'}, {'name': '20200731_155024_hsi_kml_0000.kml', 'size': 96798, 'md5': None, 'crc32': 'f2230690', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200731_155024_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=6ba7562e4454a412d058ab569d96734c9842e424543265e5248470834387b71c'}, {'name': '20200801_161441_hsi_kml_0000.kml', 'size': 97229, 'md5': None, 'crc32': '5e6f7bd6', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_161441_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=afe77781ef2cbfc3159d64e873e97a8241cb4bab5a201c5c3f953c1aeb529f39'}, {'name': 'NEON_D13_NIWO_DP2_20200807_171424_ALBD.tif', 'size': 37936956, 'md5': None, 'crc32': 'fcb47a6f', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_171424_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=64cf26b4325efb08e12fb557fd03648ceeae15cae7b6a4cf2ad1213cba27e987'}, {'name': '20200801_160814_hsi_kml_0000.kml', 'size': 93587, 'md5': None, 'crc32': '15834bdb', 'url': 'https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_160814_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=9c16851c44327d3f5dfb68a8fc20a736f5890ef46a1d7c196f1dcbff78a7674e'}]}}

``` r
py$data_json
```

    ## $data
    ## $data$productCode
    ## [1] "DP2.30011.001"
    ## 
    ## $data$siteCode
    ## [1] "NIWO"
    ## 
    ## $data$month
    ## [1] "2020-08"
    ## 
    ## $data$release
    ## [1] "RELEASE-2022"
    ## 
    ## $data$packages
    ## list()
    ## 
    ## $data$files
    ## $data$files[[1]]
    ## $data$files[[1]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200801_160156_ALBD.tif"
    ## 
    ## $data$files[[1]]$size
    ## [1] 38642604
    ## 
    ## $data$files[[1]]$md5
    ## NULL
    ## 
    ## $data$files[[1]]$crc32
    ## [1] "29bc1828"
    ## 
    ## $data$files[[1]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_160156_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=06ac7546a78d5379b5dc05cb2cdbf1f390f973c4fa3655457bfe4520558c4371"
    ## 
    ## 
    ## $data$files[[2]]
    ## $data$files[[2]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_174713_ALBD.tif"
    ## 
    ## $data$files[[2]]$size
    ## [1] 28805904
    ## 
    ## $data$files[[2]]$md5
    ## NULL
    ## 
    ## $data$files[[2]]$crc32
    ## [1] "e3da049a"
    ## 
    ## $data$files[[2]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_174713_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=19c74756a09a68c992d0d7b84914bddbc96aee49869dc29bd3577dd87e8eb213"
    ## 
    ## 
    ## $data$files[[3]]
    ## $data$files[[3]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_154502_ALBD.tif"
    ## 
    ## $data$files[[3]]$size
    ## [1] 57664080
    ## 
    ## $data$files[[3]]$md5
    ## NULL
    ## 
    ## $data$files[[3]]$crc32
    ## [1] "64c29b25"
    ## 
    ## $data$files[[3]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_154502_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=5678f5fa59296556d5d8b9423ab2fbd8ef31a17822869d3601fb46079defb2fd"
    ## 
    ## 
    ## $data$files[[4]]
    ## $data$files[[4]]$name
    ## [1] "20200807_172100_hsi_kml_0000.kml"
    ## 
    ## $data$files[[4]]$size
    ## [1] 86436
    ## 
    ## $data$files[[4]]$md5
    ## NULL
    ## 
    ## $data$files[[4]]$crc32
    ## [1] "de1c3aa8"
    ## 
    ## $data$files[[4]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_172100_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=1effd8694038a79c2238b8c9ea1a10195038f4b9430d6e7701f5bcfb0f736835"
    ## 
    ## 
    ## $data$files[[5]]
    ## $data$files[[5]]$name
    ## [1] "20200801_153959_hsi_kml_0000.kml"
    ## 
    ## $data$files[[5]]$size
    ## [1] 96369
    ## 
    ## $data$files[[5]]$md5
    ## NULL
    ## 
    ## $data$files[[5]]$crc32
    ## [1] "6ac41d0b"
    ## 
    ## $data$files[[5]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_153959_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=5de7fae4dbfb1086b88146a18278e6ee309d9f574bbc9cd4316750f9d409263f"
    ## 
    ## 
    ## $data$files[[6]]
    ## $data$files[[6]]$name
    ## [1] "20200720_173638_hsi_kml_0000.kml"
    ## 
    ## $data$files[[6]]$size
    ## [1] 64811
    ## 
    ## $data$files[[6]]$md5
    ## NULL
    ## 
    ## $data$files[[6]]$crc32
    ## [1] "5b4fe6dd"
    ## 
    ## $data$files[[6]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_173638_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=234c7556d550fd5f96e968d8d34d24555ecce33d2486bce080cdf60a7f5f589f"
    ## 
    ## 
    ## $data$files[[7]]
    ## $data$files[[7]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_154719_ALBD.tif"
    ## 
    ## $data$files[[7]]$size
    ## [1] 43899972
    ## 
    ## $data$files[[7]]$md5
    ## NULL
    ## 
    ## $data$files[[7]]$crc32
    ## [1] "9281b3b3"
    ## 
    ## $data$files[[7]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_154719_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=2f197c17f820a484e04ffc7f0696ff42535ded440f912100afacae1495d7957b"
    ## 
    ## 
    ## $data$files[[8]]
    ## $data$files[[8]]$name
    ## [1] "20200720_171552_hsi_kml_0000.kml"
    ## 
    ## $data$files[[8]]$size
    ## [1] 65285
    ## 
    ## $data$files[[8]]$md5
    ## NULL
    ## 
    ## $data$files[[8]]$crc32
    ## [1] "59d6bc99"
    ## 
    ## $data$files[[8]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_171552_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=c0ff2137ce2e378d67973a4726014168d687995679f844ccc3c80871cc00e397"
    ## 
    ## 
    ## $data$files[[9]]
    ## $data$files[[9]]$name
    ## [1] "20200807_164808_hsi_kml_0000.kml"
    ## 
    ## $data$files[[9]]$size
    ## [1] 86916
    ## 
    ## $data$files[[9]]$md5
    ## NULL
    ## 
    ## $data$files[[9]]$crc32
    ## [1] "cbc002c0"
    ## 
    ## $data$files[[9]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_164808_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=7318a4f849757d2540867ff495be88a8db1ede6546a67bb0b0102e36c6bb4c81"
    ## 
    ## 
    ## $data$files[[10]]
    ## $data$files[[10]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200731_154336_ALBD.tif"
    ## 
    ## $data$files[[10]]$size
    ## [1] 39327772
    ## 
    ## $data$files[[10]]$md5
    ## NULL
    ## 
    ## $data$files[[10]]$crc32
    ## [1] "48811ba8"
    ## 
    ## $data$files[[10]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020073114/NEON_D13_NIWO_DP2_20200731_154336_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=bd74c2253305aa544f3654564341f3ce85331f7275df418c32ce670ceaeb91f4"
    ## 
    ## 
    ## $data$files[[11]]
    ## $data$files[[11]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_162134_ALBD.tif"
    ## 
    ## $data$files[[11]]$size
    ## [1] 39824460
    ## 
    ## $data$files[[11]]$md5
    ## NULL
    ## 
    ## $data$files[[11]]$crc32
    ## [1] "da2319ca"
    ## 
    ## $data$files[[11]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_162134_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=cd1c3678a97f89f3b664a66134b470eb7819e11dbe6475182aff12e07f248aab"
    ## 
    ## 
    ## $data$files[[12]]
    ## $data$files[[12]]$name
    ## [1] "20200720_174713_hsi_kml_0000.kml"
    ## 
    ## $data$files[[12]]$size
    ## [1] 57367
    ## 
    ## $data$files[[12]]$md5
    ## NULL
    ## 
    ## $data$files[[12]]$crc32
    ## [1] "e9c3255e"
    ## 
    ## $data$files[[12]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_174713_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=498f8b4def621e55cda5aedb78415012513d0f1f2a949825b43ea56d04ce9128"
    ## 
    ## 
    ## $data$files[[13]]
    ## $data$files[[13]]$name
    ## [1] "20200720_154719_hsi_kml_0000.kml"
    ## 
    ## $data$files[[13]]$size
    ## [1] 99635
    ## 
    ## $data$files[[13]]$md5
    ## NULL
    ## 
    ## $data$files[[13]]$crc32
    ## [1] "a6d9b08a"
    ## 
    ## $data$files[[13]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_154719_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=2ff989ec3fcf1c22a67d38cffaab462b77f5d77e6d6555ae13f78628365b68c5"
    ## 
    ## 
    ## $data$files[[14]]
    ## $data$files[[14]]$name
    ## [1] "NEON.D13.NIWO.DP2.30011.001.readme.20220302T205717Z.txt"
    ## 
    ## $data$files[[14]]$size
    ## [1] 11097
    ## 
    ## $data$files[[14]]$md5
    ## [1] "a70e6306cb0d644131ea81acbd210cc2"
    ## 
    ## $data$files[[14]]$crc32
    ## NULL
    ## 
    ## $data$files[[14]]$url
    ## [1] "https://neon-prod-pub-1.s3.data.neonscience.org/release/tag/RELEASE-2022A/NEON.DOM.SITE.DP2.30011.001/NIWO/20200801T000000--20200901T000000/basic/NEON.D13.NIWO.DP2.30011.001.readme.20220302T205717Z.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=d68268504cc6fafaad1d073573727017147a17348dc388c3747fce7465194466"
    ## 
    ## 
    ## $data$files[[15]]
    ## $data$files[[15]]$name
    ## [1] "20200720_165434_hsi_kml_0000.kml"
    ## 
    ## $data$files[[15]]$size
    ## [1] 57583
    ## 
    ## $data$files[[15]]$md5
    ## NULL
    ## 
    ## $data$files[[15]]$crc32
    ## [1] "598bd72e"
    ## 
    ## $data$files[[15]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_165434_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=b44a3696436d76a95b00df8b2bcf74417f2114f4d586b63024a4c9abf67df936"
    ## 
    ## 
    ## $data$files[[16]]
    ## $data$files[[16]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_175213_ALBD.tif"
    ## 
    ## $data$files[[16]]$size
    ## [1] 28402140
    ## 
    ## $data$files[[16]]$md5
    ## NULL
    ## 
    ## $data$files[[16]]$crc32
    ## [1] "11d1e68b"
    ## 
    ## $data$files[[16]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_175213_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=323139c2904827be885e0f6ea5163a1aa7a42a9098a216dd67b20cdcae67c2c6"
    ## 
    ## 
    ## $data$files[[17]]
    ## $data$files[[17]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200801_153959_ALBD.tif"
    ## 
    ## $data$files[[17]]$size
    ## [1] 36437144
    ## 
    ## $data$files[[17]]$md5
    ## NULL
    ## 
    ## $data$files[[17]]$crc32
    ## [1] "253a1107"
    ## 
    ## $data$files[[17]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_153959_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=20bb41da360db8467ff27edcf1c7cdefa0597ab0d2514b56473435937a83cc69"
    ## 
    ## 
    ## $data$files[[18]]
    ## $data$files[[18]]$name
    ## [1] "20200807_155314_hsi_kml_0000.kml"
    ## 
    ## $data$files[[18]]$size
    ## [1] 86727
    ## 
    ## $data$files[[18]]$md5
    ## NULL
    ## 
    ## $data$files[[18]]$crc32
    ## [1] "0f733a96"
    ## 
    ## $data$files[[18]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_155314_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=6baae473256a0cdd82eadd29512e0d5737aece537cfd4ee90147af6613d67598"
    ## 
    ## 
    ## $data$files[[19]]
    ## $data$files[[19]]$name
    ## [1] "20200720_161640_hsi_kml_0000.kml"
    ## 
    ## $data$files[[19]]$size
    ## [1] 103688
    ## 
    ## $data$files[[19]]$md5
    ## NULL
    ## 
    ## $data$files[[19]]$crc32
    ## [1] "1492b156"
    ## 
    ## $data$files[[19]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_161640_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=68f639bf594273b49f84e4a51c09ae9ef62d4c4fec7fe04f0d0fb82745fba63d"
    ## 
    ## 
    ## $data$files[[20]]
    ## $data$files[[20]]$name
    ## [1] "20200807_160118_hsi_kml_0000.kml"
    ## 
    ## $data$files[[20]]$size
    ## [1] 89464
    ## 
    ## $data$files[[20]]$md5
    ## NULL
    ## 
    ## $data$files[[20]]$crc32
    ## [1] "ae83ed04"
    ## 
    ## $data$files[[20]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_160118_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=466e7d1b87cf594b8f156f4f1548a6e71e7ec37ba6a4d9dedcfe54196c318d0b"
    ## 
    ## 
    ## $data$files[[21]]
    ## $data$files[[21]]$name
    ## [1] "20200807_162134_hsi_kml_0000.kml"
    ## 
    ## $data$files[[21]]$size
    ## [1] 86142
    ## 
    ## $data$files[[21]]$md5
    ## NULL
    ## 
    ## $data$files[[21]]$crc32
    ## [1] "08ae1a9f"
    ## 
    ## $data$files[[21]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_162134_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=8556053606de2b4cd4b552ef32efd5c19e47721531f73d39a6c46b375e0b214a"
    ## 
    ## 
    ## $data$files[[22]]
    ## $data$files[[22]]$name
    ## [1] "20200720_172208_hsi_kml_0000.kml"
    ## 
    ## $data$files[[22]]$size
    ## [1] 60804
    ## 
    ## $data$files[[22]]$md5
    ## NULL
    ## 
    ## $data$files[[22]]$crc32
    ## [1] "9b30f65f"
    ## 
    ## $data$files[[22]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_172208_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=5f6f8e4ae11f011b0df875fc21ac0e14f92b68ec3d8fae31a5985e87159f1353"
    ## 
    ## 
    ## $data$files[[23]]
    ## $data$files[[23]]$name
    ## [1] "20200807_170802_hsi_kml_0000.kml"
    ## 
    ## $data$files[[23]]$size
    ## [1] 82406
    ## 
    ## $data$files[[23]]$md5
    ## NULL
    ## 
    ## $data$files[[23]]$crc32
    ## [1] "17d4aa4f"
    ## 
    ## $data$files[[23]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_170802_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=dbd7b3bb4c37bf33c6091b73fa12f038e69e554e359416893db0f2cf796eb060"
    ## 
    ## 
    ## $data$files[[24]]
    ## $data$files[[24]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_173121_ALBD.tif"
    ## 
    ## $data$files[[24]]$size
    ## [1] 21724904
    ## 
    ## $data$files[[24]]$md5
    ## NULL
    ## 
    ## $data$files[[24]]$crc32
    ## [1] "747cab6e"
    ## 
    ## $data$files[[24]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_173121_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=14c97600a2628f0052e7ca4c2e2b474e23fa2c75b9af07d5a9f04484aa9d16d0"
    ## 
    ## 
    ## $data$files[[25]]
    ## $data$files[[25]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_180514_ALBD.tif"
    ## 
    ## $data$files[[25]]$size
    ## [1] 63612396
    ## 
    ## $data$files[[25]]$md5
    ## NULL
    ## 
    ## $data$files[[25]]$crc32
    ## [1] "90ed6075"
    ## 
    ## $data$files[[25]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_180514_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=79f6062316683f0ad6f25f2e5c9fe3c2675d319b279ea2514b03857b64e581db"
    ## 
    ## 
    ## $data$files[[26]]
    ## $data$files[[26]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_172208_ALBD.tif"
    ## 
    ## $data$files[[26]]$size
    ## [1] 27488556
    ## 
    ## $data$files[[26]]$md5
    ## NULL
    ## 
    ## $data$files[[26]]$crc32
    ## [1] "95f2f06f"
    ## 
    ## $data$files[[26]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_172208_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=74083f2699c61c123d5ae67fef4c62b21c8cd0c50e0eb73b4a6b8a4e4e8b787e"
    ## 
    ## 
    ## $data$files[[27]]
    ## $data$files[[27]]$name
    ## [1] "20200801_152930_hsi_kml_0000.kml"
    ## 
    ## $data$files[[27]]$size
    ## [1] 81447
    ## 
    ## $data$files[[27]]$md5
    ## NULL
    ## 
    ## $data$files[[27]]$crc32
    ## [1] "948af222"
    ## 
    ## $data$files[[27]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_152930_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=7b038cf7242f1e11c8468dad3f8993dffc9c3efe1175e61eb75bc790d5b4d83f"
    ## 
    ## 
    ## $data$files[[28]]
    ## $data$files[[28]]$name
    ## [1] "20200720_173121_hsi_kml_0000.kml"
    ## 
    ## $data$files[[28]]$size
    ## [1] 68313
    ## 
    ## $data$files[[28]]$md5
    ## NULL
    ## 
    ## $data$files[[28]]$crc32
    ## [1] "5c4068a1"
    ## 
    ## $data$files[[28]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_173121_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=bb2e05f7d569f0ed10cc9c71853369023a795a36433a9895f3beafa7a0033a76"
    ## 
    ## 
    ## $data$files[[29]]
    ## $data$files[[29]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_164154_ALBD.tif"
    ## 
    ## $data$files[[29]]$size
    ## [1] 43818724
    ## 
    ## $data$files[[29]]$md5
    ## NULL
    ## 
    ## $data$files[[29]]$crc32
    ## [1] "c63cb4f0"
    ## 
    ## $data$files[[29]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_164154_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=5c3d256c22baf62e66b11a84445d6514aae23f3bec10ac39554de6b8d257e820"
    ## 
    ## 
    ## $data$files[[30]]
    ## $data$files[[30]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200801_163340_ALBD.tif"
    ## 
    ## $data$files[[30]]$size
    ## [1] 45453708
    ## 
    ## $data$files[[30]]$md5
    ## NULL
    ## 
    ## $data$files[[30]]$crc32
    ## [1] "0d036c36"
    ## 
    ## $data$files[[30]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_163340_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=10510c5dbc329caa454957bb38badaba72e52aff7e4b16ff6a2796c46f1a8cec"
    ## 
    ## 
    ## $data$files[[31]]
    ## $data$files[[31]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_165500_ALBD.tif"
    ## 
    ## $data$files[[31]]$size
    ## [1] 41225052
    ## 
    ## $data$files[[31]]$md5
    ## NULL
    ## 
    ## $data$files[[31]]$crc32
    ## [1] "9d3c9b23"
    ## 
    ## $data$files[[31]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_165500_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=148541cd256cd58328f1499653ce804f2c5b1496aa3b74a8c19c870f22020baf"
    ## 
    ## 
    ## $data$files[[32]]
    ## $data$files[[32]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_162829_ALBD.tif"
    ## 
    ## $data$files[[32]]$size
    ## [1] 45339260
    ## 
    ## $data$files[[32]]$md5
    ## NULL
    ## 
    ## $data$files[[32]]$crc32
    ## [1] "877d4bc3"
    ## 
    ## $data$files[[32]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_162829_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f38f907c39f81e8a7eee5809002bdf232025ef4f13f56e119cd4255d6d28e51a"
    ## 
    ## 
    ## $data$files[[33]]
    ## $data$files[[33]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_164808_ALBD.tif"
    ## 
    ## $data$files[[33]]$size
    ## [1] 37719276
    ## 
    ## $data$files[[33]]$md5
    ## NULL
    ## 
    ## $data$files[[33]]$crc32
    ## [1] "fde41c25"
    ## 
    ## $data$files[[33]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_164808_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=19a389a0074959a9eeedc0c902faaab016d2777cbff3c2e5ce96022e5ad5487a"
    ## 
    ## 
    ## $data$files[[34]]
    ## $data$files[[34]]$name
    ## [1] "20200807_160712_hsi_kml_0000.kml"
    ## 
    ## $data$files[[34]]$size
    ## [1] 88457
    ## 
    ## $data$files[[34]]$md5
    ## NULL
    ## 
    ## $data$files[[34]]$crc32
    ## [1] "15a16625"
    ## 
    ## $data$files[[34]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_160712_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f72356d7e60e6d08c5892b70ce3e10cc58544950ba7849fc2809d6ae1e5426ae"
    ## 
    ## 
    ## $data$files[[35]]
    ## $data$files[[35]]$name
    ## [1] "2020_NIWO_4_NIWO_L3_spectrometer_processing.pdf"
    ## 
    ## $data$files[[35]]$size
    ## [1] 10756903
    ## 
    ## $data$files[[35]]$md5
    ## NULL
    ## 
    ## $data$files[[35]]$crc32
    ## [1] "508637ea"
    ## 
    ## $data$files[[35]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/2020_NIWO_4_NIWO_L3_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=94e54e16959f043c9de8ab6cd0c5b6e120505a60698347278a292f2bd5a9ed83"
    ## 
    ## 
    ## $data$files[[36]]
    ## $data$files[[36]]$name
    ## [1] "20200807_165500_hsi_kml_0000.kml"
    ## 
    ## $data$files[[36]]$size
    ## [1] 86905
    ## 
    ## $data$files[[36]]$md5
    ## NULL
    ## 
    ## $data$files[[36]]$crc32
    ## [1] "bd5efef0"
    ## 
    ## $data$files[[36]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_165500_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=31536a3a211a08719c4cde993391cb6678bd08eb12c0e20e1e97e02f760b8828"
    ## 
    ## 
    ## $data$files[[37]]
    ## $data$files[[37]]$name
    ## [1] "20200801_155424_hsi_kml_0000.kml"
    ## 
    ## $data$files[[37]]$size
    ## [1] 95980
    ## 
    ## $data$files[[37]]$md5
    ## NULL
    ## 
    ## $data$files[[37]]$crc32
    ## [1] "6826f0e5"
    ## 
    ## $data$files[[37]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_155424_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=4863c49cbce6f6017de2e22a5d41e44ee1f4c6299efd06139d7802d995c92129"
    ## 
    ## 
    ## $data$files[[38]]
    ## $data$files[[38]]$name
    ## [1] "20200720_175809_hsi_kml_0000.kml"
    ## 
    ## $data$files[[38]]$size
    ## [1] 91955
    ## 
    ## $data$files[[38]]$md5
    ## NULL
    ## 
    ## $data$files[[38]]$crc32
    ## [1] "0115faa7"
    ## 
    ## $data$files[[38]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_175809_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=73b1eefa9a931ec9396b9408cfbe278324339f8d43a28adb448ccadf8ec62be4"
    ## 
    ## 
    ## $data$files[[39]]
    ## $data$files[[39]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_170802_ALBD.tif"
    ## 
    ## $data$files[[39]]$size
    ## [1] 41780604
    ## 
    ## $data$files[[39]]$md5
    ## NULL
    ## 
    ## $data$files[[39]]$crc32
    ## [1] "8513245f"
    ## 
    ## $data$files[[39]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_170802_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=96b88cc9593c07566208ff953f5fc112471b58a6d78c71463736b0a6aca05601"
    ## 
    ## 
    ## $data$files[[40]]
    ## $data$files[[40]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200801_162711_ALBD.tif"
    ## 
    ## $data$files[[40]]$size
    ## [1] 46640088
    ## 
    ## $data$files[[40]]$md5
    ## NULL
    ## 
    ## $data$files[[40]]$crc32
    ## [1] "9e9af92e"
    ## 
    ## $data$files[[40]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_162711_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=4216e84017603bf995f53cd84545712bcef7907c37081e0791709d8fc95d8dd4"
    ## 
    ## 
    ## $data$files[[41]]
    ## $data$files[[41]]$name
    ## [1] "NIWO_2020072014_L1_spectrometer_processing.pdf"
    ## 
    ## $data$files[[41]]$size
    ## [1] 1878105
    ## 
    ## $data$files[[41]]$md5
    ## NULL
    ## 
    ## $data$files[[41]]$crc32
    ## [1] "1b2cd1e1"
    ## 
    ## $data$files[[41]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/NIWO_2020072014_L1_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=b23a2e78b1e4a26452f63096e80cbc5cd128185590b60cf6041041de33713a72"
    ## 
    ## 
    ## $data$files[[42]]
    ## $data$files[[42]]$name
    ## [1] "2020_NIWO_4_2020073114_P2C1_L2_spectrometer_processing.pdf"
    ## 
    ## $data$files[[42]]$size
    ## [1] 5288295
    ## 
    ## $data$files[[42]]$md5
    ## NULL
    ## 
    ## $data$files[[42]]$crc32
    ## [1] "4d2d5604"
    ## 
    ## $data$files[[42]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/2020_NIWO_4_2020073114_P2C1_L2_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=e831345c73edea0699f6e5989278d96a467bba8d936f113e531bd4182b3f3fc4"
    ## 
    ## 
    ## $data$files[[43]]
    ## $data$files[[43]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_163444_ALBD.tif"
    ## 
    ## $data$files[[43]]$size
    ## [1] 47069132
    ## 
    ## $data$files[[43]]$md5
    ## NULL
    ## 
    ## $data$files[[43]]$crc32
    ## [1] "07b39945"
    ## 
    ## $data$files[[43]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_163444_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=8074b7aa7101d2520b32ff018772b7341e81cfb7e19939b3214a05278238d085"
    ## 
    ## 
    ## $data$files[[44]]
    ## $data$files[[44]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200801_152930_ALBD.tif"
    ## 
    ## $data$files[[44]]$size
    ## [1] 46156596
    ## 
    ## $data$files[[44]]$md5
    ## NULL
    ## 
    ## $data$files[[44]]$crc32
    ## [1] "6b5823a1"
    ## 
    ## $data$files[[44]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_152930_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a7aba67a05e94696a53255ffd7bdff9b06816b7388916c8d24ca8261f71f1514"
    ## 
    ## 
    ## $data$files[[45]]
    ## $data$files[[45]]$name
    ## [1] "20200720_163955_hsi_kml_0000.kml"
    ## 
    ## $data$files[[45]]$size
    ## [1] 94161
    ## 
    ## $data$files[[45]]$md5
    ## NULL
    ## 
    ## $data$files[[45]]$crc32
    ## [1] "6bcda0c7"
    ## 
    ## $data$files[[45]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_163955_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=13ae1a9ca09f3935155420de9a963b9929e23ea84c318266aa6f6ee3f83e582e"
    ## 
    ## 
    ## $data$files[[46]]
    ## $data$files[[46]]$name
    ## [1] "20200720_165934_hsi_kml_0000.kml"
    ## 
    ## $data$files[[46]]$size
    ## [1] 57290
    ## 
    ## $data$files[[46]]$md5
    ## NULL
    ## 
    ## $data$files[[46]]$crc32
    ## [1] "c7272a7e"
    ## 
    ## $data$files[[46]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_165934_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=b84f3d5a4aa5175b8ebe6eeb0c03129dd4fb45ca31b2a458290ebf2f4adbb499"
    ## 
    ## 
    ## $data$files[[47]]
    ## $data$files[[47]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_181654_ALBD.tif"
    ## 
    ## $data$files[[47]]$size
    ## [1] 59266476
    ## 
    ## $data$files[[47]]$md5
    ## NULL
    ## 
    ## $data$files[[47]]$crc32
    ## [1] "f2ab3328"
    ## 
    ## $data$files[[47]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_181654_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=aed1a68600441a392d43186eecd0dfaf245c415d0b55980e9eb339a734f37c38"
    ## 
    ## 
    ## $data$files[[48]]
    ## $data$files[[48]]$name
    ## [1] "2020_NIWO_4_2020072014_P2C1_L2_spectrometer_processing.pdf"
    ## 
    ## $data$files[[48]]$size
    ## [1] 7630749
    ## 
    ## $data$files[[48]]$md5
    ## NULL
    ## 
    ## $data$files[[48]]$crc32
    ## [1] "fb18a236"
    ## 
    ## $data$files[[48]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/2020_NIWO_4_2020072014_P2C1_L2_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=aa82a9e1372b92770bde10426665ae06681fbb7f4fcdf04ed7215adc98f6326d"
    ## 
    ## 
    ## $data$files[[49]]
    ## $data$files[[49]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200731_155024_ALBD.tif"
    ## 
    ## $data$files[[49]]$size
    ## [1] 46022612
    ## 
    ## $data$files[[49]]$md5
    ## NULL
    ## 
    ## $data$files[[49]]$crc32
    ## [1] "f1f232e9"
    ## 
    ## $data$files[[49]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020073114/NEON_D13_NIWO_DP2_20200731_155024_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=955991d9759eda5421b086df015ea11587f3b79b7bb341b0352975a57028ac14"
    ## 
    ## 
    ## $data$files[[50]]
    ## $data$files[[50]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_161640_ALBD.tif"
    ## 
    ## $data$files[[50]]$size
    ## [1] 44265996
    ## 
    ## $data$files[[50]]$md5
    ## NULL
    ## 
    ## $data$files[[50]]$crc32
    ## [1] "1e6539a8"
    ## 
    ## $data$files[[50]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_161640_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=4e2aedb47162996b784173c819c7f5ccd3d0c07c46b1851019734b421ff7fd2f"
    ## 
    ## 
    ## $data$files[[51]]
    ## $data$files[[51]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200731_151902_ALBD.tif"
    ## 
    ## $data$files[[51]]$size
    ## [1] 61566796
    ## 
    ## $data$files[[51]]$md5
    ## NULL
    ## 
    ## $data$files[[51]]$crc32
    ## [1] "44fef28d"
    ## 
    ## $data$files[[51]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020073114/NEON_D13_NIWO_DP2_20200731_151902_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=ecef8be215022c6bf621b5c00c50dc7cf2a1197b2fc852f62ec060d421421f8c"
    ## 
    ## 
    ## $data$files[[52]]
    ## $data$files[[52]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_161351_ALBD.tif"
    ## 
    ## $data$files[[52]]$size
    ## [1] 48779984
    ## 
    ## $data$files[[52]]$md5
    ## NULL
    ## 
    ## $data$files[[52]]$crc32
    ## [1] "ea61f89e"
    ## 
    ## $data$files[[52]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_161351_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=272e76d52e806169398b9d8f259bc32475331cb1641788bc71942cc7e4c7d0fa"
    ## 
    ## 
    ## $data$files[[53]]
    ## $data$files[[53]]$name
    ## [1] "20200720_175213_hsi_kml_0000.kml"
    ## 
    ## $data$files[[53]]$size
    ## [1] 67545
    ## 
    ## $data$files[[53]]$md5
    ## NULL
    ## 
    ## $data$files[[53]]$crc32
    ## [1] "2f50ff15"
    ## 
    ## $data$files[[53]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_175213_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a911e7b69a8b33da58dc38ee8c8d9c2ce7f0063528b26c06f78f5b3ec0b8d556"
    ## 
    ## 
    ## $data$files[[54]]
    ## $data$files[[54]]$name
    ## [1] "20200720_181208_hsi_kml_0000.kml"
    ## 
    ## $data$files[[54]]$size
    ## [1] 47537
    ## 
    ## $data$files[[54]]$md5
    ## NULL
    ## 
    ## $data$files[[54]]$crc32
    ## [1] "6259bd59"
    ## 
    ## $data$files[[54]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_181208_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=38a4256dc6e5edcae6a270d11a24471b65bee7f8180543fd74fcab629f902870"
    ## 
    ## 
    ## $data$files[[55]]
    ## $data$files[[55]]$name
    ## [1] "20200807_163444_hsi_kml_0000.kml"
    ## 
    ## $data$files[[55]]$size
    ## [1] 86263
    ## 
    ## $data$files[[55]]$md5
    ## NULL
    ## 
    ## $data$files[[55]]$crc32
    ## [1] "ab70059e"
    ## 
    ## $data$files[[55]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_163444_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f31e4f0115477b5545421532d2aa0ab5fad6768482858b6d22f86f23d59c6ff6"
    ## 
    ## 
    ## $data$files[[56]]
    ## $data$files[[56]]$name
    ## [1] "NIWO_2020080114_L1_spectrometer_processing.pdf"
    ## 
    ## $data$files[[56]]$size
    ## [1] 1908264
    ## 
    ## $data$files[[56]]$md5
    ## NULL
    ## 
    ## $data$files[[56]]$crc32
    ## [1] "7210a099"
    ## 
    ## $data$files[[56]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/NIWO_2020080114_L1_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=22cb1ab21d4b8160d0ab86c2adeba46a0cab55320c848dcacc1b263ecb0d4c14"
    ## 
    ## 
    ## $data$files[[57]]
    ## $data$files[[57]]$name
    ## [1] "20200720_181654_hsi_kml_0000.kml"
    ## 
    ## $data$files[[57]]$size
    ## [1] 109293
    ## 
    ## $data$files[[57]]$md5
    ## NULL
    ## 
    ## $data$files[[57]]$crc32
    ## [1] "918390bd"
    ## 
    ## $data$files[[57]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_181654_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=c5f5d8764cdc1b3270abbdc09c755e3cc5daa6521761c8e75578c5451930fddf"
    ## 
    ## 
    ## $data$files[[58]]
    ## $data$files[[58]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200801_155424_ALBD.tif"
    ## 
    ## $data$files[[58]]$size
    ## [1] 38847116
    ## 
    ## $data$files[[58]]$md5
    ## NULL
    ## 
    ## $data$files[[58]]$crc32
    ## [1] "526a2459"
    ## 
    ## $data$files[[58]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_155424_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=8c43fd36c127ec761253738ef81172f384a3ca8d85921b1029e34158d3d9c67d"
    ## 
    ## 
    ## $data$files[[59]]
    ## $data$files[[59]]$name
    ## [1] "NIWO_2020073114_L1_spectrometer_processing.pdf"
    ## 
    ## $data$files[[59]]$size
    ## [1] 1845176
    ## 
    ## $data$files[[59]]$md5
    ## NULL
    ## 
    ## $data$files[[59]]$crc32
    ## [1] "e9c01aac"
    ## 
    ## $data$files[[59]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/NIWO_2020073114_L1_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=8ee873f54c4fb91525f7b37f91a948fa2b305baa660e3ac3295f28b30145116c"
    ## 
    ## 
    ## $data$files[[60]]
    ## $data$files[[60]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_164656_ALBD.tif"
    ## 
    ## $data$files[[60]]$size
    ## [1] 25740444
    ## 
    ## $data$files[[60]]$md5
    ## NULL
    ## 
    ## $data$files[[60]]$crc32
    ## [1] "5d35dcdc"
    ## 
    ## $data$files[[60]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_164656_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=36d2ea76b3603f42ebce5d21260817590db68266c5ec6d146b4a4edc144bab89"
    ## 
    ## 
    ## $data$files[[61]]
    ## $data$files[[61]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_160107_ALBD.tif"
    ## 
    ## $data$files[[61]]$size
    ## [1] 40368396
    ## 
    ## $data$files[[61]]$md5
    ## NULL
    ## 
    ## $data$files[[61]]$crc32
    ## [1] "1c23ca26"
    ## 
    ## $data$files[[61]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_160107_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=15a201252bd0be3350dea392694ff85e32edc626ae527eadee0d857165957f69"
    ## 
    ## 
    ## $data$files[[62]]
    ## $data$files[[62]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_170127_ALBD.tif"
    ## 
    ## $data$files[[62]]$size
    ## [1] 43046296
    ## 
    ## $data$files[[62]]$md5
    ## NULL
    ## 
    ## $data$files[[62]]$crc32
    ## [1] "83e5c5de"
    ## 
    ## $data$files[[62]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_170127_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=163aefb2915450aed65656a8fc5ed6fb00a9ebed9d925b0a576f9c196f90be7f"
    ## 
    ## 
    ## $data$files[[63]]
    ## $data$files[[63]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200801_162052_ALBD.tif"
    ## 
    ## $data$files[[63]]$size
    ## [1] 46864744
    ## 
    ## $data$files[[63]]$md5
    ## NULL
    ## 
    ## $data$files[[63]]$crc32
    ## [1] "036ba1a5"
    ## 
    ## $data$files[[63]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_162052_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=b0ddc8be869ef48c63ffcf06d5512a7d17c8fcaa62266948cf476908a1a83935"
    ## 
    ## 
    ## $data$files[[64]]
    ## $data$files[[64]]$name
    ## [1] "20200807_162829_hsi_kml_0000.kml"
    ## 
    ## $data$files[[64]]$size
    ## [1] 89038
    ## 
    ## $data$files[[64]]$md5
    ## NULL
    ## 
    ## $data$files[[64]]$crc32
    ## [1] "a0edd785"
    ## 
    ## $data$files[[64]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_162829_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=0e22ff591a85f35a07f8a8647280115263b0dc504be566024ae371e4f9f7d5c3"
    ## 
    ## 
    ## $data$files[[65]]
    ## $data$files[[65]]$name
    ## [1] "20200720_180514_hsi_kml_0000.kml"
    ## 
    ## $data$files[[65]]$size
    ## [1] 110205
    ## 
    ## $data$files[[65]]$md5
    ## NULL
    ## 
    ## $data$files[[65]]$crc32
    ## [1] "ff71fc5d"
    ## 
    ## $data$files[[65]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_180514_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a2fc3208800ed7bbe9ddd5a175c257876fec80267febdbca75a9fa1e9675eb81"
    ## 
    ## 
    ## $data$files[[66]]
    ## $data$files[[66]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_171113_ALBD.tif"
    ## 
    ## $data$files[[66]]$size
    ## [1] 27175960
    ## 
    ## $data$files[[66]]$md5
    ## NULL
    ## 
    ## $data$files[[66]]$crc32
    ## [1] "07b8277e"
    ## 
    ## $data$files[[66]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_171113_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=31bccbf4111dda9d8359282c5ded8c53f203a0ddd04bb5d4d4bffd7bc1b45330"
    ## 
    ## 
    ## $data$files[[67]]
    ## $data$files[[67]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200731_155646_ALBD.tif"
    ## 
    ## $data$files[[67]]$size
    ## [1] 37108680
    ## 
    ## $data$files[[67]]$md5
    ## NULL
    ## 
    ## $data$files[[67]]$crc32
    ## [1] "773c8948"
    ## 
    ## $data$files[[67]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020073114/NEON_D13_NIWO_DP2_20200731_155646_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=eaa543f0d6f399ff884b47f03c98bc1b3f361ba2aafcac02a5b4e2f3f0bd7297"
    ## 
    ## 
    ## $data$files[[68]]
    ## $data$files[[68]]$name
    ## [1] "20200731_151902_hsi_kml_0000.kml"
    ## 
    ## $data$files[[68]]$size
    ## [1] 99179
    ## 
    ## $data$files[[68]]$md5
    ## NULL
    ## 
    ## $data$files[[68]]$crc32
    ## [1] "c4e83492"
    ## 
    ## $data$files[[68]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200731_151902_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=747a22ac2dd3c9e8ac5de0d7d5ee257a4d77179f708adb635680a64692331f9e"
    ## 
    ## 
    ## $data$files[[69]]
    ## $data$files[[69]]$name
    ## [1] "20200801_162711_hsi_kml_0000.kml"
    ## 
    ## $data$files[[69]]$size
    ## [1] 94102
    ## 
    ## $data$files[[69]]$md5
    ## NULL
    ## 
    ## $data$files[[69]]$crc32
    ## [1] "1422751c"
    ## 
    ## $data$files[[69]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_162711_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f52eb8ca2f1ef5e79ef36670292377cc9802436b9fceda5e07011ca07837fbd4"
    ## 
    ## 
    ## $data$files[[70]]
    ## $data$files[[70]]$name
    ## [1] "20200731_154336_hsi_kml_0000.kml"
    ## 
    ## $data$files[[70]]$size
    ## [1] 109300
    ## 
    ## $data$files[[70]]$md5
    ## NULL
    ## 
    ## $data$files[[70]]$crc32
    ## [1] "5c52aea3"
    ## 
    ## $data$files[[70]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200731_154336_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=64079ae8b6fdc19479bb5c1435254a2270cee9c22e3a53f3ca1302380e499a20"
    ## 
    ## 
    ## $data$files[[71]]
    ## $data$files[[71]]$name
    ## [1] "20200807_171424_hsi_kml_0000.kml"
    ## 
    ## $data$files[[71]]$size
    ## [1] 81434
    ## 
    ## $data$files[[71]]$md5
    ## NULL
    ## 
    ## $data$files[[71]]$crc32
    ## [1] "9ed62eb8"
    ## 
    ## $data$files[[71]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_171424_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=8b053519adb5d76449de5048f0ff205b3ce504d13fefab053db1d4e8a383072a"
    ## 
    ## 
    ## $data$files[[72]]
    ## $data$files[[72]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_163955_ALBD.tif"
    ## 
    ## $data$files[[72]]$size
    ## [1] 42262236
    ## 
    ## $data$files[[72]]$md5
    ## NULL
    ## 
    ## $data$files[[72]]$crc32
    ## [1] "5f83fdae"
    ## 
    ## $data$files[[72]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_163955_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=41ff285eb6a890e61926b3ca5c9c17a6bdc743489df124872a9f8c0393aaa3b5"
    ## 
    ## 
    ## $data$files[[73]]
    ## $data$files[[73]]$name
    ## [1] "20200720_163210_hsi_kml_0000.kml"
    ## 
    ## $data$files[[73]]$size
    ## [1] 105799
    ## 
    ## $data$files[[73]]$md5
    ## NULL
    ## 
    ## $data$files[[73]]$crc32
    ## [1] "0af7718e"
    ## 
    ## $data$files[[73]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_163210_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=fab950f931faf3ac77412a56f664b8fc80eafcb60d6dd1a9001818cc9d01015e"
    ## 
    ## 
    ## $data$files[[74]]
    ## $data$files[[74]]$name
    ## [1] "20200807_154502_hsi_kml_0000.kml"
    ## 
    ## $data$files[[74]]$size
    ## [1] 72216
    ## 
    ## $data$files[[74]]$md5
    ## NULL
    ## 
    ## $data$files[[74]]$crc32
    ## [1] "f955614a"
    ## 
    ## $data$files[[74]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_154502_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=d6ca15990c55eda4ab5b15410046e8461ffe1f8fe284aed05298a12c6639951d"
    ## 
    ## 
    ## $data$files[[75]]
    ## $data$files[[75]]$name
    ## [1] "20200720_160107_hsi_kml_0000.kml"
    ## 
    ## $data$files[[75]]$size
    ## [1] 106434
    ## 
    ## $data$files[[75]]$md5
    ## NULL
    ## 
    ## $data$files[[75]]$crc32
    ## [1] "2858b1b9"
    ## 
    ## $data$files[[75]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_160107_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=aa2d46612e963778502c63afd827e334fdadd52aae1ac06f33edb28927fd0a18"
    ## 
    ## 
    ## $data$files[[76]]
    ## $data$files[[76]]$name
    ## [1] "20200801_162052_hsi_kml_0000.kml"
    ## 
    ## $data$files[[76]]$size
    ## [1] 93345
    ## 
    ## $data$files[[76]]$md5
    ## NULL
    ## 
    ## $data$files[[76]]$crc32
    ## [1] "c7e6ece0"
    ## 
    ## $data$files[[76]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_162052_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=685e566966a6612ac25fd9f47048eddecc45f50f8fd05eaefd1359cd217ef1dd"
    ## 
    ## 
    ## $data$files[[77]]
    ## $data$files[[77]]$name
    ## [1] "20200720_174206_hsi_kml_0000.kml"
    ## 
    ## $data$files[[77]]$size
    ## [1] 61724
    ## 
    ## $data$files[[77]]$md5
    ## NULL
    ## 
    ## $data$files[[77]]$crc32
    ## [1] "2e26de6d"
    ## 
    ## $data$files[[77]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_174206_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=9ed597ed6d3748274c98e9e9791d0128b9ae8f789f7b0a7ed32451ac3fabc25f"
    ## 
    ## 
    ## $data$files[[78]]
    ## $data$files[[78]]$name
    ## [1] "20200807_170127_hsi_kml_0000.kml"
    ## 
    ## $data$files[[78]]$size
    ## [1] 83951
    ## 
    ## $data$files[[78]]$md5
    ## NULL
    ## 
    ## $data$files[[78]]$crc32
    ## [1] "0e33d07c"
    ## 
    ## $data$files[[78]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_170127_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=b755de3501da7b22f734538367d31ea1ce3a9229c95c9d44800bf7a893d284a8"
    ## 
    ## 
    ## $data$files[[79]]
    ## $data$files[[79]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_163210_ALBD.tif"
    ## 
    ## $data$files[[79]]$size
    ## [1] 42240108
    ## 
    ## $data$files[[79]]$md5
    ## NULL
    ## 
    ## $data$files[[79]]$crc32
    ## [1] "d0e8285e"
    ## 
    ## $data$files[[79]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_163210_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=11a19690696383acf33d5ce55584404c82e7da5983dbeb062bb56cd78ea21139"
    ## 
    ## 
    ## $data$files[[80]]
    ## $data$files[[80]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_171552_ALBD.tif"
    ## 
    ## $data$files[[80]]$size
    ## [1] 29202540
    ## 
    ## $data$files[[80]]$md5
    ## NULL
    ## 
    ## $data$files[[80]]$crc32
    ## [1] "ff31ee58"
    ## 
    ## $data$files[[80]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_171552_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=8a437c99fe8d3f39fbf8b492d7fa31a604f358c6bcc575f0c68ae6dbc71596e5"
    ## 
    ## 
    ## $data$files[[81]]
    ## $data$files[[81]]$name
    ## [1] "20200801_163340_hsi_kml_0000.kml"
    ## 
    ## $data$files[[81]]$size
    ## [1] 95590
    ## 
    ## $data$files[[81]]$md5
    ## NULL
    ## 
    ## $data$files[[81]]$crc32
    ## [1] "ef59e0ab"
    ## 
    ## $data$files[[81]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_163340_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f889e7deef2db680b94b901ebe766389c0e65e7ea792386feb906d2b52eb06fa"
    ## 
    ## 
    ## $data$files[[82]]
    ## $data$files[[82]]$name
    ## [1] "NIWO_2020080714_L1_spectrometer_processing.pdf"
    ## 
    ## $data$files[[82]]$size
    ## [1] 1877999
    ## 
    ## $data$files[[82]]$md5
    ## NULL
    ## 
    ## $data$files[[82]]$crc32
    ## [1] "7e69a3c1"
    ## 
    ## $data$files[[82]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/NIWO_2020080714_L1_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=509b0bb4698435a3e29bce3c58de6468d51ac538213188e3f7b2813678466798"
    ## 
    ## 
    ## $data$files[[83]]
    ## $data$files[[83]]$name
    ## [1] "20200801_154719_hsi_kml_0000.kml"
    ## 
    ## $data$files[[83]]$size
    ## [1] 98791
    ## 
    ## $data$files[[83]]$md5
    ## NULL
    ## 
    ## $data$files[[83]]$crc32
    ## [1] "271fb78a"
    ## 
    ## $data$files[[83]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_154719_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=b5b1ffb62635dd44604a1d25e4b9d74693f813d36b59866b3651be6071200bb8"
    ## 
    ## 
    ## $data$files[[84]]
    ## $data$files[[84]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_155401_ALBD.tif"
    ## 
    ## $data$files[[84]]$size
    ## [1] 48346876
    ## 
    ## $data$files[[84]]$md5
    ## NULL
    ## 
    ## $data$files[[84]]$crc32
    ## [1] "cac96623"
    ## 
    ## $data$files[[84]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_155401_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=9870586468e178d3f63a2fc4377b0f748313b884ecf8645a516c73051f1be820"
    ## 
    ## 
    ## $data$files[[85]]
    ## $data$files[[85]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_172628_ALBD.tif"
    ## 
    ## $data$files[[85]]$size
    ## [1] 26367660
    ## 
    ## $data$files[[85]]$md5
    ## NULL
    ## 
    ## $data$files[[85]]$crc32
    ## [1] "681579d1"
    ## 
    ## $data$files[[85]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_172628_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=bd01a2109fa79fcc9bda142ba50363e360c0ea48635b49d7c57a4fa42ce2b02d"
    ## 
    ## 
    ## $data$files[[86]]
    ## $data$files[[86]]$name
    ## [1] "20200720_171113_hsi_kml_0000.kml"
    ## 
    ## $data$files[[86]]$size
    ## [1] 69914
    ## 
    ## $data$files[[86]]$md5
    ## NULL
    ## 
    ## $data$files[[86]]$crc32
    ## [1] "2ecba22e"
    ## 
    ## $data$files[[86]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_171113_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=13eea8615de51246c933dad1b5aee243e1046a9d2a46b249f86a2651bfd545b0"
    ## 
    ## 
    ## $data$files[[87]]
    ## $data$files[[87]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200801_160814_ALBD.tif"
    ## 
    ## $data$files[[87]]$size
    ## [1] 41934156
    ## 
    ## $data$files[[87]]$md5
    ## NULL
    ## 
    ## $data$files[[87]]$crc32
    ## [1] "cb447f1b"
    ## 
    ## $data$files[[87]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_160814_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=989e6142084164ec2a234545dc38bb0cee9594c7331d8abd83e02644e5237707"
    ## 
    ## 
    ## $data$files[[88]]
    ## $data$files[[88]]$name
    ## [1] "20200731_155646_hsi_kml_0000.kml"
    ## 
    ## $data$files[[88]]$size
    ## [1] 109728
    ## 
    ## $data$files[[88]]$md5
    ## NULL
    ## 
    ## $data$files[[88]]$crc32
    ## [1] "f26eaaeb"
    ## 
    ## $data$files[[88]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200731_155646_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=2b5400ac7bd3da1a0e7810b680d7bcbaf0c91fc1cb28c049e3d1cf1c327b55f5"
    ## 
    ## 
    ## $data$files[[89]]
    ## $data$files[[89]]$name
    ## [1] "20200720_162438_hsi_kml_0000.kml"
    ## 
    ## $data$files[[89]]$size
    ## [1] 99368
    ## 
    ## $data$files[[89]]$md5
    ## NULL
    ## 
    ## $data$files[[89]]$crc32
    ## [1] "760937e5"
    ## 
    ## $data$files[[89]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_162438_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a51432d7fca8c25c3520bc2e401d54b794acc74ee73d537c0f4d618d40a169be"
    ## 
    ## 
    ## $data$files[[90]]
    ## $data$files[[90]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200731_153721_ALBD.tif"
    ## 
    ## $data$files[[90]]$size
    ## [1] 38072628
    ## 
    ## $data$files[[90]]$md5
    ## NULL
    ## 
    ## $data$files[[90]]$crc32
    ## [1] "e88ccd5f"
    ## 
    ## $data$files[[90]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020073114/NEON_D13_NIWO_DP2_20200731_153721_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=20f843568d001f9b30053dfe1dd731d77c62f0f0c81d155fb61b56975e745fbd"
    ## 
    ## 
    ## $data$files[[91]]
    ## $data$files[[91]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200801_161441_ALBD.tif"
    ## 
    ## $data$files[[91]]$size
    ## [1] 46276740
    ## 
    ## $data$files[[91]]$md5
    ## NULL
    ## 
    ## $data$files[[91]]$crc32
    ## [1] "b96086b9"
    ## 
    ## $data$files[[91]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_161441_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=25e0908160dcd290faefbc6ad0369d6d206fa38dcb02b9057d2369b2562c02a8"
    ## 
    ## 
    ## $data$files[[92]]
    ## $data$files[[92]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_173638_ALBD.tif"
    ## 
    ## $data$files[[92]]$size
    ## [1] 28996344
    ## 
    ## $data$files[[92]]$md5
    ## NULL
    ## 
    ## $data$files[[92]]$crc32
    ## [1] "52ebc3e1"
    ## 
    ## $data$files[[92]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_173638_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=88d098870d454343b0e6ea385fd3c761a950ccf81e61712e287afbea39afb641"
    ## 
    ## 
    ## $data$files[[93]]
    ## $data$files[[93]]$name
    ## [1] "2020_NIWO_4_2020080114_P2C1_L2_spectrometer_processing.pdf"
    ## 
    ## $data$files[[93]]$size
    ## [1] 7040620
    ## 
    ## $data$files[[93]]$md5
    ## NULL
    ## 
    ## $data$files[[93]]$crc32
    ## [1] "d6a77ebc"
    ## 
    ## $data$files[[93]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/2020_NIWO_4_2020080114_P2C1_L2_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a13de3ac0bf0223ce8a10ae745aec9f47382d115e9a52959553d358f4e247b58"
    ## 
    ## 
    ## $data$files[[94]]
    ## $data$files[[94]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_160118_ALBD.tif"
    ## 
    ## $data$files[[94]]$size
    ## [1] 52812980
    ## 
    ## $data$files[[94]]$md5
    ## NULL
    ## 
    ## $data$files[[94]]$crc32
    ## [1] "c18c7f41"
    ## 
    ## $data$files[[94]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_160118_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=012774a63c748669121aae89561de7f2f00b1a70cb95ba3e08b352e20012f65a"
    ## 
    ## 
    ## $data$files[[95]]
    ## $data$files[[95]]$name
    ## [1] "20200731_153721_hsi_kml_0000.kml"
    ## 
    ## $data$files[[95]]$size
    ## [1] 91416
    ## 
    ## $data$files[[95]]$md5
    ## NULL
    ## 
    ## $data$files[[95]]$crc32
    ## [1] "7763d03b"
    ## 
    ## $data$files[[95]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200731_153721_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=88503bb3e3b724ce23e171ba59ed0f9c7383924699c5806a47bbff6bf6a42ed1"
    ## 
    ## 
    ## $data$files[[96]]
    ## $data$files[[96]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_162438_ALBD.tif"
    ## 
    ## $data$files[[96]]$size
    ## [1] 50897652
    ## 
    ## $data$files[[96]]$md5
    ## NULL
    ## 
    ## $data$files[[96]]$crc32
    ## [1] "c31cee40"
    ## 
    ## $data$files[[96]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_162438_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a2de8e771c02e45cbde335bb238217da175e4b7e687947e54fd07cc47a581227"
    ## 
    ## 
    ## $data$files[[97]]
    ## $data$files[[97]]$name
    ## [1] "20200720_155401_hsi_kml_0000.kml"
    ## 
    ## $data$files[[97]]$size
    ## [1] 91761
    ## 
    ## $data$files[[97]]$md5
    ## NULL
    ## 
    ## $data$files[[97]]$crc32
    ## [1] "8ee36152"
    ## 
    ## $data$files[[97]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_155401_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=35d24fdcf37a758645a9eaae2f6b4f331f9528e0cd6dc962ada0f1f1c709175f"
    ## 
    ## 
    ## $data$files[[98]]
    ## $data$files[[98]]$name
    ## [1] "20200720_170523_hsi_kml_0000.kml"
    ## 
    ## $data$files[[98]]$size
    ## [1] 55008
    ## 
    ## $data$files[[98]]$md5
    ## NULL
    ## 
    ## $data$files[[98]]$crc32
    ## [1] "1d8c11de"
    ## 
    ## $data$files[[98]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_170523_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=912f6e1ddeb2dbc19c148d6c56111dc89634c4fefc430d3fe266137584464c45"
    ## 
    ## 
    ## $data$files[[99]]
    ## $data$files[[99]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_160855_ALBD.tif"
    ## 
    ## $data$files[[99]]$size
    ## [1] 39287724
    ## 
    ## $data$files[[99]]$md5
    ## NULL
    ## 
    ## $data$files[[99]]$crc32
    ## [1] "11a19050"
    ## 
    ## $data$files[[99]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_160855_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=9e86a319cc4765e162acd3785f1ab593956a1977ab8700bd6d1f7265c8f37411"
    ## 
    ## 
    ## $data$files[[100]]
    ## $data$files[[100]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_155314_ALBD.tif"
    ## 
    ## $data$files[[100]]$size
    ## [1] 46807296
    ## 
    ## $data$files[[100]]$md5
    ## NULL
    ## 
    ## $data$files[[100]]$crc32
    ## [1] "02e08db4"
    ## 
    ## $data$files[[100]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_155314_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=cb88d962daf1ab367b59c949a36a8fb8c1d5a148586d9935ef9f8ebd9c7583fa"
    ## 
    ## 
    ## $data$files[[101]]
    ## $data$files[[101]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_181208_ALBD.tif"
    ## 
    ## $data$files[[101]]$size
    ## [1] 17270364
    ## 
    ## $data$files[[101]]$md5
    ## NULL
    ## 
    ## $data$files[[101]]$crc32
    ## [1] "852671de"
    ## 
    ## $data$files[[101]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_181208_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=36459a0f9d09d8eadcec7c7e9c760a3d0aded7e0e20bceab4e79acd35824d1bc"
    ## 
    ## 
    ## $data$files[[102]]
    ## $data$files[[102]]$name
    ## [1] "20200801_160156_hsi_kml_0000.kml"
    ## 
    ## $data$files[[102]]$size
    ## [1] 101356
    ## 
    ## $data$files[[102]]$md5
    ## NULL
    ## 
    ## $data$files[[102]]$crc32
    ## [1] "71e4ab2e"
    ## 
    ## $data$files[[102]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_160156_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f2350bf3adc2b82f8342f432dab136c1d42bdc325301d9f8929e3c88b5f9cb4b"
    ## 
    ## 
    ## $data$files[[103]]
    ## $data$files[[103]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_154017_ALBD.tif"
    ## 
    ## $data$files[[103]]$size
    ## [1] 48905196
    ## 
    ## $data$files[[103]]$md5
    ## NULL
    ## 
    ## $data$files[[103]]$crc32
    ## [1] "29c31e4d"
    ## 
    ## $data$files[[103]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_154017_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=547890fd7da4ca659dea5a2e0fdb2199ec6c895ffeb8bdaa287974cd65a55e30"
    ## 
    ## 
    ## $data$files[[104]]
    ## $data$files[[104]]$name
    ## [1] "2020_NIWO_4_2020080714_P2C1_L2_spectrometer_processing.pdf"
    ## 
    ## $data$files[[104]]$size
    ## [1] 8549613
    ## 
    ## $data$files[[104]]$md5
    ## NULL
    ## 
    ## $data$files[[104]]$crc32
    ## [1] "d0d5b7e0"
    ## 
    ## $data$files[[104]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/Reports/2020_NIWO_4_2020080714_P2C1_L2_spectrometer_processing.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=f1a50d6c3fd7116c05ca44e2722b7e2c159ce859797a76ca28adb3e721efdbd9"
    ## 
    ## 
    ## $data$files[[105]]
    ## $data$files[[105]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_165934_ALBD.tif"
    ## 
    ## $data$files[[105]]$size
    ## [1] 25473820
    ## 
    ## $data$files[[105]]$md5
    ## NULL
    ## 
    ## $data$files[[105]]$crc32
    ## [1] "93b540f6"
    ## 
    ## $data$files[[105]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_165934_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=e4e3b667d45df19a420d32648ffa3c1a5b6eefc99cdecd15f9b49ce158d59109"
    ## 
    ## 
    ## $data$files[[106]]
    ## $data$files[[106]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_175809_ALBD.tif"
    ## 
    ## $data$files[[106]]$size
    ## [1] 46057708
    ## 
    ## $data$files[[106]]$md5
    ## NULL
    ## 
    ## $data$files[[106]]$crc32
    ## [1] "be88f2dd"
    ## 
    ## $data$files[[106]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_175809_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=4efa2c78c704e086628d40de644781413e1b80971752874ceebbb4b567126e12"
    ## 
    ## 
    ## $data$files[[107]]
    ## $data$files[[107]]$name
    ## [1] "20200807_161351_hsi_kml_0000.kml"
    ## 
    ## $data$files[[107]]$size
    ## [1] 86314
    ## 
    ## $data$files[[107]]$md5
    ## NULL
    ## 
    ## $data$files[[107]]$crc32
    ## [1] "7e073854"
    ## 
    ## $data$files[[107]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_161351_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=c495fd930d458df488396bfcbbf6bafe1dc6c32771843dea02a76fcec916a98a"
    ## 
    ## 
    ## $data$files[[108]]
    ## $data$files[[108]]$name
    ## [1] "20200807_164154_hsi_kml_0000.kml"
    ## 
    ## $data$files[[108]]$size
    ## [1] 91130
    ## 
    ## $data$files[[108]]$md5
    ## NULL
    ## 
    ## $data$files[[108]]$crc32
    ## [1] "a0385e7e"
    ## 
    ## $data$files[[108]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200807_164154_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=0a5f414ee308b506e4a3c909c12c05c7408e574ca747c35276476475c7e317b0"
    ## 
    ## 
    ## $data$files[[109]]
    ## $data$files[[109]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_170523_ALBD.tif"
    ## 
    ## $data$files[[109]]$size
    ## [1] 26911436
    ## 
    ## $data$files[[109]]$md5
    ## NULL
    ## 
    ## $data$files[[109]]$crc32
    ## [1] "8ce1f0c1"
    ## 
    ## $data$files[[109]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_170523_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=ccc8bc65d3fd9ecb744d1368d5852b65b863859100a18e9dda7f071040cea8ce"
    ## 
    ## 
    ## $data$files[[110]]
    ## $data$files[[110]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_165434_ALBD.tif"
    ## 
    ## $data$files[[110]]$size
    ## [1] 31206476
    ## 
    ## $data$files[[110]]$md5
    ## NULL
    ## 
    ## $data$files[[110]]$crc32
    ## [1] "61546397"
    ## 
    ## $data$files[[110]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_165434_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=50acfc629a1ac096529028b1a567897dd7375bbff22794e0f64f709af064fb0a"
    ## 
    ## 
    ## $data$files[[111]]
    ## $data$files[[111]]$name
    ## [1] "20200720_164656_hsi_kml_0000.kml"
    ## 
    ## $data$files[[111]]$size
    ## [1] 65526
    ## 
    ## $data$files[[111]]$md5
    ## NULL
    ## 
    ## $data$files[[111]]$crc32
    ## [1] "85290801"
    ## 
    ## $data$files[[111]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_164656_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=1b3ca0832efdadd84267f6569d04ed30bd9b4202ba5d69ed9b08af5aafd64559"
    ## 
    ## 
    ## $data$files[[112]]
    ## $data$files[[112]]$name
    ## [1] "20200720_160855_hsi_kml_0000.kml"
    ## 
    ## $data$files[[112]]$size
    ## [1] 104058
    ## 
    ## $data$files[[112]]$md5
    ## NULL
    ## 
    ## $data$files[[112]]$crc32
    ## [1] "b74d9038"
    ## 
    ## $data$files[[112]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_160855_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=ed2350095922e631b97132c24e659c25efac8513b7f9b8e2acb5f82d27270bf0"
    ## 
    ## 
    ## $data$files[[113]]
    ## $data$files[[113]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_172100_ALBD.tif"
    ## 
    ## $data$files[[113]]$size
    ## [1] 43666956
    ## 
    ## $data$files[[113]]$md5
    ## NULL
    ## 
    ## $data$files[[113]]$crc32
    ## [1] "ff979f92"
    ## 
    ## $data$files[[113]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_172100_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=7d2c2709fa4c2788f6a87f20c57faad15e92293118cf20271bfd780811fd9e4e"
    ## 
    ## 
    ## $data$files[[114]]
    ## $data$files[[114]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200801_154719_ALBD.tif"
    ## 
    ## $data$files[[114]]$size
    ## [1] 42130636
    ## 
    ## $data$files[[114]]$md5
    ## NULL
    ## 
    ## $data$files[[114]]$crc32
    ## [1] "22cb8329"
    ## 
    ## $data$files[[114]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080114/NEON_D13_NIWO_DP2_20200801_154719_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=2842e570a57e4df0bcd80fdfe844481a45a62fb9e9a7e0ad7633153a16918017"
    ## 
    ## 
    ## $data$files[[115]]
    ## $data$files[[115]]$name
    ## [1] "20200720_172628_hsi_kml_0000.kml"
    ## 
    ## $data$files[[115]]$size
    ## [1] 57392
    ## 
    ## $data$files[[115]]$md5
    ## NULL
    ## 
    ## $data$files[[115]]$crc32
    ## [1] "540355d4"
    ## 
    ## $data$files[[115]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_172628_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=564d0e5c5b8646d1e121385794a08347c0b57e270ad681610b89a24aaf2a2f12"
    ## 
    ## 
    ## $data$files[[116]]
    ## $data$files[[116]]$name
    ## [1] "20200720_154017_hsi_kml_0000.kml"
    ## 
    ## $data$files[[116]]$size
    ## [1] 96453
    ## 
    ## $data$files[[116]]$md5
    ## NULL
    ## 
    ## $data$files[[116]]$crc32
    ## [1] "cabba58f"
    ## 
    ## $data$files[[116]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200720_154017_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=bde4e8781930393da84b0746fee4a9b9854a4f5f35532863998f12d0c1a51d60"
    ## 
    ## 
    ## $data$files[[117]]
    ## $data$files[[117]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_160712_ALBD.tif"
    ## 
    ## $data$files[[117]]$size
    ## [1] 50044236
    ## 
    ## $data$files[[117]]$md5
    ## NULL
    ## 
    ## $data$files[[117]]$crc32
    ## [1] "770836d1"
    ## 
    ## $data$files[[117]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_160712_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=a67b42081da565f332fdee2588f7767638f36e84ce9a92fa0901492984a8713d"
    ## 
    ## 
    ## $data$files[[118]]
    ## $data$files[[118]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200720_174206_ALBD.tif"
    ## 
    ## $data$files[[118]]$size
    ## [1] 23881984
    ## 
    ## $data$files[[118]]$md5
    ## NULL
    ## 
    ## $data$files[[118]]$crc32
    ## [1] "f896242f"
    ## 
    ## $data$files[[118]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020072014/NEON_D13_NIWO_DP2_20200720_174206_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=0539c24ca289a615308c1a6fb32b8e29a6266d2fd8cfdb1197ffada9dc76d0c9"
    ## 
    ## 
    ## $data$files[[119]]
    ## $data$files[[119]]$name
    ## [1] "20200731_155024_hsi_kml_0000.kml"
    ## 
    ## $data$files[[119]]$size
    ## [1] 96798
    ## 
    ## $data$files[[119]]$md5
    ## NULL
    ## 
    ## $data$files[[119]]$crc32
    ## [1] "f2230690"
    ## 
    ## $data$files[[119]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200731_155024_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=6ba7562e4454a412d058ab569d96734c9842e424543265e5248470834387b71c"
    ## 
    ## 
    ## $data$files[[120]]
    ## $data$files[[120]]$name
    ## [1] "20200801_161441_hsi_kml_0000.kml"
    ## 
    ## $data$files[[120]]$size
    ## [1] 97229
    ## 
    ## $data$files[[120]]$md5
    ## NULL
    ## 
    ## $data$files[[120]]$crc32
    ## [1] "5e6f7bd6"
    ## 
    ## $data$files[[120]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_161441_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=afe77781ef2cbfc3159d64e873e97a8241cb4bab5a201c5c3f953c1aeb529f39"
    ## 
    ## 
    ## $data$files[[121]]
    ## $data$files[[121]]$name
    ## [1] "NEON_D13_NIWO_DP2_20200807_171424_ALBD.tif"
    ## 
    ## $data$files[[121]]$size
    ## [1] 37936956
    ## 
    ## $data$files[[121]]$md5
    ## NULL
    ## 
    ## $data$files[[121]]$crc32
    ## [1] "fcb47a6f"
    ## 
    ## $data$files[[121]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/L2/Spectrometer/Albedo/2020080714/NEON_D13_NIWO_DP2_20200807_171424_ALBD.tif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=64cf26b4325efb08e12fb557fd03648ceeae15cae7b6a4cf2ad1213cba27e987"
    ## 
    ## 
    ## $data$files[[122]]
    ## $data$files[[122]]$name
    ## [1] "20200801_160814_hsi_kml_0000.kml"
    ## 
    ## $data$files[[122]]$size
    ## [1] 93587
    ## 
    ## $data$files[[122]]$md5
    ## NULL
    ## 
    ## $data$files[[122]]$crc32
    ## [1] "15834bdb"
    ## 
    ## $data$files[[122]]$url
    ## [1] "https://neon-aop-products.s3.data.neonscience.org/2020/FullSite/D13/2020_NIWO_4/Metadata/Spectrometer/FlightlineBoundary/20200801_160814_hsi_kml_0000.kml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220311T184854Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=pub-internal-read%2F20220311%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Signature=9c16851c44327d3f5dfb68a8fc20a736f5890ef46a1d7c196f1dcbff78a7674e"
