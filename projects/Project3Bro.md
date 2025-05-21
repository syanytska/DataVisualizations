Climate Inequality: How the Poor Face the Worst of Global Warming
================
Syanytska
2025-05-02

# Introduction

This project explores global climate inequality.

## Data & Methods

We combined three datasets:

- **Temperature Anomalies:** Global average temperature anomaly relative
  to 1861–1890 from Our World in Data (HadCRUT5)

- **ND-GAIN Index:** Country-level vulnerability data from ND-GAIN,
  reshaped from wide to long format.

- **GDP & CO₂ Data:** Country-level GDP per capita and CO₂ emissions per
  capita from Our World in Data (World Bank).

Data were merged on `Country` and `Year` to allow for analysis of
climate vulnerability in relation to economic performance and emissions.

``` r
data <- read.csv("merged_gain_gdp_co2.csv")
```

``` r
temp <- read.csv("temperature-anomaly.csv")
```

------------------------------------------------------------------------

# Global Temperature Trend

![](Project3Bro_files/figure-gfm/final-owid-style-temp-anomaly-1.png)<!-- -->

### Climate Vulnerability by Country (ND-GAIN Index)

This choropleth map shows **national-level climate vulnerability** based
on the **ND-GAIN index for the year 2020**. The ND-GAIN (Notre Dame
Global Adaptation Initiative) index measures each country’s exposure,
sensitivity, and capacity to adapt to the negative impacts of climate
change.

A **viridis “magma” color palette** is used, where:

- **Lighter shades** represent **less vulnerable** countries—typically
  wealthier nations with more resources to adapt.
- **Darker shades** represent countries that are **more vulnerable**,
  often due to economic instability, political conflict, or
  climate-sensitive geographies.

The map provides a visual overview of which countries are most at risk
from climate impacts, such as extreme weather, rising sea levels, and
food or water insecurity.

By combining this map with data on carbon emissions, we can highlight
the imbalance: many of the **most vulnerable nations contribute least to
climate change**, while those **most responsible are often the least
affected**.

\##Countries in Africa have some of the lowest national greenhouse gas
emissions, and yet the continent is home to many of the world’s most
climate-vulnerable countries.
![](Project3Bro_files/figure-gfm/unnamed-chunk-1-1.png)<!-- --> \###
Annual Per Capita CO₂ Emissions

This choropleth map visualizes **annual per capita carbon dioxide (CO₂)
emissions by country** using data from 2019. The values reflect
**production-based emissions**—that is, emissions generated within each
country’s borders, regardless of where the goods are consumed.

The map uses a **viridis “magma” color palette**, where:

- **Lighter shades** indicate countries with **low per capita
  emissions**, typically lower-income nations.
- **Darker shades** highlight countries with **high emissions per
  person**, such as the United States, Australia, and several oil-rich
  nations.

To enhance contrast and focus on global disparities, the color scale is
capped at **20 metric tons per person**. This ensures that
higher-emitting countries stand out more clearly, while preserving
visible differentiation among countries emitting below 10 tons per
person.

The map helps visually reinforce the disproportionate contribution of
wealthier countries to climate change, especially when compared to those
most vulnerable to its impacts.

![](Project3Bro_files/figure-gfm/unnamed-chunk-2-1.png)<!-- --> A small
number of wealthy, industrialized countries are responsible for the vast
majority of global CO₂ emissions, despite being far less vulnerable to
climate change.

## Bibliography

- Met Office Hadley Centre – HadCRUT5. “Global temperature anomaly
  relative to 1861-1890.” Our World in Data, 2025.

- Notre Dame Global Adaptation Initiative (ND-GAIN). “Country Index.”
  Retrieved from <https://gain.nd.edu/our-work/country-index/>

- Our World in Data. “CO₂ and GDP datasets based on World Bank data.”
  Retrieved from <https://github.com/owid/owid-datasets>
