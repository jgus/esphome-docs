ESP32 Bluetooth Low Energy Tracker Hub
======================================

.. seo::
    :description: Instructions for setting up ESP32 bluetooth low energy device trackers using ESPHome.
    :image: bluetooth.svg

The ``esp32_ble_tracker`` component creates a global hub so that you can track bluetooth low
energy devices using your ESP32 node.

See :ref:`Setting up devices <esp32_ble_tracker-setting_up_devices>`
for information on how you can find out the MAC address of a device and track it using ESPHome.

.. code-block:: yaml

    # Example configuration entry
    esp32_ble_tracker:

    binary_sensor:
      - platform: ble_presence
        mac_address: AC:37:43:77:5F:4C
        name: "ESP32 BLE Presence Google Home Mini"

    sensor:
      - platform: ble_rssi
        mac_address: AC:37:43:77:5F:4C
        name: "BLE Google Home Mini RSSI value"
      - platform: xiaomi_hhccjcy01
        mac_address: 94:2B:FF:5C:91:61
        temperature:
          name: "Xiaomi MiFlora Temperature"
        moisture:
          name: "Xiaomi MiFlora Moisture"
        illuminance:
          name: "Xiaomi MiFlora Illuminance"
        conductivity:
          name: "Xiaomi MiFlora Soil Conductivity"
        battery_level:
          name: "Xiaomi MiFlora Battery Level"
      - platform: xiaomi_lywsdcgq
        mac_address: 7A:80:8E:19:36:BA
        temperature:
          name: "Xiaomi MiJia Temperature"
        humidity:
          name: "Xiaomi MiJia Humidity"
        battery_level:
          name: "Xiaomi MiJia Battery Level"

.. note::

    The first time this component is enabled for an ESP32, the code partition needs to be
    resized. Please flash the ESP32 via USB when adding this to your configuration. After that,
    you can use OTA updates again.

.. _config-esp32_ble_tracker:

Configuration variables:
------------------------


- **scan_parameters** (*Optional*): Advanced parameters for configuring the scan behavior of the ESP32.
  See also `this guide by Texas Instruments <https://dev.ti.com/tirex/explore/content/simplelink_academy_cc2640r2sdk_5_10_02_00/modules/blestack/ble_scan_adv_basic/ble_scan_adv_basic.html#scanning-basics>`__
  for reference.

  - **interval** (*Optional*, :ref:`config-time`): The interval between each consecutive scan window.
    This is the time the ESP spends on each of the 3 BLE advertising channels.
    Defaults to ``320ms``.
  - **window** (*Optional*, :ref:`config-time`): The time the ESP is actively listening for packets
    on a channel during each scan interval. If this is close to the ``interval`` value, the ESP will
    spend more time listening to packets (but also consume more power). Defaults to ``30ms``
  - **duration** (*Optional*, :ref:`config-time`): The duration of each complete scan. This has no real
    impact on the device but can be used to debug the BLE stack. Defaults to ``5min``.
  - **active** (*Optional*, boolean): Whether to actively send scan requests to request more data
    after having received an advertising packet. With some devices this is necessary to receive all data,
    but also drains those devices' power a bit more. Some devices don't need this, in that case
    you can save power and RF pollution by setting it to ``false``. Defaults to ``true``.
  - **continuous** (*Optional*, boolean): Whether to scan continuously (forever) or to only scan when
    asked to start a scan (with start_scan action). Defaults to ``true``.

- **id** (*Optional*, :ref:`config-id`): Manually specify the ID for this ESP32 BLE Hub.

Automations:

- **on_ble_advertise** (*Optional*, :ref:`Automation <automation>`): An automation to perform
  when a Bluetooth advertising is received. See :ref:`esp32_ble_tracker-on_ble_advertise`.
- **on_ble_manufacturer_data_advertise** (*Optional*, :ref:`Automation <automation>`): An automation to
  perform when a Bluetooth advertising with manufacturer data is received. See
  :ref:`esp32_ble_tracker-on_ble_manufacturer_data_advertise`.
- **on_ble_service_data_advertise** (*Optional*, :ref:`Automation <automation>`): An automation to
  perform when a Bluetooth advertising with service data is received. See
  :ref:`esp32_ble_tracker-on_ble_service_data_advertise`.
- **on_scan_end** (*Optional*, :ref:`Automation <automation>`): An automation to perform when
  a BLE scan has completed (the duration of the scan). This works with continuous set to true or false.


ESP32 Bluetooth Low Energy Tracker Automation
---------------------------------------------

.. _esp32_ble_tracker-on_ble_advertise:

``on_ble_advertise`` Trigger
************************************************

This automation will be triggered when a Bluetooth advertising is received. A variable ``x`` of type
:apiclass:`esp32_ble_tracker::ESPBTDevice` is passed to the automation for use in lambdas.

.. code-block:: yaml

    esp32_ble_tracker:
      on_ble_advertise:
        - mac_address: 
            - 11:11:11:11:11:11
            - 22:22:22:22:22:22
          then:
            - lambda: |-
                ESP_LOGD("ble_adv", "New BLE device");
                ESP_LOGD("ble_adv", "  address: %s", x.address_str().c_str());
                ESP_LOGD("ble_adv", "  name: %s", x.get_name().c_str());
                ESP_LOGD("ble_adv", "  Advertised service UUIDs:");
                for (auto uuid : x.get_service_uuids()) {
                    ESP_LOGD("ble_adv", "    - %s", uuid.to_string().c_str());
                }
                ESP_LOGD("ble_adv", "  Advertised service data:");
                for (auto data : x.get_service_datas()) {
                    ESP_LOGD("ble_adv", "    - %s: (length %i)", data.uuid.to_string().c_str(), data.data.size());
                }
                ESP_LOGD("ble_adv", "  Advertised manufacturer data:");
                for (auto data : x.get_manufacturer_datas()) {
                    ESP_LOGD("ble_adv", "    - %s: (length %i)", data.uuid.to_string().c_str(), data.data.size());
                }

Configuration variables:

- **mac_address** (*Optional*, list of MAC Address): The MAC address to filter for this automation.
- See :ref:`Automation <automation>`.

.. _esp32_ble_tracker-on_ble_manufacturer_data_advertise:

``on_ble_manufacturer_data_advertise`` Trigger
************************************************

This automation will be triggered when a Bluetooth advertising with manufcaturer data is received. A
variable ``x`` of type ``std::vector<uint8_t>`` is passed to the automation for use in lambdas.

.. code-block:: yaml

    sensor:
      - platform: template
        name: "BLE Sensor"
        id: ble_sensor

    esp32_ble_tracker:
      on_ble_manufacturer_data_advertise:
        - mac_address: 11:22:33:44:55:66
          manufacturer_id: 0590
          then:
            - lambda: |-
                if (x[0] != 0x7b || x[1] != 0x61) return;
                int value = x[2] + (x[3] << 8);
                id(ble_sensor).publish_state(value);

Configuration variables:

- **mac_address** (*Optional*, MAC Address): The MAC address to filter for this automation.
- **manufacturer_id** (**Required**, string): 16 bit, 32 bit, or 128 bit BLE Manufacturer ID.
- See :ref:`Automation <automation>`.

.. _esp32_ble_tracker-on_ble_service_data_advertise:

``on_ble_service_data_advertise`` Trigger
************************************************

This automation will be triggered when a Bluetooth advertising with service data is received. A
variable ``x`` of type ``std::vector<uint8_t>`` is passed to the automation for use in lambdas.

.. code-block:: yaml

    sensor:
      - platform: template
        name: "BLE Sensor"
        id: ble_sensor

    esp32_ble_tracker:
      on_ble_service_data_advertise:
        - mac_address: 11:22:33:44:55:66
          service_uuid: 181A
          then:
            - lambda: 'id(ble_sensor).publish_state(x[0]);'

Configuration variables:

- **mac_address** (*Optional*, MAC Address): The MAC address to filter for this automation.
- **service_uuid** (**Required**, string): 16 bit, 32 bit, or 128 bit BLE Service UUID.
- See :ref:`Automation <automation>`.

``on_scan_end`` Trigger
************************************************

This automation will be triggered when a Bluetooth scanning sequence has completed. If running
with continuous set to true, this will trigger every time the scan completes (the duration of
a scan).

.. code-block:: yaml

    esp32_ble_tracker:
      on_scan_end:
        - then:
            - lambda: |-
                 ESP_LOGD("ble_auto", "The scan has ended!");

Configuration variables:

- None

- See :ref:`Automation <automation>`.

``esp32_ble_tracker.start_scan`` Action
************************************************

Start a Bluetooth scan. If there is a scan already in progress, then the action is ignored.

.. code-block:: yaml

    esp32_ble_tracker:
      scan_parameters:
        continuous: false

    on_...:
      - esp32_ble_tracker.start_scan:

Configuration variables:

- **continuous** (*Optional*, boolean): Whether to start the scan in continuous mode. Defaults to ``false``

.. note::

    This action can also be written in :ref:`lambdas <config-lambda>`:
.. code-block:: yaml

    esp32_ble_tracker:
      id: ble_tracker_id

.. code-block:: cpp

    id(ble_tracker_id).start_scan()

``esp32_ble_tracker.stop_scan`` Action
************************************************

Stops the bluetooth scanning. It can be started again with the above start scan action.

.. code-block:: yaml

    esp32_ble_tracker:

    on_...:
      - esp32_ble_tracker.stop_scan:

See Also
--------

- :doc:`text_sensor/ble_scanner`
- :doc:`sensor/ble_rssi`
- :doc:`sensor/b_parasite`
- :doc:`sensor/xiaomi_ble`
- :doc:`sensor/xiaomi_miscale`
- :doc:`sensor/inkbird_ibsth1_mini`
- :doc:`sensor/mopeka_pro_check`
- :doc:`sensor/ruuvitag`
- :doc:`ble_client`
- :doc:`bluetooth_proxy`
- :apiref:`esp32_ble_tracker/esp32_ble_tracker.h`
- `ESP32 BLE for Arduino <https://github.com/nkolban/ESP32_BLE_Arduino>`__ by `Neil Kolban <https://github.com/nkolban>`__.
- :ghedit:`Edit`
