# Калибровка принтера для Klipper

3D принтер **Ender 3 Pro** с апгрейдами:

* Исполнительная плата **BigTreeTech SKR Mini E3 v2.0**
* Управляющая плата **BigTreeTech BTT-PI v1.2**
* Экструдер **BMG clone**
* Хотенд **E3DV6**
* Датчик уровня стола **Trianglelab 2021 v3 3D Touch**
* Собственное спроектированное крепление **E3BMGS - Крепление директ BMG(clone) + E3DV6 экструдер**

## Порядок калибровки

0. Механика и электрика собраны как следует. Точно?
1. Калибровка PID Экструдера и Стола
2. Калибровка шагов/разрешения экструдера
3. Калибровка стола (z-offset)
4. Калибровка Pressure Advance
5. Калибровка откатов (fw-retract)
6. Калибровка потока
7. РУЧНАЯ Калибровка шейперов(Input Shaping) / Ускорений
8. АВТОМАТИЧЕСКАЯ Калибровка шейперов(Input Shaping) / Ускорений

## Видео материалы и литература

Порядок калибровки основан на материалах:

* <Дмитрий Соркин>: [Как увеличить точность 3D принтера?](https://youtu.be/POG_Th2k3_I)
* <Дмитрий Соркин>: [Калибровка PID](https://youtu.be/doenKnVk0Ec?t=937)
* [Klipper Wiki - Rotation Distance](https://klipper.wiki/ru/home/tuning/rotation)
* <Дмитрий Соркин>: [Калибровка Pressure Advance](https://k3d.tech/calibrations/la/)
* <Дмитрий Соркин>: [Ручной подбор частоты Input Shaping](https://k3d.tech/calibrations/manual_is_calibration/)

## 1. Калибровка PID Экструдера и Стола

Калибровка PID для экструдера.  
Важно чтобы в конфигурации принтера параметр **max_temp** был больше чем калибровочное значение.
Пример: `max_temp: 280` для тестов на 250 градусах

```shell
PID_CALIBRATE HEATER=extruder TARGET=250
```

Калибровка PID для стола. `max_temp: 130`

```shell
PID_CALIBRATE HEATER=heater_bed TARGET=100
```

После окончания калибровки в консоль Web интерфейса выведет откалиброванные параметры `pid_Kp`, `pid_Ki` и `pid_Kd`.  
Вносим их в конфигурацию принтера в разделы `[extruder]` и `[heater_bed]`

## 2. Калибровка шагов/разрешения экструдера

Калибровка отличается от Marlin и немного сложнее.

### Если ранее была проведена калибровка в Marlin, то можно пересчитать это значение

В Klipper для этого используется параметр `rotation_distance`

В Marlin выставляется так: `DEFAULT_AXIS_STEPS_PER_UNIT {80, 80, 400, 410.7} | X, Y, Z [, I [, J [, K...]]], E0 [, E1[, E2...]]`

E0 = **410.7** Steps Per Unit

Формула: `rotation_distance` = `full_steps_per_rotation` * `microsteps` / `steps_per_mm`  
где:  

* `full_steps_per_rotation` - из документации к шаговикам. Если шаговик с углом 1.8, то для него 200 шагов на оборот
(для угла 0.9 будет уже 400 шагов на оборот)
* `microsteps` - микрошаги драйвера. Можно/нужно найти в параметрах экструдера `[extruder]`, но обычно 16

`rotation_distance = 200 * 16 / 410.7 = 7.7915`

Вставляем в раздел `[extruder]` значение `rotation_distance: 7.7915`

**!!! `gear_ratio` тут не используем !!!**

### Если сразу Klipper

В случае использования редукторов - указывается коэффициент редукции. Для BMG клонов как правило `3:1`

Чтобы посчитать, нужно замерить диаметр шестерни по зубьям.

Формула: (`π` * `gear_diameter`) / `full_steps_per_rotation`

К примеру диаметр шестерни 7 мм.

`(3,14159 * 7) / 200 = 21.99113 / 200 = 0.10995565 ~ 0.1099`

Вставляем в раздел `[extruder]` значение `rotation_distance: 0.1099` и если с редукцией, то `gear_ratio: 3:1`

### Калибровка

Калибруем исключительно подающую часть без нагрузки. БЕЗ ГОРЛА И СОПЛА!!!

В конфигурации экструдера [extruder] убираем защиты

```ini
min_extrude_temp: 0
max_extrude_only_distance: 150
```

Подаем команду на подачу 100мм прутка.

```gcode
G1 E100 F600
```

NRD - **New Rotation Distance**  
ORD - Old Rotation Distance  
RL - Required Length  
FL - Fact Length

`NRD = ORD * (RL/FL)`

Пример:  
ORD=7.9, RL=100, FL=112, NRD=7.9*(100/112)=7.05

Вставляем в раздел `[extruder]` значение `rotation_distance: 7.05`

## 3. Калибровка стола (z-offset)

Т.к. установлен датчик автоуровня (клон BLTouch) и настройка стола будет с его помощью.

!!! Датчик должен быть уже настроен !!!

!!! Проводим первично настройку через бумажку !!!

### Z-offset

Указываем в разделе `[bltouch]` значение в 10мм `z_offset: 10`

Далее делаем Home Z

После этого берем бумажку и двигаем по немногу сопло вниз пока бумажка не коснется сопла и не станет немного туговато двигаться под прижимом сопла.

Смотрим на полчившееся значение по оси Z. Вычитаем из 10 полученное и получаем итоговый z-offset

Пример:  
10 - 8.3 = **1.7**

### Screws Tilt Adjust

Хоумим все оси и далее двигаем экструдер к каждому винту стола

```text
            FRONT
      2---------------1
RIGHT |                  LEFT
      3---------------4
            BACK
```

Далее указываем в конфигурации принтера

```ini
[screws_tilt_adjust]
screw1: 70,18
screw1_name: Front-Left
screw2: 235,18
screw2_name: Front-Right
screw3: 235,187
screw3_name: Back-Right
screw4: 70,187
screw4_name: Back-Left
horizontal_move_z: 10
speed: 200
screw_thread: CW-M3
```

* `horizontal_move_z` - скорость движения по оси Z
* `speed` - скорость перемещений
* `screw_thread` - CW-M3 - это ClockWork(по часовой стрелке) и резьба M3

После этого в Toolhead выбриаем `SCREWS TILT CALCULATE`

Принтер сам пройдет по нужным точкам и выведет информацию о том, на сколько минут (по часам) и в какую сторону нужно повернуть рукоять винта.

После этого снятие карты стола будет сильно лучше, чем настройка руками.

## 4. Калибровка Pressure Advance

Используем Web калибровщик для создания нужного G-кода по своим параметрам: <https://k3d.tech/calibrations/la/calibrator/>

Полученные значения не фиксируем в конфиге. Их нужно фиксировать в слайсере.
Klipper не поддерживает команду M900 из Marlin, но можно создать макрос, который преобразуем этот G-код в команды Klipper.
А также будет сообщать о выставленных значениях в Web консоль

```ini
[gcode_macro M900]
# M900 K{pressure_factor} S{pressure_smooth_time}
description: Replace M900 Linear Advance
gcode:
  {% set K = params.K|default(0.0)|float %}
  {% set S = params.S|default(0.04)|float %}
  RESPOND TYPE=echo MSG='{"SET PRESSURE_ADVANCE ADVANCE=\"%3.3f\", SMOOTH_TIME=\"%3.3f\"" % (K, S)}'
  SET_PRESSURE_ADVANCE ADVANCE={K} SMOOTH_TIME={S}
```

Далее нужно указывать для конкретного филамента его значение калибровки

К примеру в PrusaSlicer в меню `Настройки филамента` -> `Пользовательский G-код` -> `Стартовый G-код` добавляем что-то вроде:

```gcode
M900 K0.09 ; K-Factor Linear/Pressure Advance
```

## 5. Калибровка откатов (Firmware retraction)

Используем Web калибровщик для создания нужного G-кода по своим параметрам <https://k3d.tech/calibrations/retractions/calibrator/>

## 6. Калибровка потока

Этот метод калибровки считается устаревшим, но можно его тоже попробовать: [Калибровка экструдера и потока 3D принтера](https://www.youtube.com/watch?v=Mga_ezYDTNI)  
Нужен микрометр, либо цифровой штангенциркуль.

!!! Но лучше выравнивать поток через печать тестовых моделей и подгоняя поток в сторону наилучшего визуального результата.

## 7. РУЧНАЯ Калибровка шейперов(Input Shaping) / Ускорений

Исходная методика: [Ручной подбор частоты Input Shaping](https://k3d.tech/calibrations/manual_is_calibration/)

Пробуем разные шейперы. Калибровка в целом не особо быстрая если идти от MZV -> EI -> 2HUMP_EI (3HUMP_EI это крайний случай)

Сначала нужно выставить параметры в разделе `[printer]`. Параметр `square_corner_velocity` имеет 3 значения 5, 10 и 15.

```ini
max_accel: 6000
minimum_cruise_ratio: 0
square_corner_velocity: 10.0
```

* `max_accel` - ставим максимум ускорений. Для Ender 3 с базовой механикой в целом можно выставить 5000-6000
* `square_corner_velocity` - можно начать с 5 и потом если почувствуете в принтере силы, то 10.

Из-за большого кол-ва комбинайций такая калибровка может затянутся по времени.  
3 варианта square_corner_velocity, 3 варианта шейпера (3hump_ei не берем в расчет) по 2 башни для каждого шейпера  
`3*3*2 = 18` башен для проверки всех вариантов  
Не говоря уже о тестовых моделях для каждого итогового набора шейперов

Вписываем пока пустой параметр `[input_shaper]`

### 7.1. Шейпер MZV

Подготавливаем модель по инструкции по ссылке.

Далее ставим шейпер MZV на обе оси

```txt
SET_INPUT_SHAPER SHAPER_TYPE_X=mzv SHAPER_TYPE_Y=mzv
```

Первая итерация калибровки. Ось Y  
Отключаем шейпер на ось X и включаем калибровку башни

```txt
SET_INPUT_SHAPER SHAPER_FREQ_X=0
TUNING_TOWER COMMAND=SET_INPUT_SHAPER PARAMETER=SHAPER_FREQ_Y START=30 STEP_DELTA=5 STEP_HEIGHT=3
```

После окончания печати (или остановки) смотрим где эхо меньше всего  
Выставляем его для оси Y и калибруем ось X

К примеру получили для Y значение `40`

```txt
SET_INPUT_SHAPER SHAPER_FREQ_Y=40
```

Включаем калибровку башни

```txt
TUNING_TOWER COMMAND=SET_INPUT_SHAPER PARAMETER=SHAPER_FREQ_X START=40 STEP_DELTA=10 STEP_HEIGHT=3
```

### 7.2. Шейпер EI

Ставим шейпер EI на обе оси

```txt
SET_INPUT_SHAPER SHAPER_TYPE_X=ei SHAPER_TYPE_Y=ei
```

Повторяем команды для осей из раздела 7.1. шейпера MZV

### 7.3. Шейпер 2HUMP_EI

Этот шейпер шире по частотам, поэтому числа будут другие.

Калибруем ось Y. На ось X ставим сразу 50

```txt
SET_INPUT_SHAPER SHAPER_TYPE_X=2hump_ei SHAPER_TYPE_Y=2hump_ei SHAPER_FREQ_X=50
TUNING_TOWER COMMAND=SET_INPUT_SHAPER PARAMETER=SHAPER_FREQ_Y START=50 STEP_DELTA=10 STEP_HEIGHT=3
```

После оси Y калибруем ось X. Для оси Y ставим значение которое выбрали после печати. К примеру 60

```txt
SET_INPUT_SHAPER SHAPER_TYPE_X=2hump_ei SHAPER_TYPE_Y=2hump_ei SHAPER_FREQ_Y=60
TUNING_TOWER COMMAND=SET_INPUT_SHAPER PARAMETER=SHAPER_FREQ_X START=50 STEP_DELTA=10 STEP_HEIGHT=3
```

На крайний случай можно провести калибровку 3HUMP_EI. Набор команд такой же кроме самого шейпера `=3hump_ei`

### 7.4. Подбор максимальных ускорений печати

Подбираем максимальные адекватные ускорения для печати

К примеру у нас получились следующие значения

* `mzv` y=35 x=60
* `ei` y=40 x=50
* `2hump_ei` y=60 x=60

По изображениям из источника подбираем ближайшие частоты

* `mzv` 2800
* `ei` 3000
* `2hump_ei` 4000

Выбираем наболее удовлетворящий шейпер и частоту.  
Я к примеру выберу тут `mzv` и ускорение 2800

Вносим эти значения в параметр `[input_shaper]`

```ini
[input_shaper]
shaper_type_y: mzv
shaper_freq_y: 35
shaper_type_x: mzv
shaper_freq_x: 60
```

И правим параметр `max_accel` в разделе `[printer]`

```ini
[printer]
max_accel: 2800
```

### 7.5. Подбор актуального ускорения

После фиксации максимальных параметров подберем на этой же башне с максимальной скоростью печати эффективные ускорения  
Начнем с 500

```txt
TUNING_TOWER COMMAND=SET_VELOCITY_LIMIT PARAMETER=ACCEL START=500 STEP_DELTA=500 STEP_HEIGHT=3
```

Согласно высоте модели по 3мм, получаем 20 сегментов, если мы начинаем с 500 с шагом в 500, то на последнем участке получим 10000 ускорения.

После печати ищем самый лучший участок. К примеру может получиться так, что данный тест выдаст значения лучшие расчетного, к примеру 2500 мм/с2

Также можно повторить печать башни но уже с более маленьким шагом для более точного подбора.

К примеру при печати мы получили 2500 при шаге в 500, расчетный 2800.  
Значит нам нужно башню из 20 сегментов по 3мм поделить между 2300 и 2800 т.е. всего 500(2800-2300), т.е. шаг 25 мм/сек2

```txt
TUNING_TOWER COMMAND=SET_VELOCITY_LIMIT PARAMETER=ACCEL START=2300 STEP_DELTA=25 STEP_HEIGHT=3
```

В идеале должен подтвердиться расчетный 2800

Финально смотрим на башню и выбираем наилучший результат, либо что-то среднее.

```ini
[printer]
max_accel: 2800
```

### 7.6. Подбор актуальной скорости

Продолжим поиск оптимальной скорости печати c найденными ускорениями. Нужно нарезать модель так, чтобы скорости в ней выходили за рамки калибровки.
Иначе выставление новой скорости через TUNING_TOWER не будет давать результатов после лимита в слайсере.

```txt
TUNING_TOWER COMMAND=SET_VELOCITY_LIMIT PARAMETER=VELOCITY START=30 STEP_DELTA=5 STEP_HEIGHT=3
```

Начинаем с 30 мм/сек с шагом в 5 мм/сек, на последнем участке получим 125 мм/сек

После печати выставляем **в слайсере** удовлетворяющее нас значение. Например хорошо выглядит 125 мм/сек и 65 мм/сек  
Печать периметров можно выставить 125, а малых периметров 65

Печатаем проверочную модель, если не устраивает 125, то снижаемся сразу к 65 и печатаем повторно

На данном этапе модель уже должна давать хорошее качество (в зависимости от принтера)

## 8. АВТОМАТИЧЕСКАЯ Калибровка шейперов(Input Shaping) / Ускорений

На примере акселерометра [BTT S2DW он же LIS2DW v1.02](https://github.com/bigtreetech/LIS2DW)

Прикрепляем к экструдеру принтера (если есть две штуки, то и к столу тоже). В комплекте как правило идет винт для крепления вместо сопла экструдера.  
Но если есть возможность, можно прикрепить и отдельно, чтобы не разбирать экструдер.
Для стола придется "колхозить/печатать" крепление

Подключаем в плату с Klipper по USB

Находим устройство

```sh
ls -l /dev/serial/by-id/ | grep rp2040

lrwxrwxrwx 1 root root 13 Jan 20 21:57 usb-Klipper_rp2040_btt_acc-if00 -> ../../ttyACM0
```

Прошиваем микроконтроллер акселерометра т.к. он подключен у нас по USB

```sh
cd ~/klipper
make clean
make menuconfig
```

Выбираем:

* Micro-controller Architecture (Raspberry Pi RP2040/RP235x)
* Processor model (rp2040)
* Bootloader offset (No bootloader)
* Flash chip (GENERIC_03H with CLKDIV 4)

Компилируем

```sh
make
```

В папке `out` будет лежать файл `klipper.uf2`. Его и нужно прошить в акселерометр.
Для этого нужно перевести его в режим загрузчика (DFU).  
Отключаем USB от платы акселерометра, зажимаем на нем кнопку и с зажатой кнопкой (не отпуская) подключаем обратно.

Ищем устройство

```sh
lsusb | grep RP2

Bus 007 Device 005: ID 2e8a:0003 Raspberry Pi RP2 Boot
```

Нам нужен идентификатор устройства `2e8a:0003`

Прошиваем

```sh
make flash FLASH_DEVICE=2e8a:0003
```

Повторно ищем устройство т.к. оно могло сменить адрес после прошивки

```sh
ls -l /dev/serial/by-id/ | grep rp2040

lrwxrwxrwx 1 root root 13 Jan 20 22:25 usb-Klipper_rp2040_50445059382E591C-if00 -> ../../ttyACM1
```

Как видим действительно сменило

Создаем отдельный файл конфигурации `bigtreetech-lis2dw-v1.0.cfg`  
Вносим в него настройки

```ini
# This file contains common pin mappings for the bigtreetech lis2dw v1.0.1
# To use this config, the firmware should be compiled for the
# RP2040 with "USB"
# The micro-controller will be used to control the components on the nozzle.

# See docs/Config_Reference.md for a description of parameters.

# Source: https://github.com/bigtreetech/LIS2DW/blob/master/Firmware/sample-bigtreetech-lis2dw-v1.0.cfg

# Doc: https://www.klipper3d.org/Measuring_Resonances.html#measuring-the-resonances

[mcu btt_lis2dw_serial]
serial: /dev/serial/by-id/usb-Klipper_rp2040_50445059382E591C-if00

# Doc: https://www.klipper3d.org/Config_Reference.html#lis2dw
[lis2dw]
cs_pin: btt_lis2dw_serial:gpio9
#spi_bus: spi1a
spi_software_sclk_pin: btt_lis2dw_serial:gpio10
spi_software_mosi_pin: btt_lis2dw_serial:gpio11
spi_software_miso_pin: btt_lis2dw_serial:gpio8
axes_map: -y, -x, z

# Doc: https://www.klipper3d.org/Config_Reference.html#resonance_tester
[resonance_tester]
accel_chip: lis2dw
probe_points: 117.5, 117.5, 20
min_freq: 30
max_freq: 135
move_speed: 150
```

Отдельно стоит сказать про `axes_map`. На акселерометре есть шелкография указывающая где у него оси X и Y.
Соответственно если закрепить акселерометр иначе, то придется указать axes_map отличный от базового `x, y, z`.
Визуальное определение может быть проблематичным поэтому можно воспользоваться отдельным инструментом.

Добавляем в файл конфигурации принтера

```ini
####################################################################################
# ACCELEROMETER
####################################################################################

[include bigtreetech-lis2dw-v1.0.cfg]

####################################################################################
```

Акселерометр должен определиться как еще одно MCU

Проверяем ответ

```txt
ACCELEROMETER_QUERY
```

Видим ответ

```txt
accelerometer values (x, y, z): -9762.716208, 497.707101, 57.427742
```

Пробуем команду `MEASURE_AXES_NOISE` получаем некоторые базовые числа уровня шума с акселерометра на осях

```txt
Axes noise for xy-axis accelerometer: 672.011523 (x), 381.487674 (y), 239.147542 (z)
```

### 8.1. Плагин Klipper Shake&Tune

[Klipper Shake&Tune plugin](https://github.com/Frix-x/klippain-shaketune)

Устанавливаем по инструкции. После создаем отдельный раздел в конфигурации акселерометра `bigtreetech-lis2dw-v1.0.cfg`

Но сначала создадим отдельную папку где будут размещаться результаты

```sh
mkdir ~/printer_data/config/shaketune_results
```

```ini
####################################################################################
# Klipper Shake&Tune plugin
# https://github.com/Frix-x/klippain-shaketune
####################################################################################

[shaketune]
result_folder: ~/printer_data/config/shaketune_results
```

Паркуем оси `G28` и запускаем команду `AXES_MAP_CALIBRATION`

Получим в консоль нужную информацию. К примеру `==> Detected axes_map: y, -z, -x`

Можем также пойти в папку `shaketune_results`(через Web) и найти там папку `axes_map` там будет изображение с визуализацией проверки.

Вносим правки, еще раз проверяем и по сути можем приступать к любым калибровкам с помощью акселерометра.

Проводим калибровку оси на которой установлен датчик. Для Ender 3 это оси X и Y (обе оси если стоит два датчика, иначе нужно переставлять датчик)

```txt
G28
AXES_SHAPER_CALIBRATION AXIS=X
AXES_SHAPER_CALIBRATION AXIS=Y
```

Далее следуем иструкции и вносим изменения.

По оси X максимальные ускорения к примеру 7780 для шейпера MZV  
По оси Y максимальные ускорения к примеру 3970 для шейпера MZV  
Исходя из этого выбираем минимальную(т.е. для Y), либо еще раз калибруем для полученных шейперов.

```ini
[printer]
max_accel: 3970

[input_shaper]
shaper_type_y: mzv
shaper_freq_y: 36.8
damping_ratio_y: 0.070
shaper_type_x: mzv
shaper_freq_x: 51.6
damping_ratio_x: 0.048
```

Чтобы не выходить за максимальные показатели пересчитаем значения, начнем также с 500.  
Шаг: (3970-500)/19=~182  
Итог: 182*19+500=3958

```txt
TUNING_TOWER COMMAND=SET_VELOCITY_LIMIT PARAMETER=ACCEL START=500 STEP_DELTA=182 STEP_HEIGHT=3
```
