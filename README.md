# Garden Irrigation System

System nawadniania ogrodu oparty na ESP32 i Home Assistant, zastepujacy sterownik Toro Temp-4.

## Zdjecia

| Toro Temp-4 (zastepowany) | Waveshare ESP32-S3-Relay-6CH |
|---|---|
| ![Toro Temp-4](images/toro-temp4.jpg) | ![ESP32-S3-Relay-6CH](images/esp32-relay-6ch.jpg) |

## Opis

Projekt zamiany tradycyjnego sterownika nawadniania Toro Temp-4 na rozwiazanie smart home z pelna kontrola przez Home Assistant. Wykorzystuje plytke Waveshare ESP32-S3-Relay-6CH z firmware ESPHome.

Cala logika bezpieczenstwa dziala na poziomie firmware — interlock stref, auto-off, reakcja na czujnik deszczu, restore mode — niezaleznie od Home Assistant i WiFi.

## Funkcje

- 4 strefy nawadniania sterowane niezaleznie (interlock — max 1 aktywna)
- Czujnik deszczu — natychmiastowe wylaczenie stref na poziomie firmware
- Konfigurowalne max runtime per strefa (slider 5-60 min w HA, domyslnie 30 min)
- Liczniki czasu pracy per strefa (diagnostyka zuzycia wody / wykrywanie usterek)
- Harmonogram nawadniania w Home Assistant
- Reczne sterowanie z dashboardu HA lub lokalnego web UI
- Dioda LED RGB jako wskaznik stanu (zielony = aktywna strefa, niebieski = deszcz)
- Buzzer — sygnal startu strefy, ostrzezenie przed auto-off, alarm deszczowy
- Monitoring WiFi i uptime
- Fallback AP z captive portal w razie utraty WiFi
- Lokalny web UI (`http://<ip>`) jako fallback gdy HA niedostepny
- Improv Serial — rekonfiguracja WiFi przez USB bez reflashowania

### Zabezpieczenia na poziomie firmware

> Szczegolowy opis: [docs/safety.md](docs/safety.md)

| Zabezpieczenie | Wymaga HA? | Wymaga WiFi? |
|----------------|------------|--------------|
| Interlock stref (max 1 aktywna, 2s przerwa) | Nie | Nie |
| Auto-off timer (konfigurowalne max runtime) | Nie | Nie |
| Czujnik deszczu (natychmiastowe wylaczenie) | Nie | Nie |
| Restore mode (ALWAYS_OFF po reboocie) | Nie | Nie |
| Niezaleznosc od WiFi (reboot_timeout: 0s) | Nie | Nie |
| Fallback AP + captive portal | Nie | Nie |
| Web server (panel sterowania na porcie 80) | Nie | Tak (LAN) |
| Improv Serial (rekonfiguracja WiFi po USB) | Nie | Nie |

## Hardware

| Element | Model |
|---------|-------|
| Kontroler | [Waveshare ESP32-S3-Relay-6CH](https://www.waveshare.com/wiki/ESP32-S3-Relay-6CH) |
| Firmware | [ESPHome](https://esphome.io/) 2026.3+ |
| Platforma | [Home Assistant](https://www.home-assistant.io/) |
| Zawory | Istniejace zawory 24V AC z systemu Toro |
| Transformator | Toro 24V AC, 625mA (istniejacy) |
| Zasilanie ESP32 | Ladowarka USB-C 5V lub zasilacz DC 7-36V |
| Czujnik deszczu | Styk NO (normally open) |

## Mapowanie kanalow

| Kanal | GPIO   | Funkcja  |
|-------|--------|----------|
| CH1   | GPIO1  | Strefa 1 |
| CH2   | GPIO2  | Strefa 2 |
| CH3   | GPIO41 | Strefa 3 |
| CH4   | GPIO42 | Strefa 4 |
| CH5   | GPIO45 | Rezerwa  |
| CH6   | GPIO46 | Rezerwa  |
| -     | GPIO4  | Czujnik deszczu (Pico header) |
| -     | GPIO38 | LED RGB (WS2812, onboard) |
| -     | GPIO21 | Buzzer (pasywny, onboard) |

## Struktura projektu

```
.
├── esphome/
│   ├── irrigation.yaml              # Konfiguracja ESPHome
│   └── secrets.yaml.example         # Szablon secrets
├── homeassistant/
│   └── automations/
│       ├── irrigation_rain_guard.yaml   # Automatyka - ochrona przed deszczem
│       └── irrigation_schedule.yaml     # Automatyka - harmonogram nawadniania
├── docs/
│   ├── hardware.md                  # Specyfikacja techniczna
│   ├── safety.md                    # Opis wszystkich zabezpieczen
│   ├── wiring.md                    # Schemat podlaczenia (tekst)
│   └── schemat.html                 # Interaktywny schemat (otworz w przegladarce)
├── images/                          # Zdjecia sprzetu
├── CHANGELOG.md                     # Historia zmian
└── pyproject.toml                   # Srodowisko Python (uv)
```

## Instalacja

### 1. Srodowisko

```bash
# Wymagany uv (https://docs.astral.sh/uv/)
uv sync
```

### 2. ESPHome

```bash
# Skopiuj i uzupelnij secrets
cp esphome/secrets.yaml.example esphome/secrets.yaml
# Edytuj secrets.yaml: wifi_ssid, wifi_password, api_key (base64), ota_password, fallback_password

# Pierwszy flash (USB-C)
uv run esphome run esphome/irrigation.yaml

# Kolejne aktualizacje (OTA, po WiFi)
uv run esphome run esphome/irrigation.yaml --device <IP>
```

### 3. Home Assistant

1. **Settings** -> **Devices & Services** -> **Add Integration** -> **ESPHome**
2. Podaj adres IP urzadzenia i klucz API z `secrets.yaml`
3. Skopiuj automatyzacje z `homeassistant/automations/` do konfiguracji HA
4. Dostosuj harmonogram w `irrigation_schedule.yaml`

### 4. Podlaczenie fizyczne

> Szczegolowy schemat krok po kroku: [docs/wiring.md](docs/wiring.md)
>
> Interaktywny schemat z kolorami: [docs/schemat.html](docs/schemat.html)

Skrotowo:
1. **USB-C** (ladowarka 5V) → zasilanie ESP32
2. **Mostek** COM na CH1-CH4 (skretka ethernetowa)
3. **Transformator Toro 24V AC**: przewod 1 → dowolny COM, przewod 2 → zlaczka z przewodem COM zaworow
4. **Przewody stref** → NO na CH1-CH4 (lewa srubka kazdego kanalu)
5. **Czujnik deszczu** → GPIO4 + GND na Pico headerze (konektory dupont)

## Konfiguracja harmonogramu

Domyslny harmonogram (`irrigation_schedule.yaml`):
- Codziennie o 6:00
- Kazda strefa po 15 minut, sekwencyjnie
- Automatyczne pominiecie gdy pada deszcz

Czasy i kolejnosc stref mozna dostosowac. Interlock gwarantuje ze nawet przy blednej konfiguracji nigdy nie dziala wiecej niz jedna strefa.

## Diagnostyka

- **Web UI**: `http://<ip>` — pelny panel sterowania, podglad sensorow
- **Runtime counters**: calkowity czas pracy kazdej strefy widoczny w HA — jesli jedna strefa ma znaczaco wiecej godzin, moze miec zatkana dysze lub uszkodzony zawor
- **WiFi Signal**: sila sygnalu w dBm — jesli ponizej -75 dBm, rozważ antene zewnetrzna lub zmiane lokalizacji
- **Logi ESPHome**: `uv run esphome logs esphome/irrigation.yaml --device <IP>`

## Licencja

MIT
