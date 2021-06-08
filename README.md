# Lattitude


We have built a dataframe of the postal code of each neighborhood along with the borough name and neighborhood name, in order to utilize the Foursquare location data, we need to get the latitude and the longitude coordinates of each neighborhood.

In [1]:
import pandas as pd
import requests
from bs4 import BeautifulSoup
In [2]:
List_url = "https://en.wikipedia.org/wiki/List_of_postal_codes_of_Canada:_M"
source = requests.get(List_url).text
In [3]:
soup = BeautifulSoup(source, 'xml')
In [4]:
table=soup.find('table')
In [5]:
column_names=['Postalcode','Borough','Neighbourhood']
df = pd.DataFrame(columns=column_names)
In [6]:
for tr_cell in table.find_all('tr'):
    row_data=[]
    for td_cell in tr_cell.find_all('td'):
        row_data.append(td_cell.text.strip())
    if len(row_data)==3:
        df.loc[len(df)] = row_data
In [7]:
df.head()
Out[7]:
Postalcode	Borough	Neighbourhood
0	M1A	Not assigned	Not assigned
1	M2A	Not assigned	Not assigned
2	M3A	North York	Parkwoods
3	M4A	North York	Victoria Village
4	M5A	Downtown Toronto	Harbourfront
In [8]:
df=df[df['Borough']!='Not assigned']
In [9]:
df[df['Neighbourhood']=='Not assigned']=df['Borough']
df.head()
Out[9]:
Postalcode	Borough	Neighbourhood
2	M3A	North York	Parkwoods
3	M4A	North York	Victoria Village
4	M5A	Downtown Toronto	Harbourfront
5	M5A	Downtown Toronto	Regent Park
6	M6A	North York	Lawrence Heights
In [10]:
temp_df=df.groupby('Postalcode')['Neighbourhood'].apply(lambda x: "%s" % ', '.join(x))
temp_df=temp_df.reset_index(drop=False)
temp_df.rename(columns={'Neighbourhood':'Neighbourhood_joined'},inplace=True)
In [11]:
df_merge = pd.merge(df, temp_df, on='Postalcode')
In [12]:
df_merge.drop(['Neighbourhood'],axis=1,inplace=True)
In [13]:
df_merge.drop_duplicates(inplace=True)
In [15]:
df_merge.rename(columns={'Neighbourhood_joined':'Neighbourhood'},inplace=True)
df_merge.head()
Out[15]:
Postalcode	Borough	Neighbourhood
0	M3A	North York	Parkwoods
1	M4A	North York	Victoria Village
2	M5A	Downtown Toronto	Harbourfront, Regent Park
4	M6A	North York	Lawrence Heights, Lawrence Manor
6	Queen's Park	Queen's Park	Queen's Park
In [16]:
df_merge.shape
Out[16]:
(103, 3)
In [17]:
def get_geocode(postal_code):
    # initialize your variable to None
    lat_lng_coords = None
    while(lat_lng_coords is None):
        g = geocoder.google('{}, Toronto, Ontario'.format(postal_code))
        lat_lng_coords = g.latlng
    latitude = lat_lng_coords[0]
    longitude = lat_lng_coords[1]
    return latitude,longitude
In [18]:
geo_df=pd.read_csv('http://cocl.us/Geospatial_data')
In [19]:
geo_df.head()
Out[19]:
Postal Code	Latitude	Longitude
0	M1B	43.806686	-79.194353
1	M1C	43.784535	-79.160497
2	M1E	43.763573	-79.188711
3	M1G	43.770992	-79.216917
4	M1H	43.773136	-79.239476
In [20]:
geo_df.rename(columns={'Postal Code':'Postalcode'},inplace=True)
geo_merged = pd.merge(geo_df, df_merge, on='Postalcode')
In [21]:
geo_data=geo_merged[['Postalcode','Borough','Neighbourhood','Latitude','Longitude']]
In [22]:
geo_data.head()
Out[22]:
Postalcode	Borough	Neighbourhood	Latitude	Longitude
0	M1B	Scarborough	Rouge, Malvern	43.806686	-79.194353
1	M1C	Scarborough	Highland Creek, Rouge Hill, Port Union	43.784535	-79.160497
2	M1E	Scarborough	Guildwood, Morningside, West Hill	43.763573	-79.188711
3	M1G	Scarborough	Woburn	43.770992	-79.216917
4	M1H	Scarborough	Cedarbrae	43.773136	-79.239476
In [ ]:
