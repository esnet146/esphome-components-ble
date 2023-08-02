# `ESPHome` components

A collection of my ESPHome components.

To use this repository you should confugure it inside your yaml-configuration:
```yaml
external_components:
  - source: github://dvb6666/esphome-components
```

or download component into `custom_components` folder (you can use another name) and add following lines to your yaml-configuration:
```yaml
external_components:
  - source: custom_components
```

You can take a look at samples of usage of those components in [examples](examples) folder.

## [BLE Client2](components/myhomeiot_ble_client2)
Copy of [myhomeiot BLE Client](https://github.com/myhomeiot/esphome-components/tree/main) that allows:
- To read set of characteristics from device at the **same** connection.
- To write characteristics (when option `value` presents). Examples:
```yaml
      - service_uuid: '0000fff0-0000-1000-8000-00805f9b34fb'
        characteristic_uuid: '0000fff1-0000-1000-8000-00805f9b34fb'
        value: [0xFD, 0x37, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xCB]
        
      - service_uuid: '0000fff0-0000-1000-8000-00805f9b34fb'
        characteristic_uuid: '0000fff1-0000-1000-8000-00805f9b34fb'
        value: !lambda |-
          return {0xFD, 0x37, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xCB};
```
- To subscribe for notifies (when option `notify` set to `true`). Example:
```yaml
      - service_uuid: '0000fff0-0000-1000-8000-00805f9b34fb'
        characteristic_uuid: '0000fff4-0000-1000-8000-00805f9b34fb'
        notify: true
```
- To force update BLE-client processing with action `myhomeiot_ble_client2.force_update` or method `force_update()` from lambda. Example:
```yaml
button:
  - platform: template
    name: "Force Update"
    on_press:
      - myhomeiot_ble_client2.force_update:
          id: my_ble_client
      # - lambda: |-
      #     id(my_ble_client).force_update();

myhomeiot_ble_client2:
  - mac_address: "01:23:45:67:89:AB"
    id: my_ble_client
```

Difference from build-in [ESPHome BLE Client](https://esphome.io/components/sensor/ble_client.html):
- Always disconnects from device after reading characteristic, this will allow to save device battery. You can specify `update_interval`, defaults to 60min.
- Uses lambda for parsing and extracting data into specific sensors make this component very flexible and useful for prototyping.
- There is no limit to the number of BLE Clients used (build-in BLE Client has limit of 3 instances). This component uses BLE Host component which you should count as one instance of build-in BLE Client. All BLE clients are processed sequentially inside the host component at time when they was detected and update interval reached.

#### ESPHome configuration example
Note: This example needs three template sensors with id: battery_level, tx_power, link_loss
```yaml
external_components:
  - source: github://myhomeiot/esphome-components
  - source: github://dvb6666/esphome-components

esp32_ble_tracker:
myhomeiot_ble_host:
myhomeiot_ble_client2:
  - mac_address: "01:23:45:67:89:AB"
    update_interval: 300sec
    services:
      - service_uuid: '180F'
        characteristic_uuid: '2A19'
      - service_uuid: '1804'
        characteristic_uuid: '2A07'
      - service_uuid: '1803'
        characteristic_uuid: '2A06'
    on_value:
      then:
        lambda: |-
          ESP_LOGD("M1", "Received [%d] for service %d", x[0], service);
          if (service==1) id(battery_level).publish_state(x[0]);
          else if (service==2) id(tx_power).publish_state(x[0]);
          else if (service==3) id(link_loss).publish_state(x[0]);
```

**More configuration examples you can find in [examples](examples) folder.**
