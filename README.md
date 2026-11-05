# E-Ink Weather Quotes Display

A battery-powered ESP32 e-ink display that shows current weather conditions alongside literary quotes from classic literature. Built with ESPHome for seamless Home Assistant integration.

![Display Size](https://img.shields.io/badge/Display-7.5%22%20E--Ink-black)
![Platform](https://img.shields.io/badge/Platform-ESP32--S3-blue)
![Quotes](https://img.shields.io/badge/Quotes-1400%2B-green)
![ESPHome](https://img.shields.io/badge/ESPHome-Compatible-orange)

## Features

- **1,400+ Real Literary Quotes** - 100+ per weather condition, each checked word for word against its source text
- **Adaptive Font Sizing** - 11 font sizes (36pt-74pt) with granular 3pt increments for optimal text fitting
- **Bold Text Support** - Weather-related keywords highlighted in bold using asterisk markup
- **Deep Sleep Mode** - Months of battery life on a single charge
- **Battery Monitoring** - Battery level reported to Home Assistant, low-battery icon on screen
- **New Quote Button** - Press KEY1 to wake the frame and show the next quote
- **Weather Icons** - Material Design Icons for all weather conditions
- **Automatic Updates** - Scheduled refresh at 6:00, 10:00, 14:00, 18:00, and 22:00

## Hardware Requirements

- **Seeed Studio XIAO ePaper Display Board EE04** (XIAO ESP32-S3 Plus, built-in LiPo charger and power switch)
- **7.5" 800×480 monochrome e-paper panel, 24-pin** (Waveshare 7.5inV2 or Seeed equivalent)
- **3.7V LiPo battery** with a JST 2.0 mm plug (e.g. 2000–3000 mAh)
- **Home Assistant** (for weather data)

### Assembly

1. Set the EE04 jumper to **24 Pin**. With the wrong setting the screen stays blank or shows garbage.
2. Connect the panel's ribbon cable to the 24-pin connector, contacts the right way round.
3. Check battery polarity before plugging it in: red must go to **+** on the board. Many generic LiPo plugs are wired the other way and can damage the board.
4. Charge over USB-C; slide the board's power switch to turn the frame on.

## Weather Conditions Supported

Each condition has its own quote list:

- ☀️ **Sunny/Clear** - Bright, sunny day quotes
- 🌙 **Clear Night** - Starry night quotes
- ☁️ **Cloudy** - Overcast and grey day quotes
- ⛅ **Partly Cloudy** - Sun and clouds quotes
- 🌙☁️ **Night Partly Cloudy** - Moon and clouds quotes
- 🌫️ **Foggy/Hazy** - Mist and fog quotes
- 🌧️ **Rainy** - Gentle rain quotes
- ⛈️ **Pouring** - Heavy rain and storm quotes
- ❄️ **Snowy** - Snow and snowflakes quotes
- 🌨️ **Snowy-Rainy** - Sleet and mixed precipitation quotes
- ⚡ **Lightning/Thunderstorm** - Thunder and lightning quotes
- 💨 **Windy** - Wind and gale quotes

## Quote Examples

**Sunny:**
> "I'll tell you how the sun rose, — / A ribbon at a time." - Emily Dickinson, *Poems*

**Rainy:**
> "Rain was falling on the chapel, on the garden, on the college. It would rain for ever, noiselessly." - James Joyce, *A Portrait of the Artist as a Young Man*

**Snowy:**
> "All the earth was flat with snow, all the air was thick with snow; more than this no man could see, for all the world was snowing." - R. D. Blackmore, *Lorna Doone*

### Where the quotes come from

Every quote is real. Each one was taken from a book on Project Gutenberg (public domain in the US) and checked word for word against the book's text. The only edits are:
- `*bold*` markers on weather words
- ` / ` between lines of verse
- a trailing `…` where a verse excerpt stops mid-sentence

Quotes in translation use the Gutenberg translation's wording. Some sources were deliberately excluded because the author of a passage can't be pinned down reliably. These are verse quoted inside novels, Longfellow's translations of other poets, and *Lyrical Ballads*, which mixes Wordsworth and Coleridge. Every quote fits the 384×438 px quote area at 36pt or larger.

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/kliszaj/eink-weather-frame.git
cd eink-weather-frame
```

### 2. Add the Quote Fonts

The quotes are set in PP Mondwest by Pangram Pangram, which isn't included here. Get your own copy and add these to `fonts/`, or point `weatherman.yaml` at another font:
- `PPMondwest-Regular.otf` - Quote text
- `PPMondwest-Bold.otf` - Bold emphasis

`materialdesignicons-webfont.ttf`, for the weather icons, is already included.

### 3. Create secrets.yaml

Create a `secrets.yaml` file with your WiFi credentials:

```yaml
wifi_ssid: "YourWiFiSSID"
wifi_password: "YourWiFiPassword"
```

### 4. Configure Home Assistant

Ensure you have the OpenWeatherMap integration set up in Home Assistant with these sensors:
- `sensor.openweathermap_feels_like_temperature`
- `sensor.openweathermap_condition`

### 5. Flash to the EE04

```bash
esphome run weatherman.yaml
```

The first flash has to be over USB-C (press RESET on the board if no serial port shows up). After that, OTA updates only work while the frame is awake. Press KEY1 (or RESET) to wake it, then flash.

## Configuration

### Font Sizes

The display automatically selects the optimal font size from 11 available options:
- 74pt, 66pt, 60pt, 57pt, 54pt, 51pt, 48pt, 45pt, 42pt, 39pt, 36pt

### Deep Sleep Schedule

Default refresh times (configurable in `weatherman.yaml`):
- 06:00
- 10:00
- 14:00
- 18:00
- 22:00

### Quote Selection

Quotes rotate based on display refresh count and weather condition, ensuring variety across updates.

## Technical Details

### Display Layout

```
┌─────────────────────────────────────┐
│  [Icon] Temperature                 │  (y: 120)
│                                     │
│  "Quote text here with *bold*       │  (y: 180-666)
│   words for weather conditions      │
│   wrapped across multiple lines"    │
│                                     │
│  Author Name                        │  (y: 666)
│  Source/Book Title                  │  (y: 698)
└─────────────────────────────────────┘
```

### Text Rendering

- **Quote Area**: 384px width × 438px height
- **Line Spacing**: 3-6px depending on font size
- **Bold Markup**: Use `*text*` to highlight weather keywords
- **Wrapping**: Word-based with automatic line height calculation

### Power Management

- **Active Time**: ~10-15 seconds per update (every wait on boot has a timeout, so a WiFi or Home Assistant outage can't keep it awake)
- **Deep Sleep**: Between scheduled updates
- **WiFi**: Fast connect mode for quick wake-up
- **No data, no redraw**: if Home Assistant doesn't answer, the screen keeps its last good image and the frame goes back to sleep
- **Battery Life**: Seeed rates the EE04 at about 3 months on 2000 mAh; expect roughly 3-5 months on 3000 mAh with 5 refreshes a day
- **Battery Sensors**: `Weatherframe Battery Voltage` and `Weatherframe Battery Level` in Home Assistant, updated once per wake. A battery icon appears on screen below 15%. If the voltage doesn't match a multimeter reading, adjust `multiply` on the `battery_voltage` sensor.

## File Structure

```
eink-weather-frame/
├── weatherman.yaml          # Main ESPHome configuration
├── secrets.yaml            # WiFi credentials (not in repo)
├── fonts/                  # Font files
│   ├── PPMondwest-Regular.otf       (not in repo, add your own)
│   ├── PPMondwest-Bold.otf          (not in repo, add your own)
│   └── materialdesignicons-webfont.ttf
└── README.md              # This file
```

## Customization

### Adding Your Own Quotes

Edit the quote arrays in `weatherman.yaml`:

```cpp
static const Quote sunny_quotes[] = {
  {"Your quote text with *bold* words.", "Author Name", "Book Title"},
  // Add more quotes here...
};
```

Each list can be any length; the code reads its size automatically. Bold single words only (`*sun*`, not `*bright sun*`). A bold phrase that wraps onto the next line loses its formatting. Avoid double quotes and accented letters, because the fonts only include basic Latin characters. Keep a quote to roughly 200 characters or fewer so it fits on the screen.

### Changing Update Schedule

Modify the sleep calculation in the `update_and_sleep` script:

```yaml
# Example: Update every 4 hours instead
if (current_hour >= 0 && current_hour < 4) {
  sleep_minutes = (4 - current_hour) * 60 - current_minute;
} else if (current_hour >= 4 && current_hour < 8) {
  // ...
```

### Adjusting Font Sizes

Add or remove font definitions in the `font:` section and update the `FontOption fonts[]` array.

## Troubleshooting

### Display Not Updating
- Check WiFi connection in logs
- Verify Home Assistant API connection
- Ensure weather sensors are available

### Text Overlapping
- Verify all font files are present
- Check font size definitions match font files
- Review debug logs for line height measurements

### Quotes Not Changing
- Check `recorded_display_refresh` counter is incrementing
- Verify quote selection modulo calculation

### Battery Draining Quickly
- Verify deep sleep is working (check logs)
- Check the frame isn't waking repeatedly from the KEY1 wake-up button (look for frequent boots in the logs)
- Ensure WiFi fast_connect is enabled
- Check for continuous wake loops

## Contributing

Contributions are welcome! Feel free to:
- Add more literary quotes
- Support additional weather conditions
- Improve font selection algorithm
- Enhance display layout
- Fix bugs or improve documentation

## License

This project is provided as-is for personal use. Font licenses apply to their respective files.

## Acknowledgments

- **ESPHome** - For the excellent ESP32 framework
- **Home Assistant** - For weather data integration
- **Waveshare** - For the e-ink display
- **Material Design Icons** - For weather icons
- **Classic Literature Authors** - For the beautiful quotes

---

Built with ❤️ for book lovers and weather watchers
