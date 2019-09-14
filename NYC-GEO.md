
# IBM Data Science Capstone Project      
# Opening a Chinese Restaurant in Boston


## I. Introduction    
Boston is the capital and most populous city of the Commonwealth of Massachusetts in the United States, as well as the 21st most populous city in the United States. Boston is one of the oldest municipalities in the United States, founded on the Shawmut Peninsula in 1630 by Puritan settlers from England. Today, Boston is a thriving port city. The Boston area's many colleges and universities make it an international center of higher education, including law, medicine, engineering, and business, and the city is considered to be a world leader in innovation and entrepreneurship, with nearly 2,000 startups.
Along with its long history, food is a quintessential component of New York City. Cuisine in Boston is similar to the rest of New England cuisine, in that it has a large emphasis on seafood and dairy products. Its best-known dishes are New England clam chowder, fish and chips (usually with cod or scrod), baked beans, lobsters, steamed clams, and fried clams.      

The **Union Oyster House** is the oldest operating restaurant in the United States. Their menu includes oysters on the half-shell served straight from an oyster bar, New England clam chowder, and other seafood dishes. Quincy Market, part of Faneuil Hall Marketplace, has a variety of restaurants and food shops. Nearby Cheers is a popular tourist dining spot.         

Boston's **Chinatown** has a variety of Asian restaurants, bakeries, grocery stores, and medicinal herb and spice vendors. In addition to dim sum and other Chinese dining styles, there are Vietnamese, Japanese, Korean and Thai restaurants in the neighborhood.           

The **North End** has a variety of Italian restaurants, pizzerias, and bakeries and is well known as Boston's "Little Italy." A favorite spot bringing in tourists is Mike's Pastry, located on Hanover Street and is extremely popular for its cannolis. Newbury Street has many ethnic street cafes, while Copley Place houses a multitude of restaurants, also the home of Legal Sea Foods, a New England institution that offers gourmet seafood dishes.            

The objective of this project is to locate and recommend which neighborhood of Boston will be best choice to start a Chinese restaurant and explain the rationale of the recommendations.

## II. DATA ACQUISITION

This demonstration will make use of the following data sources:

**Boston Neighborhoods Data**       

Data will retrieved from Boston open dataset from https://data.boston.gov website.

The Neighborhood boundaries data is a combination of zoning neighborhood boundaries, zip code boundaries and 2010 Census tract boundaries.  These boundaries are used in the broad sense for visualization purposes for zoning and planning studies.  

**Boston location data retrieved using Google maps API**          

Data coordinates of Neighborhood Venues will be retrieved using google API. I also make use of subway stations coordinate as a more important center of for all towns included in venue recommendations.

**Boston Top Venue Recommendations from FourSquare API**     

(FourSquare website: www.foursquare.com)

I will be using the FourSquare API to explore neighborhoods in Boston. The Foursquare explore function will be used to get the most common venue categories in each neighborhood, and then use this feature to group the neighborhoods into clusters. The following information are retrieved on the first query:

- Venue ID
- Venue Name
= Coordinates : Latitude and Longitude
- Category Name        

# III. METHODOLOGY

**Load libraries**


```python
import numpy as np # library to handle data in a vectorized manner

import pandas as pd # library for data analsysis
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)

import json # library to handle JSON files

#!conda install -c conda-forge geopy --yes # uncomment this line if you haven't completed the Foursquare API lab
from geopy.geocoders import Nominatim # convert an address into latitude and longitude values

import requests # library to handle requests
from pandas.io.json import json_normalize # tranform JSON file into a pandas dataframe

# Matplotlib and associated plotting modules
import matplotlib.cm as cm
import matplotlib.colors as colors

# import k-means from clustering stage
from sklearn.cluster import KMeans

!conda install -c conda-forge folium=0.5.0 --yes # uncomment this line if you haven't completed the Foursquare API lab
import folium # map rendering library

print('Libraries imported.')
```

    Solving environment: done
    
    ## Package Plan ##
    
      environment location: /opt/conda/envs/Python36
    
      added / updated specs: 
        - folium=0.5.0
    
    
    The following packages will be downloaded:
    
        package                    |            build
        ---------------------------|-----------------
        branca-0.3.1               |             py_0          25 KB  conda-forge
        vincent-0.4.4              |             py_1          28 KB  conda-forge
        folium-0.5.0               |             py_0          45 KB  conda-forge
        altair-3.2.0               |           py36_0         770 KB  conda-forge
        openssl-1.1.1c             |       h516909a_0         2.1 MB  conda-forge
        ca-certificates-2019.9.11  |       hecc5488_0         144 KB  conda-forge
        certifi-2019.9.11          |           py36_0         147 KB  conda-forge
        ------------------------------------------------------------
                                               Total:         3.3 MB
    
    The following NEW packages will be INSTALLED:
    
        altair:          3.2.0-py36_0      conda-forge
        branca:          0.3.1-py_0        conda-forge
        folium:          0.5.0-py_0        conda-forge
        vincent:         0.4.4-py_1        conda-forge
    
    The following packages will be UPDATED:
    
        ca-certificates: 2019.5.15-1                   --> 2019.9.11-hecc5488_0 conda-forge
        certifi:         2019.6.16-py36_1              --> 2019.9.11-py36_0     conda-forge
    
    The following packages will be DOWNGRADED:
    
        openssl:         1.1.1c-h7b6447c_1             --> 1.1.1c-h516909a_0    conda-forge
    
    
    Downloading and Extracting Packages
    branca-0.3.1         | 25 KB     | ##################################### | 100% 
    vincent-0.4.4        | 28 KB     | ##################################### | 100% 
    folium-0.5.0         | 45 KB     | ##################################### | 100% 
    altair-3.2.0         | 770 KB    | ##################################### | 100% 
    openssl-1.1.1c       | 2.1 MB    | ##################################### | 100% 
    ca-certificates-2019 | 144 KB    | ##################################### | 100% 
    certifi-2019.9.11    | 147 KB    | ##################################### | 100% 
    Preparing transaction: done
    Verifying transaction: done
    Executing transaction: done
    Libraries imported.
    

## 1. Download and Explore Dataset

### Download the Boston Neighborhood data from https://data.boston.gov


```python
!wget -q -O 'boston_data.csv' http://bostonopendata-boston.opendata.arcgis.com/datasets/3525b0ee6e6b427f9aab5d0a1d0a1a28_0.csv
print('Data downloaded!')
```

    Data downloaded!
    

### Load and explore the data


```python
df = pd.read_csv('boston_data.csv')
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OBJECTID</th>
      <th>Name</th>
      <th>Acres</th>
      <th>Neighborhood_ID</th>
      <th>SqMiles</th>
      <th>ShapeSTArea</th>
      <th>ShapeSTLength</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>27</td>
      <td>Roslindale</td>
      <td>1605.568237</td>
      <td>15</td>
      <td>2.51</td>
      <td>6.993827e+07</td>
      <td>53563.912597</td>
    </tr>
    <tr>
      <th>1</th>
      <td>28</td>
      <td>Jamaica Plain</td>
      <td>2519.245394</td>
      <td>11</td>
      <td>3.94</td>
      <td>1.097379e+08</td>
      <td>56349.937161</td>
    </tr>
    <tr>
      <th>2</th>
      <td>29</td>
      <td>Mission Hill</td>
      <td>350.853564</td>
      <td>13</td>
      <td>0.55</td>
      <td>1.528312e+07</td>
      <td>17918.724113</td>
    </tr>
    <tr>
      <th>3</th>
      <td>30</td>
      <td>Longwood</td>
      <td>188.611947</td>
      <td>28</td>
      <td>0.29</td>
      <td>8.215904e+06</td>
      <td>11908.757148</td>
    </tr>
    <tr>
      <th>4</th>
      <td>31</td>
      <td>Bay Village</td>
      <td>26.539839</td>
      <td>33</td>
      <td>0.04</td>
      <td>1.156071e+06</td>
      <td>4650.635493</td>
    </tr>
    <tr>
      <th>5</th>
      <td>32</td>
      <td>Leather District</td>
      <td>15.639908</td>
      <td>27</td>
      <td>0.02</td>
      <td>6.812717e+05</td>
      <td>3237.140537</td>
    </tr>
    <tr>
      <th>6</th>
      <td>33</td>
      <td>Chinatown</td>
      <td>76.324410</td>
      <td>26</td>
      <td>0.12</td>
      <td>3.324678e+06</td>
      <td>9736.590413</td>
    </tr>
    <tr>
      <th>7</th>
      <td>34</td>
      <td>North End</td>
      <td>126.910439</td>
      <td>14</td>
      <td>0.20</td>
      <td>5.527506e+06</td>
      <td>16177.826815</td>
    </tr>
    <tr>
      <th>8</th>
      <td>35</td>
      <td>Roxbury</td>
      <td>2108.469072</td>
      <td>16</td>
      <td>3.29</td>
      <td>9.184455e+07</td>
      <td>49488.800485</td>
    </tr>
    <tr>
      <th>9</th>
      <td>36</td>
      <td>South End</td>
      <td>471.535356</td>
      <td>32</td>
      <td>0.74</td>
      <td>2.054000e+07</td>
      <td>17912.333569</td>
    </tr>
    <tr>
      <th>10</th>
      <td>37</td>
      <td>Back Bay</td>
      <td>399.314411</td>
      <td>2</td>
      <td>0.62</td>
      <td>1.739407e+07</td>
      <td>19455.671146</td>
    </tr>
    <tr>
      <th>11</th>
      <td>38</td>
      <td>East Boston</td>
      <td>3012.059593</td>
      <td>8</td>
      <td>4.71</td>
      <td>1.313845e+08</td>
      <td>121089.100852</td>
    </tr>
    <tr>
      <th>12</th>
      <td>39</td>
      <td>Charlestown</td>
      <td>871.541223</td>
      <td>4</td>
      <td>1.36</td>
      <td>3.796418e+07</td>
      <td>57509.688645</td>
    </tr>
    <tr>
      <th>13</th>
      <td>40</td>
      <td>West End</td>
      <td>190.490732</td>
      <td>31</td>
      <td>0.30</td>
      <td>8.297743e+06</td>
      <td>17728.590027</td>
    </tr>
    <tr>
      <th>14</th>
      <td>41</td>
      <td>Beacon Hill</td>
      <td>200.156904</td>
      <td>30</td>
      <td>0.31</td>
      <td>8.718800e+06</td>
      <td>14303.829017</td>
    </tr>
    <tr>
      <th>15</th>
      <td>42</td>
      <td>Downtown</td>
      <td>397.472846</td>
      <td>7</td>
      <td>0.62</td>
      <td>1.731385e+07</td>
      <td>34612.804441</td>
    </tr>
    <tr>
      <th>16</th>
      <td>43</td>
      <td>Fenway</td>
      <td>560.618461</td>
      <td>34</td>
      <td>0.88</td>
      <td>2.442044e+07</td>
      <td>24620.876452</td>
    </tr>
    <tr>
      <th>17</th>
      <td>44</td>
      <td>Brighton</td>
      <td>1840.408596</td>
      <td>25</td>
      <td>2.88</td>
      <td>8.016788e+07</td>
      <td>48787.519652</td>
    </tr>
    <tr>
      <th>18</th>
      <td>45</td>
      <td>West Roxbury</td>
      <td>3516.421786</td>
      <td>19</td>
      <td>5.49</td>
      <td>1.531747e+08</td>
      <td>66067.419838</td>
    </tr>
    <tr>
      <th>19</th>
      <td>46</td>
      <td>Hyde Park</td>
      <td>2927.221168</td>
      <td>10</td>
      <td>4.57</td>
      <td>1.275092e+08</td>
      <td>66861.244955</td>
    </tr>
    <tr>
      <th>20</th>
      <td>47</td>
      <td>Mattapan</td>
      <td>1352.098354</td>
      <td>12</td>
      <td>2.11</td>
      <td>5.889717e+07</td>
      <td>42005.773707</td>
    </tr>
    <tr>
      <th>21</th>
      <td>48</td>
      <td>Dorchester</td>
      <td>4662.879457</td>
      <td>6</td>
      <td>7.29</td>
      <td>2.031142e+08</td>
      <td>104344.034005</td>
    </tr>
    <tr>
      <th>22</th>
      <td>49</td>
      <td>South Boston Waterfront</td>
      <td>621.843524</td>
      <td>29</td>
      <td>0.97</td>
      <td>2.708740e+07</td>
      <td>38391.352905</td>
    </tr>
    <tr>
      <th>23</th>
      <td>50</td>
      <td>South Boston</td>
      <td>1439.888807</td>
      <td>17</td>
      <td>2.25</td>
      <td>6.272131e+07</td>
      <td>64998.420283</td>
    </tr>
    <tr>
      <th>24</th>
      <td>51</td>
      <td>Allston</td>
      <td>998.534479</td>
      <td>24</td>
      <td>1.56</td>
      <td>4.349599e+07</td>
      <td>37859.091242</td>
    </tr>
    <tr>
      <th>25</th>
      <td>52</td>
      <td>Harbor Islands</td>
      <td>824.888658</td>
      <td>22</td>
      <td>1.29</td>
      <td>3.593201e+07</td>
      <td>92482.183568</td>
    </tr>
  </tbody>
</table>
</div>




```python
df['Latitude'] = 0.0
df['Longitude'] = 0.0

for idx,town in df['Name'].iteritems():
    address = town + " subway station, Boston" ; # I use subway stations as more important central location of each neighborhood
    url = 'https://maps.googleapis.com/maps/api/geocode/json?address={}&key={}'.format(address,google_key)
    lat = requests.get(url).json()["results"][0]["geometry"]["location"]['lat']
    lng = requests.get(url).json()["results"][0]["geometry"]["location"]['lng']
    df.loc[idx,'Latitude'] = lat
    df.loc[idx,'Longitude'] = lng
```


```python
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OBJECTID</th>
      <th>Name</th>
      <th>Acres</th>
      <th>Neighborhood_ID</th>
      <th>SqMiles</th>
      <th>ShapeSTArea</th>
      <th>ShapeSTLength</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>27</td>
      <td>Roslindale</td>
      <td>1605.568237</td>
      <td>15</td>
      <td>2.51</td>
      <td>6.993827e+07</td>
      <td>53563.912597</td>
      <td>42.300690</td>
      <td>-71.113972</td>
    </tr>
    <tr>
      <th>1</th>
      <td>28</td>
      <td>Jamaica Plain</td>
      <td>2519.245394</td>
      <td>11</td>
      <td>3.94</td>
      <td>1.097379e+08</td>
      <td>56349.937161</td>
      <td>42.317265</td>
      <td>-71.104160</td>
    </tr>
    <tr>
      <th>2</th>
      <td>29</td>
      <td>Mission Hill</td>
      <td>350.853564</td>
      <td>13</td>
      <td>0.55</td>
      <td>1.528312e+07</td>
      <td>17918.724113</td>
      <td>42.331341</td>
      <td>-71.095499</td>
    </tr>
    <tr>
      <th>3</th>
      <td>30</td>
      <td>Longwood</td>
      <td>188.611947</td>
      <td>28</td>
      <td>0.29</td>
      <td>8.215904e+06</td>
      <td>11908.757148</td>
      <td>42.336046</td>
      <td>-71.099727</td>
    </tr>
    <tr>
      <th>4</th>
      <td>31</td>
      <td>Bay Village</td>
      <td>26.539839</td>
      <td>33</td>
      <td>0.04</td>
      <td>1.156071e+06</td>
      <td>4650.635493</td>
      <td>42.347350</td>
      <td>-71.075727</td>
    </tr>
    <tr>
      <th>5</th>
      <td>32</td>
      <td>Leather District</td>
      <td>15.639908</td>
      <td>27</td>
      <td>0.02</td>
      <td>6.812717e+05</td>
      <td>3237.140537</td>
      <td>42.351922</td>
      <td>-71.055070</td>
    </tr>
    <tr>
      <th>6</th>
      <td>33</td>
      <td>Chinatown</td>
      <td>76.324410</td>
      <td>26</td>
      <td>0.12</td>
      <td>3.324678e+06</td>
      <td>9736.590413</td>
      <td>42.352392</td>
      <td>-71.062573</td>
    </tr>
    <tr>
      <th>7</th>
      <td>34</td>
      <td>North End</td>
      <td>126.910439</td>
      <td>14</td>
      <td>0.20</td>
      <td>5.527506e+06</td>
      <td>16177.826815</td>
      <td>42.366352</td>
      <td>-71.062150</td>
    </tr>
    <tr>
      <th>8</th>
      <td>35</td>
      <td>Roxbury</td>
      <td>2108.469072</td>
      <td>16</td>
      <td>3.29</td>
      <td>9.184455e+07</td>
      <td>49488.800485</td>
      <td>42.331341</td>
      <td>-71.095499</td>
    </tr>
    <tr>
      <th>9</th>
      <td>36</td>
      <td>South End</td>
      <td>471.535356</td>
      <td>32</td>
      <td>0.74</td>
      <td>2.054000e+07</td>
      <td>17912.333569</td>
      <td>42.347350</td>
      <td>-71.075727</td>
    </tr>
    <tr>
      <th>10</th>
      <td>37</td>
      <td>Back Bay</td>
      <td>399.314411</td>
      <td>2</td>
      <td>0.62</td>
      <td>1.739407e+07</td>
      <td>19455.671146</td>
      <td>42.347350</td>
      <td>-71.075727</td>
    </tr>
    <tr>
      <th>11</th>
      <td>38</td>
      <td>East Boston</td>
      <td>3012.059593</td>
      <td>8</td>
      <td>4.71</td>
      <td>1.313845e+08</td>
      <td>121089.100852</td>
      <td>42.390501</td>
      <td>-70.997123</td>
    </tr>
    <tr>
      <th>12</th>
      <td>39</td>
      <td>Charlestown</td>
      <td>871.541223</td>
      <td>4</td>
      <td>1.36</td>
      <td>3.796418e+07</td>
      <td>57509.688645</td>
      <td>42.373678</td>
      <td>-71.069654</td>
    </tr>
    <tr>
      <th>13</th>
      <td>40</td>
      <td>West End</td>
      <td>190.490732</td>
      <td>31</td>
      <td>0.30</td>
      <td>8.297743e+06</td>
      <td>17728.590027</td>
      <td>42.366817</td>
      <td>-71.067777</td>
    </tr>
    <tr>
      <th>14</th>
      <td>41</td>
      <td>Beacon Hill</td>
      <td>200.156904</td>
      <td>30</td>
      <td>0.31</td>
      <td>8.718800e+06</td>
      <td>14303.829017</td>
      <td>42.355820</td>
      <td>-71.062826</td>
    </tr>
    <tr>
      <th>15</th>
      <td>42</td>
      <td>Downtown</td>
      <td>397.472846</td>
      <td>7</td>
      <td>0.62</td>
      <td>1.731385e+07</td>
      <td>34612.804441</td>
      <td>42.355453</td>
      <td>-71.060453</td>
    </tr>
    <tr>
      <th>16</th>
      <td>43</td>
      <td>Fenway</td>
      <td>560.618461</td>
      <td>34</td>
      <td>0.88</td>
      <td>2.442044e+07</td>
      <td>24620.876452</td>
      <td>42.345399</td>
      <td>-71.104332</td>
    </tr>
    <tr>
      <th>17</th>
      <td>44</td>
      <td>Brighton</td>
      <td>1840.408596</td>
      <td>25</td>
      <td>2.88</td>
      <td>8.016788e+07</td>
      <td>48787.519652</td>
      <td>42.348688</td>
      <td>-71.138024</td>
    </tr>
    <tr>
      <th>18</th>
      <td>45</td>
      <td>West Roxbury</td>
      <td>3516.421786</td>
      <td>19</td>
      <td>5.49</td>
      <td>1.531747e+08</td>
      <td>66067.419838</td>
      <td>42.286640</td>
      <td>-71.145578</td>
    </tr>
    <tr>
      <th>19</th>
      <td>46</td>
      <td>Hyde Park</td>
      <td>2927.221168</td>
      <td>10</td>
      <td>4.57</td>
      <td>1.275092e+08</td>
      <td>66861.244955</td>
      <td>42.254750</td>
      <td>-71.125585</td>
    </tr>
    <tr>
      <th>20</th>
      <td>47</td>
      <td>Mattapan</td>
      <td>1352.098354</td>
      <td>12</td>
      <td>2.11</td>
      <td>5.889717e+07</td>
      <td>42005.773707</td>
      <td>42.267784</td>
      <td>-71.091829</td>
    </tr>
    <tr>
      <th>21</th>
      <td>48</td>
      <td>Dorchester</td>
      <td>4662.879457</td>
      <td>6</td>
      <td>7.29</td>
      <td>2.031142e+08</td>
      <td>104344.034005</td>
      <td>42.293128</td>
      <td>-71.065803</td>
    </tr>
    <tr>
      <th>22</th>
      <td>49</td>
      <td>South Boston Waterfront</td>
      <td>621.843524</td>
      <td>29</td>
      <td>0.97</td>
      <td>2.708740e+07</td>
      <td>38391.352905</td>
      <td>42.351922</td>
      <td>-71.055070</td>
    </tr>
    <tr>
      <th>23</th>
      <td>50</td>
      <td>South Boston</td>
      <td>1439.888807</td>
      <td>17</td>
      <td>2.25</td>
      <td>6.272131e+07</td>
      <td>64998.420283</td>
      <td>42.351922</td>
      <td>-71.055070</td>
    </tr>
    <tr>
      <th>24</th>
      <td>51</td>
      <td>Allston</td>
      <td>998.534479</td>
      <td>24</td>
      <td>1.56</td>
      <td>4.349599e+07</td>
      <td>37859.091242</td>
      <td>42.348688</td>
      <td>-71.138024</td>
    </tr>
    <tr>
      <th>25</th>
      <td>52</td>
      <td>Harbor Islands</td>
      <td>824.888658</td>
      <td>22</td>
      <td>1.29</td>
      <td>3.593201e+07</td>
      <td>92482.183568</td>
      <td>42.390501</td>
      <td>-70.997123</td>
    </tr>
  </tbody>
</table>
</div>



### Generate Boston basemap


```python
geo = Nominatim(user_agent='My-IBMNotebook')
address = 'Boston'
location = geo.geocode(address)
latitude = location.latitude
longitude = location.longitude
print('The geograpical coordinate of Boston {}, {}.'.format(latitude, longitude))

# create map of Boston using latitude and longitude values
map_boston = folium.Map(location=[latitude, longitude],tiles="OpenStreetMap", zoom_start=10)

# add markers to map
for lat, lng, town in zip(
    df['Latitude'],
    df['Longitude'],
    df['Name']):
    label = town
    label = folium.Popup(label, parse_html=True)
    folium.CircleMarker(
        [lat, lng],
        radius=4,
        popup=label,
        color='blue',
        fill=True,
        fill_color='#87cefa',
        fill_opacity=0.5,
        parse_html=False).add_to(map_boston)
map_boston
```

    The geograpical coordinate of Boston 42.3602534, -71.0582912.
    




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzhmMWUwZTgyMzhmMzQ4YWRhYTRhMGE1MzI3YTgzNDhhIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGJvdW5kcyA9IG51bGw7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgdmFyIG1hcF84ZjFlMGU4MjM4ZjM0OGFkYWE0YTBhNTMyN2E4MzQ4YSA9IEwubWFwKAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgJ21hcF84ZjFlMGU4MjM4ZjM0OGFkYWE0YTBhNTMyN2E4MzQ4YScsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB7Y2VudGVyOiBbNDIuMzYwMjUzNCwtNzEuMDU4MjkxMl0sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB6b29tOiAxMCwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1heEJvdW5kczogYm91bmRzLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbGF5ZXJzOiBbXSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHdvcmxkQ29weUp1bXA6IGZhbHNlLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NwogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB9KTsKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfYjExYjI1OTU0MTVjNDQxMTlkZDI4YWI1MzQxODcxMmUgPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICdodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZycsCiAgICAgICAgICAgICAgICB7CiAgImF0dHJpYnV0aW9uIjogbnVsbCwKICAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsCiAgIm1heFpvb20iOiAxOCwKICAibWluWm9vbSI6IDEsCiAgIm5vV3JhcCI6IGZhbHNlLAogICJzdWJkb21haW5zIjogImFiYyIKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzNjNDk4OGUzMDExNDQ3NzdhNDE4NDM0NWQ0YjcwYTRkID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDIuMzAwNjkwNDk5OTk5OTksLTcxLjExMzk3MjQwMDAwMDAxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODdjZWZhIiwKICAiZmlsbE9wYWNpdHkiOiAwLjUsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA0LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzhmMWUwZTgyMzhmMzQ4YWRhYTRhMGE1MzI3YTgzNDhhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2RkZDkzNjk0MGU0YTQ4Y2U4MjRjYjQzMmQ4OTU5NTdkID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzc3N2IyNjA0NTRlNjRiZTRhYjk3NjhkYzg1NmRmZDM0ID0gJCgnPGRpdiBpZD0iaHRtbF83NzdiMjYwNDU0ZTY0YmU0YWI5NzY4ZGM4NTZkZmQzNCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Um9zbGluZGFsZTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZGRkOTM2OTQwZTRhNDhjZTgyNGNiNDMyZDg5NTk1N2Quc2V0Q29udGVudChodG1sXzc3N2IyNjA0NTRlNjRiZTRhYjk3NjhkYzg1NmRmZDM0KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzNjNDk4OGUzMDExNDQ3NzdhNDE4NDM0NWQ0YjcwYTRkLmJpbmRQb3B1cChwb3B1cF9kZGQ5MzY5NDBlNGE0OGNlODI0Y2I0MzJkODk1OTU3ZCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84MmUzYWVmMzA3Njk0MWM2YmIxMzY1YzE3NWE4OTFhOSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjMxNzI2NTQsLTcxLjEwNDE2MDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4N2NlZmEiLAogICJmaWxsT3BhY2l0eSI6IDAuNSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDQsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMjg3MjMyODBjNmFlNDM4NmJkYmQwNzI0M2YwNDk4YTYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMWUwODQwZWYwNWU4NDU3Y2E4OGIxMzkzYzc2OTVhYzIgPSAkKCc8ZGl2IGlkPSJodG1sXzFlMDg0MGVmMDVlODQ1N2NhODhiMTM5M2M3Njk1YWMyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5KYW1haWNhIFBsYWluPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8yODcyMzI4MGM2YWU0Mzg2YmRiZDA3MjQzZjA0OThhNi5zZXRDb250ZW50KGh0bWxfMWUwODQwZWYwNWU4NDU3Y2E4OGIxMzkzYzc2OTVhYzIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfODJlM2FlZjMwNzY5NDFjNmJiMTM2NWMxNzVhODkxYTkuYmluZFBvcHVwKHBvcHVwXzI4NzIzMjgwYzZhZTQzODZiZGJkMDcyNDNmMDQ5OGE2KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2Y4MjQ0YjI3ZDk2YjQwNWJhZGUyOTQ1NTdkZmRkODg0ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDIuMzMxMzQwNiwtNzEuMDk1NDk5MV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzg3Y2VmYSIsCiAgImZpbGxPcGFjaXR5IjogMC41LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84ZjFlMGU4MjM4ZjM0OGFkYWE0YTBhNTMyN2E4MzQ4YSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81ZTkwMmYyNzY3M2Q0NTAyOGUwZmYxZTExZmZmMmI4ZiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zMjEyNmYzY2JkZTA0MDc2ODRkN2I0ZDNiOGE4ODAyNiA9ICQoJzxkaXYgaWQ9Imh0bWxfMzIxMjZmM2NiZGUwNDA3Njg0ZDdiNGQzYjhhODgwMjYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk1pc3Npb24gSGlsbDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNWU5MDJmMjc2NzNkNDUwMjhlMGZmMWUxMWZmZjJiOGYuc2V0Q29udGVudChodG1sXzMyMTI2ZjNjYmRlMDQwNzY4NGQ3YjRkM2I4YTg4MDI2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2Y4MjQ0YjI3ZDk2YjQwNWJhZGUyOTQ1NTdkZmRkODg0LmJpbmRQb3B1cChwb3B1cF81ZTkwMmYyNzY3M2Q0NTAyOGUwZmYxZTExZmZmMmI4Zik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84MWEzMTY4OGRjZGU0NmJhOWYyNzQ1NzY3ZDA4ZDE4ZiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjMzNjA0NTYsLTcxLjA5OTcyNjZdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4N2NlZmEiLAogICJmaWxsT3BhY2l0eSI6IDAuNSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDQsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYzBjYjI5ZDU4ZGUyNGQ4NzhhOGEzNjQ4M2U2NGZjNDAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZDI3ZTdiZWU1NzhiNGVkZWI3Yzg0NjRkM2JmZjc0MzMgPSAkKCc8ZGl2IGlkPSJodG1sX2QyN2U3YmVlNTc4YjRlZGViN2M4NDY0ZDNiZmY3NDMzIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Mb25nd29vZDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYzBjYjI5ZDU4ZGUyNGQ4NzhhOGEzNjQ4M2U2NGZjNDAuc2V0Q29udGVudChodG1sX2QyN2U3YmVlNTc4YjRlZGViN2M4NDY0ZDNiZmY3NDMzKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzgxYTMxNjg4ZGNkZTQ2YmE5ZjI3NDU3NjdkMDhkMThmLmJpbmRQb3B1cChwb3B1cF9jMGNiMjlkNThkZTI0ZDg3OGE4YTM2NDgzZTY0ZmM0MCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9hNGFmNWI3MDQyMWM0NjU2OGRkMzA3NmZiMmQ3MzVmOSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM0NzM1LC03MS4wNzU3MjddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4N2NlZmEiLAogICJmaWxsT3BhY2l0eSI6IDAuNSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDQsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYTk4ZGY2ZDc3NDZiNDk3ZWIyNzBiZTljMzI3ZGUzZWYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfOTA5OTM0NGEwNGZhNGRmOGI2Y2ZmMWM3ZDUyYjM5OTEgPSAkKCc8ZGl2IGlkPSJodG1sXzkwOTkzNDRhMDRmYTRkZjhiNmNmZjFjN2Q1MmIzOTkxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CYXkgVmlsbGFnZTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYTk4ZGY2ZDc3NDZiNDk3ZWIyNzBiZTljMzI3ZGUzZWYuc2V0Q29udGVudChodG1sXzkwOTkzNDRhMDRmYTRkZjhiNmNmZjFjN2Q1MmIzOTkxKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2E0YWY1YjcwNDIxYzQ2NTY4ZGQzMDc2ZmIyZDczNWY5LmJpbmRQb3B1cChwb3B1cF9hOThkZjZkNzc0NmI0OTdlYjI3MGJlOWMzMjdkZTNlZik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81MjAwZjUwNDhhNjI0ZGM5OGUxM2FmZWRhZTBhNDUyYiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM1MTkyMTcsLTcxLjA1NTA3MDNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4N2NlZmEiLAogICJmaWxsT3BhY2l0eSI6IDAuNSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDQsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYjJmNjc2MGE1NjYxNDkxNjgzZmFjNDc3ZjM2ODgyZTQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNWFmMDFkOWMzNWM3NDlkMmIxNTI0ODI0ZjM2OGMzYWUgPSAkKCc8ZGl2IGlkPSJodG1sXzVhZjAxZDljMzVjNzQ5ZDJiMTUyNDgyNGYzNjhjM2FlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5MZWF0aGVyIERpc3RyaWN0PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9iMmY2NzYwYTU2NjE0OTE2ODNmYWM0NzdmMzY4ODJlNC5zZXRDb250ZW50KGh0bWxfNWFmMDFkOWMzNWM3NDlkMmIxNTI0ODI0ZjM2OGMzYWUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNTIwMGY1MDQ4YTYyNGRjOThlMTNhZmVkYWUwYTQ1MmIuYmluZFBvcHVwKHBvcHVwX2IyZjY3NjBhNTY2MTQ5MTY4M2ZhYzQ3N2YzNjg4MmU0KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2RhMGVhNTExZWFlNjQwMDRhNWYyNjAxZWE2MWM0MWVjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDIuMzUyMzkyMSwtNzEuMDYyNTcyNl0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzg3Y2VmYSIsCiAgImZpbGxPcGFjaXR5IjogMC41LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84ZjFlMGU4MjM4ZjM0OGFkYWE0YTBhNTMyN2E4MzQ4YSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xNzQxYzlkYjk2MTI0Yzg0YjYzN2NkOGZhNjRmYTYyNyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9lMjczZWQ3NmRjMDQ0ZTc2YmIyZDY3MDk2ZjkxNGUwZiA9ICQoJzxkaXYgaWQ9Imh0bWxfZTI3M2VkNzZkYzA0NGU3NmJiMmQ2NzA5NmY5MTRlMGYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNoaW5hdG93bjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMTc0MWM5ZGI5NjEyNGM4NGI2MzdjZDhmYTY0ZmE2Mjcuc2V0Q29udGVudChodG1sX2UyNzNlZDc2ZGMwNDRlNzZiYjJkNjcwOTZmOTE0ZTBmKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2RhMGVhNTExZWFlNjQwMDRhNWYyNjAxZWE2MWM0MWVjLmJpbmRQb3B1cChwb3B1cF8xNzQxYzlkYjk2MTI0Yzg0YjYzN2NkOGZhNjRmYTYyNyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83MjlkNDhmYTVmNGE0MjgzOTYyYTVmYWJmNzM4Yjk2MSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM2NjM1MTcsLTcxLjA2MjE1MDRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4N2NlZmEiLAogICJmaWxsT3BhY2l0eSI6IDAuNSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDQsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMjRiNTlmZjBiZWVhNDczNWIxZDNjZjZhNWE2YzcxYzIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNzc1YjNmMDlkZGY1NGI1MmE2NzQ3YTQ0ZmQzZjdhOWMgPSAkKCc8ZGl2IGlkPSJodG1sXzc3NWIzZjA5ZGRmNTRiNTJhNjc0N2E0NGZkM2Y3YTljIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ob3J0aCBFbmQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzI0YjU5ZmYwYmVlYTQ3MzViMWQzY2Y2YTVhNmM3MWMyLnNldENvbnRlbnQoaHRtbF83NzViM2YwOWRkZjU0YjUyYTY3NDdhNDRmZDNmN2E5Yyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl83MjlkNDhmYTVmNGE0MjgzOTYyYTVmYWJmNzM4Yjk2MS5iaW5kUG9wdXAocG9wdXBfMjRiNTlmZjBiZWVhNDczNWIxZDNjZjZhNWE2YzcxYzIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNDBhMmFkNzBiYmIyNGEwMDk2NmM3NGJmZTI4ZTk4YmEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zMzEzNDA2LC03MS4wOTU0OTkxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODdjZWZhIiwKICAiZmlsbE9wYWNpdHkiOiAwLjUsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA0LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzhmMWUwZTgyMzhmMzQ4YWRhYTRhMGE1MzI3YTgzNDhhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2EyOTgwMDUyMzNjMjQ2YTBiZGQyYmNiZTNjOWNjZjcxID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2JkNTMzNjAxMGRmNDRkZGNhYTkzYzYxN2M2YmJkOTViID0gJCgnPGRpdiBpZD0iaHRtbF9iZDUzMzYwMTBkZjQ0ZGRjYWE5M2M2MTdjNmJiZDk1YiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Um94YnVyeTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYTI5ODAwNTIzM2MyNDZhMGJkZDJiY2JlM2M5Y2NmNzEuc2V0Q29udGVudChodG1sX2JkNTMzNjAxMGRmNDRkZGNhYTkzYzYxN2M2YmJkOTViKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzQwYTJhZDcwYmJiMjRhMDA5NjZjNzRiZmUyOGU5OGJhLmJpbmRQb3B1cChwb3B1cF9hMjk4MDA1MjMzYzI0NmEwYmRkMmJjYmUzYzljY2Y3MSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84ODZmYmNjMTlmYmY0ZTkzYjRmMDY5NzBmYmExZGUxNSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM0NzM1LC03MS4wNzU3MjddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4N2NlZmEiLAogICJmaWxsT3BhY2l0eSI6IDAuNSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDQsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfODM5NWUwMmEwOGQ4NGIwNTllMzQ5NGNiNGQxNjkyNTMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMGU4ZjJjNTU5MWVhNDI2OWI5Zjc3MTcwZWE3ZmU2ZWMgPSAkKCc8ZGl2IGlkPSJodG1sXzBlOGYyYzU1OTFlYTQyNjliOWY3NzE3MGVhN2ZlNmVjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Tb3V0aCBFbmQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzgzOTVlMDJhMDhkODRiMDU5ZTM0OTRjYjRkMTY5MjUzLnNldENvbnRlbnQoaHRtbF8wZThmMmM1NTkxZWE0MjY5YjlmNzcxNzBlYTdmZTZlYyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl84ODZmYmNjMTlmYmY0ZTkzYjRmMDY5NzBmYmExZGUxNS5iaW5kUG9wdXAocG9wdXBfODM5NWUwMmEwOGQ4NGIwNTllMzQ5NGNiNGQxNjkyNTMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMWRmMWJjOTU1NDEzNDk2N2E2MTUyNWMzNzViNjkxYTcgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zNDczNSwtNzEuMDc1NzI3XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODdjZWZhIiwKICAiZmlsbE9wYWNpdHkiOiAwLjUsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA0LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzhmMWUwZTgyMzhmMzQ4YWRhYTRhMGE1MzI3YTgzNDhhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzc0OGU2Y2QyYzE1NTQ0M2Y5YzJlNTQ5Y2IyODUzZTVlID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzVjYThlMzRkYWUwZjRmMWViMzA2NzExMGZmMmNlNTM1ID0gJCgnPGRpdiBpZD0iaHRtbF81Y2E4ZTM0ZGFlMGY0ZjFlYjMwNjcxMTBmZjJjZTUzNSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QmFjayBCYXk8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzc0OGU2Y2QyYzE1NTQ0M2Y5YzJlNTQ5Y2IyODUzZTVlLnNldENvbnRlbnQoaHRtbF81Y2E4ZTM0ZGFlMGY0ZjFlYjMwNjcxMTBmZjJjZTUzNSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8xZGYxYmM5NTU0MTM0OTY3YTYxNTI1YzM3NWI2OTFhNy5iaW5kUG9wdXAocG9wdXBfNzQ4ZTZjZDJjMTU1NDQzZjljMmU1NDljYjI4NTNlNWUpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMmYyNWNlZGI5ZjEyNDdjODljOGEyYjYwYTRiNGJjYTIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zOTA1MDEsLTcwLjk5NzEyM10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzg3Y2VmYSIsCiAgImZpbGxPcGFjaXR5IjogMC41LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84ZjFlMGU4MjM4ZjM0OGFkYWE0YTBhNTMyN2E4MzQ4YSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9kNjM2OGY2M2NkZTk0NmM2YTY0YWU2NGRjNGRlZDY5OSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83MzQ1NmRiYTEyN2U0YTE5OTc3Yjg3OTgyYTlmZmVlZiA9ICQoJzxkaXYgaWQ9Imh0bWxfNzM0NTZkYmExMjdlNGExOTk3N2I4Nzk4MmE5ZmZlZWYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkVhc3QgQm9zdG9uPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9kNjM2OGY2M2NkZTk0NmM2YTY0YWU2NGRjNGRlZDY5OS5zZXRDb250ZW50KGh0bWxfNzM0NTZkYmExMjdlNGExOTk3N2I4Nzk4MmE5ZmZlZWYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMmYyNWNlZGI5ZjEyNDdjODljOGEyYjYwYTRiNGJjYTIuYmluZFBvcHVwKHBvcHVwX2Q2MzY4ZjYzY2RlOTQ2YzZhNjRhZTY0ZGM0ZGVkNjk5KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2VlNDFkMWJkMDQ4MzQ1YzQ5Njc5MGQ5ZDNiYmU1OGI3ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDIuMzczNjc4Mjk5OTk5OTksLTcxLjA2OTY1MzVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4N2NlZmEiLAogICJmaWxsT3BhY2l0eSI6IDAuNSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDQsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNWIyMjE3ZWFjYThlNDI4OGJlN2Q2ZWIwODQ0ZGY4ODggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjkwNjJkM2ZiNDI3NGIxYmI0ZjIyMjEwYjk3OTcyMzcgPSAkKCc8ZGl2IGlkPSJodG1sX2Y5MDYyZDNmYjQyNzRiMWJiNGYyMjIxMGI5Nzk3MjM3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaGFybGVzdG93bjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNWIyMjE3ZWFjYThlNDI4OGJlN2Q2ZWIwODQ0ZGY4ODguc2V0Q29udGVudChodG1sX2Y5MDYyZDNmYjQyNzRiMWJiNGYyMjIxMGI5Nzk3MjM3KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2VlNDFkMWJkMDQ4MzQ1YzQ5Njc5MGQ5ZDNiYmU1OGI3LmJpbmRQb3B1cChwb3B1cF81YjIyMTdlYWNhOGU0Mjg4YmU3ZDZlYjA4NDRkZjg4OCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kMzQ4MzBjMjk5Yzg0ODQ5YjlhZjM1YWE4MDdmOWY2OCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM2NjgxNzIsLTcxLjA2Nzc3NjldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4N2NlZmEiLAogICJmaWxsT3BhY2l0eSI6IDAuNSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDQsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZmY5YzNkNDFmN2Y4NDY5Y2I5Mzg0MzE2Yjk2ODhlYmYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNzE4NjBiNTg0NDYxNDMzZWI2MDhiMGJmNjk0OGQ2MjMgPSAkKCc8ZGl2IGlkPSJodG1sXzcxODYwYjU4NDQ2MTQzM2ViNjA4YjBiZjY5NDhkNjIzIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5XZXN0IEVuZDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZmY5YzNkNDFmN2Y4NDY5Y2I5Mzg0MzE2Yjk2ODhlYmYuc2V0Q29udGVudChodG1sXzcxODYwYjU4NDQ2MTQzM2ViNjA4YjBiZjY5NDhkNjIzKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2QzNDgzMGMyOTljODQ4NDliOWFmMzVhYTgwN2Y5ZjY4LmJpbmRQb3B1cChwb3B1cF9mZjljM2Q0MWY3Zjg0NjljYjkzODQzMTZiOTY4OGViZik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kZDgwYWU1YTU2MjI0MDkyYjk1OWIzZTUxOTI2N2ZhNCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM1NTgxOTksLTcxLjA2MjgyNTY5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODdjZWZhIiwKICAiZmlsbE9wYWNpdHkiOiAwLjUsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA0LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzhmMWUwZTgyMzhmMzQ4YWRhYTRhMGE1MzI3YTgzNDhhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzBiZWE4MDllMDQ2MjQxM2Q5OTA3NDgwY2IzMWQ2ODExID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzdmMTlhMTRhMTQxNzQyZjM4NGJiNWIwNDUyOTQ0ZTlkID0gJCgnPGRpdiBpZD0iaHRtbF83ZjE5YTE0YTE0MTc0MmYzODRiYjViMDQ1Mjk0NGU5ZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QmVhY29uIEhpbGw8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzBiZWE4MDllMDQ2MjQxM2Q5OTA3NDgwY2IzMWQ2ODExLnNldENvbnRlbnQoaHRtbF83ZjE5YTE0YTE0MTc0MmYzODRiYjViMDQ1Mjk0NGU5ZCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kZDgwYWU1YTU2MjI0MDkyYjk1OWIzZTUxOTI2N2ZhNC5iaW5kUG9wdXAocG9wdXBfMGJlYTgwOWUwNDYyNDEzZDk5MDc0ODBjYjMxZDY4MTEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYzYyODgxZGNlMzZjNGFhZGIzZGQwYmQwZWQ0NTcyODIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zNTU0NTMsLTcxLjA2MDQ1MzI5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODdjZWZhIiwKICAiZmlsbE9wYWNpdHkiOiAwLjUsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA0LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzhmMWUwZTgyMzhmMzQ4YWRhYTRhMGE1MzI3YTgzNDhhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzQ3MzhhOTMwY2U5NDRhNGI5MGUxZDhiYzI5ZDQ2M2M0ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2Q1ODE4YzVmYzE4NjQyNDE5OGRkOTgyZWUxMjE1MWFhID0gJCgnPGRpdiBpZD0iaHRtbF9kNTgxOGM1ZmMxODY0MjQxOThkZDk4MmVlMTIxNTFhYSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RG93bnRvd248L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzQ3MzhhOTMwY2U5NDRhNGI5MGUxZDhiYzI5ZDQ2M2M0LnNldENvbnRlbnQoaHRtbF9kNTgxOGM1ZmMxODY0MjQxOThkZDk4MmVlMTIxNTFhYSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9jNjI4ODFkY2UzNmM0YWFkYjNkZDBiZDBlZDQ1NzI4Mi5iaW5kUG9wdXAocG9wdXBfNDczOGE5MzBjZTk0NGE0YjkwZTFkOGJjMjlkNDYzYzQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMWNhYjVlNzhlZjU2NDM4NzliODU4MjdhMjJjYzBiZTggPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zNDUzOTg5LC03MS4xMDQzMzIzOTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzg3Y2VmYSIsCiAgImZpbGxPcGFjaXR5IjogMC41LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84ZjFlMGU4MjM4ZjM0OGFkYWE0YTBhNTMyN2E4MzQ4YSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9iOWRjNzhkMmY5NmU0NDQxYmM4NTI2M2E3ZTBlNDJkMyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8wZDdhZjNlNjJjNGQ0NzUzYTA1N2IxODhhMjc1NWFjMSA9ICQoJzxkaXYgaWQ9Imh0bWxfMGQ3YWYzZTYyYzRkNDc1M2EwNTdiMTg4YTI3NTVhYzEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkZlbndheTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYjlkYzc4ZDJmOTZlNDQ0MWJjODUyNjNhN2UwZTQyZDMuc2V0Q29udGVudChodG1sXzBkN2FmM2U2MmM0ZDQ3NTNhMDU3YjE4OGEyNzU1YWMxKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzFjYWI1ZTc4ZWY1NjQzODc5Yjg1ODI3YTIyY2MwYmU4LmJpbmRQb3B1cChwb3B1cF9iOWRjNzhkMmY5NmU0NDQxYmM4NTI2M2E3ZTBlNDJkMyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zZjI5NjNjZjY1OGQ0MWExODc0NGYwZDg3NTAzMGU1YSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM0ODY4NzgsLTcxLjEzODAyMzVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4N2NlZmEiLAogICJmaWxsT3BhY2l0eSI6IDAuNSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDQsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZjYzYjA1ODM5ZDYyNDJiODlmZmM1ZjgwYzIxZjMyYjIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZGUyZGMwNWRkNWQ0NGJjYmE0ZjIzNGYyY2IxNzE5NTYgPSAkKCc8ZGl2IGlkPSJodG1sX2RlMmRjMDVkZDVkNDRiY2JhNGYyMzRmMmNiMTcxOTU2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CcmlnaHRvbjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZjYzYjA1ODM5ZDYyNDJiODlmZmM1ZjgwYzIxZjMyYjIuc2V0Q29udGVudChodG1sX2RlMmRjMDVkZDVkNDRiY2JhNGYyMzRmMmNiMTcxOTU2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzNmMjk2M2NmNjU4ZDQxYTE4NzQ0ZjBkODc1MDMwZTVhLmJpbmRQb3B1cChwb3B1cF9mNjNiMDU4MzlkNjI0MmI4OWZmYzVmODBjMjFmMzJiMik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kMjQ0NGQyY2U1MzM0OTJjOTk4YjdjOTc2MjZiZTk2MCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjI4NjYzOTYsLTcxLjE0NTU3ODJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4N2NlZmEiLAogICJmaWxsT3BhY2l0eSI6IDAuNSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDQsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMzNiZWRlNTA4YWRjNGM2MzhmMWM5NThmMjViYTg1YTcgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjQ0YWE3NDRiMDUzNDhhMzhmOWFkZGNiMWIxODg4ODcgPSAkKCc8ZGl2IGlkPSJodG1sX2Y0NGFhNzQ0YjA1MzQ4YTM4ZjlhZGRjYjFiMTg4ODg3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5XZXN0IFJveGJ1cnk8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzMzYmVkZTUwOGFkYzRjNjM4ZjFjOTU4ZjI1YmE4NWE3LnNldENvbnRlbnQoaHRtbF9mNDRhYTc0NGIwNTM0OGEzOGY5YWRkY2IxYjE4ODg4Nyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kMjQ0NGQyY2U1MzM0OTJjOTk4YjdjOTc2MjZiZTk2MC5iaW5kUG9wdXAocG9wdXBfMzNiZWRlNTA4YWRjNGM2MzhmMWM5NThmMjViYTg1YTcpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZTA2OTgzMTMyNzZmNDUxZmI3ZGVmN2VhMTU3NTJhZDcgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4yNTQ3NDk3LC03MS4xMjU1ODQ4OTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzg3Y2VmYSIsCiAgImZpbGxPcGFjaXR5IjogMC41LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84ZjFlMGU4MjM4ZjM0OGFkYWE0YTBhNTMyN2E4MzQ4YSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xZGM5YjNkNDZlYWQ0YjcxYWE0YmZjZjk2MDQ2N2Y2MSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yYzE0ZGU5ZTE4Njc0YmQ2YjZkZmVlMjVjZjk3ZWU1YyA9ICQoJzxkaXYgaWQ9Imh0bWxfMmMxNGRlOWUxODY3NGJkNmI2ZGZlZTI1Y2Y5N2VlNWMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkh5ZGUgUGFyazwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMWRjOWIzZDQ2ZWFkNGI3MWFhNGJmY2Y5NjA0NjdmNjEuc2V0Q29udGVudChodG1sXzJjMTRkZTllMTg2NzRiZDZiNmRmZWUyNWNmOTdlZTVjKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2UwNjk4MzEzMjc2ZjQ1MWZiN2RlZjdlYTE1NzUyYWQ3LmJpbmRQb3B1cChwb3B1cF8xZGM5YjNkNDZlYWQ0YjcxYWE0YmZjZjk2MDQ2N2Y2MSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9hMDcxODFkNzI1OTI0MzM1OTY4NzgzM2E2OTJkMThmYSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjI2Nzc4NDIsLTcxLjA5MTgyOTE5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODdjZWZhIiwKICAiZmlsbE9wYWNpdHkiOiAwLjUsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA0LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzhmMWUwZTgyMzhmMzQ4YWRhYTRhMGE1MzI3YTgzNDhhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzNiNmRmOTE1MmM3ODRhMzhiMTFmZThmZTBiNjRlNTJmID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzNjNDhjYjk2ZWFhNzQxOTdiY2E5ZTgyZTgzZTdhMzc2ID0gJCgnPGRpdiBpZD0iaHRtbF8zYzQ4Y2I5NmVhYTc0MTk3YmNhOWU4MmU4M2U3YTM3NiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TWF0dGFwYW48L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzNiNmRmOTE1MmM3ODRhMzhiMTFmZThmZTBiNjRlNTJmLnNldENvbnRlbnQoaHRtbF8zYzQ4Y2I5NmVhYTc0MTk3YmNhOWU4MmU4M2U3YTM3Nik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9hMDcxODFkNzI1OTI0MzM1OTY4NzgzM2E2OTJkMThmYS5iaW5kUG9wdXAocG9wdXBfM2I2ZGY5MTUyYzc4NGEzOGIxMWZlOGZlMGI2NGU1MmYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYTU5OTA3YjZjNzEyNDQyY2FkODY3MjQyZTE2YzJlMjcgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4yOTMxMjg0LC03MS4wNjU4MDMzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODdjZWZhIiwKICAiZmlsbE9wYWNpdHkiOiAwLjUsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA0LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzhmMWUwZTgyMzhmMzQ4YWRhYTRhMGE1MzI3YTgzNDhhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzQ5YmQwOWFhY2E1NDQxZTM4NDk4OTBmOTA0MmIzZTZiID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzhlZjVmNmE4YWI4NDQ1ZDVhM2MzZTQ4YmI3NWQ5ZmVjID0gJCgnPGRpdiBpZD0iaHRtbF84ZWY1ZjZhOGFiODQ0NWQ1YTNjM2U0OGJiNzVkOWZlYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RG9yY2hlc3RlcjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNDliZDA5YWFjYTU0NDFlMzg0OTg5MGY5MDQyYjNlNmIuc2V0Q29udGVudChodG1sXzhlZjVmNmE4YWI4NDQ1ZDVhM2MzZTQ4YmI3NWQ5ZmVjKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2E1OTkwN2I2YzcxMjQ0MmNhZDg2NzI0MmUxNmMyZTI3LmJpbmRQb3B1cChwb3B1cF80OWJkMDlhYWNhNTQ0MWUzODQ5ODkwZjkwNDJiM2U2Yik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83M2M5ZDYyYzlkZGU0NDllODhkZmIwY2Y1YmNmMmEzYSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM1MTkyMTcsLTcxLjA1NTA3MDNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4N2NlZmEiLAogICJmaWxsT3BhY2l0eSI6IDAuNSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDQsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNTE4MDQ2M2I3MTVmNGY2NWE3MjM2YTdkZGQxZGMwNDEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZGQ4OWJhYzQwYzY0NGEyNDgwYzliNDk2NjZjMDNkYzIgPSAkKCc8ZGl2IGlkPSJodG1sX2RkODliYWM0MGM2NDRhMjQ4MGM5YjQ5NjY2YzAzZGMyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Tb3V0aCBCb3N0b24gV2F0ZXJmcm9udDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTE4MDQ2M2I3MTVmNGY2NWE3MjM2YTdkZGQxZGMwNDEuc2V0Q29udGVudChodG1sX2RkODliYWM0MGM2NDRhMjQ4MGM5YjQ5NjY2YzAzZGMyKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzczYzlkNjJjOWRkZTQ0OWU4OGRmYjBjZjViY2YyYTNhLmJpbmRQb3B1cChwb3B1cF81MTgwNDYzYjcxNWY0ZjY1YTcyMzZhN2RkZDFkYzA0MSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl80ZWQ5NWI1MGZhZGE0YmY2OGQyOTI5ZDQ1ZjIxY2FmNyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM1MTkyMTcsLTcxLjA1NTA3MDNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4N2NlZmEiLAogICJmaWxsT3BhY2l0eSI6IDAuNSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDQsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOGYxZTBlODIzOGYzNDhhZGFhNGEwYTUzMjdhODM0OGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNjlkNjQ3YzczN2FhNGVjOGI0N2ZlOGMxNzM5NThkOTggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNjMxNjBkNWE1ZDZhNGJlZmE4NWFmNDcwZWNkNjJjNDQgPSAkKCc8ZGl2IGlkPSJodG1sXzYzMTYwZDVhNWQ2YTRiZWZhODVhZjQ3MGVjZDYyYzQ0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Tb3V0aCBCb3N0b248L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzY5ZDY0N2M3MzdhYTRlYzhiNDdmZThjMTczOTU4ZDk4LnNldENvbnRlbnQoaHRtbF82MzE2MGQ1YTVkNmE0YmVmYTg1YWY0NzBlY2Q2MmM0NCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl80ZWQ5NWI1MGZhZGE0YmY2OGQyOTI5ZDQ1ZjIxY2FmNy5iaW5kUG9wdXAocG9wdXBfNjlkNjQ3YzczN2FhNGVjOGI0N2ZlOGMxNzM5NThkOTgpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYWQ5YzU3MjU3N2ZmNDY5Yzk2NWJkZTljYWFiZDIxMzkgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zNDg2ODc4LC03MS4xMzgwMjM1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODdjZWZhIiwKICAiZmlsbE9wYWNpdHkiOiAwLjUsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA0LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzhmMWUwZTgyMzhmMzQ4YWRhYTRhMGE1MzI3YTgzNDhhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzg2N2Q0MWUxZTQ3ZjQ1ZDU5YjY5ZGQxNTUyM2NkMmVkID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzhkOTYyNTEzMzFiNTRiMjk4ZTJkNTE0NjIxZmFmODhiID0gJCgnPGRpdiBpZD0iaHRtbF84ZDk2MjUxMzMxYjU0YjI5OGUyZDUxNDYyMWZhZjg4YiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QWxsc3RvbjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfODY3ZDQxZTFlNDdmNDVkNTliNjlkZDE1NTIzY2QyZWQuc2V0Q29udGVudChodG1sXzhkOTYyNTEzMzFiNTRiMjk4ZTJkNTE0NjIxZmFmODhiKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2FkOWM1NzI1NzdmZjQ2OWM5NjViZGU5Y2FhYmQyMTM5LmJpbmRQb3B1cChwb3B1cF84NjdkNDFlMWU0N2Y0NWQ1OWI2OWRkMTU1MjNjZDJlZCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81OTZjMDc1ZWUzMjg0YzE2YWRlYTUyOTQ0OWViMGJkMyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM5MDUwMSwtNzAuOTk3MTIzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODdjZWZhIiwKICAiZmlsbE9wYWNpdHkiOiAwLjUsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA0LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzhmMWUwZTgyMzhmMzQ4YWRhYTRhMGE1MzI3YTgzNDhhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2QxYTVlOTBmNmMyYjRlZWY4NGM2OWEzYmEwYmNmMzk4ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2NmMGQzMGJhNDQyNTRlMTA4ODEwYWEyZWY1MmFhNWU1ID0gJCgnPGRpdiBpZD0iaHRtbF9jZjBkMzBiYTQ0MjU0ZTEwODgxMGFhMmVmNTJhYTVlNSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SGFyYm9yIElzbGFuZHM8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2QxYTVlOTBmNmMyYjRlZWY4NGM2OWEzYmEwYmNmMzk4LnNldENvbnRlbnQoaHRtbF9jZjBkMzBiYTQ0MjU0ZTEwODgxMGFhMmVmNTJhYTVlNSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81OTZjMDc1ZWUzMjg0YzE2YWRlYTUyOTQ0OWViMGJkMy5iaW5kUG9wdXAocG9wdXBfZDFhNWU5MGY2YzJiNGVlZjg0YzY5YTNiYTBiY2YzOTgpOwoKICAgICAgICAgICAgCiAgICAgICAgCjwvc2NyaXB0Pg==" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



## Segmenting and Clustering Towns in Boston

### Retrieving FourSquare Places of interest

Using the Foursquare API, the explore API function was be used to get the most common venue categories in each neighborhood, and then used this feature to group the neighborhoods into clusters. The k-means clustering algorithm was used for the analysis. Fnally, the Folium library is used to visualize the recommended neighborhoods and their emerging clusters.     

In the ipynb notebook, the function **getNearbyVenues** extracts the following information for the dataframe it generates:

- Venue ID
- Venue Name
- Coordinates : Latitude and Longitude
- Category Name    

The function **getVenuesByCategory** performs the following:

1. **category** based venue search to simulate user venue searches based on certain places of interest. This search extracts the following information:    
 - Venue ID
 - Venue Name
 - Coordinates : Latitude and Longitude
 - Category Name   
 
 
2. For each retrieved **venueID**, retrive the venues category rating.


```python
# define Foursquare Credentials and Version
CLIENT_ID = 'UVIZWYJTNFQWIQHMC1KFPXMCT4HZBPACYDA2RZH2KRENFCAI' # your Foursquare ID
CLIENT_SECRET = 'BXORRJFMB5THDVCNFP5MSVVSWSD1ZHRD20LNLKZHH1ZHY1FF' # your Foursquare Secret
VERSION = '20180605' # Foursquare API version

print('Your credentails:')
print('CLIENT_ID: ' + CLIENT_ID)
print('CLIENT_SECRET:' + CLIENT_SECRET)
```

    Your credentails:
    CLIENT_ID: UVIZWYJTNFQWIQHMC1KFPXMCT4HZBPACYDA2RZH2KRENFCAI
    CLIENT_SECRET:BXORRJFMB5THDVCNFP5MSVVSWSD1ZHRD20LNLKZHH1ZHY1FF
    

### 1. Exploring neighborhoods in Boston

**Using the following foursquare api query url, search venues on all boroughs in Boston neighborhoods.**  
> https://<i></i> api.foursquare.com/v2/venues/**search**
**client_id**=CLIENT_ID&client_secret=**CLIENT_SECRET**&ll=**LATITUDE**,**LONGITUDE**&v=**VERSION**&query=**QUERY**&radius=**RADIUS**&limit=**LIMIT**


```python
radius = 500
LIMIT = 100

venues = []

for lat, long, neighborhood in zip(df['Latitude'], df['Longitude'], df['Name']):
    url = "https://api.foursquare.com/v2/venues/explore?client_id={}&client_secret={}&v={}&ll={},{}&radius={}&limit={}".format(
        CLIENT_ID,
        CLIENT_SECRET,
        VERSION,
        lat,
        long,
        radius, 
        LIMIT)
    
    results = requests.get(url).json()["response"]['groups'][0]['items']
    
    for venue in results:
        venues.append((
            neighborhood,
            lat, 
            long, 
            venue['venue']['name'], 
            venue['venue']['location']['lat'], 
            venue['venue']['location']['lng'],  
            venue['venue']['categories'][0]['name']))
```


```python
# convert the venues list into a new DataFrame
venues_df = pd.DataFrame(venues)

# define the column names
venues_df.columns = ['Neighborhood', 'BoroughLatitude', 'BoroughLongitude', 'VenueName', 'VenueLatitude', 'VenueLongitude', 'VenueCategory']

print(venues_df.shape)
venues_df.head()
```

    (1227, 7)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Neighborhood</th>
      <th>BoroughLatitude</th>
      <th>BoroughLongitude</th>
      <th>VenueName</th>
      <th>VenueLatitude</th>
      <th>VenueLongitude</th>
      <th>VenueCategory</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Roslindale</td>
      <td>42.30069</td>
      <td>-71.113972</td>
      <td>Brassica Kitchen &amp; Cafe</td>
      <td>42.300266</td>
      <td>-71.113160</td>
      <td>New American Restaurant</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Roslindale</td>
      <td>42.30069</td>
      <td>-71.113972</td>
      <td>Mike's Donuts</td>
      <td>42.300735</td>
      <td>-71.114029</td>
      <td>Donut Shop</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Roslindale</td>
      <td>42.30069</td>
      <td>-71.113972</td>
      <td>The Dogwood</td>
      <td>42.300279</td>
      <td>-71.113281</td>
      <td>American Restaurant</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Roslindale</td>
      <td>42.30069</td>
      <td>-71.113972</td>
      <td>Forest Hills Diner</td>
      <td>42.300730</td>
      <td>-71.112889</td>
      <td>Breakfast Spot</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Roslindale</td>
      <td>42.30069</td>
      <td>-71.113972</td>
      <td>Simpli Bar &amp; Bites</td>
      <td>42.297241</td>
      <td>-71.116600</td>
      <td>Bar</td>
    </tr>
  </tbody>
</table>
</div>



### 2. Check venue count per neighborhood


```python
venues_df.groupby('Neighborhood').count()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>BoroughLatitude</th>
      <th>BoroughLongitude</th>
      <th>VenueName</th>
      <th>VenueLatitude</th>
      <th>VenueLongitude</th>
      <th>VenueCategory</th>
    </tr>
    <tr>
      <th>Neighborhood</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Allston</th>
      <td>14</td>
      <td>14</td>
      <td>14</td>
      <td>14</td>
      <td>14</td>
      <td>14</td>
    </tr>
    <tr>
      <th>Back Bay</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Bay Village</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Beacon Hill</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Brighton</th>
      <td>14</td>
      <td>14</td>
      <td>14</td>
      <td>14</td>
      <td>14</td>
      <td>14</td>
    </tr>
    <tr>
      <th>Charlestown</th>
      <td>21</td>
      <td>21</td>
      <td>21</td>
      <td>21</td>
      <td>21</td>
      <td>21</td>
    </tr>
    <tr>
      <th>Chinatown</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Dorchester</th>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>Downtown</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>East Boston</th>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
    </tr>
    <tr>
      <th>Fenway</th>
      <td>53</td>
      <td>53</td>
      <td>53</td>
      <td>53</td>
      <td>53</td>
      <td>53</td>
    </tr>
    <tr>
      <th>Harbor Islands</th>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
    </tr>
    <tr>
      <th>Hyde Park</th>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
    </tr>
    <tr>
      <th>Jamaica Plain</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
    </tr>
    <tr>
      <th>Leather District</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Longwood</th>
      <td>23</td>
      <td>23</td>
      <td>23</td>
      <td>23</td>
      <td>23</td>
      <td>23</td>
    </tr>
    <tr>
      <th>Mattapan</th>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
    </tr>
    <tr>
      <th>Mission Hill</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
    </tr>
    <tr>
      <th>North End</th>
      <td>58</td>
      <td>58</td>
      <td>58</td>
      <td>58</td>
      <td>58</td>
      <td>58</td>
    </tr>
    <tr>
      <th>Roslindale</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
    </tr>
    <tr>
      <th>Roxbury</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
    </tr>
    <tr>
      <th>South Boston</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>South Boston Waterfront</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>South End</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>West End</th>
      <td>37</td>
      <td>37</td>
      <td>37</td>
      <td>37</td>
      <td>37</td>
      <td>37</td>
    </tr>
    <tr>
      <th>West Roxbury</th>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Verify the dtypes 
venues_df.dtypes
```




    Neighborhood         object
    BoroughLatitude     float64
    BoroughLongitude    float64
    VenueName            object
    VenueLatitude       float64
    VenueLongitude      float64
    VenueCategory        object
    dtype: object



**How many unique categories can be curated from all the returned venues?**


```python
# Count number of categories that can be curated.
print('There are {} uniques categories.'.format(len(venues_df['VenueCategory'].unique())))
```

    There are 161 uniques categories.
    

**What are the top 20 most common venue types?**


```python
# Check top 10 most frequently occuring venue type
venues_df.groupby('VenueCategory')['VenueName'].count().sort_values(ascending=False)[:20]
```




    VenueCategory
    Coffee Shop                54
    American Restaurant        48
    Sandwich Place             38
    Chinese Restaurant         35
    Asian Restaurant           32
    Gym                        31
    Pizza Place                30
    Hotel                      30
    Bakery                     27
    Italian Restaurant         26
    Gym / Fitness Center       26
    Café                       24
    Bar                        24
    Seafood Restaurant         23
    Mexican Restaurant         23
    New American Restaurant    22
    Park                       22
    Donut Shop                 21
    Steakhouse                 16
    Salad Place                15
    Name: VenueName, dtype: int64



### 3. Analyze Each Boston Neighborhood nearby recommended venues


```python
# one hot encoding
bos_onehot = pd.get_dummies(venues_df[['VenueCategory']], prefix="", prefix_sep="")

# add Town column back to dataframe
bos_onehot['Neighborhood'] = venues_df['Neighborhood'] 

# move neighborhood column to the first column
fixed_columns = [bos_onehot.columns[-1]] + list(bos_onehot.columns[:-1])
bos_onehot = bos_onehot[fixed_columns]

# Check returned one hot encoding data:
print('One hot encoding returned "{}" rows.'.format(bos_onehot.shape[0]))

# Regroup rows by town and mean of frequency occurrence per category.
bos_grouped = bos_onehot.groupby('Neighborhood').mean().reset_index()

print('One hot encoding re-group returned "{}" rows.'.format(bos_grouped.shape[0]))
bos_grouped.head()
```

    One hot encoding returned "1227" rows.
    One hot encoding re-group returned "26" rows.
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Neighborhood</th>
      <th>Accessories Store</th>
      <th>African Restaurant</th>
      <th>American Restaurant</th>
      <th>Art Gallery</th>
      <th>Art Museum</th>
      <th>Asian Restaurant</th>
      <th>Athletics &amp; Sports</th>
      <th>BBQ Joint</th>
      <th>Bagel Shop</th>
      <th>Bakery</th>
      <th>Bank</th>
      <th>Bar</th>
      <th>Bed &amp; Breakfast</th>
      <th>Big Box Store</th>
      <th>Boat or Ferry</th>
      <th>Bookstore</th>
      <th>Boutique</th>
      <th>Boxing Gym</th>
      <th>Brazilian Restaurant</th>
      <th>Breakfast Spot</th>
      <th>Brewery</th>
      <th>Bubble Tea Shop</th>
      <th>Burger Joint</th>
      <th>Burrito Place</th>
      <th>Bus Station</th>
      <th>Business Service</th>
      <th>Café</th>
      <th>Caribbean Restaurant</th>
      <th>Chinese Restaurant</th>
      <th>Chocolate Shop</th>
      <th>Clothing Store</th>
      <th>Club House</th>
      <th>Cocktail Bar</th>
      <th>Coffee Shop</th>
      <th>Colombian Restaurant</th>
      <th>Comedy Club</th>
      <th>Concert Hall</th>
      <th>Construction &amp; Landscaping</th>
      <th>Convenience Store</th>
      <th>Cosmetics Shop</th>
      <th>Cupcake Shop</th>
      <th>Cycle Studio</th>
      <th>Dance Studio</th>
      <th>Deli / Bodega</th>
      <th>Department Store</th>
      <th>Dessert Shop</th>
      <th>Dim Sum Restaurant</th>
      <th>Diner</th>
      <th>Discount Store</th>
      <th>Dive Bar</th>
      <th>Doctor's Office</th>
      <th>Dog Run</th>
      <th>Donut Shop</th>
      <th>Falafel Restaurant</th>
      <th>Farmers Market</th>
      <th>Fast Food Restaurant</th>
      <th>Food</th>
      <th>Food Court</th>
      <th>Food Truck</th>
      <th>French Restaurant</th>
      <th>Fried Chicken Joint</th>
      <th>Furniture / Home Store</th>
      <th>Gastropub</th>
      <th>Gift Shop</th>
      <th>Gourmet Shop</th>
      <th>Greek Restaurant</th>
      <th>Grocery Store</th>
      <th>Gym</th>
      <th>Gym / Fitness Center</th>
      <th>Historic Site</th>
      <th>History Museum</th>
      <th>Hockey Arena</th>
      <th>Hostel</th>
      <th>Hotel</th>
      <th>Hotel Bar</th>
      <th>Hotpot Restaurant</th>
      <th>Ice Cream Shop</th>
      <th>Indian Restaurant</th>
      <th>Insurance Office</th>
      <th>Irish Pub</th>
      <th>Israeli Restaurant</th>
      <th>Italian Restaurant</th>
      <th>Japanese Restaurant</th>
      <th>Jewelry Store</th>
      <th>Juice Bar</th>
      <th>Korean Restaurant</th>
      <th>Lake</th>
      <th>Library</th>
      <th>Light Rail Station</th>
      <th>Lingerie Store</th>
      <th>Liquor Store</th>
      <th>Lounge</th>
      <th>Market</th>
      <th>Massage Studio</th>
      <th>Mediterranean Restaurant</th>
      <th>Men's Store</th>
      <th>Metro Station</th>
      <th>Mexican Restaurant</th>
      <th>Middle Eastern Restaurant</th>
      <th>Mobile Phone Shop</th>
      <th>Movie Theater</th>
      <th>Museum</th>
      <th>Nail Salon</th>
      <th>New American Restaurant</th>
      <th>Noodle House</th>
      <th>Office</th>
      <th>Opera House</th>
      <th>Outdoor Sculpture</th>
      <th>Park</th>
      <th>Parking</th>
      <th>Pedestrian Plaza</th>
      <th>Performing Arts Venue</th>
      <th>Pet Store</th>
      <th>Pharmacy</th>
      <th>Pizza Place</th>
      <th>Planetarium</th>
      <th>Playground</th>
      <th>Plaza</th>
      <th>Pub</th>
      <th>Rental Car Location</th>
      <th>Restaurant</th>
      <th>River</th>
      <th>Salad Place</th>
      <th>Salon / Barbershop</th>
      <th>Sandwich Place</th>
      <th>Scenic Lookout</th>
      <th>Science Museum</th>
      <th>Sculpture Garden</th>
      <th>Seafood Restaurant</th>
      <th>Shoe Store</th>
      <th>Shopping Mall</th>
      <th>Skate Park</th>
      <th>Skating Rink</th>
      <th>Ski Area</th>
      <th>Ski Chalet</th>
      <th>Southern / Soul Food Restaurant</th>
      <th>Souvenir Shop</th>
      <th>Spa</th>
      <th>Speakeasy</th>
      <th>Sporting Goods Shop</th>
      <th>Sports Bar</th>
      <th>Steakhouse</th>
      <th>Sushi Restaurant</th>
      <th>Szechuan Restaurant</th>
      <th>Tapas Restaurant</th>
      <th>Tea Room</th>
      <th>Tennis Court</th>
      <th>Thai Restaurant</th>
      <th>Theater</th>
      <th>Tour Provider</th>
      <th>Tourist Information Center</th>
      <th>Track</th>
      <th>Trail</th>
      <th>Vegetarian / Vegan Restaurant</th>
      <th>Video Game Store</th>
      <th>Vietnamese Restaurant</th>
      <th>Wine Bar</th>
      <th>Wine Shop</th>
      <th>Women's Store</th>
      <th>Yoga Studio</th>
      <th>Zoo Exhibit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Allston</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.142857</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.071429</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Back Bay</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.06</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.03</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.010000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.05</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.000000</td>
      <td>0.02</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.020000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.04</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bay Village</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.06</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.03</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.010000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.05</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.000000</td>
      <td>0.02</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.020000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.04</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Beacon Hill</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.04</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.060000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.02</td>
      <td>0.020000</td>
      <td>0.03</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.04</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.020000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.05</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.03</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Brighton</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.142857</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.071429</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



### 4. Analyze Boston Town most visited venues


```python
num_top_venues = 10
for town in bos_grouped['Neighborhood']:
    print("# Town=< "+town+" >")
    temp = bos_grouped[bos_grouped['Neighborhood'] == town].T.reset_index()
    temp.columns = ['venue','freq']
    temp = temp.iloc[1:]
    temp['freq'] = temp['freq'].astype(float)
    temp = temp.round({'freq': 2})
    print(temp.sort_values('freq', ascending=False).reset_index(drop=True).head(num_top_venues))
    print('\n')
```

    # Town=< Allston >
                      venue  freq
    0                   Bar  0.14
    1              Pharmacy  0.07
    2               Dog Run  0.07
    3           Coffee Shop  0.07
    4  Gym / Fitness Center  0.07
    5             Gastropub  0.07
    6           Pizza Place  0.07
    7                  Food  0.07
    8          Liquor Store  0.07
    9    Athletics & Sports  0.07
    
    
    # Town=< Back Bay >
                         venue  freq
    0      American Restaurant  0.06
    1                      Gym  0.05
    2                    Hotel  0.05
    3       Seafood Restaurant  0.04
    4     Gym / Fitness Center  0.03
    5         Department Store  0.03
    6           Ice Cream Shop  0.02
    7             Dessert Shop  0.02
    8  New American Restaurant  0.02
    9              Coffee Shop  0.02
    
    
    # Town=< Bay Village >
                         venue  freq
    0      American Restaurant  0.06
    1                      Gym  0.05
    2                    Hotel  0.05
    3       Seafood Restaurant  0.04
    4     Gym / Fitness Center  0.03
    5         Department Store  0.03
    6           Ice Cream Shop  0.02
    7             Dessert Shop  0.02
    8  New American Restaurant  0.02
    9              Coffee Shop  0.02
    
    
    # Town=< Beacon Hill >
                         venue  freq
    0              Coffee Shop  0.06
    1           Sandwich Place  0.05
    2      American Restaurant  0.04
    3  New American Restaurant  0.04
    4       Seafood Restaurant  0.03
    5       Italian Restaurant  0.03
    6       Falafel Restaurant  0.03
    7               Steakhouse  0.03
    8                    Hotel  0.03
    9            Historic Site  0.03
    
    
    # Town=< Brighton >
                      venue  freq
    0                   Bar  0.14
    1              Pharmacy  0.07
    2               Dog Run  0.07
    3           Coffee Shop  0.07
    4  Gym / Fitness Center  0.07
    5             Gastropub  0.07
    6           Pizza Place  0.07
    7                  Food  0.07
    8          Liquor Store  0.07
    9    Athletics & Sports  0.07
    
    
    # Town=< Charlestown >
                     venue  freq
    0  American Restaurant  0.10
    1          Coffee Shop  0.10
    2           Skate Park  0.10
    3        Grocery Store  0.05
    4             Pharmacy  0.05
    5          Bus Station  0.05
    6            Gastropub  0.05
    7                 Park  0.05
    8      Thai Restaurant  0.05
    9            Pet Store  0.05
    
    
    # Town=< Chinatown >
                       venue  freq
    0       Asian Restaurant  0.08
    1     Chinese Restaurant  0.08
    2                 Bakery  0.07
    3            Coffee Shop  0.05
    4                Theater  0.04
    5            Pizza Place  0.03
    6  Performing Arts Venue  0.03
    7         Sandwich Place  0.03
    8     Seafood Restaurant  0.03
    9       Sushi Restaurant  0.03
    
    
    # Town=< Dorchester >
                       venue  freq
    0                   Park  0.25
    1          Metro Station  0.25
    2     Chinese Restaurant  0.25
    3           Liquor Store  0.25
    4      Accessories Store  0.00
    5  Performing Arts Venue  0.00
    6                 Office  0.00
    7            Opera House  0.00
    8      Outdoor Sculpture  0.00
    9                Parking  0.00
    
    
    # Town=< Downtown >
                         venue  freq
    0              Coffee Shop  0.06
    1      American Restaurant  0.04
    2           Sandwich Place  0.04
    3  New American Restaurant  0.04
    4               Steakhouse  0.03
    5            Historic Site  0.03
    6     Gym / Fitness Center  0.03
    7       Falafel Restaurant  0.03
    8                    Hotel  0.03
    9               Restaurant  0.03
    
    
    # Town=< East Boston >
                      venue  freq
    0                 River  0.17
    1      Business Service  0.17
    2                  Park  0.17
    3  Colombian Restaurant  0.17
    4         Metro Station  0.17
    5              Ski Area  0.17
    6      Pedestrian Plaza  0.00
    7                Office  0.00
    8           Opera House  0.00
    9     Outdoor Sculpture  0.00
    
    
    # Town=< Fenway >
                        venue  freq
    0     American Restaurant  0.06
    1  Furniture / Home Store  0.06
    2      Mexican Restaurant  0.06
    3                    Café  0.04
    4        Greek Restaurant  0.04
    5                  Bakery  0.04
    6      Chinese Restaurant  0.04
    7         Thai Restaurant  0.04
    8            Cycle Studio  0.02
    9                     Spa  0.02
    
    
    # Town=< Harbor Islands >
                      venue  freq
    0                 River  0.17
    1      Business Service  0.17
    2                  Park  0.17
    3  Colombian Restaurant  0.17
    4         Metro Station  0.17
    5              Ski Area  0.17
    6      Pedestrian Plaza  0.00
    7                Office  0.00
    8           Opera House  0.00
    9     Outdoor Sculpture  0.00
    
    
    # Town=< Hyde Park >
                     venue  freq
    0  American Restaurant  0.17
    1        Grocery Store  0.08
    2          Pizza Place  0.08
    3  Fried Chicken Joint  0.08
    4              Theater  0.08
    5                  Bar  0.08
    6             Pharmacy  0.08
    7       Ice Cream Shop  0.08
    8           Donut Shop  0.08
    9       Discount Store  0.08
    
    
    # Town=< Jamaica Plain >
                    venue  freq
    0                 Gym  0.12
    1         Coffee Shop  0.12
    2             Brewery  0.12
    3          Bagel Shop  0.06
    4            Tea Room  0.06
    5        Tennis Court  0.06
    6  Mexican Restaurant  0.06
    7      Farmers Market  0.06
    8       Shopping Mall  0.06
    9  Chinese Restaurant  0.06
    
    
    # Town=< Leather District >
                     venue  freq
    0       Sandwich Place  0.07
    1          Coffee Shop  0.07
    2   Chinese Restaurant  0.06
    3     Asian Restaurant  0.06
    4               Bakery  0.04
    5           Food Truck  0.04
    6  American Restaurant  0.03
    7                 Café  0.03
    8             Tea Room  0.02
    9           Steakhouse  0.02
    
    
    # Town=< Longwood >
                    venue  freq
    0          Donut Shop  0.13
    1      Sandwich Place  0.09
    2         Pizza Place  0.09
    3  Italian Restaurant  0.09
    4                 Gym  0.04
    5           Bookstore  0.04
    6  Falafel Restaurant  0.04
    7         Coffee Shop  0.04
    8           Gastropub  0.04
    9                Café  0.04
    
    
    # Town=< Mattapan >
                                 venue  freq
    0                Mobile Phone Shop  0.17
    1             Caribbean Restaurant  0.17
    2                         Pharmacy  0.17
    3  Southern / Soul Food Restaurant  0.17
    4                           Bakery  0.17
    5             Fast Food Restaurant  0.17
    6                Accessories Store  0.00
    7                          Parking  0.00
    8                           Office  0.00
    9                      Opera House  0.00
    
    
    # Town=< Mission Hill >
                         venue  freq
    0               Donut Shop  0.19
    1              Pizza Place  0.19
    2  New American Restaurant  0.06
    3       Light Rail Station  0.06
    4   Furniture / Home Store  0.06
    5                    Track  0.06
    6             Liquor Store  0.06
    7             Burger Joint  0.06
    8       African Restaurant  0.06
    9       Italian Restaurant  0.06
    
    
    # Town=< North End >
                    venue  freq
    0         Pizza Place  0.10
    1  Italian Restaurant  0.07
    2          Donut Shop  0.07
    3               Hotel  0.07
    4      Sandwich Place  0.05
    5                 Bar  0.05
    6             Brewery  0.03
    7         Coffee Shop  0.03
    8          Sports Bar  0.03
    9                Park  0.03
    
    
    # Town=< Roslindale >
                            venue  freq
    0                         Bar  0.12
    1               Grocery Store  0.06
    2           Indian Restaurant  0.06
    3                 Bus Station  0.06
    4     New American Restaurant  0.06
    5              Breakfast Spot  0.06
    6                   Pet Store  0.06
    7                 Comedy Club  0.06
    8  Construction & Landscaping  0.06
    9                 Pizza Place  0.06
    
    
    # Town=< Roxbury >
                         venue  freq
    0               Donut Shop  0.19
    1              Pizza Place  0.19
    2  New American Restaurant  0.06
    3       Light Rail Station  0.06
    4   Furniture / Home Store  0.06
    5                    Track  0.06
    6             Liquor Store  0.06
    7             Burger Joint  0.06
    8       African Restaurant  0.06
    9       Italian Restaurant  0.06
    
    
    # Town=< South Boston >
                     venue  freq
    0       Sandwich Place  0.07
    1          Coffee Shop  0.07
    2   Chinese Restaurant  0.06
    3     Asian Restaurant  0.06
    4               Bakery  0.04
    5           Food Truck  0.04
    6  American Restaurant  0.03
    7                 Café  0.03
    8             Tea Room  0.02
    9           Steakhouse  0.02
    
    
    # Town=< South Boston Waterfront >
                     venue  freq
    0       Sandwich Place  0.07
    1          Coffee Shop  0.07
    2   Chinese Restaurant  0.06
    3     Asian Restaurant  0.06
    4               Bakery  0.04
    5           Food Truck  0.04
    6  American Restaurant  0.03
    7                 Café  0.03
    8             Tea Room  0.02
    9           Steakhouse  0.02
    
    
    # Town=< South End >
                         venue  freq
    0      American Restaurant  0.06
    1                      Gym  0.05
    2                    Hotel  0.05
    3       Seafood Restaurant  0.04
    4     Gym / Fitness Center  0.03
    5         Department Store  0.03
    6           Ice Cream Shop  0.02
    7             Dessert Shop  0.02
    8  New American Restaurant  0.02
    9              Coffee Shop  0.02
    
    
    # Town=< West End >
                venue  freq
    0  Science Museum  0.16
    1      Donut Shop  0.11
    2             Bar  0.08
    3            Café  0.05
    4     Pizza Place  0.05
    5            Park  0.05
    6     Zoo Exhibit  0.03
    7      Playground  0.03
    8     Planetarium  0.03
    9      Food Truck  0.03
    
    
    # Town=< West Roxbury >
                       venue  freq
    0      Convenience Store  0.11
    1              BBQ Joint  0.11
    2         Clothing Store  0.11
    3     Mexican Restaurant  0.11
    4                    Bar  0.11
    5      Indian Restaurant  0.11
    6  Vietnamese Restaurant  0.11
    7                    Spa  0.11
    8    American Restaurant  0.11
    9             Playground  0.00
    
    
    


```python
def return_most_common_venues(row, num_top_venues):
    row_categories = row.iloc[1:]
    row_categories_sorted = row_categories.sort_values(ascending=False)
    return row_categories_sorted.index.values[0:num_top_venues]
```


```python
num_top_venues = 10

indicators = ['st', 'nd', 'rd']

# create columns according to number of top venues
columns = ['Neighborhood']
for ind in np.arange(num_top_venues):
    try:
        columns.append('{}{} Most Common Venue'.format(ind+1, indicators[ind]))
    except:
        columns.append('{}th Most Common Venue'.format(ind+1))

# create a new dataframe
town_venues_sorted = pd.DataFrame(columns=columns)
town_venues_sorted['Neighborhood'] = bos_grouped['Neighborhood']

for ind in np.arange(bos_grouped.shape[0]):
    town_venues_sorted.iloc[ind, 1:] = return_most_common_venues(bos_grouped.iloc[ind, :], num_top_venues)

print(town_venues_sorted.shape)
town_venues_sorted
```

    (26, 11)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Neighborhood</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Allston</td>
      <td>Bar</td>
      <td>Gym / Fitness Center</td>
      <td>Chinese Restaurant</td>
      <td>Gastropub</td>
      <td>Dog Run</td>
      <td>Liquor Store</td>
      <td>Coffee Shop</td>
      <td>Food</td>
      <td>Plaza</td>
      <td>Athletics &amp; Sports</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Back Bay</td>
      <td>American Restaurant</td>
      <td>Gym</td>
      <td>Hotel</td>
      <td>Seafood Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Department Store</td>
      <td>Accessories Store</td>
      <td>Juice Bar</td>
      <td>Plaza</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bay Village</td>
      <td>American Restaurant</td>
      <td>Gym</td>
      <td>Hotel</td>
      <td>Seafood Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Department Store</td>
      <td>Accessories Store</td>
      <td>Juice Bar</td>
      <td>Plaza</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Beacon Hill</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>American Restaurant</td>
      <td>New American Restaurant</td>
      <td>Italian Restaurant</td>
      <td>Seafood Restaurant</td>
      <td>Hotel</td>
      <td>Falafel Restaurant</td>
      <td>Historic Site</td>
      <td>Steakhouse</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Brighton</td>
      <td>Bar</td>
      <td>Gym / Fitness Center</td>
      <td>Chinese Restaurant</td>
      <td>Gastropub</td>
      <td>Dog Run</td>
      <td>Liquor Store</td>
      <td>Coffee Shop</td>
      <td>Food</td>
      <td>Plaza</td>
      <td>Athletics &amp; Sports</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Charlestown</td>
      <td>American Restaurant</td>
      <td>Coffee Shop</td>
      <td>Skate Park</td>
      <td>Convenience Store</td>
      <td>Gastropub</td>
      <td>Pet Store</td>
      <td>Park</td>
      <td>Chinese Restaurant</td>
      <td>Donut Shop</td>
      <td>Shopping Mall</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Chinatown</td>
      <td>Chinese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Bakery</td>
      <td>Coffee Shop</td>
      <td>Theater</td>
      <td>Performing Arts Venue</td>
      <td>Pizza Place</td>
      <td>Sandwich Place</td>
      <td>Seafood Restaurant</td>
      <td>Sushi Restaurant</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Dorchester</td>
      <td>Park</td>
      <td>Metro Station</td>
      <td>Chinese Restaurant</td>
      <td>Liquor Store</td>
      <td>Dog Run</td>
      <td>Food</td>
      <td>Fast Food Restaurant</td>
      <td>Farmers Market</td>
      <td>Falafel Restaurant</td>
      <td>Donut Shop</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Downtown</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>American Restaurant</td>
      <td>New American Restaurant</td>
      <td>Falafel Restaurant</td>
      <td>Restaurant</td>
      <td>Hotel</td>
      <td>Historic Site</td>
      <td>Gym / Fitness Center</td>
      <td>Steakhouse</td>
    </tr>
    <tr>
      <th>9</th>
      <td>East Boston</td>
      <td>Park</td>
      <td>River</td>
      <td>Metro Station</td>
      <td>Business Service</td>
      <td>Colombian Restaurant</td>
      <td>Ski Area</td>
      <td>Dive Bar</td>
      <td>Farmers Market</td>
      <td>Falafel Restaurant</td>
      <td>Donut Shop</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Fenway</td>
      <td>American Restaurant</td>
      <td>Furniture / Home Store</td>
      <td>Mexican Restaurant</td>
      <td>Café</td>
      <td>Chinese Restaurant</td>
      <td>Bakery</td>
      <td>Greek Restaurant</td>
      <td>Thai Restaurant</td>
      <td>Cycle Studio</td>
      <td>Movie Theater</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Harbor Islands</td>
      <td>Park</td>
      <td>River</td>
      <td>Metro Station</td>
      <td>Business Service</td>
      <td>Colombian Restaurant</td>
      <td>Ski Area</td>
      <td>Dive Bar</td>
      <td>Farmers Market</td>
      <td>Falafel Restaurant</td>
      <td>Donut Shop</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Hyde Park</td>
      <td>American Restaurant</td>
      <td>Ice Cream Shop</td>
      <td>Theater</td>
      <td>Pizza Place</td>
      <td>Pharmacy</td>
      <td>Discount Store</td>
      <td>Donut Shop</td>
      <td>Grocery Store</td>
      <td>Bar</td>
      <td>Fried Chicken Joint</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Jamaica Plain</td>
      <td>Brewery</td>
      <td>Coffee Shop</td>
      <td>Gym</td>
      <td>American Restaurant</td>
      <td>Tennis Court</td>
      <td>Shopping Mall</td>
      <td>Chinese Restaurant</td>
      <td>Farmers Market</td>
      <td>Liquor Store</td>
      <td>Art Gallery</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Leather District</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Chinese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Bakery</td>
      <td>Food Truck</td>
      <td>American Restaurant</td>
      <td>Café</td>
      <td>Park</td>
      <td>Dive Bar</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Longwood</td>
      <td>Donut Shop</td>
      <td>Italian Restaurant</td>
      <td>Sandwich Place</td>
      <td>Pizza Place</td>
      <td>Pub</td>
      <td>Gym</td>
      <td>Sushi Restaurant</td>
      <td>Liquor Store</td>
      <td>Bookstore</td>
      <td>Café</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Mattapan</td>
      <td>Bakery</td>
      <td>Southern / Soul Food Restaurant</td>
      <td>Pharmacy</td>
      <td>Fast Food Restaurant</td>
      <td>Caribbean Restaurant</td>
      <td>Mobile Phone Shop</td>
      <td>Dog Run</td>
      <td>Food</td>
      <td>Farmers Market</td>
      <td>Falafel Restaurant</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Mission Hill</td>
      <td>Pizza Place</td>
      <td>Donut Shop</td>
      <td>Furniture / Home Store</td>
      <td>Track</td>
      <td>New American Restaurant</td>
      <td>Liquor Store</td>
      <td>Light Rail Station</td>
      <td>Gym</td>
      <td>Burger Joint</td>
      <td>Italian Restaurant</td>
    </tr>
    <tr>
      <th>18</th>
      <td>North End</td>
      <td>Pizza Place</td>
      <td>Hotel</td>
      <td>Donut Shop</td>
      <td>Italian Restaurant</td>
      <td>Sandwich Place</td>
      <td>Bar</td>
      <td>Brewery</td>
      <td>Mexican Restaurant</td>
      <td>Sports Bar</td>
      <td>Coffee Shop</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Roslindale</td>
      <td>Bar</td>
      <td>Pub</td>
      <td>Breakfast Spot</td>
      <td>Grocery Store</td>
      <td>Liquor Store</td>
      <td>Donut Shop</td>
      <td>New American Restaurant</td>
      <td>Pet Store</td>
      <td>Pizza Place</td>
      <td>Rental Car Location</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Roxbury</td>
      <td>Pizza Place</td>
      <td>Donut Shop</td>
      <td>Furniture / Home Store</td>
      <td>Track</td>
      <td>New American Restaurant</td>
      <td>Liquor Store</td>
      <td>Light Rail Station</td>
      <td>Gym</td>
      <td>Burger Joint</td>
      <td>Italian Restaurant</td>
    </tr>
    <tr>
      <th>21</th>
      <td>South Boston</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Chinese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Bakery</td>
      <td>Food Truck</td>
      <td>American Restaurant</td>
      <td>Café</td>
      <td>Park</td>
      <td>Dive Bar</td>
    </tr>
    <tr>
      <th>22</th>
      <td>South Boston Waterfront</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Chinese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Bakery</td>
      <td>Food Truck</td>
      <td>American Restaurant</td>
      <td>Café</td>
      <td>Park</td>
      <td>Dive Bar</td>
    </tr>
    <tr>
      <th>23</th>
      <td>South End</td>
      <td>American Restaurant</td>
      <td>Gym</td>
      <td>Hotel</td>
      <td>Seafood Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Department Store</td>
      <td>Accessories Store</td>
      <td>Juice Bar</td>
      <td>Plaza</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>24</th>
      <td>West End</td>
      <td>Science Museum</td>
      <td>Donut Shop</td>
      <td>Bar</td>
      <td>Pizza Place</td>
      <td>Park</td>
      <td>Café</td>
      <td>Zoo Exhibit</td>
      <td>Food Truck</td>
      <td>Planetarium</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>25</th>
      <td>West Roxbury</td>
      <td>Convenience Store</td>
      <td>BBQ Joint</td>
      <td>Mexican Restaurant</td>
      <td>Clothing Store</td>
      <td>Spa</td>
      <td>Bar</td>
      <td>Indian Restaurant</td>
      <td>Vietnamese Restaurant</td>
      <td>American Restaurant</td>
      <td>Asian Restaurant</td>
    </tr>
  </tbody>
</table>
</div>



### 5. Clustering Neighborhoods  

Run k-means to cluster the Neighborhoods into 5 clusters.


```python
# set number of clusters
kclusters = 5
bos_grouped_clustering = bos_grouped.drop('Neighborhood', 1)
# run k-means clustering
kmeans = KMeans(n_clusters=kclusters, random_state=1).fit(bos_grouped_clustering)

# check cluster labels generated for each row in the dataframe
print(kmeans.labels_[0:10])
print(len(kmeans.labels_))
```

    [1 1 1 1 1 1 1 4 1 2]
    26
    


```python
town_venues_sorted.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
    <tr>
      <th>Neighborhood</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Allston</th>
      <td>Bar</td>
      <td>Gym / Fitness Center</td>
      <td>Chinese Restaurant</td>
      <td>Gastropub</td>
      <td>Dog Run</td>
      <td>Liquor Store</td>
      <td>Coffee Shop</td>
      <td>Food</td>
      <td>Plaza</td>
      <td>Athletics &amp; Sports</td>
    </tr>
    <tr>
      <th>Back Bay</th>
      <td>American Restaurant</td>
      <td>Gym</td>
      <td>Hotel</td>
      <td>Seafood Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Department Store</td>
      <td>Accessories Store</td>
      <td>Juice Bar</td>
      <td>Plaza</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>Bay Village</th>
      <td>American Restaurant</td>
      <td>Gym</td>
      <td>Hotel</td>
      <td>Seafood Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Department Store</td>
      <td>Accessories Store</td>
      <td>Juice Bar</td>
      <td>Plaza</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>Beacon Hill</th>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>American Restaurant</td>
      <td>New American Restaurant</td>
      <td>Italian Restaurant</td>
      <td>Seafood Restaurant</td>
      <td>Hotel</td>
      <td>Falafel Restaurant</td>
      <td>Historic Site</td>
      <td>Steakhouse</td>
    </tr>
    <tr>
      <th>Brighton</th>
      <td>Bar</td>
      <td>Gym / Fitness Center</td>
      <td>Chinese Restaurant</td>
      <td>Gastropub</td>
      <td>Dog Run</td>
      <td>Liquor Store</td>
      <td>Coffee Shop</td>
      <td>Food</td>
      <td>Plaza</td>
      <td>Athletics &amp; Sports</td>
    </tr>
  </tbody>
</table>
</div>




```python
bos_merged = df.set_index("Name")
# add clustering labels
bos_merged['Cluster Labels'] = kmeans.labels_
bos_merged = bos_merged.join(town_venues_sorted)
bos_merged
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OBJECTID</th>
      <th>Acres</th>
      <th>Neighborhood_ID</th>
      <th>SqMiles</th>
      <th>ShapeSTArea</th>
      <th>ShapeSTLength</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
    <tr>
      <th>Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Roslindale</th>
      <td>27</td>
      <td>1605.568237</td>
      <td>15</td>
      <td>2.51</td>
      <td>6.993827e+07</td>
      <td>53563.912597</td>
      <td>42.300690</td>
      <td>-71.113972</td>
      <td>1</td>
      <td>Bar</td>
      <td>Pub</td>
      <td>Breakfast Spot</td>
      <td>Grocery Store</td>
      <td>Liquor Store</td>
      <td>Donut Shop</td>
      <td>New American Restaurant</td>
      <td>Pet Store</td>
      <td>Pizza Place</td>
      <td>Rental Car Location</td>
    </tr>
    <tr>
      <th>Jamaica Plain</th>
      <td>28</td>
      <td>2519.245394</td>
      <td>11</td>
      <td>3.94</td>
      <td>1.097379e+08</td>
      <td>56349.937161</td>
      <td>42.317265</td>
      <td>-71.104160</td>
      <td>1</td>
      <td>Brewery</td>
      <td>Coffee Shop</td>
      <td>Gym</td>
      <td>American Restaurant</td>
      <td>Tennis Court</td>
      <td>Shopping Mall</td>
      <td>Chinese Restaurant</td>
      <td>Farmers Market</td>
      <td>Liquor Store</td>
      <td>Art Gallery</td>
    </tr>
    <tr>
      <th>Mission Hill</th>
      <td>29</td>
      <td>350.853564</td>
      <td>13</td>
      <td>0.55</td>
      <td>1.528312e+07</td>
      <td>17918.724113</td>
      <td>42.331341</td>
      <td>-71.095499</td>
      <td>1</td>
      <td>Pizza Place</td>
      <td>Donut Shop</td>
      <td>Furniture / Home Store</td>
      <td>Track</td>
      <td>New American Restaurant</td>
      <td>Liquor Store</td>
      <td>Light Rail Station</td>
      <td>Gym</td>
      <td>Burger Joint</td>
      <td>Italian Restaurant</td>
    </tr>
    <tr>
      <th>Longwood</th>
      <td>30</td>
      <td>188.611947</td>
      <td>28</td>
      <td>0.29</td>
      <td>8.215904e+06</td>
      <td>11908.757148</td>
      <td>42.336046</td>
      <td>-71.099727</td>
      <td>1</td>
      <td>Donut Shop</td>
      <td>Italian Restaurant</td>
      <td>Sandwich Place</td>
      <td>Pizza Place</td>
      <td>Pub</td>
      <td>Gym</td>
      <td>Sushi Restaurant</td>
      <td>Liquor Store</td>
      <td>Bookstore</td>
      <td>Café</td>
    </tr>
    <tr>
      <th>Bay Village</th>
      <td>31</td>
      <td>26.539839</td>
      <td>33</td>
      <td>0.04</td>
      <td>1.156071e+06</td>
      <td>4650.635493</td>
      <td>42.347350</td>
      <td>-71.075727</td>
      <td>1</td>
      <td>American Restaurant</td>
      <td>Gym</td>
      <td>Hotel</td>
      <td>Seafood Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Department Store</td>
      <td>Accessories Store</td>
      <td>Juice Bar</td>
      <td>Plaza</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>Leather District</th>
      <td>32</td>
      <td>15.639908</td>
      <td>27</td>
      <td>0.02</td>
      <td>6.812717e+05</td>
      <td>3237.140537</td>
      <td>42.351922</td>
      <td>-71.055070</td>
      <td>1</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Chinese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Bakery</td>
      <td>Food Truck</td>
      <td>American Restaurant</td>
      <td>Café</td>
      <td>Park</td>
      <td>Dive Bar</td>
    </tr>
    <tr>
      <th>Chinatown</th>
      <td>33</td>
      <td>76.324410</td>
      <td>26</td>
      <td>0.12</td>
      <td>3.324678e+06</td>
      <td>9736.590413</td>
      <td>42.352392</td>
      <td>-71.062573</td>
      <td>1</td>
      <td>Chinese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Bakery</td>
      <td>Coffee Shop</td>
      <td>Theater</td>
      <td>Performing Arts Venue</td>
      <td>Pizza Place</td>
      <td>Sandwich Place</td>
      <td>Seafood Restaurant</td>
      <td>Sushi Restaurant</td>
    </tr>
    <tr>
      <th>North End</th>
      <td>34</td>
      <td>126.910439</td>
      <td>14</td>
      <td>0.20</td>
      <td>5.527506e+06</td>
      <td>16177.826815</td>
      <td>42.366352</td>
      <td>-71.062150</td>
      <td>4</td>
      <td>Pizza Place</td>
      <td>Hotel</td>
      <td>Donut Shop</td>
      <td>Italian Restaurant</td>
      <td>Sandwich Place</td>
      <td>Bar</td>
      <td>Brewery</td>
      <td>Mexican Restaurant</td>
      <td>Sports Bar</td>
      <td>Coffee Shop</td>
    </tr>
    <tr>
      <th>Roxbury</th>
      <td>35</td>
      <td>2108.469072</td>
      <td>16</td>
      <td>3.29</td>
      <td>9.184455e+07</td>
      <td>49488.800485</td>
      <td>42.331341</td>
      <td>-71.095499</td>
      <td>1</td>
      <td>Pizza Place</td>
      <td>Donut Shop</td>
      <td>Furniture / Home Store</td>
      <td>Track</td>
      <td>New American Restaurant</td>
      <td>Liquor Store</td>
      <td>Light Rail Station</td>
      <td>Gym</td>
      <td>Burger Joint</td>
      <td>Italian Restaurant</td>
    </tr>
    <tr>
      <th>South End</th>
      <td>36</td>
      <td>471.535356</td>
      <td>32</td>
      <td>0.74</td>
      <td>2.054000e+07</td>
      <td>17912.333569</td>
      <td>42.347350</td>
      <td>-71.075727</td>
      <td>2</td>
      <td>American Restaurant</td>
      <td>Gym</td>
      <td>Hotel</td>
      <td>Seafood Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Department Store</td>
      <td>Accessories Store</td>
      <td>Juice Bar</td>
      <td>Plaza</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>Back Bay</th>
      <td>37</td>
      <td>399.314411</td>
      <td>2</td>
      <td>0.62</td>
      <td>1.739407e+07</td>
      <td>19455.671146</td>
      <td>42.347350</td>
      <td>-71.075727</td>
      <td>1</td>
      <td>American Restaurant</td>
      <td>Gym</td>
      <td>Hotel</td>
      <td>Seafood Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Department Store</td>
      <td>Accessories Store</td>
      <td>Juice Bar</td>
      <td>Plaza</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>East Boston</th>
      <td>38</td>
      <td>3012.059593</td>
      <td>8</td>
      <td>4.71</td>
      <td>1.313845e+08</td>
      <td>121089.100852</td>
      <td>42.390501</td>
      <td>-70.997123</td>
      <td>2</td>
      <td>Park</td>
      <td>River</td>
      <td>Metro Station</td>
      <td>Business Service</td>
      <td>Colombian Restaurant</td>
      <td>Ski Area</td>
      <td>Dive Bar</td>
      <td>Farmers Market</td>
      <td>Falafel Restaurant</td>
      <td>Donut Shop</td>
    </tr>
    <tr>
      <th>Charlestown</th>
      <td>39</td>
      <td>871.541223</td>
      <td>4</td>
      <td>1.36</td>
      <td>3.796418e+07</td>
      <td>57509.688645</td>
      <td>42.373678</td>
      <td>-71.069654</td>
      <td>1</td>
      <td>American Restaurant</td>
      <td>Coffee Shop</td>
      <td>Skate Park</td>
      <td>Convenience Store</td>
      <td>Gastropub</td>
      <td>Pet Store</td>
      <td>Park</td>
      <td>Chinese Restaurant</td>
      <td>Donut Shop</td>
      <td>Shopping Mall</td>
    </tr>
    <tr>
      <th>West End</th>
      <td>40</td>
      <td>190.490732</td>
      <td>31</td>
      <td>0.30</td>
      <td>8.297743e+06</td>
      <td>17728.590027</td>
      <td>42.366817</td>
      <td>-71.067777</td>
      <td>1</td>
      <td>Science Museum</td>
      <td>Donut Shop</td>
      <td>Bar</td>
      <td>Pizza Place</td>
      <td>Park</td>
      <td>Café</td>
      <td>Zoo Exhibit</td>
      <td>Food Truck</td>
      <td>Planetarium</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>Beacon Hill</th>
      <td>41</td>
      <td>200.156904</td>
      <td>30</td>
      <td>0.31</td>
      <td>8.718800e+06</td>
      <td>14303.829017</td>
      <td>42.355820</td>
      <td>-71.062826</td>
      <td>1</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>American Restaurant</td>
      <td>New American Restaurant</td>
      <td>Italian Restaurant</td>
      <td>Seafood Restaurant</td>
      <td>Hotel</td>
      <td>Falafel Restaurant</td>
      <td>Historic Site</td>
      <td>Steakhouse</td>
    </tr>
    <tr>
      <th>Downtown</th>
      <td>42</td>
      <td>397.472846</td>
      <td>7</td>
      <td>0.62</td>
      <td>1.731385e+07</td>
      <td>34612.804441</td>
      <td>42.355453</td>
      <td>-71.060453</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>American Restaurant</td>
      <td>New American Restaurant</td>
      <td>Falafel Restaurant</td>
      <td>Restaurant</td>
      <td>Hotel</td>
      <td>Historic Site</td>
      <td>Gym / Fitness Center</td>
      <td>Steakhouse</td>
    </tr>
    <tr>
      <th>Fenway</th>
      <td>43</td>
      <td>560.618461</td>
      <td>34</td>
      <td>0.88</td>
      <td>2.442044e+07</td>
      <td>24620.876452</td>
      <td>42.345399</td>
      <td>-71.104332</td>
      <td>3</td>
      <td>American Restaurant</td>
      <td>Furniture / Home Store</td>
      <td>Mexican Restaurant</td>
      <td>Café</td>
      <td>Chinese Restaurant</td>
      <td>Bakery</td>
      <td>Greek Restaurant</td>
      <td>Thai Restaurant</td>
      <td>Cycle Studio</td>
      <td>Movie Theater</td>
    </tr>
    <tr>
      <th>Brighton</th>
      <td>44</td>
      <td>1840.408596</td>
      <td>25</td>
      <td>2.88</td>
      <td>8.016788e+07</td>
      <td>48787.519652</td>
      <td>42.348688</td>
      <td>-71.138024</td>
      <td>0</td>
      <td>Bar</td>
      <td>Gym / Fitness Center</td>
      <td>Chinese Restaurant</td>
      <td>Gastropub</td>
      <td>Dog Run</td>
      <td>Liquor Store</td>
      <td>Coffee Shop</td>
      <td>Food</td>
      <td>Plaza</td>
      <td>Athletics &amp; Sports</td>
    </tr>
    <tr>
      <th>West Roxbury</th>
      <td>45</td>
      <td>3516.421786</td>
      <td>19</td>
      <td>5.49</td>
      <td>1.531747e+08</td>
      <td>66067.419838</td>
      <td>42.286640</td>
      <td>-71.145578</td>
      <td>1</td>
      <td>Convenience Store</td>
      <td>BBQ Joint</td>
      <td>Mexican Restaurant</td>
      <td>Clothing Store</td>
      <td>Spa</td>
      <td>Bar</td>
      <td>Indian Restaurant</td>
      <td>Vietnamese Restaurant</td>
      <td>American Restaurant</td>
      <td>Asian Restaurant</td>
    </tr>
    <tr>
      <th>Hyde Park</th>
      <td>46</td>
      <td>2927.221168</td>
      <td>10</td>
      <td>4.57</td>
      <td>1.275092e+08</td>
      <td>66861.244955</td>
      <td>42.254750</td>
      <td>-71.125585</td>
      <td>1</td>
      <td>American Restaurant</td>
      <td>Ice Cream Shop</td>
      <td>Theater</td>
      <td>Pizza Place</td>
      <td>Pharmacy</td>
      <td>Discount Store</td>
      <td>Donut Shop</td>
      <td>Grocery Store</td>
      <td>Bar</td>
      <td>Fried Chicken Joint</td>
    </tr>
    <tr>
      <th>Mattapan</th>
      <td>47</td>
      <td>1352.098354</td>
      <td>12</td>
      <td>2.11</td>
      <td>5.889717e+07</td>
      <td>42005.773707</td>
      <td>42.267784</td>
      <td>-71.091829</td>
      <td>0</td>
      <td>Bakery</td>
      <td>Southern / Soul Food Restaurant</td>
      <td>Pharmacy</td>
      <td>Fast Food Restaurant</td>
      <td>Caribbean Restaurant</td>
      <td>Mobile Phone Shop</td>
      <td>Dog Run</td>
      <td>Food</td>
      <td>Farmers Market</td>
      <td>Falafel Restaurant</td>
    </tr>
    <tr>
      <th>Dorchester</th>
      <td>48</td>
      <td>4662.879457</td>
      <td>6</td>
      <td>7.29</td>
      <td>2.031142e+08</td>
      <td>104344.034005</td>
      <td>42.293128</td>
      <td>-71.065803</td>
      <td>1</td>
      <td>Park</td>
      <td>Metro Station</td>
      <td>Chinese Restaurant</td>
      <td>Liquor Store</td>
      <td>Dog Run</td>
      <td>Food</td>
      <td>Fast Food Restaurant</td>
      <td>Farmers Market</td>
      <td>Falafel Restaurant</td>
      <td>Donut Shop</td>
    </tr>
    <tr>
      <th>South Boston Waterfront</th>
      <td>49</td>
      <td>621.843524</td>
      <td>29</td>
      <td>0.97</td>
      <td>2.708740e+07</td>
      <td>38391.352905</td>
      <td>42.351922</td>
      <td>-71.055070</td>
      <td>1</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Chinese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Bakery</td>
      <td>Food Truck</td>
      <td>American Restaurant</td>
      <td>Café</td>
      <td>Park</td>
      <td>Dive Bar</td>
    </tr>
    <tr>
      <th>South Boston</th>
      <td>50</td>
      <td>1439.888807</td>
      <td>17</td>
      <td>2.25</td>
      <td>6.272131e+07</td>
      <td>64998.420283</td>
      <td>42.351922</td>
      <td>-71.055070</td>
      <td>1</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Chinese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Bakery</td>
      <td>Food Truck</td>
      <td>American Restaurant</td>
      <td>Café</td>
      <td>Park</td>
      <td>Dive Bar</td>
    </tr>
    <tr>
      <th>Allston</th>
      <td>51</td>
      <td>998.534479</td>
      <td>24</td>
      <td>1.56</td>
      <td>4.349599e+07</td>
      <td>37859.091242</td>
      <td>42.348688</td>
      <td>-71.138024</td>
      <td>1</td>
      <td>Bar</td>
      <td>Gym / Fitness Center</td>
      <td>Chinese Restaurant</td>
      <td>Gastropub</td>
      <td>Dog Run</td>
      <td>Liquor Store</td>
      <td>Coffee Shop</td>
      <td>Food</td>
      <td>Plaza</td>
      <td>Athletics &amp; Sports</td>
    </tr>
    <tr>
      <th>Harbor Islands</th>
      <td>52</td>
      <td>824.888658</td>
      <td>22</td>
      <td>1.29</td>
      <td>3.593201e+07</td>
      <td>92482.183568</td>
      <td>42.390501</td>
      <td>-70.997123</td>
      <td>1</td>
      <td>Park</td>
      <td>River</td>
      <td>Metro Station</td>
      <td>Business Service</td>
      <td>Colombian Restaurant</td>
      <td>Ski Area</td>
      <td>Dive Bar</td>
      <td>Farmers Market</td>
      <td>Falafel Restaurant</td>
      <td>Donut Shop</td>
    </tr>
  </tbody>
</table>
</div>




```python
# create map
map_clusters = folium.Map(location=[latitude, longitude], tiles="Openstreetmap", zoom_start=11)

# set color scheme for the clusters
x = np.arange(kclusters)
ys = [i+x+(i*x)**2 for i in range(kclusters)]
colors_array = cm.rainbow(np.linspace(0, 1, len(ys)))
rainbow = [colors.rgb2hex(i) for i in colors_array]

# add markers to the map
markers_colors = []
for lat, lon, poi, cluster in zip(bos_merged['Latitude'], bos_merged['Longitude'], bos_merged.index.values,kmeans.labels_):
    label = folium.Popup(str(poi) + ' Cluster ' + str(cluster), parse_html=True)
    folium.CircleMarker(
        [lat, lon],
        radius=10,
        popup=label,
        color=rainbow[cluster-1],
        fill=True,
        fill_color=rainbow[cluster-1],
        fill_opacity=1).add_to(map_clusters)
       
map_clusters
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfYjcyNTY5NzE3YzRjNDA1OWI1ZTg1OTM2YWIwYjA5YTMgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwX2I3MjU2OTcxN2M0YzQwNTliNWU4NTkzNmFiMGIwOWEzIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGJvdW5kcyA9IG51bGw7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgdmFyIG1hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMyA9IEwubWFwKAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgJ21hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMycsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB7Y2VudGVyOiBbNDIuMzYwMjUzNCwtNzEuMDU4MjkxMl0sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB6b29tOiAxMSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1heEJvdW5kczogYm91bmRzLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbGF5ZXJzOiBbXSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHdvcmxkQ29weUp1bXA6IGZhbHNlLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NwogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB9KTsKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfNzM3NDNlNmY1YjBiNDhmZTg2ZTVlZmJkYzY4MGY3YzYgPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICdodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZycsCiAgICAgICAgICAgICAgICB7CiAgImF0dHJpYnV0aW9uIjogbnVsbCwKICAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsCiAgIm1heFpvb20iOiAxOCwKICAibWluWm9vbSI6IDEsCiAgIm5vV3JhcCI6IGZhbHNlLAogICJzdWJkb21haW5zIjogImFiYyIKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjcyNTY5NzE3YzRjNDA1OWI1ZTg1OTM2YWIwYjA5YTMpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzU0M2ZiNjhjNzA1MjQyYjdhZTBlM2VhZmNhZmFkNjA1ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDIuMzAwNjkwNDk5OTk5OTksLTcxLjExMzk3MjQwMDAwMDAxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM4MDAwZmYiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODAwMGZmIiwKICAiZmlsbE9wYWNpdHkiOiAxLAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMTAsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjcyNTY5NzE3YzRjNDA1OWI1ZTg1OTM2YWIwYjA5YTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYzBjMzllMDk2YzQyNDFiNmI0MWY3YTZlMTVhZWI3YjkgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfM2RhY2Q4MmVmOTg0NGY1Y2E1Yjk0ODAzYmIyZWZmYTkgPSAkKCc8ZGl2IGlkPSJodG1sXzNkYWNkODJlZjk4NDRmNWNhNWI5NDgwM2JiMmVmZmE5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Sb3NsaW5kYWxlIENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYzBjMzllMDk2YzQyNDFiNmI0MWY3YTZlMTVhZWI3Yjkuc2V0Q29udGVudChodG1sXzNkYWNkODJlZjk4NDRmNWNhNWI5NDgwM2JiMmVmZmE5KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzU0M2ZiNjhjNzA1MjQyYjdhZTBlM2VhZmNhZmFkNjA1LmJpbmRQb3B1cChwb3B1cF9jMGMzOWUwOTZjNDI0MWI2YjQxZjdhNmUxNWFlYjdiOSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl80Y2JlOTJiMjVlYjM0YWM4YjFhZTY5MWQ5MDU5NTdhZiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjMxNzI2NTQsLTcxLjEwNDE2MDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDEsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8yZDY0MmRjOTU4YWE0MzQ3YTdlODFkMjEwYjFjM2ViOSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF80ZjdmNWY2YTg2OTA0NDlhOTVmZDM1ZWJmMDE4MTUxMCA9ICQoJzxkaXYgaWQ9Imh0bWxfNGY3ZjVmNmE4NjkwNDQ5YTk1ZmQzNWViZjAxODE1MTAiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkphbWFpY2EgUGxhaW4gQ2x1c3RlciAxPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8yZDY0MmRjOTU4YWE0MzQ3YTdlODFkMjEwYjFjM2ViOS5zZXRDb250ZW50KGh0bWxfNGY3ZjVmNmE4NjkwNDQ5YTk1ZmQzNWViZjAxODE1MTApOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNGNiZTkyYjI1ZWIzNGFjOGIxYWU2OTFkOTA1OTU3YWYuYmluZFBvcHVwKHBvcHVwXzJkNjQyZGM5NThhYTQzNDdhN2U4MWQyMTBiMWMzZWI5KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2I5ZTljNjdlYjJhMzQwOGY5ZjI5MDRmMWIzZDM2YzlkID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDIuMzMxMzQwNiwtNzEuMDk1NDk5MV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjODAwMGZmIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzgwMDBmZiIsCiAgImZpbGxPcGFjaXR5IjogMSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I3MjU2OTcxN2M0YzQwNTliNWU4NTkzNmFiMGIwOWEzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzg1ZGViNjk3ZDExYjQzYjk4ZDAxMjVjNmM3ZjJmMjBiID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzhmMTA1M2JiYjBjMzQ3ZmViOTIwZDYyYWIzZGEwOTY3ID0gJCgnPGRpdiBpZD0iaHRtbF84ZjEwNTNiYmIwYzM0N2ZlYjkyMGQ2MmFiM2RhMDk2NyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TWlzc2lvbiBIaWxsIENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfODVkZWI2OTdkMTFiNDNiOThkMDEyNWM2YzdmMmYyMGIuc2V0Q29udGVudChodG1sXzhmMTA1M2JiYjBjMzQ3ZmViOTIwZDYyYWIzZGEwOTY3KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2I5ZTljNjdlYjJhMzQwOGY5ZjI5MDRmMWIzZDM2YzlkLmJpbmRQb3B1cChwb3B1cF84NWRlYjY5N2QxMWI0M2I5OGQwMTI1YzZjN2YyZjIwYik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl85NjhjOWZkM2U3YjA0MmYwYmU3NTFjNTcwZjhiNTAwMyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjMzNjA0NTYsLTcxLjA5OTcyNjZdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDEsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF84NzBkNmMyYzM3YjY0MDY2YTFkYWVlM2FhMTdiNzhiYyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9mM2ZjYmI0ZTVhNjM0ZWQxODcwOThmM2ZjYzE2YjA3MyA9ICQoJzxkaXYgaWQ9Imh0bWxfZjNmY2JiNGU1YTYzNGVkMTg3MDk4ZjNmY2MxNmIwNzMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxvbmd3b29kIENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfODcwZDZjMmMzN2I2NDA2NmExZGFlZTNhYTE3Yjc4YmMuc2V0Q29udGVudChodG1sX2YzZmNiYjRlNWE2MzRlZDE4NzA5OGYzZmNjMTZiMDczKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzk2OGM5ZmQzZTdiMDQyZjBiZTc1MWM1NzBmOGI1MDAzLmJpbmRQb3B1cChwb3B1cF84NzBkNmMyYzM3YjY0MDY2YTFkYWVlM2FhMTdiNzhiYyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wZWVhZDhjNDZiNWE0MzdmOTliOWQ0MDRmZGUyNzBiNCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM0NzM1LC03MS4wNzU3MjddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDEsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9iMmY3ZWMxNjM0OTk0Zjg3YjdiMmY1NjUyOWRjYmNjZSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8xMGZhZDk2YzdhZTk0ODI1OWM0NzFkNDU4NTgyZDBlNiA9ICQoJzxkaXYgaWQ9Imh0bWxfMTBmYWQ5NmM3YWU5NDgyNTljNDcxZDQ1ODU4MmQwZTYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJheSBWaWxsYWdlIENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYjJmN2VjMTYzNDk5NGY4N2I3YjJmNTY1MjlkY2JjY2Uuc2V0Q29udGVudChodG1sXzEwZmFkOTZjN2FlOTQ4MjU5YzQ3MWQ0NTg1ODJkMGU2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzBlZWFkOGM0NmI1YTQzN2Y5OWI5ZDQwNGZkZTI3MGI0LmJpbmRQb3B1cChwb3B1cF9iMmY3ZWMxNjM0OTk0Zjg3YjdiMmY1NjUyOWRjYmNjZSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83MzVhZmE2NjE5Zjk0YjJlOWYzMDVlNDlhZmZmZTA4NyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM1MTkyMTcsLTcxLjA1NTA3MDNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDEsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81OTdkODMxODlmZjI0Njk4YTVjZmY5YjE4NzgxN2EzNSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF85MWFjNDRkMTY1ZjM0ZTY2YjkzYTNjODc0NTAxZmE0YiA9ICQoJzxkaXYgaWQ9Imh0bWxfOTFhYzQ0ZDE2NWYzNGU2NmI5M2EzYzg3NDUwMWZhNGIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxlYXRoZXIgRGlzdHJpY3QgQ2x1c3RlciAxPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF81OTdkODMxODlmZjI0Njk4YTVjZmY5YjE4NzgxN2EzNS5zZXRDb250ZW50KGh0bWxfOTFhYzQ0ZDE2NWYzNGU2NmI5M2EzYzg3NDUwMWZhNGIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNzM1YWZhNjYxOWY5NGIyZTlmMzA1ZTQ5YWZmZmUwODcuYmluZFBvcHVwKHBvcHVwXzU5N2Q4MzE4OWZmMjQ2OThhNWNmZjliMTg3ODE3YTM1KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2NjMjc5ZGM5ODU2NDRiMTViYWY3ODM1NmE4YTk3OGM1ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDIuMzUyMzkyMSwtNzEuMDYyNTcyNl0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjODAwMGZmIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzgwMDBmZiIsCiAgImZpbGxPcGFjaXR5IjogMSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I3MjU2OTcxN2M0YzQwNTliNWU4NTkzNmFiMGIwOWEzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2VkYjhhMTIzYmMzYjRlYjNhYjAzZGJlNzQ2ZjVhNmVmID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzU4MDEzNzk4OGRiNjQ0OTg5MmVkNTU0OGI0NzkwMzQxID0gJCgnPGRpdiBpZD0iaHRtbF81ODAxMzc5ODhkYjY0NDk4OTJlZDU1NDhiNDc5MDM0MSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2hpbmF0b3duIENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZWRiOGExMjNiYzNiNGViM2FiMDNkYmU3NDZmNWE2ZWYuc2V0Q29udGVudChodG1sXzU4MDEzNzk4OGRiNjQ0OTg5MmVkNTU0OGI0NzkwMzQxKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2NjMjc5ZGM5ODU2NDRiMTViYWY3ODM1NmE4YTk3OGM1LmJpbmRQb3B1cChwb3B1cF9lZGI4YTEyM2JjM2I0ZWIzYWIwM2RiZTc0NmY1YTZlZik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl82YzlkMmRjYTc1YTg0MDAzYTRkYWZlYmViZDM3YzAxYyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM2NjM1MTcsLTcxLjA2MjE1MDRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmYjM2MCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZmIzNjAiLAogICJmaWxsT3BhY2l0eSI6IDEsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8wZDg5ZTIyMWQ5Yzk0NjI1Yjc5MDA2ZGNiYmZlM2QzMyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zNTMyOWRkNDRhMzU0ZmZlOWY0MzgxOGE2MTg5ZWM4NCA9ICQoJzxkaXYgaWQ9Imh0bWxfMzUzMjlkZDQ0YTM1NGZmZTlmNDM4MThhNjE4OWVjODQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk5vcnRoIEVuZCBDbHVzdGVyIDQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzBkODllMjIxZDljOTQ2MjViNzkwMDZkY2JiZmUzZDMzLnNldENvbnRlbnQoaHRtbF8zNTMyOWRkNDRhMzU0ZmZlOWY0MzgxOGE2MTg5ZWM4NCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl82YzlkMmRjYTc1YTg0MDAzYTRkYWZlYmViZDM3YzAxYy5iaW5kUG9wdXAocG9wdXBfMGQ4OWUyMjFkOWM5NDYyNWI3OTAwNmRjYmJmZTNkMzMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNThjMDdiNTNlYWUyNDA1ZWEyNzM5ZTk0NWNmOTk2ODggPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zMzEzNDA2LC03MS4wOTU0OTkxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM4MDAwZmYiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODAwMGZmIiwKICAiZmlsbE9wYWNpdHkiOiAxLAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMTAsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjcyNTY5NzE3YzRjNDA1OWI1ZTg1OTM2YWIwYjA5YTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOWY0N2E1OGJmYzczNDk0Yjk4YTJlNDM2YjFhNzk1NjAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfOWZlZTI2MWE2OTZlNGE0NDliMzEwMGEyYWI4Yzg1NzkgPSAkKCc8ZGl2IGlkPSJodG1sXzlmZWUyNjFhNjk2ZTRhNDQ5YjMxMDBhMmFiOGM4NTc5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Sb3hidXJ5IENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOWY0N2E1OGJmYzczNDk0Yjk4YTJlNDM2YjFhNzk1NjAuc2V0Q29udGVudChodG1sXzlmZWUyNjFhNjk2ZTRhNDQ5YjMxMDBhMmFiOGM4NTc5KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzU4YzA3YjUzZWFlMjQwNWVhMjczOWU5NDVjZjk5Njg4LmJpbmRQb3B1cChwb3B1cF85ZjQ3YTU4YmZjNzM0OTRiOThhMmU0MzZiMWE3OTU2MCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81YzkzZWZjNDQzYjI0OTJiYjNmMGVkMDMwZmYxY2ViNSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM0NzM1LC03MS4wNzU3MjddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzAwYjVlYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMwMGI1ZWIiLAogICJmaWxsT3BhY2l0eSI6IDEsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9mM2ZiZGNjMDkzNWY0NjQ0OWZiNjk5YWMxNjliNWFkNSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zNDA5ZDViZDc3NzY0ZDRjODk1ZGNiZGI4NWFiYWQyYyA9ICQoJzxkaXYgaWQ9Imh0bWxfMzQwOWQ1YmQ3Nzc2NGQ0Yzg5NWRjYmRiODVhYmFkMmMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNvdXRoIEVuZCBDbHVzdGVyIDI8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2YzZmJkY2MwOTM1ZjQ2NDQ5ZmI2OTlhYzE2OWI1YWQ1LnNldENvbnRlbnQoaHRtbF8zNDA5ZDViZDc3NzY0ZDRjODk1ZGNiZGI4NWFiYWQyYyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81YzkzZWZjNDQzYjI0OTJiYjNmMGVkMDMwZmYxY2ViNS5iaW5kUG9wdXAocG9wdXBfZjNmYmRjYzA5MzVmNDY0NDlmYjY5OWFjMTY5YjVhZDUpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMDY5ZWQzNTBiNGM5NDhkZWEzZWQ5NTU2ZTM1MWNkNTcgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zNDczNSwtNzEuMDc1NzI3XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM4MDAwZmYiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODAwMGZmIiwKICAiZmlsbE9wYWNpdHkiOiAxLAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMTAsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjcyNTY5NzE3YzRjNDA1OWI1ZTg1OTM2YWIwYjA5YTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfODQzMDhkM2Q2ZTE4NDRhNjkxMjJkZWNmNGU2OGFkMzQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNWU1ODM4YTMwYjA4NDQ2NmFmNjZiZjY0NGZkMDI2ZjkgPSAkKCc8ZGl2IGlkPSJodG1sXzVlNTgzOGEzMGIwODQ0NjZhZjY2YmY2NDRmZDAyNmY5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CYWNrIEJheSBDbHVzdGVyIDE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzg0MzA4ZDNkNmUxODQ0YTY5MTIyZGVjZjRlNjhhZDM0LnNldENvbnRlbnQoaHRtbF81ZTU4MzhhMzBiMDg0NDY2YWY2NmJmNjQ0ZmQwMjZmOSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8wNjllZDM1MGI0Yzk0OGRlYTNlZDk1NTZlMzUxY2Q1Ny5iaW5kUG9wdXAocG9wdXBfODQzMDhkM2Q2ZTE4NDRhNjkxMjJkZWNmNGU2OGFkMzQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMjMyOWE0MGE5NTQzNGFkMThkOWEyM2U4YWM1MjYyMDUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zOTA1MDEsLTcwLjk5NzEyM10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjMDBiNWViIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzAwYjVlYiIsCiAgImZpbGxPcGFjaXR5IjogMSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I3MjU2OTcxN2M0YzQwNTliNWU4NTkzNmFiMGIwOWEzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzBhYWQ2N2FkZjhkZjQzOGVhZGUxODZlMTkyNTRkNzBjID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzQ2ZGQxZDhiNWIwODQ0OWI5ZjBhMmQxYTRmZmZjOGIyID0gJCgnPGRpdiBpZD0iaHRtbF80NmRkMWQ4YjViMDg0NDliOWYwYTJkMWE0ZmZmYzhiMiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RWFzdCBCb3N0b24gQ2x1c3RlciAyPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wYWFkNjdhZGY4ZGY0MzhlYWRlMTg2ZTE5MjU0ZDcwYy5zZXRDb250ZW50KGh0bWxfNDZkZDFkOGI1YjA4NDQ5YjlmMGEyZDFhNGZmZmM4YjIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMjMyOWE0MGE5NTQzNGFkMThkOWEyM2U4YWM1MjYyMDUuYmluZFBvcHVwKHBvcHVwXzBhYWQ2N2FkZjhkZjQzOGVhZGUxODZlMTkyNTRkNzBjKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2RhYjMzY2JkNDI1YjRjZGFiOWEwMWE2NTNlMDRiMDA2ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDIuMzczNjc4Mjk5OTk5OTksLTcxLjA2OTY1MzVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDEsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9hMGY3OGY5YTU0MDQ0NGRhYjU5YWY1ODhmNWI0MjM3MyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hMzcyMTJiMDljNmI0NGUxYmYxMTgwNmUyMTE5YzBkNyA9ICQoJzxkaXYgaWQ9Imh0bWxfYTM3MjEyYjA5YzZiNDRlMWJmMTE4MDZlMjExOWMwZDciIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNoYXJsZXN0b3duIENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYTBmNzhmOWE1NDA0NDRkYWI1OWFmNTg4ZjViNDIzNzMuc2V0Q29udGVudChodG1sX2EzNzIxMmIwOWM2YjQ0ZTFiZjExODA2ZTIxMTljMGQ3KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2RhYjMzY2JkNDI1YjRjZGFiOWEwMWE2NTNlMDRiMDA2LmJpbmRQb3B1cChwb3B1cF9hMGY3OGY5YTU0MDQ0NGRhYjU5YWY1ODhmNWI0MjM3Myk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81N2NhMTRmMTY4ZDc0ODM3OWQ0NjE4YmIwN2MyNGJmZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM2NjgxNzIsLTcxLjA2Nzc3NjldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDEsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9lZTliZmViOTAwNTY0ODljYjM2ZTZiYTZmOGFmMTA4OSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF85MmI1Y2JjNThhYTI0ZmU5YWIzY2U1NzhiYzc0NWY5MiA9ICQoJzxkaXYgaWQ9Imh0bWxfOTJiNWNiYzU4YWEyNGZlOWFiM2NlNTc4YmM3NDVmOTIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPldlc3QgRW5kIENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZWU5YmZlYjkwMDU2NDg5Y2IzNmU2YmE2ZjhhZjEwODkuc2V0Q29udGVudChodG1sXzkyYjVjYmM1OGFhMjRmZTlhYjNjZTU3OGJjNzQ1ZjkyKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzU3Y2ExNGYxNjhkNzQ4Mzc5ZDQ2MThiYjA3YzI0YmZkLmJpbmRQb3B1cChwb3B1cF9lZTliZmViOTAwNTY0ODljYjM2ZTZiYTZmOGFmMTA4OSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9lZTNhYTU3MDkzMmY0YTA4YmZiODhhMmY3M2Q2YWE1ZiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM1NTgxOTksLTcxLjA2MjgyNTY5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM4MDAwZmYiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODAwMGZmIiwKICAiZmlsbE9wYWNpdHkiOiAxLAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMTAsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjcyNTY5NzE3YzRjNDA1OWI1ZTg1OTM2YWIwYjA5YTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYWE0MmQ0NDBkZDFhNDY5ZGE2MjljNTI1ODQ4NWIyNzYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfOTE1NDk1ZmRmMjNkNDU2Njg3NGMwNTA3YTJlZjk5NzkgPSAkKCc8ZGl2IGlkPSJodG1sXzkxNTQ5NWZkZjIzZDQ1NjY4NzRjMDUwN2EyZWY5OTc5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CZWFjb24gSGlsbCBDbHVzdGVyIDE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2FhNDJkNDQwZGQxYTQ2OWRhNjI5YzUyNTg0ODViMjc2LnNldENvbnRlbnQoaHRtbF85MTU0OTVmZGYyM2Q0NTY2ODc0YzA1MDdhMmVmOTk3OSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9lZTNhYTU3MDkzMmY0YTA4YmZiODhhMmY3M2Q2YWE1Zi5iaW5kUG9wdXAocG9wdXBfYWE0MmQ0NDBkZDFhNDY5ZGE2MjljNTI1ODQ4NWIyNzYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfM2VmOThjNDY5NTY5NGQ3Njk0N2Q4ZTdkMTI4YTllNjIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zNTU0NTMsLTcxLjA2MDQ1MzI5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAxLAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMTAsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjcyNTY5NzE3YzRjNDA1OWI1ZTg1OTM2YWIwYjA5YTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMTJhYzdkZTFlZThkNDg1MGI5YjJkMjY4MGU1N2U3NzkgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNGU5ZTJjMGM1ZjBhNDAyNTk5NzA2YTc4YjJkOTU4N2YgPSAkKCc8ZGl2IGlkPSJodG1sXzRlOWUyYzBjNWYwYTQwMjU5OTcwNmE3OGIyZDk1ODdmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Eb3dudG93biBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzEyYWM3ZGUxZWU4ZDQ4NTBiOWIyZDI2ODBlNTdlNzc5LnNldENvbnRlbnQoaHRtbF80ZTllMmMwYzVmMGE0MDI1OTk3MDZhNzhiMmQ5NTg3Zik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8zZWY5OGM0Njk1Njk0ZDc2OTQ3ZDhlN2QxMjhhOWU2Mi5iaW5kUG9wdXAocG9wdXBfMTJhYzdkZTFlZThkNDg1MGI5YjJkMjY4MGU1N2U3NzkpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYjg2MzJkZjg5MTUyNDMxOWIwOTM5NTljZjc3Y2E5OGMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zNDUzOTg5LC03MS4xMDQzMzIzOTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjODBmZmI0IiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzgwZmZiNCIsCiAgImZpbGxPcGFjaXR5IjogMSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I3MjU2OTcxN2M0YzQwNTliNWU4NTkzNmFiMGIwOWEzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2YzODFiODFjMWNiYjQ2MTdhODY5MzBkMDQ4OGI4YjA3ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2NlYjUxZjA2NjYxNjRhMTU4ZGE2MWJiMzA5M2MxMzcwID0gJCgnPGRpdiBpZD0iaHRtbF9jZWI1MWYwNjY2MTY0YTE1OGRhNjFiYjMwOTNjMTM3MCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RmVud2F5IENsdXN0ZXIgMzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZjM4MWI4MWMxY2JiNDYxN2E4NjkzMGQwNDg4YjhiMDcuc2V0Q29udGVudChodG1sX2NlYjUxZjA2NjYxNjRhMTU4ZGE2MWJiMzA5M2MxMzcwKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2I4NjMyZGY4OTE1MjQzMTliMDkzOTU5Y2Y3N2NhOThjLmJpbmRQb3B1cChwb3B1cF9mMzgxYjgxYzFjYmI0NjE3YTg2OTMwZDA0ODhiOGIwNyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zMGJjNzlhNjRkYjg0MDE1OTFkODE0NTNkNzliMTQyNSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM0ODY4NzgsLTcxLjEzODAyMzVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDEsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83MDhjMzdlNDAxYmU0MGM2OGY2ZWQxOThkMDliNmM3YSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9mN2FlZTdhYTA3OTk0YzliYWE3MWVjMjQzNDBlYmZkMyA9ICQoJzxkaXYgaWQ9Imh0bWxfZjdhZWU3YWEwNzk5NGM5YmFhNzFlYzI0MzQwZWJmZDMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJyaWdodG9uIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNzA4YzM3ZTQwMWJlNDBjNjhmNmVkMTk4ZDA5YjZjN2Euc2V0Q29udGVudChodG1sX2Y3YWVlN2FhMDc5OTRjOWJhYTcxZWMyNDM0MGViZmQzKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzMwYmM3OWE2NGRiODQwMTU5MWQ4MTQ1M2Q3OWIxNDI1LmJpbmRQb3B1cChwb3B1cF83MDhjMzdlNDAxYmU0MGM2OGY2ZWQxOThkMDliNmM3YSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wNWEzMjRkNzY3NDY0MzJhYjU5ZmRlODJkNTdjMDE0MiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjI4NjYzOTYsLTcxLjE0NTU3ODJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDEsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8yMTM4MDhlMjJjM2M0YWE2OTExMGFlMGU4ODYyMDI5NyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yOTJiNjU2ZWM3ZWY0Y2VmOWQzZDlmNjc5ZTNkNTViYyA9ICQoJzxkaXYgaWQ9Imh0bWxfMjkyYjY1NmVjN2VmNGNlZjlkM2Q5ZjY3OWUzZDU1YmMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPldlc3QgUm94YnVyeSBDbHVzdGVyIDE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzIxMzgwOGUyMmMzYzRhYTY5MTEwYWUwZTg4NjIwMjk3LnNldENvbnRlbnQoaHRtbF8yOTJiNjU2ZWM3ZWY0Y2VmOWQzZDlmNjc5ZTNkNTViYyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8wNWEzMjRkNzY3NDY0MzJhYjU5ZmRlODJkNTdjMDE0Mi5iaW5kUG9wdXAocG9wdXBfMjEzODA4ZTIyYzNjNGFhNjkxMTBhZTBlODg2MjAyOTcpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOWY5YmYyZjMzNGU2NDk0YjgzN2FhZDQwYTI1MGNlMzUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4yNTQ3NDk3LC03MS4xMjU1ODQ4OTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjODAwMGZmIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzgwMDBmZiIsCiAgImZpbGxPcGFjaXR5IjogMSwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I3MjU2OTcxN2M0YzQwNTliNWU4NTkzNmFiMGIwOWEzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzNhMTJiNzMxYjIyZjQ1YjM4YWZiN2Y2YTNhYzhkZmFjID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzY2MDhjZmI0YmI5MjQ5ZWNiZTlhZDc5ZmM4NTQ2YTA1ID0gJCgnPGRpdiBpZD0iaHRtbF82NjA4Y2ZiNGJiOTI0OWVjYmU5YWQ3OWZjODU0NmEwNSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SHlkZSBQYXJrIENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfM2ExMmI3MzFiMjJmNDViMzhhZmI3ZjZhM2FjOGRmYWMuc2V0Q29udGVudChodG1sXzY2MDhjZmI0YmI5MjQ5ZWNiZTlhZDc5ZmM4NTQ2YTA1KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzlmOWJmMmYzMzRlNjQ5NGI4MzdhYWQ0MGEyNTBjZTM1LmJpbmRQb3B1cChwb3B1cF8zYTEyYjczMWIyMmY0NWIzOGFmYjdmNmEzYWM4ZGZhYyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl80NTU3NTE1NTg3MWQ0MDBjYWNiMDgxY2VlMzE3MTYyYyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjI2Nzc4NDIsLTcxLjA5MTgyOTE5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAxLAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMTAsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjcyNTY5NzE3YzRjNDA1OWI1ZTg1OTM2YWIwYjA5YTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYzcwNTllZTc0YjczNDhiYzhhNTgzYWRmNTlmMjI4OTMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfY2M4NTVmY2ExYTExNGFiNzhhZGFjZjlkYjI5MmVkYmEgPSAkKCc8ZGl2IGlkPSJodG1sX2NjODU1ZmNhMWExMTRhYjc4YWRhY2Y5ZGIyOTJlZGJhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5NYXR0YXBhbiBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2M3MDU5ZWU3NGI3MzQ4YmM4YTU4M2FkZjU5ZjIyODkzLnNldENvbnRlbnQoaHRtbF9jYzg1NWZjYTFhMTE0YWI3OGFkYWNmOWRiMjkyZWRiYSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl80NTU3NTE1NTg3MWQ0MDBjYWNiMDgxY2VlMzE3MTYyYy5iaW5kUG9wdXAocG9wdXBfYzcwNTllZTc0YjczNDhiYzhhNTgzYWRmNTlmMjI4OTMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOTg5OTA4YTg2NWY5NDliNDg5YmZhMDkzYjM1MTJlN2YgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4yOTMxMjg0LC03MS4wNjU4MDMzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM4MDAwZmYiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODAwMGZmIiwKICAiZmlsbE9wYWNpdHkiOiAxLAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMTAsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjcyNTY5NzE3YzRjNDA1OWI1ZTg1OTM2YWIwYjA5YTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZjU1ZTdiMDcxMjMzNGI4ZTgyZGVlMmRkYTQ5ZjI3YzUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYTVhZjRhMDE0ZGQ4NDViODk4ZWMzZmY5YjZlMmZkYjQgPSAkKCc8ZGl2IGlkPSJodG1sX2E1YWY0YTAxNGRkODQ1Yjg5OGVjM2ZmOWI2ZTJmZGI0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Eb3JjaGVzdGVyIENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZjU1ZTdiMDcxMjMzNGI4ZTgyZGVlMmRkYTQ5ZjI3YzUuc2V0Q29udGVudChodG1sX2E1YWY0YTAxNGRkODQ1Yjg5OGVjM2ZmOWI2ZTJmZGI0KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzk4OTkwOGE4NjVmOTQ5YjQ4OWJmYTA5M2IzNTEyZTdmLmJpbmRQb3B1cChwb3B1cF9mNTVlN2IwNzEyMzM0YjhlODJkZWUyZGRhNDlmMjdjNSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl85Y2NhZWUyNzQ4ZTA0YWExYjI1MzVmZWE3YmM5NDc5ZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM1MTkyMTcsLTcxLjA1NTA3MDNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDEsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF82ZmUzNjY0ODBjOWU0MGM1OTVmYTgxMGQ1Mjc0MThiMCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9kOGE5YzYzYzFmZTk0NzgyYmE0MjFiOGY0NzZjOGUyOSA9ICQoJzxkaXYgaWQ9Imh0bWxfZDhhOWM2M2MxZmU5NDc4MmJhNDIxYjhmNDc2YzhlMjkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNvdXRoIEJvc3RvbiBXYXRlcmZyb250IENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNmZlMzY2NDgwYzllNDBjNTk1ZmE4MTBkNTI3NDE4YjAuc2V0Q29udGVudChodG1sX2Q4YTljNjNjMWZlOTQ3ODJiYTQyMWI4ZjQ3NmM4ZTI5KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzljY2FlZTI3NDhlMDRhYTFiMjUzNWZlYTdiYzk0NzllLmJpbmRQb3B1cChwb3B1cF82ZmUzNjY0ODBjOWU0MGM1OTVmYTgxMGQ1Mjc0MThiMCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8xYmRiYzEwMGRmOTE0ZTNkYmUwMTliZjhmN2Y3ZDYzMiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM1MTkyMTcsLTcxLjA1NTA3MDNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDEsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iNzI1Njk3MTdjNGM0MDU5YjVlODU5MzZhYjBiMDlhMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8zYjNlYmNkZDI5ZGE0NzhiYjNiZmMzYmI5NGNlYzI4ZSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83MzY5MGE5OTE1YzE0NDIwODA1NjcyNzA1ZGQ2ZTg0NCA9ICQoJzxkaXYgaWQ9Imh0bWxfNzM2OTBhOTkxNWMxNDQyMDgwNTY3MjcwNWRkNmU4NDQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNvdXRoIEJvc3RvbiBDbHVzdGVyIDE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzNiM2ViY2RkMjlkYTQ3OGJiM2JmYzNiYjk0Y2VjMjhlLnNldENvbnRlbnQoaHRtbF83MzY5MGE5OTE1YzE0NDIwODA1NjcyNzA1ZGQ2ZTg0NCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8xYmRiYzEwMGRmOTE0ZTNkYmUwMTliZjhmN2Y3ZDYzMi5iaW5kUG9wdXAocG9wdXBfM2IzZWJjZGQyOWRhNDc4YmIzYmZjM2JiOTRjZWMyOGUpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNzMwMTVhZmI3NjFlNDk3NzlhZjlkZGZmYTQxYzhmODUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zNDg2ODc4LC03MS4xMzgwMjM1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM4MDAwZmYiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODAwMGZmIiwKICAiZmlsbE9wYWNpdHkiOiAxLAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMTAsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjcyNTY5NzE3YzRjNDA1OWI1ZTg1OTM2YWIwYjA5YTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfY2VmMzgwYjgzOTU1NGQ4MTg5MzY5MDUxY2I0YTYxM2YgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYzIxM2ZmZjNmZTAwNDAyZmEzMTJiZmM4NzJkYTNkNDUgPSAkKCc8ZGl2IGlkPSJodG1sX2MyMTNmZmYzZmUwMDQwMmZhMzEyYmZjODcyZGEzZDQ1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5BbGxzdG9uIENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfY2VmMzgwYjgzOTU1NGQ4MTg5MzY5MDUxY2I0YTYxM2Yuc2V0Q29udGVudChodG1sX2MyMTNmZmYzZmUwMDQwMmZhMzEyYmZjODcyZGEzZDQ1KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzczMDE1YWZiNzYxZTQ5Nzc5YWY5ZGRmZmE0MWM4Zjg1LmJpbmRQb3B1cChwb3B1cF9jZWYzODBiODM5NTU0ZDgxODkzNjkwNTFjYjRhNjEzZik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mNDUzOGJiODQ5NDM0MTEwYjYzZTgwYjJjZTVkMDZmOSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM5MDUwMSwtNzAuOTk3MTIzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM4MDAwZmYiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODAwMGZmIiwKICAiZmlsbE9wYWNpdHkiOiAxLAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMTAsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjcyNTY5NzE3YzRjNDA1OWI1ZTg1OTM2YWIwYjA5YTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfM2U1MjljZTNjZThhNGY2Y2JkM2I2NGRjYWY0MzRmYzEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNGU3ZTZhMmQ5N2NhNGJlODk4MjZkMWQ2N2ExNTlmNmEgPSAkKCc8ZGl2IGlkPSJodG1sXzRlN2U2YTJkOTdjYTRiZTg5ODI2ZDFkNjdhMTU5ZjZhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IYXJib3IgSXNsYW5kcyBDbHVzdGVyIDE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzNlNTI5Y2UzY2U4YTRmNmNiZDNiNjRkY2FmNDM0ZmMxLnNldENvbnRlbnQoaHRtbF80ZTdlNmEyZDk3Y2E0YmU4OTgyNmQxZDY3YTE1OWY2YSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9mNDUzOGJiODQ5NDM0MTEwYjYzZTgwYjJjZTVkMDZmOS5iaW5kUG9wdXAocG9wdXBfM2U1MjljZTNjZThhNGY2Y2JkM2I2NGRjYWY0MzRmYzEpOwoKICAgICAgICAgICAgCiAgICAgICAgCjwvc2NyaXB0Pg==" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



### 6. Exam Clusters


```python
#Cluster 0
bos_merged.loc[bos_merged['Cluster Labels'] == 0]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OBJECTID</th>
      <th>Acres</th>
      <th>Neighborhood_ID</th>
      <th>SqMiles</th>
      <th>ShapeSTArea</th>
      <th>ShapeSTLength</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
    <tr>
      <th>Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Downtown</th>
      <td>42</td>
      <td>397.472846</td>
      <td>7</td>
      <td>0.62</td>
      <td>1.731385e+07</td>
      <td>34612.804441</td>
      <td>42.355453</td>
      <td>-71.060453</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>American Restaurant</td>
      <td>New American Restaurant</td>
      <td>Falafel Restaurant</td>
      <td>Restaurant</td>
      <td>Hotel</td>
      <td>Historic Site</td>
      <td>Gym / Fitness Center</td>
      <td>Steakhouse</td>
    </tr>
    <tr>
      <th>Brighton</th>
      <td>44</td>
      <td>1840.408596</td>
      <td>25</td>
      <td>2.88</td>
      <td>8.016788e+07</td>
      <td>48787.519652</td>
      <td>42.348688</td>
      <td>-71.138024</td>
      <td>0</td>
      <td>Bar</td>
      <td>Gym / Fitness Center</td>
      <td>Chinese Restaurant</td>
      <td>Gastropub</td>
      <td>Dog Run</td>
      <td>Liquor Store</td>
      <td>Coffee Shop</td>
      <td>Food</td>
      <td>Plaza</td>
      <td>Athletics &amp; Sports</td>
    </tr>
    <tr>
      <th>Mattapan</th>
      <td>47</td>
      <td>1352.098354</td>
      <td>12</td>
      <td>2.11</td>
      <td>5.889717e+07</td>
      <td>42005.773707</td>
      <td>42.267784</td>
      <td>-71.091829</td>
      <td>0</td>
      <td>Bakery</td>
      <td>Southern / Soul Food Restaurant</td>
      <td>Pharmacy</td>
      <td>Fast Food Restaurant</td>
      <td>Caribbean Restaurant</td>
      <td>Mobile Phone Shop</td>
      <td>Dog Run</td>
      <td>Food</td>
      <td>Farmers Market</td>
      <td>Falafel Restaurant</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Cluster 1
bos_merged.loc[bos_merged['Cluster Labels'] == 1]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OBJECTID</th>
      <th>Acres</th>
      <th>Neighborhood_ID</th>
      <th>SqMiles</th>
      <th>ShapeSTArea</th>
      <th>ShapeSTLength</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
    <tr>
      <th>Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Roslindale</th>
      <td>27</td>
      <td>1605.568237</td>
      <td>15</td>
      <td>2.51</td>
      <td>6.993827e+07</td>
      <td>53563.912597</td>
      <td>42.300690</td>
      <td>-71.113972</td>
      <td>1</td>
      <td>Bar</td>
      <td>Pub</td>
      <td>Breakfast Spot</td>
      <td>Grocery Store</td>
      <td>Liquor Store</td>
      <td>Donut Shop</td>
      <td>New American Restaurant</td>
      <td>Pet Store</td>
      <td>Pizza Place</td>
      <td>Rental Car Location</td>
    </tr>
    <tr>
      <th>Jamaica Plain</th>
      <td>28</td>
      <td>2519.245394</td>
      <td>11</td>
      <td>3.94</td>
      <td>1.097379e+08</td>
      <td>56349.937161</td>
      <td>42.317265</td>
      <td>-71.104160</td>
      <td>1</td>
      <td>Brewery</td>
      <td>Coffee Shop</td>
      <td>Gym</td>
      <td>American Restaurant</td>
      <td>Tennis Court</td>
      <td>Shopping Mall</td>
      <td>Chinese Restaurant</td>
      <td>Farmers Market</td>
      <td>Liquor Store</td>
      <td>Art Gallery</td>
    </tr>
    <tr>
      <th>Mission Hill</th>
      <td>29</td>
      <td>350.853564</td>
      <td>13</td>
      <td>0.55</td>
      <td>1.528312e+07</td>
      <td>17918.724113</td>
      <td>42.331341</td>
      <td>-71.095499</td>
      <td>1</td>
      <td>Pizza Place</td>
      <td>Donut Shop</td>
      <td>Furniture / Home Store</td>
      <td>Track</td>
      <td>New American Restaurant</td>
      <td>Liquor Store</td>
      <td>Light Rail Station</td>
      <td>Gym</td>
      <td>Burger Joint</td>
      <td>Italian Restaurant</td>
    </tr>
    <tr>
      <th>Longwood</th>
      <td>30</td>
      <td>188.611947</td>
      <td>28</td>
      <td>0.29</td>
      <td>8.215904e+06</td>
      <td>11908.757148</td>
      <td>42.336046</td>
      <td>-71.099727</td>
      <td>1</td>
      <td>Donut Shop</td>
      <td>Italian Restaurant</td>
      <td>Sandwich Place</td>
      <td>Pizza Place</td>
      <td>Pub</td>
      <td>Gym</td>
      <td>Sushi Restaurant</td>
      <td>Liquor Store</td>
      <td>Bookstore</td>
      <td>Café</td>
    </tr>
    <tr>
      <th>Bay Village</th>
      <td>31</td>
      <td>26.539839</td>
      <td>33</td>
      <td>0.04</td>
      <td>1.156071e+06</td>
      <td>4650.635493</td>
      <td>42.347350</td>
      <td>-71.075727</td>
      <td>1</td>
      <td>American Restaurant</td>
      <td>Gym</td>
      <td>Hotel</td>
      <td>Seafood Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Department Store</td>
      <td>Accessories Store</td>
      <td>Juice Bar</td>
      <td>Plaza</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>Leather District</th>
      <td>32</td>
      <td>15.639908</td>
      <td>27</td>
      <td>0.02</td>
      <td>6.812717e+05</td>
      <td>3237.140537</td>
      <td>42.351922</td>
      <td>-71.055070</td>
      <td>1</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Chinese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Bakery</td>
      <td>Food Truck</td>
      <td>American Restaurant</td>
      <td>Café</td>
      <td>Park</td>
      <td>Dive Bar</td>
    </tr>
    <tr>
      <th>Chinatown</th>
      <td>33</td>
      <td>76.324410</td>
      <td>26</td>
      <td>0.12</td>
      <td>3.324678e+06</td>
      <td>9736.590413</td>
      <td>42.352392</td>
      <td>-71.062573</td>
      <td>1</td>
      <td>Chinese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Bakery</td>
      <td>Coffee Shop</td>
      <td>Theater</td>
      <td>Performing Arts Venue</td>
      <td>Pizza Place</td>
      <td>Sandwich Place</td>
      <td>Seafood Restaurant</td>
      <td>Sushi Restaurant</td>
    </tr>
    <tr>
      <th>Roxbury</th>
      <td>35</td>
      <td>2108.469072</td>
      <td>16</td>
      <td>3.29</td>
      <td>9.184455e+07</td>
      <td>49488.800485</td>
      <td>42.331341</td>
      <td>-71.095499</td>
      <td>1</td>
      <td>Pizza Place</td>
      <td>Donut Shop</td>
      <td>Furniture / Home Store</td>
      <td>Track</td>
      <td>New American Restaurant</td>
      <td>Liquor Store</td>
      <td>Light Rail Station</td>
      <td>Gym</td>
      <td>Burger Joint</td>
      <td>Italian Restaurant</td>
    </tr>
    <tr>
      <th>Back Bay</th>
      <td>37</td>
      <td>399.314411</td>
      <td>2</td>
      <td>0.62</td>
      <td>1.739407e+07</td>
      <td>19455.671146</td>
      <td>42.347350</td>
      <td>-71.075727</td>
      <td>1</td>
      <td>American Restaurant</td>
      <td>Gym</td>
      <td>Hotel</td>
      <td>Seafood Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Department Store</td>
      <td>Accessories Store</td>
      <td>Juice Bar</td>
      <td>Plaza</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>Charlestown</th>
      <td>39</td>
      <td>871.541223</td>
      <td>4</td>
      <td>1.36</td>
      <td>3.796418e+07</td>
      <td>57509.688645</td>
      <td>42.373678</td>
      <td>-71.069654</td>
      <td>1</td>
      <td>American Restaurant</td>
      <td>Coffee Shop</td>
      <td>Skate Park</td>
      <td>Convenience Store</td>
      <td>Gastropub</td>
      <td>Pet Store</td>
      <td>Park</td>
      <td>Chinese Restaurant</td>
      <td>Donut Shop</td>
      <td>Shopping Mall</td>
    </tr>
    <tr>
      <th>West End</th>
      <td>40</td>
      <td>190.490732</td>
      <td>31</td>
      <td>0.30</td>
      <td>8.297743e+06</td>
      <td>17728.590027</td>
      <td>42.366817</td>
      <td>-71.067777</td>
      <td>1</td>
      <td>Science Museum</td>
      <td>Donut Shop</td>
      <td>Bar</td>
      <td>Pizza Place</td>
      <td>Park</td>
      <td>Café</td>
      <td>Zoo Exhibit</td>
      <td>Food Truck</td>
      <td>Planetarium</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>Beacon Hill</th>
      <td>41</td>
      <td>200.156904</td>
      <td>30</td>
      <td>0.31</td>
      <td>8.718800e+06</td>
      <td>14303.829017</td>
      <td>42.355820</td>
      <td>-71.062826</td>
      <td>1</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>American Restaurant</td>
      <td>New American Restaurant</td>
      <td>Italian Restaurant</td>
      <td>Seafood Restaurant</td>
      <td>Hotel</td>
      <td>Falafel Restaurant</td>
      <td>Historic Site</td>
      <td>Steakhouse</td>
    </tr>
    <tr>
      <th>West Roxbury</th>
      <td>45</td>
      <td>3516.421786</td>
      <td>19</td>
      <td>5.49</td>
      <td>1.531747e+08</td>
      <td>66067.419838</td>
      <td>42.286640</td>
      <td>-71.145578</td>
      <td>1</td>
      <td>Convenience Store</td>
      <td>BBQ Joint</td>
      <td>Mexican Restaurant</td>
      <td>Clothing Store</td>
      <td>Spa</td>
      <td>Bar</td>
      <td>Indian Restaurant</td>
      <td>Vietnamese Restaurant</td>
      <td>American Restaurant</td>
      <td>Asian Restaurant</td>
    </tr>
    <tr>
      <th>Hyde Park</th>
      <td>46</td>
      <td>2927.221168</td>
      <td>10</td>
      <td>4.57</td>
      <td>1.275092e+08</td>
      <td>66861.244955</td>
      <td>42.254750</td>
      <td>-71.125585</td>
      <td>1</td>
      <td>American Restaurant</td>
      <td>Ice Cream Shop</td>
      <td>Theater</td>
      <td>Pizza Place</td>
      <td>Pharmacy</td>
      <td>Discount Store</td>
      <td>Donut Shop</td>
      <td>Grocery Store</td>
      <td>Bar</td>
      <td>Fried Chicken Joint</td>
    </tr>
    <tr>
      <th>Dorchester</th>
      <td>48</td>
      <td>4662.879457</td>
      <td>6</td>
      <td>7.29</td>
      <td>2.031142e+08</td>
      <td>104344.034005</td>
      <td>42.293128</td>
      <td>-71.065803</td>
      <td>1</td>
      <td>Park</td>
      <td>Metro Station</td>
      <td>Chinese Restaurant</td>
      <td>Liquor Store</td>
      <td>Dog Run</td>
      <td>Food</td>
      <td>Fast Food Restaurant</td>
      <td>Farmers Market</td>
      <td>Falafel Restaurant</td>
      <td>Donut Shop</td>
    </tr>
    <tr>
      <th>South Boston Waterfront</th>
      <td>49</td>
      <td>621.843524</td>
      <td>29</td>
      <td>0.97</td>
      <td>2.708740e+07</td>
      <td>38391.352905</td>
      <td>42.351922</td>
      <td>-71.055070</td>
      <td>1</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Chinese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Bakery</td>
      <td>Food Truck</td>
      <td>American Restaurant</td>
      <td>Café</td>
      <td>Park</td>
      <td>Dive Bar</td>
    </tr>
    <tr>
      <th>South Boston</th>
      <td>50</td>
      <td>1439.888807</td>
      <td>17</td>
      <td>2.25</td>
      <td>6.272131e+07</td>
      <td>64998.420283</td>
      <td>42.351922</td>
      <td>-71.055070</td>
      <td>1</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Chinese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Bakery</td>
      <td>Food Truck</td>
      <td>American Restaurant</td>
      <td>Café</td>
      <td>Park</td>
      <td>Dive Bar</td>
    </tr>
    <tr>
      <th>Allston</th>
      <td>51</td>
      <td>998.534479</td>
      <td>24</td>
      <td>1.56</td>
      <td>4.349599e+07</td>
      <td>37859.091242</td>
      <td>42.348688</td>
      <td>-71.138024</td>
      <td>1</td>
      <td>Bar</td>
      <td>Gym / Fitness Center</td>
      <td>Chinese Restaurant</td>
      <td>Gastropub</td>
      <td>Dog Run</td>
      <td>Liquor Store</td>
      <td>Coffee Shop</td>
      <td>Food</td>
      <td>Plaza</td>
      <td>Athletics &amp; Sports</td>
    </tr>
    <tr>
      <th>Harbor Islands</th>
      <td>52</td>
      <td>824.888658</td>
      <td>22</td>
      <td>1.29</td>
      <td>3.593201e+07</td>
      <td>92482.183568</td>
      <td>42.390501</td>
      <td>-70.997123</td>
      <td>1</td>
      <td>Park</td>
      <td>River</td>
      <td>Metro Station</td>
      <td>Business Service</td>
      <td>Colombian Restaurant</td>
      <td>Ski Area</td>
      <td>Dive Bar</td>
      <td>Farmers Market</td>
      <td>Falafel Restaurant</td>
      <td>Donut Shop</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Cluster 2
bos_merged.loc[bos_merged['Cluster Labels'] == 2]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OBJECTID</th>
      <th>Acres</th>
      <th>Neighborhood_ID</th>
      <th>SqMiles</th>
      <th>ShapeSTArea</th>
      <th>ShapeSTLength</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
    <tr>
      <th>Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>South End</th>
      <td>36</td>
      <td>471.535356</td>
      <td>32</td>
      <td>0.74</td>
      <td>2.054000e+07</td>
      <td>17912.333569</td>
      <td>42.347350</td>
      <td>-71.075727</td>
      <td>2</td>
      <td>American Restaurant</td>
      <td>Gym</td>
      <td>Hotel</td>
      <td>Seafood Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Department Store</td>
      <td>Accessories Store</td>
      <td>Juice Bar</td>
      <td>Plaza</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>East Boston</th>
      <td>38</td>
      <td>3012.059593</td>
      <td>8</td>
      <td>4.71</td>
      <td>1.313845e+08</td>
      <td>121089.100852</td>
      <td>42.390501</td>
      <td>-70.997123</td>
      <td>2</td>
      <td>Park</td>
      <td>River</td>
      <td>Metro Station</td>
      <td>Business Service</td>
      <td>Colombian Restaurant</td>
      <td>Ski Area</td>
      <td>Dive Bar</td>
      <td>Farmers Market</td>
      <td>Falafel Restaurant</td>
      <td>Donut Shop</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Cluster 3
bos_merged.loc[bos_merged['Cluster Labels'] == 3]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OBJECTID</th>
      <th>Acres</th>
      <th>Neighborhood_ID</th>
      <th>SqMiles</th>
      <th>ShapeSTArea</th>
      <th>ShapeSTLength</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
    <tr>
      <th>Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Fenway</th>
      <td>43</td>
      <td>560.618461</td>
      <td>34</td>
      <td>0.88</td>
      <td>2.442044e+07</td>
      <td>24620.876452</td>
      <td>42.345399</td>
      <td>-71.104332</td>
      <td>3</td>
      <td>American Restaurant</td>
      <td>Furniture / Home Store</td>
      <td>Mexican Restaurant</td>
      <td>Café</td>
      <td>Chinese Restaurant</td>
      <td>Bakery</td>
      <td>Greek Restaurant</td>
      <td>Thai Restaurant</td>
      <td>Cycle Studio</td>
      <td>Movie Theater</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Cluster 4
bos_merged.loc[bos_merged['Cluster Labels'] == 4]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OBJECTID</th>
      <th>Acres</th>
      <th>Neighborhood_ID</th>
      <th>SqMiles</th>
      <th>ShapeSTArea</th>
      <th>ShapeSTLength</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
    <tr>
      <th>Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>North End</th>
      <td>34</td>
      <td>126.910439</td>
      <td>14</td>
      <td>0.2</td>
      <td>5.527506e+06</td>
      <td>16177.826815</td>
      <td>42.366352</td>
      <td>-71.06215</td>
      <td>4</td>
      <td>Pizza Place</td>
      <td>Hotel</td>
      <td>Donut Shop</td>
      <td>Italian Restaurant</td>
      <td>Sandwich Place</td>
      <td>Bar</td>
      <td>Brewery</td>
      <td>Mexican Restaurant</td>
      <td>Sports Bar</td>
      <td>Coffee Shop</td>
    </tr>
  </tbody>
</table>
</div>



## Discussion

**In this notebook, analysis of neighborhood recommendations based on Food venue category has been presented. Based on the analysis above, Chinese restaurants appear in Cluster 0, 1 and 3. In Chinatown, Chinese restaurant is the most common venue, which is pretty resonable, and in Dorchester, South Boston Waterfront, South Boston and Allston, Chinese restaurant is the third most common venue. Therefore, apart from Chinatown, which is an obvious option for opening a Chinese restaurant, neighborhoods like Dorchester, South Boston Waterfront, South Boston and Allston could also be reasonable options.**
