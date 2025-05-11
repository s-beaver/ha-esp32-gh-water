# ha-esp32-gh-water (home assistant greenhouse watering by esp32)
Qwen2.5-Coder-32B-Instruct

У меня есть сервер home assistant, плата esp32 (она уже подключена к серверу) и вентиль, который может открывать и закрывать воду, если подавать на его контакты напряжение. Мне нужно разобраться каким образом запрограммировать esp32, чтобы я мог из интерфейса home assistant открывать и закрывать воду. Расскажи, что мне нужно сделать и каким образом я могу написать код, управляющий вентилем
Для того чтобы управлять вентилем с помощью ESP32 и Home Assistant, вам потребуется выполнить несколько шагов. Ниже приведены подробные инструкции:

### 1. Подключение оборудования
Сначала подключите вентиль к ESP32. Вентиль должен быть подключен через реле, поскольку напряжение для управления вентилем обычно выше, чем то, что может выдавать ESP32. Вот пример подключения:

- **ESP32 GPIO**: Подключите GPIO пин ESP32 к входному контакту реле.
- **Реле**: Выходное контактное реле подключите к вентилю.
- **Вентиль**: Подключите вентиль к источнику питания и земле.

### 2. Установка среды разработки
Если у вас еще не установлен Arduino IDE, скачайте и установите его с [официального сайта](https://www.arduino.cc/en/software).

### 3. Установка библиотеки для ESP32
Откройте Arduino IDE и перейдите в `Скетч` > `Подключить библиотеку` > `Управление библиотеками`. В поиске найдите и установите библиотеку `ESPAsyncWebServer` для создания веб-сервера на ESP32 и библиотеку `PubSubClient` для работы с MQTT, если вы хотите использовать MQTT для взаимодействия с Home Assistant.

### 4. Программирование ESP32
Ниже приведен пример кода, который позволит управлять вентилем через HTTP-запросы. Этот код можно расширить для использования MQTT или других протоколов.

```cpp
#include <WiFi.h>
#include <ESPAsyncWebServer.h>

const char* ssid = "ВАШ_SSID";
const char* password = "ВАШ_ПАРОЛЬ";

// Пин, к которому подключено реле
const int relayPin = 2;

AsyncWebServer server(80);

void setup() {
  Serial.begin(115200);
  pinMode(relayPin, OUTPUT);
  digitalWrite(relayPin, LOW); // Вентиль закрыт при старте

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println(WiFi.localIP());

  server.on("/open", HTTP_GET, [](AsyncWebServerRequest *request){
    digitalWrite(relayPin, HIGH);
    request->send(200, "text/plain", "Ventil opened");
  });

  server.on("/close", HTTP_GET, [](AsyncWebServerRequest *request){
    digitalWrite(relayPin, LOW);
    request->send(200, "text/plain", "Ventil closed");
  });

  server.begin();
}

void loop() {
  // В цикле ничего не делаем, так как все управление происходит через HTTP
}
```

### 5. Настройка Home Assistant
Теперь, когда ESP32 готов, добавьте его в Home Assistant. Для этого можно использовать компонент `rest_command`.

Добавьте следующие строки в файл `configuration.yaml` вашего Home Assistant:

```yaml
rest_command:
  open_valve:
    url: 'http://IP_ESP32/open'
    method: GET
  close_valve:
    url: 'http://IP_ESP32/close'
    method: GET

script:
  open_valve:
    sequence:
      - service: rest_command.open_valve

  close_valve:
    sequence:
      - service: rest_command.close_valve

input_boolean:
  valve_state:
    name: Valve State
    initial: off

automation:
  - alias: Open Valve
    trigger:
      platform: state
      entity_id: input_boolean.valve_state
      to: 'on'
    action:
      service: script.open_valve

  - alias: Close Valve
    trigger:
      platform: state
      entity_id: input_boolean.valve_state
      to: 'off'
    action:
      service: script.close_valve
```

Замените `IP_ESP32` на IP-адрес вашего ESP32.

Теперь вы можете управлять вентилем, изменяя состояние `input_boolean.valve_state` в Home Assistant.

### 6. Проверка
Перезагрузите Home Assistant и проверьте работу системы. Изменяйте состояние `input_boolean.valve_state`, чтобы открыть и закрыть вентиль.

Это базовая настройка. Вы можете расширить функционал, добавив больше функций и улучшив безопасность.

![wiring drawio](wiring.drawio.svg)
