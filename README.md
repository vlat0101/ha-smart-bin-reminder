# Smart Bin/Trash Reminder for Home Assistant

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fvlat0101%2Fha-smart-bin-reminder%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fsmart_bin_reminder.yaml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![HA Version](https://img.shields.io/badge/Home%20Assistant-2025.8%2B-blue.svg)](https://www.home-assistant.io)

A smart bin night reminder system for Home Assistant with optional AI camera verification, escalating notifications, and visual light alerts.

Never miss bin night again!

## Features

- **Automatic bin night detection** from any schedule sensor integration
- **Up to 3 bin types** (general waste, recycling, organics, etc.)
- **Escalating notifications** - friendly reminders escalate to urgent warnings
- **Multi-device notifications** - notify up to 3 phones/devices simultaneously
- **Optional AI camera verification** - uses any AI provider to visually confirm bins are out
- **Optional light flashing** - flash a color light as a visual nudge
- **Actionable notifications** - tap "Bins are out!" to trigger immediate verification
- **Morning reset** - automatic cleanup with bring-back reminder for the afternoon
- **Afternoon return check** - periodic reminders to bring bins back from the curb
- **Fully customizable** - timing, messages, colors, and behavior are all configurable

## How It Works

```
BIN NIGHT EVENING:
6:00 PM  - Check schedule sensors → tomorrow is collection day?
         - YES → send first friendly reminder, activate monitoring
8:00 PM  - Begin periodic checks every 15 minutes
         - [AI enabled] Camera snapshot → AI verifies bins are out
         - [AI disabled] Simple reminder notifications
         - Escalating urgency: friendly → stern → FINAL WARNING
         - [Light flash] Optional colored light flash with each nag
??:??    - Bins confirmed out → "Great work!" → stop nagging

NEXT MORNING:
7:00 AM  - Reset monitoring, activate bring-back reminder

NEXT AFTERNOON:
4:00 PM  - Check every 30 min if bins are back
         - [AI enabled] Camera verifies bins returned to normal spot
         - Nag until bins are back → "All done until next bin night!"
```

## Prerequisites

- **Home Assistant 2025.8.0** or newer
- **Bin collection schedule sensor** from any integration:
  - [Waste Collection Schedule](https://github.com/mampfes/hacs_waste_collection_schedule) (HACS - most popular)
  - [Garbage Collection](https://github.com/bruxy70/Garbage-Collection) (HACS)
  - Any sensor that shows the next collection date or days until collection
- **Mobile app** on at least one phone (for notifications)
- **(Optional)** Camera that can see the bin storage area + AI Task integration
- **(Optional)** Color-capable smart light for flash alerts

## Installation

### Step 1: Install the Helper Entities Package

The blueprint needs 6 helper entities to track state between runs. These are provided as a package file.

1. Download [`packages/smart_bin_reminder.yaml`](packages/smart_bin_reminder.yaml) to your Home Assistant `/config/packages/` directory

2. If you haven't enabled packages yet, add this to your `configuration.yaml`:

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

3. Restart Home Assistant

4. Verify the helpers exist: go to **Developer Tools → States** and search for `sbr_`

### Step 2: Import the Blueprint

Click the button below or manually import the URL:

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fvlat0101%2Fha-smart-bin-reminder%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fsmart_bin_reminder.yaml)

Or manually: **Settings → Automations & Scenes → Blueprints → Import Blueprint** and paste:
```
https://github.com/vlat0101/ha-smart-bin-reminder/blob/main/blueprints/automation/smart_bin_reminder.yaml
```

### Step 3: Create the Automation

1. Go to **Settings → Automations & Scenes → Create Automation → Smart Bin/Trash Reminder**
2. Configure the required settings:
   - **Bin Schedule**: Select your bin schedule sensor(s) and give them display names
   - **Notifications**: Enter your notify service name(s)
3. Optionally configure (collapsed sections):
   - **Schedule & Timing**: Adjust reminder times and intervals
   - **AI Camera Verification**: Enable AI-powered visual confirmation
   - **Light Flash Alerts**: Enable colored light flashing
   - **Custom Messages**: Personalize notification text
4. Save the automation

## Configuration Guide

### Finding Your Bin Schedule Sensor

Go to **Developer Tools → States** and search for your bin sensor. Common entity patterns:

| Integration | Example Entity | Format |
|---|---|---|
| Waste Collection Schedule | `sensor.general_waste` | Date string: `2026-03-15` or attribute `days: 1` |
| Garbage Collection | `sensor.garbage_collection` | State: `Tomorrow` or attribute `days_until_collection: 1` |
| Custom sensor | `sensor.bin_day` | Varies |

The blueprint checks multiple formats automatically:
- State equals tomorrow's date (`YYYY-MM-DD`)
- State equals `"tomorrow"` (case-insensitive)
- Attribute `days` or `days_until` or `days_until_collection` equals `1`

### Finding Your Notification Service

Go to **Developer Tools → Services** and search for `notify.mobile_app`. Your service will look like `notify.mobile_app_your_phone_name`.

### Setting Up AI Vision (Optional)

See [docs/AI_SETUP.md](docs/AI_SETUP.md) for detailed instructions on setting up AI camera verification with different providers (OpenAI, Google, etc.).

**Important**: You must customize the AI prompts to describe YOUR specific camera view and bin appearance. The default prompts are generic.

### Light Flash Setup (Optional)

Any color-capable smart light works (Hue, LIFX, Zigbee RGBW, etc.). The light's previous state is saved and restored after flashing.

## Dashboard Card (Optional)

Add a monitoring card to your dashboard to see the current system status.

Copy the YAML from [`dashboard/bin_monitor_card.yaml`](dashboard/bin_monitor_card.yaml) into a **Manual Card** on your dashboard.

## Testing

After installation, test each component:

1. **Schedule detection**: Manually set your bin sensor to show tomorrow as collection day, then trigger the automation
2. **Notifications**: Go to **Developer Tools → Services**, call `input_boolean.turn_on` on `input_boolean.sbr_bin_night_active`, then set `input_text.sbr_bins_required_tonight` to your bin name
3. **AI vision** (if enabled): Check `input_text.sbr_bins_detected_out` for the AI response after a nag check
4. **Light flash** (if enabled): Watch the configured light during a nag cycle
5. **Manual confirm**: Tap "Bins are out!" on a notification and verify the response

## Troubleshooting

See [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) for common issues and solutions.

## FAQ

**Q: Can I use this without AI/camera?**
A: Yes! Leave "Enable AI Vision Checks" turned off. The system works as a simple time-based reminder.

**Q: How many bin types are supported?**
A: Up to 3 bin types. Most councils have 2-3 (waste, recycling, organics).

**Q: Does it work with [specific bin schedule integration]?**
A: It should work with any sensor that reports the next collection date. The blueprint checks multiple common formats. If yours doesn't work, please open an issue.

**Q: Can I notify more than 3 devices?**
A: The blueprint supports 3 devices natively. For more, create a [notification group](https://www.home-assistant.io/integrations/group/#notify-groups) and use that as one of the services.

**Q: What AI providers work?**
A: Any provider that supports the Home Assistant AI Task integration with vision capabilities. See [docs/AI_SETUP.md](docs/AI_SETUP.md).

**Q: The nag notifications are too frequent/infrequent.**
A: Adjust "Minutes Between Nag Checks" in the timing section (default: 15 minutes).

**Q: Can I stop the nagging temporarily?**
A: Turn off `input_boolean.sbr_bin_night_active` in the HA UI or via the entities card. Or tap "Bins are out!" on the notification.

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Test your changes with the blueprint
4. Submit a pull request

For bug reports or feature requests, please [open an issue](https://github.com/vlat0101/ha-smart-bin-reminder/issues).

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Built for the [Home Assistant](https://www.home-assistant.io/) community
- Uses the [Home Assistant Blueprint](https://www.home-assistant.io/docs/blueprint/) system
- AI vision powered by [Home Assistant AI Task](https://www.home-assistant.io/integrations/ai_task/) integration
