# Home Assistant Blueprints

A collection of Home Assistant blueprints.

| Blueprint | Description | Import |
| --- | --- | --- |
| [IKEA BILRESA Scroll Wheel (Matter)](#ikea-bilresa-scroll-wheel-matter) | Control lights with the IKEA BILRESA scroll wheel remote over Matter. All 3 channels, brightness, color temperature, hue, a combined mode, custom actions, single/double/triple/long press and hold. | [![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fpentacore%2FHome-Assistant-Blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fikea_bilresa_scroll_wheel.yaml) |

## IKEA BILRESA Scroll Wheel (Matter)

File: [`blueprints/automation/ikea_bilresa_scroll_wheel.yaml`](blueprints/automation/ikea_bilresa_scroll_wheel.yaml)

Requires Home Assistant 2026.9 or newer and the BILRESA scroll wheel added through the **Matter** integration.

### Setup

1. Import the blueprint with the button above (or paste the file URL under Settings > Automations > Blueprints > Import).
2. Create an automation from the blueprint and pick your BILRESA under **BILRESA scroll wheel**.
3. For **Channel 1**, choose what the wheel controls and select the lights. Channels 2 and 3 are optional; switch channels on the remote with the small button below the three LEDs.
4. Save. Turning the wheel dims the lights, a single press toggles them.

One automation covers the whole remote. Channels without lights or custom actions simply do nothing.

### What each channel offers

| Input | What it does |
| --- | --- |
| Wheel controls | **Light brightness**, **Light color temperature**, **Light color (hue)**, **Brightness + color** or **Custom actions** |
| Lights | Lights driven by the wheel and toggled by a single press |
| Custom scroll up / down | Your own actions when the mode is Custom actions. Variables: `steps` (1 to 8 notches), `direction` (`up` or `down`), `channel` (1 to 3) |
| Single press | Runs after the default light toggle (the toggle can be turned off under Behaviour) |
| Double press, Triple press | Custom actions. In the combined mode they also open the color window (see below) |
| Long press | Runs once when the hold is detected |
| Hold (repeating) | Repeats every *Hold repeat interval* until the button is released |
| Release after hold | Custom action |

### Brightness and color on one channel (combined mode)

Set *Wheel controls* to **Brightness + color**. Turning the wheel changes brightness. **Double press**, then turn, changes color temperature. **Triple press**, then turn, changes hue on color lights. The color window stays open for 15 seconds after the press (adjustable under Behaviour), then the wheel returns to brightness. No helper entities are needed.

Lights that do not support the requested color type are skipped, so a mixed group of white and RGB lights works: double press affects the white-capable ones, triple press the color-capable ones.

### Settings

**Light settings**: brightness per notch (default 10 %), minimum brightness (5 %), scroll up turns lights on (on), scroll down turns lights off (off), color temperature per notch (150 K, clockwise is cooler), hue per notch (10 degrees), saturation for hue mode (100 %, used when a light switches from white to color), transition (0.3 s).

**Behaviour**: single press toggles the lights, reverse wheel direction, scroll response, color window, hold repeat interval.

### Scroll response: batched vs live

Home Assistant's Matter integration reports a scroll as **one event after you stop turning**, containing the number of notches. It caps that number at 8 and drops anything above, so a very fast spin can be lost. This is the default **batched** behaviour and needs no setup.

For **live** per-notch response:

1. Open the BILRESA device page in Home Assistant, show the hidden entities and enable the nine **Current switch position** sensors.
2. In the automation set *Scroll response* to **Live** and save. Saving matters: the blueprint looks up the device's entities when the automation loads, and disabled entities are invisible to it.

Live mode sends one light update per notch, so it feels smoother but generates more traffic.

### How the entities are found

The BILRESA appears as nine `event` entities, one per Matter endpoint:

| Channel | Scroll up (clockwise) | Scroll down (counter-clockwise) | Button |
| --- | --- | --- | --- |
| 1 | endpoint 1 | endpoint 2 | endpoint 3 |
| 2 | endpoint 4 | endpoint 5 | endpoint 6 |
| 3 | endpoint 7 | endpoint 8 | endpoint 9 |

The blueprint finds them from the selected device by the endpoint number in the entity ID (for example `event.bilresa_scroll_wheel_button_4`, localized names such as `..._knapp_4` work too) or by label (`..._pos_2_cw`). Home Assistant appends a number when an entity ID already exists, for example after re-adding the device (`..._knapp_4_3`); that suffix is detected and handled. If none of this matches, the nine event entities sorted by name are used in endpoint order, and you can always set them by hand under **Advanced - manual entity mapping**.

After re-importing the blueprint or changing the device's entities, open the automation and save it once so the entity lookup runs again.

### Events used

| Event type | Meaning |
| --- | --- |
| `multi_press_1` ... `multi_press_8` on a scroll entity | Turned N notches |
| `multi_press_1`, `multi_press_2`, `multi_press_3` on the button entity | Single, double, triple press |
| `long_press`, `long_release` | Hold started, hold released |

### Credits

Inspired by the community blueprints by jhol-byte and bradbowles, and by the analysis of the BILRESA Matter endpoints in the ha-ikea-bilresa project.
