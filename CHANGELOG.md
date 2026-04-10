# Changelog

## v1.2.0 — Refaktor logiki stref i liczniki czasu pracy

- Wyciagniecie logiki wlaczania/wylaczania stref do wspolnych scriptow (`zone_started`, `zone_stopped`) — jeden punkt prawdy zamiast 4 kopii
- Naprawa buga double-timer: zamiana inline `delay` na `script` z `mode: restart` — ponowne wlaczenie strefy czysto resetuje odliczanie, zamiast tworzyc dwa rownolegle timery
- Dodanie `script.stop` przy wylaczeniu strefy — anuluje timer auto-off jesli strefa zostala wylaczona recznie
- Dodanie sygnalow dzwiekowych: krotki beep przy starcie strefy + ostrzezenie przed auto-off
- Dodanie `duty_time` per strefa — calkowity czas pracy z `restore: true` (przezywa reboot), diagnostyka zuzycia wody
- Dodanie `web_server` na porcie 80 — lokalny panel sterowania jako fallback gdy HA jest niedostepny
- Dodanie `improv_serial` — mozliwosc rekonfiguracji WiFi przez port szeregowy w razie blednych credentials
- Ustawienie `wifi: reboot_timeout: 0s` — ESP32 nie restartuje sie przy utracie WiFi, konczy biezacy cykl nawadniania

## v1.1.0 — Zabezpieczenia i konfigurowalne czasy

- Dodanie interlocka stref — tylko jedna strefa moze byc aktywna w danym momencie
- Dodanie `interlock_wait_time: 2s` — przerwa miedzy przelaczeniem stref, daje zaworom czas na zamkniecie i chroni transformator przed podwojnym poborem pradu
- Dodanie konfigurowalnego max runtime (slider w HA, zakres 5-60 min, domyslnie 30 min) — automatyczne wylaczenie strefy po zadanym czasie, zabezpieczenie przed zalaniem
- Dodanie reakcji czujnika deszczu na poziomie firmware — natychmiastowe wylaczenie wszystkich stref bez udzialu HA
- LED zmienia kolor na niebieski przy wykryciu deszczu
- Sygnal dzwiekowy przy wykryciu deszczu
- Naprawa nazwy platformy LED (`esp32_rmt_led_strip`)
- Dodanie interaktywnego schematu podlaczen (`docs/schemat.html`)
- Konfiguracja srodowiska Python (uv + pyproject.toml)

## v1.0.0 — Pierwsza wersja

- Konfiguracja ESPHome dla Waveshare ESP32-S3-Relay-6CH
- 4 strefy nawadniania na przekaznikach CH1-CH4 (GPIO1, GPIO2, GPIO41, GPIO42)
- Czujnik deszczu na GPIO4 z INPUT_PULLUP i asymetrycznym debounce (500ms on / 5000ms off)
- LED RGB (WS2812) jako wskaznik stanu — zielony = strefa aktywna, wygaszenie gdy wszystkie off
- Buzzer pasywny (RTTTL) na GPIO21
- `restore_mode: ALWAYS_OFF` na wszystkich strefach i LED — bezpieczny stan po reboocie
- Fallback AP z captive portal w razie utraty WiFi
- Szyfrowana komunikacja z HA (API + encryption key)
- OTA z haslem
- Sensory diagnostyczne: uptime, sygnal WiFi, adres IP, SSID
- Przycisk restartu w HA
- Automatyzacje HA: harmonogram poranny (6:00, 4x15min) + rain guard
- Dokumentacja techniczna: specyfikacja hardware, schemat podlaczen
- Licencja MIT
