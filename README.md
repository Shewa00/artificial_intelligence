# КУРСОВАЯ РАБОТА

## 1.	Введение

Индустрия компьютерных игр (также индустрия интерактивных развлечений) — сектор экономики, связанный с разработкой, продвижением и продажей компьютерных игр. В неё входит большое количество специальностей, по которым работают десятки тысяч человек по всему миру.
Индустрия компьютерных игр зародилась в середине 1970-х годов как движение энтузиастов и за несколько десятилетий выросла из небольшого рынка в мэйнстрим. На рынке работают как крупные игроки, так и небольшие фирмы и стартапы, а также независимые разработчики и сообщества.

Современные персональные компьютеры множеством новшеств обязаны игровой индустрии. К числу самых значимых относят звуковые и графические карты, CD- и DVD-приводы, Unix и центральные процессоры.
Звуковые карты изначально были разработаны для интегрирования качественного цифрового звука в компьютерные игры, и только потом звуковое оборудование было усовершенствовано под нужды меломанов[2].
Графические карты, которые на заре компьютерной эпохи эволюционировали в направлении увеличения количества поддерживаемых цветов, позже стали развиваться для аппаратной поддержки графических интерфейсов пользователя (англ. GUI) и игр. Для GUI требовалось увеличение разрешения экрана, а для игр — ускорение трёхмерной графики.

Существует огромное разнообразие жанров геймдева и практически в каждой игре так или иначе используются искусственный интеллект для управления ботов или неигровых персонажей.
Неигровой персонаж (сокр. NPC от англ. Non-Player Character — «персонаж, управляемый не игроком») — персонаж в играх, который не находится под контролем игрока. В компьютерных играх поведение таких персонажей определяется програмно, т.е. посредством искусственного интеллекта (ИИ).

## 2.	Актуальность

С каждым годом игровая индустрия развивается всё сильнее, в том числе и в сфере ИИ. NPC с каждым поколением всё умнее, а методов их поведения всё больше, что не может не заинтересовать.  


## 3.	Цель работы

Разработка игровой платформы для создания модулей с искусственным интеллектом.
4.	Задание

1)	Написать платформу для тестирования модулей искусственного интеллекта, используя кроссплатформенный фреймворк Qt.
2)	Реализовать собственную библиотеку, отвечающую за поведение неигровых персонажей.
3)	Выполнить апробацию на примере игры.
4)	Unit-tests и CI
5)	Оформление пояснительной записки и защита курсового проекта.

## 5.	Ход работы

### 5.1	Теоретическая часть

#### 1)	Платформа для тестирования:
Сцена на которую помещаются Npc и другие объекты, производящая обмен данными между объектами: 
1.1)	Собирает информацию об объектах, с помощью методов этих объектов.
1.2)	Передавать эту информацию каждому объекту, через функции этих объектов.
1.3)	Определяет момент окончания игры с помощью анализа жизни каждого объекта.
1.4)	Генерирует промежуточные объекты(пули).
 
#### 2)	Библиотека, отвечающая за поведение неигровых персонажей:
Рассматривает состояние покоя объекта и его агрессивное состояние, умеет переключаться между ними посредством его булевых состояний.
2.1) В спокойном состоянии анализирует достижимость цели, если цель не достижима – перемещается назад по пройденному в агрессии пути.
2.2) В момент агрессии запоминает пройденный маршрут, рассчитывает расстояние до цели, после чего выбирает между погоней или стоячей стрельбой.
3)	Игра: совокупность пунктов (1) и (2).

### 5.2	Практическая часть

####Реализация искусственного интеллекта:
```
/*!
Слот обработки игры.
*/
void slotGameTimer();                                          (1)                                                                    

/*!
функция, отвечающая за атаку на противников.
*/
void agressiveFunction();                                      (2)


/*!
Функция, отвечающая за передвижение объекта в состоянии покоя по заданному маршрута.
*/
void passiveFunction();                                        (3)

/*!
Функция, отвечающая за передвижение объекта в состоянии покоя до заданного маршрута (если он не доступен из-за коллизий).
*/
void returnOnPassiveWay();                                     (4)

/*!
функция поиска цели, после того как она была потеряна.
*/
void findTarget();                                             (5)
```
В слоте (1) анализируется ситуация состояния агрессии (с помощью переменной nowAgressive).
 
Если моб агрессивен в данный момент (nowAgressive = true), то запускается функция агрессии (2), в ней делается проверка на наличие цели и собирается информация о внеплановом маршруте, если цель найдена, то взависимости от расстояния Npc либо стреляет и идёт к нему, либо просто стреляет. 
 
Если цели нет, то моб идет к последнему месту где была замечена цель (5).
 
Если же моб в данный момент пассивен, то идет проверка на беспрепятственное достижение к точке основного маршрута, если точка доступна, то запускается слот (3).
 
Если же нет, то запускается слот возврата к главному маршруту (4).
Информация о всех объектах передается в слот анализа мишеней:
/*!
Функция анализа живых мишеней.
targets - доступные мишени.
*/
void attackAnalizeFunction(std::vector <Target> targets);



#### Платформа, на которую помещаются объекты:

 Помимо установления объектов, необходимо подключить слоты:

##### Для Player:
```
// Слот передвижения игрока
connect(this, &Widget::playerGo, player , &Player::Go);
// Соединяем сигнала стрельбы с графической сцены со слотом разрешения стрельбы игрока
connect(scene, &MouseControl::signalShot, player, &Player::slotShot);
// Соединяем сигнал на создание пули со слотом, создающим пули в игре
connect(player, &Player::signalBullet, this, &Widget::slotBullet);
// Отслежка мыши
connect(scene, &MouseControl::signalTargetCoordinate, player, &Player::slotTarget);
// Отслежка состояния жизни
connect(player, &Player::endOfLife, this, &Widget::endOfLifeSlot);
```
##### Для Npc:
```
// Отслежка состояния жизни
connect(npc1, &Npc::signalBullet, this, &Widget::slotBullet);
// Отслежка состояния жизни
connect(npc1, &Npc::endOfLife, this, &Widget::endOfLifeSlot);
```
##### Общие: 
Отслежка конца игры:
```
endTimer = new QTimer();
connect(endTimer, &QTimer::timeout, this, &Widget::endSlot);
endTimer->start(10);
```
Добавляем все объекты в вектор участников сцены:
```
men.push_back(player);
men.push_back(npc1);
men.push_back(npc2);
men.push_back(npc3);
```
## 6.	Итог

Был разработан механизм перемещения и взаимодействия объектов на сцене, таким образом,
что каждый объект сам способен анализировать ситуацию, в которую он попал.
Получившаяся платформа подходит для тестированя NPC с искусственным интеллектом.
Платформа также поддерживает «внеплановую ситуацию» на вроде действий со стороны
тестировщика посредством объекта типа Player. 
