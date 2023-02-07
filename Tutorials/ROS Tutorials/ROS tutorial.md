### Підручник ROS ###

Цей підручник спробує простим способом пояснити, як можна ввімкнути CoppeliaSim ROS на основі збірки [ROS Melodic](http://wiki.ros.org/melodic) і [Catkin](http://catkin-tools.readthedocs.io/en/latest/installing.html).

Перш за все, ви повинні переконатися, що ви пройшли офіційні [підручники ROS](http://www.ros.org/wiki/ROS/Tutorials), принаймні розділ для початківців, і що ви встановили [інструменти Catkin](http://catkin-tools.readthedocs.io/en/latest/installing.html). Тоді ми припускаємо, що у вас запущено найновішу версію Ubuntu, що встановлено ROS і налаштовано папки робочої області. Тут також зверніться до [офіційної документації щодо встановлення ROS](http://wiki.ros.org/melodic/Installation/Ubuntu).

Загальні функції ROS у CoppeliaSim підтримуються через [інтерфейс ROS](https://www.coppeliarobotics.com/helpFiles/en/rosInterf.htm) (libsimExtROS.so). Дистрибутив Linux має включати цей файл, уже скомпільований у CoppeliaSim/compiledROSPlugins, але спочатку його потрібно скопіювати в CoppeliaSim/, інакше він не завантажиться. Однак у вас можуть виникнути проблеми із завантаженням плагіна, залежно від особливостей вашої системи: переконайтеся, що завжди перевіряєте вікно терміналу CoppeliaSim, щоб дізнатися більше про операції завантаження плагіна. Плагіни завантажуються під час запуску CoppeliaSim. Плагін ROS успішно завантажиться та ініціалізується, лише якщо roscore запущено в цей час (roscore є головним ROS). Також перед запуском CoppeliaSim переконайтеся, що джерело середовища ROS.

Якщо плагін не може бути завантажений, вам слід перекомпілювати його самостійно. Він має відкритий вихідний код і може бути змінений стільки, скільки потрібно, щоб підтримувати певну функцію або розширювати її функціональність. Якщо конкретні повідомлення/послуги тощо. потребує підтримки, обов’язково відредагуйте файли, розташовані в simExtROS/meta/, перед перекомпіляцією.Є 2 пакети:

- [simExtROS](https://github.com/CoppeliaRobotics/simExtROS): цей пакунок є [інтерфейсом ROS](https://www.coppeliarobotics.com/helpFiles/en/rosInterf.htm), який буде скомпільовано у файл ".so", який використовується CoppeliaSim.
- [ros_bubble_rob](https://github.com/CoppeliaRobotics/ros_bubble_rob): це пакет дуже простого контролера робота, який підключається до CoppeliaSim через [інтерфейс ROS](https://www.coppeliarobotics.com/helpFiles/en/rosInterf.htm). Цей вузол буде відповідати за керування червоним роботом у демонстраційній сцені controlTypeExamples/controlledViaRos.ttt

Вищезазначені пакети слід скопіювати у вашу папку catkin_ws/src. Переконайтеся, що ROS знає про ці пакунки, тобто ви можете перейти до папок вище за допомогою:

```
$ roscd sim_ros_interface
$ roscd ros_bubble_rob
```

Щоб створити пакети, перейдіть до папки catkin_ws і введіть:
```
$ export COPPELIASIM_ROOT_DIR=~/path/to/coppeliaSim/folder
$ catkin build --cmake-args -DCMAKE_BUILD_TYPE=Release
```

Примітка для Ubuntu 20.04: через помилку в python-catkin-tools вам потрібно використовувати catkin_make замість catkin build.

Це воно! Пакунки мають бути згенеровані та скомпільовані до бібліотеки. Скопіюйте файл devel/lib/libsimExtROS.so до папки встановлення CoppeliaSim. Тепер плагін готовий до використання!

Тепер відкрийте термінал і запустіть майстер ROS за допомогою:

```
$ roscore
```

Відкрийте інший термінал, перейдіть до папки встановлення CoppeliaSim і запустіть CoppeliaSim. Це те, що ви повинні мати (або подібне):

```
$ ./coppeliaSim.sh
...
Plugin 'ROSInterface': loading...
Plugin 'ROSInterface': load succeeded.
...
```

Після успішного завантаження інтерфейсу ROS перевірка доступних вузлів дає наступне:

```
$ rosnode list
/rosout
/sim_ros_interface
```

У порожній сцені CoppeliaSim виберіть об’єкт, а потім приєднайте до нього непотоковий [дочірній сценарій](https://www.coppeliarobotics.com/helpFiles/en/childScripts.htm) за допомогою [Menu bar --> Add --> Associated child script --> Non threaded]. Відкрийте [редактор сценаріїв](https://www.coppeliarobotics.com/helpFiles/en/scriptEditor.htm) для цього сценарію та замініть вміст таким:

```
function subscriber_callback(msg)
    -- This is the subscriber callback function
    sim.addLog(sim.verbosity_scriptinfos,'subscriber receiver following Float32: '..msg.data)
end

function getTransformStamped(objHandle,name,relTo,relToName)
    -- This function retrieves the stamped transform for a specific object
    t=sim.getSystemTime()
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

    -- Prepare the float32 publisher and subscriber (we subscribe to the topic we advertise):
    if simROS then
        publisher=simROS.advertise('/simulationTime','std_msgs/Float32')
        subscriber=simROS.subscribe('/simulationTime','std_msgs/Float32','subscriber_callback')
    end
end

function sysCall_actuation()
    -- Send an updated simulation time message, and send the transform of the object attached to this script:
    if simROS then
        simROS.publish(publisher,{data=sim.getSimulationTime()})
        simROS.sendTransform(getTransformStamped(objectHandle,objectAlias,-1,'world'))
        -- To send several transforms at once, use simROS.sendTransforms instead
    end
end

function sysCall_cleanup()
    -- Following not really needed in a simulation script (i.e. automatically shut down at simulation end):
    if simROS then
        simROS.shutdownPublisher(publisher)
        simROS.shutdownSubscriber(subscriber)
    end
end
```

Наведений вище сценарій опублікує час симуляції та одночасно підпишеться на нього. Він також опублікує перетворення об’єкта, до якого приєднано сценарій. Ви повинні мати змогу побачити тему часу моделювання за допомогою:

```
$ rostopic list
```

Щоб переглянути вміст повідомлення, ви можете ввести:

```
$ rostopic echo /simulationTime
```

Тепер завантажте демонстраційну сцену rosInterfaceTopicPublisherAndSubscriber.ttt і запустіть симуляцію. Код у [дочірньому сценарії](https://www.coppeliarobotics.com/helpFiles/en/childScripts.htm), прикріпленому до Vision_sensor, дозволить видавцю передавати потокове зображення датчика зору, а також дозволить передплатнику слухати той самий потік. Абонент передає зчитані дані на датчик пасивного зору, який використовується лише як контейнер даних. Отже, CoppeliaSim транслює дані, одночасно прослуховуючи ті самі дані! Ось що відбувається:

![image](https://user-images.githubusercontent.com/121936602/217101205-bc4f6cf9-8e0f-4da7-9218-5c60d48afdeb.png)

[Демонстрація видавця зображень і передплатників]

Спробуйте трохи поекспериментувати з кодом. Ви також можете візуалізувати зображення, яке транслює CoppeliaSim, за допомогою такої команди:

```
$ rosrun image_view image_view image:=/visionSensorData
```

Якби ви передавали простіші дані, ви також могли б візуалізувати їх за допомогою:

```
$ rostopic echo /visionSensorData
```

Тепер зупиніть симуляцію та завантажте демонстраційну сцену controlTypeExamples/controlledViaRos.ttt і запустіть симуляцію. Робот є спрощеним, а також поводиться спрощено для цілей спрощення. Управління здійснюється через [інтерфейс ROS](https://www.coppeliarobotics.com/helpFiles/en/rosInterf.htm):

![image](https://user-images.githubusercontent.com/121936602/217101507-affe7e6d-f123-4c30-a016-886913dd21e8.png)

[Зовнішня клієнтська програма, яка керує роботом через ROS]

Дочірній сценарій, приєднаний до робота, який працює безпотоковим способом, відповідає за наступне:

- визначити ручки деяких об’єктів (наприклад, ручки з’єднання двигуна та ручка датчика наближення)
- перевірте, чи завантажено інтерфейс ROS
- Запустіть абонентів швидкості двигуна
- Запустіть видавця датчиків і видавця симуляції часу
- і, нарешті, запустіть клієнтську програму. Програма викликається з деякими назвами тем як аргументами, щоб вона знала, які теми слухати та підписатися. Потім клієнтська програма (rosBubbleRob) бере на себе керування роботом через ROS.

Під час моделювання скопіюйте та вставте кілька разів робота. Зауважте, що кожна копія є безпосередньо робочою та незалежною. Це одна з багатьох сильних сторін CoppeliaSim.

Тепер зупиніть симуляцію та відкрийте нову сцену, а потім перетягніть туди таку модель: Models/tools/rosInterface helper tool.ttm. Ця модель складається з єдиного [сценарію налаштування](https://www.coppeliarobotics.com/helpFiles/en/customizationScripts.htm), який пропонує видавцям і передплатникам наступних тем:

- Тема startSimulation: можна використати для запуску симуляції шляхом публікації в цій темі повідомлення std_msgs::Bool.
- Тема pauseSimulation: можна використовувати для призупинення симуляції шляхом публікації в цій темі повідомлення std_msgs::Bool.
- Тема stopSimulation: можна використовувати для зупинки симуляції шляхом публікації в цій темі повідомлення std_msgs::Bool.
- Тема enableSyncMode: опублікувавши повідомлення std_msgs::Bool у цій темі, ви можете ввімкнути/вимкнути [ступінчастий режим](https://www.coppeliarobotics.com/helpFiles/en/simulation.htm#stepped).
- Тема triggerNextStep: опублікувавши повідомлення std_msgs::Bool у цій темі, ви можете запустити наступний крок симуляції, перебуваючи в [ступінчастому режимі](https://www.coppeliarobotics.com/helpFiles/en/simulation.htm#stepped).
- Тема simulationStepDone: повідомлення типу std_msgs::Bool буде опубліковано в кінці кожного проходу симуляції.
- Тема simulationState: повідомлення типу std_msgs::Int32 будуть публікуватися на регулярній основі. 0 вказує на те, що моделювання зупинено, 1 — що воно працює, а 2 — що воно призупинено.
- Тема simulationTime: повідомлення типу std_msgs::Float32 публікуватимуться на регулярній основі із зазначенням поточного часу симуляції.

Подивіться на вміст сценарію налаштування, який можна повністю налаштувати для різних цілей. Спробуйте створити повідомлення теми з командного рядка, наприклад:

```
$ rostopic pub /startSimulation std_msgs/Bool true --once
$ rostopic pub /pauseSimulation std_msgs/Bool true --once
$ rostopic pub /stopSimulation std_msgs/Bool true --once
$ rostopic pub /enableSyncMode std_msgs/Bool true --once
$ rostopic pub /startSimulation std_msgs/Bool true --once
$ rostopic pub /triggerNextStep std_msgs/Bool true --once
$ rostopic pub /triggerNextStep std_msgs/Bool true --once
$ rostopic pub /triggerNextStep std_msgs/Bool true --once
$ rostopic pub /stopSimulation std_msgs/Bool true --once
```

Щоб відобразити поточний час симуляції, ви можете ввести:

```
$ rostopic echo /simulationTime
```

Нарешті, обов’язково подивіться на функціональність [віддаленого API у CoppeliaSim](https://www.coppeliarobotics.com/helpFiles/en/remoteApiOverview.htm): вона дозволяє дистанційно виконувати функції, швидко передавати потокові дані вперед і назад, досить проста у використанні, легка та кросплатформна. Функція віддаленого API доступна для 7 різних мов і в деяких випадках може бути цікавою альтернативою ROS.
