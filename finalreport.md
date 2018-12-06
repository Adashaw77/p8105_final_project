final report
================
Jingwei Ren, Baoyi Shi, Xinyao Wu, Qi Shao
12/2/2018

motivation
----------

Over the past decades, the prevalence of asthma has increased in the urban areas with the potential effects of airflow, air quality and production of aeroallergens. Asthma is the most prevalent chronic disease among children. The disease can make breathing difficult and trigger coughing, wheezing and shortness of breath by the presence of extra mucus in narrow airways. Data on the influence of green spaces on asthma in children are inconstant. Previous research that did in Kaunas, Lithuania showed a positive association between the level of the surrounding greenness and risk of asthma in children. Their study suggested that high exposure to green spaces may increase the risk of allergic conditions and the prevalence of asthma through the effect of pollen. Another ecological design study did in New York City observed an inverse association between street tree density and the prevalence of asthma. Others have reported no relationships between greenery densities, canopy cover and asthma.

Our goal was to investigate the association between tree densities and asthma among children in New York City, including variables like poverties and air quality factors like fine particulate matter (PM2.5), and ambient concentrations of sulfur dioxide (SO2).

related work
------------

Asthma infection has been an issue for a long time. Inconsistent associations between urban greenery and childhood asthma had stimulated studies on the relations between those two factors. Scientific reviews related to tree densities and asthma rate are here: [Children Living in areas with more street tress have lower prevalence of asthma](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3415223/), [Associations between neighborhood greenness and asthma in preschool children in Kaunas,Lithuania](https://repository.lsmuni.lt/bitstream/handle/1/1787/27067890.pdf?sequence=1&isAllowed=y) and [Risks and benefits of Green Spaces for children: A Cross-Sectional Study of Associations with Sedentary Behavior, Obesity, Asthma, and Allergy](https://ehp.niehs.nih.gov/doi/10.1289/ehp.1308038).

Initial Questions
-----------------

Our initial questions was whether there is an association between tree densities and asthma among children in New York City based on 42 different neighborhoods.

Furthermore, we investigated the relationships between asthma and other factors, like poverties and air qualities, factors like fine particulate matter (PM2.5), and ambient concentrations of sulfur dioxide (SO2)). We were also interested in the association between tree densities in 42 different neighborhoods and air quality factors.

Data
----

Our GitHub repo of the steps in this analysis can be found [here](https://github.com/Adashaw77/p8105_final_project/blob/master/final.Rmd).

The original tree data includes variables: zip code of each neighborhoods, boroname, tree's diameter, status(alive and dead), health, spc\_common (tree species) and tree's latitude and longitude.

The asthma dataset has a total 16 variables and it includes important variables like neighborhoods, asthma rate for children from 0 to 4 years old and children from 5 to 14 years old, so2 level, poverty percent, percent of the children under 5 years living in the poverty areas, tree's density and total asthma rate.

### Data Source

All data about asthma, poverty and air qualities was retrieved from NYC health (<http://a816-dohbesp.nyc.gov/IndicatorPublic/PublicTracking.aspx>)

All data related to trees was retrieved from NYC open data (<https://data.cityofnewyork.us/Environment/2015-Street-Tree-Census-Tree-Data/pi5s-9p35>)

All neighborhood data was retrieved from (<http://www.infoshare.org/misc/UHF.pdf>)

### scraping method, cleaning

``` r
#import and tidy tree data
tree_df = read_csv("./data/2015StreetTreesCensus_TREES.csv") %>%
  janitor::clean_names() %>%
  filter(status == "Alive") 
```

    ## Parsed with column specification:
    ## cols(
    ##   zipcode = col_integer(),
    ##   boroname = col_character(),
    ##   tree_dbh = col_integer(),
    ##   status = col_character(),
    ##   health = col_character(),
    ##   spc_common = col_character(),
    ##   Latitude = col_double(),
    ##   longitude = col_double()
    ## )

``` r
zipcode_uhf42 = read_excel("./data/Zipcode_UHF42.xlsx") %>%
   gather(key = zipcode_no, value = zipcode, zipcode1:zipcode9) %>%
   select(-zipcode_no, uhf42_name) %>%
   filter(is.na(zipcode) == FALSE)

tree_df = left_join(tree_df, zipcode_uhf42, by = "zipcode") 

mydat = rgdal::readOGR("./UHF42/UHF_42_DOHMH.shp")
```

    ## OGR data source with driver: ESRI Shapefile 
    ## Source: "/Users/wuxinyao/Desktop/p8105_final_project/UHF42/UHF_42_DOHMH.shp", layer: "UHF_42_DOHMH"
    ## with 43 features
    ## It has 8 fields

``` r
area=data.frame(uhf42_code = mydat$UHFCODE,area = mydat$SHAPE_Area) %>%
  filter(is.na(uhf42_code) == FALSE)

tree_df = left_join(tree_df, area, by = "uhf42_code")

#tree density element is added
tree_density = tree_df %>%
  group_by(boroname, uhf42_name, uhf42_code, area) %>%
  dplyr::summarize(tree_total = n()) %>%
  filter(is.na(uhf42_name) == FALSE) %>%
  group_by(uhf42_name) %>%
  dplyr::mutate(tree_density = tree_total/area) %>%
  ungroup() %>%
  mutate(uhf42_name = forcats::fct_reorder(uhf42_name, tree_density))
```

``` r
# import and tidy astham data

asthma_air_poverty = read_csv("./data/asthma_pollutes_poverty.csv") %>%
  select(geo_entity_id, geo_entity_name, name, data_value) %>%
  filter(is.na(geo_entity_id) == FALSE) %>%
  spread(key = name, value = data_value) %>%
  janitor::clean_names() %>%
  select(poverty, children_under_5_years_old_in_poverty, everything()) %>%
  mutate(asthma_total = asthma_emergency_department_visits_children_0_to_4_yrs_old + asthma_emergency_department_visits_children_5_to_14_yrs_old,
         geo_entity_name = forcats::fct_reorder(geo_entity_name, asthma_total))
```

    ## Parsed with column specification:
    ## cols(
    ##   `Unique Id` = col_character(),
    ##   indicator_id = col_integer(),
    ##   geo_type_id = col_integer(),
    ##   measurement_type_id = col_integer(),
    ##   internal_id = col_integer(),
    ##   subtopic_id = col_integer(),
    ##   name = col_character(),
    ##   Measure = col_character(),
    ##   geo_type_name = col_character(),
    ##   description = col_character(),
    ##   geo_entity_id = col_integer(),
    ##   geo_entity_name = col_character(),
    ##   year_description = col_character(),
    ##   data_value = col_double(),
    ##   message = col_character()
    ## )

``` r
tree_density_total = tree_density %>%
  select(boroname, geo_entity_id=uhf42_code, tree_density) %>%
  distinct()

final_df = left_join(asthma_air_poverty, tree_density_total)
```

    ## Joining, by = "geo_entity_id"

Exploratory analysis:
---------------------

### Asthma Visualizations

### Step one

``` r
#plot between asthma and tree
final_asthma_df = gather(final_df, key = asthma_age, value = rate, asthma_emergency_department_visits_children_0_to_4_yrs_old:asthma_emergency_department_visits_children_5_to_14_yrs_old) %>%
  gather(key = poverty_age, value = poverty, poverty:children_under_5_years_old_in_poverty) %>%
  filter((asthma_age == "asthma_emergency_department_visits_children_5_to_14_yrs_old"&poverty_age == "poverty")|(asthma_age == "asthma_emergency_department_visits_children_0_to_4_yrs_old"& poverty_age == "children_under_5_years_old_in_poverty"))

#plot between tree and astham rate
ggplot(final_asthma_df) +
  geom_point(aes(x = tree_density, y = rate, color = asthma_age)) +
   geom_smooth(aes(x = tree_density, y = rate, color = asthma_age), method = "lm",se = F) +
   labs(
    title = "plot 1: Asthma and Tree Density",
    x = "Tree Density",
    y = "Asthma Rate"
  ) +
   theme(legend.position = "bottom") +
   guides(color = guide_legend(nrow = 2)) +
   scale_color_hue(name = "Age")
```

![](finalreport_files/figure-markdown_github/unnamed-chunk-4-1.png)

This plot 1 showed the relationship between asthma and tree. Visually, there was a positive association between tree density and astham rate.

### Step two

> > > > > > > 9b3979ef49895e8f5e21553305a8014306fd4d43 Other factors may also impact the astham rate. We made plots showing the association between astham and air qualities (fine particulate matter (PM2.5), and ambient concentrations of sulfur dioxide (SO2) ) and the association between asthma and poverty rate.

``` r
#plot between asthma and so2
ggplot(final_asthma_df) +
   geom_point(aes(x = sulfur_dioxide_so2, y = rate, color = asthma_age))+
   geom_smooth(aes(x = sulfur_dioxide_so2, y = rate, color = asthma_age), method = "lm", se = F) +
   labs(
    title = "plot 2: Asthma and Sulfur Dioxide",
    x = "Sulfur Dioxide Average",
    y = "Asthma Rate"
  ) +
   theme(legend.position = "bottom") +
   guides(color = guide_legend(nrow = 2)) +
   scale_color_hue(name = "Age")
```

![](finalreport_files/figure-markdown_github/unnamed-chunk-5-1.png)

``` r
# plot asthma and pm2.5
ggplot(final_asthma_df) +
   geom_point(aes(x = fine_particulate_matter_pm2_5, y = rate, color = asthma_age))+
   geom_smooth(aes(x = fine_particulate_matter_pm2_5, y = rate, color = asthma_age), method = "lm", se = F) +
   labs(
    title = "plot 3: Asthma and PM2.5",
    x = "PM2.5 Average",
    y = "Asthma Rate"
  ) +
   theme(legend.position = "bottom") +
   guides(color = guide_legend(nrow = 2)) +
   scale_color_hue(name = "Age")
```

![](finalreport_files/figure-markdown_github/unnamed-chunk-5-2.png)

``` r
# plot between asthma and poverty
ggplot(final_asthma_df) +
   geom_point(aes(x = poverty, y = rate, color = asthma_age))+
   geom_smooth(aes(x = poverty, y = rate, color = asthma_age), method = "lm", se = F) +
   labs(
    title = "plot 4: Asthma and Poverty",
    x = "Poverty Percent",
    y = "Asthma Rate"
  ) +
   theme(legend.position = "bottom") +
   guides(color = guide_legend(nrow = 2)) +
   scale_color_hue(name = "Age")
```

![](finalreport_files/figure-markdown_github/unnamed-chunk-5-3.png)

plot2 showed the relationship between asthma rate and so2. plot3 showed the relationship between astham rate rate and pM.25 plot4 showed the relationship between asthma and poverty. Visually, there was a positive association between astham rate and SO2, PM 2.5 and poverty level.

#### step three

### Step three

We are interested in seeing whether asthma rate is different between children from 0 to 4 years old and children from 5 to 14 years old in 42 neighborhoods in new york city.

``` r
#plot between asthma 0-4 and asthma 5-14 in each UHF42
ggplot(final_asthma_df) +
  geom_bar(aes(x = geo_entity_name, y = rate, fill = asthma_age), stat = "identity", position = "dodge") +
   labs(
    title = "plot 5: Asthma Rate In Each Entity",
    x = "Entity Name",
    y = "Asthma Rate"
  ) +
   theme(legend.position = "bottom", axis.text.x = element_text(angle = 75, hjust = 1)) +
   guides(fill = guide_legend(nrow = 2)) +
   scale_fill_hue(name = "Age") 
```

![](finalreport_files/figure-markdown_github/unnamed-chunk-6-1.png)

Plot 5 showed that children from 0 to 4 years have higher asthma rate compared to children from 5 to 14 years old.

Tree Visualizations
-------------------

We were also interested testing the association between tree density and air quality factors.

``` r
#plot between tree and SO2
ggplot(final_asthma_df) +
  geom_point(aes(x = sulfur_dioxide_so2, y = tree_density)) +
  geom_smooth(aes(x = sulfur_dioxide_so2, y = tree_density), method = "lm", se = F) +
   labs(
    title = "plot 6: Tree Density and Sulfur Dioxide",
    x = "Sulfur Dioxide Average",
    y = "Tree Density"
  ) 
```

![](finalreport_files/figure-markdown_github/unnamed-chunk-7-1.png)

``` r
#plot between tree and pm2.5
ggplot(final_asthma_df) +
  geom_point(aes(x = fine_particulate_matter_pm2_5, y = tree_density)) +
  geom_smooth(aes(x = fine_particulate_matter_pm2_5, y = tree_density), method = "lm", se = F) +
   labs(
    title = "plot 7: Tree Density and PM2.5",
    x = "PM2.5 Average",
    y = "Tree Density"
  ) 
```

![](finalreport_files/figure-markdown_github/unnamed-chunk-7-2.png)

plot 6 showed the association between tree density and so2. plot 7 showed the association between tree density and pm 2.5 Visually, there was a positive association between tree density and air quality factors.

Statistical analyses
--------------------

``` r
# mlr showing the relationship between asthma rate of kids from 0 to 4 years and tree density, so2 levels, percent of the children under 5 years living in the poverty areas
summary(lm(asthma_emergency_department_visits_children_0_to_4_yrs_old~tree_density+sulfur_dioxide_so2+children_under_5_years_old_in_poverty,data=final_df)) %>%
  broom::tidy() %>%
  knitr::kable()
```

| term                                        |       estimate|     std.error|   statistic|    p.value|
|:--------------------------------------------|--------------:|-------------:|-----------:|----------:|
| (Intercept)                                 |     -103.55417|  7.107578e+01|  -1.4569545|  0.1527456|
| tree\_density                               |  -450005.47197|  7.087065e+05|  -0.6349673|  0.5289775|
| sulfur\_dioxide\_so2                        |      192.94852|  6.144257e+01|   3.1403066|  0.0031272|
| children\_under\_5\_years\_old\_in\_poverty |       11.17321|  1.438942e+00|   7.7648807|  0.0000000|

``` r
# mlr howing the relationship between asthma rate of kids from 5 to 14 years and tree density, so2 levels, poverty levels

summary(lm(asthma_emergency_department_visits_children_5_to_14_yrs_old~tree_density+sulfur_dioxide_so2+poverty,data=final_df)) %>%
  broom::tidy() %>%
  knitr::kable()
```

| term                 |       estimate|     std.error|   statistic|    p.value|
|:---------------------|--------------:|-------------:|-----------:|----------:|
| (Intercept)          |      -82.55365|  5.019181e+01|  -1.6447634|  0.1076654|
| tree\_density        |  -471392.09498|  4.963879e+05|  -0.9496446|  0.3478574|
| sulfur\_dioxide\_so2 |      122.64218|  4.405184e+01|   2.7840421|  0.0080856|
| poverty              |       12.87822|  1.587895e+00|   8.1102445|  0.0000000|

Based on the multiple linear regression models, all the factors are significant except tree density ( p value is 0.86, 0.64). This indicated that there are associations between asthma rate of kids and so2 and poverty levels. model one: asthma = -103.55-450005 tree\_density +192SO2+ 11.17poverty model two: asthma = -82.55 -471392 tree\_density+ 122SO2+12.87poverty

Additional analysis
-------------------

We tested the association between astham rate and all pollutes including ozone, black carbon, PM 2.5, NO, NO2, SO2.

``` r
#SLR:choose the pollutes associated with asthma
o3_p1 = summary(lm(asthma_emergency_department_visits_children_0_to_4_yrs_old~ozone_o3, data=final_df))$coefficients[2,4]

o3_p2 = summary(lm(asthma_emergency_department_visits_children_5_to_14_yrs_old~ozone_o3, data=final_df))$coefficients[2,4]

black_carbon_p1 = summary(lm(asthma_emergency_department_visits_children_0_to_4_yrs_old~black_carbon, data=final_df))$coefficients[2,4]

black_carbon_p2 = summary(lm(asthma_emergency_department_visits_children_5_to_14_yrs_old~black_carbon, data=final_df))$coefficients[2,4]

pm2_5_p1 = summary(lm(asthma_emergency_department_visits_children_0_to_4_yrs_old~fine_particulate_matter_pm2_5, data=final_df))$coefficients[2,4]

pm2_5_p2 = summary(lm(asthma_emergency_department_visits_children_5_to_14_yrs_old~fine_particulate_matter_pm2_5, data=final_df))$coefficients[2,4]

no_p1 = summary(lm(asthma_emergency_department_visits_children_0_to_4_yrs_old~nitric_oxide_no, data=final_df))$coefficients[2,4]

no_p2 = summary(lm(asthma_emergency_department_visits_children_5_to_14_yrs_old~nitric_oxide_no, data=final_df))$coefficients[2,4]

no2_p1 = summary(lm(asthma_emergency_department_visits_children_0_to_4_yrs_old~nitrogen_dioxide_no2, data=final_df))$coefficients[2,4]

no2_p2 = summary(lm(asthma_emergency_department_visits_children_5_to_14_yrs_old~nitrogen_dioxide_no2, data=final_df))$coefficients[2,4]

so2_p1 = summary(lm(asthma_emergency_department_visits_children_0_to_4_yrs_old~sulfur_dioxide_so2, data=final_df))$coefficients[2,4]

so2_p2 = summary(lm(asthma_emergency_department_visits_children_5_to_14_yrs_old~sulfur_dioxide_so2, data=final_df))$coefficients[2,4]

data.frame(Pollute = c("Ozone", "Black Carbon", "PM2.5", "NO", "NO2", "SO2"),
                     P_value1 = c(o3_p1, black_carbon_p1, pm2_5_p1, no_p1, no2_p1, so2_p1),
                     P_value2 = c(o3_p2, black_carbon_p2, pm2_5_p2, no_p2, no2_p2, so2_p2)) %>%
  knitr::kable(digits = 3)
```

| Pollute      |  P\_value1|  P\_value2|
|:-------------|----------:|----------:|
| Ozone        |      0.893|      0.376|
| Black Carbon |      0.569|      0.220|
| PM2.5        |      0.678|      0.265|
| NO           |      0.490|      0.883|
| NO2          |      0.758|      0.328|
| SO2          |      0.023|      0.006|

``` r
#based on p values, we choose SO2
```

The result showed that only SO2 had an association with asthma, since the p value is lower than 0.05. All the other air quality factors are not significantly related to children asthma rate.

We did a paired T test to test if there is significant difference between asthma rate of 0-4 and that of 5-14

``` r
#Paired T test to test if there is significant difference between asthma rate of 0-4 and that of 5-14
t.test(final_df$asthma_emergency_department_visits_children_0_to_4_yrs_old, final_df$asthma_emergency_department_visits_children_5_to_14_yrs_old, paired=T) %>%
  broom::tidy() %>%
  knitr::kable()
```

|  estimate|  statistic|   p.value|  parameter|  conf.low|  conf.high| method        | alternative |
|---------:|----------:|---------:|----------:|---------:|----------:|:--------------|:------------|
|  57.01778|   4.761863|  2.11e-05|         44|  32.88609|   81.14946| Paired t-test | two.sided   |

The T test result showed that there is significant difference between children asthma rate of 0-4 and that of 5-14. P value is 2.11e-05, lower than 0.05.

Discussion
----------

Our main goal is to discuss the relationship between tree density in New York City and children asthma rate in 2015. We built two multilinear regression models, investigating the relationship between asthma rate and tree density, SO2 and poverty level for children from 0 to 4 years old and children from 5 to 14 years old. The result showed that exposure to air pollution (SO2) (P-value = 0.048, 0.0116) and increased poverty level(p-value = 6.41e-09,2.83e-09) could contribute to excess asthma in urban areas. All the factors are significant except tree density ( p-value = 0.86, 0.64). We listed some possible reasons that explained why no obvious association was shown between tree density and asthma. One possible explanation is the season. In the spring and summer months, increasing tree densities might have a positive impact on asthma rate. Pollen could be an allergen that gives some people sneezing fits and watery eyes, which could indirectly cause an asthma attack in others. Certain trees like Ash, Birch, and Oak could cause aggravate respiratory allergies. However, in the fall and winter months, increasing tree densities might decrease the asthma rate through the effect on local air quality. Those two situations might counteract each other and result in the conclusion that overall, there was no association between tree density in New York City and children asthma rate in 2015. Other factors might also influence the asthma rate, including sociodemographic characteristics, population density and hospitals amount in the neighborhood. After adjustment, the association between tree density and hospitalizations as a result of asthma might no longer significant.

We were also interested to see whether there was a difference in asthma rate between children from 0 to 4 years old and children from 5 to 14 years old. The result from plot 5 showed that children from 0 to 4 years have a higher asthma rate compared to children from 5 to 14 years old. The T-test also proved that there is a significant difference between children asthma rate of 0-4 and that of 5-14. (P value = 2.11e-05). A positive relationship between tree density and air quality factors in 42 neighborhood in New York city was reported. Tree density increases as CO2 and PM 2.5 levels increase. One possible explanation for this could be that people's awareness of environment increases as the air quality in one area decreases. Therefore, we saw a positive result. We would expect to see a negative association. In order to verify this, we could make plots detecting the change of air quality's levels throughout a couple of years. Increasing tree densities might help to decrease air quality's levels through a long-term process.

In conclusion, we found out that there is no association between tree density and asthma rate based on our data set. If more variables are included in the future, we would expect to see that afforestation in low-income communities will help improve air-quality and thus decrease the risk of child asthma over a long-term period