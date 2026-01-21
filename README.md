# Конфигурации 3D принтера

Здесь собраны конфигурации для моего 3D принтера **Ender 3 Pro**:

* Исполнительная плата **BigTreeTech SKR Mini E3 v2.0**
* Управляющая плата **BigTreeTech BTT-PI v1.2**
* Экструдер **BMG clone**
* Хотенд **E3DV6**
* Датчик уровня стола **Trianglelab 2021 v3 3D Touch**
* Собственное спроектированное крепление **E3BMGS - Крепление директ BMG(clone) + E3DV6 экструдер**

## Klipper (текущее)

Докуплена плата **BigTreeTech BTT-PI v1.2**  
Базовая прошивка от BTT оказалась не стабильной. За основу взят Armbian 25.11.2  
Уже в нем некорректно работал Wi-Fi, пришлось заменить `netplan.io` + `systemd-networkd` на привычный `ifupdown2`

Далее уже ставился Klipper через [Klipper Installation And Update Helper - KIAUH](https://github.com/dw-0/kiauh)

Калибровки использовались из материалов на <https://k3d.tech/calibrations/vfa/>

### Калибровка в Klipper

Информация по калибровке принтера для Klipper: [CALIBRATION_KLIPPER.md](./CALIBRATION_KLIPPER.md)

## Marlin 2 (не используется)

* Базовая конфигурация для принетра Ender 3 Pro с платой **BigTreeTech SKR Mini E3 2.0**: [(2.0.9.2)github.com/MarlinFirmware/Configurations/Creality/Ender-3 Pro/BigTreeTech SKR Mini E3 2.0](https://github.com/MarlinFirmware/Configurations/tree/release-2.0.9.2/config/examples/Creality/Ender-3%20Pro/BigTreeTech%20SKR%20Mini%20E3%202.0)
* Процесс настройки Марлин `2.0.7.2` на стриме <Виктора Шаповалова>: [Чиллстрим - Кунгфугурируем Marlin 2.0 !!!7 ЧАСОВ!!!](https://www.youtube.com/watch?v=E55FJnJHKto)
* Немного устарело, но объяснение базовых опций хорошее <Дмитрий Соркин>: [Конфигурация и установка прошивки Marlin 1.1.9](https://www.youtube.com/watch?v=pSBb9GLrM1s)
* Настройка BLTouch <Сергей Ирбис>: [Настройка Bltouch на Creality Ender-3 с платой SKR E3 DIP](https://www.youtube.com/watch?v=ELIY_ZwpuRs)
* Настройка BLTouch <Дмитрий Соркин>: [BLTouch. Стоит ли покупать? Установка, прошивка, калибровка. На примере Tevo Tornado](https://www.youtube.com/watch?v=oJgKQKbN8nE)

### Калибровка в Marlin

Информация по калибровке принтера для Marlin 2: [CALIBRATION_MARLIN.md](./CALIBRATION_MARLIN.md) (устарело, но может быть полезным местами)

### Конфигурации

* [27.12.2021] Конфигурация Марлин `2.0.9.2` для Direct BMG с двигателем **STEPPERONLINE NEMA 17** 20mm [17HS08-1004S](https://aliexpress.ru/item/32585429251.html)
* [12.10.2020] (архив) Конфигурация Марлин `2.0.7.1` **BigTreeTech SKR Mini E3 2.0** с базовым экструдером переделанным под Direct.
* [22.09.2020] (архив) Конфигурация Марлин `2.0.6.1` **BigTreeTech SKR Mini E3 2.0** с базовым экструдером переделанным под Direct.
* [19.09.2020] (архив) Конфигурация Марлин `1.1.9.1` для стоковой платы Ender 3 Pro **Creality** с базовым экструдером Bouden.
