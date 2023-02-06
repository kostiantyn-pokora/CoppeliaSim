# Підручник з зворотної кінематики #

Цей підручник пояснює, як використовувати кінематику CoppeliaSim під час створення надлишкового маніпулятора із 7 DoF. Але перед цим обов’язково подивіться різні приклади сцен, пов’язаних з IK та FK, у папці scenes/kinematics.

Цей підручник розділений на 3 частини:
- Побудова простої імітаційної моделі резервного маніпулятора
- Постановка оберненої кінематичної задачі
- Тестування зворотної кінематик

## Проста імітаційна модель ##
Для цього підручника ми створимо нединамічний маніпулятор, який просто використовує інверсну кінематику без використання будь-яких функцій фізичного механізму. Дані CoppeliaSim CAD, пов’язані з цим підручником (redundantManipulator.stl), знаходяться в папці встановлення CoppeliaSim cadFiles. Сцену CoppeliaSim, пов’язану з цим посібником, можна знайти в scenes/tutorials/InverseKinematics. Натисніть [Menu bar --> File --> Import --> Mesh...], а потім виберіть файл для імпорту. Також зверніться до розділу про те, як імпортувати/експортувати фігури. Відкриється діалогове вікно з різними параметрами імпорту. Натисніть Імпорт. Імпортовано одну просту форму, яка розташована посередині сцени. Фігура також з’являється в ієрархії сцен у лівій частині головного вікна. Залежно від того, як було експортовано оригінальні дані САПР, імпортовані дані САПР можуть мати інший масштаб, інше розташування або навіть розділені на кілька форм. На наступному малюнку показано імпортовану форму:

![image](https://user-images.githubusercontent.com/121936602/217073442-d63b4b51-4b19-40e6-bda3-44e230b442d4.png)

Як ви можете бачити, операція імпорту залишила нам одну форму, де ми очікували кілька форм. Це означає, що нам доведеться самостійно розділити об’єкт маніпулятора: виберіть об’єкт (просто клацніть на ньому в сцені або в ієрархії сцени), а потім натисніть [Menu bar --> Edit --> Grouping/Merging --> Divide selected shapes]. Ось що ви повинні мати

![image](https://user-images.githubusercontent.com/121936602/217073596-bec4593d-21cb-453e-ba01-847e02513d82.png)

Оригінальна форма була розділена на кілька підформ (див. також ієрархію сцени). Алгоритм поділу фігур працює шляхом групування всіх трикутників, які з’єднані спільними ребрами. Залежно від того, як було створено або експортовано оригінальну сітку, така процедура поділу не може бути виконана. У такому випадку вам доведеться вручну видобувати фігури в режимі редагування трикутника або у зовнішньому редакторі.

Далі ми змінимо кольори різних об’єктів, щоб мати гарний візуальний вигляд. Спочатку двічі клацніть значок фігури в ієрархії сцени. Відкриється діалогове вікно властивостей форми. Поки фігуру вибрано, клацніть «Налаштувати колір» у діалоговому вікні: це дозволить вам налаштувати різні компоненти кольору вибраної фігури. Наразі просто відрегулюйте компонент навколишнього/розсіяного кольору ваших фігур. Щоб перенести колір однієї фігури в іншу, виберіть обидві фігури та переконайтеся, що остання вибрана фігура (позначена білою рамкою) є тією, з якої ви хочете взяти колір, а потім просто натисніть кнопку Застосувати до виділення у вікні Розділ кольорів діалогового вікна форми. Коли ви закінчите розфарбовувати, у вас може виникнути така ситуація:

![image](https://user-images.githubusercontent.com/121936602/217073712-167660e0-efc2-419e-9548-a47c7a44f997.png)

На наступному кроці ми додамо 7 суглобів маніпулятора. Один із способів зробити це — додати з’єднання до сцени, а потім вказати їх відповідне положення та орієнтацію (через діалогове вікно позиції та діалогове вікно орієнтації). Однак це неможливо, якщо ви не знаєте точного лінійного/кутового положення з’єднань, як у нашому випадку, і тому нам доведеться витягти їх із форм, які у нас є:

Виберіть усі імпортовані форми та клацніть [Menu bar --> Edit --> Reorient bounding box --> with reference frame of world]. Ця операція гарантує, що наші обмежувальні прямокутники вирівняні з абсолютною системою відліку, і, враховуючи поточну конфігурацію маніпулятора, представляють найменші обмежувальні рамки. Клацніть [Menu bar --> Add --> Joint --> Revolute], щоб вставити поворотне з’єднання в сцену. Положення за замовчуванням — (0;0;0), а орієнтація за замовчуванням — вертикальна, тому з’єднання приховано базовим циліндром маніпулятора. Поки з’єднання все ще вибрано, виберіть базовий циліндр, утримуючи ctrl, потім відкрийте діалогове вікно позиції на вкладці позиції та натисніть «Застосувати до виділення». Це просто розмістило з’єднання в тих самих координатах, що й базовий циліндр (однак ця операція лише трохи відкоригувала вертикальне положення з’єднання, оскільки воно вже було майже на місці). Тепер повторіть процедуру для всіх інших суглобів маніпулятора (пам’ятайте, що їх має бути 7). Зараз усі суглоби на місці, однак деякі з них неправильно орієнтовані.Виберіть усі з’єднання, які мають бути вирівняні зі світовою віссю Y, потім введіть (90,0,0) для елементів «Альфа», «Бета» та «Гама» в діалоговому вікні орієнтації на вкладці «Орієнтація», а потім натисніть кнопку «Застосувати до вибору». Далі виберіть з’єднання, яке має бути вирівняно з віссю X світу, а потім введіть (0,90,0) для Альфа, Бета та Гама. Тепер усі суглоби мають правильне положення та орієнтацію.

Тепер ви можете налаштувати розміри з’єднання (перевірте елементи «Довжина» та «Діаметр») у діалоговому вікні властивостей з’єднання (яке можна відкрити, двічі клацнувши піктограму з’єднання в ієрархії сцени). Переконайтеся, що всі стики добре видно. Ось що ви повинні отримати: 

![image](https://user-images.githubusercontent.com/121936602/217073953-44cf3f6e-29cf-4b07-985b-04465d1f7baa.png)

Наступним кроком у цьому підручнику є групування фігур, які належать одній твердій сутності. Виберіть 5 фігур, які є частиною зв’язку 1 (основний циліндр є зв’язком 0), а потім натисніть [Menu bar --> Edit --> Grouping/Merging --> Group selected shapes]. Коли фігури згруповано в складену фігуру, ви можете повторно вирівняти її обмежувальну рамку зі світом, але цей крок не є обов’язковим (і має лише візуальний ефект). Повторіть ту саму процедуру з усіма фігурами, які логічно належать одна до одної. У цьому уроці ми не будемо активувати пальці захвату, а просто жорстко згрупуємо їх з останньою ланкою. Якщо всі фігури, призначені для групування, мають однакові візуальні атрибути, натомість спробуйте об’єднати їх разом: [Menu bar --> Edit --> Grouping/Merging --> Merge selected shapes].

На цьому етапі ви можете перейменувати всі об’єкти сцени таким чином, переходячи від основи до підказки: redundantRobot — redundantRob_joint1 — redundantRob_link1 — redundantRob_joint2 тощо. Просто двічі клацніть псевдонім об’єкта в ієрархії сцени, щоб відредагувати його.

Тепер ми можемо побудувати кінематичний ланцюг, йдучи від вершини до основи: виберіть об’єкт redundantRob_link7, потім ctrl виберіть об’єкт redundantRob_joint7 і натисніть [Menu bar --> Edit --> Make last selected object parent]. Крім того, ви можете перетягнути об’єкт на інший в ієрархії сцени, щоб виконати подібну операцію. Далі виконайте те саме для об’єктів redundantRob_joint7 і object redundantRob_link6. Продовжуйте так само, поки не буде побудований весь кінематичний ланцюг маніпулятора. Ось що ви повинні мати (зверніть увагу на структуру ієрархії сцени):

![image](https://user-images.githubusercontent.com/121936602/217074167-098044f9-d3e4-4006-b44c-64b5d96732e0.png)

Виберіть усі з’єднання, потім у діалоговому вікні з’єднань виберіть кінематичний режим і натисніть «Застосувати до вибору». Залиште вибраними з’єднання, потім відкрийте загальні властивості об’єкта та в розділі «Шари видимості» вимкніть шар 2 і ввімкніть шар 10, а потім натисніть відповідну кнопку «Застосувати до вибору». Це просто відправило всі з’єднання на шар видимості 10, фактично зробивши їх невидимими. Перегляньте діалогове вікно вибору шару, якщо ви бажаєте тимчасово ввімкнути/вимкнути деякі шари.

У CoppeliaSim завдання IK вимагає визначення принаймні таких елементів:
- кінематичний ланцюг, описаний макетом наконечника та базовим об’єктом.
- цільовий манекен, за яким манекен підказки буде змушений слідувати.

У нас вже є базовий об’єкт (об’єкт redundantRobot). Давайте додамо фіктивний об’єкт, перейменуємо його на redundantRob_tip і встановимо його позицію (0.324,0,0.62) за допомогою діалогового вікна координат і перетворення. Далі прикріпіть манекен до redundantRob_link7 (виберіть redundantRob_tip, потім redundantRob_link7, потім [Menu bar --> Edit --> Make last selected object parent]. Манекен підказки готовий!

Тепер давайте підготуємо цільовий манекен: скопіюйте та вставте redundantRob_tip і перейменуйте копію на redundantRob_target. Манекен мішені готовий.

Тепер ми додамо спосіб легкого керування роботом, не турбуючись про те, що його можна зламати, пересуваючи неправильні об’єкти. Тому ми визначимо його як модель. По-перше, перемістіть redundantRob_tip і redundantRob_target на рівень 11, щоб зробити обидва манекени невидимими. Потім виберіть усі видимі об’єкти в поданні сцени, утримуючи клавішу Shift, клацніть об’єкт redundantRobot в ієрархії сцени, утримуючи клавішу Ctrl, щоб видалити його з виділення, а потім відкрийте діалогове вікно загальних властивостей об’єкта. Перевірте пункт Вибрати базу моделі натомість, а потім відповідну кнопку Застосувати до вибору. Зніміть вибір за допомогою <ESC>, а потім виберіть надлишкового робота. У цьому ж діалоговому вікні встановіть прапорець біля пункту «Об’єкт — базова модель», а потім закрийте діалогові вікна. Зверніть увагу, як пунктирна обмежувальна рамка тепер охоплює весь маніпулятор:

![image](https://user-images.githubusercontent.com/121936602/217074464-8aab2915-a18e-4908-b375-7933b22efb2f.png)

Клацніть будь-який об’єкт на маніпуляторі та зверніть увагу, як натомість завжди вибирається базовий об’єкт redundantRobot.

Далі ми додаємо сферу маніпулювання, яку будемо використовувати для керування положенням/орієнтацією захоплення робота. Клацніть [Menu bar --> Add --> Primitive shape --> Sphere], щоб відкрити діалогове вікно примітивної форми, вкажіть 0,05 для розміру X, Y-розміру та Z-розміру, а потім зніміть прапорець біля пункту «Створити динамічну форму, що відповідає вимогам». і натисніть OK. Відкоригуйте положення щойно доданої сфери так, щоб воно було таким самим, як і redundantRob_target (за допомогою діалогового вікна координат і перетворення). Тепер сфера з’являється на кінчику маніпулятора. Перейменуйте сферу на redundantRob_manipSphere, а потім зробіть її батьківською для redundantRob_target. Зробити надлишковимRobot батьком rendundantRob_manipSphere: цільовий манекен і сфера маніпуляції тепер також є частиною моделі робота. Згорніть надлишкове дерево Robot в ієрархії сцени. Резервна модель маніпулятора готова!