# Coursera_Capstone
Applied Data Science Capstone

# Exploration of Edinburgh's short rental market 
## Airbnb price prediction with Machine Learning.

*Keywords: Airbnb, Edinburgh, city, data, data science, design, geopandas, geospatial, gis, land use, livability, maps, matplotlib, modeling, neighborhood, networks, new urbanism, numpy, pandas,pandana, osm, planning, python, smart cities, smart growth, urban, urban design, urban planning, visualization*

### Project background and aim

 Airbnb is a internet marketplace for short-term home and apartment rentals. It allows you to, for example, rent out your home for a week while you’re away, or rent out your empty bedroom. One challenge that Airbnb hosts face is determining the optimal nightly rent price. In many areas, renters are presented with a good selection of listings and can filter by criteria like price, number of bedrooms, room type, and more. Since Airbnb is a market, the amount a host can charge is ultimately tied to market prices.

Although Airbnb provides hosts with general guidance, there are no easy to access methods to determine the best price to rent out a space. There is third-party software available, but for a hefty price (see an example on available software, click [here](https://beyondpricing.com/)).

One method could be to find a few listings that are similar to the place that will be up for rent, average the listed prices and set our price to this calculated average price. However, with the market being so dynamic, we would probably be looking to update the price regularly and this method can become tedious. Also, this is not be very accurate, as we are not taking into account other important factors that may give us a comparative advantage over other listings around us. This could be property characteristics such as number of rooms, bathrooms and extra services on offer. 

The aim of this project is to propose a data-driven solution, by using machine learning to predict rental price.

For this project, a predictor based on space will is used to the model: the property's proximity to certain venues. This will allowed the model to put an implicit price on things such as living close to a bar or a supermarket.

I published a post in Medium: **to be updated**, a more detailed report can be found [here](https://gracecarrillo.github.io/Coursera_Capstone/#some_useful_references) and the notebook [here](https://nbviewer.jupyter.org/github/gracecarrillo/Coursera_Capstone/blob/master/Exploring_Edinburgh_Graciela_Carrillo.ipynb). 

### Findings and conclusion:

After a rather long notebook and playing with a lot of variables, the best performing model was able to predict 65.56% of the variation in price with an RMSE of 0.19. Several other features are not part of our dataset or needed to be analysed more closely. It was noticeable that reviews about listing location, rather than the location features themselves, were higher in the feature importance list. Thus, perhaps Airbnb hosts should highlight this when writing their listing's description. Highlighting accessibility and location benefits of staying with them could perhaps benefit them and how much they can ask for their listing. This also shows the importance of reviews. So, image quality analysis is an interesting path to follow.
