# PART 1

```
Pour commencer votre investigation, vous devez retrouver le standard utilisé par les systèmes de navigation dont voici l'une des trames :
$GPRMC,.00,A,4821.984,N,00429.118,W,17.0,170.4,2023,,,A,S*09
Format du Flag : DGHACK{nom_du_standard_numéro} (tout attaché, sans underscore)
```

Just by googling GPRMC part, we easily see that the standard is NMEA 0183. The flag is DGHACK{NMEA0183}

# PART 2

```
Lors de vos analyses, vous avez besoin de retrouver l'une des routes maritime empruntée par le Sam Brandy.
Retrouver les coordonnées GPS à partir de la trame suivante :
$GPGGA,123519,4815.970,N,0447.340,W,1,08,0.9,545.4,M,46.9,M, ,*41
Donnez le résultat au format "degré" "minute" "seconde" (arrondi au centième de seconde).
Format du Flag : DGHACK{DDMMSS.SS[N|S],DDMMSS.SS[W|E]}
```

# PART 4
```
Il semble que le Sam Brandy ait été identifié le 14/09/2023 loin de sa zone d'opération. Bien que cela vous semble troublant, vous décidez de reconstituer les évènements grâces aux trames que vous avez récupérés :
$GPGSV,3,1,9,27,65,309,24,31,63,061,10,16,57,203,34,26,55,143,171E $GPGSV,3,2,9,05,36,257,29,28,32,048,29,08,27,312,34,18,27,109,1940
$GPGSV,3,3,9,09,17,232,2876
Les coordonnées indiquées sont les suivantes :
30°41'24.0''S 133°42'0.0''W
Vous devez retrouver à qu'elle heure il a été aperçu (GMT+00) ?
Format du Flag : DGHACK{OCEAN_HH_MM}
Fichier join : https://www.dghack.fr/uploads/event-dghack-2023/a-maritime-journey/diagram.png
```
We are given a set of GSV data.
Once analysed, we extract these important data:
USA-242 : 65,309
USA-190 : 63,061
USA-166 : 57,203
USA-260 : 55,143
USA-206 : 36,257
USA-343 : 32,048
USA-262 : 27,312
USA-293 : 27,109
USA-256 : 17,232
There are the satelite name which i got from the SV PRN number, the altitude, and the azimuth that has been registered.

We know the day is 14th september 2023.

I used this map:
https://in-the-sky.org/satmap_worldmap.php?year=2023&month=09&day=16
I've set my cordinates to 30°41'24.0''S 133°42'0.0''W, which are equal to Lat: -30.69 Long :-133.7.
Using the satelite research function, i found the USA 242.
Once i've done it, i changed hours, until i saw the values matching with the USA 242 ones, which means 65 latitude and 309 Azimuth.
That matched with the hour 20h53. Since the website is in GMT +1, i removed 1 hour, which gave me the exact hour: 19h53.
Also, the cordinates are in the Pacific Ocean.

That gives us the flag DGHACK{PACIFIC_19_53} !
