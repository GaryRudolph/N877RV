# Garmin Axis from G3X Touch Upgrade
---
## High Level Changes
1. HSDB from GDU to GPS 175
2. Extra Power to GDU
3. MapMX Unused on GDU
4. GDU Battery Power
5. Ok to retain TO/GA to GPS and GMC (per g3xpert)

## Specific Changes
~~1. GDU J1011 Swap Pins 6 & 9~~
~~2. GSU 25 SWAP Pins 6 & 9~~
3. GDU J1011 Add GDU Bat, Fuse & Switch from Backup Battery
4. GDU J1012 Bridge Ground to 9 & 10 to 15 & 16; Move and bridge Pin 31 to 11 & 12. Bridge 31 to 32.
5. GDU J1012 Remove Pin 10 & 27 strap
6. GDU Add HSDB Pins 3, 2, 20, 19
7. GPS Add HSDB Pins 10, 32, 11, 33

## Typical HSDB 4 Wire Colors
1. White/<Color> +VE (i.e. A)
2. <Color>/White -VE (i.e. B)
3. Blue is 1st, Orange is 2nd

From Manual
## Nav HSDB Interconnect
Page B-14

| J1012       | P1751  | Color        |
|-------------|--------|--------------|
| **3**,5,22  | **11** | White/Blue   |
| **2**,4,21  | **33** | Blue/White   |
| **20**,7,39 | **10** | White/Orange |
| **19**,6,38 | **32** | Orange/White |

## Bring
1. Parts
    1. 6" Extensions
        1. G5
        2. GDU
        3. GSU
    2. Bracket
        1. Bolts, Washers, Lock Washers
    3. GDU Batt
        1. Red AWG 20
        2. 2A Fuse
        3. Fuse Holder
        4. Crimp Connector
        5. SD Pin
    4. GDU Power / Ground
        1. 20 AWG Red
        2. 20 AWG Black
        3. Solder Sleeves
        4. SD Pin
    5. HSDB
        1. GPS to GDU
        2. SD Pin
2. Buy
    1. SD Pin Extractor (JR)
    2. HD Pin Extractor x10
    3. Switch
    4. 20 AWG Red Wire (5 Feet)
    5. 20 AWG Black Wire (5 Feet)
    6. 2A Fuse
    7. Fuse Holder
    8. Fabric Tape
3. Bring
    1. SD Pins
    2. HD Pins
    3. AFM8 DMC Crimper K13-1, K42
    4. Regular Crimper
4. Tools
    1. Cutters
    2. Heat Gun