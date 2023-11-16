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
