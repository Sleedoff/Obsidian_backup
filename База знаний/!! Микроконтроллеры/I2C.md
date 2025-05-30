I2C - последовательная асимметричная шина для связи между интегральными схемами внутри электронных приборов.
##### Физический уровень
Данные передаются по двум проводам - провод данных и провод тактов. Есть ведущее устройство (master) и ведомое устройство (slave). Такты генерирует ведущий, а ведомое устройство их только принимает при приеме байта. На одной шине может быть подключено до 127 устройств, схема подключения - "монтажное И".
![[Снимок экрана 2024-11-25 180326.png]]
##### Монтажное И
Есть линия, подтянутая резистором к плюсу питания. Так как сопротивление между линией и землей бесконечность, а между питанием и линией равно резистору, то напряжение на линии равно напряжению питания. То есть высокий уровень, обычно логическая единица.
![[Снимок экрана 2024-11-25 181444.png]]
К линии подключены девайсы, которые могут замыкать линию на землю. Если один из девайсов замкнет линию на землю, то на всей шине станет логический ноль.
Подобная схема может применяться в схемах "готовности", все девайсы в начале своей работы отрабатывают определенный алгоритм инициализации, после отработки которого размыкают цепь, тем самым, как только все девайсы разомкнут линию, на главное устройство поступит высокий уровень сигнала, что будет означать, что все устройства готовы к работе.
##### Прием и передача сигнала
Передача и прием сигналов осуществляется прижиманием линии в 0, а к единице линия восстанавливается сама за счёт подтягивающих резисторов. Чем больше резистор, тем дольше линия восстанавливается в единицу и тем сильнее заваливаются фронты импульсов, а значит скорость передачи падает.
Обычная скорость передачи равна 10кбит/с - в медленном режиме, либо 100кбит/с в быстром. 
Передача данных состоит из: 
- Старт бита
- Информации
- Стоп бита
![[Снимок экрана 2024-11-27 094334.png]]
![[Снимок экрана 2024-11-27 193620.png]]
Шина I2C совершенно не зависит от временных интервалов, так как считывание информационного бита происходит только в момент высокого сигнала на линии SCL. 
- Начало передачи определяется **Старт-битом** последовательностью - провал SDA при высоком уровне SCL.
- При передаче информации **от Master к Slave**, ведущий генерирует такты SCL и выдает биты SDA. Ведомый считывает данные, когда на линии SCL высокий сигнал.
- При передаче информации **от Slave к Master**, ведущий генерирует такты SCL и считывает данные, которые генерирует ведомый. Ведомый, при 0 на SCL выставляет бит данных, который в дальнейшем будет считывать ведущий, когда на линии SCL будет 1.
- Конец передачи - **Стоп-бит** последовательность - при высоком уровне SCL, линия SDA переходит с низкого на высокий уровень.
==Изменение состояния на шине данных происходит в момент, когда на линии тактового сигнала низкий уровень сигнала.==
Если **Slave** торомоз и не успевает (у EEPROM, например, низкая скорость записи), то он может **насильно положить линию SCL в землю и не давать ведущему генерировать новые такты**. Мастер должен это понять и дать слейву прожевать байт. Так что нельзя тупо генерить такты, при отпускании **SCL** надо следить за тем, что линия поднялась. Если не поднялась, то надо остановиться и ждать до тех пор, пока Slave ее не отпустит. Потом продолжить с того же места
![[Снимок экрана 2024-11-27 193840.png]]
##### Пакеты данных
Пакет данных состоит из 9 бит: 8 бит данных и 1 бита подтверждения/не подтверждения приёма.
Первый пакет - физический адрес устройства и бит направления.
![[Снимок экрана 2024-11-27 194154.png]]
После адресного пакета идут пакеты с данными в ту или иную сторону. 
![[Снимок экрана 2024-11-27 195637.png]]
![[Снимок экрана 2024-11-27 195707.png]]
##### Реализация I2C в STM32
