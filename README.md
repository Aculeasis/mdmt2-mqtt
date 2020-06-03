# mqtt plugin for mdmTerminal2
Позволяет работать с [терминал](https://github.com/Aculeasis/mdmTerminal2) по MQTT протоколу.

# Установка
```
mdmTerminal2/env/bin/python -m pip install paho-mqtt
cd mdmTerminal2/src/plugins
git clone https://github.com/Aculeasis/mdmt2-mqtt.git
```
И перезапустить терминал.
```
systemctl restart mdmterminal2
```

## Настройка
Настройки хранятся в `mdmTerminal2/src/settings.ini`, в секции `"smarthome"`:
- **ip**: На который будут отправлены `mqtt` сообщения.
- **terminal**: топик. По умолчанию `terminal`.

# Дополнительно

```
MQTT топики для работы с терминалом
#topics:
TOPIC составляется из настройки terminal =  в секции smarthome и выставляется в "terminal" при отсутствии 
в настройках.
cmd: передача: текста терминалу для проговаривания голосом, команд для управления терминалом
conversation: прием терминалом текста для обработки в интеграции conversation
state: прием статусов терминала для автоматизаций 
Например в настройках установлено:
terminal = 'room' то топик будет room/cmd
terminal = то топик автоматически будет установлен в terminal/cmd


Команды принимаемые терминалом формате json
voice, tts', 'ask', 'volume', 'nvolume'
Их описание: https://github.com/Aculeasis/mdmTerminal2/wiki/API-(draft)
поример
{"tts":"ТЕСТ"} - сказать "ТЕКСТ"
{"volume":"50"} - установить громкость терминала в 50%

 ###Home Assistant config###

Сенсоры для приема сообщений от терминала
#sensor
- platform: mqtt
  state_topic: "terminal/conversation"
  name: 'terminal_room'

- platform: mqtt
  state_topic: "terminal/state"
  name: 'terminal_room_state'

Загоняем пришедший текст в интеграцию conversation
#automation 
- alias: 'Terminal Room'
  trigger:
    platform: state
    entity_id: sensor.terminal_room
  action:
  - service: conversation.process
    data_template:
      text: '{{ states("sensor.terminal_room") }}'

Скрипт передачи текста терминалу для проговаривания голосом
#scripts
notify_mqtt:
  sequence:
  - service: mqtt.publish
    data_template:
      payload: '{"tts":"{{ message }}"}'
      topic: terminal/cmd

Пример использования в автоматизациях 
- alias: 'ТЕСТ'
  trigger:
    - platform: state
      entity_id:
        - binary_sensor.window_sensor
      to: 'on'
  action:
   - service: script.notify_mqtt
      data_template:
        message: "Тут текс для проговаривания терминалом"
```