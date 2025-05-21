Discrimination in Hiring Callback Rates: Focus on Name-Based Bias
================

# Introduction

What if your name alone could stop you from getting a job interview?

This project investigates name-based discrimination in hiring practices
across Europe. Using real-world data from field experiments, we analyze
how callback rates differ between applicants with majority-sounding
names and those with names perceived as Arab, Maghrebi, or Middle
Eastern.

Hiring discrimination is often invisible — it happens behind closed
doors, during screening processes we never see. But this data reveals
how deep the bias runs, and how it spikes during times of crisis and
fear.

# Data & Sources

The data used in this project comes from the Kaggle dataset titled *The
State of Hiring Discrimination*, compiled by Louis Lippens, Siel
Vermeiren, and Stijn Baert. It is based on a systematic review of
correspondence experiments measuring hiring discrimination, covering
studies published between 2005 and 2020. The dataset includes callback
rates for job applicants based on name groups representing different
racial, ethnic, and cultural backgrounds. Data was downloaded from
[Kaggle](https://www.kaggle.com/datasets/lippenslouis/the-state-of-hiring-discrimination)
and converted to `.rds` format for use in R. All visualizations were
created using `tidyverse`, `ggplot2`, and `sf` in RStudio Cloud.

# Libraries

We use the following libraries: - `tidyverse`: for data manipulation and
plotting - `readxl`: to read Excel files (not used here, but included if
needed) - `sf`: for handling spatial (map) data - `rnaturalearth` &
`rnaturalearthdata`: for country map shapes - `RColorBrewer`: for color
palettes

``` r
library(tidyverse)
library(readxl)
library(sf)
library(rnaturalearth)
library(rnaturalearthdata)
library(RColorBrewer)
library(patchwork)
```

# Prepare and Clean the Data

``` r
df <- readRDS("discrimination_data.rds")

df_clean <- df %>%
  filter(!is.na(callback_maj), !is.na(callback_min), !is.na(country), !is.na(treatment_class)) %>%
  mutate(
    callback_diff = callback_maj - callback_min,
    admin = case_when(
      country == "United Kingdom of Great Britain and Northern Ireland" ~ "United Kingdom",
      country == "Russian Federation" ~ NA_character_,
      TRUE ~ country
    )
  ) %>%
  filter(!is.na(admin))

df_grouped <- df_clean %>%
  group_by(admin, treatment_class) %>%
  summarise(mean_diff = mean(callback_diff, na.rm = TRUE), .groups = "drop")

top2_discriminated <- df_grouped %>%
  group_by(admin) %>%
  arrange(desc(mean_diff)) %>%
  mutate(rank = row_number()) %>%
  filter(rank <= 2) %>%
  ungroup()

# Keep only countries with both ranks
valid_admins <- top2_discriminated %>%
  count(admin) %>%
  filter(n == 2) %>%
  pull(admin)

top2_discriminated <- filter(top2_discriminated, admin %in% valid_admins)

world <- ne_countries(scale = "medium", returnclass = "sf")
map_data <- left_join(world, top2_discriminated, by = c("name" = "admin"))

map_west_europe <- map_data %>%
  filter(
    name %in% c(
      "United Kingdom", "Ireland", "France", "Belgium", "Netherlands",
      "Germany", "Switzerland", "Italy", "Spain", "Portugal",
      "Denmark", "Sweden", "Norway", "Austria"
    ) & !is.na(treatment_class)
  )

map_rank1 <- filter(map_west_europe, rank == 1)
map_rank2 <- filter(map_west_europe, rank == 2)
```

This section loads and prepares the data for mapping discrimination
patterns in Western Europe. It begins by loading a cleaned RDS dataset
that contains callback rates from hiring discrimination studies. The
data is filtered to remove missing values and a new column
(callback_diff) is created to represent the difference in callback rates
between majority and minority name groups.

The country names are standardized for mapping purposes (e.g., renaming
“United Kingdom of Great Britain and Northern Ireland” to “United
Kingdom”), and any unmatched country entries are removed.

Then, the data is grouped by country and name group to calculate the
average discrimination level. The group with the highest average
callback gap in each country is selected as the most discriminated name
group.

Finally, this data is joined with geographic map data using
rnaturalearth, and filtered down to include only countries in Western
Europe that have valid discrimination data. This prepares the final
dataset used in the first map visualization. \# Rising Insight:
Discrimination Across Western Europe

``` r
p1 <- ggplot(map_rank1) +
  geom_sf(aes(fill = treatment_class), color = "white", size = 0.4) +
  geom_sf_label(aes(label = name), size = 3, fontface = "bold", fill = "white", label.size = 0.2) +
  scale_fill_brewer(palette = "Set2", name = "Name Group") +
  coord_sf(xlim = c(-25, 30), ylim = c(35, 72), expand = FALSE) +
  labs(title = "Most Discriminated Name Group") +
  theme_minimal(base_size = 13) +
  theme(legend.position = "none", plot.title = element_text(face = "bold"))

p2 <- ggplot(map_rank2) +
  geom_sf(aes(fill = treatment_class), color = "white", size = 0.4) +
  geom_sf_label(aes(label = name), size = 3, fontface = "bold", fill = "white", label.size = 0.2) +
  scale_fill_brewer(palette = "Set2", name = "Name Group") +
  coord_sf(xlim = c(-25, 30), ylim = c(35, 72), expand = FALSE) +
  labs(title = "Second Most Discriminated Name Group") +
  theme_minimal(base_size = 13) +
  theme(legend.position = "bottom", plot.title = element_text(face = "bold"))

p1 + p2
```

    ## Warning in st_point_on_surface.sfc(sf::st_zm(x)): st_point_on_surface may not
    ## give correct results for longitude/latitude data
    ## Warning in st_point_on_surface.sfc(sf::st_zm(x)): st_point_on_surface may not
    ## give correct results for longitude/latitude data

    ## Warning in RColorBrewer::brewer.pal(n, pal): n too large, allowed maximum for palette Set2 is 8
    ## Returning the palette you asked for with that many colors

![](social_issue_map_files/figure-gfm/plot-map-1.png)<!-- -->

This code generates two choropleth maps showing the most discriminated
groups in each Western European country, based on callback rate
differences. Countries are colored by which group faced the highest
average discrimination. Labels are added for clarity, and styling
ensures the map is clean and easy to interpret. This visual helps
highlight regional patterns in name-based hiring bias.

# Key Insight: Discrimination Spikes in Times of Crisis

``` r
df %>%
  filter(
    !is.na(callback_maj), !is.na(callback_min),
    treatment_class == "Arab/Maghrebi/Middle Eastern",
    region == "Europe"
  ) %>%
  mutate(callback_diff = callback_maj - callback_min) %>%
  group_by(year_research) %>%
  summarise(mean_diff = mean(callback_diff, na.rm = TRUE)) %>%
  ggplot(aes(x = year_research, y = mean_diff)) +
  geom_line(color = "firebrick", linewidth = 1.2) +
  geom_point(size = 2) +
  geom_vline(xintercept = 2010, linetype = "dotted", color = "gray40") +
  annotate("text", x = 2010.4, y = 290, label = "Post-Financial Crisis\nAnti-Muslim Climate", size = 3) +
  geom_vline(xintercept = 2015, linetype = "dotted", color = "gray40") +
  annotate("text", x = 2015.4, y = 140, label = "Refugee Crisis &\nParis Attacks", size = 3) +
  geom_vline(xintercept = 2018, linetype = "dotted", color = "gray40") +
  annotate("text", x = 2018.4, y = 250, label = "Rise of Far-Right\nAnti-Muslim Parties", size = 3) +
  labs(
    title = "Discrimination Against Arab/Maghrebi/Middle Eastern Names in Europe",
    subtitle = "Callback gap over time, with major discrimination spikes annotated",
    x = "Year of Research", y = "Avg Callback Gap (Majority - Minority)"
  ) +
  theme_minimal()
```

![](social_issue_map_files/figure-gfm/arab-trend-annotated-peaks-1.png)<!-- -->

This time-series graph shows how callback discrimination against
Arab/Maghrebi/Middle Eastern names in Europe changed over time. Each
point represents the average gap in callbacks between majority and
minority names for a given year. Major historical events are annotated
where discrimination spiked — including the post-financial crisis
period, the 2015 refugee crisis and Paris attacks, and the 2018 rise of
far-right political parties. The pattern reveals how social and
political events can intensify hiring bias. \# Call to Action

Discrimination in hiring is often silent and unseen — but it’s
measurable, and we can do something about it.

# How we can take action:

- Encourage **blind resume screening** in hiring (removing names and
  identifiers)
- Push for **bias training** and diversity hiring initiatives in
  companies
- Vote for leaders who support **inclusive labor laws**
- Support transparency in **hiring metrics and audit systems**

Change begins when we stop ignoring the data. Everyone deserves a fair
shot — not filtered by their name.
