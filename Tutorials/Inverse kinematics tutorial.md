# Підручник з зворотної кінематики #

Цей підручник пояснює, як [використовувати кінематику](https://www.coppeliarobotics.com/helpFiles/en/kinematics.htm) CoppeliaSim під час створення надлишкового маніпулятора із 7 DoF. Але перед цим обов’язково подивіться різні приклади сцен, пов’язаних з IK та FK, у папці scenes/kinematics.

Цей підручник розділений на 3 частини:
- [Побудова](https://www.coppeliarobotics.com/helpFiles/en/inverseKinematicsTutorial.htm#model) простої імітаційної моделі резервного маніпулятора
- [Постановка](https://www.coppeliarobotics.com/helpFiles/en/inverseKinematicsTutorial.htm#setup) оберненої кінематичної задачі
- [Тестування](https://www.coppeliarobotics.com/helpFiles/en/inverseKinematicsTutorial.htm#run) зворотної кінематик

## Проста імітаційна модель ##
Для цього підручника ми створимо нединамічний маніпулятор, який просто використовує інверсну кінематику без використання будь-яких функцій фізичного механізму. Дані CoppeliaSim CAD, пов’язані з цим підручником (redundantManipulator.stl), знаходяться в папці встановлення CoppeliaSim cadFiles. Сцену CoppeliaSim, пов’язану з цим посібником, можна знайти в scenes/tutorials/InverseKinematics. Натисніть [Menu bar --> File --> Import --> Mesh...], а потім виберіть файл для імпорту. Також зверніться до розділу про те, як [імпортувати/експортувати фігури](https://www.coppeliarobotics.com/helpFiles/en/importExport.htm). Відкриється діалогове вікно з різними параметрами імпорту. Натисніть Імпорт. Імпортовано одну [просту форму](https://www.coppeliarobotics.com/helpFiles/en/shapes.htm), яка розташована посередині сцени. Фігура також з’являється в [ієрархії сцен](https://www.coppeliarobotics.com/helpFiles/en/userInterface.htm#SceneHierarchy) у лівій частині головного вікна. Залежно від того, як було експортовано оригінальні дані САПР, імпортовані дані САПР можуть мати інший масштаб, інше розташування або навіть розділені на кілька форм. На наступному малюнку показано імпортовану форму:

![image](https://user-images.githubusercontent.com/121936602/217073442-d63b4b51-4b19-40e6-bda3-44e230b442d4.png)

Як ви можете бачити, операція імпорту залишила нам одну форму, де ми очікували кілька форм. Це означає, що нам доведеться самостійно розділити об’єкт маніпулятора: виберіть об’єкт (просто клацніть на ньому в сцені або в ієрархії сцени), а потім натисніть [Menu bar --> Edit --> Grouping/Merging --> Divide selected shapes]. Ось що ви повинні мати

![image](https://user-images.githubusercontent.com/121936602/217073596-bec4593d-21cb-453e-ba01-847e02513d82.png)

Оригінальна форма була розділена на кілька підформ (див. також ієрархію сцени). Алгоритм поділу фігур працює шляхом групування всіх трикутників, які з’єднані спільними ребрами. Залежно від того, як було створено або експортовано оригінальну сітку, така процедура поділу не може бути виконана. У такому випадку вам доведеться вручну видобувати фігури в [режимі редагування трикутника](https://www.coppeliarobotics.com/helpFiles/en/triangleEditMode.htm) або у зовнішньому редакторі.

Далі ми змінимо кольори різних об’єктів, щоб мати гарний візуальний вигляд. Спочатку двічі клацніть значок фігури в ієрархії сцени. Відкриється [діалогове вікно властивостей форми](https://www.coppeliarobotics.com/helpFiles/en/shapeProperties.htm). Поки фігуру вибрано, клацніть «Налаштувати колір» у діалоговому вікні: це дозволить вам налаштувати різні компоненти кольору вибраної фігури. Наразі просто відрегулюйте компонент навколишнього/розсіяного кольору ваших фігур. Щоб перенести колір однієї фігури в іншу, виберіть обидві фігури та переконайтеся, що остання вибрана фігура (позначена білою рамкою) є тією, з якої ви хочете взяти колір, а потім просто натисніть кнопку Застосувати до виділення у вікні Розділ кольорів діалогового вікна форми. Коли ви закінчите розфарбовувати, у вас може виникнути така ситуація:

![image](https://user-images.githubusercontent.com/121936602/217073712-167660e0-efc2-419e-9548-a47c7a44f997.png)

На наступному кроці ми додамо 7 [суглобів](https://www.coppeliarobotics.com/helpFiles/en/joints.htm) маніпулятора. Один із способів зробити це — додати з’єднання до сцени, а потім вказати їх відповідне положення та орієнтацію (через [діалогове вікно позиції](https://www.coppeliarobotics.com/helpFiles/en/positionDialog.htm) та [діалогове вікно орієнтації](https://www.coppeliarobotics.com/helpFiles/en/positionDialog.htm)). Однак це неможливо, якщо ви не знаєте точного лінійного/кутового положення з’єднань, як у нашому випадку, і тому нам доведеться витягти їх із форм, які у нас є:

Виберіть усі імпортовані форми та клацніть [Menu bar --> Edit --> Reorient bounding box --> with reference frame of world]. Ця операція гарантує, що наші [обмежувальні прямокутники](https://www.coppeliarobotics.com/helpFiles/en/shapeReferenceFrames.htm) вирівняні з абсолютною системою відліку, і, враховуючи поточну конфігурацію маніпулятора, представляють найменші обмежувальні рамки. Клацніть [Menu bar --> Add --> Joint --> Revolute], щоб вставити поворотне з’єднання в сцену. Положення за замовчуванням — (0;0;0), а орієнтація за замовчуванням — вертикальна, тому з’єднання приховано базовим циліндром маніпулятора. Поки з’єднання все ще вибрано, виберіть базовий циліндр, утримуючи ctrl, потім відкрийте [діалогове вікно позиції](https://www.coppeliarobotics.com/helpFiles/en/shapeReferenceFrames.htm) на вкладці позиції та натисніть «Застосувати до виділення». Це просто розмістило з’єднання в тих самих координатах, що й базовий циліндр (однак ця операція лише трохи відкоригувала вертикальне положення з’єднання, оскільки воно вже було майже на місці). Тепер повторіть процедуру для всіх інших суглобів маніпулятора (пам’ятайте, що їх має бути 7). Зараз усі суглоби на місці, однак деякі з них неправильно орієнтовані.Виберіть усі з’єднання, які мають бути вирівняні зі світовою віссю Y, потім введіть (90,0,0) для елементів «Альфа», «Бета» та «Гама» в [діалоговому вікні орієнтації](https://www.coppeliarobotics.com/helpFiles/en/orientationDialog.htm) на вкладці «Орієнтація», а потім натисніть кнопку «Застосувати до вибору». Далі виберіть з’єднання, яке має бути вирівняно з віссю X світу, а потім введіть (0,90,0) для Альфа, Бета та Гама. Тепер усі суглоби мають правильне положення та орієнтацію.

Тепер ви можете налаштувати розміри з’єднання (перевірте елементи «Довжина» та «Діаметр») у [діалоговому вікні властивостей з’єднання](https://www.coppeliarobotics.com/helpFiles/en/jointProperties.htm) (яке можна відкрити, двічі клацнувши піктограму з’єднання в ієрархії сцени). Переконайтеся, що всі стики добре видно. Ось що ви повинні отримати: 

![image](https://user-images.githubusercontent.com/121936602/217073953-44cf3f6e-29cf-4b07-985b-04465d1f7baa.png)

Наступним кроком у цьому підручнику є групування фігур, які належать одній твердій сутності. Виберіть 5 фігур, які є частиною зв’язку 1 (основний циліндр є зв’язком 0), а потім натисніть [Menu bar --> Edit --> Grouping/Merging --> Group selected shapes]. Коли фігури згруповано в складену фігуру, ви можете повторно вирівняти її обмежувальну рамку зі світом, але цей крок не є обов’язковим (і має лише візуальний ефект). Повторіть ту саму процедуру з усіма фігурами, які логічно належать одна до одної. У цьому уроці ми не будемо активувати пальці захвату, а просто жорстко згрупуємо їх з останньою ланкою. Якщо всі фігури, призначені для групування, мають однакові візуальні атрибути, натомість спробуйте об’єднати їх разом: [Menu bar --> Edit --> Grouping/Merging --> Merge selected shapes].

На цьому етапі ви можете перейменувати всі об’єкти сцени таким чином, переходячи від основи до підказки: redundantRobot — redundantRob_joint1 — redundantRob_link1 — redundantRob_joint2 тощо. Просто двічі клацніть псевдонім об’єкта в ієрархії сцени, щоб відредагувати його.

Тепер ми можемо побудувати кінематичний ланцюг, йдучи від вершини до основи: виберіть об’єкт redundantRob_link7, потім ctrl виберіть об’єкт redundantRob_joint7 і натисніть [Menu bar --> Edit --> Make last selected object parent]. Крім того, ви можете перетягнути об’єкт на інший в ієрархії сцени, щоб виконати подібну операцію. Далі виконайте те саме для об’єктів redundantRob_joint7 і object redundantRob_link6. Продовжуйте так само, поки не буде побудований весь кінематичний ланцюг маніпулятора. Ось що ви повинні мати (зверніть увагу на структуру ієрархії сцени):

![image](https://user-images.githubusercontent.com/121936602/217074167-098044f9-d3e4-4006-b44c-64b5d96732e0.png)

Виберіть усі з’єднання, потім у діалоговому вікні з’єднань виберіть кінематичний режим і натисніть «Застосувати до вибору». Залиште вибраними з’єднання, потім відкрийте [загальні властивості об’єкта](https://www.coppeliarobotics.com/helpFiles/en/commonPropertiesDialog.htm) та в розділі «Шари видимості» вимкніть шар 2 і ввімкніть шар 10, а потім натисніть відповідну кнопку «Застосувати до вибору». Це просто відправило всі з’єднання на шар видимості 10, фактично зробивши їх невидимими. Перегляньте [діалогове вікно вибору шару](https://www.coppeliarobotics.com/helpFiles/en/layerSelectionDialog.htm), якщо ви бажаєте тимчасово ввімкнути/вимкнути деякі шари.

У CoppeliaSim завдання IK вимагає визначення принаймні таких елементів:
- кінематичний ланцюг, описаний макетом наконечника та базовим об’єктом.
- цільовий манекен, за яким манекен підказки буде змушений слідувати.

У нас вже є базовий об’єкт (об’єкт redundantRobot). Давайте додамо [фіктивний об’єкт](https://www.coppeliarobotics.com/helpFiles/en/dummies.htm), перейменуємо його на redundantRob_tip і встановимо його позицію (0.324,0,0.62) за допомогою діалогового вікна координат і перетворення. Далі прикріпіть манекен до redundantRob_link7 (виберіть redundantRob_tip, потім redundantRob_link7, потім [Menu bar --> Edit --> Make last selected object parent]. Манекен підказки готовий!

Тепер давайте підготуємо цільовий манекен: скопіюйте та вставте redundantRob_tip і перейменуйте копію на redundantRob_target. Манекен мішені готовий.

Тепер ми додамо спосіб легкого керування роботом, не турбуючись про те, що його можна зламати, пересуваючи неправильні об’єкти. Тому ми визначимо його як [модель](https://www.coppeliarobotics.com/helpFiles/en/models.htm). По-перше, перемістіть redundantRob_tip і redundantRob_target на рівень 11, щоб зробити обидва манекени невидимими. Потім виберіть усі видимі об’єкти в поданні сцени, утримуючи клавішу Shift, клацніть об’єкт redundantRobot в ієрархії сцени, утримуючи клавішу Ctrl, щоб видалити його з виділення, а потім відкрийте діалогове вікно [загальних властивостей об’єкта](https://www.coppeliarobotics.com/helpFiles/en/commonPropertiesDialog.htm). Перевірте пункт Вибрати базу моделі натомість, а потім відповідну кнопку Застосувати до вибору. Зніміть вибір за допомогою <ESC>, а потім виберіть надлишкового робота. У цьому ж діалоговому вікні встановіть прапорець біля пункту «Об’єкт — базова модель», а потім закрийте діалогові вікна. Зверніть увагу, як пунктирна обмежувальна рамка тепер охоплює весь маніпулятор:

![image](https://user-images.githubusercontent.com/121936602/217074464-8aab2915-a18e-4908-b375-7933b22efb2f.png)

Клацніть будь-який об’єкт на маніпуляторі та зверніть увагу, як натомість завжди вибирається базовий об’єкт redundantRobot.

Далі ми додаємо сферу маніпулювання, яку будемо використовувати для керування положенням/орієнтацією захоплення робота. Клацніть [Menu bar --> Add --> Primitive shape --> Sphere], щоб відкрити [діалогове вікно примітивної форми](https://www.coppeliarobotics.com/helpFiles/en/primitiveShapes.htm), вкажіть 0,05 для розміру X, Y-розміру та Z-розміру, а потім зніміть прапорець біля пункту «Створити динамічну форму, що відповідає вимогам». і натисніть OK. Відкоригуйте положення щойно доданої сфери так, щоб воно було таким самим, як і redundantRob_target (за допомогою діалогового вікна координат і перетворення). Тепер сфера з’являється на кінчику маніпулятора. Перейменуйте сферу на redundantRob_manipSphere, а потім зробіть її батьківською для redundantRob_target. Зробити надлишковимRobot батьком rendundantRob_manipSphere: цільовий манекен і сфера маніпуляції тепер також є частиною моделі робота. Згорніть надлишкове дерево Robot в ієрархії сцени. Резервна модель маніпулятора готова!

## Постановка оберненої кінематичної задачі ##

Інверсна кінематика повністю налаштована за допомогою сценарію, що викликає відповідні команди API: ідея полягає в тому, щоб побудувати еквівалентну кінематичну модель за допомогою функцій, наданих [плагіном кінематики](https://www.coppeliarobotics.com/helpFiles/en/kinematicsPlugin.htm). Підхід використовує поняття та термінологію [груп IK та елементів IK](https://www.coppeliarobotics.com/helpFiles/en/basicsOnIkGroupsAndIkElements.htm).
Виберіть об’єкт redundantRobot, а потім [Menu bar --> Add --> Associated child script --> Non-threaded], щоб приєднати [дочірній сценарій](https://www.coppeliarobotics.com/helpFiles/en/childScripts.htm) до цього об’єкта. Двічі клацніть значок сценарію, який з’явився поруч із псевдонімом об’єкта, і замініть вміст сценарію таким кодом:
```function sysCall_init()
    -- Build a kinematic chain and 2 IK groups (undamped and damped) inside of the IK plugin environment,
    -- based on the kinematics of the robot in the scene:
    -- There is a simple way, and a more elaborate way (but which gives you more options/flexibility):

    -- Simple way:
    local simBase=sim.getObject('.')
    local simTip=sim.getObject('./redundantRob_tip')
    local simTarget=sim.getObject('./redundantRob_target')
    -- create an IK environment:
    ikEnv=simIK.createEnvironment()
    -- create an IK group: 
    ikGroup_undamped=simIK.createIkGroup(ikEnv)
    -- set its resolution method to undamped: 
    simIK.setIkGroupCalculation(ikEnv,ikGroup_undamped,simIK.method_pseudo_inverse,0,6)
    -- create an IK element based on the scene content: 
    simIK.addIkElementFromScene(ikEnv,ikGroup_undamped,simBase,simTip,simTarget,simIK.constraint_pose)
    -- create another IK group: 
    ikGroup_damped=simIK.createIkGroup(ikEnv)
    -- set its resolution method to damped: 
    simIK.setIkGroupCalculation(ikEnv,ikGroup_damped,simIK.method_damped_least_squares,1,99)
    -- create an IK element based on the scene content: 
    simIK.addIkElementFromScene(ikEnv,ikGroup_damped,simBase,simTip,simTarget,simIK.constraint_pose) 
    
    -- Elaborate way:
    --[[
    simBase=sim.getObject('.')
    simTip=sim.getObject('./redundantRob_tip')
    simTarget=sim.getObject('./redundantRob_target')
    simJoints={}
    for i=1,7,1 do
        simJoints[i]=sim.getObject('./redundantRob_joint'..i)
    end
    ikJoints={}
    -- create an IK environment:
    ikEnv=simIK.createEnvironment()
    -- create a dummy in the IK environment: 
    ikBase=simIK.createDummy(ikEnv)
    -- set that dummy into the same pose as its CoppeliaSim counterpart: 
    simIK.setObjectMatrix(ikEnv,ikBase,-1,sim.getObjectMatrix(simBase,-1)) 
    local parent=ikBase
    for i=1,#simJoints,1 do -- loop through all joints
        -- create a joint in the IK environment:
        ikJoints[i]=simIK.createJoint(ikEnv,simIK.jointtype_revolute)
        -- set it into IK mode: 
        simIK.setJointMode(ikEnv,ikJoints[i],simIK.jointmode_ik)
        -- set the same joint limits as its CoppeliaSim counterpart joint: 
        local cyclic,interv=sim.getJointInterval(simJoints[i])
        simIK.setJointInterval(ikEnv,ikJoints[i],cyclic,interv)
        -- set the same joint lin./ang. position as its CoppeliaSim counterpart joint: 
        simIK.setJointPosition(ikEnv,ikJoints[i],sim.getJointPosition(simJoints[i]))
        -- set the same object pose as its CoppeliaSim counterpart joint: 
        simIK.setObjectMatrix(ikEnv,ikJoints[i],-1,sim.getObjectMatrix(simJoints[i],-1))
        -- set its corresponding parent: 
        simIK.setObjectParent(ikEnv,ikJoints[i],parent,true) 
        parent=ikJoints[i]
    end
    -- create the tip dummy in the IK environment:
    ikTip=simIK.createDummy(ikEnv)
    -- set that dummy into the same pose as its CoppeliaSim counterpart: 
    simIK.setObjectMatrix(ikEnv,ikTip,-1,sim.getObjectMatrix(simTip,-1))
    -- attach it to the kinematic chain: 
    simIK.setObjectParent(ikEnv,ikTip,parent,true)
    -- create the target dummy in the IK environment: 
    ikTarget=simIK.createDummy(ikEnv)
    -- set that dummy into the same pose as its CoppeliaSim counterpart: 
    simIK.setObjectMatrix(ikEnv,ikTarget,-1,sim.getObjectMatrix(simTarget,-1))
    -- link the two dummies: 
    simIK.setLinkedDummy(ikEnv,ikTip,ikTarget)
    -- create an IK group: 
    ikGroup_undamped=simIK.createIkGroup(ikEnv)
    -- set its resolution method to undamped: 
    simIK.setIkGroupCalculation(ikEnv,ikGroup_undamped,simIK.method_pseudo_inverse,0,6)
    -- make sure the robot doesn't shake if the target position/orientation wasn't reached: 
    simIK.setIkGroupFlags(ikEnv,ikGroup_undamped,1+2+4+8)
    -- add an IK element to that IK group: 
    local ikElementHandle=simIK.addIkElement(ikEnv,ikGroup_undamped,ikTip)
    -- specify the base of that IK element: 
    simIK.setIkElementBase(ikEnv,ikGroup_undamped,ikElementHandle,ikBase)
    -- specify the constraints of that IK element: 
    simIK.setIkElementConstraints(ikEnv,ikGroup_undamped,ikElementHandle,simIK.constraint_pose)
    -- create another IK group: 
    ikGroup_damped=simIK.createIkGroup(ikEnv)
    -- set its resolution method to damped: 
    simIK.setIkGroupCalculation(ikEnv,ikGroup_damped,simIK.method_damped_least_squares,1,99)
    -- add an IK element to that IK group: 
    local ikElementHandle=simIK.addIkElement(ikEnv,ikGroup_damped,ikTip)
    -- specify the base of that IK element: 
    simIK.setIkElementBase(ikEnv,ikGroup_damped,ikElementHandle,ikBase)
    -- specify the constraints of that IK element: 
    simIK.setIkElementConstraints(ikEnv,ikGroup_damped,ikElementHandle,simIK.constraint_pose) 
    --]]
end

function sysCall_actuation()
    -- There is a simple way, and a more elaborate way (but which gives you more options/flexibility):
    
    -- Simple way:
    -- try to solve with the undamped method:
    if simIK.applyIkEnvironmentToScene(ikEnv,ikGroup_undamped,true)==simIK.result_fail then 
        -- the position/orientation could not be reached.
        -- try to solve with the damped method:
        simIK.applyIkEnvironmentToScene(ikEnv,ikGroup_damped)
        -- We display a IK failure report message:
        sim.addLog(sim.verbosity_scriptwarnings,"IK solver failed.") 
    end
    
    -- Elaborate way:
    --[[
    -- reflect the pose of the target dummy to its counterpart in the IK environment:
    simIK.setObjectMatrix(ikEnv,ikTarget,ikBase,sim.getObjectMatrix(simTarget,simBase)) 
    -- try to solve with the undamped method:
    if simIK.handleIkGroup(ikEnv,ikGroup_undamped)==simIK.result_fail then 
        -- the position/orientation could not be reached.
        -- try to solve with the damped method:
        simIK.handleIkGroup(ikEnv,ikGroup_damped)
        -- We display a IK failure report message: 
        sim.addLog(sim.verbosity_scriptwarnings,"IK solver failed.") 
    end
    -- apply the joint values computed in the IK environment to their CoppeliaSim joint counterparts:
    for i=1,#simJoints,1 do
        sim.setJointPosition(simJoints[i],simIK.getJointPosition(ikEnv,ikJoints[i])) 
    end
    --]]
end 

function sysCall_cleanup()
    -- erase the IK environment: 
    simIK.eraseEnvironment(ikEnv) 
end 
```
Наведений вище сценарій створює еквівалентну кінематичну модель із моделі CoppeliaSim, потім на кожному кроці симуляції зчитує позицію/орієнтацію цілі CoppeliaSim, застосовує її до цілі еквівалентної кінематичної моделі, запускає розв’язувач IK і, нарешті, зчитує кути з’єднання еквівалентної кінематичної моделі та застосовує їх до з’єднань моделі CoppeliaSim. Для обробки окремих конфігурацій і ситуацій, коли ціль недосяжна, ми спочатку намагаємося використовувати незатухаючий розв’язувач, а якщо це не вдається, ми повертаємося до демпфованого розв’язувача (з демпфованим розв’язувачем, коли демпфування велике, роздільна здатність стає більш стабільною але зближення повільніше).

## Запуск моделювання ##

Наша зворотна задача з кінематики готова! Давайте перевіримо це. Запустіть симуляцію, а потім виберіть зелену сферу маніпуляції. Далі виберіть кнопку панелі [інструментів перекладу об’єктів](https://www.coppeliarobotics.com/helpFiles/en/positionDialog.htm):

![image](https://user-images.githubusercontent.com/121936602/217078451-2a0efcd7-2011-4f1f-b1d0-52a7d596a6c6.png)

Тепер перетягніть об'єкт мишею: маніпулятор повинен слідувати. Також спробуйте [кнопку обертання об’єкта](https://www.coppeliarobotics.com/helpFiles/en/orientationDialog.htm) на панелі інструментів:

![image](https://user-images.githubusercontent.com/121936602/217078530-d56537e4-0071-4db1-8ffc-b510f99e9335.png)

Спробуйте також утримувати клавіші ctr або shift під час маніпуляції. Поверніться до кнопки панелі інструментів перекладу об’єкта та спробуйте перетягнути об’єкт якомога далі, і зверніть увагу на те, що обернена кінематична задача досить надійна завдяки демпфованій компоненті. Зупиніть симуляцію, потім вимкніть демпфовану групу IK і повторіть спробу. Спробуйте також вимкнути індивідуальні обмеження у відповідному елементі IK і зверніть увагу на те, як поводиться маніпулятор під час симуляції.
Запустіть симуляцію та кілька разів скопіюйте та вставте маніпулятор і перемістіть/обертайте копії, також змінюючи їхні конфігурації, перетягуючи їхні сфери маніпулювання. Зверніть увагу, що кожен екземпляр маніпулятора повністю функціональний щодо IK.

