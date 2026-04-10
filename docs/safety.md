# Zabezpieczenia

Cala logika bezpieczenstwa dziala na poziomie firmware ESP32 — niezaleznie od Home Assistant, WiFi czy jakiegokolwiek zewnetrznego systemu.

## Interlock stref

**Problem:** Jednoczesne wlaczenie kilku zaworow przeciaza transformator 24V AC (625mA, jeden zawor ~200-300mA).

**Rozwiazanie:** ESPHome `interlock` — tylko jedna strefa moze byc aktywna w danym momencie. Wlaczenie strefy 2 automatycznie wylacza strefe 1.

**Dodatkowa ochrona:** `interlock_wait_time: 2s` — 2 sekundy przerwy miedzy wylaczeniem jednej i wlaczeniem drugiej strefy. Daje zaworowi czas na fizyczne zamkniecie zanim otworzy sie nastepny.

```yaml
interlock: &interlock_group [zone_1, zone_2, zone_3, zone_4]
interlock_wait_time: 2s
```

## Auto-off (max runtime)

**Problem:** Jesli HA straci polaczenie, ktos zapomni wylaczyc strefe, lub automatyzacja sie zawiesi — zawor moze dzialac w nieskonczonosc, zalewajac ogrod.

**Rozwiazanie:** Kazda strefa ma timer auto-off. Domyslnie 30 minut, konfigurowalne z HA (slider 5-60 min). Wartosc przezywa restart ESP32 (`restore_value: true`).

**Naprawa double-timer:** Timer jest zaimplementowany jako `script` z `mode: restart`. Jesli strefa zostanie wylaczona i wlaczona ponownie przed uplywem czasu — timer resetuje sie czysto, zamiast tworzyc dwa rownolegle odliczania. Przy recznym wylaczeniu strefy `script.stop` anuluje timer.

```yaml
script:
  - id: zone_auto_off_1
    mode: restart
    then:
      - delay: !lambda "return id(max_runtime).state * 60 * 1000;"
      - switch.turn_off: zone_1
```

## Czujnik deszczu

**Problem:** Nawadnianie podczas deszczu to marnowanie wody i potencjalne przelanie ogrodu.

**Rozwiazanie:** Czujnik deszczu podlaczony bezposrednio do GPIO4 ESP32. Reakcja na firmware:

1. Natychmiastowe wylaczenie wszystkich 4 stref
2. LED zmienia kolor na niebieski (sygnalizacja deszczu)
3. Sygnal dzwiekowy z buzzera

**Debounce:** Asymetryczny filtr — szybkie wykrycie deszczu (500ms), wolne odwolanie (5000ms). "Wykryj szybko, ale nie odwoluj alarmu zanim naprawde wyschnie."

```yaml
filters:
  - delayed_on: 500ms
  - delayed_off: 5000ms
```

**Dodatkowa warstwa:** Automatyzacja w HA (`irrigation_rain_guard.yaml`) jako backup + powiadomienie push.

## Restore mode

**Problem:** Po utracie zasilania lub reboocie ESP32 — w jakim stanie startuja przekazniki?

**Rozwiazanie:** `restore_mode: ALWAYS_OFF` na wszystkich strefach i LED. Niezaleznie od poprzedniego stanu — po reboocie wszystko jest wylaczone. Zadnego niekontrolowanego nawadniania.

```yaml
restore_mode: ALWAYS_OFF
```

## Niezaleznosc od WiFi

**Problem:** Domyslnie ESPHome restartuje urzadzenie po 15 min bez WiFi. Jesli strefa jest aktywna podczas utraty WiFi — restart ja przerwie (ALWAYS_OFF), ale tez przerwie caly cykl nawadniania.

**Rozwiazanie:** `reboot_timeout: 0s` — ESP32 nie restartuje sie przy utracie WiFi. Kontynuuje prace, konczy cykl nawadniania. Auto-off timer i interlock dzialaja niezaleznie od WiFi.

```yaml
wifi:
  reboot_timeout: 0s
```

## Fallback AP

**Problem:** Jesli dane WiFi sa bledne lub siec niedostepna — ESP32 jest nieosiagalny.

**Rozwiazanie:** Automatyczny hotspot `garden-irrigation-fallback` z captive portal. Mozna sie polaczyc i skonfigurowac nowe dane WiFi.

## Web server

**Problem:** HA jest offline — jak sterowac nawadnianiem?

**Rozwiazanie:** Lokalny web server na porcie 80 (`http://192.168.254.235`). Pelny panel sterowania: wlaczanie/wylaczanie stref, podglad sensorow, zmiana max runtime.

## Improv Serial

**Problem:** Secrets.yaml ma literowke w hasle WiFi — urzadzenie nie laczy sie z siecia, fallback AP nie pomaga bo trzeba reflashowac.

**Rozwiazanie:** `improv_serial` pozwala zmienic dane WiFi przez port USB bez reflashowania firmware.

## Runtime counters (diagnostyka)

**Problem:** Jak wykryc ze jedna strefa dziala znaczaco dluzej niz inne? (np. zatkana dysza, uszkodzony zawor)

**Rozwiazanie:** `duty_time` per strefa z `restore: true`. Calkowity czas pracy kazdej strefy, widoczny w HA. Jesli strefa 3 ma 2x wiecej godzin niz reszta — cos jest nie tak.

## Podsumowanie

| Zabezpieczenie | Gdzie dziala | Wymaga HA? | Wymaga WiFi? |
|----------------|-------------|------------|--------------|
| Interlock stref | Firmware | Nie | Nie |
| Auto-off timer | Firmware | Nie | Nie |
| Czujnik deszczu | Firmware + HA | Nie (firmware wystarczy) | Nie |
| Restore mode | Firmware | Nie | Nie |
| WiFi independence | Firmware | Nie | Nie |
| Fallback AP | Firmware | Nie | Nie |
| Web server | Firmware | Nie | Tak (LAN) |
| Improv Serial | Firmware | Nie | Nie (USB) |
| Runtime counters | Firmware + HA | Do podgladu tak | Tak |
| Rain guard automation | HA | Tak | Tak |
| Harmonogram | HA | Tak | Tak |
