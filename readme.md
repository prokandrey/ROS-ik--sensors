# ROSir-sensors
**Robot Operating System - interactive Kit**  

![Робот ROSik](/images/ROSik.png)

Мини-робот на **ESP32 Wroom-32 + ROS 2**, который:
* транслирует одометрию, лидар и данные ИК-датчиков по Wi-Fi через WebSocket-мост;
* строит карты и локализуется (`slam_toolbox`);
* планирует путь (`nav2`);
* обнаруживает препятствия с помощью ИК-датчиков и лидара;
* легко расширяется модулями — камера, IMU, динамик, адресные LED, OLED-дисплей.

Работает **на любой машине**: чистая Ubuntu 24.04, WSL 2 под Windows или виртуалка — нужен лишь Wi-Fi.

Инструкция по установке [Ubuntu](https://stepik.org/lesson/1505338/step/1?unit=1525484), [WSL](https://stepik.org/lesson/1505339/step/4?unit=1525485), а также  [запуску графический приложений (RVIZ)](https://stepik.org/lesson/1505339/step/5?unit=1525485).

---

## 📑 Содержание
1. [Аппаратная часть (BOM)](#аппаратная-часть-bom)  
2. [Электрическая схема](#электрическая-схема)  
3. [3D-модели и сборка](#3d-модели-и-сборка)  
4. [Прошивка ESP32](#прошивка-esp32)
5. [Настройка PID и тестирование](#настройка-pid)
6. [Установка нод в ROS2](#установка-по-на-пк)  
7. [Запуск ROS 2-нод](#запуск-ros-2-нод)  
8. [Контакты](#контакты)

---

## 🛒<a id="аппаратная-часть-bom">Аппаратная часть (BOM)</a>

* | 1 | **ESP32 Wroom-32 DevKit**         | https://aliexpress.ru/item/1005006697750568.html
* | 2 | DC-мотор N20 60 RPM + энкодер     | https://aliexpress.ru/item/1005007145668771.html
* | 3 | Драйвер моторов ZK-5AD            | https://aliexpress.ru/item/1005005798367960.html
* | 4 | Лидар                             | https://aliexpress.ru/item/1005008371059417.html
* | 5 | Аккумулятор 18650 × 2 + держатель | https://aliexpress.ru/item/1005005447141518.html
* | 6 | Понижающий DC-DC                  | https://aliexpress.ru/item/32896699470.html
* | 7 | Переключатель питания             | https://aliexpress.ru/item/4000973563250.html
* | 8 | Type-C зарядка  2S 1A             | https://aliexpress.ru/item/1005006628986640.html
* | 9 | ИК-датчик × 2                     | https://aliexpress.ru/item/1005001456186405.html

---

## 🔌 <a id="электрическая-схема">Электрическая схема</a>

Файлы схемы:
* CorelDraw: **`/Scheme/ROSik_scheme.cdr`**  
* PNG: **`/Scheme/scheme.png`**  
* PDF: **`/Scheme/scheme.pdf`**

В схеме показаны:
* Соединения питания (8,4 V → 5 V)
* Сигнальные линии к драйверу моторов
* Подключение энкодеров
* Лидар (UART)
* ИК-датчики (GPIO)

![Схема робота](/Scheme/scheme.png)

**Подключение ИК-датчиков:**
* Левый датчик: GPIO 14
* Правый датчик: GPIO 12
* Питание: 5V
* Земля: GND

---

## 🖨 <a id="3d-модели-и-сборка">3D-модели и сборка</a>

Каталог **`/3D`** содержит:
* Файлы SolidWorks (`.SLDPRT/.SLDASM`)
* Готовые `.STL` для печати
* Обновленные модели с креплениями для ИК-датчиков

![3D-модель](images/3d.png)
* | `00 - WheelLayer.*`    | Нижняя плита с моторами и ИК-датчиками
* | `01 - LidarLayer.*`    | Верхняя плита под лидар 
* | `10 - ESPLayer.*`      | Средняя плита под ESP32 и драйвер 
* | `Bracket.* / Holder.*` | Крепёж моторов, DC-DC, ИК-датчиков
* | `RosikAssambl.SLDASM`  | Полная сборка 
* | `STL/`                 | Файлы для печати (0,2 мм, PLA) 
* | `ROSik.png`            | Рендер итогового вида 

**Сборка:**
1. Установите ИК-датчики на нижний слой (направлены вперед)
2. Проложите провода к ESP32 через внутренние каналы
3. Собирайте снизу → вверх, фиксируя слои стойками M3 × 20 мм

---

## 🔥<a id="прошивка-esp32"> Прошивка ESP32 </a>

### Требуемые библиотеки (Arduino IDE)

* Скопируйте ZIP-файлы из `/esp32_libraries/` в `Documents/Arduino/libraries`  
* Добавьте плату ESP32: `File → Preferences → Additional URL` → `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`
* Через Board Manager добавьте ESP32

### Обновления прошивки для ИК-датчиков

В файле **`/esp32_firmware/esp32_firmware.ino`** добавлены:
1. Чтение данных с ИК-датчиков
2. Публикация состояния датчиков через WebSocket
3. Логика экстренной остановки при обнаружении препятствий

**Шаги прошивки:**
1. Откройте скетч в Arduino IDE
2. Настройте параметры:
   ```cpp
   const char* ssid = "your_SSID";
   const char* password = "your_PASSWORD";
   const int IR_LEFT_PIN = 34;
   const int IR_RIGHT_PIN = 35;
   const int IR_THRESHOLD = 2000; // Порог срабатывания
Загрузите прошивку (Ctrl + U)

В Serial Monitor (115200 baud) появится IP и данные датчиков

<a id="настройка-pid">Настройка PID и тестирование робота</a>
Обновленная утилита включает визуализацию данных с ИК-датчиков:
https:///pythonGUI/gui.png

Новые функции:

Отображение состояния ИК-датчиков

Настройка порога срабатывания

Тестирование реакции на препятствия

🐧 Установка Ноды в ROS2
bash
# 1. Создаём рабочую директорию
mkdir -p ~/ros2_ws/src && cd ~/ros2_ws/src

# 2. Копируем esp32_bridge в ~/ros2_ws/src

# 3. Сборка
cd ~/ros2_ws
colcon build
source install/setup.bash
Обновления:

Новый топик /ir_sensors с данными ИК-датчиков

Конфигурационные файлы в esp32_bridge/config

🚀<a id="запуск-ros-2-нод"> Запуск ROS 2-нод </a>
Команда	Описание
ros2 run esp32_bridge esp32_bridge --ros-args -p host:=<IP_ESP32>	Запуск моста с поддержкой ИК-датчиков
ros2 topic echo /ir_sensors	Просмотр данных с ИК-датчиков
ros2 launch nav2_bringup navigation_launch.py params_file:=~/ros2_ws/src/esp32_bridge/config/nav_param.yaml	Навигация с учетом ИК-датчиков
Новые параметры в nav_param.yaml:

yaml
ir_sensors:
  enabled: true
  safety_distance: 0.15  # Минимальная дистанция до препятствия
📊 Пример работы с ИК-датчиками
python
# Пример подписки на данные ИК-датчиков
import rclpy
from rclpy.node import Node
from std_msgs.msg import Int32MultiArray

class IRSubscriber(Node):
    def __init__(self):
        super().__init__('ir_subscriber')
        self.subscription = self.create_subscription(
            Int32MultiArray,
            '/ir_sensors',
            self.listener_callback,
            10)
        
    def listener_callback(self, msg):
        left, right = msg.data
        self.get_logger().info(f'IR Sensors: L={left}, R={right}')

def main(args=None):
    rclpy.init(args=args)
    node = IRSubscriber()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
