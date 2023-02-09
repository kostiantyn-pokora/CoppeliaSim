# Навчальний посібник BubbleRob #
Цей посібник допоможе ознайомитись з усіма функціями CoppeliaSim під час розробки простого мобільного робота *BubbleRob*. Файл сцени CoppeliaSim, пов’язаний із цим матеріалом, знаходиться в *scenes/tutorials/BubbleRob*. Наступний малюнок ілюструє сцену моделювання, яку ми розробимо:

![bubbleRobTut1](bubbleRobTut1.jpg)

Оскільки цей посібник охоплюватиме багато різних аспектів, не забудьте також переглянути [інші посібники](https://www.coppeliarobotics.com/helpFiles/en/tutorials.htm), головним чином про [створення імітаційної моделі](https://www.coppeliarobotics.com/helpFiles/en/buildingAModelTutorial.htm). Перш за все, заново запустіть CoppeliaSim. Симулятор відображає стандартну [сцену](https://www.coppeliarobotics.com/helpFiles/en/scenes.htm). Ми почнемо з тіла *BubbleRob*.

Ми додаємо примітивну сферу діаметром 0,2 до сцени за допомогою [Рядок меню --> Додати --> Примітивна форма --> Сфера]. Налаштовуємо елемент **X-size** на 0,2, потім натискаємо **OK**. Створена сфера з’явиться на [шарі видимості](https://www.coppeliarobotics.com/helpFiles/en/layerSelectionDialog.htm) 1 за замовчуванням і буде [динамічною та відповідаючою](https://www.coppeliarobotics.com/helpFiles/en/designingDynamicSimulations.htm#staticAndRespondable) (оскільки ми залишили ввімкненим пункт **Створити динамічну та чуйну форму**). Це означає, що тіло *BubbleRob* буде падати та зможе реагувати на зіткнення з іншими відповідними формами (тобто змодельованими фізичним механізмом). Ми бачимо, що це [властивості динаміки форми](https://www.coppeliarobotics.com/helpFiles/en/shapeDynamicsProperties.htm): увімкнено елементи **Body is respondable** і **Body is dynamic**. Ми починаємо симуляцію (за допомогою кнопки на панелі інструментів або натисканням <control-space> у вікні сцени) і копіюємо та вставляємо створену сферу (за допомогою [Панель меню --> Редагувати --> Копіювати вибрані об’єкти], а потім [Рядок меню --> Редагувати -> Вставити буфер] або за допомогою <control-c>, потім <control-v>): дві сфери реагуватимуть на зіткнення та відкотяться. Ми зупиняємо симуляцію: дубльована сфера буде автоматично видалена. Цю типову поведінку можна змінити в [діалоговому вікні симуляції](https://www.coppeliarobotics.com/helpFiles/en/simulationPropertiesDialog.htm).
 
Ми також хочемо, щоб тіло *BubbleRob* можна було використовувати іншими модулями обчислення (наприклад, [обчислення відстані](https://www.coppeliarobotics.com/helpFiles/en/distanceCalculation.htm)). З цієї причини ми вмикаємо [Зіткнення](https://www.coppeliarobotics.com/helpFiles/en/collidableObjects.htm), [Вимірювання](https://www.coppeliarobotics.com/helpFiles/en/measurableObjects.htm) та [Виявлення](https://www.coppeliarobotics.com/helpFiles/en/detectableObjects.htm) у [загальних властивостях об’єкта](https://www.coppeliarobotics.com/helpFiles/en/commonPropertiesDialog.htm) для цієї форми, якщо вони ще не ввімкнені. Тепер за бажанням ми могли б змінити візуальний вигляд нашої сфери у [властивостях форми](https://www.coppeliarobotics.com/helpFiles/en/shapeProperties.htm).

Тепер ми відкриваємо [діалогове вікно позиції](https://www.coppeliarobotics.com/helpFiles/en/positionDialog.htm) на вкладці **переміщення**, вибираємо сферу, що представляє тіло *BubbleRob*, і вводимо 0,02 для **Along Z**. Переконаємося, що **Relative to**-item встановлено на **World**. Потім ми натискаємо **Перемістити виділення**. Це переміщує всі вибрані об’єкти на 2 см уздовж абсолютної осі Z і фактично трохи підняло нашу сферу. В [ієрархії сцени](https://www.coppeliarobotics.com/helpFiles/en/userInterface.htm#SceneHierarchy) ми двічі клацаємо псевдонім сфери, щоб мати змогу редагувати його. Вводимо *bubbleRob* і натискаємо enter.

Далі ми додаємо [датчик наближення](https://www.coppeliarobotics.com/helpFiles/en/proximitySensors.htm), щоб *BubbleRob* знав, коли він наближається до перешкод: ми вибираємо [Рядок меню --> Додати --> Датчик наближення --> Тип конуса]. У [діалоговому вікні орієнтації](https://www.coppeliarobotics.com/helpFiles/en/orientationDialog.htm) на вкладці **Орієнтація** ми вводимо 90 для **Навколо Y** і **Навколо Z**, а потім натискаємо **Повернути виділення**. У [діалоговому вікні позиції](https://www.coppeliarobotics.com/helpFiles/en/positionDialog.htm) на вкладці позиції ми вводимо 0,1 для **X-coord**. і 0,12 для **Z-коорд**. Датчик наближення тепер правильно розташований відносно тіла *BubbleRob*. Ми двічі клацаємо піктограму датчика наближення в [ієрархії сцени](https://www.coppeliarobotics.com/helpFiles/en/userInterface.htm#SceneHierarchy), щоб відкрити діалогове вікно [властивостей](https://www.coppeliarobotics.com/helpFiles/en/proximitySensorPropertiesDialog.htm). Ми натискаємо **Показати параметр об'єму**, щоб відкрити [діалогове вікно об'єму датчика наближення](https://www.coppeliarobotics.com/helpFiles/en/proximitySensorVolumeDialog.htm). Ми регулюємо елементи **Offset** на 0,005, **Angle** на 30 і **Range** на 0,15. Потім у [властивостях датчика наближення](https://www.coppeliarobotics.com/helpFiles/en/proximitySensorPropertiesDialog.htm) натискаємо **Показати параметри виявлення**. Відкриється [діалогове вікно параметрів виявлення датчика наближення](https://www.coppeliarobotics.com/helpFiles/en/proximitySensorDetectionParameterDialog.htm). Ми знімаємо позначку з пункту **Не дозволяти виявлення якщо відстань менша ніж** а потім знову закриваємо це діалогове вікно. В ієрархії сцен ми двічі клацаємо псевдонім датчика наближення, щоб відредагувати його. Вводимо *sensingNose* і натискаємо enter.

Ми вибираємо *sensingNose*, потім control-select *bubbleRob*, потім клацаємо [Рядок меню --> Редагувати --> Зробити останній вибраний об’єкт батьківським]. Це кріпить датчик до тіла робота. Ми також могли б перетягнути *sensingNose* на *bubbleRob* в ієрархії сцени. Ось що ми зараз маємо:

 ![bubbleRobTut2](bubbleRobTut2.jpg)
 
 [Датчик наближення, прикріплений до тіла bubbleRob]

Далі ми подбаємо про колеса *BubbleRob*. Ми створюємо нову сцену за допомогою [Рядок меню --> Файл --> Нова сцена]. Часто дуже зручно працювати з кількома сценами, щоб візуалізувати та працювати лише над окремими елементами. Додаємо примітивний циліндр з розмірами (0,08,0,08,0,02). Що стосується тіла *BubbleRob*, ми вмикаємо [Зіткнення](https://www.coppeliarobotics.com/helpFiles/en/collidableObjects.htm), [Вимірювання](https://www.coppeliarobotics.com/helpFiles/en/measurableObjects.htm) та [Виявлення](https://www.coppeliarobotics.com/helpFiles/en/detectableObjects.htm) у [загальних властивостях об’єкта](https://www.coppeliarobotics.com/helpFiles/en/commonPropertiesDialog.htm) для цього циліндра, якщо вони ще не ввімкнені. Потім ми встановлюємо абсолютну позицію циліндра (0,05,0,1,0,04), а його абсолютну орієнтацію (-90,0,0). Ми змінюємо псевдонім на *leftWheel*. Ми копіюємо та вставляємо колесо та встановлюємо абсолютну координату Y копії на -0,1. Перейменовуємо копію на *rightWheel*. Ми вибираємо два колеса, копіюємо їх, потім повертаємося до сцени 1 і вставляємо колеса.

Тепер нам потрібно додати [шарніри](https://www.coppeliarobotics.com/helpFiles/en/joints.htm) (або двигуни) для коліс. Ми клацаємо [Рядок меню --> Додати --> З’єднання --> Поворот], щоб додати поворотне з’єднання до сцени. У більшості випадків, коли до сцени додається новий об’єкт, цей об’єкт з’являтиметься біля початку світу. Ми зберігаємо з’єднання вибраним, потім натискаємо клавішу Control і вибираємо *leftWheel*. У [діалоговому вікні](https://www.coppeliarobotics.com/helpFiles/en/positionDialog.htm) «Позиція» на вкладці **Позиція** ми натискаємо кнопку **Застосувати до вибору**: це розташовує з’єднання в центрі лівого колеса. Потім у [діалоговому вікні орієнтації](https://www.coppeliarobotics.com/helpFiles/en/orientationDialog.htm) на вкладці **Орієнтація** ми робимо те саме: це орієнтує з’єднання так само, як і ліве колесо. Ми змінюємо назву суглоба на leftMotor. Тепер ми двічі клацаємо піктограму з’єднання в ієрархії сцени, щоб відкрити діалогове вікно [властивостей з’єднання](https://www.coppeliarobotics.com/helpFiles/en/jointProperties.htm). Потім ми натискаємо **Показати динамічні параметри**, щоб відкрити діалогове вікно [властивостей динаміки суглоба](https://www.coppeliarobotics.com/helpFiles/en/jointDynamicsProperties.htm). Ми **вмикаємо двигун** і ставимо галочку біля пункту **Lock motor when target velocity нуль**. Тепер ми повторюємо ту саму процедуру для правого двигуна та перейменуємо його на *rightMotor*. Тепер ми прикріплюємо ліве колесо до лівого двигуна, праве колесо до правого двигуна, потім приєднуємо два двигуни до *bubbleRob*. Ось що ми маємо:

![bubbleRobTut3](bubbleRobTut3.jpg)
 
[Датчик наближення, двигуни та колеса]
 
Ми запускаємо симуляцію і помічаємо, що робот падає назад. Нам досі не вистачає третьої точки контакту з підлогою. Тепер ми додаємо маленький повзунок (або колесо). У новій сцені ми додаємо примітивну сферу з діаметром 0,05 і робимо сферу здатною до [Зіткнення](https://www.coppeliarobotics.com/helpFiles/en/collidableObjects.htm), [Вимірювання](https://www.coppeliarobotics.com/helpFiles/en/measurableObjects.htm) та [Виявлення](https://www.coppeliarobotics.com/helpFiles/en/detectableObjects.htm) (якщо це ще не ввімкнено), а потім перейменуємо її на *slider*. Ми встановлюємо **Матеріал** на *noFrictionMaterial* у [властивостях динаміки форми](https://www.coppeliarobotics.com/helpFiles/en/shapeDynamicsProperties.htm). Щоб жорстко зв’язати повзунок з рештою робота, ми додаємо [об’єкт датчика сили](https://www.coppeliarobotics.com/helpFiles/en/forceSensors.htm) за допомогою [Рядок меню --> Додати --> Датчик сили]. Перейменуємо його в *підключення* і зрушимо вгору на 0,05. Ми прикріплюємо повзунок до датчика сили, потім копіюємо обидва об’єкти, повертаємося до сцени 1 і вставляємо їх. Потім ми зміщуємо датчик сили на -0,07 вздовж абсолютної осі X, а потім прикріплюємо його до тіла робота. Якщо ми запустимо симуляцію зараз, то помітимо, що повзунок трохи рухається відносно тіла робота: це тому, що обидва об’єкти (тобто *slider* і *bubbleRob*) стикаються один з одним. Щоб уникнути дивних ефектів під час моделювання динаміки, ми маємо повідомити CoppeliaSim, що обидва об’єкти не стикаються між собою, і ми робимо це таким чином: у [властивостях динаміки фігури](https://www.coppeliarobotics.com/helpFiles/en/shapeDynamicsProperties.htm) для *slider* ми встановлюємо **локальну чуйну маску** на 00001111, а для *bubbleRob*, ми встановлюємо **локальну чуйну маску** на 11110000. Якщо ми знову запустимо симуляцію, ми помітимо, що обидва об’єкти більше не заважають. Ось що ми зараз маємо: 

 ![bubbleRobTut4](bubbleRobTut4.jpg)
 
 [Датчик наближення, двигуни, колеса та повзунок]

Ми запускаємо симуляцію ще раз і помічаємо, що *BubbleRob* трохи рухається, навіть із заблокованим двигуном. Ми також намагаємося запустити симуляцію з різними фізичними двигунами: результат буде іншим. Стабільність динамічного моделювання тісно пов’язана з масами та інерцією залучених нестатичних форм. Для пояснення цього ефекту обов’язково уважно прочитайте [цей](https://www.coppeliarobotics.com/helpFiles/en/designingDynamicSimulations.htm#masses) і [той](https://www.coppeliarobotics.com/helpFiles/en/designingDynamicSimulations.htm#inertias) розділи. Зараз ми намагаємося виправити цей небажаний ефект. Вибираємо два коліщатка та повзунок, а в діалоговому вікні динаміки фігури тричі клацаємо **M=M*2 (для вибору)**. Внаслідок цього маса всіх вибраних форм буде помножена на 8. Ми робимо те саме з інерціями 3-х вибраних форм, а потім знову запускаємо симуляцію - стабільність покращилася. У діалоговому вікні спільної динаміки ми встановили **Target velocity** 50 для обох двигунів. Ми запускаємо симуляцію: *BubbleRob* тепер рухається вперед і зрештою падає з підлоги. Ми скидаємо пункт **Target velocity** до нуля для обох двигунів.
 
Об’єкт *bubbleRob* є основою всіх [об’єктів](https://www.coppeliarobotics.com/helpFiles/en/objects.htm), які пізніше сформують [модель](https://www.coppeliarobotics.com/helpFiles/en/models.htm) *BubbleRob*. Модель ми визначимо трохи пізніше. Далі ми збираємося додати [графічний об’єкт](https://www.coppeliarobotics.com/helpFiles/en/graphs.htm) до *BubbleRob*, щоб відобразити його відстань. Клацніть [Рядок меню --> Додати --> Графік] і перейменуйте його на графік. Ми приєднуємо графік до *bubbleRob* і встановлюємо абсолютні координати графіка (0,0,0.005).
 
Тепер ми встановлюємо одну **Target velocity** двигуна на 50, запускаємо симуляцію та побачимо траєкторію *BubbleRob*, що відображається на сцені. Потім ми зупиняємо симуляцію та скидаємо цільову швидкість двигуна до нуля.
 
Додаємо примітивний циліндр з такими розмірами: (0,1, 0,1, 0,2). Ми хочемо, щоб цей циліндр був статичним (тобто на нього не впливала сила тяжіння чи зіткнень), але все ж мав певну реакцію на зіткнення з нестатичними фігурами. Для цього ми вимикаємо **Body is dynamic** у [властивостях динаміки форми](https://www.coppeliarobotics.com/helpFiles/en/shapeDynamicsProperties.htm). Ми також хочемо, щоб наш циліндр можна було [Зіткнути](https://www.coppeliarobotics.com/helpFiles/en/collidableObjects.htm), [Виміряти](https://www.coppeliarobotics.com/helpFiles/en/measurableObjects.htm) та [Виявити](https://www.coppeliarobotics.com/helpFiles/en/detectableObjects.htm). Ми робимо це в [загальних властивостях об’єкта](https://www.coppeliarobotics.com/helpFiles/en/commonPropertiesDialog.htm). Тепер, поки циліндр все ще виділено, ми натискаємо кнопку панелі інструментів перекладу об’єкта:
 
![objectShiftButton](objectShiftButton.jpg)
 
Тепер ми можемо перетягувати будь-яку точку сцени: циліндр слідуватиме за рухом, але завжди буде змушений зберігати ту саму Z-координату. Копіюємо і вставляємо циліндр кілька разів і переміщуємо їх на позиції навколо *BubbleRob* (найзручніше це робити, дивлячись на сцену зверху). Під час переміщення об’єктів утримування клавіші shift дозволяє виконувати менші кроки переміщення. Утримуючи клавішу ctrl, можна рухатися в ортогональному напрямку до *звичайних* напрямків. Коли закінчите, знову натисніть кнопку панорамування камери на панелі інструментів:
 
Ми встановлюємо **Target velocity** 50 для лівого двигуна та запускаємо симуляцію: на графіку тепер відображається відстань до найближчої перешкоди, а сегмент відстані також видно на сцені. Ми зупиняємо симуляцію та скидаємо цільову швидкість до нуля.
 
Тепер нам потрібно закінчити **BubbleRob** як визначення [моделі](https://www.coppeliarobotics.com/helpFiles/en/models.htm). Ми вибираємо базу моделі (тобто об’єкт *bubbleRob*), а потім у [загальних властивостях об’єкта](https://www.coppeliarobotics.com/helpFiles/en/commonPropertiesDialog.htm) встановлюємо прапорець **Об’єкт є базою моделі**: тепер є пунктирна обмежувальна рамка, яка охоплює всі об’єкти в ієрархії моделі. Ми вибираємо два з’єднання, датчик наближення та графік, потім увімкніть пункт **Ігнорується обмежувальною рамкою моделі** та натискаємо **Застосувати до вибору** в тому самому діалоговому вікні: обмежувальна рамка моделі тепер ігнорує два з’єднання та датчик наближення. У тому самому діалоговому вікні ми вимикаємо **шар видимості камери** 2 і вмикаємо **шар видимості камери** 10 для двох з’єднань і датчика сили: це фактично приховує два з’єднання та датчик сили, оскільки шари 9-16 вимкнено за замовчуванням. У будь-який час ми можемо [змінити шари видимості для всієї сцени](https://www.coppeliarobotics.com/helpFiles/en/layerSelectionDialog.htm). Щоб завершити визначення моделі, ми вибираємо датчик зору, два коліщатка, повзунок і графік, а потім увімкніть пункт **Вибрати базу моделі**. Якщо ми зараз спробуємо вибрати об’єкт у нашій моделі на сцені, уся модель замість цього буде вибрано, що є зручним способом обробки та маніпулювання цілою моделлю як одним об’єктом. Крім того, це захищає модель від ненавмисної зміни. Окремі об’єкти в моделі все ще можна вибрати на сцені, вибравши їх клацанням миші за допомогою Control-Shift або як зазвичай вибрати їх в ієрархії сцени. Нарешті ми згортаємо дерево моделі в ієрархії сцени.
 
Далі ми додаємо [датчик зору](https://www.coppeliarobotics.com/helpFiles/en/visionSensors.htm) в тому ж положенні та орієнтації, що й датчик наближення *BubbleRob*. Ми знову відкриваємо ієрархію моделі, потім клацаємо [Рядок меню --> Додати --> Датчик зору --> Тип перспективи], потім приєднуємо датчик зору до датчика наближення та встановлюємо локальне положення та орієнтацію датчика зору на (0,0,0). Ми також переконаємося, що датчик зору не є невидимим, не є частиною обмежувальної рамки моделі, і якщо натиснути на нього, замість нього буде вибрано модель. Для того щоб налаштувати датчик зору, ми відкриваємо діалогове вікно його властивостей. Ми встановлюємо для елемента **Далека площина відсікання** значення 1, а для елементів **Дозвіл x** і **Дозвіл y** — значення 256 і 256. Ми додаємо до сцени плаваючий вигляд, а над щойно доданим плаваючим видом клацніть правою кнопкою миші [Спливаюче меню --> Перегляд --> Зв’язати перегляд із вибраним датчиком зору] (ми переконаємося, що датчик зору вибрано під час цього процесу).
 
Ми приєднуємо дочірній сценарій до датчика зору, клацнувши [Рядок меню --> Додати --> Пов’язаний дочірній сценарій --> Безпотоковий]. Ми двічі клацаємо значок, який з’явився поруч із датчиком зору в ієрархії сцен: це відкриває дочірній сценарій, який ми щойно додали. Ми копіюємо та вставляємо наступний код у [редактор скриптів](https://www.coppeliarobotics.com/helpFiles/en/scriptEditor.htm), а потім закриваємо його:

 ```function sysCall_vision(inData)
    simVision.sensorImgToWorkImg(inData.handle) -- copy the vision sensor image to the work image
    simVision.edgeDetectionOnWorkImg(inData.handle,0.2) -- perform edge detection on the work image
    simVision.workImgToSensorImg(inData.handle) -- copy the work image to the vision sensor image buffer
end

function sysCall_init()
end
``` 
Щоб побачити зображення датчика зору, ми починаємо симуляцію, а потім знову зупиняємо її.
 
Останнє, що нам знадобиться для нашої сцени, це маленький [дочірній скрипт](https://www.coppeliarobotics.com/helpFiles/en/childScripts.htm), який контролюватиме поведінку *BubbleRob*. Ми вибираємо *bubbleRob* і натискаємо [Рядок меню --> Додати --> Пов’язаний дочірній сценарій --> Безпотоковий]. Ми двічі клацаємо піктограму сценарію, яка з’явилася поруч із псевдонімом *bubbleRob* в ієрархії сцени, копіюємо та вставляємо наступний код у [редактор сценаріїв](https://www.coppeliarobotics.com/helpFiles/en/scriptEditor.htm), а потім закриваємо його:
 
```function speedChange_callback(ui,id,newVal)
    speed=minMaxSpeed[1]+(minMaxSpeed[2]-minMaxSpeed[1])*newVal/100
end

function sysCall_init()
    -- This is executed exactly once, the first time this script is executed
    bubbleRobBase=sim.getObject('.') -- this is bubbleRob's handle
    leftMotor=sim.getObject("./leftMotor") -- Handle of the left motor
    rightMotor=sim.getObject("./rightMotor") -- Handle of the right motor
    noseSensor=sim.getObject("./sensingNose") -- Handle of the proximity sensor
    minMaxSpeed={50*math.pi/180,300*math.pi/180} -- Min and max speeds for each motor
    backUntilTime=-1 -- Tells whether bubbleRob is in forward or backward mode
    robotCollection=sim.createCollection(0)
    sim.addItemToCollection(robotCollection,sim.handle_tree,bubbleRobBase,0)
    distanceSegment=sim.addDrawingObject(sim.drawing_lines,4,0,-1,1,{0,1,0})
    robotTrace=sim.addDrawingObject(sim.drawing_linestrip+sim.drawing_cyclic,2,0,-1,200,{1,1,0})
    graph=sim.getObject('./graph')
    distStream=sim.addGraphStream(graph,'bubbleRob clearance','m',0,{1,0,0})
    -- Create the custom UI:
        xml = '<ui title="'..sim.getObjectAlias(bubbleRobBase,1)..' speed" closeable="false" resizeable="false" activate="false">'..[[
        <hslider minimum="0" maximum="100" onchange="speedChange_callback" id="1"/>
        <label text="" style="* {margin-left: 300px;}"/>
        </ui>
        ]]
    ui=simUI.create(xml)
    speed=(minMaxSpeed[1]+minMaxSpeed[2])*0.5
    simUI.setSliderValue(ui,1,100*(speed-minMaxSpeed[1])/(minMaxSpeed[2]-minMaxSpeed[1]))
end

function sysCall_sensing()
    local result,distData=sim.checkDistance(robotCollection,sim.handle_all)
    if result>0 then
        sim.addDrawingObjectItem(distanceSegment,nil)
        sim.addDrawingObjectItem(distanceSegment,distData)
        sim.setGraphStreamValue(graph,distStream,distData[7])
    end
    local p=sim.getObjectPosition(bubbleRobBase,-1)
    sim.addDrawingObjectItem(robotTrace,p)
end

function sysCall_actuation()
    result=sim.readProximitySensor(noseSensor) -- Read the proximity sensor
    -- If we detected something, we set the backward mode:
    if (result>0) then backUntilTime=sim.getSimulationTime()+4 end 

    if (backUntilTime<sim.getSimulationTime()) then
        -- When in forward mode, we simply move forward at the desired speed
        sim.setJointTargetVelocity(leftMotor,speed)
        sim.setJointTargetVelocity(rightMotor,speed)
    else
        -- When in backward mode, we simply backup in a curve at reduced speed
        sim.setJointTargetVelocity(leftMotor,-speed/2)
        sim.setJointTargetVelocity(rightMotor,-speed/8)
    end
end

function sysCall_cleanup()
    simUI.destroy(ui)
end
```                                            
Ми запускаємо симуляцію. *BubbleRob* тепер рухається вперед, намагаючись уникати перешкод (дуже просто). Поки симуляція все ще працює, змініть швидкість *BubbleRob* і скопіюйте/вставте її кілька разів. Також спробуйте масштабувати кілька з них, поки симуляція все ще працює. Майте на увазі, що функція обчислення мінімальної відстані може сильно сповільнювати симуляцію залежно від середовища.
                                               
Використання сценарію для керування роботом або моделлю – це лише один із способів. CoppeliaSim пропонує багато різних способів (також комбінованих), подивіться [навчальний посібник зовнішнього контролера](https://www.coppeliarobotics.com/helpFiles/en/externalControllerTutorial.htm).                    
