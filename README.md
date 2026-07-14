# Živý obraz — Pervasive Displays / VUSION fork

Fork [zivyobraz-fw](https://github.com/MultiTricker/zivyobraz-fw) pridávajúci podporu pre Pervasive
Displays / SES-imagotag VUSION panely — recyklované e-ink cenovky z obchodov. Všeobecné info o
projekte, dokumentácia, inštalácia, kúpa hardvéru atď. — pozri **originálny repo vyššie**. Tu je len
to, čo je v tomto forku iné.

Panely aj drivery pochádzajú z [GxEPD2_PervasiveDisplays](https://github.com/shanislav/GxEPD2_PervasiveDisplays)
— tam nájdeš zapojenie, fotky a reverse-engineering poznámky k jednotlivým panelom.

## Podporované panely

| PlatformIO env | Panel | Doska | Poznámka |
|---|---|---|---|
| `esp32c3_supermini_e2266` | SE2266JS0C5, 2.66" BWR | ESP32-C3 Super Mini | genuine iTC |
| `esp32c3_supermini_e2581` | SE2581JSBF1, 5.81" BWR | ESP32-C3 Super Mini | non-iTC, reverse-engineered |
| `esp32c3_supermini_e0g1` | SE2581JS0G1, 5.81" BWR | ESP32-C3 Super Mini | genuine iTC, iný COG než JSBF1 (rovnaká doska/veľkosť skla) |
| `esp32s2_mini_e2969` | TE2969JS0B4, 9.7" BWR, dual-COG | Wemos/LOLIN S2 Mini | potrebuje PSRAM |

Build: `pio run -e <env>`, flash: `pio run -e <env> -t upload`.

## Zapojenie

Piny sú v `src/board.h` pod `ESP32C3_SUPERMINI` / `ESP32S2_MINI`. Skrátene:

**ESP32-C3 Super Mini** (2.66", oba 5.81"): CS=10, DC=9, RST=3, BUSY=2, SCLK=6, MOSI=7,
`ePaperPowerPin`=8 (na tejto doske bez funkcie — panel treba napájať priamo 3.3V, nie cez FET).

**Wemos S2 Mini** (9.7"): M-CS=34, S-CS=33, DC=37, RST=38, BUSY=39, SCLK=36, MOSI=35,
`ePaperPowerPin`=40 (tag doska má vlastný MOSFET na tomto pine, active-low).

Panel napájanie a dátové linky potrebujú 3.3V.

## Čo je v tomto forku iné oproti upstream

- Nová doska `ESP32S2_MINI` **bez `REMAP_SPI`** — 9.7" driver si SPI rieši sám (číta OTP panela
  bit-bangom pred `SPI.begin()`); na S2 by `REMAP_SPI` túto OTP inicializáciu otrávil a panel by sa
  zapol, ale nikdy nič nevykreslil.
- 9.7" buffery idú do **PSRAM** (S2 Mini má 2 MB) — bez toho vyčerpá interný RAM spolu s WiFi/JSON.
- V `main.cpp` je pre `ESP32S2_MINI` pri boote jeden `Display::clear()` navyše — 9.7" dual-COG
  potrebuje "zahriatie" prvým refreshom, inak prvý reálny obsah (napr. WiFi setup obrazovka) vyjde
  prázdny.

## Prvé spustenie (WiFi provisioning)

Po flashnutí vysiela doska vlastnú AP sieť `INK_<MAC>`, heslo `zivyobraz`. Pripoj sa na ňu **hneď**
(nečakaj na obrazovku na paneli — najmä 9.7" má prvý refresh pomalý, cca 1–2 min aj s warm-up
clearom). Otvorí sa captive portal, tam zadaj domácu WiFi a token zo zivyobraz.eu.

## Ktorý 5.81" panel mám?

`SE2581JSBF1` a `SE2581JS0G1` sú rovnako veľké, sedia na tú istú dosku, ale sú to úplne odlišné
COG-y — vyber env podľa označenia na zadnej strane panela.

----

## License info

![CC BY-NC-SA 4.0](https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png)

Rovnako ako upstream, ZivyObraz FW je licencovaný pod
[Creative Commons Attribution-NonCommercial-ShareAlike 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/).
Drivery z GxEPD2_PervasiveDisplays sú GPL-3.0.
