# Algemene werkwijze

Na start/herstart van je Mac zijn (afhankelijk van je setup) Apache en MySQL al meteen actief.
In mijn geval moet ik de ontwikkelomgeving handmatig starten. Ik open een Terminal en start de servers:

```
$ startdevelopment
```

Apache en MySQL zijn nu klaar voor gebruik.

# Nieuwe database aanmaken

- Een nieuwe database aanmaken doe je via phpMyAdmin door in je browser te gaan naar http://phpmyadmin.dev
- Klik op **Databases** en vul onder Create database de naam van je database in en klik op Create.
- De database is dan klaar voor gebruik. Gebruikersnaam en wachtwoord is **root**.

# Nieuwe website aanmaken

Een nieuwe website aanmaken is zeer eenvoudig. Maak simpel in je Sites map een nieuwe submap aan voor de website.
Als voorbeeld maken we een website aan **testwebsite**.

```
$ mkdir ~/Development/Sites/testwebsite
```

Plaats daarna de website bestanden in de map ~/Development/Sites/testwebsite.

Je hoeft dan verder niets meer aan te passen. De website is direct in je browser benaderbaar via http://testwebsite.dev


