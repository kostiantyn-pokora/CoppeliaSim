# Навчальний посібник із плагінів #
Цей посібник описує, як написати плагін для CoppeliaSim. Файл сцени CoppeliaSim, пов’язаний із цим посібником, знаходиться в scenes/tutorials/BubbleRobExt. Файли проекту плагіна цього посібника можна знайти тут.
CoppeliaSim автоматично завантажує всі плагіни, які він може знайти у своїй папці (тобто папці інсталяції або тій же папці, що містить coppeliaSim.exe) під час запуску програми. CoppeliaSim розпізнає файли плагінів із такою маскою: "simExt*.dll" у Windows, "libsimExt*.dylib" у Mac OS та "libsimExt*.so" у Linux. Крім того, назва файлу плагіна не повинна містити символів підкреслення. Файл плагіна цього підручника – simExtBubbleRob.dll. Перевіряючи його, переконайтеся, що він був правильно завантажений під час запуску CoppeliaSim: перемкніть вікно консолі на видиме, знявши прапорець біля пункту «Приховати вікно консолі» в діалоговому вікні налаштувань користувача ([Menu bar --> Tools --> Settings]). Ця опція доступна лише у версії Windows. На Mac подивіться на консоль системи, а на Linux спробуйте запустити CoppeliaSim із консолі. Вікно консолі повинно відображати щось подібне до цього:

![image](https://user-images.githubusercontent.com/121936602/217090408-3129d288-dc0a-44ff-8d99-769075f2eaa8.png)

Як ви вже зрозуміли, цей плагін був написаний для BubbleRob з підручника BubbleRob. Завантажте відповідний файл сцени (scenes/tutorials/BubbleRobExt/BubbleRobExt.ttt). Плагін BubbleRob додає 4 нові функції сценарію (спеціальні функції сценарію мають відповідати угоді: "simXXX.YYY" для імені, наприклад simRob.start):

### simBubble.create ###
| Oпис | Створює екземпляр контролера BubbleRob у плагіні |
| --- | --- |
| Lua synopsis  | int bubbleRobHandle=simBubble.create(table[2] motorJointHandles,int sensorHandle,table[2] backRelativeVelocities) |
| Lua parameters  | **motorJointHandles**: таблиця, що містить маркери лівого та правого рухових суглобів BubbleRob, яким ви хочете керувати. **sensorHandle**: маркер датчика наближення або BubbleRob, яким ви хочете керувати  **backRelativeVelocities** коли BubbleRob виявляє перешкоду, він деякий час рухатиметься назад. relativeBackVelocities[1] — відносна швидкість лівого колеса під час руху назад. relativeBackVelocities[2] — відносна швидкість правого колеса під час руху назад |
| Lua return values | результат: -1 у разі помилки, інакше дескриптор контролера BubbleRob плагіна. |
| Python synopsis | int bubbleRobHandle=simBubble.create(list motorJointHandles,int sensorHandle,list backRelativeVelocities)

### simBubble.destroy ###
| Oпис | Знищує екземпляр контролера BubbleRob, раніше створений за допомогою simBubble.create |
| --- | --- |
| Lua synopsis | bool result=simBubble.destroy(int bubbleRobHandle) |
| Lua parameters | **bubbleRobHandle**: дескриптор екземпляра BubbleRob, попередньо повернутий із simBubble.create |
| Lua return values | результат: false у разі помилки |
| Python synopsis | bool result=simBubble.destroy(int bubbleRobHandle) |

### simBubble.start ###
| Oпис | Перемикає BubbleRob у автоматичний режим руху |
| --- | --- |
| Lua synopsis | bool result=simBubble.start(int bubbleRobHandle) |
| Lua parameters | **bubbleRobHandle**: дескриптор екземпляра BubbleRob, попередньо повернутий із simBubble.create |
| Lua return values | результат: false у разі помилки |
| Python synopsis | bool result=simBubble.start(int bubbleRobHandle) |

### simBubble.stop ###
| Oпис | Зупиняє автоматичний рух BubbleRob |
| --- | --- |
| Lua synopsis | bool result=simBubble.stop(int bubbleRobHandle) |
| Lua parameters | **bubbleRobHandle**: дескриптор екземпляра BubbleRob, попередньо повернутий із simBubble.create |
| Lua return values | результат: false у разі помилки |
| Python synopsis | bool result=simBubble.stop(int bubbleRobHandle) |

Тепер відкрийте потоковий дочірній сценарій, прикріплений до моделі BubbleRob на сцені (наприклад, двічі клацніть піктограму сценарію поруч із об’єктом bubbleRob в ієрархії сцени). Перевірте код:

```function sysCall_init()
    corout=coroutine.create(coroutineMain)
end

function sysCall_actuation()
    if coroutine.status(corout)~='dead' then
        local ok,errorMsg=coroutine.resume(corout)
        if errorMsg then
            error(debug.traceback(corout,errorMsg),2)
        end
    end
end

function coroutineMain()
    -- Check if the required plugin is there:
    moduleName=0
    moduleVersion=0
    index=0
    bubbleRobModuleNotFound=true
    while moduleName do
        moduleName,moduleVersion=sim.getModuleName(index)
        if (moduleName=='BubbleRob') then
            bubbleRobModuleNotFound=false
        end
        index=index+1
    end
    if bubbleRobModuleNotFound then
        sim.addLog(sim.verbosity_scripterrors,'BubbleRob plugin was not found. Simulation will not run properly.')
    else
        local jointHandles={sim.getObject('./leftMotor'),sim.getObject('./rightMotor')}
        local sensorHandle=sim.getObject('./sensingNose')
        local robHandle=simBubble.create(jointHandles,sensorHandle,{0.5,0.25})
        if robHandle>=0 then
            simBubble.start(robHandle) -- start the robot
            local st=sim.getSimulationTime()
            sim.wait(20) -- run for 20 seconds
            simBubble.stop(robHandle)
            simBubble.destroy(robHandle)
        end
    end
end
```
Перша частина коду відповідає за перевірку наявності плагіна, необхідного для запуску цього сценарію (тобто simExtBubbleRob.dll) (тобто його знайдено та успішно завантажено). Якщо ні, відображається повідомлення про помилку. В іншому випадку дескриптори з’єднання та датчика витягуються та передаються функції спеціального сценарію, яка створює екземпляр контролера нашого BubbleRob у плагіні. Якщо виклик був успішним, ми можемо викликати simBubble.start. Ця функція наказує плагіну переміщати модель BubbleRob, уникаючи перешкод. Запустіть симуляцію: BubbleRob рухається протягом 20 секунд, а потім зупиняється, як очікувалося. Тепер залиште CoppeliaSim. Тимчасово перейменуйте плагін на TEMP_simExtBubbleRob.dll, щоб CoppeliaSim більше не завантажував його, а потім запустіть CoppeliaSim знову. Завантажте попередню сцену та запустіть симуляцію: тепер з’являється повідомлення про помилку, яке вказує на те, що потрібний плагін не знайдено. Знову залиште CoppeliaSim, перейменуйте плагін на simExtBubbleRob.dll і знову запустіть CoppeliaSim.

Давайте подивимося, як плагін реєструє та обробляє вищезазначені 4 спеціальні функції Lua. Відкрийте проект плагіна BubbleRob і перегляньте файл simExtBubbleRob.cpp:

Зверніть увагу на 3 необхідні точки входу плагіна: simStart, simEnd і simMessage: simStart викликається один раз, коли плагін завантажується (ініціалізація), simEnd викликається один раз, коли плагін вивантажується (очищення), і simMessage викликається регулярно базу з кількома типами повідомлень.Під час фази ініціалізації плагін завантажує бібліотеку CoppeliaSim (щоб мати доступ до всіх функцій API CoppeliaSim), а потім реєструє 4 спеціальні функції сценарію. Функція спеціального сценарію реєструється шляхом вказівки:
- ім'я функції
- рядок підказки виклику
- адреса зворотного дзвінка

Коли скрипт викликає вказане ім’я функції, CoppeliaSim викликає адресу зворотного виклику. Найскладнішим завданням у функції зворотного виклику є правильне читання вхідних аргументів і правильний запис вихідних значень. Щоб полегшити завдання, використовуються два допоміжні класи, які відповідатимуть за це: CScriptFunctionData та CScriptFunctionDataItem, розташовані в програмуванні/загальному та програмуванні/включенні.

Під час написання власних функцій сценарію намагайтеся використовувати той самий макет/скелет коду, що й у файлі simExtBubbleRob.cpp.

Загалом, підпрограми зворотного виклику мають виконуватися якомога швидше, а потім керування має бути повернуто CoppeliaSim, інакше весь симулятор зупиниться.
