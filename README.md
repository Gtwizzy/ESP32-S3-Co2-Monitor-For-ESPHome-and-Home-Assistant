# CO₂ Sensor Display (ESPHome)

CO₂ monitor built on the back of a Waveshare ESP32-S3-LCD-1.28" (non-touch) board and using a Sensirion SCD41 sensor.  
Built direclty inside of home assistant using ESPHome and displays real-time CO₂ levels on its circular LCD screen in a calm, intuitive way.

---

## Hardware Used

| Component | Model / Link |
|------------|---------------|
| Microcontroller + Display | https://www.waveshare.com/wiki/ESP32-S3-LCD-1.28 |
| CO₂ Sensor | https://www.adafruit.com/product/5190 |
| Cable | JST-SH (STEMMA QT / Qwiic) for I²C connection |
| Power | USB-C 5 V |

⚠️ Important: This project is built for the **non-touch** version of the Waveshare ESP32-S3-LCD-1.28.  
The touch variant uses a different pinout and will not work without modifying the YAML configuration.

---

## Setup (Home Assistant + ESPHome Add-on)

Before you begin, complete the **required assets** step. If this step is skipped, the first build will fail.

### 1) Create Required Folders & Add Assets (fonts + images)

This project references image and font assets during compile. These folders do **not** exist by default inside of home assistant and must be created manually inside your Home Assistant configuration directory:

config/esphome/
  ├── co2_sensor/
      ├── fonts/
      ├── images/

Copy the provided **font files** into the `fonts` folder and all required **image files** (background and animation frames) into the `images` folder **before the first install**.  
If these folders or files are missing, ESPHome cannot package the assets into flash and the build will fail.

### 2) Add the Device in ESPHome

1. Open the **ESPHome** add-on in Home Assistant.  
2. Create a new device. If you want to use the prebuilt yaml provided here you'll need to namegive the device the exact name below as the ESPHome wizzard ties the device name permenantly to the device and can't be changed after the fact without completely reflashing a fresh install. If you choose to use my code you'll need to **name it exactly**:

   co2-sensor

Alternatively you can alter the yaml name to match what you chose for device name of your ESP board 
3. If you connect your board to the computer you intend to work on this project from and run the setup wizzard ESPHome will pick up all the board settings correctly. Otherwise step through this section manually and enter the matching information for the device you're using. Once the default information is flashed to the device click edit and you should see some information regarding your Home Assistant API key and OTA password. It's not imperative that you use these in your own device but the template code does allow for them and it's easier to just copy and paste these somewhere safe so that you can use them in the template later rather than having to got through the process of comissioning these from home assistant etc.
4. If you've chosen to go with the same waveshare board I did you can simply open the newly created device and replace all default code with code found in the template .yaml file provided 
5. Update the Wifi SSID with your own network information (if not using a secrets file). And add in the API key and OTA password you copied earlier and place them in the alloted areas.
6. Confirm the `fonts` and `images` folders (and their files) are in the correct folders on your home assistant machine as shown above.  
7. Click **Install** and choose **Plug into this computer** (or OTA if already flashed).  
8. When the device comes online, you can add it to Home Assistant through the ESPHome integration by inputing the.

---

## Display Philosophy

Although the SCD41 also reports **temperature** and **humidity**, those values are **not shown on the device screen**.  
This display is designed for **CO₂ at a glance** from a moderate distance. Adding more on-screen data reduced instant readability.  
Temperature and humidity are still published to Home Assistant as entities, so you can use them freely in dashboards and automations.

---

## Air Quality Threshold Model

The sensor translates CO₂ levels into clear visual “moods” represented by a stylised lily that opens or closes depending on air quality.

| Threshold | Name | Approx. CO₂ Range (ppm)* | Visual State |
|------------|------|---------------------------|---------------|
| **T1** | Soft & Still | 400–600 ppm | Lily fully open / soft pale blue background |
| **T2** | Feeling Fine | 601–900 ppm | Lily gently closing / calm green background |
| **T3** | Time to Refresh | 901–1200 ppm | Lily mostly closed / amber background |
| **T4** | Bring the Outside In | >1200 ppm | Lily tightly closed / red background |

\* Exact ranges can be refined once your individual sensor stabilises and self-calibrates in its final environment.

---

## Wake and Sleep Behaviour

The display is event-driven — it **wakes when something meaningful changes**, then returns to quiet mode.

| Event / Condition | Behaviour |
|-------------------|-----------|
| **Startup** | Wakes for ~30s to show current CO₂ ppm, then sleeps. |
| **Crossing a threshold (up or down)** | Wakes for ~15s with the new background colour and ppm value, then sleeps. |
| **No improvement for >30 minutes within the same threshold** | Brief reminder wake, then sleeps. |
| **CO₂ rising steadily (+30 ppm/reading)** | Normal 30-minute reminder cadence. |
| **CO₂ falling steadily (−30 ppm/reading)** | “Sleep-on-actioned” — stays quiet until the trend flattens (~10 min) or drops to the next lower threshold. |
| **Flat trend ≥10 min while still in poor air (>T3 minimum)** | Gentle reminder wake. |
| **Recovered (<T1 upper limit)** | Brief positive wake (current colour + ppm), then sleep. |

**Hysteresis:** ±20 ppm around each boundary to prevent flicker.  
**Smoothing:** 15 s rolling average to ignore brief 1–2 s spikes.

---

## Entities Exposed to Home Assistant

### Sensors
- `sensor.co2_sensor_co2` — CO₂ in ppm (primary signal).  
- `sensor.co2_sensor_temperature` — ambient temperature (not shown on device screen).  
- `sensor.co2_sensor_humidity` — relative humidity (not shown on device screen).

### Switches (User Controls)
- `switch.co2_sensor_co2_sensor_display_always_on`  
  Keeps the screen on continuously so the current CO₂ reading is always visible. The lily animation still plays on threshold changes.  
  *Why it’s optional:* TFT panels can show image retention with long-term static content. This is a **precautionary note** — this specific panel has **not** shown issues in testing, but the option exists to minimise any risk.

- `switch.co2_sensor_co2_sensor_display_wake`  
  Manually triggers the “wake” behaviour on demand. Use this when the display is asleep but you want an immediate visual update (current colour + ppm).

- `switch.co2_sensor_co2_sensor_sleep_mode`  
  Forces the screen to stay off and disables wake behaviour. Handy for scheduled quiet hours (e.g., evenings in a bedroom).

These switches let you choose between a subtle, ambient experience or a more persistent visual readout depending on room, lighting, and personal preference.

---

## Storage and Build Notes

This project runs close to the ESP32-S3’s flash limit — the current firmware uses roughly **96% of available flash**.  
If you plan to make changes (e.g., add frames or higher-resolution assets), keep storage in mind. If a compile fails, you may have exceeded flash size.

**How it fits:** all animation frames are stored as **grayscale** images for compact storage. At runtime, ESPHome tints them in code (e.g., via pixel draw operations) to achieve colour.  
This keeps visuals expressive while dramatically reducing file size so the project builds reliably.

---

## Folder Layout

CO2-Sensor-Display/  
├── config/esphome/co2_sensor/  
│   ├── Template_v2.yaml  
│   ├── fonts/  
│   ├── images/  
│   └── README.md  

---

## License

Released under the MIT License.

