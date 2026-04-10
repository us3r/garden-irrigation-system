# Schemat podlaczenia

> Interaktywny schemat z kolorami: [schemat.html](schemat.html) (otworz w przegladarce)

## Zasilanie — dwa niezalezne obwody

System wymaga dwoch zrodel zasilania:

| Obwod | Zrodlo | Co zasila |
|-------|--------|-----------|
| Logika (ESP32) | Ladowarka USB-C 5V **lub** zasilacz DC 7-36V | Plytka, WiFi, przekazniki (cewki) |
| Zawory 24V AC | Transformator Toro (24V AC, 625mA) | Zawory elektromagnetyczne |

**Uwaga:** Transformator Toro daje 24V AC (prad zmienny) — nie mozna nim zasilac ESP32 (wymaga DC). Dlatego potrzebna jest osobna ladowarka USB-C lub zasilacz DC.

## Przypisanie kanalow

| Kanal | GPIO   | Funkcja           | Kabel        |
|-------|--------|--------------------|--------------|
| CH1   | GPIO1  | Strefa 1           | Zawor 24V AC |
| CH2   | GPIO2  | Strefa 2           | Zawor 24V AC |
| CH3   | GPIO41 | Strefa 3           | Zawor 24V AC |
| CH4   | GPIO42 | Strefa 4           | Zawor 24V AC |
| CH5   | GPIO45 | Wolny (rezerwa)    | -            |
| CH6   | GPIO46 | Wolny (rezerwa)    | -            |

## Terminale przekaznikow

Kazdy kanal (CH1-CH6) ma 3 terminale srubowe. Patrzac od przodu plytki (numery kanalow u gory):

```
Lewa srubka = NO  (Normally Open)   ← tu podlaczasz zawor
Srodkowa    = COM (Common)          ← tu podlaczasz transformator
Prawa srubka = NC (Normally Closed) ← zostawiasz pusta
```

## Podlaczenie krok po kroku

### Krok 1 — Zasilanie ESP32

Podlacz ladowarke USB-C (5V, np. od telefonu) do portu USB-C na plytce.

Alternatywnie: zasilacz DC 7-36V do zlacza srubowego u gory plytki (oznaczone `DC 7-36V`).

### Krok 2 — Mostek COM na przekaznikach

Polacz srodkowe srubki (COM) kanalow CH1-CH4 miedzy soba kawalkami drutu (np. skretka ethernetowa, obrana z izolacji ~5-7mm na koncach).

```
CH1        CH2        CH3        CH4
NO COM NC  NO COM NC  NO COM NC  NO COM NC
    |          |          |          |
    +---mostek-+---mostek-+---mostek-+
```

### Krok 3 — Transformator 24V AC

Transformator Toro ma dwa przewody wyjsciowe. AC nie ma biegunowosci — nie ma znaczenia ktory gdzie.

- **Przewod 1** → wkrec w dowolny COM na przekazniku (juz zmostkowane, wiec trafi do wszystkich)
- **Przewod 2** → polacz z przewodem wspolnym zaworow (COM z ogrodu) zlaczka Wago lub kostka elektryczna

### Krok 4 — Przewody stref do NO

Kazdy przewod sterujacy zaworem wkrec w lewy terminal (NO) odpowiedniego kanalu:

| Przewod z ogrodu | Terminal |
|------------------|----------|
| Strefa 1         | NO na CH1 |
| Strefa 2         | NO na CH2 |
| Strefa 3         | NO na CH3 |
| Strefa 4         | NO na CH4 |

### Krok 5 — Czujnik deszczu

Czujnik deszczu podlaczamy do Pico headera na plytce (dolny rzad pinow, od prawej):

| Pin na headerze | Funkcja |
|-----------------|---------|
| IO4 (4. pin od prawej, dolny rzad) | Sygnal czujnika |
| GND (2. pin od prawej, dolny rzad) | Masa czujnika |

Czujnik deszczu to styk NO — nie ma biegunowosci. Do podlaczenia potrzebujesz konektorow dupont (zenskie) lub lutowania.

## Schemat ogolny

```
                         ┌─────────────────────────────────┐
 Ladowarka USB 5V ──────┤ USB-C                            │
                         │                                  │
                         │   ESP32-S3-Relay-6CH             │
                         │                                  │
                         │   CH1    CH2    CH3    CH4       │
                         │  NO COM NO COM NO COM NO COM     │
                         └──┬──┬───┬──┬───┬──┬───┬──┬──────┘
                            │  │   │  │   │  │   │  │
                            │  └───┴──┴───┴──┘   │  │
                            │       │ (mostek)    │  │
 Transformator     przewod 1┤       │             │  │
 Toro 24V AC ──────────────>├───────┘             │  │
                   przewod 2┤                     │  │
                            │   ┌─────────────────┘  │
 Przewod COM  <─────────────┘   │                    │
 (wspolny zaworow)              │                    │
       │                        │                    │
       │    ┌───────────────────┘                    │
       │    │         ┌──────────────────────────────┘
       ▼    ▼         ▼                    ▼
    ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
    │ Zawor S1 │ │ Zawor S2 │ │ Zawor S3 │ │ Zawor S4 │
    └──────────┘ └──────────┘ └──────────┘ └──────────┘

 Czujnik deszczu ──── IO4 + GND (Pico header)
```

## Uwagi

- **NC (Normally Closed)** — zostawiasz puste na wszystkich kanalach. Podlaczenie do NC sprawi ze zawory beda otwarte domyslnie i zamykane przy wlaczeniu (odwrotnie niz chcemy).
- Przekazniki sa izolowane optycznie od ESP32 — zwarcie w obwodzie 24V AC nie uszkodzi mikrokontrolera
- Transformator 24V AC (625mA) wystarczy na 1 zawor naraz (~200-300mA). Interlock w firmware gwarantuje ze nigdy nie dziala wiecej niz jeden.
- Na mostek COM mozna uzyc skretki ethernetowej (24AWG) — wystarczajaco na te prady
- Przy podlaczaniu przewodow do zaciskow srubowych — obierz izolacje ~5-7mm na koncu
