WiFi Component
==============

.. seo::
    :description: Instructions for setting up the WiFi configuration for your ESP node in ESPHome.
    :image: network-wifi.svg
    :keywords: WiFi, WLAN, ESP8266, ESP32

This core ESPHome component sets up WiFi connections to access points
for you. You need to have a network configuration (either Wifi or Ethernet)
or ESPHome will fail in the config validation stage. You also can't have both Wifi
and Ethernet setup in same time (even if your ESP has both wired).

It’s recommended to provide a static IP for your node, as it can
dramatically improve connection times.

.. code-block:: yaml

    # Example configuration entry
    wifi:
      ssid: MyHomeNetwork
      password: VerySafePassword

      # Optional manual IP
      manual_ip:
        static_ip: 192.168.0.123
        gateway: 192.168.0.1
        subnet: 255.255.255.0

.. code-block:: yaml

    # It is highly recommended to use secrets
    wifi:
      ssid: !secret wifi_ssid
      password: !secret wifi_password

.. _wifi-configuration_variables:

Configuration variables:
------------------------

- **ssid** (*Optional*, string): The name (or `service set
  identifier <https://www.lifewire.com/definition-of-service-set-identifier-816547>`__)
  of the WiFi access point your device should connect to.
- **password** (*Optional*, string): The password (or PSK) for your
  WiFi network. Leave empty for no password.
- **networks** (*Optional*): Configure multiple WiFi networks to connect to, the best one
  that is reachable will be connected to. See :ref:`wifi-networks`.
- **manual_ip** (*Optional*): Manually configure the static IP of the node.

  - **static_ip** (**Required**, IPv4 address): The static IP of your node.
  - **gateway** (**Required**, IPv4 address): The gateway of the local network.
  - **subnet** (**Required**, IPv4 address): The subnet of the local network.
  - **dns1** (*Optional*, IPv4 address): The main DNS server to use.
  - **dns2** (*Optional*, IPv4 address): The backup DNS server to use.

- **use_address** (*Optional*, string): Manually override what address to use to connect
  to the ESP. Defaults to auto-generated value. Example, if you have changed your static IP and want to flash OTA to the previously configured IP address.

- **ap** (*Optional*): Enable an access point mode on the node.

  - **ssid** (*Optional*, string): The name of the access point to create. Leave empty to use
    the device name.
  - **password** (*Optional*, string): The password for the access point. Leave empty for
    no password.
  - **channel** (*Optional*, int): The channel the AP should operate on from 1 to 14.
    Defaults to 1.
  - **manual_ip** (*Optional*): Manually set the IP options for the AP. Same options as
    manual_ip for station mode.
  - **ap_timeout** (*Optional*, :ref:`config-time`): The time after which to enable the
    configured fallback hotspot. Defaults to ``1min``.

- **domain** (*Optional*, string): Set the domain of the node hostname used for uploading.
  For example, if it's set to ``.local``, all uploads will be sent to ``<HOSTNAME>.local``.
  Defaults to ``.local``.
- **reboot_timeout** (*Optional*, :ref:`config-time`): The amount of time to wait before rebooting when no
  WiFi connection exists. Can be disabled by setting this to ``0s``, but note that the low level IP stack currently
  seems to have issues with WiFi where a full reboot is required to get the interface back working. Defaults to ``15min``.
- **power_save_mode** (*Optional*, string): The power save mode for the WiFi interface.
  See :ref:`wifi-power_save_mode`

- **output_power** (*Optional*, string): The amount of TX power for the WiFi interface from 8.5dB to 20.5dB. Default for ESP8266 is 20dB, 20.5dB might cause unexpected restarts.
- **fast_connect** (*Optional*, boolean): If enabled, directly connects to WiFi network without doing a full scan
  first. This is required for hidden networks and can significantly improve connection times. Defaults to ``off``.
  The downside is that this option connects to the first network the ESP sees, even if that network is very far away and
  better ones are available.
- **passive_scan** (*Optional*, boolean): If enabled, then the device will perform WiFi scans in a passive fashion. Defaults to ``false``.

- **enable_btm** (*Optional*, bool): Only on ``esp32`` with ``esp-idf``. Enable 802.11v BSS Transition Management support.
- **enable_rrm** (*Optional*, bool): Only on ``esp32`` with ``esp-idf``. Enable 802.11k Radio Resource Management support.

- **enable_on_boot** (*Optional*, boolean): If enabled, the WiFi interface will be enabled on boot. Defaults to ``true``.

- **id** (*Optional*, :ref:`config-id`): Manually specify the ID used for code generation.

Access Point Mode
-----------------

ESPHome has an optional "Access Point Mode". If you include ``ap:``
in your wifi configuration, ESPHome will automatically set up an access point that you
can connect to. Additionally, you can specify both a "normal" station mode and AP mode at the
same time. This will cause ESPHome to only enable the access point when no connection
to the WiFi router can be made.

.. code-block:: yaml

    wifi:
      ap:
        ssid: "Livingroom Fallback Hotspot"
        password: "W1PBGyrokfLz"

You can also create a simple ``ap`` config which will set up the access point to have the
devices name as the ssid with no password.

.. code-block:: yaml

    wifi:
      ap: {}

    # or if you still want the ap to have a password

    wifi:
      ap:
        password: "W1PBGyrokfLz"

User Entered Credentials
------------------------

Some components such as :doc:`captive_portal`, :doc:`improv_serial` and :doc:`esp32_improv`
enable the user to send and save Wi-Fi credentials to the device. Beginning in 2022.11.0,
as long as no credentials are set in the config file, and firmware is uploaded without erasing
the flash (via OTA), the device will keep the saved credentials.


.. _wifi-manual_ip:

Manual IPs
----------

If you're having problems with your node not connecting to WiFi or the connection
process taking a long time, it can be a good idea to assign a static IP address
to the ESP. This way, the ESP doesn't need to go through the slow DHCP process.

You can do so with the ``manual_ip:`` option in the WiFi configuration.

.. code-block:: yaml

    wifi:
      # ...
      manual_ip:
        # Set this to the IP of the ESP
        static_ip: 10.0.0.42
        # Set this to the IP address of the router. Often ends with .1
        gateway: 10.0.0.1
        # The subnet of the network. 255.255.255.0 works for most home networks.
        subnet: 255.255.255.0

After putting a manual IP in your configuration, the ESP will no longer need to negotiate
a dynamic IP address with the router, thus improving the time until connection.

Additionally, this can help with :doc:`Over-The-Air updates <ota>` if for example the
home network doesn't allow for ``.local`` addresses. When a manual IP is in your configuration,
the OTA process will automatically choose that as the target for the upload.

.. note::

    See also :ref:`esphome-changing_node_name`.

.. _wifi-power_save_mode:

Power Save Mode
---------------

The WiFi interface of all ESPs offer three power save modes to reduce the amount of power spent on
WiFi. While some options *can* reduce the power usage of the ESP, they generally also decrease the
reliability of the WiFi connection, with frequent disconnections from the router in the highest
power saving mode.

- ``NONE`` (least power saving, Default for ESP8266)
- ``LIGHT`` (Default for ESP32)
- ``HIGH`` (most power saving)

.. code-block:: yaml

    wifi:
      # ...
      power_save_mode: none

.. _wifi-networks:

Connecting to Multiple Networks
-------------------------------

Starting with version 1.10.0, you can give ESPHome a number of WiFi networks to connect to.
ESPHome will then attempt to connect to the one with the highest signal strength.

To enable this mode, remove the ``ssid`` and ``password`` options from your wifi configuration
and move everything under the ``networks`` key:

.. code-block:: yaml

    # Example configuration entry
    wifi:
      networks:
      - ssid: FirstNetworkToConnectTo
        password: VerySafePassword
      - ssid: SecondNetworkToConnectTo
        password: VerySafePassword
      # Other options
      # ...

Configuration variables:

- **ssid** (*Optional*, string): The SSID or WiFi network name.
- **password** (*Optional*, string): The password to use for authentication. Leave empty for no password.
- **manual_ip** (*Optional*): Manually configure the static IP of the node when using this network. Note that
  when using different static IP addresses on each network, it is required to set ``use_address``, as ESPHome
  cannot infer to which network the node is connected.

  - **static_ip** (**Required**, IPv4 address): The static IP of your node.
  - **gateway** (**Required**, IPv4 address): The gateway of the local network.
  - **subnet** (**Required**, IPv4 address): The subnet of the local network.
  - **dns1** (*Optional*, IPv4 address): The main DNS server to use.
  - **dns2** (*Optional*, IPv4 address): The backup DNS server to use.

- **eap** (*Optional*): See :ref:`eap`.
- **channel** (*Optional*, int): The channel of the network (1-14). If given, only connects to networks
  that are on this channel.
- **bssid** (*Optional*, string): The connection's BSSID (MAC address). BSSIDs must consist of six
  two-digit hexadecimal values separated by colon characters ("``:``"). All letters must be in upper case.
- **hidden** (*Optional*, boolean): Whether this network is hidden. Defaults to false.
  If you add this option you also have to specify ssid.
- **priority** (*Optional*, float): The priority of this network. After each time, the network with
  the highest priority is chosen. If the connection fails, the priority is decreased by one.
  Defaults to ``0``.

.. _eap:

Enterprise Authentication
-------------------------

WPA2_EAP Enterprise Authentication is supported on ESP32s and ESP8266s.
In order to configure this feature you must use the :ref:`wifi-networks` style configuration.
The ESP32 is known to work with PEAP, EAP-TTLS, and the certificate based EAP-TLS.
These are advanced settings and you will usually need to consult your enterprise network administrator.

.. code-block:: yaml

    # Example EAP configuration
    wifi:
      networks:
      - ssid: EAP-TTLS_EnterpriseNetwork
        eap:
          username: bob
          password: VerySafePassword
      - ssid: EAP-TLS_EnterpriseNetwork
        eap:
          identity: bob
          certificate_authority: ca_cert.pem
          certificate: cert.pem
          key: key.pem

Configuration variables:

- **identity** (*Optional*, string): The outer identity to pass to the EAP authentication server.
  This is required for EAP-TLS.
- **username** (*Optional*, string): The username to present to the authenticating server.
- **password** (*Optional*, string): The password to present to the authentication server.
  For EAP-TLS this password may be set to decrypt to private key instead.
- **certificate_authority** (*Optional*, string): Path to a PEM encoded certificate to use when validating the authentication server.
- **certificate** (*Optional*, string): Path to a PEM encoded certificate to use for EAP-TLS authentication.
- **key** (*Optional*, string): Path to a PEM encoded private key matching ``certificate`` for EAP-TLS authentication.
  Optionally encrypted with ``password``.


``wifi.disable`` Action
-----------------------

This action turns off the WiFi interface on demand.

.. code-block:: yaml

    on_...:
      then:
        - wifi.disable:

.. note::

    Be aware that if you disable WiFi, the API timeout will need to be disabled otherwise the device will reboot.


``wifi.enable`` Action
----------------------

This action turns on the WiFi interface on demand.

.. code-block:: yaml

    on_...:
      then:
        - wifi.enable:

.. note::

    The configuration option ``enable_on_boot`` can be set to ``false`` if you do not want wifi to be enabled on boot.


.. _wifi-connected_condition:

``wifi.connected`` Condition
----------------------------

This :ref:`Condition <config-condition>` checks if the WiFi client is currently connected to a station.

.. code-block:: yaml

    on_...:
      if:
        condition:
          wifi.connected:
        then:
          - logger.log: WiFi is connected!


The lambda equivalent for this is ``id(wifi_id).is_connected()``.


.. _wifi-enabled_condition:

``wifi.enabled`` Condition
--------------------------

This :ref:`Condition <config-condition>` checks if WiFi is currently enabled or not.

.. code-block:: yaml

    on_...:
      - if:
          condition: wifi.enabled
          then:
            - wifi.disable:
          else:
            - wifi.enable:


The lambda equivalent for this is ``!id(wifi_id).is_disabled()``.


See Also
--------

- :doc:`captive_portal`
- :doc:`text_sensor/wifi_info`
- :doc:`sensor/wifi_signal`
- :doc:`network`
- :doc:`/components/ethernet`
- :doc:`api`
- :apiref:`wifi/wifi_component.h`
- :ghedit:`Edit`
