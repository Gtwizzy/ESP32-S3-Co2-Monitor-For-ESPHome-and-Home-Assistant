# CO₂ Sensor Display (ESPHome)

CO₂ monitor built on the back of a Waveshare ESP32-S3-LCD-1.28" (non-touch) board and using a Sensirion SCD41 sensor.  
Built direclty inside of home assistant using ESPHome and displays real-time CO₂ levels on its circular LCD screen in a calm, intuitive way.

<h4> I've ***purposefully*** not not added any referal purchase links into this git beacuse I just don't believe in Amazon or whoever else having one more data point on you when it comes to buying things.
But if you found this helpful or got use out of it I'd your support is always appreciated and you can "buy me a coffee" below.
  <br><br>
<a href="https://www.buymeacoffee.com/gtwizzy">
  <img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" width="200" />
</a>



<h2>⚠️ Huge Caveates ⚠️<h2></h2>
  
This project started life with a plan to build a full firmware in C++ with data being pushed to HA via MQTT as that is where I felt my skills were in getting this working in the exact way I wanted with the clean interface and animations I wanted while being lean on system resources. However! After I was given a significant humbling at just how much by C coding skills have dropped off after too may years of not using them to their fullest, I pivoted this into being a ESPHome device build. 
  
The only reason I didn't go this way to begin with was I had grander visions for the UI and animations etc and ESPHome does not natively support a lot of the needs I had out of the box just yet (at least not within the flash constrains of this device). AND I was less assured of my ability to make my logic work in .yaml due to me just being less expereienced at coding in .yaml for this kind of project. SO, it was built in 36 hours (originally I gave myself a 4 day long weekend to get it working the way I wanted and burned the first 2.5 days fighting my loosing battle with C) So byt the time I pivoted to ESPHome I had significantly compromised my inital ideas AND I ***had*** to rely on AI to help me get the logic just right and functioning. So I am **CONFIDENT** there are inefficiencies in this code. And I am **CONFIDENT** it could probably have been written better.

But this was designed for personal use so I will work on it in moments of spare time to see if I can improve this foundation. But please do not expect this project to be maintained in ANY way, or for me to even be able to provide significant support because for some of this template logic I am still coming to grips with the actual layout of it myself. I am happy for you to fork and work on it for yourself and feedback your improvements to me I would in fact ***LOVE*** that. But I just don't have the time to maintain this ongoing or try to promise reliable updates.

For now it works the way I want and I'll address the other pieces I'm still working on, when I can.

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
  <br>&nbsp;&nbsp;├── co2_sensor/
      <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── fonts/
      <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── images/

Copy the provided **font files** into the `fonts` folder and all required **image files** (background and animation frames) into the `images` folder **before the first install**.  

***If these folders or files are missing, ESPHome cannot package the assets into flash and the build will fail.***

### 2) Add the Device in ESPHome

1. Open the **ESPHome** add-on in Home Assistant.  
2. Create a new device. If you want to use the prebuilt yaml provided here you'll need to name the device the exact name below as the ESPHome wizzard ties the device name permenantly to the device and (to the best of my knowledge) this can't be changed after the fact without completely reflashing a fresh install. If you choose to use my code in a copy and paste way you'll need to **name it exactly**:

   "**co2-sensor**"

Alternatively you can alter the yaml name to match what you chose for device name of your ESP board 

### 3) ESPHome First install

If you connected your board to the computer you intend to work on this project from and run the setup wizzard ESPHome will pick up all the board settings correctly. Otherwise step through this section manually and enter the matching information for the device you're using. Once the default information is flashed to the device click edit and you should see some information regarding your Home Assistant API key and OTA password. It's not imperative that you use these in your own device but the template code does allow for them and it's easier to just copy and paste these somewhere safe so that you can use them in the template later rather than having to got through the process of comissioning these from home assistant etc.

### 4) Copy and Paste template.yaml

If you've chosen to go with the same waveshare board I did you can simply open the newly created device and replace all default code with code found in the template .yaml file provided 

### 4a) Update the code

Update the Wifi SSID with your own network information (if not using a secrets file). And add in the API key and OTA password you copied earlier and place them in the alloted areas if you're going to use them.

### 4b) Double check your folders

Confirm the `fonts` and `images` folders (and their files) are in the correct folders on your home assistant machine as shown above.  

### 5) Run the your install

Click **Install** and choose **Plug into this computer** (or OTA if already flashed).  

### 6) Add the device to Home Assistant

When the device comes online, you can add it to Home Assistant through the ESPHome integration by inputing the ip address of the device as it appears on your network.

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
| **T1** | Soft & Still | 400–600 ppm | Lily fully open / soft pale blue background with blue text|
| **T2** | Feeling Fine | 601–900 ppm | Lily gently closing / calm green background with green text|
| **T3** | Time to Refresh | 901–1200 ppm | Lily mostly closed / amber background with yellow text|
| **T4** | Bring the Outside In | >1200 ppm | Lily tightly closed / deep orange background with purple text|

\* Exact ranges can be refined once your individual sensor stabilises and self-calibrates in its final environment. The SCD41 is designed for self calibration right out of the box.

---

## Wake and Sleep Behaviour

The display is event-driven — it **wakes when something meaningful changes**, then returns to quiet mode. The plan for this longer term is for it to be able to register when the user has done something to initate a positive downward change in the Co2 reading and have the display only wake again when it crosses into a lower threshold or if the decrease in co2 levels plateaus for a 10 min period and then therefore wakes to let the user know that things haven't improved significantly. 

| Event / Condition | Behaviour | Status |
|-------------------|-----------|--------|
| **Startup** | Wakes with brief animation for ~30s to show current CO₂ ppm and zone with associated background and colours, then sleeps. |
| **Crossing a threshold (up or down)** | Wakes with brief animation shows new zone information for ~15s with the new background colour and ppm value displaying after animation, then sleeps. |
| **No improvement for >30 minutes within the same threshold** | Brief reminder wake, then sleeps. | this functionality is still not complete but I am hoping to fix this in later edits| itterations}
| **CO₂ rising steadily (+30 ppm/reading)** | Normal 30-minute reminder cadence. | as above I'm still dialing this in correctly so as of right now it's not functional |
| **CO₂ falling steadily (−30 ppm/reading)** | “Sleep-on-actioned” — stays quiet until the trend flattens (~10 min) or drops to the next lower threshold. | reviewing the logs this seems to be working MOST of the time but still requires fine tuning |
| **Flat trend ≥10 min while still in poor air (>T3 minimum)** | Gentle reminder wake. | WIP |
| **Recovered (<T1 upper limit)** | Brief positive wake (current colour + ppm), then sleep. |

**Hysteresis/Smoothed oscilation logic:** ±20 ppm around each boundary to prevent screen wake/sleep flicker.  
**Smoothing:** 15 s rolling average to ignore brief 1–2 s spikes.

---

## Entities Exposed to Home Assistant

### Sensors
- `sensor.co2_sensor_co2` — CO₂ in ppm (primary signal).  
- `sensor.co2_sensor_temperature` — ambient temperature (not shown on device screen).  
- `sensor.co2_sensor_humidity` — relative humidity (not shown on device screen).
- `sensor.co2_sensor_co2_air_quality_stage` — shown on the device UI as a baked in element to the background images but also a nice simple way to run automations off the back of in order to create automations like *if* sensor.co2_sensor_air_quality_stage = "time to refresh" *then* Open kitchen window. This way you don't have to mess around with specific and exact Co2 reading numbers if you just want a blanket "hey when its this — do this; but when its that — do this"

### Switches (User Controls)
- `switch.co2_sensor_co2_sensor_display_always_on`  
  Keeps the screen on continuously so the current CO₂ reading is always visible. The lily animation still plays on threshold changes.  
  *Why it’s optional:* TFT panels can show image retention with long-term static content. This is a **precautionary note** — this specific panel has **not** shown issues in testing, but the option exists to minimise any risk.

- `switch.co2_sensor_co2_sensor_display_wake`  
  Manually triggers the “wake” behaviour on demand. Use this when the display is asleep but you want an immediate visual update (current zone + ppm) acn be useful for voice assistant automations that you want to use to trigger a wake up of the display "*hey Jarvis what is the air feeling like in here*".

- `switch.co2_sensor_co2_sensor_sleep_mode`  
  Forces the screen to stay off and disables wake behaviour. Handy for scheduled quiet hours (e.g., evenings in a bedroom).

These switches let you choose between a subtle, ambient experience or a more persistent visual readout depending on room, lighting, and personal preference.

---

## Storage and Build Notes

This project runs close to the ESP32-S3’s flash limit — the current firmware uses roughly **96% of available flash** *(not ideal but fk it for now it works **(≧∇≦)ﾉ** )*.  
If you plan to make changes (e.g., add frames or colour-resolution assets), keep storage in mind. If a compile fails, you may have exceeded flash size.

**How it fits:** all animation frames are stored as **grayscale** images for compact storage. At runtime, ESPHome tints them in code (e.g., via pixel draw operations) to achieve colour.  
This keeps visuals expressive while dramatically reducing file size so the project builds reliably.

---

## Folder Layout

Below is where you can find each of the necessary files needed for the instructions above.

CO2-Sensor-Display/  
├── main/    
│   ├── fonts/  
│   ├── images/
│   ├── device images/
│   ├── Template.yaml
│   └── README.md  

---



## License

Released under the MIT License.

