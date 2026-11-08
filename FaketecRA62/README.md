# Faketec RA62 — MeshCore для Faketec + HT-RA62 (SX1262)

Прошивки MeshCore для **Faketec (ProMicro nRF52840)** з радіомодулем **HT-RA62 (SX1262)**, сенсорами, RTC та GPS.

Конфігурація збірки: [`variants/faketec_ra62/platformio.ini`](../variants/faketec_ra62/platformio.ini).

## Залізо

| Вузол | Модель | Підключення |
|---|---|---|
| MCU | ProMicro nRF52840 (Faketec) | — |
| Радіо | HT-RA62 (SX1262), TCXO 1.8 В, DIO2 як RF-switch | стандартна розводка Faketec |
| Сенсори (за наявності) | AHT20, BMP280/BME280, INA219, INA226, INA3221 | I²C SDA=8, SCL=7 |
| RTC (за наявності) | DS3231, RV3028, PCF8563, RX8130CE | I²C |
| GPS | NMEA, UART (швидкість визначається автоматично) | RX=3, TX=4, EN=5 |
| Дільник батареї | 1 МОм до «+», 1.5 МОм до «−» (множник 1.465) | ADC пін 17 |

> HT-RA62 розрахований на 868/915 МГц; на 433 МГц чутливість і потужність можуть бути нижчими.

## Прошивки

| Файл | Роль |
|---|---|
| `Faketec_RA62_repeater.uf2` / `.zip` | Repeater |
| `Faketec_RA62_sensor.uf2` / `.zip` | Sensor |
| `Faketec_RA62_companion_radio_ble.uf2` / `.zip` | Companion (BLE, PIN `123456`) |

Прошивка UF2: двічі натиснути reset → з'явиться USB-диск → скопіювати `.uf2`.
ZIP — для DFU або веб-flasher.

Радіопараметри за замовчуванням: **433.650 МГц, BW 62.5 кГц, SF 8, CR 8**. Змінюються через `set radio f,bw,sf,cr` або застосунок.

## Що змінено відносно MeshCore

- **Регістр 0x8B5** (`SX126X_REGISTER_PATCH`): біт для RX/AGC, повторно записується після кожного скидання AGC.
- **Скидання AGC**: чекає завершення калібрування перед записом регістрів; не виконується під час прийому пакета.
- **TCXO у standby**: TCXO не вимикається між Rx/Tx.
- **Рівень шуму**: перші 50 мс після старту прийому не враховуються.
- **АЦП батареї**: множник 1.465 під дільник 1М/1.5М, час вибірки 40 мкс; змінюється командою `set adc.multiplier` (тепер і на компаньйоні).
- **GPS**: автовизначення швидкості UART, інтервальний режим (раз на 30 хв, до 300 с), захист від спуфінгу (`GPS_GUARD`, регіон — Україна).
- **CLI**: `telemetry`, `time force <epoch>`.

## Дільник батареї

```
set adc.multiplier <x>    0 = значення зі збірки (1.465), діапазон 0–10
get adc.multiplier
```

Множник = 3600/4096 × (R_верх + R_низ) / R_низ. Наприклад, 680k/330k → 2.690.

## Збірка

```bash
pio run -e Faketec_RA62_repeater -e Faketec_RA62_sensor -e Faketec_RA62_companion_radio_ble
```
