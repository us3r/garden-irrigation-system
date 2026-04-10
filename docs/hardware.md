# Specyfikacja techniczna

## Waveshare ESP32-S3-Relay-6CH

| Parametr              | Wartosc                          |
|-----------------------|----------------------------------|
| MCU                   | ESP32-S3-WROOM-1U-N8            |
| CPU                   | Dual-core Xtensa LX7, 240 MHz   |
| Flash                 | 8 MB                             |
| SRAM                  | 512 KB                           |
| WiFi                  | 802.11 b/g/n, 2.4 GHz           |
| Bluetooth             | BLE 5.0                          |
| Kanaly przekaznikow   | 6x (uzywamy 4)                   |
| Przekaznik - obciazenie | 10A / 250V AC lub 10A / 30V DC |
| Izolacja              | Optyczna (przekazniki) + galwaniczna (RS485) |
| Zasilanie             | 7-36V DC (zlacze srubowe) lub 5V USB-C |
| RS485                 | Izolowany, UART (GPIO17 TX / GPIO18 RX) |
| Buzzer                | Pasywny, GPIO21 (PWM)           |
| LED                   | WS2812 RGB, GPIO38               |
| Antena                | Zewnetrzna, zlacze SMA (WiFi)   |

## Zawory elektromagnetyczne

Wykorzystujemy istniejace zawory z systemu Toro Temp-4:
- Napiecie: 24V AC
- Typ: elektromagnetyczne, normalnie zamkniete
- 4 sztuki (po jednym na strefe)

## Czujnik deszczu

- Typ: styk NO (normally open)
- Podlaczenie: GPIO4 + GND
- Zamyka obwod przy wykryciu deszczu

## Transformator

Wykorzystujemy istniejacy transformator 24V AC z systemu Toro:
- Model: XY-2400625AG (Xing Yuan Electronics)
- Input: 230-240V ~50Hz
- Output: 24V AC, 625mA, 15VA
- Sluzy wylacznie do zasilania zaworow elektromagnetycznych (nie do zasilania ESP32)
- AC nie ma biegunowosci — nie ma znaczenia ktory przewod gdzie

## Zasilanie ESP32

ESP32 wymaga osobnego zasilania DC:
- **Opcja 1 (zalecana):** Ladowarka USB-C 5V (np. od telefonu)
- **Opcja 2:** Zasilacz DC 7-36V do zlacza srubowego na plytce

Nie mozna zasilac ESP32 z transformatora Toro (24V AC) bez prostownika.
