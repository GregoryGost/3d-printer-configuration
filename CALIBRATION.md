# Калибровка принтера

3D принтер **Ender 3 Pro** с апгрейдами:

* Плата **BigTreeTech SKR Mini E3 2.0**
* Экструдер **BMG clone**
* Хотенд **E3DV6**
* Датчик уровня стола **Trianglelab 2021 v3 3D Touch**
* Собственное спроектированное крепление **E3BMGS - Крепление директ BMG(clone) + E3DV6 экструдера**

## Материалы

Порядок калибровки основан на материалах:

* <Дмитрий Соркин>: [Как увеличить точность 3D принтера?](https://youtu.be/POG_Th2k3_I)
* <Дмитрий Соркин>: [Калибровка ускорений и рывков 3D принтера](https://youtu.be/t5dJHWF-RGo)
* <Дмитрий Соркин>: [Делаем откаты удобными - Firmware retraction](https://youtu.be/30Kh-LyFH2w)

### 1. Калибровка PID Экструдера и Стола

```gcode
;PID autotune Extruder
M303 E0 S235 C8 U

;PID autotune Bed
M303 E-1 S80 C8 U

;Save to EEPROM
M500
```

(2) Калибровка шагов/разрешения экструдера

```gcode
;Turn on the extruder without heating
M302 S0

;Set relative coordinates for the extruder
M83

;Extrude 100mm plastic
G1 E100 F600

;Set absolute coordinates for the extruder
M82
```

NS - **New Step**  
OS - Old Step  
RL - Required Length  
FL - Fact Length

`NS = OS * (RL/FL)`

```gcode
;Set Axis Steps-per-unit
M92 E410

;Save to EEPROM
M500
```

Пример:  
OS=460, RL=100, FL=112, NS=460*(100/112)=410.7

(3) Калибровка ускорений и рывков

Использовался метод калибровки из видео: [Калибровка ускорений и рывков 3D принтера](https://www.youtube.com/watch?v=t5dJHWF-RGo)

Высота первого слоя 0.3 мм

PrusaSlicer-2.3.3 (Макрос ускорения)

```gcode
{if layer_z == 0.3}
M900 K0 ; Disable Linear Advance
M201 X1700 Y1700 ; Max Acceleration
M204 P1000 T1600 ; Print 1000, Travel 1600
M205 J0.01 ; Junction Deviation Max 0.2
{elsif layer_z == 4.90}M205 J0.02
{elsif layer_z == 9.90}M205 J0.03
{elsif layer_z == 14.90}M205 J0.04
{elsif layer_z == 19.90}M205 J0.05
{elsif layer_z == 24.90}M205 J0.06
{elsif layer_z == 29.90}M205 J0.07
{elsif layer_z == 34.90}M205 J0.08
{elsif layer_z == 39.90}M205 J0.09
{elsif layer_z == 44.90}M205 J0.1
{elsif layer_z == 49.90}M205 J0.11
{elsif layer_z == 54.90}M205 J0.12
{elsif layer_z == 59.90}M205 J0.13
{elsif layer_z == 64.90}M205 J0.14
{elsif layer_z == 69.90}M205 J0.15
{elsif layer_z == 74.90}M205 J0.16
{elsif layer_z == 79.90}M205 J0.17
{elsif layer_z == 84.90}M205 J0.18
{elsif layer_z == 89.90}M205 J0.19
{elsif layer_z == 94.90}M205 J0.2
{endif}
```

PrusaSlicer-2.3.3 (Макрос рывки)

```gcode
{if layer_z == 0.3}
M900 K0 ; Disable Linear Advance
M201 X1700 Y1700 ; Max Acceleration
M204 P1000 T1600 ; Print 1000, Travel 1600
M205 J0.01 ; Junction Deviation Max 0.2
{elsif layer_z == 4.90}M205 J0.02
{elsif layer_z == 9.90}M205 J0.03
{elsif layer_z == 14.90}M205 J0.04
{elsif layer_z == 19.90}M205 J0.05
{elsif layer_z == 24.90}M205 J0.06
{elsif layer_z == 29.90}M205 J0.07
{elsif layer_z == 34.90}M205 J0.08
{elsif layer_z == 39.90}M205 J0.09
{elsif layer_z == 44.90}M205 J0.1
{elsif layer_z == 49.90}M205 J0.11
{elsif layer_z == 54.90}M205 J0.12
{elsif layer_z == 59.90}M205 J0.13
{elsif layer_z == 64.90}M205 J0.14
{elsif layer_z == 69.90}M205 J0.15
{elsif layer_z == 74.90}M205 J0.16
{elsif layer_z == 79.90}M205 J0.17
{elsif layer_z == 84.90}M205 J0.18
{elsif layer_z == 89.90}M205 J0.19
{elsif layer_z == 94.90}M205 J0.2
{endif}
```

(4) Калибровка **Linear Pressure Control v1.5** (LIN_ADVANCE)

Для калибровки использовался [Kcalibrator](https://github.com/ArtificalSUN/Kcalibrator)  
Видео по использованию: [Kcalibrator - Альтернативная калибровка Linear Advance (Pressure Advance)](https://www.youtube.com/watch?v=p9IKwwKTIFM)

Стартовый GCODE

```gcode
M900 K{if filament_type[0]=~/PETG/}0.07{else}0{endif}
```

(5) Калибровка потока

Использовался метод калибровки из видео: [Калибровка экструдера и потока 3D принтера](https://www.youtube.com/watch?v=Mga_ezYDTNI)

(6) Калибровка температуры и ретракта для различных пластиков

Калибровочная температурная башня для PETG: [Temperature Tower 230-250](https://www.thingiverse.com/thing:4642743/files)

Использовался метод калибровки из видео: [Делаем откаты удобными - Firmware retraction](https://youtu.be/30Kh-LyFH2w?t=678)

PrusaSlicer-2.3.3 (Макрос скорость ретракта). Высота башен 60мм  
Первый слой 0.3мм

```gcode
{if layer_z==0.3}M207 S2 F{60*60} ; 60mm/s
{elsif layer_z==5.10}M207 S2 F{55*60} ; 55mm/s
{elsif layer_z==10.10}M207 S2 F{50*60} ; 50mm/s
{elsif layer_z==15.10}M207 S2 F{45*60} ; 45mm/s
{elsif layer_z==20.10}M207 S2 F{40*60} ; 40mm/s
{elsif layer_z==25.10}M207 S2 F{35*60} ; 35mm/s
{elsif layer_z==30.10}M207 S2 F{30*60} ; 30mm/s
{elsif layer_z==35.10}M207 S2 F{25*60} ; 25mm/s
{elsif layer_z==40.10}M207 S2 F{20*60} ; 20mm/s
{elsif layer_z==45.10}M207 S2 F{15*60} ; 15mm/s
{elsif layer_z==50.10}M207 S2 F{10*60} ; 10mm/s
{endif}
```

PrusaSlicer-2.3.3 (Макрос длина ретракта). Высота башен 80мм

```gcode
{if layer_z==0.3}M207 S2.5 F{25*60} ; 25mm/s
{elsif layer_z==5.10}M207 S2.6 F{25*60} ; 25mm/s
{elsif layer_z==10.10}M207 S2.7 F{25*60} ; 25mm/s
{elsif layer_z==15.10}M207 S2.8 F{25*60} ; 25mm/s
{elsif layer_z==20.10}M207 S2.9 F{25*60} ; 25mm/s
{elsif layer_z==25.10}M207 S3 F{25*60} ; 25mm/s
{elsif layer_z==30.10}M207 S3.1 F{25*60} ; 25mm/s
{elsif layer_z==35.10}M207 S3.2 F{25*60} ; 25mm/s
{elsif layer_z==40.10}M207 S3.3 F{25*60} ; 25mm/s
{elsif layer_z==45.10}M207 S3.4 F{25*60} ; 25mm/s
{elsif layer_z==50.10}M207 S3.5 F{25*60} ; 25mm/s
{elsif layer_z==55.10}M207 S3.6 F{25*60} ; 25mm/s
{elsif layer_z==60.10}M207 S3.7 F{25*60} ; 25mm/s
{elsif layer_z==65.10}M207 S3.8 F{25*60} ; 25mm/s
{elsif layer_z==70.10}M207 S3.8 F{25*60} ; 25mm/s
{elsif layer_z==75.10}M207 S4 F{25*60} ; 25mm/s
{endif}
```
