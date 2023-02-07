# Рядок після підручника BubbleRob #

У цьому підручнику ми прагнемо розширити функціональність BubbleRob, щоб він міг слідувати лінією на землі. Переконайтеся, що ви повністю прочитали та зрозуміли [перший підручник BubbleRob](https://www.coppeliarobotics.com/helpFiles/en/bubbleRobTutorial.htm). Цей підручник люб’язно надано Еріком Ромером.

Завантажте сцену з першого підручника BubbleRob, розташованого в scenes/tutorials/BubbleRob. Файл сцени, пов’язаний з цим підручником, знаходиться в scenes/tutorials/LineFollowingBubbleRob. Наступний малюнок ілюструє сцену моделювання, яку ми розробимо:

![image](https://user-images.githubusercontent.com/121936602/217070315-69648f4b-a320-4e95-99d9-d4edc840a2eb.png)

Спочатку ми створюємо перший із 3 [датчиків зору](https://www.coppeliarobotics.com/helpFiles/en/visionSensors.htm), які ми приєднаємо до об’єкта bubbleRob. Виберіть [Menu bar --> Add --> Vision sensor --> Orthographic type]. Відредагуйте його властивості, двічі клацнувши піктограму щойно створеного датчика зору в [ієрархії сцени](https://www.coppeliarobotics.com/helpFiles/en/userInterface.htm#SceneHierarchy), і змініть параметри, щоб відобразити таке діалогове вікно:

![image](https://user-images.githubusercontent.com/121936602/217070505-12c41609-cf17-4048-aaf2-b9b9c17c20fa.png)

Датчик зору має бути спрямований до землі, тому виберіть його та в [діалоговому вікні орієнтації](https://www.coppeliarobotics.com/helpFiles/en/orientationDialog.htm) на вкладці орієнтації встановіть [180;0;0] для елементів Alpha-Beta-Gamma.

У нас є кілька можливостей зчитування датчика зору. Оскільки наш датчик зору має лише один піксель і працює легко, ми просто запитаємо середнє значення інтенсивності зображення, яке зчитує наш датчик зору. Для більш складних випадків ми могли б налаштувати [функцію зворотного виклику зору](https://www.coppeliarobotics.com/helpFiles/en/visionCallbackFunctions.htm). Тепер двічі скопіюйте та вставте датчик зору та налаштуйте його псевдоніми на eftSensor, middleSensor і rightSensor. Зробіть bubbleRob їхнім батьківським (тобто приєднайте їх до об’єкта bubbleRob). Тепер ваші датчики мають виглядати так в ієрархії сцени:

![image](https://user-images.githubusercontent.com/121936602/217070678-851c513a-f1a7-4afb-abb0-bcdf4a8c625e.png)

Давайте правильно розташуємо датчики. Для цього скористайтеся [діалоговим вікном позиції](https://www.coppeliarobotics.com/helpFiles/en/positionDialog.htm) на вкладці позиції та встановіть такі абсолютні координати:
- лівий датчик: [0,2;0,042;0,018]
- середній датчик: [0,2;0;0,018]
- правий датчик: [0,2;-0,042;0,018]

Тепер модифікуємо середовище. Ми можемо видалити кілька циліндрів перед BubbleRob. Далі ми побудуємо [шлях](), яким намагатиметься слідувати робот: натисніть [Menu bar --> Add --> Path --> Closed]. У вас є кілька можливостей змінити форму контуру, маніпулюючи його контрольними точками: ви можете видалити або скопіювати їх, а також ви можете зсунути/переорієнтувати їх. Увімкніть рух об'єкта за допомогою миші та налаштуйте шлях на свій смак.

Коли ви будете задоволені геометрією контуру (ви завжди можете змінити його пізніше), відкрийте доданий до нього сценарій налаштування та замініть його вміст на:
```path=require('path_customization')

function path.shaping(path,pathIsClosed,upVector)
    local section={-0.02,0.001,0.02,0.001}
    local color={0.3,0.3,0.3}
    local options=0
    if pathIsClosed then
        options=options|4
    end
    local shape=sim.generateShapeFromPath(path,section,options,upVector)
    sim.setShapeColor(shape,nil,sim.colorcomponent_ambient_diffuse,color)
    return shape
end
```
Перезапустіть сценарій налаштування, щоб зміни набули чинності, потім відкрийте діалогове вікно конфігурації шляху користувача та поставте прапорець у полі «Створити екструдовану форму».
Останнім кроком є налаштування контролера BubbleRob, щоб він також слідував чорному шляху. Відкрийте дочірній сценарій, прикріплений до bubbleRob, і замініть його таким кодом:
```function sysCall_init()
    bubbleRobBase=sim.getObject('.')
    leftMotor=sim.getObject("./leftMotor")
    rightMotor=sim.getObject("./rightMotor")
    noseSensor=sim.getObject("./sensingNose")
    minMaxSpeed={50*math.pi/180,300*math.pi/180}
    backUntilTime=-1 -- Tells whether bubbleRob is in forward or backward mode
    floorSensorHandles={-1,-1,-1}
    floorSensorHandles[1]=sim.getObject("./leftSensor")
    floorSensorHandles[2]=sim.getObject("./middleSensor")
    floorSensorHandles[3]=sim.getObject("./rightSensor")
    robotTrace=sim.addDrawingObject(sim.drawing_linestrip+sim.drawing_cyclic,2,0,-1,200,{1,1,0})
    -- Create the custom UI:
    xml = '<ui title="'..sim.getObjectAlias(bubbleRobBase,1)..' speed" closeable="false" resizeable="false" activate="false">'..[[
                <hslider minimum="0" maximum="100" on-change="speedChange_callback" id="1"/>
            <label text="" style="* {margin-left: 300px;}"/>
        </ui>
        ]]
    ui=simUI.create(xml)
    speed=(minMaxSpeed[1]+minMaxSpeed[2])*0.5
    simUI.setSliderValue(ui,1,100*(speed-minMaxSpeed[1])/(minMaxSpeed[2]-minMaxSpeed[1]))
    
end

function sysCall_sensing()
    local p=sim.getObjectPosition(bubbleRobBase,-1)
    sim.addDrawingObjectItem(robotTrace,p)
end 

function speedChange_callback(ui,id,newVal)
    speed=minMaxSpeed[1]+(minMaxSpeed[2]-minMaxSpeed[1])*newVal/100
end

function sysCall_actuation() 
    result=sim.readProximitySensor(noseSensor)
    if (result>0) then backUntilTime=sim.getSimulationTime()+4 end
    
    -- read the line detection sensors:
    sensorReading={false,false,false}
    for i=1,3,1 do
        result,data=sim.readVisionSensor(floorSensorHandles[i])
        if (result>=0) then
            sensorReading[i]=(data[11]<0.5) -- data[11] is the average of intensity of the image
        end
    end
    
    -- compute left and right velocities to follow the detected line:
    rightV=speed
    leftV=speed
    if sensorReading[1] then
        leftV=0.03*speed
    end
    if sensorReading[3] then
        rightV=0.03*speed
    end
    if sensorReading[1] and sensorReading[3] then
        backUntilTime=sim.getSimulationTime()+2
    end
    
    if (backUntilTime<sim.getSimulationTime()) then
        -- When in forward mode, we simply move forward at the desired speed
        sim.setJointTargetVelocity(leftMotor,leftV)
        sim.setJointTargetVelocity(rightMotor,rightV)
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
Ви можете легко налагодити свою лінію за такими датчиками зору: виберіть один, потім у перегляді сцени виберіть [Right-click --> Add --> Floating view], а потім у щойно доданому плаваючому перегляді виберіть [Right click --> View --> Associate view with selected vision sensor].
