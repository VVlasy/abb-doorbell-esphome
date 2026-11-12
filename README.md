# ABB Doorbell – ESPHome firmware for the ABB Welcome / Busch-Welcome ESP32 bus interface

This is the ESPHome configuration pre-installed on the assembled boards sold by VVlasy.
The hardware is Mat931's open-source [esp32-doorbell-bus-interface](https://github.com/Mat931/esp32-doorbell-bus-interface) (CERN-OHL-W v2).

- **Quick Start Guide** (wiring, Wi-Fi, Home Assistant, finding your bus addresses): [docs/ABB-Doorbell-Quick-Start-Guide.pdf](docs/ABB-Doorbell-Quick-Start-Guide.pdf)
- **How the bus works and how to find your addresses:** [how_to_customize.md](https://github.com/Mat931/esp32-doorbell-bus-interface/blob/main/how_to_customize.md) in the hardware project

## Setting up a board

1. Connect the board to the 2-wire bus (20–30 V DC). It opens an open Wi-Fi hotspot named `abb-doorbell-xxxxxx`.
2. Join the hotspot, enter your 2.4 GHz Wi-Fi in the captive portal (or browse to http://192.168.4.1).
3. Home Assistant discovers the device under **Settings → Devices & services**.
4. In the **ESPHome Device Builder** add-on the device appears as discovered. Click **Adopt**, then **Install**.
   Adoption creates a small config that pulls `abb-doorbell.yaml` from this repository as a package and adds an API encryption key.
5. Edit the adopted config and put your own bus values under `substitutions:`, then **Install → Wirelessly**.

```yaml
substitutions:
  name: abb-doorbell-xxxxxx
  friendly_name: ABB Doorbell xxxxxx
  indoor_station_address: "0x000110"
  outdoor_station_address: "0x000201"
  door_address: "0x000401"
  three_byte_address: "true"
  door_open_type: "0x0e"
  door_open_reply_type: "0x8e"
  door_open_code: "0x00, 0x00, 0x00"
```

Until the substitutions match your installation, the doorbell sensors may stay silent and *Front Door* / *Stop Doorbell* do nothing: the shipped values are placeholders.

## Substitutions

| Substitution | Default | Meaning |
| --- | --- | --- |
| `indoor_station_address` | `"0x1001"` | Your indoor station, the most frequent address in the log |
| `outdoor_station_address` | `"0x2001"` | Source of the `Type: 01` message when the bell rings |
| `door_address` | `"0x4001"` | Destination of the door-opener message, source of its reply |
| `three_byte_address` | `"false"` | `"true"` when addresses show 6 hex digits |
| `door_open_type` | `"0x0d"` | Door-opener message type, `0x0d` or `0x0e` |
| `door_open_reply_type` | `"0x8d"` | The door's reply, `0x8d` or `0x8e` |
| `door_open_code` | `"0xab, 0xcd, 0xef"` | `Data` bytes of the door-opener message, comma separated |
| `unlock_time` | `"3s"` | How long the lock entity shows *Unlocking* |
| `doorbell_outdoor_type` | `"0x01"` | Outdoor → indoor station message while ringing |
| `doorbell_indoor_type` | `"0x11"` | Sent by the indoor station for the floor bell |
| `end_call_type` | `"0x02"` | Ends the call, used by *Stop Doorbell* |

## Entities

Doorbell outdoor and Doorbell indoor (binary sensors), Front Door (lock), Stop Doorbell (button), Doorbell Intercom (text sensor with the last decoded bus message), Internal Temperature, Reboot and Reboot (Safe Mode).

## Without adoption

Copy `abb-doorbell.yaml` into your ESPHome dashboard, set `name:` to the full device name (`abb-doorbell-xxxxxx`), delete `name_add_mac_suffix: true`, edit the substitutions and install wirelessly.

## Factory flash (seller)

```
esphome compile abb-doorbell.yaml
esptool --chip esp32 --port COMx erase-flash
esptool --chip esp32 --port COMx write-flash 0x0 .esphome/build/abb-doorbell/.pioenvs/abb-doorbell/firmware.factory.bin
```

## Credits

Hardware design and bus protocol work by [Mat931](https://github.com/Mat931/esp32-doorbell-bus-interface), licensed CERN-OHL-W v2. Firmware built on [ESPHome](https://esphome.io). ABB, ABB Welcome and Busch-Welcome are trademarks of their owners; this project is not affiliated with ABB.
