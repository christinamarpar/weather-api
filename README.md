# weather-api

# Dependencies
import json
import requests
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from citipy import citipy
from config import api_key, g_key
from pprint import pprint

# Save config information
url = "http://api.openweathermap.org/data/2.5/weather?"
#I fill an array of cities with 500 evenly spaced cities; 
cities = []
city_lats = []
city_lngs = []
temperatures = []
humidities = []
cloudiness = []
windspeeds = []

lats = np.linspace(-90,90,45)
lngs = np.linspace(-180,180,45)

count = 0

for lat in lats:
    for lng in lngs:
        city = citipy.nearest_city(lat, lng)
        city_name = city.city_name + ", " + city.country_code.upper()
        if (city_name not in cities):
            try:
                query_url = url + "appid=" + api_key + "&q=" + city_name + "&units=imperial"
                weather_response = requests.get(query_url)
                weather_json = weather_response.json()
                #pprint(weather_json)
                temperatures.append(weather_json['main']['temp'])
                humidities.append(weather_json['main']['humidity'])
                cloudiness.append(weather_json['clouds']['all'])
                windspeeds.append(weather_json['wind']['speed'])

                #I add city to my list after checking that the weather API was able to find data on the city
                print(str(count+1) + ": " + city_name)
                print(weather_response.url)
                cities.append(city_name)

                #Now, I search for that city's actual, more precise lat and lng using the google geocode wrapper
                params = {"key": g_key,
                        "address":city_name}
                base_url = "https://maps.googleapis.com/maps/api/geocode/json"
                city_lat_lng = requests.get(base_url, params=params).json()

                city_lats.append(city_lat_lng["results"][0]["geometry"]["location"]["lat"])
                city_lngs.append(city_lat_lng["results"][0]["geometry"]["location"]["lng"])

                count +=1

            except:
                #print("HERE WITHIN THE EXCEPT STATEMENT!!!!")
                lens = [len(cities),len(city_lats),len(city_lngs),len(temperatures),
                len(humidities),len(cloudiness),len(windspeeds)]
                min_len = min(lens)

                for x in [cities, city_lats, city_lngs, temperatures, humidities, cloudiness, windspeeds]:
                    if len(x)>min_len:
                        x.pop()
                pass

#print(f"cities: {len(cities)}")
#print(f"cities: {len(city_lats)}")
#print(f"cities: {len(city_lngs)}")
#print(f"cities: {len(temperatures)}")
#print(f"cities: {len(humidities)}")
#print(f"cities: {len(cloudiness)}")
#print(f"cities: {len(windspeeds)}")

city_pd = pd.DataFrame({
    "City": cities,
    "Latitude": city_lats,
    "Longitude": city_lngs,
    "Temperature": temperatures,
    "Humidity": humidities,
    "Cloudiness": cloudiness,
    "Wind Speed": windspeeds
})

#print(city_pd.head())
city_pd.to_csv("weather_data.csv", encoding="utf-8", index=False, mode="w")

#Create my scatter plots
for n, characteristic in zip(["Temperature (F)", "Humidity (%)", "Cloudiness (%)", "Wind Speed (mph)"],["Temperature", "Humidity", "Cloudiness", "Wind Speed"]) :

    plt.scatter(x=city_lats,y=city_pd[f"{characteristic}"],edgecolor="black", marker="o")

    #Incorporate the other graph properties
    plt.title(f"City Latitude vs. {characteristic} (Apr 1, 2018)")
    plt.ylabel(f"{n}")
    plt.xlabel("Latitude")
    plt.grid(True)
    plt.xlim([-92, 92])

    # Save the figure
    plt.savefig(f"Latitude_{characteristic}_n{count}.png")

    # Show plot
    plt.show()