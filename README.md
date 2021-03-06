## How to create choropleth map with Plotly
![Image of average income 2015 Vancouver](https://github.com/chestnutcone/plotly_choropleth_tutorial/blob/master/2015_vancouver_average_income.png)

Visualization is a key part when working with data. I was working with data from Census Canada and I wanted graph the average income distribution of Vancouver, but I didn't know how to do it. Well, if you are in the same boat, no worries, I can get you up to speed.

I will be graphing average and median income of 2015 Vancouver. The data is separated into neighborhoods and I got it from [open data Vancouver](https://opendata.vancouver.ca/explore/dataset/census-local-area-profiles-2016/information/). You can also find the necessary files and Jupyter notebook on my [Github](https://github.com/chestnutcone/plotly_choropleth_tutorial) so you can follow along.

### GeoJSON

First, we need GeoJSON of the Vancouver neighborhood. My understanding is that GeoJSON contains information of shapes on the map. I Googled for Vancouver neighborhood GeoJSON and got the GeoJSON file from this [website](https://audreychowster.carto.com/tables/vancouver_neighbourhoods/public/map). You can find more about GeoJSON file format on [Wikipedia](https://en.wikipedia.org/wiki/GeoJSON). 

My quick overview: it contains two keys: 'type' and 'features'. The value of 'features' is a list of dictionaries containing information on the shape of the region. Each feature also has a 'properties' information. I will give an example:

```import pandas as pd
import geopandas as gpd
import json
import chart_studio.plotly as py
import plotly.figure_factory as ff
import plotly.graph_objects as go
import plotly.io as pio
# opening geojson file
with open("cov_localareas.geojson", 'r') as f:
    van_geojson = json.load(f)
print(van_geojson['features'][0].keys())
``` 

```
out: 
dict_keys(['type', 'geometry', 'properties'])
```

```
print(van_geojson['features'][0]['properties'])
```

```out: 
{'cartodb_id': 7,
 'draworder': None,
 'visibility': -1,
 'extrude': 0,
 'tessellate': -1,
 'icon': None,
 'altitudemode': None,
 'description': None,
 'name': 'Oakridge',
 '_end': None,
 'begin': None,
 'timestamp': None}
 ``` 

In my case, the neighborhood name is stored underneath the properties. However, this GeoJSON is also missing a key of 'id' in each feature. The 'id' helps Plotly to map your data to the map area. I will use the neighborhood name as 'id' 

```
for feature in van_geojson['features']:
    feature['id'] = feature['properties']['name']
print(van_geojson['features'][0].keys())
print(van_geojson['features'][0]['id']) # verify we have an id
```

```
out:
dict_keys(['type', 'geometry', 'properties', 'id'])
Oakridge
```

Good, now we can look at the data

### Data
I will exclude the data processing. It is pretty simple: load data into pandas dataframe, strip the column names (since it contains leading and trailing spaces), got the slice (panda series) of data I need, and turn it into dtype of int32. Below is the output of the data I will be using. Note the index is the neighborhood names which will match to the 'id' of our GeoJSON.

```
Arbutus-Ridge                62675
Downtown                     63251
Dunbar-Southlands            78117
Fairview                     61627
Grandview-Woodland           42896
Hastings-Sunrise             38258
Kensington-Cedar Cottage     38411
Kerrisdale                   77248
Killarney                    39013
Kitsilano                    63092
Marpole                      39020
Mount Pleasant               54260
Oakridge                     46515
Renfrew-Collingwood          33360
Riley Park                   53060
Shaughnessy                 118668
South Cambie                 65459
Strathcona                   31534
Sunset                       34212
Victoria-Fraserview          34298
West End                     47253
West Point Grey              82042
Name: 2, dtype: int32
```

### Plot with Plotly
[View code on gist](https://gist.github.com/chestnutcone/68b1e992de68c6850a0cbe354b4db620)
```
fig = go.Figure() # empty figure

# first figure
fig.add_trace(
    go.Choroplethmapbox(geojson=van_geojson, # geojson with 'id'
                        locations=average_income.index,   # index is the neighborhood names, which will match to 'id' of geojson
                        z=average_income, # values of data you want to plot
                        colorscale='Magma', # your colorscale
                        marker_opacity=0.5, 
                        marker_line_width=0, 
                        name='2015 Vancouver Average Income'
    )
)
    
# updates the layout
fig.update_layout(
    updatemenus=[
        go.layout.Updatemenu(
            active=0,
            buttons=[
                dict(label='Average income',
                     method='update',
                     args=[{'visible':[True]}, # graph visibility, comes into play when doing multiple graph
                           {'title': "Average income"}]), # title
            ]
        )
    ]
)


fig.update_layout(mapbox_style="carto-positron",
                 mapbox_zoom=11, # how much zoom you want at the start
                  mapbox_center={'lat': 49.24, 'lon':-123.1}) # center of the map at the start
fig.update_layout(margin={"r":0,"t":30,"l":0,"b":0}) # margins
fig.update_layout(title_text='2015 Vancouver Average Income') #title
fig.show()
```

[View Plotly](https://chestnutcone.github.io/plotly_choropleth_tutorial/vancouver_average_income.html)

And there you have it, an interactive choropleth. What if you want to plot two graphs in one?

[View code on gist](https://gist.github.com/chestnutcone/2865518b70fd8a97ef69eddfda672b7b)

```
fig = go.Figure() # empty figure

# plotting average income
fig.add_trace(
    go.Choroplethmapbox(geojson=van_geojson,
                        locations=average_income.index,   
                        z=average_income, 
                        colorscale='Magma', 
                        marker_opacity=0.5, 
                        marker_line_width=0, 
                        name='2015 Vancouver Average Income'
    )
)

# plotting median income
fig.add_trace(
    go.Choroplethmapbox(geojson=van_geojson, # geojson with 'id'
                        locations=median_income.index,
                        z=median_income, 
                        colorscale='Magma', 
                        marker_opacity=0.5, 
                        marker_line_width=0, 
                        name='2015 Vancouver Median Income',
                        visible=False, # this is important because you don't want the graphs to overlap at the start
    )
)
    
# updates the layout
fig.update_layout(
    updatemenus=[
        go.layout.Updatemenu(
            active=0,
            buttons=[
                dict(label='Average income',
                     method='update',
                     args=[{'visible':[True, False]}, # graph visibility
                           {'title': "Average income"}]),
                dict(label='Median income', 
                     method='update',
                     args=[{'visible':[False, True]}, # want to show only the second graph when clicking button
                           {'title': "Median income"}]),
            ]
        )
    ]
)


fig.update_layout(mapbox_style="carto-positron",
                 mapbox_zoom=11, 
                  mapbox_center={'lat': 49.24, 'lon':-123.1}) 
fig.update_layout(margin={"r":0,"t":30,"l":0,"b":0})
fig.update_layout(title_text='2015 Vancouver Average Income') #title of the first graph that appears
fig.show()
```

[View Plotly with Two Graphs](https://chestnutcone.github.io/plotly_choropleth_tutorial/vancouver_average_median_income.html)

That is it!
