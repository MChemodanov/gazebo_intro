#Создание робота
На примере создания небольшой рабочей модели робота познакомимся с основными моментами Gazebo. 
Начнем с создания структуры файлов и пакетов для нашего робота в рабочей папке.
В терминале:
~~~~
$ mkdir ~/catkin_ws/src/mobot
$ cd ~/catkin_ws/src/mobot
$ catkin_create_pkg mobot_gazebo gazebo_ros
$ mkdir mobot_gazebo/launch mobot_gazebo/worlds
$ cd ../models
$ catkin_create_pkg mobot_description xacro
$ mkdir mobot_description/sdf
~~~~
В папке src/mobot будут находится пакеты связанные с роботом, включая mobot_gazebo, который интергрирует робота в Gazebo, в папке src/models/mobot_description будет находится описание модели робота. 
##Описание модели робота 
3-х мерная физическая модель робота в Gazebo описывается с помощью файла формата sdf, близкого родственника urdf, который обычно используется в ROS’е. В сети множество туториалов как использовать urdf в Gazebo, однако все они основаны на невероятно забагованной стандартной утилите, которая преобразует один формат в другой. Я не рекомендую пользоваться urdf для Gazebo, если urdf файл все же необходим, то проще написать его отдельно.

Создадим файл model.config в папке mobot_description со следующим содержанием 
~~~~
<?xml version="1.0"?>
<model>
  <name>Mobot</name>
  <version>1.0</version>
  <sdf version='1.4'>sdf/model.sdf</sdf>

  <author>
   <name>My Name</name>
   <email>me@my.email</email>
  </author>

  <description>
    My awesome robot.
  </description>
</model>
~~~~
В этом файле самое главное указать путь до sdf файла модели.
##Введение в sfd формат
Создадим файл sfd/model.sdf и добавим в него содержимое:
~~~~
<?xml version='1.0'?>
<sdf version='1.4'>
  <model name="mobot" xmlns:xacro="http://www.ros.org/wiki/xacro">
  </model>
</sdf>
~~~~
Описание самой модели прописывается между тэгами `model`, добавим между тэгами следующий код:
~~~~
<link name='chassis'>
  <pose>0 0 0.1 0 0 0</pose>
  <collision name='collision'>
    <geometry>
      <box>
        <size>0.5 0.2 0.1</size>
      </box>
    </geometry>
  </collision>

  <visual name='visual'>
    <geometry>
      <box>
        <size>0.5 0.2 0.1</size>
      </box>
    </geometry>
    <material>
      <script>
        <name>Gazebo/Orange</name>
        <uri>__default__</uri>
      </script>
    </material>
  </visual>

  <inertial> 
    <mass>5</mass> 
    <inertial>
      <inertia>
        <ixx>0.2</ixx>
        <iyy>0.3</iyy>
        <izz>0.2</izz>
        <ixy>0</ixy>
        <ixz>0</ixz>
        <iyz>0</iyz>
      </inertia>
    </inertial>
</link>
~~~~
Тэг `link` определяет базовый структурный элемент, их может быть несколько в одной моделе.
Внутри `link` нужно определить тэги:
- `pose` в формате xyz rpy для определения позиции относительно начала координат модели
- `collision`(их может быть много внутри однога линка) используется для просчета столкновений, в нем можно использовать как стандартные примитивы, так и импортировать модель форматов .dae или .stl. Здесь же описываются свойства поверхности вроде трения. Добавив сюда `pose` можно сдвинуть мэш относительно родительского линка
- `visual`(их так же может быть много) добавляется для рендеринга, здесь также можно использовать как стандартные примитивы, так и импортировать модель форматов .dae или .stl. Здесь же добавляются текстуры. Добавив сюда `pose` можно сдвинуть мэш относительно родительского линка
- `inertial`(он в линке один) описывает физические инерциальные свойства линка: его массу, тензор инерции и т.д. Добавив сюда `pose` можно сдвинуть координаты центра масс относительно родительского линка
Подробнее про спецификации sdf формата написано [здесь](http://sdformat.org/spec) (в нашей версии Gazebo нужно выбрать 1.4 версию sdf)

##Создание sdf файла модели робота с помощью утилиты xacro



Теперь создадим в папке sdf файл с названием model.sdf.xacro и добавим в него следующее содержание:
~~~~
<?xml version='1.0'?>
<sdf version='1.4'>
  <model name="mobot" xmlns:xacro="http://www.ros.org/wiki/xacro">


    <xacro:property name="PI" value="3.1415926535897931"/>

    <xacro:property name="chassisHeight" value="0.1"/>
    <xacro:property name="chassisLength" value="0.4"/>
    <xacro:property name="chassisWidth" value="0.2"/>
    <xacro:property name="chassisMass" value="50"/>

    <xacro:property name="casterRadius" value="0.05"/>
    <xacro:property name="casterMass" value="5"/>

    <xacro:property name="wheelWidth" value="0.05"/>
    <xacro:property name="wheelRadius" value="0.1"/>
    <xacro:property name="wheelPos" value="0.12"/>
    <xacro:property name="wheelMass" value="5"/>
    <xacro:property name="wheelDamping" value="1"/>

    <xacro:property name="cameraSize" value="0.05"/>
    <xacro:property name="cameraMass" value="0.1"/>

    
    <xacro:include filename="$(find mobot_description)/sdf/macros.xacro" />

    <static>false</static>

    <link name='chassis'>
      <pose>0 0 ${wheelRadius} 0 0 0</pose>
      <collision name='collision'>
        <geometry>
          <box>
            <size>${chassisLength} ${chassisWidth} ${chassisHeight}</size>
          </box>
        </geometry>
      </collision>

      <visual name='visual'>
        <geometry>
          <box>
            <size>${chassisLength} ${chassisWidth} ${chassisHeight}</size>
          </box>
        </geometry>
        <material>
          <script>
            <name>Gazebo/Orange</name>
            <uri>__default__</uri>
          </script>
        </material>
      </visual>

      <inertial> 
        <mass>${chassisMass}</mass> 
        <box_inertia m="${chassisMass}" x="${chassisLength}" y="${chassisWidth}" z="${chassisHeight}"/>
      </inertial>

      <collision name='caster_collision'>
        <pose>${-chassisLength/3} 0 ${-chassisHeight/2} 0 0 0</pose>
        <geometry>
          <sphere>
            <radius>${casterRadius}</radius>
          </sphere>
        </geometry>

        <surface>
          <friction>
            <ode>
              <mu>0</mu>
              <mu2>0</mu2>
              <slip1>1.0</slip1>
              <slip2>1.0</slip2>
            </ode>
          </friction>
        </surface>
      </collision>

      <visual name='caster_visual'>
        <pose>${-chassisLength/3} 0 ${-chassisHeight/2} 0 0 0</pose>
        <geometry>
          <sphere>
            <radius>${casterRadius}</radius>
          </sphere>
        </geometry>
        <material>
          <script>
            <name>Gazebo/Red</name>
            <uri>__default__</uri>
          </script>
        </material>
      </visual>
    </link>

  </model>
</sdf>
~~~~