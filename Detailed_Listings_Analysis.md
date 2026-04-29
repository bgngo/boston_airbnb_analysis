Comprehensive Analysis of Boston Airbnb Detailed Listings
================
Fall Foliage
2025-12-10

- [1. Data Overview](#1-data-overview)
  - [1.1 Data Loading](#11-data-loading)
  - [1.2 Missing Data Analysis](#12-missing-data-analysis)
- [2. Exploratory Data Analysis](#2-exploratory-data-analysis)
  - [2.1 Price Cleaning](#21-price-cleaning)
  - [2.2 Host Response Rate and Acceptance Rate
    Cleaning](#22-host-response-rate-and-acceptance-rate-cleaning)
  - [2.3 Date Column Processing](#23-date-column-processing)
  - [2.4 Amenities Feature Engineering &
    Analysis](#24-amenities-feature-engineering--analysis)
  - [2.5 Revenue and Performance
    Metrics](#25-revenue-and-performance-metrics)
  - [2.6 Neighbourhood Grouping](#26-neighbourhood-grouping)
  - [2.7 Geographic Distribution
    Overview](#27-geographic-distribution-overview)
  - [2.8 Correlation Analysis](#28-correlation-analysis)
- [3. Host Analysis](#3-host-analysis)
  - [3.1 Superhosts vs. Regular Hosts
    Performance](#31-superhosts-vs-regular-hosts-performance)
  - [3.2 Average Revenue by Host Response
    Rate](#32-average-revenue-by-host-response-rate)
  - [3.3 Average Revenue by Host
    Portfolio](#33-average-revenue-by-host-portfolio)
  - [3.4 Annual Revenue Time Series by Host
    Experience](#34-annual-revenue-time-series-by-host-experience)
- [4. Property Analysis](#4-property-analysis)
  - [4.1 Room and Property Type
    Analysis](#41-room-and-property-type-analysis)
  - [4.2 Property Location
    Distributions](#42-property-location-distributions)
  - [4.3 Property Type and Amenities](#43-property-type-and-amenities)
- [5. Price and Revenue Analysis](#5-price-and-revenue-analysis)
  - [5.1 Univariate Analysis - Price
    Distribution](#51-univariate-analysis---price-distribution)
  - [5.2 Bivariate Analysis - Price vs Key
    Variables](#52-bivariate-analysis---price-vs-key-variables)
  - [5.3 Detailed Interactive Map with Revenue
    Analysis](#53-detailed-interactive-map-with-revenue-analysis)
  - [5.4 Revenue Optimization
    Analysis](#54-revenue-optimization-analysis)
  - [5.5 Neighbourhood Analysis](#55-neighbourhood-analysis)
  - [5.6 Marginal Revenue per Additional
    Amenity](#56-marginal-revenue-per-additional-amenity)
  - [5.7 Revenue Surface: Interaction of Price and
    Occupancy](#57-revenue-surface-interaction-of-price-and-occupancy)
  - [5.8 Revenue Elasticity](#58-revenue-elasticity)
  - [5.9 Revenue Misc](#59-revenue-misc)
- [6. Clustering Analysis](#6-clustering-analysis)
  - [K-Means Clustering](#k-means-clustering)
  - [Cluster Characterization](#cluster-characterization)
  - [Geographic Distribution of
    Clusters](#geographic-distribution-of-clusters)
- [References and Data Sources](#references-and-data-sources)

``` r
# Core data manipulation
library(readr)
library(glue)
library(tidyr)
library(dplyr)
library(stringr)
library(lubridate)
library(purrr)
library(mgcv)


# Visualization
library(ggplot2)
library(gridExtra)
library(grid)
library(RColorBrewer)
library(viridis)
library(scales)

# Mapping and geographic analysis
library(leaflet)
library(sf)
library(ggmap)

# Statistical analysis
library(corrplot)
library(skimr)
library(cluster)
library(factoextra)
library(NbClust)

# Time series
library(xts)
library(zoo)
library(forecast)

# Text analysis (for amenities)
library(stringr)
```

# 1. Data Overview

## 1.1 Data Loading

``` r
# Load the detailed listings dataset
df_raw <- read_csv("data/listings_detailed.csv", 
                   na = c("", "NA", "N/A", "null"),
                   show_col_types = FALSE)

# Initial dataset dimensions
cat("=== DATASET OVERVIEW ===\n")
```

    ## === DATASET OVERVIEW ===

``` r
cat("Total Observations:", nrow(df_raw), "\n")
```

    ## Total Observations: 4419

``` r
cat("Total Variables:", ncol(df_raw), "\n")
```

    ## Total Variables: 79

``` r
cat("\nColumn Names:\n")
```

    ## 
    ## Column Names:

``` r
cat(paste(colnames(df_raw), collapse = ", "), "\n")
```

    ## id, listing_url, scrape_id, last_scraped, source, name, description, neighborhood_overview, picture_url, host_id, host_url, host_name, host_since, host_location, host_about, host_response_time, host_response_rate, host_acceptance_rate, host_is_superhost, host_thumbnail_url, host_picture_url, host_neighbourhood, host_listings_count, host_total_listings_count, host_verifications, host_has_profile_pic, host_identity_verified, neighbourhood, neighbourhood_cleansed, neighbourhood_group_cleansed, latitude, longitude, property_type, room_type, accommodates, bathrooms, bathrooms_text, bedrooms, beds, amenities, price, minimum_nights, maximum_nights, minimum_minimum_nights, maximum_minimum_nights, minimum_maximum_nights, maximum_maximum_nights, minimum_nights_avg_ntm, maximum_nights_avg_ntm, calendar_updated, has_availability, availability_30, availability_60, availability_90, availability_365, calendar_last_scraped, number_of_reviews, number_of_reviews_ltm, number_of_reviews_l30d, availability_eoy, number_of_reviews_ly, estimated_occupancy_l365d, estimated_revenue_l365d, first_review, last_review, review_scores_rating, review_scores_accuracy, review_scores_cleanliness, review_scores_checkin, review_scores_communication, review_scores_location, review_scores_value, license, instant_bookable, calculated_host_listings_count, calculated_host_listings_count_entire_homes, calculated_host_listings_count_private_rooms, calculated_host_listings_count_shared_rooms, reviews_per_month

This dataset comes from Airbnb’s public listings for Boston (as compiled
by Inside Airbnb) and reflects the state of the market from June to
September 2025. Each row corresponds to a single listing and includes
host, property, pricing, review and availability information used later
for segmentation and performance analysis.

## 1.2 Missing Data Analysis

``` r
# Comprehensive missing data analysis
missing_analysis <- df_raw |>
  summarise(across(everything(), ~sum(is.na(.)))) |>
  pivot_longer(everything(), names_to = "variable", values_to = "missing_count") |>
  mutate(missing_pct = round(missing_count / nrow(df_raw) * 100, 2)) |>
  filter(missing_count > 0) |>
  arrange(desc(missing_pct))

cat("=== MISSING DATA SUMMARY ===\n")
```

    ## === MISSING DATA SUMMARY ===

``` r
cat("Variables with missing data:", nrow(missing_analysis), "out of", ncol(df_raw), "\n\n")
```

    ## Variables with missing data: 42 out of 79

``` r
print(head(missing_analysis, 20))
```

    ## # A tibble: 20 × 3
    ##    variable                     missing_count missing_pct
    ##    <chr>                                <int>       <dbl>
    ##  1 neighbourhood_group_cleansed          4419       100  
    ##  2 calendar_updated                      4419       100  
    ##  3 neighborhood_overview                 2140        48.4
    ##  4 neighbourhood                         2140        48.4
    ##  5 license                               1500        33.9
    ##  6 host_about                            1487        33.6
    ##  7 review_scores_checkin                  973        22.0
    ##  8 review_scores_location                 973        22.0
    ##  9 review_scores_value                    973        22.0
    ## 10 review_scores_accuracy                 972        22  
    ## 11 first_review                           971        22.0
    ## 12 last_review                            971        22.0
    ## 13 review_scores_rating                   971        22.0
    ## 14 review_scores_cleanliness              971        22.0
    ## 15 review_scores_communication            971        22.0
    ## 16 reviews_per_month                      971        22.0
    ## 17 price                                  913        20.7
    ## 18 estimated_revenue_l365d                913        20.7
    ## 19 host_location                          888        20.1
    ## 20 beds                                   869        19.7

``` r
# Visualize missing data pattern
missing_data <-
  ggplot(missing_analysis |> head(15),
         aes(x = reorder(variable, missing_pct), y = missing_pct)) +
    geom_bar(stat = "identity", fill = theme_colors$primary, alpha = 0.8) +
    coord_flip() +
    labs(title = "Missing Data Percentage by Variable (Top 15)",
         x = "Variable",
         y = "Missing Data (%)") +
    theme_transparent() +
    theme(
      plot.title = element_text(face = "bold", color = theme_colors$text),
      panel.grid.minor = element_blank()
    )

print(missing_data)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/missing_data_analysis-1.png" style="display: block; margin: auto;" />

Variables that were entirely missing (e.g.,
neighbourhood_group_cleansed, calendar_updated) were excluded from
modeling and, where necessary, reconstructed from related fields.

# 2. Exploratory Data Analysis

## 2.1 Price Cleaning

``` r
# Clean price column (remove $ and convert to numeric)
df <- df_raw |>
  mutate(
    price_clean = as.numeric(str_remove_all(price, "[$,\\s]")),
   price_category = case_when(
   price_clean < 100 ~ "Budget",
   price_clean < 170 ~ "Economy",
   price_clean < 250 ~ "Mid-range",
   price_clean < 350 ~ "Premium",
   TRUE ~ "Luxury"
)
,
    price_category = factor(price_category, 
                           levels = c("Budget", "Economy", "Mid-range", "Premium", "Luxury"))
  )

# Remove extreme outliers (top 0.5% and bottom 0.5%)
price_q01 <- quantile(df$price_clean, 0.005, na.rm = TRUE)
price_q995 <- quantile(df$price_clean, 0.995, na.rm = TRUE)

df <- df |>
  mutate(price_clean = ifelse(price_clean < price_q01 | price_clean > price_q995, 
                               NA, price_clean))

cat("Price range after cleaning: $", min(df$price_clean, na.rm = TRUE), 
    " - $", max(df$price_clean, na.rm = TRUE), "\n")
```

    ## Price range after cleaning: $ 39  - $ 50000

``` r
cat("Price outliers removed (0.5% tails):", 
    sum(df_raw$price_clean < price_q01 | df_raw$price_clean > price_q995, na.rm = TRUE), 
    "observations\n")
```

    ## Price outliers removed (0.5% tails): 0 observations

``` r
cat("Key Percentiles")
```

    ## Key Percentiles

``` r
quantile(df$price_clean, probs = seq(0, 1, 0.1), na.rm = TRUE)
```

    ##      0%     10%     20%     30%     40%     50%     60%     70%     80%     90% 
    ##    39.0    68.8   103.0   135.0   170.0   207.0   247.0   290.0   347.0   472.4 
    ##    100% 
    ## 50000.0

Although a 0.5% tail-trimming rule was applied to remove potential
price-entry errors, no observations fell outside the statistical bounds.
This indicates that the extremely high nightly prices (e.g.,
\$40,000–\$50,000) are genuine rather than numerical anomalies. Because
such ultra-luxury listings represent a meaningful market segment rather
than noise, we deliberately keep them for further analysis and allow the
clustering algorithm to treat them as a distinct pricing subgroup.

## 2.2 Host Response Rate and Acceptance Rate Cleaning

``` r
# Clean host response and acceptance rates (convert percentages)
df <- df |>
  mutate(
    host_response_rate_num = as.numeric(str_remove_all(host_response_rate, "[%]")) / 100,
    host_acceptance_rate_num = as.numeric(str_remove_all(host_acceptance_rate, "[%]")) / 100,
    host_is_superhost_bool = ifelse(host_is_superhost == "TRUE", TRUE, FALSE)
  )

cat("Host response rate range:", 
    round(min(df$host_response_rate_num, na.rm = TRUE) * 100, 1), "% -",
    round(max(df$host_response_rate_num, na.rm = TRUE) * 100, 1), "%\n")
```

    ## Host response rate range: 0 % - 100 %

``` r
cat("Superhosts:", sum(df$host_is_superhost_bool, na.rm = TRUE), 
    "(", round(mean(df$host_is_superhost_bool, na.rm = TRUE) * 100, 1), "% )\n")
```

    ## Superhosts: 1373 ( 32.8 % )

Almost a third of all hosts are Superhosts, meaning they are top-rated,
experienced host for providing exceptional hospitality, meeting strict
criteria like high response rates (90%+), low cancellation rates (\<1%),
high overall ratings (4.8+), and hosting a minimum number of
stays/nights, earning perks like travel coupons and increased guest
trust.

## 2.3 Date Column Processing

``` r
# Convert date columns
df <- df |>
  mutate(
    host_since_date = as.Date(host_since),
    first_review_date = as.Date(first_review),
    last_review_date = as.Date(last_review),
    last_scraped_date = as.Date(last_scraped),
    # Calculate host experience in days
    host_experience_days = as.numeric(last_scraped_date - host_since_date),
    host_experience_years = host_experience_days / 365,
    # Time since first review
    days_since_first_review = as.numeric(last_scraped_date - first_review_date),
    # Time since last review
    days_since_last_review = as.numeric(last_scraped_date - last_review_date)
  )

cat("Host experience range:", round(min(df$host_experience_years, na.rm = TRUE), 1), 
    "-", round(max(df$host_experience_years, na.rm = TRUE), 1), "years\n")
```

    ## Host experience range: 0 - 16.8 years

## 2.4 Amenities Feature Engineering & Analysis

``` r
# Count amenities and identify key amenities
df <- df |>
  mutate(
    amenity_count = str_count(amenities, ",") + 1,
    has_wifi = str_detect(amenities, "(?i)wifi|(?i)internet"),
    has_kitchen = str_detect(amenities, "(?i)kitchen"),
    has_heating = str_detect(amenities, "(?i)heating"),
    has_ac = str_detect(amenities, "(?i)air conditioning|(?i)ac"),
    has_parking = str_detect(amenities, "(?i)parking|(?i)free parking"),
    has_washer = str_detect(amenities, "(?i)washer"),
    has_dryer = str_detect(amenities, "(?i)dryer"),
    has_tv = str_detect(amenities, "(?i)tv|(?i)television"),
    has_workspace = str_detect(amenities, "(?i)workspace|(?i)laptop"),
    has_gym = str_detect(amenities, "(?i)gym|(?i)fitness"),
    premium_amenity_score = has_wifi + has_kitchen + has_heating + has_ac + 
                           has_parking + has_washer + has_dryer + has_tv + 
                           has_workspace + has_gym
  )

amenities <- tibble(
  amenity = c("wifi", "kitchen", "heating", "ac", "parking", "washer", "dryer", "tv", "workspace", "gym"),
  listings_with_amenity = c(
    sum(df$has_wifi, na.rm = TRUE),
    sum(df$has_kitchen, na.rm = TRUE),
    sum(df$has_heating, na.rm = TRUE),
    sum(df$has_ac, na.rm = TRUE),
    sum(df$has_parking, na.rm = TRUE),
    sum(df$has_washer, na.rm = TRUE),
    sum(df$has_dryer, na.rm = TRUE),
    sum(df$has_tv, na.rm = TRUE),
    sum(df$has_workspace, na.rm = TRUE),
    sum(df$has_gym, na.rm = TRUE)
  ),
  pct_of_listings_with_amenity = c(
    round(mean(df$has_wifi, na.rm = TRUE) * 100, 2),
    round(mean(df$has_kitchen, na.rm = TRUE) * 100, 2),
    round(mean(df$has_heating, na.rm = TRUE) * 100, 2),
    round(mean(df$has_ac, na.rm = TRUE) * 100, 2),
    round(mean(df$has_parking, na.rm = TRUE) * 100, 2),
    round(mean(df$has_washer, na.rm = TRUE) * 100, 2),
    round(mean(df$has_dryer, na.rm = TRUE) * 100, 2),
    round(mean(df$has_tv, na.rm = TRUE) * 100, 2),
    round(mean(df$has_workspace, na.rm = TRUE) * 100, 2),
    round(mean(df$has_gym, na.rm = TRUE) * 100, 2)
  )
)

amenities
```

    ## # A tibble: 10 × 3
    ##    amenity   listings_with_amenity pct_of_listings_with_amenity
    ##    <chr>                     <int>                        <dbl>
    ##  1 wifi                       4377                         99.0
    ##  2 kitchen                    3851                         87.2
    ##  3 heating                    4154                         94  
    ##  4 ac                         4207                         95.2
    ##  5 parking                    2758                         62.4
    ##  6 washer                     3572                         80.8
    ##  7 dryer                      4052                         91.7
    ##  8 tv                         3851                         87.2
    ##  9 workspace                  2585                         58.5
    ## 10 gym                         719                         16.3

``` r
amenities_distribution <- amenities |>
  ggplot(aes(x = reorder(amenity, -pct_of_listings_with_amenity), y = pct_of_listings_with_amenity, fill = amenity)) +
  geom_bar(stat = "identity", fill = theme_colors$primary) +
  geom_text(aes(label = paste0(round(pct_of_listings_with_amenity, 1), "%")),
            vjust = -0.4, size = 3) +
  labs(title = "Distribution of Popular Amenities",
       x = "Amenity",
       y = "Percentage of Listings with Amenity") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        legend.position = "none",
        axis.text.x = element_text(angle = 15, hjust = 1),
        panel.grid.minor = element_blank())

amenities_distribution
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/process_amenities-1.png" style="display: block; margin: auto;" />

The selected amenities focus on features that are both frequently
present and plausibly monetizable (e.g., WiFi, kitchen, climate control,
parking, in-unit laundry, workspace, gym). This compact set avoids
overfitting to very rare amenities while still capturing a meaningful
notion of “amenity richness” that is interpretable for hosts and guests.

## 2.5 Revenue and Performance Metrics

``` r
# Create comprehensive performance metrics

review_subscores <- as.matrix(df[, c("review_scores_accuracy", "review_scores_cleanliness",
                                      "review_scores_checkin", "review_scores_communication",
                                      "review_scores_location", "review_scores_value")])


review_sub_mean <- rowMeans(review_subscores, na.rm = TRUE)
review_sub_mean[is.nan(review_sub_mean)] <- NA_real_


df <- df |>
  mutate(
    avg_review_score = ifelse(is.na(review_scores_rating),
                              review_sub_mean,
                              review_scores_rating),
    revenue_l365d = ifelse(is.na(estimated_revenue_l365d), 
                           price_clean * (365 - availability_365), 
                           estimated_revenue_l365d),
    revenue_per_bed = revenue_l365d / ifelse(beds > 0, beds, 1),
    revenue_per_bedroom = revenue_l365d / ifelse(bedrooms > 0, bedrooms, 1),
    occupancy_rate = ifelse(is.na(estimated_occupancy_l365d),
                            (365 - availability_365) / 365,
                            estimated_occupancy_l365d / 100),
    review_activity_score = reviews_per_month * avg_review_score,
    price_per_person = price_clean / accommodates,
    listing_efficiency = revenue_l365d / (price_clean + 1)  # Revenue per unit price
  )

df <- df |>
  mutate(
    reviews_per_month = ifelse(is.na(reviews_per_month), 0, reviews_per_month),
    number_of_reviews = ifelse(is.na(number_of_reviews), 0, number_of_reviews),
    occupancy_rate = ifelse(is.na(occupancy_rate) | occupancy_rate < 0, 0, 
                           ifelse(occupancy_rate > 1, 1, occupancy_rate))
  )

cat("Revenue metrics created successfully.\n")
```

    ## Revenue metrics created successfully.

``` r
cat("Average annual revenue: $", round(mean(df$revenue_l365d, na.rm = TRUE), 0), "\n")
```

    ## Average annual revenue: $ 25596

``` r
cat("Average occupancy rate:", round(mean(df$occupancy_rate, na.rm = TRUE) * 100, 1), "%\n")
```

    ## Average occupancy rate: 47.2 %

When Airbnb’s estimated revenue or occupancy figures were missing, we
approximated them using nightly price and availability under the
assumption of a constant price over the year. This introduces a mild
optimistic bias for highly seasonal listings but provides a consistent,
transparent way to recover performance metrics for a large share of the
dataset.

## 2.6 Neighbourhood Grouping

``` r
# Load neighbourhood boundaries GeoJSON for mapping (used in geographic sections)
neighbourhoods_geo <- st_read("data/neighbourhoods.geojson", quiet = TRUE)
cat("\nLoaded", nrow(neighbourhoods_geo), "neighbourhood boundaries from GeoJSON\n")
```

    ## 
    ## Loaded 26 neighbourhood boundaries from GeoJSON

## 2.7 Geographic Distribution Overview

``` r
# Basic geographic summary
cat("=== GEOGRAPHIC COVERAGE ===\n")
```

    ## === GEOGRAPHIC COVERAGE ===

``` r
cat("Latitude range:", round(min(df$latitude, na.rm = TRUE), 4), "-", 
    round(max(df$latitude, na.rm = TRUE), 4), "\n")
```

    ## Latitude range: 42.2353 - 42.3918

``` r
cat("Longitude range:", round(min(df$longitude, na.rm = TRUE), 4), "-", 
    round(max(df$longitude, na.rm = TRUE), 4), "\n")
```

    ## Longitude range: -71.174 - -70.996

``` r
cat("\nNeighbourhoods covered:", length(unique(df$neighbourhood_cleansed)), "\n")
```

    ## 
    ## Neighbourhoods covered: 25

``` r
df_map <- df |>
  filter(!is.na(latitude), !is.na(longitude), !is.na(price_clean),
         latitude > 42.2 & latitude < 42.4,
         longitude > -71.2 & longitude < -70.9)


df_map_sample <- df_map |>
  sample_n(min(2000, nrow(df_map)))


price_pal <- colorNumeric(
  palette = colorRampPalette(c(theme_colors$light, theme_colors$primary, 
                               theme_colors$accent, theme_colors$dark))(100),
  domain = df_map_sample$price_clean
)

price_range <- range(df_map_sample$price_clean, na.rm = TRUE)

visible_prices <- df_map_sample$price_clean[!is.na(df_map_sample$longitude)]

price_pal <- colorNumeric(
  palette = colorRampPalette(c(
    theme_colors$light,
    theme_colors$primary,
    theme_colors$accent,
    theme_colors$dark
  ))(100),
  domain = price_range
)

# Map 1: Price Distribution with Neighbourhood Boundaries
map_price <- leaflet() |>
  addProviderTiles(providers$CartoDB.Positron, options = providerTileOptions(opacity = 0.9)) |>
  addPolygons(
    data = neighbourhoods_geo,
    fillColor = "transparent",
    fillOpacity = 0,
    color = theme_colors$text,
    weight = 1.5,
    opacity = 0.6,
    label = ~neighbourhood,
    labelOptions = labelOptions(noHide = FALSE, textOnly = FALSE)
  ) |>
  addCircleMarkers(
    data = df_map_sample,
    lng = ~longitude,
    lat = ~latitude,
    radius = 4,
    color = ~price_pal(price_clean),
    fillColor = ~price_pal(price_clean),
    fillOpacity = 0.6,
    stroke = TRUE,
    weight = 0.5,
    popup = ~paste(
      "<b>Price:</b> $", round(price_clean, 0), "<br>",
      "<b>Room Type:</b>", room_type, "<br>",
      "<b>Accommodates:</b>", accommodates, "<br>",
      "<b>Neighbourhood:</b>", neighbourhood_cleansed, "<br>",
      "<b>Reviews:</b>", number_of_reviews, "<br>",
      "<b>Rating:</b>", round(review_scores_rating, 1), "/5.0"
    ),
    clusterOptions = markerClusterOptions(
      spiderfyOnMaxZoom = TRUE,
      showCoverageOnHover = TRUE,
      zoomToBoundsOnClick = TRUE
    )
  ) |>
  leaflet::addLegend(
    "bottomright",
    pal = price_pal,
    values = price_range,
    title = "Price ($)",
    opacity = 0.8
  )

map_price
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/geographic_overview-1.png" style="display: block; margin: auto;" />

## 2.8 Correlation Analysis

``` r
# Select numeric variables for correlation
numeric_vars <- df |>
  select(price_clean, accommodates, bedrooms, beds, bathrooms, 
         number_of_reviews, reviews_per_month, review_scores_rating,
         host_response_rate_num, host_listings_count,
         availability_365, occupancy_rate, revenue_l365d,
         amenity_count, premium_amenity_score,
         host_experience_years) |>
  drop_na() 


cor_matrix <- cor(numeric_vars)


corrplot(cor_matrix, 
         method = "color",
         type = "upper",
         order = "hclust",
         tl.col = theme_colors$text,
         tl.srt = 45,
         tl.cex = 0.7,
         addCoef.col = "black",
         number.cex = 0.6,
         col = colorRampPalette(c(theme_colors$dark, "white", theme_colors$primary))(200),
         title = "Correlation Matrix: Key Numeric Variables",
         mar = c(0, 0, 2, 0))
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/correlation_analysis-1.png" style="display: block; margin: auto;" />

``` r
cat("\n=== STRONG CORRELATIONS (|r| > 0.5) ===\n")
```

    ## 
    ## === STRONG CORRELATIONS (|r| > 0.5) ===

``` r
cor_pairs <- expand.grid(row = rownames(cor_matrix), col = colnames(cor_matrix)) |>
  mutate(cor_value = as.vector(cor_matrix)) |>
  filter(row != col, abs(cor_value) > 0.5) |>
  arrange(desc(abs(cor_value))) |>
  distinct(cor_value, .keep_all = TRUE) |>
  head(10)

print(cor_pairs)
```

    ##                     row               col cor_value
    ## 1                  beds      accommodates 0.8689910
    ## 2              bedrooms      accommodates 0.8203380
    ## 3                  beds          bedrooms 0.7976233
    ## 4         revenue_l365d       price_clean 0.7054785
    ## 5 premium_amenity_score     amenity_count 0.6327915
    ## 6             bathrooms          bedrooms 0.5790858
    ## 7     reviews_per_month number_of_reviews 0.5773609
    ## 8             bathrooms      accommodates 0.5396559
    ## 9             bathrooms              beds 0.5348394

# 3. Host Analysis

## 3.1 Superhosts vs. Regular Hosts Performance

``` r
# Compare superhosts vs regular hosts
host_comparison <- df |>
  filter(!is.na(host_is_superhost_bool)) |>
  group_by(host_is_superhost_bool) |>
  summarise(
    n_listings = n(),
    avg_rating = mean(review_scores_rating, na.rm = TRUE),
    avg_revenue = mean(revenue_l365d, na.rm = TRUE),
    avg_occupancy = mean(occupancy_rate, na.rm = TRUE) * 100,
    avg_response_rate = mean(host_response_rate_num, na.rm = TRUE) * 100,
    pct_entire_home = mean(room_type == "Entire home/apt", na.rm = TRUE) * 100,
    .groups = "drop"
  ) |>
  mutate(host_type = ifelse(host_is_superhost_bool, "Superhost", "Regular Host"))

cat("=== SUPERHOST VS REGULAR HOST COMPARISON ===\n")
```

    ## === SUPERHOST VS REGULAR HOST COMPARISON ===

``` r
print(host_comparison)
```

    ## # A tibble: 2 × 8
    ##   host_is_superhost_bool n_listings avg_rating avg_revenue avg_occupancy
    ##   <lgl>                       <int>      <dbl>       <dbl>         <dbl>
    ## 1 FALSE                        2817       4.64      22604.          35.2
    ## 2 TRUE                         1373       4.86      31627.          70.3
    ## # ℹ 3 more variables: avg_response_rate <dbl>, pct_entire_home <dbl>,
    ## #   host_type <chr>

``` r
# Visualization
p1 <- host_comparison |>
  select(host_type, avg_rating, avg_revenue, avg_occupancy) |>
  pivot_longer(cols = -host_type, names_to = "metric", values_to = "value") |>
  mutate(metric = str_remove_all(metric, "avg_") |>
                    str_replace_all("_", " ") |>
                    str_to_title(),
         metric = ifelse(metric == "Revenue", "Revenue ($)", 
                        ifelse(metric == "Occupancy", "Occupancy (%)",
                              metric))) |>
  ggplot(aes(x = metric, y = value, fill = host_type)) +
  geom_bar(stat = "identity", position = "dodge", alpha = 0.8) +
  scale_fill_manual(values = c("Superhost" = theme_colors$accent,
                               "Regular Host" = theme_colors$primary)) +
  facet_wrap(~metric, scales = "free", ncol = 4) +
  labs(title = "Superhost vs Regular Host Performance",
       x = "",
       y = "Value",
       fill = "Host Type") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        legend.position = "bottom",
        axis.text.x = element_blank(),
        strip.text = element_text(face = "bold"))

print(p1)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/superhost_analysis-1.png" style="display: block; margin: auto;" />

## 3.2 Average Revenue by Host Response Rate

``` r
df_clean <- df |>
  filter(!is.na(host_response_rate_num),
         !is.na(revenue_l365d),
         host_response_rate_num >= 0,
         host_response_rate_num <= 1,
         !is.na(host_is_superhost_bool)) |>
  mutate(
    host_type = if_else(host_is_superhost_bool, "Superhost", "Regular host")
  )

df_buckets_type <- df_clean |>
  mutate(response_bucket = cut(
    host_response_rate_num,
    breaks = c(0, 0.5, 0.7, 0.9, 0.99, 1),
    labels = c("0–50%", "50–70%", "70–90%", "90–99%", "100%"),
    include.lowest = TRUE
  )) |>
  group_by(response_bucket, host_type) |>
  summarise(
    mean_revenue = mean(revenue_l365d, na.rm = TRUE),
    n = n(),
    .groups = "drop"
  ) |>
  filter(!is.na(response_bucket))

p2 <- ggplot(df_buckets_type,
       aes(x = response_bucket, y = mean_revenue, fill = host_type)) +
  geom_col(position = "dodge", alpha = 0.85, color = "white") +
  scale_y_continuous(labels = scales::dollar_format()) +
  scale_fill_manual(values = c(
    "Regular host" = theme_colors$primary,
    "Superhost"    = theme_colors$accent
  )) +
  labs(
    title = "Average Annual Revenue by Response Rate and Host Type",
    x = "Host Response Rate",
    y = "Average Annual Revenue ($)",
    fill = "Host Type"
  ) +
  theme_transparent() +
  theme(
    axis.text.x = element_text(angle = 15, hjust = 1),
    legend.position = "bottom"
  )

print(p2)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/host_type_revenue-1.png" style="display: block; margin: auto;" />

## 3.3 Average Revenue by Host Portfolio

``` r
host_portfolio_summary <- df |>
  filter(!is.na(host_listings_count),
         host_listings_count > 0,
         !is.na(revenue_l365d),
         !is.na(host_is_superhost_bool)) |>
  mutate(
    portfolio_group = case_when(
      host_listings_count == 1        ~ "Single",
      host_listings_count <= 3        ~ "Small (2–3)",
      host_listings_count <= 10       ~ "Medium (4–10)",
      host_listings_count <= 50       ~ "Large (11–50)",
      TRUE                            ~ "Very Large (50+)"
    ),
    portfolio_group = factor(
      portfolio_group,
      levels = c("Single", "Small (2–3)", "Medium (4–10)",
                 "Large (11–50)", "Very Large (50+)")
    ),
    host_type = if_else(host_is_superhost_bool, "Superhost", "Regular host")
  ) |>
  group_by(portfolio_group, host_type) |>
  summarise(
    mean_rev   = mean(revenue_l365d, na.rm = TRUE),
    n_listings = n(),
    .groups    = "drop"
  )

p3 <- ggplot(host_portfolio_summary,
       aes(x = portfolio_group, y = mean_rev, fill = host_type)) +
  geom_col(position = "dodge", alpha = 0.85, color = "white") +
  scale_y_continuous(labels = scales::dollar_format()) +
  scale_fill_manual(values = c(
    "Regular host" = theme_colors$primary,
    "Superhost"    = theme_colors$accent
  )) +
  labs(
    title    = "Average Revenue per Listing by Portfolio Size and Host Type",
    x        = "Portfolio Size",
    y        = "Avg Annual Revenue ($)",
    fill     = "Host Type",
    subtitle = "Comparing regular hosts and Superhosts at each portfolio size"
  ) +
  theme_transparent() +
  theme(
    legend.position = "bottom",
    axis.text.x     = element_text(angle = 15, hjust = 1)
  )

print(p3)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/portfolio_vs_listing_revenue_host_type-1.png" style="display: block; margin: auto;" />

## 3.4 Annual Revenue Time Series by Host Experience

``` r
host_exp_perf <- df |>
filter(!is.na(host_experience_years),
host_experience_years >= 0,
host_experience_years <= 15)  # trim extreme values

p4 <- ggplot(host_exp_perf, aes(x = host_experience_years,
                          y = revenue_l365d)) +
  geom_point(alpha = 0.15, color = theme_colors$primary, size = 1) +
  geom_smooth(method = "loess", color = theme_colors$accent, 
              linewidth = 1.4, se = TRUE) +
  scale_y_log10(labels = scales::dollar_format(),
                breaks = c(1000, 5000, 10000, 25000, 50000, 
                           100000, 250000, 500000, 1000000)) +
  labs(
    title = "Host Experience vs. Annual Revenue",
    subtitle = "Log scale reveals trend hidden by extreme outliers",
    x = "Years as Host",
    y = "Annual Revenue ($, log scale)"
  ) +
  theme_transparent()

print(p4)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/time_series_experience-1.png" style="display: block; margin: auto;" />

This shows that most hosts—no matter how long they’ve been hosting—earn
roughly similar annual revenue.

# 4. Property Analysis

## 4.1 Room and Property Type Analysis

``` r
# Room type distribution
room_type_dist <- df |>
  select(room_type) |>
  filter(!is.na(room_type)) |>
  count(room_type) |>
  mutate(pct_of_listings = round(n/sum(n) * 100, 2)) |>
  arrange(desc(pct_of_listings))

room_type_dist
```

    ## # A tibble: 4 × 3
    ##   room_type           n pct_of_listings
    ##   <chr>           <int>           <dbl>
    ## 1 Entire home/apt  2990           67.7 
    ## 2 Private room     1366           30.9 
    ## 3 Hotel room         58            1.31
    ## 4 Shared room         5            0.11

``` r
room_type_plot <-
  ggplot(room_type_dist, aes(x = reorder(room_type, -pct_of_listings), y = pct_of_listings, fill = room_type)) +
  geom_col(width = 0.8, color = "white", linewidth = 0.5) + 
  geom_text(aes(label = paste0(round(pct_of_listings, 2), "%")),
            position = position_stack(vjust = 0.8), size = 4) +
  scale_fill_manual(values = c(theme_colors$primary, theme_colors$secondary, theme_colors$light, theme_colors$dark)) +
  labs(title = "Distribution of Room Types",
       x = "Room Type", y = "Percentage of Listings") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        legend.position = "none",
        axis.text.x = element_text(angle = 15, hjust = 1),
        panel.grid.minor = element_blank())
```

``` r
# Property type distribution
unique(df$property_type)
```

    ##  [1] "Entire rental unit"                 "Entire guest suite"                
    ##  [3] "Entire condo"                       "Private room in home"              
    ##  [5] "Entire home"                        "Private room in rental unit"       
    ##  [7] "Private room in condo"              "Private room in townhouse"         
    ##  [9] "Entire townhouse"                   "Houseboat"                         
    ## [11] "Entire loft"                        "Private room in bed and breakfast" 
    ## [13] "Entire guesthouse"                  "Private room in villa"             
    ## [15] "Private room in guest suite"        "Private room in bungalow"          
    ## [17] "Private room in loft"               "Shared room in condo"              
    ## [19] "Entire serviced apartment"          "Boat"                              
    ## [21] "Shared room in home"                "Private room in guesthouse"        
    ## [23] "Room in boutique hotel"             "Room in hotel"                     
    ## [25] "Private room"                       "Private room in serviced apartment"
    ## [27] "Shared room in rental unit"         "Entire place"                      
    ## [29] "Private room in casa particular"    "Entire vacation home"              
    ## [31] "Private room in hostel"             "Tiny home"

``` r
df$property_type <- case_when(
  str_detect(df$property_type, "rental unit") ~ "rental unit",
  str_detect(df$property_type, "condo") ~ "condo",
  str_detect(df$property_type, "home") ~ "home",
  str_detect(df$property_type, "townhouse") ~ "townhouse",
  str_detect(df$property_type, "guesthouse") ~ "guesthouse",
  str_detect(df$property_type, "serviced apartment") ~ "serviced apartment",
  str_detect(df$property_type, "hostel") ~ "hostel",
  str_detect(df$property_type, "hotel") ~ "hotel",
  str_detect(df$property_type, "guest suite") ~ "guest suite",
  TRUE ~ "other"
)

property_type_dist <- df |>
  select(property_type) |>
  filter(!is.na(property_type)) |>
  count(property_type) |>
  mutate(pct_of_listings = round(n/sum(n) * 100, 2)) |>
  arrange(desc(pct_of_listings))

property_type_dist
```

    ## # A tibble: 10 × 3
    ##    property_type          n pct_of_listings
    ##    <chr>              <int>           <dbl>
    ##  1 rental unit         2870           65.0 
    ##  2 home                 604           13.7 
    ##  3 condo                381            8.62
    ##  4 hotel                206            4.66
    ##  5 serviced apartment   114            2.58
    ##  6 townhouse             80            1.81
    ##  7 guest suite           79            1.79
    ##  8 other                 79            1.79
    ##  9 guesthouse             5            0.11
    ## 10 hostel                 1            0.02

``` r
property_type_plot <-
  ggplot(property_type_dist, aes(x = reorder(property_type, pct_of_listings), y = pct_of_listings, fill = property_type)) +
  geom_col(width = 0.8, color = "white", linewidth = 0.5) + 
  geom_text(aes(label = paste0(round(pct_of_listings, 4), "%")),
            position = position_stack(vjust = 0.8), size = 4) +
  scale_fill_manual(values = c(theme_colors$primary, theme_colors$secondary, theme_colors$light, theme_colors$dark, theme_colors$accent, theme_colors$gradient_start, theme_colors$gradient_end, theme_colors$highlight, theme_colors$text, theme_colors$additional)) +
  coord_flip() +
  labs(title = "Distribution of Property Types",
       x = "Room Type", y = "Percentage of Listings") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        legend.position = "none",
        axis.text.x = element_text(angle = 15, hjust = 1),
        panel.grid.minor = element_blank())
```

``` r
grid.arrange(room_type_plot, property_type_plot, ncol=2)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/room_property_type_plots-1.png" style="display: block; margin: auto;" />

``` r
review_value_by_prop_and_room_type <- df |>
  select(property_type, room_type, review_scores_value) |>
  filter(!is.na(review_scores_value)) |>
  group_by(property_type, room_type) |>
  summarize(num_listings = n(),
            avg_value = round(mean(review_scores_value), 2)) |>
  filter(num_listings >= 50) |>
  arrange(desc(avg_value))

review_value_by_prop_and_room_type
```

    ## # A tibble: 9 × 4
    ## # Groups:   property_type [6]
    ##   property_type      room_type       num_listings avg_value
    ##   <chr>              <chr>                  <int>     <dbl>
    ## 1 guest suite        Entire home/apt           65      4.78
    ## 2 condo              Private room              70      4.76
    ## 3 home               Entire home/apt          201      4.76
    ## 4 condo              Entire home/apt          263      4.72
    ## 5 home               Private room             338      4.69
    ## 6 rental unit        Entire home/apt         1681      4.56
    ## 7 rental unit        Private room             431      4.51
    ## 8 hotel              Private room              94      4.44
    ## 9 serviced apartment Entire home/apt          100      4.44

``` r
ggplot(review_value_by_prop_and_room_type, aes(x = reorder(property_type, avg_value), y = room_type, fill = avg_value)) +
  geom_tile(color = "white", 
            linewidth = 0) +
  scale_fill_gradient(low = theme_colors$gradient_start, 
                      high = theme_colors$gradient_end) +
  coord_fixed() +
  labs(
    title = "Average Review Score Value by Property and Room Type",
    x = "Room Type",
    y = "Property Type",
    fill = "Avg Review Score Value"
  ) +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 12, color = theme_colors$text),
        panel.grid.minor = element_blank())
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/prop_room_type_values-1.png" style="display: block; margin: auto;" />

## 4.2 Property Location Distributions

``` r
top10_neighbourhoods_by_listings <- df |>
  select(neighbourhood_cleansed) |>
  count(neighbourhood_cleansed) |>
  mutate(pct = round(n/sum(n) * 100, 2)) |>
  arrange(desc(pct)) |>
  slice_head(n = 10)

top10_neighbourhoods_by_listings
```

    ## # A tibble: 10 × 3
    ##    neighbourhood_cleansed     n   pct
    ##    <chr>                  <int> <dbl>
    ##  1 Dorchester               572 12.9 
    ##  2 Downtown                 376  8.51
    ##  3 Back Bay                 340  7.69
    ##  4 Brighton                 303  6.86
    ##  5 Roxbury                  295  6.68
    ##  6 South End                291  6.59
    ##  7 Fenway                   241  5.45
    ##  8 Jamaica Plain            240  5.43
    ##  9 Beacon Hill              230  5.2 
    ## 10 East Boston              229  5.18

``` r
p1 <- top10_neighbourhoods_by_listings |>
  ggplot(aes(x = reorder(neighbourhood_cleansed, pct), 
             y = pct, 
             fill = pct)) +
  geom_bar(stat = "identity", 
           color = "white", 
           alpha = 0.8, 
           linewidth = 0.5) +
  geom_text(aes(label = paste0(round(pct, 2), "%")),
            position = position_stack(vjust = 0.9), size = 4) +
  coord_flip() +
  scale_fill_gradient(low = theme_colors$light, 
                      high = theme_colors$dark) +
  labs(title = "Top 10 Neighbourhoods by Number of Listings",
       x = "Neighbourhood", 
       y = "Percentage of Listings") +
  theme_transparent() +
  theme(
    legend.position = "none",
    plot.title = element_text(face = "bold",
                              size = 12,
                              color = theme_colors$text),
    panel.grid.minor = element_blank()
  )
```

``` r
top10_neighbourhoods_by_avg_num_of_reviews <- df |>
  filter(!is.na(number_of_reviews)) |>
  group_by(neighbourhood_cleansed) |>
  summarize(avg_num_reviews = round(mean(number_of_reviews), 0)) |>
  arrange(desc(avg_num_reviews)) |>
  head(10)  

top10_neighbourhoods_by_avg_num_of_reviews
```

    ## # A tibble: 10 × 2
    ##    neighbourhood_cleansed avg_num_reviews
    ##    <chr>                            <dbl>
    ##  1 East Boston                         88
    ##  2 Downtown                            74
    ##  3 Jamaica Plain                       71
    ##  4 Charlestown                         68
    ##  5 South End                           68
    ##  6 Roxbury                             61
    ##  7 South Boston                        58
    ##  8 Fenway                              51
    ##  9 Mission Hill                        50
    ## 10 North End                           50

``` r
p2 <- top10_neighbourhoods_by_avg_num_of_reviews |>
  ggplot(aes(x = reorder(neighbourhood_cleansed, avg_num_reviews), 
                              y = avg_num_reviews, 
                              fill = avg_num_reviews)) +
  geom_bar(stat = "identity", 
           alpha = 0.8, 
           color = "white", 
           linewidth = 0.5) +
  geom_text(aes(label = avg_num_reviews),
            position = position_stack(vjust = 0.9), size = 4) +
  coord_flip() +
  scale_fill_gradient(low = theme_colors$light, 
                      high = theme_colors$dark) +
  labs(title = "Top 10 Neighbourhoods by Average Number of Reviews Per Listing",
       x = "Neighborhood",
       y = "Average Number of Reviews Per Listing",
       fill = "Avg Reviews") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 12, color = theme_colors$text),
        panel.grid.minor = element_blank(),
        legend.position = "none")
```

``` r
top10_neighbourhoods_by_avg_num_of_monthly_reviews <- df |>
  filter(!is.na(reviews_per_month)) |>
  group_by(neighbourhood_cleansed) |>
  summarize(avg_monthly_reviews = round(mean(reviews_per_month), 2)) |>
  arrange(desc(avg_monthly_reviews)) |>
  head(10)  

top10_neighbourhoods_by_avg_num_of_monthly_reviews
```

    ## # A tibble: 10 × 2
    ##    neighbourhood_cleansed avg_monthly_reviews
    ##    <chr>                                <dbl>
    ##  1 East Boston                           2.37
    ##  2 Downtown                              2.08
    ##  3 North End                             1.68
    ##  4 Fenway                                1.57
    ##  5 Roxbury                               1.49
    ##  6 South Boston                          1.3 
    ##  7 Brighton                              1.28
    ##  8 South End                             1.28
    ##  9 Back Bay                              1.25
    ## 10 Mission Hill                          1.24

``` r
p3 <- top10_neighbourhoods_by_avg_num_of_monthly_reviews |>
  ggplot(aes(x = reorder(neighbourhood_cleansed, avg_monthly_reviews), 
                              y = avg_monthly_reviews, 
                              fill = avg_monthly_reviews)) +
  geom_bar(stat = "identity", 
           alpha = 0.8, color = "white", 
           linewidth = 0.5) +
  geom_text(aes(label = avg_monthly_reviews),
            position = position_stack(vjust = 0.9), size = 4) +
  coord_flip() +
  scale_fill_gradient(low = theme_colors$light, 
                      high = theme_colors$dark) +
  labs(title = "Top 10 Neighbourhoods by Average Number of Monthly Reviews Per Listing",
       x = "Neighborhood",
       y = "Average Number of Monthly Reviews Per Listing",
       fill = "Avg Monthly Reviews") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 12, color = theme_colors$text),
        panel.grid.minor = element_blank(),
        legend.position = "none")
```

``` r
top10_neighbourhoods_by_location_scores <- df |>
  filter(!is.na(review_scores_location)) |>
  group_by(neighbourhood_cleansed) |>
  summarize(
    n = n(),
    avg_location_score = round(mean(review_scores_location), 2)) |>
  arrange(desc(avg_location_score)) |>
  filter(n >= 30) |>
  head(10)
  
top10_neighbourhoods_by_location_scores
```

    ## # A tibble: 10 × 3
    ##    neighbourhood_cleansed      n avg_location_score
    ##    <chr>                   <int>              <dbl>
    ##  1 Beacon Hill               174               4.94
    ##  2 Back Bay                  255               4.92
    ##  3 Fenway                    181               4.92
    ##  4 North End                  87               4.9 
    ##  5 South Boston Waterfront    33               4.9 
    ##  6 South End                 234               4.9 
    ##  7 West Roxbury               68               4.86
    ##  8 Charlestown                72               4.85
    ##  9 Jamaica Plain             201               4.81
    ## 10 Downtown                  276               4.8

``` r
p4 <- top10_neighbourhoods_by_location_scores |>
  ggplot(aes(x = reorder(neighbourhood_cleansed, avg_location_score), 
                              y = avg_location_score, 
                              fill = avg_location_score)) +
  geom_bar(stat = "identity", 
           alpha = 0.8, color = "white", 
           linewidth = 0.5) +
  geom_text(aes(label = avg_location_score),
            position = position_stack(vjust = 0.9), size = 4) +
  coord_flip() +
  scale_fill_gradient(low = theme_colors$light, 
                      high = theme_colors$dark) +
  labs(title = "Top 10 Neighbourhoods by Average Location Score",
       x = "Neighborhood",
       y = "Average Location Score",
       fill = "Avg Location Score") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 12, color = theme_colors$text),
        panel.grid.minor = element_blank(),
        legend.position = "none")
```

``` r
grid.arrange(p1, p2, p3, p4, ncol = 1)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/plot_location_distributions-1.png" style="display: block; margin: auto;" />

## 4.3 Property Type and Amenities

``` r
# Property type and number of amenities

property_amenities <- df |>
  select(property_type, amenity_count, review_scores_value) |>
  filter(!is.na(review_scores_value)) |>
  group_by(property_type) |>
  summarize(
    num_listings = n(),
    avg_num_amenities = round(mean(amenity_count), 0),
    avg_review_score_val = round(mean(review_scores_value), 2)
  ) |>
  filter(num_listings >= 50)
    
property_amenities
```

    ## # A tibble: 8 × 4
    ##   property_type      num_listings avg_num_amenities avg_review_score_val
    ##   <chr>                     <int>             <dbl>                <dbl>
    ## 1 condo                       335                41                 4.73
    ## 2 guest suite                  76                40                 4.76
    ## 3 home                        540                37                 4.72
    ## 4 hotel                       117                23                 4.45
    ## 5 other                        67                35                 4.63
    ## 6 rental unit                2114                36                 4.55
    ## 7 serviced apartment          113                38                 4.4 
    ## 8 townhouse                    78                40                 4.79

``` r
property_amenities_plot <-
  ggplot(property_amenities, 
         aes(x = reorder(property_type, -avg_review_score_val), 
             y = avg_num_amenities, 
             fill = avg_review_score_val)) +
  geom_col(width = 0.8, 
           color = "white", 
           linewidth = 0.5) + 
  geom_text(aes(label = avg_num_amenities),
            position = position_stack(vjust = 0.9), 
            size = 4) +
  scale_fill_gradient(low = theme_colors$light, 
                      high = theme_colors$dark) +
  labs(title = "Property Types by Average Amenity Count and Average Review Score Value",
       x = "Property Type", 
       y = "Amenity Count",
       fill = "Average Review Score Value") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        axis.text.x = element_text(angle = 15, hjust = 1),
        panel.grid.minor = element_blank())

property_amenities_plot
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/property_and_amenities-1.png" style="display: block; margin: auto;" />

# 5. Price and Revenue Analysis

## 5.1 Univariate Analysis - Price Distribution

``` r
# Price distribution analysis
p1 <- df |>
  filter(!is.na(price_clean)) |>
  ggplot(aes(x = price_clean)) +
  geom_histogram(bins = 80, fill = theme_colors$primary, alpha = 0.7, 
                 color = "white", linewidth = 0.3) +
  scale_x_continuous(labels = dollar_format(), 
                     breaks = seq(0, 1000, 100),
                     limits = c(0, 500)) +
  labs(title = "Price Distribution (Limited to $500 for Clarity)",
       x = "Price per Night ($)",
       y = "Frequency",
       subtitle = paste("Mean: $", round(mean(df$price_clean, na.rm = TRUE), 2),
                       "| Median: $", round(median(df$price_clean, na.rm = TRUE), 2))) +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        plot.subtitle = element_text(color = theme_colors$text),
        panel.grid.minor = element_blank())

p2 <- df |>
  filter(!is.na(price_clean)) |>
  ggplot(aes(y = price_clean, x = "")) +
  geom_violin(fill = theme_colors$secondary, alpha = 0.6, color = theme_colors$dark) +
  geom_boxplot(width = 0.2, fill = theme_colors$accent, alpha = 0.8, 
               outlier.alpha = 0.3, outlier.size = 0.5) +
  scale_y_continuous(labels = dollar_format(), limits = c(0, 400)) +
  labs(title = "Price Distribution (Violin Plot)",
       y = "Price per Night ($)",
       x = "") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        axis.text.x = element_blank(),
        panel.grid.minor = element_blank())

p3 <- df |>
  filter(!is.na(price_category)) |>
  count(price_category) |>
  mutate(pct = n / sum(n) * 100) |>
  ggplot(aes(x = price_category, y = pct, fill = price_category)) +
  geom_bar(stat = "identity", alpha = 0.8, color = "white", linewidth = 0.5) +
  scale_fill_manual(values = c("Budget" = theme_colors$light,
                               "Economy" = theme_colors$primary,
                               "Mid-range" = theme_colors$secondary,
                               "Premium" = theme_colors$accent,
                               "Luxury" = theme_colors$dark)) +
  geom_text(aes(label = paste0(round(pct, 1), "%")), 
            vjust = -0.5, fontface = "bold", color = theme_colors$text) +
  labs(title = "Price Category Distribution",
       x = "Price Category",
       y = "Percentage (%)") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        legend.position = "none",
        panel.grid.minor = element_blank())

plot_price <- grid.arrange(p1, p2, p3, ncol = 3, widths = c(1.2, 0.8, 1))
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/price_distribution-1.png" style="display: block; margin: auto;" />

``` r
print(plot_price)
```

    ## TableGrob (1 x 3) "arrange": 3 grobs
    ##   z     cells    name           grob
    ## 1 1 (1-1,1-1) arrange gtable[layout]
    ## 2 2 (1-1,2-2) arrange gtable[layout]
    ## 3 3 (1-1,3-3) arrange gtable[layout]

Overall, the pricing analysis suggests that Boston’s short-term rental
market is characterized by a wide variety of offerings, ranging from
affordable units to high-end luxury stays. This variability supports the
use of clustering methods in later sections, as distinct pricing tiers
likely reflect meaningful differences in property characteristics and
host strategies.

## 5.2 Bivariate Analysis - Price vs Key Variables

``` r
# Price by room type
p1 <- df |>
  filter(!is.na(price_clean), !is.na(room_type)) |>
  ggplot(aes(x = room_type, y = price_clean, fill = room_type)) +
  geom_violin(alpha = 0.6, color = theme_colors$dark) +
  geom_boxplot(width = 0.2, alpha = 0.8, outlier.alpha = 0.2, outlier.size = 0.3) +
  scale_fill_manual(values = c("Entire home/apt" = theme_colors$primary,
                               "Private room" = theme_colors$secondary,
                               "Shared room" = theme_colors$accent)) +
  scale_y_continuous(labels = dollar_format(), limits = c(0, 400)) +
  labs(title = "Price Distribution by Room Type",
       x = "Room Type",
       y = "Price per Night ($)") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        legend.position = "none",
        axis.text.x = element_text(angle = 15, hjust = 1),
        panel.grid.minor = element_blank())

# Price by accommodates
p2 <- df |>
  filter(!is.na(price_clean), !is.na(accommodates), accommodates <= 8) |>
  group_by(accommodates) |>
  summarise(
    mean_price = mean(price_clean, na.rm = TRUE),
    median_price = median(price_clean, na.rm = TRUE),
    n = n()
  ) |>
  ggplot(aes(x = accommodates, y = mean_price)) +
  geom_line(color = theme_colors$primary, linewidth = 1.5, alpha = 0.8) +
  geom_point(aes(size = n), color = theme_colors$accent, alpha = 0.7) +
  scale_y_continuous(labels = dollar_format()) +
  scale_size_continuous(range = c(2, 8), name = "Listings") +
  labs(title = "Average Price by Accommodation Capacity",
       x = "Number of Guests",
       y = "Average Price ($)") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        panel.grid.minor = element_blank())

# Price by bedrooms
p3 <- df |>
  filter(!is.na(price_clean), !is.na(bedrooms), bedrooms <= 5) |>
  group_by(bedrooms) |>
  summarise(
    mean_price = mean(price_clean, na.rm = TRUE),
    median_price = median(price_clean, na.rm = TRUE),
    n = n()
  ) |>
  ggplot(aes(x = bedrooms, y = mean_price)) +
  geom_col(fill = theme_colors$secondary, alpha = 0.8, color = "white", linewidth = 0.5) +
  geom_text(aes(label = dollar(round(mean_price, 0))), 
            vjust = -0.5, fontface = "bold", color = theme_colors$text) +
  scale_y_continuous(labels = dollar_format()) +
  labs(title = "Average Price by Number of Bedrooms",
       x = "Number of Bedrooms",
       y = "Average Price ($)") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        panel.grid.minor = element_blank())

plot_price_2 <- grid.arrange(p1, p2, p3, ncol = 3)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/price_bivariate-1.png" style="display: block; margin: auto;" />

``` r
print(plot_price_2)
```

    ## TableGrob (1 x 3) "arrange": 3 grobs
    ##   z     cells    name           grob
    ## 1 1 (1-1,1-1) arrange gtable[layout]
    ## 2 2 (1-1,2-2) arrange gtable[layout]
    ## 3 3 (1-1,3-3) arrange gtable[layout]

- **Room type** strongly segments the market into budget (shared rooms),
  mid-range (private rooms), and high-value (entire homes) categories.
- **Accommodation capacity** drives prices upward, particularly for
  large-group listings, which form a small but highly priced niche.
- **Bedroom count** is closely associated with price increases,
  reinforcing its role as a key structural determinant of listing value.

## 5.3 Detailed Interactive Map with Revenue Analysis

``` r
# Create interactive map with revenue analysis
df_map_detailed <- df_map |>
  filter(!is.na(price_clean), !is.na(revenue_l365d)) |>
  sample_n(min(2000, nrow(df_map)))

# Color palette for revenue
revenue_pal <- colorNumeric(
  palette = colorRampPalette(c(theme_colors$light, theme_colors$primary, 
                               theme_colors$accent, theme_colors$dark))(100),
  domain = df_map_detailed$revenue_l365d
)

# Create detailed map with neighbourhood boundaries
map_revenue <- leaflet() |>
  addProviderTiles(providers$CartoDB.Positron, options = providerTileOptions(opacity = 0.9)) |>
  addPolygons(
    data = neighbourhoods_geo,
    fillColor = "transparent",
    fillOpacity = 0,
    color = theme_colors$text,
    weight = 1.5,
    opacity = 0.6,
    label = ~neighbourhood,
    labelOptions = labelOptions(noHide = FALSE, textOnly = FALSE)
  ) |>
  addCircleMarkers(
    data = df_map_detailed,
    lng = ~longitude,
    lat = ~latitude,
    radius = ~sqrt(revenue_l365d / 5000),
    color = ~revenue_pal(revenue_l365d),
    fillColor = ~revenue_pal(revenue_l365d),
    fillOpacity = 0.7,
    stroke = TRUE,
    weight = 0.5,
    popup = ~paste(
      "<b>Price:</b> $", round(price_clean, 0), "<br>",
      "<b>Annual Revenue:</b> $", round(revenue_l365d, 0), "<br>",
      "<b>Room Type:</b>", room_type, "<br>",
      "<b>Occupancy Rate:</b>", round(occupancy_rate * 100, 1), "%", "<br>",
      "<b>Neighbourhood:</b>", neighbourhood_cleansed, "<br>",
      "<b>Rating:</b>", round(review_scores_rating, 1), "/5.0"
    ),
    clusterOptions = markerClusterOptions(
      spiderfyOnMaxZoom = TRUE,
      showCoverageOnHover = TRUE,
      zoomToBoundsOnClick = TRUE
    )
  )# |>
  # leaflet::addLegend(
  #   "bottomright",
  #   pal = revenue_pal,
  #   values = df_map_detailed$revenue_l365d,
  #   title = "Annual Revenue ($)",
  #   opacity = 0.8,
  #   labFormat = labelFormat(prefix = "$", big.mark = ",")
  # )

map_revenue
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/interactive_map-1.png" style="display: block; margin: auto;" />

## 5.4 Revenue Optimization Analysis

``` r
# Analyze revenue drivers
revenue_analysis <- df |>
  filter(!is.na(revenue_l365d), !is.na(price_clean), revenue_l365d > 0) |>
  mutate(
    price_bucket = cut(price_clean, 
                      breaks = quantile(price_clean, probs = seq(0, 1, 0.2), na.rm = TRUE),
                      include.lowest = TRUE,
                      labels = c("Q1", "Q2", "Q3", "Q4", "Q5")),
    occupancy_bucket = cut(occupancy_rate,
                          breaks = c(0, 0.2, 0.4, 0.6, 0.8, 1),
                          include.lowest = TRUE,
                          labels = c("0-20%", "20-40%", "40-60%", "60-80%", "80-100%"))
  )

# Revenue by price and occupancy
revenue_summary <- revenue_analysis |>
  group_by(price_bucket, occupancy_bucket) |>
  summarise(
    avg_revenue = mean(revenue_l365d, na.rm = TRUE),
    median_revenue = median(revenue_l365d, na.rm = TRUE),
    n = n(),
    .groups = "drop"
  ) |>
  filter(n >= 10)

p1 <- ggplot(revenue_summary, 
             aes(x = price_bucket, y = occupancy_bucket, fill = avg_revenue)) +
  geom_tile(color = "white", linewidth = 0.5) +
  scale_fill_gradient(low = theme_colors$light, high = theme_colors$dark,
                     labels = dollar_format()) +
  labs(title = "Average Annual Revenue Heatmap",
       x = "Price Quintile",
       y = "Occupancy Rate",
       fill = "Revenue ($)") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        panel.grid = element_blank())

print(p1)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/revenue_analysis-1.png" style="display: block; margin: auto;" />

``` r
# Optimal pricing analysis
optimal_pricing <- revenue_analysis |>
  filter(occupancy_rate >= 0.5) |>
  group_by(price_category) |>
  summarise(
    avg_revenue = mean(revenue_l365d, na.rm = TRUE),
    avg_occupancy = mean(occupancy_rate, na.rm = TRUE) * 100,
    avg_price = mean(price_clean, na.rm = TRUE),
    n = n(),
    .groups = "drop"
  ) |>
  arrange(desc(avg_revenue))

cat("\n=== REVENUE BY PRICE CATEGORY (50%+ Occupancy) ===\n")
```

    ## 
    ## === REVENUE BY PRICE CATEGORY (50%+ Occupancy) ===

``` r
print(optimal_pricing)
```

    ## # A tibble: 5 × 5
    ##   price_category avg_revenue avg_occupancy avg_price     n
    ##   <fct>                <dbl>         <dbl>     <dbl> <int>
    ## 1 Luxury              95940.          92.4     562.    316
    ## 2 Premium             47471.          91.8     293.    332
    ## 3 Mid-range           35316.          93.2     206.    460
    ## 4 Economy             22816.          92.2     135.    461
    ## 5 Budget              11313.          90.9      70.9   371

## 5.5 Neighbourhood Analysis

``` r
# Analyze by neighbourhood
neighborhood_stats <- df |>
  filter(!is.na(neighbourhood_cleansed), !is.na(price_clean)) |>
  group_by(neighbourhood_cleansed) |>
  summarise(
    n_listings = n(),
    mean_price = mean(price_clean, na.rm = TRUE),
    median_price = median(price_clean, na.rm = TRUE),
    mean_rating = mean(review_scores_rating, na.rm = TRUE),
    mean_revenue = mean(revenue_l365d, na.rm = TRUE),
    mean_occupancy = mean(occupancy_rate, na.rm = TRUE),
    .groups = "drop"
  ) |>
  arrange(desc(n_listings)) |>
  head(20)

# Visualize top neighborhoods
p1 <- neighborhood_stats |>
  head(15) |>
  mutate(neighbourhood = factor(neighbourhood_cleansed, 
                               levels = rev(neighbourhood_cleansed))) |>
  ggplot(aes(x = neighbourhood, y = mean_price, fill = n_listings)) +
  geom_bar(stat = "identity", alpha = 0.8) +
  scale_fill_gradient(low = theme_colors$light, high = theme_colors$dark, 
                     name = "Listings") +
  coord_flip() +
  scale_y_continuous(labels = dollar_format()) +
  labs(title = "Average Price by Top 15 Neighbourhoods",
       x = "Neighbourhood",
       y = "Average Price ($)") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        panel.grid.minor = element_blank())

p2 <- neighborhood_stats |>
  head(15) |>
  mutate(neighbourhood = factor(neighbourhood_cleansed,
                               levels = rev(neighbourhood_cleansed))) |>
  ggplot(aes(x = neighbourhood, y = mean_revenue, fill = mean_occupancy)) +
  geom_bar(stat = "identity", alpha = 0.8) +
  scale_fill_gradient(low = theme_colors$light, high = theme_colors$dark,
                     labels = percent_format(), name = "Occupancy") +
  coord_flip() +
  scale_y_continuous(labels = dollar_format()) +
  labs(title = "Average Annual Revenue by Top 15 Neighbourhoods",
       x = "Neighbourhood",
       y = "Average Revenue ($)") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        panel.grid.minor = element_blank())

neighbourhood_plot <- grid.arrange(p1, p2, ncol = 2)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/neighbourhood_analysis-1.png" style="display: block; margin: auto;" />

``` r
print(neighbourhood_plot)
```

    ## TableGrob (1 x 2) "arrange": 2 grobs
    ##   z     cells    name           grob
    ## 1 1 (1-1,1-1) arrange gtable[layout]
    ## 2 2 (1-1,2-2) arrange gtable[layout]

## 5.6 Marginal Revenue per Additional Amenity

``` r
# Basic cleaning & derived columns
df2 <- df |>
  mutate(
    revenue = as.numeric(revenue_l365d),
    price = as.numeric(price_clean),
    occupancy = as.numeric(occupancy_rate),
    avg_review = as.numeric(avg_review_score),      # or review_scores_rating
    beds = as.numeric(beds),
    bedrooms = as.numeric(bedrooms),
    price_per_person = as.numeric(price_per_person),
    revenue_per_bed = as.numeric(revenue_per_bed),
    has_wifi = if_else(has_wifi %in% c(TRUE, "t", "TRUE", 1), 1, 0),
    host_experience_years = as.numeric(host_experience_years)
  ) |>
  # remove impossible / NA revenues and coordinates outside expected range
  filter(!is.na(revenue) & revenue >= 0)

glimpse(df2 |> select(revenue, price, occupancy, beds, bedrooms, avg_review, revenue_per_bed))
```

    ## Rows: 3,506
    ## Columns: 7
    ## $ revenue         <dbl> 0, 6966, 8064, 0, 0, 12928, 24240, 0, 3876, 7644, 3062…
    ## $ price           <dbl> 125, 129, 168, 140, 166, 202, 202, 162, 323, 91, 176, …
    ## $ occupancy       <dbl> 0.00, 0.54, 0.48, 0.00, 0.00, 0.64, 1.00, 0.00, 0.12, …
    ## $ beds            <dbl> 1, 1, 2, 2, 1, 1, 1, 3, 1, 1, 1, 1, 1, 1, 1, 3, 1, 1, …
    ## $ bedrooms        <dbl> 1, 1, 0, 1, 0, 0, 0, 3, 1, 1, 1, 0, 1, 1, 1, 2, 1, 1, …
    ## $ avg_review      <dbl> 4.96, 4.82, 4.81, 4.69, 4.33, 5.00, 4.50, 4.30, 4.97, …
    ## $ revenue_per_bed <dbl> 0, 6966, 4032, 0, 0, 12928, 24240, 0, 3876, 7644, 3062…

``` r
summary_stats <- df2 |>
  summarize(
    n = n(),
    mean_revenue = mean(revenue, na.rm=TRUE),
    median_revenue = median(revenue, na.rm=TRUE),
    sd_revenue = sd(revenue, na.rm=TRUE),
    p5 = quantile(revenue, .05, na.rm=TRUE),
    p25 = quantile(revenue, .25, na.rm=TRUE),
    p75 = quantile(revenue, .75, na.rm=TRUE),
    p95 = quantile(revenue, .95, na.rm=TRUE),
    max_revenue = max(revenue, na.rm=TRUE)
  )
print(summary_stats)
```

    ## # A tibble: 1 × 9
    ##       n mean_revenue median_revenue sd_revenue    p5   p25    p75   p95
    ##   <int>        <dbl>          <dbl>      <dbl> <dbl> <dbl>  <dbl> <dbl>
    ## 1  3506       25596.          10272     66197.     0     0 32396. 90930
    ## # ℹ 1 more variable: max_revenue <dbl>

``` r
r_am_roi <- df |>
  filter(!is.na(revenue_l365d), !is.na(amenity_count),
         revenue_l365d <= 250000, amenity_count <= 40) |>
  group_by(amenity_count) |>
  summarise(
    mean_rev = mean(revenue_l365d, na.rm = TRUE),
    n = n()
  ) |>
  mutate(
    marginal_rev = mean_rev - lag(mean_rev)
  ) |>
  ggplot(aes(x = amenity_count, y = marginal_rev)) +
  geom_col(fill = theme_colors$accent, alpha = 0.8) +
  geom_hline(yintercept = 0, color = theme_colors$dark, linewidth = 0.6) +
  labs(
    title = "Marginal Revenue per Additional Amenity",
    x = "Amenity Count",
    y = "Marginal Revenue Change ($)"
  ) +
  theme_transparent()

r_am_roi
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/eff-1.png" style="display: block; margin: auto;" />

## 5.7 Revenue Surface: Interaction of Price and Occupancy

``` r
df_rev_surface <- df |>
  filter(!is.na(price_clean), !is.na(occupancy_rate),
         !is.na(revenue_l365d), revenue_l365d <= 250000) |>
  mutate(
    price_bin = cut(price_clean, breaks = seq(0, 600, 50), include.lowest = TRUE),
    occ_bin = cut(occupancy_rate, breaks = seq(0, 1, 0.1), include.lowest = TRUE)
  ) |>
  group_by(price_bin, occ_bin) |>
  summarise(mean_rev = mean(revenue_l365d, na.rm = TRUE)) |> 
  ungroup()

r_heat <- df_rev_surface |>
  ggplot(aes(x = price_bin, y = occ_bin, fill = mean_rev)) +
  geom_tile() +
  scale_fill_gradient(low = theme_colors$light, high = theme_colors$dark,
                      labels = dollar_format()) +
  labs(
    title = "Revenue Surface: Interaction of Price and Occupancy",
    x = "Price per Night (Binned)",
    y = "Occupancy Rate (Binned)",
    fill = "Mean Revenue"
  ) +
  theme_transparent() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

r_heat
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/rev_heatmap-1.png" style="display: block; margin: auto;" />

## 5.8 Revenue Elasticity

``` r
# ------- PRICE ELASTICITY -------

df_el <- df |>
  filter(
    !is.na(price_clean),
    !is.na(revenue_l365d),
    is.finite(price_clean),
    is.finite(revenue_l365d),
    revenue_l365d <= 250000,
    revenue_l365d > 0,
    price_clean > 0,
    price_clean <= 5000
  )

# GAM on log-log scale
gam_price <- gam(log(revenue_l365d) ~ s(log(price_clean)), data = df_el)

# Grid over log(price)
lp_seq <- seq(
  from = log(min(df_el$price_clean, na.rm = TRUE)),
  to   = log(max(df_el$price_clean, na.rm = TRUE)),
  length.out = 300
)

grid_price <- data.frame(price_clean = exp(lp_seq))

# Fitted smooth on link scale (log revenue)
f_hat <- predict(gam_price, newdata = grid_price, type = "link")

# Central finite differences: d log(rev) / d log(price)
n <- length(lp_seq)
deriv_lp <- rep(NA_real_, n)
deriv_lp[2:(n - 1)] <- (f_hat[3:n] - f_hat[1:(n - 2)]) / (lp_seq[3:n] - lp_seq[1:(n - 2)])

elasticity_price <- tibble(
  price      = grid_price$price_clean,
  elasticity = deriv_lp
)

r_el_price <- elasticity_price |>
  ggplot(aes(x = price, y = elasticity)) +
  geom_line(color = theme_colors$primary, linewidth = 1.4) +
  geom_hline(yintercept = 0, color = theme_colors$dark) +
  labs(
    title = "Price Elasticity of Annual Revenue",
    subtitle = "Elasticity ≈ d log(Revenue) / d log(Price)",
    x = "Price ($)",
    y = "Elasticity"
  ) +
  theme_transparent()


# ------- OCCUPANCY ELASTICITY -------

df_el_occ <- df |>
  filter(
    !is.na(occupancy_rate),
    !is.na(revenue_l365d),
    revenue_l365d <= 250000,
    revenue_l365d > 0,
    occupancy_rate > 0,
    occupancy_rate < 1
  )

gam_occ <- gam(log(revenue_l365d) ~ s(log(occupancy_rate)), data = df_el_occ)

lo_seq <- seq(
  from = log(min(df_el_occ$occupancy_rate, na.rm = TRUE)),
  to   = log(max(df_el_occ$occupancy_rate, na.rm = TRUE)),
  length.out = 300
)

grid_occ <- data.frame(occupancy_rate = exp(lo_seq))

g_hat <- predict(gam_occ, newdata = grid_occ, type = "link")

n2 <- length(lo_seq)
deriv_lo <- rep(NA_real_, n2)
deriv_lo[2:(n2 - 1)] <- (g_hat[3:n2] - g_hat[1:(n2 - 2)]) / (lo_seq[3:n2] - lo_seq[1:(n2 - 2)])

elasticity_occ <- tibble(
  occ        = grid_occ$occupancy_rate,
  elasticity = deriv_lo
)

r_el_occ <- elasticity_occ |>
  ggplot(aes(x = occ, y = elasticity)) +
  geom_line(color = theme_colors$accent, linewidth = 1.4) +
  geom_hline(yintercept = 0, color = theme_colors$dark) +
  labs(
    title = "Occupancy Elasticity of Annual Revenue",
    subtitle = "Elasticity ≈ d log(Revenue) / d log(Occupancy)",
    x = "Occupancy rate",
    y = "Elasticity"
  ) +
  theme_transparent()

# Show both plots
grid.arrange(r_el_price, r_el_occ, ncol = 2)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/unnamed-chunk-1-1.png" style="display: block; margin: auto;" />

## 5.9 Revenue Misc

``` r
# ---------- Global cleaning & derived columns ----------
df_clean <- df |>
  mutate(
    revenue = as.numeric(revenue_l365d),
    price = as.numeric(price_clean),
    occupancy = as.numeric(occupancy_rate),
    avg_review = coalesce(as.numeric(avg_review_score), as.numeric(review_scores_rating)),
    beds = as.numeric(beds),
    bedrooms = as.numeric(bedrooms),
    amenity_count = as.numeric(amenity_count),
    host_years = as.numeric(host_experience_years),
    is_superhost = as.integer(host_is_superhost_bool),
    listings_owned = as.numeric(calculated_host_listings_count),
    listing_eff = as.numeric(listing_efficiency)
  ) |>
  # remove obviously bad coords / values and keep reasonable ranges
  filter(is.finite(revenue), is.finite(price), is.finite(occupancy)) |>
  filter(revenue >= 0, price > 0, occupancy >= 0, occupancy <= 1) |>
  # cap extreme revenues for stability (keeps original for reporting if needed)
  mutate(revenue_capped = pmin(revenue, 250000))

# A quick summary
df_clean |> summarise(n = n(), mean_rev = mean(revenue), median_rev = median(revenue), p95 = quantile(revenue, .95))
```

    ## # A tibble: 1 × 4
    ##       n mean_rev median_rev   p95
    ##   <int>    <dbl>      <dbl> <dbl>
    ## 1  3489   25706.      10440 91152

``` r
# ---------- Helpers ----------
# backtransform function (for log models)
inv_log10 <- function(x) 10^x - 1
```

``` r
minn_df <- df_clean |>
  filter(minimum_nights <= 30) |>
  group_by(minimum_nights) |>
  summarise(mean_occ = mean(occupancy, na.rm=TRUE), mean_rev = mean(revenue, na.rm=TRUE), n = n())

p_min <- ggplot(minn_df, aes(x = minimum_nights, y = mean_rev)) +
  geom_line(color = theme_colors$primary, size=1.2) +
  geom_point() + scale_y_continuous(labels = dollar_format()) +
  labs(title = "Mean Revenue vs Minimum Nights", x = "Minimum Nights", y = "Mean Revenue ($)") +
  theme_transparent()
p_min
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/rev test-1.png" style="display: block; margin: auto;" />

``` r
df_eff <- df_clean |>
  filter(availability_365 >= 0 & availability_365 <= 365 & price <= 5000) |>
  mutate(avail_nights = 365 - availability_365,
         rev_per_avail = ifelse(avail_nights > 0, revenue / avail_nights, NA))

# Plot efficiency vs price
p_eff <- ggplot(df_eff |> filter(!is.na(rev_per_avail) & rev_per_avail < 2000), aes(x = price, y = rev_per_avail)) +
  geom_point(alpha = 0.25) + geom_smooth(method="loess", se=FALSE, color = theme_colors$accent) +
  labs(title = "Revenue per Available Night vs Price", x = "Price ($)", y = "Revenue / Available Night ($)") +
  theme_transparent()
p_eff
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/rev test-2.png" style="display: block; margin: auto;" />

``` r
# Model with host features
df_host <- df_clean |> filter(revenue > 0 & revenue <= 250000)
host_mod <- lm(log10(revenue) ~ host_years + is_superhost + host_response_rate_num + listings_owned + avg_review, data = df_host)
summary(host_mod)
```

    ## 
    ## Call:
    ## lm(formula = log10(revenue) ~ host_years + is_superhost + host_response_rate_num + 
    ##     listings_owned + avg_review, data = df_host)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1.79739 -0.27545  0.05922  0.31251  1.16032 
    ## 
    ## Coefficients:
    ##                         Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)             2.797585   0.209418  13.359  < 2e-16 ***
    ## host_years             -0.015731   0.002624  -5.994 2.39e-09 ***
    ## is_superhost            0.102594   0.022842   4.491 7.45e-06 ***
    ## host_response_rate_num  0.404813   0.153286   2.641  0.00833 ** 
    ## listings_owned          0.001785   0.000283   6.307 3.45e-10 ***
    ## avg_review              0.244053   0.031196   7.823 8.07e-15 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.4691 on 2113 degrees of freedom
    ##   (189 observations deleted due to missingness)
    ## Multiple R-squared:  0.07178,    Adjusted R-squared:  0.06958 
    ## F-statistic: 32.68 on 5 and 2113 DF,  p-value: < 2.2e-16

``` r
# Simple visual
p_host <- df_host |> filter(host_years <= 20) |>
  ggplot(aes(x = host_years, y = revenue)) +
  geom_point(alpha=0.2) + geom_smooth(method="loess", se=FALSE, color = theme_colors$secondary) +
  scale_y_continuous(labels = dollar_format()) +
  labs(title = "Revenue vs Host Experience (years)") +
  theme_transparent()
p_host
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/rev test-3.png" style="display: block; margin: auto;" />

``` r
nb_summary <- df_clean |>
  group_by(neighbourhood_cleansed) |>
  summarise(nb_median = median(revenue, na.rm=TRUE), n = n())

bench <- df_clean |> left_join(nb_summary, by = "neighbourhood_cleansed") |>
  mutate(rel_to_nb = revenue / nb_median) |>
  arrange(neighbourhood_cleansed, desc(rel_to_nb)) 

# Underperformers: rel_to_nb < 0.6 in neighborhoods with n >= 10
underperf <- bench |> filter(n >= 10, rel_to_nb < 0.6)
underperf |> select(id, neighbourhood_cleansed, revenue, nb_median, rel_to_nb) |> head(20)
```

    ## # A tibble: 20 × 5
    ##         id neighbourhood_cleansed revenue nb_median rel_to_nb
    ##      <dbl> <chr>                    <dbl>     <dbl>     <dbl>
    ##  1 9.72e17 Allston                   3944      8577     0.460
    ##  2 1.38e18 Allston                   3888      8577     0.453
    ##  3 1.49e18 Allston                   3168      8577     0.369
    ##  4 1.49e18 Allston                   2862      8577     0.334
    ##  5 1.34e18 Allston                   2448      8577     0.285
    ##  6 1.50e18 Allston                   2178      8577     0.254
    ##  7 1.50e18 Allston                   2004      8577     0.234
    ##  8 1.50e18 Allston                   1650      8577     0.192
    ##  9 1.51e18 Allston                   1080      8577     0.126
    ## 10 1.32e 7 Allston                      0      8577     0    
    ## 11 1.56e 7 Allston                      0      8577     0    
    ## 12 1.60e 7 Allston                      0      8577     0    
    ## 13 1.82e 7 Allston                      0      8577     0    
    ## 14 2.10e 7 Allston                      0      8577     0    
    ## 15 2.74e 7 Allston                      0      8577     0    
    ## 16 4.54e 7 Allston                      0      8577     0    
    ## 17 5.06e 7 Allston                      0      8577     0    
    ## 18 5.10e 7 Allston                      0      8577     0    
    ## 19 5.16e 7 Allston                      0      8577     0    
    ## 20 5.21e 7 Allston                      0      8577     0

``` r
sup_demand <- df_clean |>
  group_by(neighbourhood_cleansed) |>
  summarise(median_price = median(price, na.rm=TRUE),
            median_occ = median(occupancy, na.rm=TRUE),
            n = n()) |>
  mutate(supply_demand_index = median_price * median_occ) |>
  arrange(desc(supply_demand_index))

head(sup_demand, 15)
```

    ## # A tibble: 15 × 5
    ##    neighbourhood_cleansed median_price median_occ     n supply_demand_index
    ##    <chr>                         <dbl>      <dbl> <int>               <dbl>
    ##  1 Downtown                       332        1      278               332  
    ##  2 North End                      262.       1       92               262. 
    ##  3 Fenway                         254        0.72   197               183. 
    ##  4 Back Bay                       289        0.56   302               162. 
    ##  5 Jamaica Plain                  207        0.78   180               161. 
    ##  6 East Boston                    158.       1      166               158. 
    ##  7 South End                      231        0.66   209               152. 
    ##  8 Mission Hill                   140        1       69               140  
    ##  9 Charlestown                    244        0.54    62               132. 
    ## 10 Brighton                       167        0.78   234               130. 
    ## 11 Beacon Hill                    140        0.9    197               126  
    ## 12 South Boston                   226        0.54   143               122. 
    ## 13 Roxbury                        158        0.72   249               114. 
    ## 14 Allston                        142.       0.58   120                82.1
    ## 15 Roslindale                     136        0.6     83                81.6

- If revenue falls as minimum_nights increases, recommend lowering
  minimum nights to increase bookings.

- If revenue increases with minimum nights (and occupancy doesn’t drop
  too much), longer minimums may be beneficial for host stability.

- Low rev_per_avail suggests either low price, low occupancy, or both —
  investigate listings with good location but low efficiency.

- Recommend: improve photos, reviews, adjust price, add amenity
  investments.

# 6. Clustering Analysis

``` r
# Prepare data for clustering
clustering_data <- df |>
  filter(
    !is.na(price_clean),
    !is.na(latitude),
    !is.na(longitude),
    !is.na(accommodates),
    !is.na(bedrooms),
    !is.na(review_scores_rating),
    !is.na(revenue_l365d)
  ) |>
  mutate(
    # Normalize key features for clustering
    price_scaled = scale(price_clean)[,1],
    revenue_scaled = scale(revenue_l365d)[,1],
    rating_scaled = scale(review_scores_rating)[,1],
    accommodates_scaled = scale(accommodates)[,1],
    latitude_scaled = scale(latitude)[,1],
    longitude_scaled = scale(longitude)[,1],
    occupancy_scaled = scale(occupancy_rate)[,1],
    amenity_scaled = scale(premium_amenity_score)[,1]
  ) |>
  select(id, price_clean, revenue_l365d, review_scores_rating, accommodates,
         bedrooms, latitude, longitude, occupancy_rate, premium_amenity_score,
         room_type, neighbourhood_cleansed,
         price_scaled, revenue_scaled, rating_scaled, accommodates_scaled,
         latitude_scaled, longitude_scaled, occupancy_scaled, amenity_scaled)

cat("Prepared", nrow(clustering_data), "observations for clustering\n")
```

    ## Prepared 2765 observations for clustering

``` r
cat("Features selected:", 
    paste(c("Price", "Revenue", "Rating", "Capacity", "Location", 
            "Occupancy", "Amenities"), collapse = ", "), "\n")
```

    ## Features selected: Price, Revenue, Rating, Capacity, Location, Occupancy, Amenities

``` r
# Select features for clustering
cluster_features <- clustering_data |>
  select(price_scaled, revenue_scaled, rating_scaled, 
         accommodates_scaled, occupancy_scaled, amenity_scaled)

# Method 1: Elbow method
set.seed(123)
wss <- map_dbl(1:10, function(k) {
  kmeans(cluster_features, centers = k, nstart = 25, iter.max = 50)$tot.withinss
})

elbow_df <- data.frame(k = 1:10, wss = wss)

p1 <- ggplot(elbow_df, aes(x = k, y = wss)) +
  geom_line(color = theme_colors$primary, linewidth = 1.2) +
  geom_point(color = theme_colors$accent, size = 3) +
  scale_x_continuous(breaks = 1:10) +
  labs(title = "Elbow Method for Optimal Clusters",
       x = "Number of Clusters (k)",
       y = "Within Sum of Squares",
       subtitle = "Look for the 'elbow' in the curve") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        panel.grid.minor = element_blank())

# Method 2: Silhouette method
silhouette_scores <- map_dbl(2:10, function(k) {
  km <- kmeans(cluster_features, centers = k, nstart = 25, iter.max = 50)
  ss <- silhouette(km$cluster, dist(cluster_features))
  mean(ss[, 3])
})

sil_df <- data.frame(k = 2:10, silhouette = silhouette_scores)

p2 <- ggplot(sil_df, aes(x = k, y = silhouette)) +
  geom_line(color = theme_colors$secondary, linewidth = 1.2) +
  geom_point(color = theme_colors$accent, size = 3) +
  scale_x_continuous(breaks = 2:10) +
  labs(title = "Silhouette Method for Optimal Clusters",
       x = "Number of Clusters (k)",
       y = "Average Silhouette Width",
       subtitle = "Higher values indicate better clustering") +
  theme_transparent() +
  theme(plot.title = element_text(face = "bold", size = 14, color = theme_colors$text),
        panel.grid.minor = element_blank())

cluster_plot <-grid.arrange(p1, p2, ncol = 2)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/optimal_clusters-1.png" style="display: block; margin: auto;" />

``` r
print(cluster_plot)
```

    ## TableGrob (1 x 2) "arrange": 2 grobs
    ##   z     cells    name           grob
    ## 1 1 (1-1,1-1) arrange gtable[layout]
    ## 2 2 (1-1,2-2) arrange gtable[layout]

``` r
# Determine optimal k
optimal_k <- sil_df$k[which.max(sil_df$silhouette)]
cat("\n=== OPTIMAL NUMBER OF CLUSTERS ===\n")
```

    ## 
    ## === OPTIMAL NUMBER OF CLUSTERS ===

``` r
cat("Based on silhouette method: k =", optimal_k, "\n")
```

    ## Based on silhouette method: k = 5

``` r
cat("Average silhouette width:", round(max(sil_df$silhouette), 3), "\n")
```

    ## Average silhouette width: 0.315

## K-Means Clustering

``` r
# Perform K-means clustering with optimal k
set.seed(123)
kmeans_result <- kmeans(cluster_features, centers = optimal_k, nstart = 50, iter.max = 100)

# Add cluster assignments
clustering_data$cluster <- as.factor(kmeans_result$cluster)

cat("=== CLUSTER ASSIGNMENTS ===\n")
```

    ## === CLUSTER ASSIGNMENTS ===

``` r
print(table(clustering_data$cluster))
```

    ## 
    ##    1    2    3    4    5 
    ## 1495  468   14  657  131

``` r
# Prepare data for geographic cluster visualization
df_clustered_geo <- df |>
  left_join(
    clustering_data |> select(id, cluster),
    by = "id"
  ) |>
  filter(!is.na(cluster), !is.na(latitude), !is.na(longitude),
         latitude > 42.2 & latitude < 42.4,
         longitude > -71.2 & longitude < -70.9)

# Sample for performance
df_clustered_geo_sample <- df_clustered_geo |>
  sample_n(min(2000, nrow(df_clustered_geo)))

# Create distinct cluster color palette - using high contrast, easily distinguishable colors
# Define colors for each cluster (1-5)
cluster_colors_list <- c("#3498db", "#2ecc71", "#f39c12", "#e74c3c", "#9b59b6")
names(cluster_colors_list) <- as.character(1:5)

# Ensure cluster is character for consistent color mapping
df_clustered_geo_sample$cluster_char <- as.character(df_clustered_geo_sample$cluster)

# Create cluster color palette with all possible cluster values
all_clusters <- sort(unique(df_clustered_geo_sample$cluster_char))
cluster_pal <- colorFactor(
  palette = cluster_colors_list[all_clusters],
  domain = all_clusters
)

# Visualize clusters on geographic map - WITHOUT clustering to show individual colors
map_clusters_geo <- leaflet() |>
  addProviderTiles(providers$CartoDB.Positron, options = providerTileOptions(opacity = 0.9)) |>
  addPolygons(
    data = neighbourhoods_geo,
    fillColor = "transparent",
    fillOpacity = 0,
    color = theme_colors$text,
    weight = 1.5,
    opacity = 0.6,
    label = ~neighbourhood,
    labelOptions = labelOptions(noHide = FALSE, textOnly = FALSE)
  ) |>
  addCircleMarkers(
    data = df_clustered_geo_sample,
    lng = ~longitude,
    lat = ~latitude,
    radius = 5,
    color = "white",
    fillColor = ~cluster_pal(cluster_char),
    fillOpacity = 0.8,
    stroke = TRUE,
    weight = 1.5,
    popup = ~paste(
      "<b>Cluster:</b>", cluster, "<br>",
      "<b>Price:</b> $", round(price_clean, 0), "<br>",
      "<b>Revenue:</b> $", round(revenue_l365d, 0), "<br>",
      "<b>Rating:</b>", round(review_scores_rating, 1), "/5.0", "<br>",
      "<b>Neighbourhood:</b>", neighbourhood_cleansed, "<br>",
      "<b>Room Type:</b>", room_type, "<br>",
      "<b>Occupancy:</b>", round(occupancy_rate * 100, 1), "%"
    )
  ) |>
  leaflet::addLegend(
    "bottomright",
    pal = cluster_pal,
    values = all_clusters,
    title = paste("Cluster (k =", optimal_k, ")"),
    opacity = 0.9
  )

print(map_clusters_geo)
```

## Cluster Characterization

``` r
# Analyze cluster characteristics using ALL relevant features
cluster_profiles <- clustering_data |>
  group_by(cluster) |>
  summarise(
    n   = n(),
    pct = round(n / nrow(clustering_data) * 100, 1),

    # --- RAW FEATURES ---
    avg_price        = mean(price_clean, na.rm = TRUE),
    median_price     = median(price_clean, na.rm = TRUE),
    avg_revenue      = mean(revenue_l365d, na.rm = TRUE),
    avg_rating       = mean(review_scores_rating, na.rm = TRUE),
    avg_occupancy    = mean(occupancy_rate, na.rm = TRUE) * 100,   # as %
    avg_accommodates = mean(accommodates, na.rm = TRUE),
    avg_bedrooms     = mean(bedrooms, na.rm = TRUE),
    avg_amenities    = mean(premium_amenity_score, na.rm = TRUE),
    pct_entire_home  = mean(room_type == "Entire home/apt", na.rm = TRUE) * 100,

    # --- RAW LOCATION ---
    avg_latitude     = mean(latitude, na.rm = TRUE),
    avg_longitude    = mean(longitude, na.rm = TRUE),

    # --- SCALED FEATURES (CENTROIDS IN FEATURE SPACE) ---
    price_scaled_mean        = mean(price_scaled, na.rm = TRUE),
    revenue_scaled_mean      = mean(revenue_scaled, na.rm = TRUE),
    rating_scaled_mean       = mean(rating_scaled, na.rm = TRUE),
    accommodates_scaled_mean = mean(accommodates_scaled, na.rm = TRUE),
    latitude_scaled_mean     = mean(latitude_scaled, na.rm = TRUE),
    longitude_scaled_mean    = mean(longitude_scaled, na.rm = TRUE),
    occupancy_scaled_mean    = mean(occupancy_scaled, na.rm = TRUE),
    amenity_scaled_mean      = mean(amenity_scaled, na.rm = TRUE),

    # --- DOMINANT CATEGORICAL FEATURES ---
    main_room_type      = names(which.max(table(room_type))),

    .groups = "drop"
  ) |>
  arrange(desc(avg_revenue))

cat("=== CLUSTER PROFILES ===\n")
```

    ## === CLUSTER PROFILES ===

``` r
print(
  cluster_profiles |>
    mutate(
      avg_price     = round(avg_price, 2),
      median_price  = round(median_price, 2),
      avg_revenue   = round(avg_revenue, 0),
      avg_rating    = round(avg_rating, 2),
      avg_occupancy = round(avg_occupancy, 1),
      avg_accommodates = round(avg_accommodates, 1),
      avg_bedrooms  = round(avg_bedrooms, 1),
      avg_amenities = round(avg_amenities, 1),
      pct_entire_home = round(pct_entire_home, 1)
    )
)
```

    ## # A tibble: 5 × 23
    ##   cluster     n   pct avg_price median_price avg_revenue avg_rating
    ##   <fct>   <int> <dbl>     <dbl>        <dbl>       <dbl>      <dbl>
    ## 1 3          14   0.5    47857.        50000      715714       4.72
    ## 2 2         468  16.9      458.          381       66297       4.85
    ## 3 1        1495  54.1      182.          163       30951       4.79
    ## 4 5         131   4.7      214.          190        6012       3.41
    ## 5 4         657  23.8      232.          187        2409       4.8 
    ## # ℹ 16 more variables: avg_occupancy <dbl>, avg_accommodates <dbl>,
    ## #   avg_bedrooms <dbl>, avg_amenities <dbl>, pct_entire_home <dbl>,
    ## #   avg_latitude <dbl>, avg_longitude <dbl>, price_scaled_mean <dbl>,
    ## #   revenue_scaled_mean <dbl>, rating_scaled_mean <dbl>,
    ## #   accommodates_scaled_mean <dbl>, latitude_scaled_mean <dbl>,
    ## #   longitude_scaled_mean <dbl>, occupancy_scaled_mean <dbl>,
    ## #   amenity_scaled_mean <dbl>, main_room_type <chr>

``` r
# Use the same cluster colors as the maps for consistency
cluster_colors_plot <- c(
  "1" = "#3498db",      # Bright Blue
  "2" = "#2ecc71",      # Green
  "3" = "#f39c12",      # Orange
  "4" = "#e74c3c",      # Red
  "5" = "#9b59b6"       # Purple
)

# Visualize key numeric characteristics (for the report)
p1 <- cluster_profiles |>
  select(cluster, avg_price, avg_revenue, avg_rating, avg_occupancy) |>
  pivot_longer(cols = c(avg_price, avg_revenue, avg_rating, avg_occupancy),
               names_to = "metric", values_to = "value") |>
  mutate(
    metric = metric |>
      str_remove_all("avg_") |>
      str_replace_all("_", " ") |>
      str_to_title(),
    cluster = as.character(cluster)
  ) |>
  ggplot(aes(x = cluster, y = value, fill = cluster)) +
  geom_bar(stat = "identity", alpha = 0.8, position = "dodge") +
  facet_wrap(~ metric, scales = "free_y", ncol = 2) +
  scale_fill_manual(values = cluster_colors_plot) +
  labs(title = "Cluster Characteristics Comparison",
       x = "Cluster",
       y = "Value",
       fill = "Cluster") +
  theme_transparent() +
  theme(
    plot.title   = element_text(face = "bold", size = 14, color = theme_colors$text),
    legend.position = "none",
    strip.text   = element_text(face = "bold")
  )

print(p1)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/cluster_characterization-1.png" style="display: block; margin: auto;" />

``` r
# -------------------------
# Name the clusters based on ALL characteristics
# -------------------------

cluster_names <- cluster_profiles |>
  mutate(
    cluster = as.integer(cluster),

    # PRICE: uses price_scaled_mean (price_scaled) and avg_price
    price_segment = case_when(
      price_scaled_mean >=  1.0 ~ "Luxury",
      price_scaled_mean >=  0.5 ~ "Upscale",
      price_scaled_mean <= -1.0 ~ "Budget",
      price_scaled_mean <= -0.5 ~ "Economy",
      TRUE                      ~ "Mid-Priced"
    ),

    # REVENUE: uses revenue_scaled_mean
    revenue_segment = case_when(
      revenue_scaled_mean >=  0.8 ~ "High-Revenue",
      revenue_scaled_mean <= -0.8 ~ "Low-Revenue",
      TRUE                        ~ "Mid-Revenue"
    ),

    # RATING: uses rating_scaled_mean (review_scores_rating / rating_scaled)
    rating_segment = case_when(
      rating_scaled_mean >=  0.7 ~ "Top-Rated",
      rating_scaled_mean <= -0.7 ~ "Lower-Rated",
      TRUE                       ~ "Mid-Rated"
    ),

    # OCCUPANCY: uses occupancy_scaled_mean (occupancy_rate / occupancy_scaled)
    occupancy_segment = case_when(
      occupancy_scaled_mean >=  0.8 ~ "High-Occupancy",
      occupancy_scaled_mean <= -0.8 ~ "Low-Occupancy",
      TRUE                          ~ "Mid-Occupancy"
    ),

    # AMENITIES: uses amenity_scaled_mean (premium_amenity_score / amenity_scaled)
    amenity_segment = case_when(
      amenity_scaled_mean >=  0.6 ~ "Amenity-Rich",
      amenity_scaled_mean <= -0.6 ~ "Amenity-Light",
      TRUE                        ~ "Standard Amenities"
    ),

    # SIZE: uses accommodates, bedrooms, and accommodates_scaled
    size_segment = case_when(
      avg_accommodates >= 6 | avg_bedrooms >= 3 ~ "Large-Group",
      avg_accommodates >= 3 | avg_bedrooms >= 2 ~ "Family-Sized",
      TRUE                                      ~ "Compact"
    ),

    # FINAL HUMAN-READABLE LABEL
    cluster_label = glue(
      "{price_segment} {revenue_segment}, {rating_segment}, ",
      "{occupancy_segment}, {amenity_segment} {size_segment} ",
      "{main_room_type}"
    )
  ) |>
  # If two clusters somehow get exactly the same label, disambiguate by cluster id
  group_by(cluster_label) |>
  mutate(
    cluster_label = if_else(
      n() > 1,
      glue("{cluster_label} (Cluster {cluster})"),
      cluster_label
    )
  ) |>
  ungroup() |>
  # Round some things for nicer printing
  mutate(
    avg_price        = round(avg_price, 2),
    median_price     = round(median_price, 2),
    avg_revenue      = round(avg_revenue, 0),
    avg_rating       = round(avg_rating, 2),
    avg_occupancy    = round(avg_occupancy, 1),
    avg_accommodates = round(avg_accommodates, 1),
    avg_bedrooms     = round(avg_bedrooms, 1),
    avg_amenities    = round(avg_amenities, 1),
    pct_entire_home  = round(pct_entire_home, 1)
  ) |>
  select(
    cluster,
    cluster_label,
    n, pct,
    avg_price, median_price, avg_revenue, avg_rating,
    avg_occupancy, avg_accommodates, avg_bedrooms, avg_amenities,
    pct_entire_home,
    main_room_type
  )

cat("\n=== CLUSTER LABELS ===\n")
```

    ## 
    ## === CLUSTER LABELS ===

``` r
print(cluster_names |> select(cluster, cluster_label, avg_price, avg_revenue))
```

    ## # A tibble: 5 × 4
    ##   cluster cluster_label                                    avg_price avg_revenue
    ##     <int> <glue>                                               <dbl>       <dbl>
    ## 1       3 Luxury High-Revenue, Mid-Rated, Low-Occupancy, …    47857.      715714
    ## 2       2 Mid-Priced Mid-Revenue, Mid-Rated, Mid-Occupanc…      458.       66297
    ## 3       1 Mid-Priced Mid-Revenue, Mid-Rated, Mid-Occupanc…      182.       30951
    ## 4       5 Mid-Priced Mid-Revenue, Lower-Rated, Low-Occupa…      214.        6012
    ## 5       4 Mid-Priced Mid-Revenue, Mid-Rated, Low-Occupanc…      232.        2409

| Original | Cleaner Name |
|----|----|
| **Luxury High-Revenue Large-Group Hotel** | **Luxury Hotel Cluster** |
| **Mid-Priced Amenity-Rich Large-Group Entire Homes** | **Amenity-Rich Large Homes** |
| **Mid-Priced Compact Entire Homes** | **Standard Mid-Tier Homes** |
| **Lower-Rated Low-Occupancy Compact Homes** | **Underperforming Units** |
| **Mid-Rated Low-Occupancy Compact Homes** | **Low-Demand Homes** |

A small number of listings (n=14) have extremely high nightly prices
(\$40–50k) and revenues exceeding \$300k–\$2M annually. These represent
luxury hotel penthouses and group suites. Because of their magnitude,
k-means correctly isolates them into a separate cluster. We retain them
to preserve meaningful segmentation, but acknowledge their outlier
nature.

``` r
cluster_names <- cluster_names |>
  mutate(
    cluster_name_short = case_when(
      cluster == 3 ~ "Luxury Hotel Cluster",
      cluster == 2 ~ "Amenity-Rich Large Homes",
      cluster == 1 ~ "Standard Mid-Tier Homes",
      cluster == 5 ~ "Underperforming Units",
      cluster == 4 ~ "Low-Demand Homes",
      TRUE         ~ as.character(cluster_label)
    )
  )
cluster_names
```

    ## # A tibble: 5 × 15
    ##   cluster cluster_label               n   pct avg_price median_price avg_revenue
    ##     <int> <glue>                  <int> <dbl>     <dbl>        <dbl>       <dbl>
    ## 1       3 Luxury High-Revenue, M…    14   0.5    47857.        50000      715714
    ## 2       2 Mid-Priced Mid-Revenue…   468  16.9      458.          381       66297
    ## 3       1 Mid-Priced Mid-Revenue…  1495  54.1      182.          163       30951
    ## 4       5 Mid-Priced Mid-Revenue…   131   4.7      214.          190        6012
    ## 5       4 Mid-Priced Mid-Revenue…   657  23.8      232.          187        2409
    ## # ℹ 8 more variables: avg_rating <dbl>, avg_occupancy <dbl>,
    ## #   avg_accommodates <dbl>, avg_bedrooms <dbl>, avg_amenities <dbl>,
    ## #   pct_entire_home <dbl>, main_room_type <chr>, cluster_name_short <chr>

## Geographic Distribution of Clusters

``` r
# Attach cluster ids to full df
df_clustered <- df |>
  left_join(
    clustering_data |> select(id, cluster),
    by = "id"
  ) |>
  filter(!is.na(cluster))

# --- Top neighbourhoods by cluster ---
cluster_neighborhood <- df_clustered |>
  filter(!is.na(cluster), !is.na(neighbourhood_cleansed)) |>
  count(cluster, neighbourhood_cleansed) |>
  # make cluster numeric so it joins cleanly with cluster_names
  mutate(cluster = as.integer(as.character(cluster))) |>
  left_join(
    cluster_names |> select(cluster, cluster_name_short),
    by = "cluster"
  ) |>
  group_by(neighbourhood_cleansed) |>
  mutate(pct = n / sum(n) * 100) |>
  ungroup() |>
  group_by(cluster, cluster_name_short) |>
  slice_max(pct, n = 5, with_ties = FALSE) |>
  ungroup()

cat("\n=== TOP NEIGHBOURHOODS BY CLUSTER (share of listings in each neighbourhood) ===\n")
```

    ## 
    ## === TOP NEIGHBOURHOODS BY CLUSTER (share of listings in each neighbourhood) ===

``` r
print(cluster_neighborhood)
```

    ## # A tibble: 23 × 5
    ##    cluster neighbourhood_cleansed      n cluster_name_short         pct
    ##      <int> <chr>                   <int> <chr>                    <dbl>
    ##  1       1 Beacon Hill               122 Standard Mid-Tier Homes   77.7
    ##  2       1 Mission Hill               41 Standard Mid-Tier Homes   77.4
    ##  3       1 Fenway                    102 Standard Mid-Tier Homes   67.1
    ##  4       1 South End                 109 Standard Mid-Tier Homes   64.9
    ##  5       1 Back Bay                  141 Standard Mid-Tier Homes   61.8
    ##  6       2 North End                  23 Amenity-Rich Large Homes  30.3
    ##  7       2 Jamaica Plain              40 Amenity-Rich Large Homes  26.3
    ##  8       2 Dorchester                 98 Amenity-Rich Large Homes  25.7
    ##  9       2 Hyde Park                  14 Amenity-Rich Large Homes  24.6
    ## 10       2 South Boston Waterfront     4 Amenity-Rich Large Homes  23.5
    ## # ℹ 13 more rows

``` r
# --- Plot: top neighbourhoods for each cluster ---
cluster_neighborhood_plot <- cluster_neighborhood |>
  mutate(
    neighbourhood_cleansed =
      forcats::fct_reorder(neighbourhood_cleansed, pct)
  ) |>
  ggplot(aes(x = neighbourhood_cleansed, y = pct, fill = cluster_name_short)) +
  geom_col(alpha = 0.9, show.legend = FALSE) +
  coord_flip() +
  facet_wrap(
    ~ cluster_name_short,
    scales = "free_y",
    ncol = 3,
    labeller = label_wrap_gen(width = 20)   # <-- wrap long cluster titles
  ) +
  labs(
    title = "Top Neighbourhoods for Each Cluster",
    x = "Neighbourhood",
    y = "Share of Listings in Neighbourhood (%)"
  ) +
  theme_transparent() +
  theme(
    plot.title = element_text(
      face = "bold", size = 16, color = theme_colors$text
    ),

    # --- fix truncated facet titles ---
    strip.text = element_text(
      face = "bold",
      size = 12,
      margin = margin(t = 10, r = 5, b = 10, l = 5)
    ),

    # more padding around the whole plot (prevents clipping)
    plot.margin = margin(20, 20, 20, 20)
  )

print(cluster_neighborhood_plot)
```

<img src="Detailed_Listings_Analysis_files/figure-gfm/cluster_geography-1.png" style="display: block; margin: auto;" />

------------------------------------------------------------------------

# References and Data Sources

- **Dataset**: Boston Airbnb Detailed Listings (`listings_detailed.csv`)
- **Analysis Date**: 2025-12-10
- **Tools**: R (tidyverse, ggplot2, leaflet, cluster, forecast)

------------------------------------------------------------------------

*Analysis conducted by Fall Foliage Team*
