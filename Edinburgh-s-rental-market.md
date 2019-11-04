---
title: "<center><div class='mytitle'>Exploring Edinburgh's short rental Market</div></center>"
subtitle: "<center><div class='mysubtitle'>Airbnb price prediction with Machine Learning</div></center>"
author: "<center><div class='mysubtitle'>Graciela Carrillo</div></center>"
output: 
   html_document:
     css: newstyle.css
     number_sections: true
     keep_md: true
     fig_height: 6
     fig_width: 6
     fig_caption: yes
     toc: yes
     toc_float: true
     includes:
      after_body: footer.html
      before_body: header.html

---
<style>
body {
text-align: justify}
</style>

<br><br>

<div class="mycontent">



*Keywords: Airbnb, Edinburgh, city, data, data science, design, geopandas, geospatial, gis, land use, livability, maps, matplotlib, modeling, neighborhood, networks, new urbanism, numpy, pandas, foursquare API, planning, python, smart cities, smart growth, urban, urban design, urban planning, visualization*

***

# Project background and aim
<br><br><br>

Airbnb is a internet marketplace for short-term home and apartment rentals. It allows you to, for example, rent out your home for a week while you’re away, or rent out your empty bedroom. One challenge that Airbnb hosts face is determining the optimal nightly rent price. In many areas, renters are presented with a good selection of listings and can filter by criteria like price, number of bedrooms, room type, and more. Since Airbnb is a market, the amount a host can charge is ultimately tied to market prices. The search experience on Airbnb looks like this: 

<br><br><br>
<center>
![Airbnb site](https://cdn0.tnwcdn.com/wp-content/blogs.dir/1/files/2013/10/Screenshot_19.jpg)
</center>
<br><br><br>
 
Although Airbnb provides hosts with general guidance, there are no easy to access methods to determine the best price to rent out a space. There is third-party software available, but for a hefty price (see an example on available software, click [here](https://beyondpricing.com/)).

One method could be to find a few listings that are similar to the place that will be up for rent, average the listed prices and set our price to this calculated average price. However, with the market being so dynamic, we would probably be looking to update the price regularly and this method can become tedious. 

Moreover, this may not be very accurate, as we are not taking into account other important factors that may give us a comparative advantage over other listings around us. This could be property characteristics such as number of rooms, bathrooms and extra services on offer. 

The aim of this project is to propose a data-driven solution, by using machine learning to predict rental price.

For this project, a predictor based on space will be introduced to the model: the property's proximity to certain venues. This will allow the model to put an implicit price on things such as living close to a bar or a supermarket.

# Data Description
<br><br><br>

Airbnb does not release any data on the listings in its marketplace, a but separate group named [Inside Airbnb](http://insideairbnb.com/get-the-data.html) scrapes and compiles publicly available information about many cities Airbnb's listings from the Airbnb web-site. For this project, their data set scraped on July 21, 2019, on the city of Edinburgh, Scotland, is used. It contains information on all Edinburgh Airbnb listings that were live on the site on that date (over 14,000). Here's a [direct link](http://insideairbnb.com/edinburgh/).

The data has certain limitations. The most noticeable one is that it scrapes the advertised price rather than the actual price paid by previous customers. More accurate data is available for a fee in sites like [AirDNA](https://www.airdna.co/).

Each row in the data set is a listing available for rental in Airbnb's site for the specific city (observations). The columns describe different characteristics of each listing (features).

Some of the more important features this project will look into are the following: 

- `accommodates`: the number of guests the rental can accommodate
- `bedrooms`: number of bedrooms included in the rental
- `bathrooms`: number of bathrooms included in the rental
- `beds`: number of beds included in the rental
- `price`: nightly price for the rental
- `minimum_nights`: minimum number of nights a guest can stay for the rental
- `maximum_nights`: maximum number of nights a guest can stay for the rental
- `number_of_reviews`: number of reviews that previous guests have left

To model the spatial relationship between Airbnb rental prices and property proximity to certain venues, we use the [Foursquare API](https://developer.foursquare.com/) to access the city's venues and the street network, available though [OpenStreepMap (OSM)](https://www.openstreetmap.org/#map=12/55.9408/-3.1835).

# Methodology

## Data cleaning

The original dataset contained 14014 Airbnb listings and 106 features. Natural Language processing was not used in for the model. Therefore, free text columns were dropped as well as other features not useful for predicting price (e.g. url, host name and other host-related features that are unrelated to the property). Multiple columns for property location were found. Due to all of the listings being located in Edinburgh, the country and city columns were dropped. There were multiple columns for minimum and maximum night stays, but the two main ones were used as there were few differences between  them (e.g. `minimum_nights` and `minimum_minimum_nights`). 

A visual inspection of whether boolean and categorical features contained sufficient numbers of instances in each category to make them worth including led to the realisation that several columns only contained one category and could be dropped. There were `has_availability`, `host_has_profile_pic`, `is_business_travel_ready`, `require_guest_phone_verification`, `require_guest_profile_picture`, and `requires_license`.

<br><br><br>
<center>
![Visual Inspection of Boolean and Categorical Features](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Histogram1.png?raw=truestyle=centerme 'Visual Inspection of Boolean and Categorical Features')
</center>
<br><br><br>

The experiences offered feature was also dropped, due do most listings not offering it. About a third of rows did not have a value for `host_response_time`, and the majority of these have also had not yet been reviewed. Therefore this section of the data set consisted primarily of properties which had not yet had a completed stay (most likely properties which had not yet had a booking, although may also include properties that had a booking currently occurring). Although this is a considerable proportion of the dataset, these listings were retained in the data because they were still legitimate properties with advertised prices and part of the comparative market when considering the price for which to advertise an Airbnb listing. However, if the dataset being used had the actual average price paid as its target, it would be necessary to drop these rows because they would not have a value, as they had not yet been booked. Because `host_response_time` is unknown for so many listings, it was be retained as its own category, 'unknown'.

The same applied to `host_response_rate`, with about a third of values being null. The feature was kept as its own category, after grouping other values into meaningful groups (i.e. transforming this into a categorical feature, rather than a numerical one). Because about 75% of hosts responded 100% of the time, this was kept as its own category, and other values were grouped into bins. There were 21 rows lacking values for each of five different host-related features, so those rows were dropped.

Some cleaning of property types features was required as there were a large number of categories with only a few listings. The categories `Apartment`, `House` and `Other` were used, as most properties could be classified as either apartment or house.

For `bathroom`, `bedroom` and `beds` missing values were replaced with the median. Most listings have the same bed type, so it was considered that it would not constitute a comparative advantage to have this feature. Hence, it was dropped. 

The `amenities` feature was a list of additional features in the property, i.e. whether it has a TV or parking. It is understood that some amenities are more important than others (i.e. a balcony is more likely to increase price than a fax machine), and some are likely to be fairly uncommon (i.e.. 'Electric profiling bed'). For the purpose of this project, amenities were extracted based on quick research into which amenities were considered by guests as important, as well as personal experience. This was further investigated in the EDA section. One way to reduce the number of features was to remove the amenities that added relatively little information, or were relatively unhelpful in differentiating between different listings. Amenity features where either the true or the false category contained fewer than 10% of listings were removed.

There were 96 unique categories for `calendar_updated`, and it was not entirely clear what this feature was adding to the model (a host might update their calendar for multiple different reasons). Therefore this column was dropped.

There were also multiple different measures of availability, which would have resulted in a multicollinearity problem. Thus, only one feature was retained, availability for 90 days (`availability_90`). This was due to the seemingly inevitable enforcement of London-like regulations proposed by the Scottish Government (90 days per year rents) after significant and growing concern from local residents and housing organisations over the increase of Airbnb and other short term lets within Edinburgh city, where there is now one Airbnb listing for every 11 residents.

Almost 20 percent of listings had not had a review written for them. This was too large a proportion of the dataset to drop, and dropping the columns would have resulted in the loss of a lot of useful information because reviews are very important in people's decisions to book, and therefore price.

This was also too large a proportion of the dataset to simply replace with median/mean values, as this would have skewed the distribution substantially. Also, the missing values here were not really missing values, as the fact that were `NaN`s was meaningful (it tells us that these are new or previously unbooked listings that have not had reviews yet). In order to make the resulting model work able to predict prices for any Airbnb listing, including brand new listings, is actually beneficial to keep them in. Therefore, these were kept as an `unknown` category, and the feature was treated as categorical (and therefore one-hot encoded) rather than numerical.

As above, listings without reviews were kept and replaced with `unknown`. Other ratings were grouped into bins. The majority of ratings were 9 or 10 out of 10. Therefore for these columns, 9/10 and 10/10 were kept as separate groups, and 1-8/10 were binned together (as this is, by Airbnb standards, a 'low' rating).

The `cancellation policy` feature was categories into three larger categories: super-strict, strict and moderate (e.g. the super strict options were only available to long-term Airbnb hosts, and is invitation only).

The features `number_of_reviews_ltm` and `reviews_per_month` were deemed to be likely highly correlated with `number_of_reviews` and so were dropped.

Numerical features `price`, `security deposit`, `cleaning fee`and `extra-people` were converted to integers (e.g. price was a string, as it contained the currency sign). Having a missing value for security deposit was considered to be functionally the same as having a security deposit of £0, so missing values were replaced with 0. As with security deposit, having a missing value for cleaning fee and extra people was also replaced with 0.

# Exploratory Data Analysis

## Time Series

Time is an important factor to consider in a model when we wish to predict prices or trends. A time series is simply a series of data points ordered in time. In a time series, time is often the independent variable and the goal is usually to make a forecast for the future. There are also other questions that come to play when dealing with time series. For example: Is there any seasonality to the price? Is it stationary? Even though we may not be able to include this aspect into our model, it is good to explore it to be aware of it and be able to make recommendations for future research. Thus, in this section, we will explore this aspect of the data.

For Airbnb prices, a high level of seasonality is expected due to the characteristics of the market. In August, during the Edinburgh Fringe Festival, room rental price rises considerably. Is an extremely popular event and much of the rented property available will have been taken up due to the number of people who attend each year. Edinburgh has lots of other art events throughout the year, mainly focused between April and December. There's word-class artists performing throughout these months. For the rest of the year, Edinburgh is uniquely placed as the cultural capital of, not just Scotland, but the UK. There's stable demand for accommodation and as well as tourism, because Edinburgh serves as a business capital after London.

<br><br><br>
<center>
![Edinburgh hosts joining Airbnb each month and receiving their first review](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Edinburgh_hosts_joining.png?raw=true 'Edinburgh hosts joining Airbnb each month and receiving their first review')
</center>

<br><br><br>
<center>
![Seasonality and Trend of Airbnb each month](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Hosts_Join_Airbnb_each_month.png?raw=true 'Seasonality and Trend of Edinburgh hosts joining Airbnb each month')
</center>
<br><br><br>

As seen above, there is a clear seasonality. Every year, there is a peak towards hosts joining around the middle of the year (summer), and the lowest points are the beginning and the end of each year. There is a big peak in the number of hosts joining Airbnb between 2015 and 2016. Indeed, there has been a fast growth of Airbnb since middle 2015, with clear peaks during Edinburgh Fringe Festival. This was the year when Airbnb became increasingly popular for short-term lease, as a way to get around local legislation and taxation. The same trend was observed in London, however, in London, the introduction of the 90-day rule for short-term rentals is another variable to consider. That legislation did not apply to Edinburgh.

Another important pattern to observe is the number of listings per owner/host. There are a number of professional Airbnb management companies which host a large number of listings under a single host profile. However, there is no consistent upwards trend in the average number of properties managed by each host.

<br><br><br>
<center>
![Change per year in number of listings per host in Airbnb Edinburgh](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Change_in_the_number_of_listings.png?raw=true 'Change per year in number of listings per host on Airbnb Edingburgh')
</center>
<br><br><br>

In term of changes in prices over time, the average price per night for Airbnb listings in Edinburgh has increased slightly over the last 10 years. In particular, the top end of property prices has increased, resulting in a larger increase in the mean price compared to the median. The mean price in 2010 was £107.33 and the median £115.0, whereas the mean price in 2018 (the last complete year of data) was £104.55 and the median £79.0.

<br><br><br>
<center>
![Change per year in nightly price of Airbnb listings in Edinburgh](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Change_per_year_in_nightly_price_listings.png?raw=true 'Change per year in the nightly price of Airbnb listings Edinburgh')
</center>
<br><br><br>

## Numerical Features

Looking at price distribution, advertised prices range from £0 to £12,345. The extreme ends of the range are due to hosts not understanding how to use Airbnb advertised prices correctly. The advertised prices can be set to any arbitrary amount, and these are the prices that show when dates are not entered on the site. Once you enter the dates you want to occupy the property, prices can vary a lot.

Unfortunately this model will be predicting advertised prices rather than the prices that were actually paid. Nonetheless, some cleaning of the particularly unhelpful values was done. Very small values under £10 were increased to £10. Values above £1,000 were reduced to £1,000.

<br><br><br>
<center>
![Advertised price distribution](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Advertised_prices_distribution.png?raw=true 'Advertised Price Distribution')
</center>
<br><br><br>

The median number of listings that the host of each listing has is 1. This means that on average (median) each listing is hosted by a host who only manages that listing. The mean is higher (8) due to some hosts managing some (very) large numbers of listings, as discussed above in the Time Series section. For example, the host with the highest number of listings has 1,058 listings under its ID. About half of listings are from hosts with one listing, and half are from multi-listing hosts.

Two difficulties in discerning how many listings hosts have on average are:

   1) this number is only known on the level of the listing, so hosts with more listings are represented more frequently (e.g a host with 10 listings may be represented up to 10 times in the dataset)

   2) a host's other listings may not be in Edinburgh, so some multi-listing hosts may appear multiple times in the dataset, and others may appear only once

The most common property setup sleeps two people in one bed in one bedroom, with one bathroom. Unsurprisingly, properties that accommodate more people achieve noticeably higher rates per night, with diminishing returns coming after about 10 people.

<br><br><br>
<center>
![Median Price per number of guest accommodated](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Median_Price_per_number_of_guests.png?raw=true 'Median Price per number of guest accommodated')
</center>
<br><br><br>

## Categorical Features

Much of Airbnb listings are centred around Edinburgh's Old Town, which is consistent with the huge draw for tourists especially during the annual Fringe festival.

<br><br><br>
<center>
![Airbnb listings per borough](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Number_listings_per_neighbourhood.png?raw=true 'Airbnb listings per borough')
</center>
<br><br><br>

About 80% of properties are apartments/flats. The remainder are houses or more uncommon property types (e.g. bed and breakfast).

About 62% of listings are entire homes (i.e. you are renting the entire property on your own). Most of the remainder are private rooms (i.e. you are renting a bedroom and possibly also a bathroom, but there will be other people in the property). Fewer than 1% are shared rooms (i.e. you are sharing a room with either the property owner or other guests).

For every review category, the majority of listings that have had a review have received a 10/10 rating for that category (or 95-100/100 overall). Ratings or 8 or below are rare. Guests seem to be most positive about communication, check-ins and accuracy. As noted previously, over a quarter of listings have not yet been reviewed.

The most common time period in which currently live Airbnb listings had their first review is 2-3 years. This means that a lot of listings on the site have been active for at least a couple of years. Relatively few have been active for more than four years, however.

The most common category for the time since a listing received its last review is 1+ years. This means that a lot of listings have not been reviewed recently. The majority of these are probably what are sometimes referred to 'inactive' listings, because although they are technically live on the site, they do not have their calendars open and are not available to book.

## Venue Proximity

As part of the model, proximity to certain venues as a possible price predictor was introduced. Walkability and ability to reach places maybe a deal-maker or breaker when it comes to choosing a accomodation. Proximity to certain venues, such as principal touristic attractions, restaurants, cafes and even shops could help us predict price. For this, the Foursquare API was used to explore the venues per neighbourhood. As discussed before, Old Town is the area which concentrates the majority of Airbnb listings.

The list of venues with their locations was retrieved and then used to explore the venues around the listings, using the latitude and longitude of each neighbourhood. Then, the most common venues were selected and used as points of interest (POIS) for the accessibility analysis.

It was found that the most common venues across the dataset were 'Hotel', 'Pub', 'Grocery Store', 'Supermarket', 'Cafe', 'Coffee Shop', with 'Bar', 'Bus Stop' and 'Indian Restaurant' coming behind. Different restaurant venues were in subcategories which made them less common than if they were aggregated. Thus, for the purpose of accounting for the venues that may have the most impact on price, the venues were limited to the most common categories. It is unlikely that having a hotel nearby will affect price, as Airbnb listings are considered to be on a different category of short-term rental, due to the different benefits they provide versus hotels. Thus, that category of venues was not be considered.

## Walkability to nearest venues

The information on venues was used to select Points of Interest (POIs) for which graphs and accessibility measures were created. For this purpose, OSMnx and Pandana libraries were used. Pandana is a handy graph library that allows for Pandas data frames to be passed through into a network graph that maps graph-level analyses to underlying C operations. In certain situations, such as the performance of accessibility analyses, this makes in-memory performance and iterative development based on this library possible, as opposed to what would be a cumbersome development process with tools that fail to leverage the same degree of C-level operations utilisation.

The street network data with 51,553 nodes was downloaded from the OSM API. Pubs, Restaurants, Cafes and Supermarkets/Grocery Stores are the POIs. Restaurants were aggregated, as they were divided in sub-categories in the POIs dataset. Thus, the POIs dataset consists of 173 restaurant venues, 87 bars, 67 pubs, 63 cafes and 51 Grocery Stores. 

## Accessibility from each node to amenities

This is where conducting geographical analysis in Python shows its power, as opposed to using a GIS package. First, a maximum search distance of 1 km for up to the 10-nearest points of interest was passed. This allowed a key step that speeds up future enquiries: Pandana builds a condensed representation of the network (implemented in C++), allowing rapid calculations within a defined radius of each node. Then, a table of distances to the nearest 5 points of interest from a couple of intersections was built. When this is finished, accessibility analyses for the selected amenities can be done in under a second leveraging contraction hierarchies and kd-trees algorithms within Pandana.

<br><br><br>
<center>
![alt text](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Walking_distance_nearest_amenity__any_Edinburgh.png?raw=true 'Walking distance to nearest amenity')
</center>
<br><br><br>

As seen above, there are some zones where people have to walk more than 500 meters to reach the nearest amenity, whereas Edinburgh's Old Town has walking distances of less than 100 meters on average.

The map shows the walking distance in meters from each network node to the nearest restaurant, bar, cafe, pub and grocery shop. However, a better indicator of accessibility might be having access to a large number of amenities. So instead of the nearest, accessibility to the fifth-nearest amenity was plotted. 

<br><br><br>
<center>
![alt text](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Walking_distance_nearest_amenity__5th_Edinburgh.png?raw=true 'Walking distance to 5th nearest amenity')
</center>
<br><br><br>

This time, is even more noticeable that Old Town and the city center area od Edinburgh is more accessible.

Accessibility scores can quickly be constructed to answer a given question: whether it's access to essential services, or walkable neighborhoods that appeal to young workers. For the purpose of this analysis, access to restaurants, shops, cafes, bars and pubs are taken as essential for Airbnb users. Hence, all amenities were weighted equally and distance to the fifth nearest amenity was usedas a compound measure of accessibility.  This gives a clearer picture of which neighborhoods are most walkable, compared with plotting just the distance to the single nearest venue/amenity.

<br><br><br>
<center>
![alt text](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Walking_distance_nearest_amenity__hexbins_Edinburgh.png?raw=true 'Walking distance to 5th nearest amenity in hexabins heatmap')
</center>
<br><br><br>

# Modeling

## Preparing the data

To assess multicollinearity, categorical variables were hot-encoded. The accessibility score (distance to the fifth nearest venue) was assigned to each listing, based on which neighbourhood they belong to. Thus, the geographical data and venue data columns were dropped, as they were no longer needed.

<br><br><br>
<center>
![alt text](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/multicolinearity_plot_initial.png?raw=true 'First multicollinearity plot')
</center>
<br><br><br>

On the above first analysis, it did not look like there were any significant collinear relationships with neighbourhood variables, so these were temporarily dropped to produce a clearer heatmap for the remaining features. 

<br><br><br>
<center>
![alt text](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/multicolinearity_plot_0.png?raw=true 'Second multicollinearity plot')
</center>
<br><br><br>

From the second analysis, certain areas of multi-collinearity were observed:

- `Beds`, `bedrooms`, `guests_included` and the number of people that a property accommodates were highly correlated. The number of people accommodated has traditionally been a more high priority search parameter on Airbnb, as it is more relevant for private and shared rooms than the number of bedrooms.

- Unsurprisingly, there were perfect correlations between `NaN` reviews (i.e. listings that are not reviewed yet) for different review categories, and first and last review times. `NaN` categories were therefore be dropped.

- The same is true of `host_response_rate_unknown` and `host_response_time_unknown`. The first rate was dropped.

- A correlation between `host_response_rate 0-49%` and `host_response_time_a few days or more` is clear. Thus, the first feature was dropped.

- Additionally, there were strong negative correlations between `property_type_House` and `property_type_Apartment`, and between `room_type_Private room` and `room_type_Entire_home_apt` (as these were the main two categories of their features before they were one-hot encoded). Although these are important categories, one of each was dropped in order to reduce multi-collinearity (apartments and private rooms, as these are the second most common categories).

After dropping problematic features, the following was the final multicollinearity assessment: 

<br><br><br>
<center>
![alt text](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/multicolinearity_plot.png?raw=true 'Final multicollinearity plot')
</center>
<br><br><br>

There were still some fairly strong correlations between highly rated properties of different reviews categories (i.e. if a property gets a 10/10 for one category, it is likely to get a 10/10 for other categories). However, these were left in for to be experimented with later in the modeling section, to see if removing them improves the model.

Other than `availability_90` and `host_days_active`, the remaining numerical features were all positively skewed, and thus log transformation was performed. The transformation helped some of the distributions, although some (e.g. `cleaning_fee`, `extra_people fee` and `security_fee`) contained a large number of 0s, hence these features are not normally distributed. Most importantly, however, the target variable `price` appeared to conform more closely to the normal distribution.

<br><br><br>
<center>
![alt text](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Log_Transformed_Num_Features.png?raw=true 'Log Transformed Numerical Features')
</center>
<br><br><br>

Finally, the predictive features and the target feature were separated and scaled for modeling. 

## Models

A Spatial Hedonic Price Model (OLS Regression), with the LinearRegression from Scikit-Learn library and the Gradient Boosting method, with the XGBRegressor from the XGBoost library were used and compared. The evaluation metrics used were mean squared error (for loss) and r-squared (for accuracy).

### Model 1: Spatial Hedonic Price Model (SHPM)

The hedonic model involves regressing observed asking-prices for the listing against those attributes of a property hypothesized to be determinants of the asking-price. It comes from hedonic price theory which assumes that a commodity, such as a house can be viewed as an aggregation of individual components or attributes (Griliches, 1971). Consumers are assumed to purchase goods embodying bundles of attributes that maximize their underlying utility functions (Rosen, 1974).

In addition to the characteristics of the Airbnb listings, we add location features as they have been shown to be important factors in influencing the price (see here, here, here and here for examples). Ideally, Lagrange multiplier tests should be conducted to verify if there is spatial lag in the dependent variable and therefore a spatial lag model is preferred for estimating a spatial HPM. However, for the purposes of this analysis, a conventional OLS model for hedonic price estimation that includes spatial and locational features is used, but not a spatial lag that accounts for spatial dependence.

The first explanatory variables are the listings characteristics (`acommodates`, `bathrooms`, etc) and the second group of explanatory variables based on spatial and locational features are `Score`, which is the network distance to 5th nearest venue computed with Pandana; and `Neighbourhood` belonging, 1 if the listing belongs to the specified neighbourhood, 0 otherwise.

Ridge Regularization was also experimented with, to decrease the influence of less important features. Ridge Regularization is a process which shrinks the regression coefficients of less important features. The Ridge Regularization model takes a parameter, alpha , which controls the strength of the regularization.

For this purpose, a few different values of alpha were tested to see how this changes results.

### Model 2: Gradient boosted decision trees

Boosting is an ensemble technique where new models are added to correct the errors made by existing models. Models are added sequentially until no further improvements can be made. A popular example is the AdaBoost algorithm that weights data points that are hard to predict.

Gradient boosting is an approach where new models are created that predict the residuals or errors of prior models and then added together to make the final prediction. It is called gradient boosting because it uses a gradient descent algorithm to minimize the loss when adding new models.

XGBoost (eXtreme Gradient Boosting) is an implementation of gradient boosted decision trees designed for speed and performance. Is a very popular algorithm that has recently been dominating applied machine learning for structured or tabular data.

This approach supports both regression and classification predictive modeling problems.

## Improving models

In the 'Preparing the data' section above, it was noted that a lot of the review columns are reasonably highly correlated with each other. They were left in to see whether they would be useful after all. However, the feature importances graph produced by the XGBoost model suggested that they were of relatively low importance.
Thus, all review columns other than the overall review rating we dropped, and the same Hedonic regression and XGBoost structure were used to see whether this produces a better models.

# Results

## Spatial Hedonic Price Regression Model

For the Spatial Hedonic Price Regression Model, the features explain approximately 51% of the variance in the target variable. 

<br><br><br>
<center>
<img src="https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Table1.png?raw=true" style="width: 50%; height: 50%"/>
</center>
<br><br><br>

Interpreting the Mean Squared Error value (RMSE) is somewhat more intuitive that the r-squared value. The RMSE measures the distance between predicted values and actual values. As the square root of a variance, RMSE can be interpreted as the standard deviation of the unexplained variance, and has the useful property of being in the same units as the response variable. Lower values of RMSE indicate better fit. RMSE is a good measure of how accurately the model predicts the response. This relationship can be observed graphically with a scatter plot:

<br><br><br>
<center>
![alt text](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/SHP_Regression_Model.png?raw=true 'Actual vs Predicted Price for Hedonic Regression')
</center>
<br><br><br>

If the predicted values were identical to the actual values, this graph would be the straight line $y = x$ because each predicted value $x$ would be equal to each actual value $y$. 

For the Ridge Regression, the RMSE values are are almost identical as the lambda value increases, which means the model prediction does not improve substantially with the ridge regression model.

<br><br><br>
<center>
![alt text](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/SHP_Regression_Model_Ridge_Reg_alpha001.png?raw=true 'Actual vs Predicted Price for Hedonic Regression with alpha=0.01')
    
![alt text](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/SHP_Regression_Model_Ridge_Reg_alpha01.png?raw=true 'Actual vs Predicted Price for Hedonic Regression with alpha=0.1')

![alt text](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/SHP_Regression_Model_Ridge_Reg_alpha1.png?raw=true.png?raw=true 'Actual vs Predicted Price for Hedonic Regression with alpha=1')

![alt text](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/SHP_Regression_Model_Ridge_Reg_alpha10.png?raw=true 'Actual vs Predicted Price for Hedonic Regression with alpha=10')

![alt text](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/SHP_Regression_Model_Ridge_Reg.png?raw=true 'Actual vs Predicted Price for Hedonic Regression with alpha=100')
</center>
<br><br><br>

## Gradient boosted decision trees

For the XGBoost model, the features explain approximately 65% of the variance in the target variable, with a lower RMSE compared to the Spatial Hedonic Model. 

<br><br><br>
<center>
<img src="https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Table2.png?raw=true" style="width: 50%; height: 50%"/>
</center>
<br><br><br>

Apart from its superior performance, a benefit of using ensembles of decision tree methods like gradient boosting is that they can automatically provide estimates of feature importance from a trained predictive model.

Generally, importance provides a score that indicates how useful or valuable each feature was in the construction of the boosted decision trees within the model. The more an attribute is used to make key decisions with decision trees, the higher its relative importance.

This importance is calculated explicitly for each attribute in the dataset, allowing attributes to be ranked and compared to each other.

Importance is calculated for a single decision tree by the amount that each attribute split point improves the performance measure, weighted by the number of observations the node is responsible for. The performance measure may be the purity (Gini index) used to select the split points or another more specific error function.

The feature importances are then averaged across all of the the decision trees within the model.

<br><br><br>
<center>
<img src="https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/XGB_Model_Feature_Importance.png?raw=true" style="width:50%; height:50%"/>
</center>
<br><br><br>
  
About a good number of features have a feature importance of 0 in this XGBoost regression model, and could potentially be removed.

The top 10 most important features are:

1. If the rental is the entire flat or not (`room_type_Entire home/apt`)
2. How many people the property accommodates (`accommodates`)
3. The type of property (`property_type_Other`)
4. The number of bathrooms (`bathrooms`)
5. How many days are available to book out of the next 90 (`availability_90`)
6. The number of reviews (`number_of_reviews`)
7. The cancellation policy being moderate (`cancellation_policy_moderate`)
8. How many other listings the host has (`host_listings_count`)
9. The minimum night stays (`minimum_nights`)
10. The maximum nights stay (`maximum_nights`)

The most important features the rental being the entire flat. Which makes sense. Asking price is higher if the offer is for the entire flat/house. This could also suggest that offering the flat/house as a whole, rather than each bedroom individually, may be better overall, given the large difference in importance compared to the second most important feature.

It is not surprising that the second how many people the property accommodates, as that's one of the main things you would use to search for properties with in the first place.

It is perhaps more surprising that location features did not appear in the top ten. Although we can observe that belonging to a certain neighbourhood increases price more than others and Score (accessibility measure) also shows some importance, they are of relative low importance compared to the top 3 features. Review Scores Location is higher on the importance list (number 11). This is, it is likely renters put more weight in other's opinion about location instead of judging the location based on neighbourhood and venues around the property. This could also be because Edinburgh is a small and walkable city with good transportation services. Thus, location is not a major problem to reaching main touristic attractions and amenities.

The eight most important feature is related to how many other listings the host manages on Airbnb, rather than the listing itself. This result showed on this analysis of Airbnb listings in London, only this feature was the third most important. What the researcher (and former data scientists at an Airbnb management company) explains is that this does not mean that a host that manages more properties will result in a listing gaining higher prices, and could be due to experienced hosts setting higher prices. Also, it could be that big Airbnb management companies that have lots of listings tend to manage more expensive properties than single listing hosts.

<br><br><br>
<center>
![](https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Median_Price_Listings_Multihosts.png?raw=true 'Median Price Multilisting Hosts')
</center>
<br><br><br>
                                                                                                                   
## Model improvement 

After dropping the review columns, both Spatial Hedonic Price Regression and XGBoost perform almost exactly the same without them. Hence, because they are able to achieve the same performance with 18 fewer columns, the second models are the preferred models as they require less data and are less computationally expensive.

<br><br><br>
<center>
<img src="https://github.com/gracecarrillo/Coursera_Capstone/blob/master/Images/Images/Table3.png?raw=true" style="width:50%; height:50%"/>
</center>
<br><br><br>

## Final model selection

Overall, the XGBoost model with dropped review columns is the preferred model, which performs better than both Spatial Hedonic Regression Models and just as good as the first model but is less computationally expensive. It could possibly be improved further with hyper-parameter tuning.

If the predicted values were identical to the actual values, this graph would be the straight line $y = x$ because each predicted value $x$ would be equal to each actual value $y$.


# Discussion and Conclusions

The best performing model was able to predict 65.56% of the variation in price with an RMSE of 0.19. This means there is a remaining 34.44% unexplained variance. This could be due to several other features that are not part of the dataset or the need to analyse the features more closely.

For example, given the importance of customer reviews of the listing in determining price, perhaps a better understanding of the reviews could improve the prediction. Using Sentiment Analysis, a score between -1 (very negative sentiment) and 1 (very positive sentiment) can be assigned to each review per listing property. The scores are then averaged across all the reviews associated with that listing and the final scores can be included as a new feature in the model.

Another suggestion is the inclusion of image quality as a feature. Using Difference-in-Difference deep learning and supervised learning analyses on a Airbnb panel dataset, researchers have found that units with verified photos (taken by Airbnb’s photographers) generate additional revenue per year on average.

It was noticeable that reviews about listing location, rather than the location features themselves, were higher in the feature importance list. Thus, this finding could perhaps be used by Airbnb hosts when writing their listing's description. Highlighting accessibility and location benefits of staying with them could perhaps benefit them and how much they can ask for their listing.

# Some useful references
<br><br><br>
Zhang, Shunyuan and Lee, Dokyun (DK) and Singh, Param Vir and Srinivasan, Kannan. (2017) How Much Is an Image Worth? Airbnb Property Demand Estimation Leveraging Large Scale Image Analytics. Available at SSRN: https://ssrn.com/abstract=2976021 or http://dx.doi.org/10.2139/ssrn.2976021

Kalehbasti, Pouya & Nikolenko, Liubov & Rezaei, Hoormazd. (2019). Airbnb Price Prediction Using Machine Learning and Sentiment Analysis. Available at: https://arxiv.org/pdf/1907.12665.pdf

J. Elith, J. R. Leathwick, T. Hastie. (2008) A working guide to boosted regression trees. Journal of Animal Ecology. Available at: https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/j.1365-2656.2008.01390.x

Chen, T., Guestrin, C., (2016) XGBoost: A Scalable Tree Boosting System, Proceeding KDD '16 Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining, Pages 785-794. Available at: https://dl.acm.org/citation.cfm?id=2939785

Rosen, Sherwin, (1974), Hedonic Prices and Implicit Markets: Product Differentiation in Pure Competition, Journal of Political Economy, 82, issue 1, p. 34-55, https://EconPapers.repec.org/RePEc:ucp:jpolec:v:82:y:1974:i:1:p:34-55.

Limsombunchai, V., (2004), House Price Prediction: Hedonic Price Model vs. Artificial Neural Network, Paper presented at the 2004 NZARES Conference Blenheim Country Hotel, Blenheim, New Zealand. Available at: https://core.ac.uk/download/pdf/35467021.pdf

Liv Osland (2010) An Application of Spatial Econometrics in Relation to Hedonic House Price Modeling. Journal of Real Estate Research: 2010, Vol. 32, No. 3, pp. 289-320. Available at: https://www.aresjournals.org/doi/abs/10.5555/rees.32.3.d4713v80614728x1

Fletcher, F, and Waddell, P. (2012), A Generalized Computational Framework for Accessibility: From the Pedestrian to the Metropolitan Scale. Transportation Research Board Annual Conference Available at: http://onlinepubs.trb.org/onlinepubs/conferences/2012/4thITM/Papers-A/0117-000062.pdf 

Luxen, D. and Vetter, C. (2011), Real-time routing with OpenStreetMap data. Proceedings of the 19th ACM SIGSPATIAL International Conference on Advances in Geographic Information Systems, New York, NY, USA,Available at: http://doi.acm.org/10.1145/2093973.2094062

<br><br><br>
</div>
