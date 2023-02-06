### Підручник ROS 2 ###

Цей посібник спробує простим способом пояснити, як можна ввімкнути CoppeliaSim ROS 2 на основі ROS 2 Foxy.
Перш за все, ви повинні переконатися, що ви пройшли офіційні навчальні посібники ROS 2, принаймні розділ для початківців. Тоді ми припускаємо, що у вас запущено найновішу версію Ubuntu, що встановлено ROS і налаштовано папки робочої області. Тут також зверніться до офіційної документації щодо встановлення ROS 2.

Загальні функції ROS 2 у CoppeliaSim підтримуються через інтерфейс ROS 2 (libsimExtROS2.so). Дистрибутив Linux має включати цей файл, уже скомпільований у CoppeliaSim/compiledROSPlugins, але спочатку його потрібно скопіювати в CoppeliaSim/, інакше він не завантажиться. Однак у вас можуть виникнути проблеми із завантаженням плагіна, залежно від особливостей вашої системи: переконайтеся, що завжди перевіряєте вікно терміналу CoppeliaSim, щоб дізнатися більше про операції завантаження плагіна. Плагіни завантажуються під час запуску CoppeliaSim. Також переконайтеся, що джерело середовища ROS 2 перед запуском CoppeliaSim.

Якщо плагін не може бути завантажений, вам слід перекомпілювати його самостійно. Він має відкритий вихідний код і може бути змінений стільки, скільки потрібно, щоб підтримувати певну функцію або розширювати її функціональність. Якщо конкретні повідомлення/послуги тощо. потребує підтримки, обов’язково відредагуйте файли, розташовані в simExtROS2/meta/, перед перекомпіляцією. Є 2 пакети:

- simExtROS2: цей пакунок є інтерфейсом ROS 2, який буде скомпільовано у файл ".so", який використовується CoppeliaSim.
- ros2_bubble_rob: це пакет дуже простого контролера робота, який підключається до CoppeliaSim через інтерфейс ROS 2. Цей вузол буде відповідати за керування червоним роботом у демонстраційній сцені controlTypeExamples/controlledViaRos.ttt

Вищезазначені пакети слід скопіювати у вашу папку ros2_ws/src.

Щоб створити пакети, перейдіть до папки ros2_ws і введіть:

```
$ export COPPELIASIM_ROOT_DIR=~/path/to/coppeliaSim/folder
$ ulimit -s unlimited #otherwise compilation might freeze/crash
$ colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
```

Це воно! Пакунки мають бути згенеровані та скомпільовані до бібліотеки, яка автоматично копіюється до папки встановлення CoppeliaSim. Тепер плагін готовий до використання.

Тепер відкрийте термінал, перейдіть до папки встановлення CoppeliaSim і запустіть CoppeliaSim. Це те, що ви повинні мати (або подібне):

```
$ ./coppeliaSim.sh
...
Plugin 'ROS2Interface': loading...
Plugin 'ROS2Interface': load succeeded.
...
```

Після успішного завантаження інтерфейсу ROS 2 перевірка доступних вузлів дає наступне:

```
$ ros2 node list
/sim_ros2_interface
```

У порожній сцені CoppeliaSim виберіть об’єкт, а потім прикріпіть до нього непотоковий дочірній сценарій за допомогою [Menu bar --> Add --> Associated child script --> Non threaded]. Відкрийте редактор сценаріїв для цього сценарію та замініть вміст таким:

```
function subscriber_callback(msg)
    -- This is the subscriber callback function
    sim.addLog(sim.verbosity_scriptinfos,'subscriber receiver following Float32: '..msg.data)
end

function getTransformStamped(objHandle,name,relTo,relToName)
    -- This function retrieves the stamped transform for a specific object
    t=simROS2.getSystemTime()
    p=sim.getObjectPosition(objHandle,relTo)
    o=sim.getObjectQuaternion(objHandle,relTo)
    return {
        header={
            stamp=t,
            frame_id=relToName
        },
        child_frame_id=name,
        transform={
            translation={x=p[1],y=p[2],z=p[3]},
            rotation={x=o[1],y=o[2],z=o[3],w=o[4]}
        }
    }
end

function sysCall_init()
    -- The child script initialization
    objectHandle=sim.getObject('.')
    objectAlias=sim.getObjectAlias(objectHandle,3)

    -- Prepare the float32 publisher and subscriber (we subscribe to the topic we publish):
    if simROS2 then
        publisher=simROS2.createPublisher('/simulationTime','std_msgs/msg/Float32')
        subscriber=simROS2.createSubscription('/simulationTime','std_msgs/msg/Float32','subscriber_callback')
    end
end

function sysCall_actuation()
    -- Send an updated simulation time message, and send the transform of the object attached to this script:
    if simROS2 then
        simROS2.publish(publisher,{data=sim.getSimulationTime()})
        simROS2.sendTransform(getTransformStamped(objectHandle,objectAlias,-1,'world'))
        -- To send several transforms at once, use simROS2.sendTransforms instead
    end
end

function sysCall_cleanup()
    -- Following not really needed in a simulation script (i.e. automatically shut down at simulation end):
    if simROS2 then
        simROS.shutdownPublisher(publisher)
        simROS.shutdownSubscriber(subscriber)
    end
end
```
Наведений вище сценарій опублікує час симуляції та одночасно підпишеться на нього. Він також опублікує перетворення об’єкта, до якого приєднано сценарій. Ви повинні мати змогу побачити тему часу моделювання за допомогою:

```
$ ros2 topic list
```

Щоб переглянути вміст повідомлення, ви можете ввести:

```
$ ros2 topic echo /simulationTime
```

Тепер завантажте демонстраційну сцену ros2InterfaceTopicPublisherAndSubscriber.ttt і запустіть симуляцію. Код у дочірньому сценарії, прикріпленому до Vision_sensor, дозволить видавцю передавати потокове зображення датчика зору, а також дозволить передплатнику слухати той самий потік. Абонент передає зчитані дані на датчик пасивного зору, який використовується лише як контейнер даних. Тож CoppeliaSim транслює дані, прослуховуючи ті самі дані! Ось що відбувається:

![image](https://user-images.githubusercontent.com/121936602/217104011-d9e7607d-8131-4831-872d-d4c6b6979c8b.png)

[Демонстрація видавця зображень і передплатників]

Спробуйте трохи поекспериментувати з кодом. Ви також можете візуалізувати зображення, яке CoppeliaSim транслює, за допомогою такої команди:

```
$ ros2 run image_tools showimage --ros-args --remap image:=/visionSensorData
```

Якби ви передавали простіші дані, ви також могли б візуалізувати їх за допомогою:

```
$ ros2 topic echo /visionSensorData
```

Тепер зупиніть симуляцію та завантажте демонстраційну сцену controlTypeExamples/controlledViaRos2.ttt і запустіть симуляцію. Робот є спрощеним, а також поводиться спрощено для цілей спрощення. Управління здійснюється через інтерфейс ROS 2:

![image](https://user-images.githubusercontent.com/121936602/217104210-cd86cb9f-aa65-473b-9344-5724e3f3708d.png)

[Зовнішня клієнтська програма, яка керує роботом через ROS]

Дочірній сценарій, приєднаний до робота, який працює безпотоковим способом, відповідає за наступне:

- визначити ручки деяких об’єктів (наприклад, ручки з’єднання двигуна та ручка датчика наближення)
- перевірте, чи завантажено інтерфейс ROS 2
- Запустіть абонентів швидкості двигуна
- Запустіть видавця датчиків і видавця симуляції часу
- і, нарешті, запустіть клієнтську програму. Програма викликається з деякими назвами тем як аргументами, щоб вона знала, які теми слухати та підписатися. Потім клієнтська програма (ros2BubbleRob) бере на себе керування роботом через ROS 2.

Під час моделювання скопіюйте та вставте кілька разів робота. Зауважте, що кожна копія є безпосередньо робочою та незалежною. Це одна з багатьох сильних сторін CoppeliaSim.

Тепер зупиніть симуляцію та відкрийте нову сцену, а потім перетягніть туди таку модель: Models/tools/ros2Interface helper tool.ttm. Ця модель складається з єдиного сценарію налаштування, який пропонує видавцям і передплатникам наступних тем:
- Тема startSimulation: можна використовувати для запуску симуляції шляхом публікації в цій темі повідомлення std_msgs/msg/Bool.
- Тема pauseSimulation: можна використовувати для призупинення симуляції шляхом публікації в цій темі повідомлення std_msgs/msg/Bool.
- Тема stopSimulation: можна використовувати для зупинки симуляції шляхом публікації в цій темі повідомлення std_msgs/msg/Bool.
- Тема enableSyncMode: опублікувавши повідомлення std_msgs/msg/Bool у цій темі, ви можете ввімкнути/вимкнути ступінчастий режим.
- Тема triggerNextStep: опублікувавши повідомлення std_msgs/msg/Bool у цій темі, ви можете запустити наступний крок симуляції, перебуваючи в покроковому режимі.
- Тема simulationStepDone: повідомлення типу std_msgs/msg/Bool буде опубліковано в кінці кожного проходу симуляції.
- Тема simulationState: повідомлення типу std_msgs/msg/Int32 публікуватимуться регулярно. 0 вказує на те, що моделювання зупинено, 1 — що воно працює, а 2 — що воно призупинено.
- Тема simulationTime: повідомлення типу std_msgs/msg/Float32 будуть публікуватися на регулярній основі із зазначенням поточного часу симуляції.

Подивіться на вміст сценарію налаштування, який можна повністю налаштувати для різних цілей. Спробуйте створити повідомлення теми з командного рядка, наприклад:

```
$ ros2 topic pub /startSimulation std_msgs/msg/Bool '{data: true}' --once
$ ros2 topic pub /pauseSimulation std_msgs/msg/Bool '{data: true}' --once
$ ros2 topic pub /stopSimulation std_msgs/msg/Bool '{data: true}' --once
$ ros2 topic pub /enableSyncMode std_msgs/msg/Bool '{data: true}' --once
$ ros2 topic pub /startSimulation std_msgs/msg/Bool '{data: true}' --once
$ ros2 topic pub /triggerNextStep std_msgs/msg/Bool '{data: true}' --once
$ ros2 topic pub /triggerNextStep std_msgs/msg/Bool '{data: true}' --once
$ ros2 topic pub /triggerNextStep std_msgs/msg/Bool '{data: true}' --once
$ ros2 topic pub /stopSimulation std_msgs/msg/Bool '{data: true}' --once
```

Щоб відобразити поточний час симуляції, ви можете ввести:

```
$ ros2 topic echo /simulationTime
```

Нарешті, обов’язково подивіться на функціональність віддаленого API у CoppeliaSim: вона дозволяє дистанційно виконувати функції, швидку передачу даних туди й назад, досить проста у використанні, легка та кросплатформна. Функція віддаленого API доступна для 7 різних мов і в деяких випадках може бути цікавою альтернативою ROS.
