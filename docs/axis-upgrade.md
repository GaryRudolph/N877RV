# Garmin Axis from G3X Touch Upgrade
---
## High Level Changes
1. HSDB from GDU to GPS 175
2. Extra Power to GDU
3. MapMX Now only GPS 175 to G5
4. GDU Update Power
5. Ok to retain TO/GA to GPS and GMC (per g3xpert)

## Specific Changes
1. GDU J1011 Swap Pins 6 & 9
2. GSU 25 SWAP Pins 6 & 9
3. GDU J1011 Add GDU Bat, Fuse & Switch
4. GDU J1012 Bridge Ground to 9 & 10 to 15 & 16; Move and bridge Pin 31 to 11 & 12. Bridge 31 to 32.
5. GDU J1012 Remove Pins 13, 30, 35 for MapMX
6. G5 Add MapMX TX Pin 5
7. GDU J1012 Remove Pin 10 & 27 strap
8. GDU Add HSDB Pins 3, 2, 20, 19
9. GPS Add HSDB Pins 10, 32, 11, 33

## Typical HSDB 4 Wire Colors
1. White/<Color> +VE (i.e. A)
2. <Color>/White -VE (i.e. B)
3. Blue is 1st, Orange is 2nd

From Manual
## Nav HSDB Interconnect
Page B-14
|-------------|--------|--------------|
| J1012       | P1751  | Color        |
|-------------|--------|--------------|
| **3**,5,22  | **11** | White/Blue   |
| **2**,4,21  | **33** | Blue/White   |
| **20**,7,39 | **10** | White/Orange |
| **19**,6,38 | **32** | Orange/White |
