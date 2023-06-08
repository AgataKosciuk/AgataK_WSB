# Praca dyplomowa Python Deweloper WSB Gdańsk

## TEMAT PRACY: 

## Analiza danych dotyczących wypadków samochodowych w NY:
### Określić najniebezpieczniejsze czynniki wypadków w danej dzielnicy NY
### Dowiedzieć się, ile zgonów oraz obrażeń zostało spowodowanych przez szybką jazdę ogólnie, oraz w danej dzielnicy
### Określić top 5 najniebezpieczniejszych czynników wypadków
### Wygenerować wizualizację określającą ilość wypadków z podziałem na dni tygodnia

#### Pobranie niezbędnych bibliotek przed pobraniem i przygotowaniem danych do analizy

import pandas as pd

import numpy as np

import matplotlib.pyplot as plt

import datetime as dt

from datetime import datetime

import geopandas as gpd

from shapely.geometry import Point

### 1. Wczytanie danych

df = pd.read_csv

### 2. Czyszczenie/przygotowanie danych:

### - usuwamy kolumny zawierające puste dane

df.dropna(axis=1,how='all', subset=29, inplace=True)

### - usuwamy wiersze zawierające puste dane

df.dropna(axis=0,how='any', inplace=True)

### - usuwamy duplikaty danych

df.drop_duplicates(subset=None, keep=False, inplace=True, ignore_index=True)

### - usuwamy kolumny, które nie są nam potrzebne do analizy danych do zadania

df= df.drop(df.columns[[1, 3, 4, 5, 6, 7, 8, 18, 19]], axis=1)

## ZADANIA
## 1. Określić najniebezpieczniejsze czynniki wypadków w danej dzielnicy NY - Wybrana dzielnica QUEENS

#### - filtrowanie danych tylko dla wybranej dzielnicy

df = df[df['BOROUGH'].str.contains('QUEENS', na=False)]

#### ODPOWIEDŹ: 

#### Wyświetlamy najniebezpieczniejsze przyczyny wypadków w wybranej dzielnicy w kolejności od najczęściej występującej

#### - grupujemy i wyświetlamy przyczyny wypadków

df = df.groupby('BOROUGH')['CONTRIBUTING FACTOR VEHICLE 1'].value_counts().nlargest(5)

## 2.A Dowiedzieć się, ile zgonów oraz obrażeń zostało spowodowanych przez szybką jazdę ogólnie

#### filtrujemy dane, szukając tego co nas interesuje

df = df[df['CONTRIBUTING FACTOR VEHICLE 1'].str.contains('Speed', na=False)].sum()                       

#### ODPOWIEDŹ: Przez szybką jazdę zostało spowodowanych 3471 zgonów i obrażeń

#### - sumujemy dane z konkretnych kolumn

df['Total'] = df['NUMBER OF PERSONS INJURED'] + df['NUMBER OF PEDESTRIANS INJURED'] + df['NUMBER OF CYCLIST INJURED'] + df['NUMBER OF MOTORIST INJURED'] + df['NUMBER OF PERSONS KILLED'] + df['NUMBER OF PEDESTRIANS KILLED'] + df['NUMBER OF CYCLIST KILLED'] + df['NUMBER OF MOTORIST KILLED']

## 2.B Dowiedzieć się, ile zgonów oraz obrażeń zostało spowodowanych przez szybką jazdę w wybranej dzielnicy Queens

#### - filtrujemy dane przez interesujące nas wartości

df = df[df['BOROUGH'].str.contains('QUEENS', na=False)].sum()

#### ODPOWIEDŹ: Przez szybką jazdę w dzielnicy Queens zostało spowodowanych 1035 zgonów i obrażeń

#### - sumujemy konkretne kolumny w celu otrzymania wyniku

df['Total'] = df['NUMBER OF PERSONS INJURED'] + df['NUMBER OF PEDESTRIANS INJURED'] + df['NUMBER OF CYCLIST INJURED'] + df['NUMBER OF MOTORIST INJURED'] + df['NUMBER OF PERSONS KILLED'] + df['NUMBER OF PEDESTRIANS KILLED'] + df['NUMBER OF CYCLIST KILLED'] + df['NUMBER OF MOTORIST KILLED']

## 3. Określić top 5 najniebezpieczniejszych czynników wypadków

#### ODPOWIEDŹ: TOP 5 najniebezpieczniejszych czynników danych wg ilości ich występowania

#### - wyszukujemy i zliczamy interesujące nas dane

df = df.value_counts('CONTRIBUTING FACTOR VEHICLE 1').nlargest(5)

## 4. Wygenerować wizualizację określającą ilość wypadków z podziałem na dni tygodnia

#### - wybieramy interesującą nas kolumnę z danymi

df['ACCIDENT DATE']

#### - konwertujemy argumenty na datatime

df['ACCIDENT DATE'] = pd.to_datetime(df['ACCIDENT DATE'])

#### - zmieniamy format daty dd/mm/rr

df['formatted_date'] = df['ACCIDENT DATE'].dt.strftime('%d/%m/%Y %H:%M:%S')

#### - przypisujemy do daty dni tygodnia

df['DAY OF WEEK'] = df['ACCIDENT DATE'].dt.day_name()

#### - zliczamy dni tygodnia

day_counts = df['DAY OF WEEK'].value_counts()

#### - generujemy wizualizację

day_counts.plot(kind='bar')
plt.title('Number of accidents by day of the week')
plt.xlabel('Day of week')
plt.ylabel('Number of accidents')
plt.show()

## ZADANIE DODATKOWE 
### Wygeneruj mapę z miejscami wypadków w danej dzielnicy.

motor_queens= motor_data[motor_data['BOROUGH'].str.contains('QUEENS', na=False)]

groupped = motor_queens.value_counts('LATITUDE').nlargest(9)
groupped1 = motor_queens.value_counts('LONGITUDE').nlargest(9)

data_dict = {'x': {0: -73.870350, 1: -73.870369, 2: -73.822250, 3: -73.767360, 4: -73.937386, 5: -73.773828, 6: -73.757110, 7: -73.834656, 8: -73.887560},
 'y': {0: 40.733536, 1: 40.733497, 2: 40.665257, 3: 40.656160, 4:40.748924, 5: 40.666120, 6: 40.659651, 7: 40.684240, 8: 40.76229040}}

df = pd.DataFrame(data_dict)
geometry = [Point(xy) for xy in zip(df['x'], df['y'])]
gdf = gpd.GeoDataFrame(df, geometry=geometry, crs=4326)
print(gdf.head())
print(gdf.dtypes)
gdf.explore()

