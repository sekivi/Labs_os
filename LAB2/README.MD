# Отчет по лабораторной работе №2
Студент группы БВБО-01-16 Левкин Кирилл Сергеевич


## Цель работы"

* настроить работоспособную систему с использованием lvm, raid
* эмулировать отказ одного из дисков
* произвести замену дисков на лету, с добавлением новых дисков и переносом разделов.

## Задание 1 (Установка ОС и настройка LVM, RAID)

Результат: Виртуальная машина с дисками ssd1, ssd2

## Задание 2 (Эмуляция отказа одного из дисков)

* Выполнено удаление диска ssd1 в свойствах машины
* Произведено удааление ssd1.vmdk с локальной машины
* Выполнили перезагрузку и убедились, что виртуальная машина все еще работает
* проверили статус RAID-массива: cat /proc/mdstat
![](https://github.com/sekivi/Labs_os/blob/master/LAB2/Screenshots/2aHp9q6pQeU.jpg)
* добавили в интерфейсе VM новый диск такого же размера и назовали его ssd3
* выполнили операции:
* посмотр что новый диск приехал в систему командой fdisk -l
* Копирование таблиц разделов со старого диска на новый: sfdisk -d /dev/XXXX | sfdisk /dev/YYY
* Добавление в рейд массив нового диска: mdadm --manage /dev/md0 --add /dev/YYY
* Результат работы:
![](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_2/VirtualBox_pojiloi_raid_2_06_04_2019_12_01_54.png)
* Выполнениие синхронизации разделов, не входящих в RAID.
* Установка grub на новый диск
![](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_2/VirtualBox_Клон_pojiloi_raid_11_05_2019_14_25_43.png)
9) Выполнение перезагрузки ВМ, для того чтобы убедиться что все работает
![](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_2/VirtualBox_Клон_pojiloi_raid_11_05_2019_14_27_15.png)

В этом задании научился:
* Удалять диск ssd1
* Проверять статус RAID-массива
* Копировать таблицу разделов со старого диска на новый
* Добавлять в рейд массив новый диск
* Выполнять синхронизацию разделов, не входящих в RAID
Результат: Был удален диск ssd1, сохранен диск ssd2, добавлен диск ssd3.

## Задание 3 (Добавление новых дисков и перенос раздела)

1) Проэмулировал отказ диска ssd2, удалив из свойств ВМ диск и перезагрузившись
2) Текущее состояние дисков и RAID:
![](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.1_pojiloi_raid_11_05_2019_14_38_42.png)
![](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.1_pojiloi_raid_11_05_2019_14_39_08.png)
![](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.1_pojiloi_raid_11_05_2019_14_39_42.png)

3) Добавляю один новый ssd диск, назвав его ssd4, а после добавления проверьте что произошло:
![](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.1_pojiloi_raid_11_05_2019_14_44_43.png)
4) Перенос данных с помощью LVM.
* Копирование файловую таблицу со старого диска на новый
* Копирование данных /boot на новый диск  
* Перемонтировака /boot на живой диск
* Установка grub на новый диск
Grub устанавливаем, чтобы могли загрузить ОС с этого диска
* Создание нового RAID-массива с включением туда только одного нового ssd диска:
* Результат
![](https://github.com/sekivi/Labs_os/blob/master/LAB2/Screenshots/ryRea9g9_0I.jpg)
Появился /dev/md63
5) Настройка LVM
* Выполнение команды pvs для просмотра информации о текущих физических томах
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.1_pojiloi_raid_11_05_2019_15_00_11.png)
* Создание нового физического тома, включив в него ранее созданный RAID массив:
* После выполнения команд lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT и pvs яувидел, что к md63 добавился FSTYPE - LVM2_member, так же dev/md63 добавился к результату команды pvs.
* Увеличение размера Volume Group system
* Выполнение команд
```
vgdisplay system -v
pvs
vgs
lvs -a -o+devices
```
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.1_pojiloi_raid_11_05_2019_15_05_10.png)
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.1_pojiloi_raid_11_05_2019_15_06_48.png)
LV var,log,root находятся на /dev/md0
* Перемещение данных со старого диска на новый
* Изменение VG, удалив из него диск старого raid.
* Выполнение команд:
```
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT
pvs
vgs
```
[alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.1_pojiloi_raid_11_05_2019_15_16_04.png)

В выводе команды pvs у /dev/md0 исчезли VG и Attr. 
В выводе команды vgs #PV - уменьшилось на 1, VSize, VFree - стали меньше

7. Удаление ssd3 и добавление ssd5,hdd1,hdd2.
8. Полсе добавления дисков:
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_08_55_12.png)

9. Восстановление работы RAID массива:
* Копирование таблицы разделов:
10. Копирование загрузочного раздела /boot с диска ssd4 на ssd5
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_08_58_40.png)
11. Установка grub на ssd5
!
[alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_08_59_38.png)
12. Изменение размера второго раздела диска ssd5
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_03_04.png)
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_01_57.png)
13. Перечитывание таблицы разделов
* Добавление нового диска к текущему raid массиву
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_12_23.png)
* Расширение количество дисков в массиве до 2-х штук:
14. Увеличение размера раздела на диске ssd4
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_18_42.png)
15. Перечитаем таблицу разделов
16. Расширение размера raid
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_26_36.png)

Размер md127 изменился, теперь он равен 5,8G.

17. Расширим размер PV.
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_28_08.png)
18. Добавление вновь появившееся место VG var,root.
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_33_25.png)
15. Создание LVM на новых hdd. 
* Имена новых hhd дисков:
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_06_02.png)
* Создание RAID массива
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_36_49.png)
* Создание нового PV на рейде из больших дисков
* Создание в этом PV группу с названием data
* Создание логического тома с размером всего свободного пространства и присвоением ему имени var_log
* Отформатирование созданного раздела в ext4
* Результат:
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_40_10.png)
20. Перенос данных логов со старого раздела на новый
* Примонтирование временно нового хранилище логов
* Выполнение синхронизации разделов
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_51_21.png)
* Процессы работающие с /var/log
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_55_07.png)
* Остановка этих процессов
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_55_50.png)
* Выполнение финальной синхронизации разделов
* Поменяем местами разделы
* Результат:
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_09_59_05.png)
21. Правим /etc/fstab.
![alt-текст](https://github.com/Teasty/admin_labs/blob/master/lab_2/screenshots_task_3/VirtualBox_Клон_3.8_pojiloi_raid_clone_23_05_2019_10_03_30.png)
23. Финальный аккорд
![](https://github.com/sekivi/Labs_os/blob/master/LAB2/Screenshots/cpOsUA_r9Vw.jpg)
![](https://github.com/sekivi/Labs_os/blob/master/LAB2/Screenshots/gI5FiSDxzO0.jpg)
