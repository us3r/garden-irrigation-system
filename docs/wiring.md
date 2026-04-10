# Schemat podlaczenia

## Waveshare ESP32-S3-Relay-6CH - przypisanie kanalow

| Kanal | GPIO  | Funkcja           | Kabel        |
|-------|-------|--------------------|--------------|
| CH1   | GPIO1 | Strefa 1           | Zawor 24V AC |
| CH2   | GPIO2 | Strefa 2           | Zawor 24V AC |
| CH3   | GPIO41| Strefa 3           | Zawor 24V AC |
| CH4   | GPIO42| Strefa 4           | Zawor 24V AC |
| CH5   | GPIO45| Wolny (rezerwa)    | -            |
| CH6   | GPIO46| Wolny (rezerwa)    | -            |

## Czujnik deszczu

| Pin   | Funkcja              |
|-------|----------------------|
| GPIO4 | Rain sensor (INPUT_PULLUP) |
| GND   | Rain sensor GND      |

Czujnik deszczu to typowy styk NO (normally open) - zamyka obwod przy wykryciu deszczu.

## Zasilanie

- **ESP32**: zasilanie 12V DC lub 24V DC przez zlacze srubowe (zakres 7-36V DC)
- **Zawory**: zasilanie z transformatora 24V AC (z obecnego systemu Toro)
- Przekazniki przelaczaja obwod 24V AC do zaworow - kazdy kanal obsluguje do 10A / 250V AC

## Schemat polaczenia zaworow

```
Transformator 24V AC
       |
       +--- COM (wspolny) ---> do kazdego zaworu (przewod wspolny)
       |
       +--- przez CH1 relay ---> Zawor strefy 1
       +--- przez CH2 relay ---> Zawor strefy 2
       +--- przez CH3 relay ---> Zawor strefy 3
       +--- przez CH4 relay ---> Zawor strefy 4
```

## Uwagi

- Przekazniki sa izolowane optycznie od ESP32
- Zawory elektromagnetyczne Toro dzialaja na 24V AC - kompatybilne z przekaznikami (10A/250V AC)
- Przy podlaczaniu zachowaj polaczenie wspolne (COM) ze wszystkimi zaworami
- Transformator 24V AC z istniejacego systemu Toro mozna wykorzystac ponownie
