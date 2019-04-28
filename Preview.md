
# Rosario bus stops
<p>
    Application based on my studies of the University of California course Python for Data Science, from The University of California, San Diego, on the edX platform.
    The notebook displays on a map of the city of Rosario the bus stops. Where is allowed to choose a bus line to see its round-trip route on the map.
</p>
<p>
    The process of data acquisition, selection and cleaning is shown, as other relevant operations such as joining and filtering of data sets.
</p>
<p>
    <a href="https://courses.edx.org/certificates/user/77345/course/course-v1:UCSanDiegoX+DSE200x+1T2019">My certificate in Python for Data Science"</a>
</p>
<p>
    <a href="https://credentials.edx.org/records/programs/482dee71e4b94b42a47b3e16bb69e8f2/">My MicroMasters in Data Science - University of California, San Diego records</a>
</p>

## Datasets
The data used belongs to the open data platform RosarioDatos, provided by the Rosario City Hall<br>
<a href=https://datos.rosario.gob.ar>RosarioDatos</a>


```python
import pandas as pd
import numpy as np
import folium
from folium.features import DivIcon
import ipywidgets as widgets
from ipywidgets import interact, interactive
from IPython.display import HTML
```

### Extraction of pertinent data
<p>The csv file is loaded and the attributes are displayed. Then a list of properties is made to leave the appropriate columns for the task.</p>


```python
# loading data from a csv
paradas = pd.read_csv('./data/paradas_tup_json.csv')
```


```python
# data attributes
print(type(paradas))
print(paradas.shape)
paradas.columns
```

    <class 'pandas.core.frame.DataFrame'>
    (10010, 20)





    Index(['SE_ROW_ID', 'MSLINK', 'PARADA', 'RAMAL', 'SENTIDO', 'ORDEN',
           'ID_PARADA', 'CALLE_UNO', 'CALLE_DOS', 'COD_SMS', 'OCHAVA', 'REFUGIO',
           'DISTRITO', 'PUNTO_X', 'PUNTO_Y', 'CAMBIO', 'USUARIO_CA', 'TYPE',
           'CHECKSUM', 'GEOJSON'],
          dtype='object')




```python
# data sample of the bus stops data file
paradas.head(3)
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
      <th>SE_ROW_ID</th>
      <th>MSLINK</th>
      <th>PARADA</th>
      <th>RAMAL</th>
      <th>SENTIDO</th>
      <th>ORDEN</th>
      <th>ID_PARADA</th>
      <th>CALLE_UNO</th>
      <th>CALLE_DOS</th>
      <th>COD_SMS</th>
      <th>OCHAVA</th>
      <th>REFUGIO</th>
      <th>DISTRITO</th>
      <th>PUNTO_X</th>
      <th>PUNTO_Y</th>
      <th>CAMBIO</th>
      <th>USUARIO_CA</th>
      <th>TYPE</th>
      <th>CHECKSUM</th>
      <th>GEOJSON</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>727</td>
      <td>0</td>
      <td>3152</td>
      <td>45</td>
      <td>V</td>
      <td>0</td>
      <td>3152</td>
      <td>1077</td>
      <td>1930</td>
      <td>9776</td>
      <td>NE</td>
      <td>P</td>
      <td>NOROESTE                                      ...</td>
      <td>-60.759785</td>
      <td>-32.940745</td>
      <td>2014-10-06</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>{ "type": "Feature", "properties": { "SE_ROW_I...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>748</td>
      <td>0</td>
      <td>2924</td>
      <td>58</td>
      <td>V</td>
      <td>399</td>
      <td>2924</td>
      <td>12</td>
      <td>1815</td>
      <td>4300</td>
      <td>N</td>
      <td>I</td>
      <td>NORTE                                         ...</td>
      <td>-60.699092</td>
      <td>-32.873835</td>
      <td>2010-11-17</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>{ "type": "Feature", "properties": { "SE_ROW_I...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>859</td>
      <td>0</td>
      <td>2983</td>
      <td>56</td>
      <td>I</td>
      <td>23</td>
      <td>2983</td>
      <td>1412</td>
      <td>1474</td>
      <td>9163</td>
      <td>NO</td>
      <td>P</td>
      <td>NORTE                                         ...</td>
      <td>-60.691754</td>
      <td>-32.892053</td>
      <td>2010-11-19</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>{ "type": "Feature", "properties": { "SE_ROW_I...</td>
    </tr>
  </tbody>
</table>
</div>




```python
# attributes list
# data tags are renamed for clarity
# columns are rearranged
propiedades = ['ID_PARADA', 'RAMAL', 'SENTIDO', 'PUNTO_X', 'PUNTO_Y']
paradas_df = paradas[propiedades]
paradas_df = paradas_df.rename(index=str, columns={"PUNTO_X": "LONGITUD", "PUNTO_Y": "LATITUD"})
paradas_df = paradas_df[['ID_PARADA', 'RAMAL', 'SENTIDO', 'LATITUD', 'LONGITUD']]
paradas_df.head(3)
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
      <th>ID_PARADA</th>
      <th>RAMAL</th>
      <th>SENTIDO</th>
      <th>LATITUD</th>
      <th>LONGITUD</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3152</td>
      <td>45</td>
      <td>V</td>
      <td>-32.940745</td>
      <td>-60.759785</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2924</td>
      <td>58</td>
      <td>V</td>
      <td>-32.873835</td>
      <td>-60.699092</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2983</td>
      <td>56</td>
      <td>I</td>
      <td>-32.892053</td>
      <td>-60.691754</td>
    </tr>
  </tbody>
</table>
</div>




```python
# The data of the routes are loaded
# Each line has a branch related to its route
recorridos = pd.read_csv('./data/recorridos_tup_json.csv')
print(type(recorridos))
print(recorridos.shape)
recorridos.columns
```

    <class 'pandas.core.frame.DataFrame'>
    (136, 15)





    Index(['SE_ROW_ID', 'MSLINK_ODB', 'LINEA', 'BANDERA', 'SENTIDO', 'ID_TUP',
           'LINEA_BANDERA', 'TIPO', 'FECHA_HORA', 'CHECKSUM', 'OBSERVACIONES',
           'HORARIO1', 'HORARIO2', 'ID_RAMAL', 'GEOJSON'],
          dtype='object')



### Data preparation
<p>Dataframes are joined and filtered to get pertinent data for the map plotting</p>


```python
# The bus stops dataframe is joined with the bus number routes
# Then the stops will be filtered according to the bus number
paradas_df = paradas_df.merge(recorridos[['ID_RAMAL', 'LINEA_BANDERA']], left_on= 'RAMAL', right_on= 'ID_RAMAL' )
del paradas_df['ID_RAMAL']
paradas_df.head(3)
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
      <th>ID_PARADA</th>
      <th>RAMAL</th>
      <th>SENTIDO</th>
      <th>LATITUD</th>
      <th>LONGITUD</th>
      <th>LINEA_BANDERA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3152</td>
      <td>45</td>
      <td>V</td>
      <td>-32.940745</td>
      <td>-60.759785</td>
      <td>142 Negra</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3152</td>
      <td>45</td>
      <td>V</td>
      <td>-32.940745</td>
      <td>-60.759785</td>
      <td>142 Negra</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1445</td>
      <td>45</td>
      <td>V</td>
      <td>-32.940207</td>
      <td>-60.717191</td>
      <td>142 Negra</td>
    </tr>
  </tbody>
</table>
</div>




```python
# a list of all bus denomination is created
lista_lineas = recorridos['LINEA_BANDERA'].unique()
lista_lineas.sort()
```


```python
# to make interaction faster when drawing the map
# the stops of each bus are separated in a dictionary 
# is also saved the average latitude and longitude to show the map centered
layers = {}
for l in lista_lineas:
    mask_ramal = paradas_df['LINEA_BANDERA'] == l
    stage = paradas_df[mask_ramal][['LATITUD', 'LONGITUD', 'SENTIDO']]
    lat_mean= stage['LATITUD'].mean()
    long_mean= stage['LONGITUD'].mean()
    layers.update({l: [stage, lat_mean, long_mean]})
```

## Map making and plotting


```python
# map legend
def add_legend(m):
    legend_html =   '''
                <div style="position: fixed; 
                            top: 50px; right: 50px; padding:5px; 
                            background-color:#E7EFE888;
                            color:#505e62; z-index:9999; font-size:14px;">
                              &nbsp;<i class="fa fa-circle fa-lg" style="color:#EE8601"></i> Ida &nbsp; <br>
                              &nbsp;<i class="fa fa-circle fa-lg" style="color:#00ACEC"></i> Vuelta &nbsp; 
                </div>
                ''' 
         
    m.get_root().html.add_child(folium.Element(legend_html))
```


```python
# the map is made and markers are added
def f(Linea):
    m = folium.Map(
        location=(layers[Linea][1], layers[Linea][2]),
        tiles='cartodbpositron',
        zoom_start=13
        )
    add_legend(m)
    stage = layers[Linea][0]
    for i, r in stage.iterrows():
        color_sentido= '#EE8601' if r['SENTIDO'] == 'I' else '#00ACEC'
        folium.CircleMarker([r['LATITUD'], r['LONGITUD']], 
                            radius=5,
                            weight=1,
                            color=color_sentido,
                            fill=True,
                            fillOpacity=0.01).add_to(m)
    display(m)
```


```javascript
%%javascript
// autoscroll is deactivated
IPython.OutputArea.prototype._should_scroll = function(lines) {
    return false;
}
```


    <IPython.core.display.Javascript object>



```python
# a widget is added to show the list of bus numbers to choose from
w = interactive(f, Linea = lista_lineas)
display(w)
```


    interactive(children=(Dropdown(description='Linea', options=('101 Negra', '101 Roja', '102 Negra', '102 Roja',…

