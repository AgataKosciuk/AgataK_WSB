# Praca dyplomowa Python Deweloper WSB Gdańsk

## TEMAT PRACY: 

## Analiza danych dotyczących wypadków samochodowych w NY:
### Określić najniebezpieczniejsze czynniki wypadków w danej dzielnicy NY
### Dowiedzieć się, ile zgonów oraz obrażeń zostało spowodowanych przez szybką jazdę ogólnie, oraz w danej dzielnicy
### Określić top 5 najniebezpieczniejszych czynników wypadków
### Wygenerować wizualizację określającą ilość wypadków z podziałem na dni tygodnia

#### Pobranie niezbędnych bibliotek przed pobraniem i przygotowaniem danych do analizy

import pandas as pd

import numpy as py

import matplotlib.pyplot as plt

import datetime as dt
from datetime import datetime

import geopandas as gpd

from shapely.geometry import Point

import folium

### 1. Wczytanie danych

df = pd.read_csv

### 2. Czyszczenie/przygotowanie danych:

### - usuwamy kolumny, które nie są nam potrzebne do analizy danych do zadania

motor_data = motor_data.drop(motor_data.columns[[1, 3, 6, 7, 8, 9, 24, 25, 26, 27, 28]], axis=1)

## ZADANIA
## 1. Określić najniebezpieczniejsze czynniki wypadków w danej dzielnicy NY - Wybrana dzielnica QUEENS

df = df[['BOROUGH', 'CONTRIBUTING FACTOR VEHICLE 1', 'CONTRIBUTING FACTOR VEHICLE 2', 'CONTRIBUTING FACTOR VEHICLE 3', 'CONTRIBUTING FACTOR VEHICLE 4', 'CONTRIBUTING FACTOR VEHICLE 5']]

#### - filtrowanie danych tylko dla wybranej dzielnicy

df = df[df['BOROUGH'].str.contains('QUEENS', na=False)]

#### ODPOWIEDŹ: 

#### Wyświetlamy najniebezpieczniejsze przyczyny wypadków w wybranej dzielnicy w kolejności od najczęściej występującej

motor_queens = motor_queens.copy()

factors_columns = ['CONTRIBUTING FACTOR VEHICLE 1', 'CONTRIBUTING FACTOR VEHICLE 2', 'CONTRIBUTING FACTOR VEHICLE 3', 'CONTRIBUTING FACTOR VEHICLE 4', 'CONTRIBUTING FACTOR VEHICLE 5']

motor_queens['FACTORS'] = motor_queens[factors_columns].stack().reset_index(drop=True)

motor_queens['FACTORS'] = motor_queens['FACTORS'][(motor_queens['FACTORS'] != 1) & (motor_queens['FACTORS'] != 80)]

motor_counts = motor_queens.groupby(['BOROUGH', 'FACTORS']).size().reset_index(name='COUNT')

motor_counts = motor_counts.sort_values(by='COUNT', ascending=False)

### ODPOWIEDŹ: Wyświetlamy najniebezpieczniejsze przyczyny wypadków w wybranej dzielnicy w kolejności od najczęściej występującej

BOROUGH	FACTORS	COUNT
54	QUEENS	Unspecified	64914
11	QUEENS	Driver Inattention/Distraction	13314
18	QUEENS	Failure to Yield Right-of-Way	4998
6	QUEENS	Backing Unsafely	2864
21	QUEENS	Following Too Closely	1902
36	QUEENS	Passing or Lane Usage Improper	1388 ...

# 2.A Dowiedzieć się, ile zgonów oraz obrażeń zostało spowodowanych przez szybką jazdę ogólnie

### filtrujemy dane dla wypadków spowodowanych przez szybką jazdę
speed_data = motor_data.copy()
speed_data['FACTORS'] = speed_data[['CONTRIBUTING FACTOR VEHICLE 1', 'CONTRIBUTING FACTOR VEHICLE 2', 'CONTRIBUTING FACTOR VEHICLE 3', 'CONTRIBUTING FACTOR VEHICLE 4', 'CONTRIBUTING FACTOR VEHICLE 5']].values.tolist()
speed_data = speed_data.explode(column='FACTORS')

### grupujemy po czynnikach przyczynowych i obliczamy sumę zgonów i obrażeń
speed_data = speed_data.groupby('BOROUGH').agg({
    'NUMBER OF PERSONS KILLED': 'sum',
    'NUMBER OF PERSONS INJURED': 'sum',
    'NUMBER OF PEDESTRIANS INJURED':'sum',
    'NUMBER OF PEDESTRIANS KILLED':'sum',
    'NUMBER OF CYCLIST INJURED':'sum',
    'NUMBER OF CYCLIST KILLED':'sum',
    'NUMBER OF MOTORIST INJURED':'sum',
    'NUMBER OF MOTORIST KILLED':'sum'
}).reset_index()

### użyłam funkcji pomocniczej do sprawdzenia, czy wartość jest niepustym ciągiem znaków
def is_nonempty_string(value):
    return isinstance(value, str) and value.strip() != ''

speed_factors = ['SPEED', 'Speeding', 'Excessive Speed']

### Filtrujemy dane dla wypadków związanych z przekroczeniem prędkości
speed_data = motor_data[motor_data['CONTRIBUTING FACTOR VEHICLE 1'].apply(lambda x: any(factor in x.upper() for factor in speed_factors) if is_nonempty_string(x) else False)]

### Sumujemy liczby zgonów i obrażeń
total_killed = speed_data['NUMBER OF PEDESTRIANS KILLED'].sum() + speed_data['NUMBER OF CYCLIST KILLED'].sum() + speed_data['NUMBER OF MOTORIST KILLED'].sum()
total_injuries = speed_data['NUMBER OF PEDESTRIANS INJURED'].sum() + speed_data['NUMBER OF CYCLIST INJURED'].sum() + speed_data['NUMBER OF MOTORIST INJURED'].sum()

#ODPOWIEDŹ
display("Liczba zgonów spowodowanych przez przekroczenie prędkości ogólnie:", total_killed)
display("Liczba obrażeń spowodowanych przez przekroczenie prędkości ogólnie:", total_injuries)

'Liczba zgonów spowodowanych przez przekroczenie prędkości ogólnie:'
150
'Liczba obrażeń spowodowanych przez przekroczenie prędkości ogólnie:'
8349
2.B Dowiedzieć się, ile zgonów oraz obrażeń zostało spowodowanych przez szybką jazdę w wybranej dzielnicy Queens


### Filtrujemy dane dla wypadków związanych z przekroczeniem prędkości i dla dzielnicy Queens
speed_data_queens = motor_data[(motor_data['BOROUGH'] == 'QUEENS') & (motor_data['CONTRIBUTING FACTOR VEHICLE 1'].apply(lambda x: any(factor in x.upper() for factor in speed_factors) if is_nonempty_string(x) else False))]

total_killed_queens = speed_data_queens['NUMBER OF PEDESTRIANS KILLED'].sum() + speed_data_queens['NUMBER OF CYCLIST KILLED'].sum() + speed_data_queens['NUMBER OF MOTORIST KILLED'].sum()
total_injuries_queens = speed_data_queens['NUMBER OF PEDESTRIANS INJURED'].sum() + speed_data_queens['NUMBER OF CYCLIST INJURED'].sum() + speed_data_queens['NUMBER OF MOTORIST INJURED'].sum()

# ODPOWIEDŹ
display("Liczba zgonów spowodowanych przez przekroczenie prędkości w dzielnicy Queens:", total_killed_queens)
display("Liczba obrażeń spowodowanych przez przekroczenie prędkości w dzielnicy Queens:", total_injuries_queens)

'Liczba zgonów spowodowanych przez przekroczenie prędkości w dzielnicy Queens:'
27
'Liczba obrażeń spowodowanych przez przekroczenie prędkości w dzielnicy Queens:'
1143

# 3. Określić top 5 najniebezpieczniejszych czynników wypadków wg ilości ich występowania

factor_counts = motor_data.groupby('CONTRIBUTING FACTOR VEHICLE 1').size()

top_5_factors = factor_counts.nlargest(5)

### ODPOWIEDŹ
print("TOP 5 najniebezpieczniejszych czynników danych:")
print(top_5_factors)

TOP 5 najniebezpieczniejszych czynników danych:
CONTRIBUTING FACTOR VEHICLE 1
Unspecified                       595805
Driver Inattention/Distraction    299425
Failure to Yield Right-of-Way      91617
Following Too Closely              78467
Backing Unsafely                   61445
dtype: int64

# 4. Wygenerować wizualizację określającą ilość wypadków z podziałem na dni tygodnia

### wybieramy interesującą nas kolumnę z danymi

motor_data['ACCIDENT DATE']

### konwertujemy argumenty na datatime

motor_data['ACCIDENT DATE'] = pd.to_datetime(motor_data['ACCIDENT DATE'])

### zmieniamy format daty dd/mm/rr

motor_data['formatted_date'] = motor_data['ACCIDENT DATE'].dt.strftime('%d/%m/%Y %H:%M:%S')

### przypisujemy do daty dni tygodnia

motor_data['DAY OF WEEK'] = motor_data['ACCIDENT DATE'].dt.day_name()

### zliczamy dni tygodnia

day_counts = motor_data['DAY OF WEEK'].value_counts()
display(day_counts)
DAY OF WEEK
Friday       256236
Thursday     242307
Tuesday      240216
Wednesday    237040
Monday       231946
Saturday     214607
Sunday       189826
Name: count, dtype: int64

### generujemy wizualizację

day_counts.plot(kind='bar')
plt.title('Number of accidents by day of the week')
plt.xlabel('Day of week')
plt.ylabel('Number of accidents')

plt.show()

# ZADANIE DODATKOWE: Wygeneruj mapę z miejscami wypadków w danej dzielnicy.
### Filtrujemy dane dla dzielnicy Queens i usuwamy wiersze z brakującymi wartościami LATITUDE / LONGITUDE
queens_data = motor_data[(motor_data['BOROUGH'] == 'QUEENS') & (~motor_data['LATITUDE'].isnull()) & (~motor_data['LONGITUDE'].isnull())]

### Ograniczamy liczbę punktów na mapie w celu przyspieszenia procesu
max_points = 50
queens_data = queens_data.head(max_points)

### Tworzymy mapę - Współrzędne Queens
queens_map = folium.Map(location=[40.7282, -73.7949], zoom_start=11)  

### Dodajemy znaczniki dla wybranej ilości wypadków
for index, row in queens_data.iterrows():
    location = [row['LATITUDE'], row['LONGITUDE']]
    folium.Marker(location).add_to(queens_map)
queens_map
