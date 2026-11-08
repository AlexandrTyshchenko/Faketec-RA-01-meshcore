# Faketec RA-01 — Faketec з модулем Ai-Thinker Ra-01 (SX1278)

Та сама збірка, що й [Faketec RA62](../FaketecRA62/README.md) (датчики, GPS із захистом від спуфінгу, дільник батареї з CLI, зокрема на компаньйоні), але радіомодуль **Ra-01**: SX1278, 410–525 МГц, до **+18 dBm**, кварц без TCXO.

Відмінності від RA62:
- драйвер `CustomSX1276` (SX1278 має ті самі регістри); патчі лише для SX126x (калібрування при скиданні AGC, регістр 0x8B5, TCXO у standby) тут не застосовуються. Загальні — пропуск скидання AGC під час прийому і фільтр рівня шуму — працюють;
- **обмеження потужності 18 dBm** (`LORA_TX_POWER_MAX=18`): за замовчуванням 18, `set tx` понад 18 відповідає `Error, max is 18`, збережене раніше більше значення обрізається до 18 і в налаштуваннях, і в драйвері; компаньйон теж не дасть виставити більше;
- струмовий захист PA 140 мА (`SX127X_CURRENT_LIMIT`).

Радіо: 433.650 МГц / 62.5 кГц / SF8 / CR8, як у FakeSolar; синхрослово сумісне з мережею на SX126x.

> На залізі ще не перевірено.

## Підключення

Плата Faketec під SX126x, модуль Ra-01 з **додатковим дротиком**:

| Ra-01 | nRF52840 | Пін на платі |
|---|---|---|
| DIO0 (Rx/Tx done) | P0.29 | BUSY (D20) — дротик |
| DIO1 | P0.10 | DIO1/IRQ |
| NSS / RESET | P1.13 / P0.09 | стандартно |
| SCK / MISO / MOSI | P1.11 / P0.02 / P1.15 | стандартно |

Так само, як RF95 у Meshtastic для цієї плати (`nrf52_promicro_diy_tcxo`).

## Дільник батареї

```
set adc.multiplier <x>    0 = значення зі збірки (1.465), діапазон 0–10
get adc.multiplier
```

Множник = 3600/4096 × (R_верх + R_низ) / R_низ. Множник із Meshtastic перераховується × 0.8789 (напр. 2.048 → 1.800).

## Файли

| Файл | Роль |
|---|---|
| `Faketec_RA01_repeater.*` | repeater, ім'я `Faketec Repeater`, пароль адміністратора `password` |
| `Faketec_RA01_sensor.*` | sensor, ім'я `Faketec Sensor`, пароль `password` |
| `Faketec_RA01_companion_radio_ble.*` | companion BLE, PIN `123456` |

`.uf2` — подвійний reset і копіювання на диск; `.zip` — DFU по USB або OTA (`start ota`); `.hex` — для програматора.

## Збірка

```
pio run -e Faketec_RA01_repeater -e Faketec_RA01_sensor -e Faketec_RA01_companion_radio_ble
python bin/uf2conv/uf2conv.py .pio/build/<env>/firmware.hex -c -f 0xADA52840 -o <env>.uf2
```

Конфігурація: `variants/faketec_ra01/platformio.ini`.
