This project was an assigned graduate capstone project at the culmination of my Master of Science in Business Analytics program at UNH.

## Executive Summary

In part one of our project, we set out to build a model that predicted high booking. This process for us was somewhat tedious, we tried several different types of models that did not perform to the standards that we had hoped. Throughout the course of this project, we tried linear models, logistic models, xgboost models, catboost models and finally lightGBM models. Our original goal was to get a model testing in the public leaderboard on Kaggle with above 0.9 accuracy, however we ultimately did not achieve this goal. Of the 26 models that we attempted, the one that achieved the highest score in Kaggle was a lightGBM model. While we came up a bit short of our intended goal, we were able to achieve the third highest final score of the teams we were competing against, with a score of 0.83121. We also were able to introduce variables around the text data and these performed quite well in the models. Our final model used the following variables:

```{r}
variable_names <- c("host_response_time", "host_response_rate", "host_acceptance_rate", "host_is_superhost",
                    "weekly_price", "accommodates", "minimum_nights", "review_scores_rating", "review_scores_accuracy",
                    "review_scores_cleanliness", "reviews_per_month", "host_listings_count", "host_identity_verified",
                    "security_deposit", "cleaning_fee", "cancellation_policy", "uber_transit", "bars_neighborhood",
                    "house_rules_length")
variables_df <- data.frame(Variable = variable_names)
kable(variables_df, caption = "Variables used in final LightGBM Model")

```


In part two of our project, we were assigned the market of Hawaii. Hawaii is a beautiful state with a unique and potentially lucrative market for Airbnbs. Hawaii is made up of over 100 islands, the largest of which are the Big Island (or the Island of Hawaii), Kauai, Oahu, Molokai, and Maui. Each island offers something unique to visitors. Maui is the most versatile island with hiking trails, waterfalls, beaches, helicopter trips, and resorts. Because of this, it is highly recommended for families. Oahu is the most residential island as it has the capital and vacationers tend to go there for the amazing surfing. Kauai is known for its scenery and beautiful views and is the most remote of the four biggest and most popular islands. The Big Island's volcano is a draw for vacationers as well as its historical landmarks. Molokai is the only island of these five without a major airport. It is much more secluded and is perfect for the vacationer that is looking to get away from the crowds.

In our analysis, we focused on finding what made each island unique in terms of what features were associated with a high booking listing. On Maui, we found that high booking resorts mentioned "kaanapali" (a beach on Maui) in the listing name and "resort" in the listing summary. On the Big Island, we found that high booking apartments had a lower accomodation number, a lower price, a lower maximum night policy, higher reviews per month, and a shorter name for the listing. On Oahu, high booking private rooms in houses had a lower minimum night policy, a lower cleaning fee, and less house rules regarding quiet hours. Apartments on Kauai that have a high booking rate tend to have lower minimum night requirements, less house rules, and a shorter name for the listing. This directly leads us to make recommendations to the investor, depending on what property type and what island they decide to invest in.

Instead of focusing on finding an overall model for all of the islands in Hawaii, we recognized that each island is unique and will have a unique market. Therefore, finding one model for all islands and property types would not be helpful. We narrowed our focus to specific property types on each island in order to provide better and more accurate findings to the investor.

## Main Focus and Questions

While analyzing the Airbnb listings in the Hawaii market, it was important to keep in mind the differences of each of the islands. Our main purpose was to discover the types of listings on each island and analyze what unique features about the island might affect the high booking rate for the major types of listings. We also wanted to discover what features are unique to superhosts, if any, and if being a superhost increases the value of the listing, when it comes to a high booking rate. In summary, our questions were:

-   What features on each island lead to a high booking listing on that island?
-   How do property types differ from island to island?
-   What features do superhosts have?
-   Does being a superhost lead to a high booking listing?

## Methodology

### Data Preparation Process

The first step is to clean the data. There are 19,613 observations in the Hawaii dataset and the high booking rate is 10.5% overall, which makes this an unbalanced dataset and is important to keep in mind during our analysis. The first step to cleaning was to convert all percentages and monetary amounts to numeric datatypes by removing the non numeric characters. Then we filled in any null values in the weekly price and monthly price columns by multiplying the price column by 7 and 30, respectively. We also did this in reverse for any nulls in the price column (divide weekly price by 7 or divide monthly price by 30).

We then cleaned the island, the market field in the dataset. Many observations were missing the island but this was filled in using the neighborhood. Luckily, all observations had a neighborhood so there are no missing values for island. We also cleaned the property type field by combining similar names like "Entire condominium (condo)" and "Entire condominium" into "Entire condo". After this, we replaced null values in numeric columns with either the mean or the median, depending on the distribution. We removed any fields that had over 60% of the data missing from that field and any extra fields that we didn't find useful for our analysis, like the url columns.

After completing text analysis, we also added five binary fields to indicate certain words in the name and description of the listing. Resort in the name, maui in the name, resort in the description, lanai in description, and quiet in the house rules were all found to be more present in either high booking or not high booking listings than the other.

### Exploratory Data Analysis

Our exploratory data analysis of the Hawaii accommodation listings overall revealed several key insights. We first analyzed the listings overall before diving deeper into each island. Our first finding found that the island chosen significantly influences booking rates. Maui has the highest high booking rate at just over 12% of listings and Molokai has the lowest at 5% of listings.

```{r, echo=FALSE}
#High booking by island
islands = data
islands %>% group_by(market) %>% summarise_at(vars(high_booking),
                                              list(avg_high_booking = mean)) %>% 
  ggplot(aes(x = market, y = avg_high_booking)) +
  geom_bar(stat='identity') +
  labs(y = "High Booking Rate", x = "Island") +
  ggtitle("High Booking Rate by Island")
```
![image](https://github.com/user-attachments/assets/8ad6780a-58d5-43a2-b415-0677ca57e820)

Then, we did analysis of Airbnb prices.

The distribution of Airbnb prices across different room types: Entire home/apt, Hotel room, Private room, and Shared room. The key observations are:

1. **Entire Home/Apt**:
   - This category shows the widest range of prices, with several outliers extending up to approximately $25,000.
   - The majority of prices are concentrated below $5,000, with a dense clustering of data points at the lower end.

2. **Hotel Room**:
   - Prices for hotel rooms are more centralized, with fewer extreme outliers compared to entire homes/apartments.
   - The price range is relatively narrow, indicating a more consistent pricing structure.

3. **Private Room**:
   - Similar to entire homes/apartments, private rooms exhibit a significant range in prices, but with fewer extreme outliers.
   - Most prices are clustered at the lower end, under $1,000.

4. **Shared Room**:
   - This category has the lowest prices overall, with a very narrow range and minimal outliers.
   - The data indicates a high level of affordability and consistency in pricing.

The correlation between the number of bedrooms and the price of Airbnb listings. Key insights include:

1. **Overall Trend**:
   - There is a general upward trend where properties with more bedrooms tend to have higher prices. However, the correlation is not perfectly linear.
   
2. **Outliers**:
   - Several listings with 0, 2, and 3 bedrooms have prices exceeding $20,000, indicating that factors other than the number of bedrooms significantly influence pricing for some properties.
   
3. **Price Clusters**:
   - Most properties, regardless of the number of bedrooms, have prices clustered below $5,000. This suggests that while the number of bedrooms does impact price, many listings fall within a more affordable range.

4. **Bedroom Count and Price Distribution**:
   - Listings with 2 to 4 bedrooms show a diverse price distribution, reflecting a variety of factors at play, such as location, amenities, and property type.
   - Properties with 6 or more bedrooms are less common and exhibit a wider range of prices, indicating a niche market.

These visualizations provide a comprehensive overview of the Airbnb pricing landscape. The first graph highlights the variability in pricing across different room types, with entire homes/apartments and private rooms showing the most significant price ranges. The second graph shows the inner quartile of the median prices by room type, showing where the majority of rooms in each room type were priced.

```{r, echo=FALSE, message=FALSE}
hawaii_data <- read_csv("/Users/dylanburbank/Downloads/hawaii_clean_data.csv")

ggplot(hawaii_data, aes(x = room_type, y = price)) +
  geom_boxplot()
iqr <- IQR(hawaii_data$price, na.rm = TRUE)
median_price <- median(hawaii_data$price, na.rm = TRUE)
lower_bound <- median_price - 1.5 * iqr
upper_bound <- median_price + 1.5 * iqr
ggplot(hawaii_data, aes(x = room_type, y = price)) +
  geom_boxplot() +
  coord_cartesian(ylim = c(lower_bound, upper_bound)) +
  labs(title = "Room Type vs Price (Zoomed to Middle Quartile)",
       x = "Room Type",
       y = "Price")

```
![image](https://github.com/user-attachments/assets/eba766e6-ff44-4d9d-aa81-9167c664102c)
![image](https://github.com/user-attachments/assets/60f2cbb6-74f8-4ed5-9d25-37bb84cda26a)


Superhosts distinguish themselves through superior responsiveness, and generally offer more amenities. These factors may contribute to their higher booking rates, emphasizing the importance of quality service and responsiveness in the competitive vacation rental market.

```{r, echo=FALSE, message=FALSE}
data$host_is_superhost <- as.logical(data$host_is_superhost)
data$host_response_rate <- as.numeric(gsub("%", "", data$host_response_rate)) / 100
data$amenities_count <- lengths(strsplit(as.character(data$amenities), ",")) 
superhosts <- data %>% filter(host_is_superhost == TRUE)

ggplot(data, aes(x = host_is_superhost, y = host_response_rate, fill = host_is_superhost)) +
  geom_boxplot() +
  labs(title = "Response Rate Comparison by Superhost Status", x = "Is Superhost", y = "Response Rate")

ggplot(data, aes(x = host_is_superhost, y = amenities_count, fill = host_is_superhost)) +
  geom_boxplot() +
  labs(title = "Amenities Count by Superhost Status", x = "Is Superhost", y = "Amenities Count")

```
![image](https://github.com/user-attachments/assets/7cae1b81-8a8f-48a0-8dd8-e885e55c97ed)
![image](https://github.com/user-attachments/assets/dfb2cf17-def2-46f7-9d7b-f79cf73e8c98)


When analyzing property types, there was one specific insight that stood out. Villas that had "beachfront" listed in the amenities column had a high booking rate of 0.382, while villas without "beachfront" listed in the amenities column had a high booking of 0.092.

```{r, echo=FALSE, message=FALSE}
villas <- data[data$property_type == 'Entire villa', ]
beachfront_villas <- villas %>%
  filter(grepl("beachfront", amenities, ignore.case = TRUE))
no_beachfront_villas <- villas %>%
  filter(!grepl("beachfront", amenities, ignore.case = TRUE))
beachfront_summary <- beachfront_villas %>%
  summarise(High_Booking_Rate = mean(high_booking))
no_beachfront_summary <- no_beachfront_villas %>%
  summarise(High_Booking_Rate = mean(high_booking))
booking_summary <- data.frame(
  Villa_Type = c("Beachfront Villas", "No Beachfront Villas"),
  High_Booking_Rate = c(beachfront_summary$High_Booking_Rate, no_beachfront_summary$High_Booking_Rate)
)
ggplot(booking_summary, aes(x = Villa_Type, y = High_Booking_Rate, fill = Villa_Type)) +
  geom_bar(stat = "identity", position = position_dodge()) +
  ggtitle("High Booking Rate Comparison for Villa Types") +
  xlab("Villa Type") +
  ylab("High Booking Rate") +
  scale_fill_manual(values = c("lightblue", "lightgreen")) +
  theme_minimal()

```
![image](https://github.com/user-attachments/assets/b12db2cd-dcc5-4bbd-a0e6-26cb21153a4f)

We then decided to compare the median price between beachfront and non-beachfront villas. We found that the median price of beachfront villas was \$1680, while the median price of non-beachfront villas was \$621.50.

```{r, echo=FALSE, message=FALSE}
median_prices <- data.frame(
  Villa_Type = c("Beachfront Villas", "No Beachfront Villas"),
  Median_Price = c(
    median(beachfront_villas$price),
    median(no_beachfront_villas$price)
  )
)

ggplot(median_prices, aes(x = Villa_Type, y = Median_Price, fill = Villa_Type)) +
  geom_bar(stat = "identity", position = position_dodge(), width = 0.7) +
  ggtitle("Median Price Comparison for Villa Types") +
  xlab("Villa Type") +
  ylab("Median Price") +
  scale_fill_manual(values = c("lightblue", "lightgreen")) +
  theme_minimal()
```
![image](https://github.com/user-attachments/assets/aba02464-7823-407f-a45b-a15f202f41be)

This analysis shows that beachfront villas seem to be in very high demand, with an extremely high high booking rate compared to the rest of the data as well as an extremely high median nightly price. Even though villas with no beachfront seem to be considerably cheaper than those with a beachfront, their high booking rate is considerably lower.

We then analyzed the property types on each island. The breakdown below shows the top property types on each island, along with the share of the listings on the island and the high booking rate.


## Property Type and High Booking Rates by Island

### Maui
| Property Type         | % of Maui Listings | High Booking Rate |
|-----------------------|---------------------|-------------------|
| Condo                 | 68.7%               | 11%               |
| Apartment             | 11.5%               | 6.7%              |
| Villa                 | 5.4%                | 1.5%              |
| House                 | 3%                  | 7.6%              |
| Resort                | 2%                  | 40%               |

### Big Island
| Property Type         | % of Big Island Listings | High Booking Rate |
|-----------------------|--------------------------|-------------------|
| Condo                 | 28.4%                    | 14.2%             |
| House                 | 24.9%                    | 10.5%             |
| Apartment             | 7.5%                     | 20.9%             |
| Guest Suite           | 5.3%                     | 2.6%              |
| Private Room in House | 2%                       | 15.2%             |

### Molokai
| Property Type         | % of Molokai Listings | High Booking Rate |
|-----------------------|-----------------------|-------------------|
| Condo                 | 45.5%                 | 2.6%              |
| Townhouse             | 21.3%                 | 2.7%              |
| Apartment             | 20.1%                 | 11.8%             |
| House                 | 9.3%                  | 0%                |
| Cottage               | 2.9%                  | 40%               |

### Oahu
| Property Type         | % of Oahu Listings    | High Booking Rate |
|-----------------------|-----------------------|-------------------|
| Condo                 | 37.6%                 | 6%                |
| Apartment             | 17.5%                 | 9%                |
| House                 | 12.3%                 | 9%                |
| Private Room in House | 4.7%                  | 10.5%             |
| Guest Suite           | 2.9%                  | 7.8%              |

### Kauai
| Property Type         | % of Kauai Listings   | High Booking Rate |
|-----------------------|-----------------------|-------------------|
| Condo                 | 52.2%                 | 10.9%             |
| House                 | 15.7%                 | 6.1%              |
| Apartment             | 7.7%                  | 13.4%             |
| Townhouse             | 4%                    | 1.9%              |
| Villa                 | 3.8%                  |​ 13.1%



### Choosing Models

We soon realized developing one model to explain high booking in Hawaii would be difficult because each island differs so much in property types and features. We decided to narrow down our models to the property type we could best predict on each island.

We used the tables in the above section to drive which property types we would develop models for on each island. From these tables, we can see that the property types that make up the largest percentage on each island do not have the highest high booking rate. Higher high booking rates allow us to create better models because there is more balanced data between high booking and not high booking listings. We also observed property types that make up a smaller percentage of listings are in higher demand and tend to have a higher high booking rate. However, because there is less data on these property types, it is more difficult to develop a model. Therefore, to develop the best models, we must strike a balance between high booking rate and size of data.

We explored a few property types for each island and found we were able to develop the most performant models for the following property types: resorts on Maui, apartments in Kauai, apartments on the Big Island, and private room in house on Oahu. We decided to omit Molokai as the amount of data was too small to develop a performant model, which will be discussed further in the following section.

### Models and Findings

#### Resorts in Maui

There are 172 resort property type listings in Maui, which makes up just 2% of listings on Maui. The high booking rate in our dataset for these listings is very high at 40%.

For these properties, the following variables have a highly negative correlation with high booking: guests included, maximum nights, review scores for location, security deposit, cleaning fee, and lanai in the summary.

The following variables have a highly positive correlation with high booking: "kaanapali" in the listing name and "resort" in the listing summary.

```{r, echo=FALSE, message=FALSE}
library(randomForest)
maui = filter(data, market == 'Maui')
maui_resort = filter(maui, property_type == 'Resort')
#guests_included: -.42
#maximum_nights: -.512
#review_scores_location: -.5
#security_deposit: -.5
#cleaning_fee: -.544
#kaanapali name: .4
#resort_summary: .4
#lanai_summary: -.4
```

Since high booking rate was so high for resorts on Maui and these variables had such a high correlation, we decided to build a model that used these variables to try and predict high booking rate for resorts on Maui.

This model is highly successful at predicting high booking on the testing data, with a 86% true positive rate and a 98% true negative rate. Please see model details in the appendix.

```{r, echo=FALSE, message=FALSE}
maui_high_booking_resorts <- maui_resort[maui_resort$high_booking == 1, ]
maui_low_booking_resorts <- maui_resort[maui_resort$high_booking == 0, ]
maui_high_booking_resorts$Booking <- 'High'
maui_low_booking_resorts$Booking <- 'Low'
combined_data <- rbind(maui_high_booking_resorts, maui_low_booking_resorts)
ggplot(combined_data, aes(x = Booking, y = price, fill = Booking)) +
  geom_boxplot() +
  labs(title = "Price Distribution by High Booking",
       x = "High Booking",
       y = "Price") +
  theme_minimal()
# Combine the data frames
ggplot(combined_data, aes(x = Booking, y = price, fill = Booking)) +
  geom_violin(trim = FALSE) +
  labs(title = "Price Distribution by High Booking",
       x = "High Booking",
       y = "Price") +
  theme_minimal()
ggplot(maui_high_booking_resorts, aes(x = accommodates)) +
  geom_histogram(binwidth = 1, fill = "lightblue") +
  labs(title = "Distribution of Accommodates High Booking Maui Resorts",
       x = "Number of Guests Accommodated",
       y = "Frequency") +
  theme_minimal()
ggplot(maui_low_booking_resorts, aes(x = accommodates)) +
  geom_histogram(binwidth = 1, fill = "lightblue") +
  labs(title = "Distribution of Accommodates Low Booking Maui Resorts",
       x = "Number of Guests Accommodated",
       y = "Frequency") +
  theme_minimal()
```
![image](https://github.com/user-attachments/assets/70e08b5c-c5cb-4c93-bdf3-92c9ab55aae9)
![image](https://github.com/user-attachments/assets/19047a27-ad40-4986-a9b1-ac318324ffb3)
![image](https://github.com/user-attachments/assets/f7da35f2-6b38-4a2a-9902-cc352b3d1da5)
![image](https://github.com/user-attachments/assets/7b6febef-2eaa-4450-a27f-c031a225d82e)


For an investor looking to invest in Maui resort, the deal may seem lucrative. However, it is important to remember that Airbnb listings in resort rooms are usually from third party entities, which likely explains why there are so very few listings in this dataset. The median nightly price for resorts on Maui is \$489, while the median nightly price for high booking resorts was \$225 and the median nightly price for non-high booking resorts was \$680. High booking rooms seemed to be priced between \$200-\$300, while non-high booking rooms seem to be priced above this, so setting a competitive nightly price will be imperative. It's also worth noting that when analyzing the accommodates variable, all high booking resorts accommodated four or more people, while the low booking resorts had 17 observations where the room only accommodated two people. If an investor is looking to invest in resort rooms on Maui, our recommendations are that they purchase rooms where they will be able to set a competitive nightly price ideally between \$200-\$300, as well as ensuring that they are purchasing rooms that can be accommodated by four or more people.

#### Apartments in Kauai
There are 183 enter apartments in Kauai but only 17 are high booking with a high booking rate of only 9.3%. Variables chosen by Shadow include the number of people the property can accommodate, the price, the minimum number of nights required to book, the frequency of reviews per month, as well as the length of the property's name, the cost of cleaning, the length of the rules, and so on. Of these, price and length of rules and minimum days of stay are key variables.

This model is highly successful at predicting high booking on the testing data, with a 91.08% accuracy along with a sensitivity of 0.6956, a specificity of 0.93854.

For Airbnb hosts or some other investor looking to increase their bookings, setting competitive prices is key. We recommend pricing your listing around \$135 per night, as this is the median rate for highly booked Airbnbs, compared to \$175 for those with fewer bookings. Also, keeping your listing name short, ideally no more than 8 characters, can make it more appealing. Finally, setting the minimum stay to less than 4 days can attract more guests looking for shorter visits. These simple changes can help make your Airbnb more attractive to potential guests.

#### Apartment on the Big Island

```{r, echo=FALSE, message=FALSE}
big_island = filter(data, market == 'The Big Island')
big_island_apartment = filter(big_island, property_type == 'Entire apartment')
```

There are 320 apartment listings on the Big Island in our dataset and the overall high booking rate is 20.9%. One of the Big Island's draws is the volcanic activity and hiking on the island. It is also typically an island where people either stay for their vacation or only stay there for a few nights while island hopping.

The model we developed for apartments on the Big Island is a logistic regression model that uses the following variables: accommodates, price, maximum nights, reviews per month, and length of the name of the listing. All but reviews per month have a negative relationship with high booking (See Appendix for model details). The model has a true positive rate of 77% and a true negative rate of 77%.

There is a noticeable difference in the price of high booking apartments versus not high booking apartments. The median price of a high booking apartment is \$89 and the median price of a not high booking apartment is \$129, almost 1.5 times that of the high booking listings. It is obvious that many people are looking for a cheaper price when it comes to apartments on the Big Island, probably due to island hopping.

```{r, echo=FALSE, message=FALSE}
ggplot(big_island_apartment, aes(x = price)) +
  geom_histogram(binwidth =5, fill = "steelblue") +
  labs(title = "Price by High Booking Rate", x = "Price", y = "Frequency") +
  facet_wrap(~ high_booking, nrow = 1) +
  theme_minimal()
```

There is also a noticeable difference in the distribution of the number of people the listing accommodates for the high booking listings compared to not high booking listings. Very few high booking apartments accommodate more than 4 people. This is probably because larger groups of people tend towards a larger space, like a house or a villa.

```{r, echo=FALSE, message=FALSE}
ggplot(big_island_apartment, aes(x = accommodates)) +
  geom_histogram(binwidth =.5, fill = "steelblue") +
  labs(title = "Accommodates by High Booking Rate", x = "Accommodates", y = "Frequency") +
  facet_wrap(~ high_booking, nrow = 1) +
  theme_minimal()
```

If an investor is looking to invest in an apartment on the Big Island, we would recommend keeping prices low (less than \$100 a night) and keeping the number of people the space can accommodate low to appeal to island hoppers looking for a cheaper place to stay for a few nights.

#### Private Room in House on Oahu

```{r, echo=FALSE, message=FALSE}
oahu = filter(data, market == 'Oahu')
oahu_private_room = filter(oahu, property_type == 'Private room in house')
```

Many people who visit Oahu are not resort tourists but vacationers who are there mainly to surf. Private rooms in houses are ideal for these types of vacationers because they are less expensive, smaller, and less amenities. There are 285 listings of private rooms in houses on Oahu, which makes up about 4.7% of Oahu listings. 10.5% of private room in house listings in Oahu have a high booking rate, which is the highest high booking rate out of all property types on Oahu.\

The model we developed for private rooms in houses in Oahu is a logistic regression model, using the following variables: host acceptance rate, extra people, minimum nights, security deposit, cleaning fee, "quiet" in the house rules (binary variable), and number of words in the house rules. All of these variables have a negative relationship with high booking (See Appendix for model details). This model has an 80% true positive rate and 91% true negative rate, which is very good for an unbalanced dataset.\

```{r, echo=FALSE, message=FALSE}
#Model for reference, do not show in report
m1 = suppressWarnings(glm(as.factor(high_booking) ~ host_acceptance_rate + extra_people + minimum_nights + security_deposit 
        + cleaning_fee + quiet_house_rules + house_rules_length, data = oahu_private_room, family="binomial")) #F1: .69, sens:.8, spec: .91
```

In our model, the two most significant variables are cleaning fee and minimum nights. There is a negative relationship between cleaning fee and high booking, which is not surprising as most people are looking to pay the least amount possible. There is also a negative relationship between minimum nights and high booking. Many people who visit Hawaii like to island hop, which means they only stay for a few nights on each island. As seen below, the listings that do not have a high booking rate have a higher rate of minimum nights over 25. However, as also seen below, there are many listings that do not have a high booking rate that have low minimum nights so this is not the only variable to consider.\

```{r, echo=FALSE, message=FALSE}
ggplot(oahu_private_room, aes(x = minimum_nights)) +
  geom_histogram(binwidth = 1, fill = "steelblue") +
  labs(title = "Minimum Nights by High Booking Rate", x = "Minimum Nights", y = "Frequency") +
  facet_wrap(~ high_booking, nrow = 1) +
  theme_minimal()
```

Another aspect to consider is the house rules. None of the listings with a high booking rate mentioned any rules around being quiet in the house rules. For listings without a high booking rate, 28% mentioned rules around being quiet in the house. In addition, the listings with a high booking rate overall have less house rules. As you can see below, there are many more non-high booking listings with a large amount of house rules. 32% of listings without a high booking rate have house rules over 75 words. This is opposed to the maximum amount of words in the house rules of the high booking listings, 31.\

```{r, echo=FALSE, message=FALSE}
ggplot(oahu_private_room, aes(x = house_rules_length)) +
  geom_histogram(binwidth = 5, fill = "steelblue") +
  labs(title = "Length of House Rules by High Booking Rate", x = "Words in House Rules", y = "Frequency") +
  facet_wrap(~ high_booking, nrow = 1) +
  theme_minimal()
```

In conclusion, our recommendation for a private room in a house in Oahu is to decrease the minimum nights to under 3, do not have rules about being quiet, and to overall decrease the amount of house rules for the listing.\

#### Molokai

In our dataset, there are only 169 listings on the whole island of Molokai. It is also very unbalanced data with only 9 (5%) listings that have a high booking. Out of the 9 high booking listings, 4 of them are the same listing at different time points. Because of this, building a model that performs well is extremely difficult. In order to fully analyze listings on the island of Molokai and provide recommendations, our team would need more data points of listings on Molokai.

## Results and Findings

In our analysis, we were able to find features of specific property types on each island that correlate with a high booking rate. It was clear to us through our analysis that each property type and island appeal to a different type of vacationer and therefore, have different features that appeal to these guests. Resorts in Maui that perform well advertise that the listing is a resort and has access to a very well known and beautiful beach on Maui. Apartments on the Big Island that perform well have lower prices and accommodate less people to appeal to island hoppers. Similarly, private rooms in Oahu that perform well keep minimum nights low and keep house rules to a minimum to entice surfers and island hoppers to stay in their Airbnb. Apartments on Kauai that perform well have a lower minimum night policy, less house rules, and a short name on the listing.

Because of these differences, our recommendations to the investor vary based on the property type and island. If investing in a resort on Maui, make sure to advertise well that it is in a resort and the prime location on the beach. If investing in an apartment on the Big Island, keep prices below \$100 a night and accommodations below 4 people. If investing in a private room in a house on Oahu, lower the minimum night requirement and keep house rules to a minimum. If investing in an apartment in Kauai, keep minimum nights and name of the listing low and have less house rules. Overall, we found that being a superhost does not make a difference as all hosts on Hawaii have a high response rate and high acceptance rate. We recommend to keep active and make sure to respond pretty quickly to keep up with the rest of the hosts but don't worry if you are not a superhost because we did not find this to make any significant difference in the high booking rate.

These recommendations are based on our models. All of our models have a relatively high true positive and true negative rate (over 75%). This tells us the features we included in each model can predict the high booking rate for that particular property type on that particular island pretty well but there is room for other features that were potentially not included in this dataset. There is also always a factor of randomness what it comes to real life predictions that we have to be aware of when developing models.

## Conclusion and Discussion

In this study, we were able to find features of property types on each island in Hawaii that helped to explain high booking rate for those listings. Through these features, we were able to make specific recommendations to an investor if they are looking to invest in an Airbnb in Hawaii.

Developing a model to explain high booking rate with unbalanced data in 8 weeks has been a challenge. In future research, we would like to spend more time on diving into the complete Hawaii dataset to see if it is possible to develop a model that can explain high booking for the full state. It also would be interesting to do K means analysis to break the full dataset into groups instead of choosing from the list of property types on each island. Another feature that would be interesting to incorporate are the pictures of the Airbnbs. This is probably one of the main draws to an Airbnb because the pictures are one of the first things presented to the potential customer.

## Appendix

### Model for Maui Resorts

```{r}
features <- maui_resort[, c('guests_included', 'maximum_nights', 'review_scores_location',
                             'security_deposit', 'cleaning_fee', 'kaanapali_name',
                             'resort_summary', 'lanai_summary', 'high_booking')]
features$high_booking <- as.factor(features$high_booking)
data_matrix <- xgb.DMatrix(data = as.matrix(features[, -which(names(features) == "high_booking")]),
                           label = as.numeric(features$high_booking) - 1) 

params <- list(
  booster = "gbtree",
  objective = "multi:softmax",  
  num_class = length(unique(features$high_booking)), 
  eval_metric = "mlogloss",
  eta = 0.1,
  max_depth = 6
)

xgb_model <- xgb.train(params = params, data = data_matrix, nrounds = 100, nthread = 1)
predictions <- predict(xgb_model, data_matrix)
predicted_classes <- as.factor(predictions + 1)  # Adjust index if necessary
predicted_classes <- factor(predictions + 1, levels = seq_along(levels(features$high_booking)))
predicted_classes <- factor(predictions + 1, levels = 1:length(levels(features$high_booking)), labels = levels(features$high_booking))
confusion_matrix <- confusionMatrix(data = predicted_classes, reference = features$high_booking)
print(confusion_matrix)

```

### Model for Apartments on the Big Island

```{r}
summary(glm(as.factor(high_booking) ~ accommodates + price 
         + maximum_nights + reviews_per_month + name_length, data = big_island_apartment, family="binomial"))
```

### Model for Private Room on Oahu

```{r}
summary(glm(as.factor(high_booking) ~ host_acceptance_rate + extra_people + minimum_nights + security_deposit 
        + cleaning_fee + quiet_house_rules + house_rules_length, data = oahu_private_room, family="binomial"))
```
### Model for Entire Apartment on Kauai 
```{r}
kauai_apartments <- subset(data, property_type == "Entire apartment" & market == "Kauai")
numeric_columns <- c('guests_included', 'maximum_nights', 'review_scores_location', 'security_deposit', 
                     'cleaning_fee', 'kaanapali_name', 'resort_summary', 'lanai_summary', 'high_booking')
kauai_apartments[numeric_columns] <- lapply(kauai_apartments[numeric_columns], function(x) {
  as.numeric(as.character(x))
})

kauai_apartments[numeric_columns] <- lapply(kauai_apartments[numeric_columns], function(x) {
  if(any(is.na(x))) x[is.na(x)] <- mean(x, na.rm = TRUE)
  x
})

train_data <- as.matrix(kauai_apartments[, setdiff(numeric_columns, "high_booking")])
train_labels <- kauai_apartments$high_booking

print(paste("Dimension of train_data:", dim(train_data)))
print(paste("Length of train_labels:", length(train_labels)))

if (nrow(train_data) == length(train_labels)) {
  
  data_matrix <- xgb.DMatrix(data = train_data, label = train_labels)
  params <- list(
    booster = "gbtree",
    objective = "binary:logistic",  
    eval_metric = "logloss",
    eta = 0.1,
    max_depth = 6
  )
  xgb_model <- xgb.train(params = params, data = data_matrix, nrounds = 100, nthread = 1)
  predictions <- predict(xgb_model, data_matrix)
  predicted_classes <- as.factor(predictions > 0.5)  
  confusion_matrix <- table(Predicted = predicted_classes, Actual = train_labels)
  print(xgb_model)
  print(confusion_matrix)
} 
accuracy <- sum(diag(confusion_matrix)) / sum(confusion_matrix)
sensitivity <- confusion_matrix[2, 2] / sum(confusion_matrix[2, ])
specificity <- confusion_matrix[1, 1] / sum(confusion_matrix[1, ])
ppv <- confusion_matrix[2, 2] / sum(confusion_matrix[, 2])
npv <- confusion_matrix[1, 1] / sum(confusion_matrix[, 1])
prevalence <- sum(confusion_matrix[2, ]) / sum(confusion_matrix)
detection_rate <- confusion_matrix[2, 2] / sum(confusion_matrix)
detection_prevalence <- sum(confusion_matrix[, 2]) / sum(confusion_matrix)
balanced_accuracy <- (sensitivity + specificity) / 2
cat("Accuracy:", accuracy, "\n")
cat("Sensitivity:", sensitivity, "\n")
cat("Specificity:", specificity, "\n")
cat("Positive Predictive Value (PPV):", ppv, "\n")
cat("Negative Predictive Value (NPV):", npv, "\n")
cat("Prevalence:", prevalence, "\n")
cat("Detection Rate:", detection_rate, "\n")
cat("Detection Prevalence:", detection_prevalence, "\n")
cat("Balanced Accuracy:", balanced_accuracy, "\n")
```


