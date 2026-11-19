# Waveshare ESP32-S3-ETH Ethernet support

This version of RadPro WiFi Bridge supports the Waveshare ESP32-S3-ETH board with its onboard W5500 Ethernet controller.

## Network behavior

Ethernet is the preferred network interface:

1. The W5500 is initialized at boot.
2. The firmware waits briefly for Ethernet/DHCP.
3. If Ethernet has an IP address, Wi-Fi is switched completely off.
4. The existing RadPro web interface is served from the Ethernet IP on port 80.
5. MQTT, OpenSenseMap, GMCMap, Radmon, OpenRadiation, Safecast, time synchronization and OTA use the normal ESP32 network stack and therefore work over Ethernet.
6. If Ethernet is unavailable, the existing Wi-FiManager behavior is retained, including saved Wi-Fi credentials and the setup AP.
7. If Ethernet is connected later, the firmware switches to Ethernet and turns Wi-Fi off.
8. If Ethernet is subsequently lost, Wi-Fi can be started again as a fallback.

## W5500 pin mapping

The Waveshare ESP32-S3-ETH uses:

| W5500 | ESP32-S3 GPIO |
|---|---:|
| SCLK | 13 |
| MISO | 12 |
| MOSI | 11 |
| CS | 14 |
| RESET | 9 |
| INT | 10 |

The onboard RGB LED is GPIO21.

## Ethernet web interface

With Ethernet active, browse to:

`http://<Ethernet-IP>/`

The same configuration pages are available as with Wi-Fi. The Ethernet server uses the same portal handlers, filesystem assets, CSRF checks, OTA upload handlers and publisher configuration pages.

The Wi-Fi setup page itself is intentionally not exposed over Ethernet, because Ethernet being active is the condition that disables Wi-Fi.

## DHCP and fallback timing

The current implementation uses DHCP for Ethernet. A DHCP lease must be available for the bridge to be considered connected.

At boot, Wi-Fi is left uninitialized and off while the W5500 is checked. If no physical Ethernet link appears within about 3 seconds, Wi-Fi fallback starts. If link is present, the firmware allows up to about 12 seconds for DHCP. The Wi-Fi setup portal runs non-blocking on this board so a late Ethernet DHCP lease can still take over and shut Wi-Fi back off.

## PlatformIO

A dedicated `waveshare-esp32-s3-eth` board definition is included. It uses the same ESP32-S3R8/16-MB flash and 8-MB OPI PSRAM settings as the compatible DevKitC-1 target. The build enables `RADPRO_WAVESHARE_ESP32_S3_ETH`, which also selects the correct RGB LED GPIO.

The firmware uses pioarduino `51.03.07` (Arduino-ESP32 3.0.7 / ESP-IDF 5.1.4). Arduino-ESP32 3.x is required for the built-in W5500 `ETH.begin(...)` API. The ESP-IDF W5500 component is explicitly enabled in `sdkconfig.defaults`; without `CONFIG_ETH_SPI_ETHERNET_W5500=y`, `ETH_PHY_W5500` is not exposed by `ETH.h` and the project will fail to compile.

After changing platform/core or sdkconfig settings, do a clean PlatformIO build so stale generated SDK configuration is not reused.
