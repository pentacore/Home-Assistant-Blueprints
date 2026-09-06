# Home Assistant Blueprints

A collection of Home Assistant blueprints.

| Blueprint | Description | Import |
| --- | --- | --- |
| [IKEA BILRESA Scroll Wheel (Matter)](#ikea-bilresa-scroll-wheel-matter) | Control lights with the IKEA BILRESA scroll wheel remote over Matter. All 3 channels, brightness, color temperature, custom actions, single/double/triple/long press and hold. | [![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fpentacore%2FHome-Assistant-Blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fikea_bilresa_scroll_wheel.yaml) |

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
| Wheel controls | **Light brightness**, **Light color temperature** or **Custom actions** |
| Lights | Lights driven by the wheel and toggled by a single press |
| Custom scroll up / down | Your own actions when the mode is Custom actions. Variables: `steps` (1 to 8 notches), `direction` (`up` or `down`), `channel` (1 to 3) |
| Single press | Runs after the default light toggle (the toggle can be turned off under Behaviour) |
| Double press, Triple press | Custom actions |
| Long press | Runs once when the hold is detected |
| Hold (repeating) | Repeats every *Hold repeat interval* until the button is released |
| Release after hold | Custom action |

### Settings

**Light settings**: brightness per notch (default 10 %), minimum brightness (5 %), scroll up turns lights on (on), scroll down turns lights off (off), color temperature per notch (150 K, clockwise is cooler), transition (0.3 s).

**Behaviour**: single press toggles the lights, reverse wheel direction, scroll response, hold repeat interval.

### Scroll response: batched vs live

Home Assistant's Matter integration reports a scroll as **one event after you stop turning**, containing the number of notches. It caps that number at 8 and drops anything above, so a very fast spin can be lost. This is the default **batched** behaviour and needs no setup.

For **live** per-notch response, open the BILRESA device page in Home Assistant, show the hidden entities and enable the nine **Current position** sensors. With *Scroll response* set to **Auto** the blueprint switches to live mode on its own as soon as those sensors are enabled. Live mode sends one light update per notch, so it feels smoother but generates more traffic.

### How the entities are found

The BILRESA appears as nine `event` entities, one per Matter endpoint:

| Channel | Scroll up (clockwise) | Scroll down (counter-clockwise) | Button |
| --- | --- | --- | --- |
| 1 | endpoint 1 | endpoint 2 | endpoint 3 |
| 2 | endpoint 4 | endpoint 5 | endpoint 6 |
| 3 | endpoint 7 | endpoint 8 | endpoint 9 |

The blueprint finds them from the selected device by matching entity IDs ending in `_1` to `_9` (for example `event.bilresa_scroll_wheel_button_4`) or in `pos_N_cw`, `pos_N_ccw`, `pos_N_press`. If you renamed the entity IDs so this no longer matches, set them by hand under **Advanced - manual entity mapping**.

### Events used

| Event type | Meaning |
| --- | --- |
| `multi_press_1` ... `multi_press_8` on a scroll entity | Turned N notches |
| `multi_press_1`, `multi_press_2`, `multi_press_3` on the button entity | Single, double, triple press |
| `long_press`, `long_release` | Hold started, hold released |

### Credits

Inspired by the community blueprints by jhol-byte and bradbowles, and by the analysis of the BILRESA Matter endpoints in the ha-ikea-bilresa project.
