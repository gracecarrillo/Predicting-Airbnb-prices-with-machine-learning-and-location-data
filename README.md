# Coursera Capstone
Applied Data Science Capstone
 
# Predicting Airbnb prices with machine learning and location data 
## A case study using data from the City of Edinburgh, Scotland

*Keywords: Airbnb, Edinburgh, city, data science, pandas, geopandas, geospatial, foursquare, maps, matplotlib, modeling, neighbourhood, networks, numpy, foursquare API, planning, python, urban planning, data visualization*


<p align="center">
  <img width="460" height="300" src="https://github.com/gracecarrillo/Coursera_Capstone/blob/gh-pages/Profesional%20Certificate%20IBM%20Data%20Science.png?raw=true">
</p>

### Table of Contents

1. [Installation](#installation)
2. [Project Motivation](#motivation)
3. [File Descriptions](#files)
4. [Results](#results)
5. [Licensing, Authors, and Acknowledgements](#licensing)

## Installation <a name="installation"></a>

All necesary libraries are indicated at the beggining of the [notebook](https://nbviewer.jupyter.org/github/gracecarrillo/Coursera_Capstone/blob/master/Exploring_Edinburgh_Graciela_Carrillo.ipynb).  The code should run with no issues using Python versions 3.*.

## Project Motivation<a name="motivation"></a>

 As part of the [IBM Data Science Professional Certificate](https://www.coursera.org/account/accomplishments/specialization/certificate/QWDCRB9GLLYL), we get to have a go at our very own Data Science Capstone, where we get a taste of what is like to solve problems and answer questions like a data scientist. For my assignment, I decided to do yet another project that looks into the relationship between Airbnb prices and its determinants. However, being that I’m all about transportation research, I added a little touch of geospatial analysis by looking into locational features as possible predictors. 

## File Descriptions <a name="files"></a>

I published a [post in Towards Data Science](https://towardsdatascience.com/predicting-airbnb-prices-with-machine-learning-and-location-data-5c1e033d0a5a), a more detailed report can be found [here](https://gracecarrillo.github.io/Coursera_Capstone/#some_useful_references) and the notebook [here](https://nbviewer.jupyter.org/github/gracecarrillo/Coursera_Capstone/blob/master/Exploring_Edinburgh_Graciela_Carrillo.ipynb). 

## Results<a name="results"></a>

After a rather long notebook and playing with a lot of variables, the best performing model was able to predict 65.56% of the variation in price with an RMSE of 0.19. Several other features are not part of our dataset or needed to be analysed more closely. It was noticeable that reviews about listing location, rather than the location features themselves, were higher in the feature importance list. Thus, perhaps Airbnb hosts should highlight this when writing their listing's description. Highlighting accessibility and location benefits of staying with them could perhaps benefit them and how much they can ask for their listing. This also shows the importance of reviews. So, image quality analysis is an interesting path to follow.

## Licensing, Authors, Acknowledgements<a name="licensing"></a>

Acknowledgements [Laura Lewis](https://github.com/L-Lewis/Airbnb-neural-network-price-prediction)'s for examples on data preprocessing and [Geoff Boeing](https://github.com/gboeing/urban-data-science/blob/master/20-Accessibility-Walkability/pandana-accessibility-demo-full.ipynb) for the accessibility analysis. Code to query foursquare for venues through all neighbourhoods was provided during the course.  Feel free to use the code here as you would like! 
