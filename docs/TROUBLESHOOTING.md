# Troubleshooting

Common issues and solutions for the Smart Bin Reminder.

## Installation Issues

### Helpers not appearing after restart

**Symptom**: The `sbr_*` entities don't show up in Developer Tools → States.

**Solutions**:
1. Verify packages are enabled in `configuration.yaml`:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
2. Make sure `smart_bin_reminder.yaml` is directly inside the `packages/` folder (not in a subfolder)
3. Check the logs for YAML syntax errors: **Settings → System → Logs**
4. Restart Home Assistant (a reload is not sufficient for new packages)

### Blueprint import fails

**Symptom**: Error when importing the blueprint URL.

**Solutions**:
1. Ensure your HA version is 2025.8.0 or newer
2. Try importing the raw YAML URL directly:
   ```
   https://raw.githubusercontent.com/vlat0101/ha-smart-bin-reminder/main/blueprints/automation/smart_bin_reminder.yaml
   ```
3. Alternatively, manually download the file and place it in `/config/blueprints/automation/smart_bin_reminder/smart_bin_reminder.yaml`

## Notification Issues

### No notifications received

**Symptom**: The automation runs but phone doesn't receive notifications.

**Solutions**:
1. Verify your notify service name is correct:
   - Go to **Developer Tools → Services**
   - Search for `notify.mobile_app`
   - Your service will be `notify.mobile_app_<device_name>`
   - The exact name must match (case-sensitive)
2. Test the notification service directly in Developer Tools
3. Ensure the HA Companion App is connected and has notification permissions
4. Check if Do Not Disturb is blocking notifications on the phone

### "Bins are out!" button doesn't work

**Symptom**: Tapping the actionable notification button does nothing.

**Solutions**:
1. The notification action ID is `SBR_BINS_DONE` - ensure no other automation uses this same ID
2. The button only works when `input_boolean.sbr_bin_night_active` is ON and `input_boolean.sbr_bins_confirmed_out` is OFF
3. On Android, ensure the notification channel `bin_reminder` has sufficient priority
4. Try clearing the HA app cache and reconnecting

### Notifications are stacking instead of replacing

**Symptom**: Getting multiple notifications instead of the old one being replaced.

**Solutions**:
1. The blueprint uses notification tags (`sbr_bin_reminder` and `sbr_bin_return_reminder`) to replace previous notifications
2. This works on both iOS and Android with the HA Companion App
3. If using a non-mobile-app notification service (e.g., Telegram), tag-based replacement may not be supported

## Schedule Detection Issues

### Automation doesn't trigger on bin night

**Symptom**: Tomorrow is collection day but the automation didn't fire.

**Solutions**:
1. Check your schedule sensor format in **Developer Tools → States**:
   - Does the state show tomorrow's date as `YYYY-MM-DD`?
   - Does it show the text `tomorrow`?
   - Does it have a `days`, `days_until`, or `days_until_collection` attribute with value `1`?
2. The blueprint checks at the configured evening trigger time (default 6:00 PM). If the sensor updates after this time, the check will be missed.
3. Verify the automation is enabled: **Settings → Automations** → find the automation → ensure it's toggled ON
4. Check the automation traces: **Settings → Automations** → click the automation → **Traces** tab

### Wrong bins detected as due

**Symptom**: The notification says bins are due when they're not (or vice versa).

**Solutions**:
1. Verify each bin sensor individually in Developer Tools → States
2. Make sure you selected the correct sensor for each bin type in the blueprint
3. Check that the sensor's format matches one of the supported patterns

## AI Vision Issues

See [AI_SETUP.md](AI_SETUP.md) for AI-specific troubleshooting.

### AI check seems to run but doesn't update status

**Symptom**: `input_text.sbr_bins_detected_out` updates but `input_boolean.sbr_bins_confirmed_out` never turns on.

**Solutions**:
1. Check the AI detection result in `input_text.sbr_bins_detected_out` to see what the AI returned
2. The logic is: bin NOT visible = bin has been taken out. Make sure the AI is correctly understanding this.
3. Customize the AI prompt to better describe your specific camera view and bin positions
4. The AI must report ALL required bins as "not visible" for confirmation to trigger

### Camera snapshot is black/dark

**Solutions**:
1. Configure a floodlight entity and increase the warmup time (try 8-10 seconds)
2. Ensure the floodlight actually illuminates the bin storage area
3. Check that your camera isn't in IR/night mode when the floodlight is on (some cameras need time to switch from IR to color)

## Timing Issues

### Nag loop fires at unexpected times

**Symptom**: Nag checks run outside the configured window.

**Solutions**:
1. The `time_pattern` trigger fires based on the interval (e.g., every 15 minutes). The time window is enforced via a condition, so most false triggers are silently ignored.
2. Verify your HA timezone is correct: **Settings → System → General → Time Zone**
3. Check the nag start/end times in the automation configuration

### Morning reset doesn't fire

**Symptom**: Helpers stay active past the morning reset time.

**Solutions**:
1. Morning reset only fires when `input_boolean.sbr_bin_night_active` is ON
2. If something turned it off early (e.g., manual toggle), the reset won't trigger
3. Manually reset by turning off all `sbr_*` input_booleans and setting `sbr_bin_nag_count` to 0

## Light Flash Issues

### Light doesn't flash

**Solutions**:
1. Ensure "Enable Light Flashing" is ON in the blueprint configuration
2. Verify the light entity supports RGB color (not all lights do)
3. Test the light manually: Developer Tools → Services → `light.turn_on` with `rgb_color: [255, 0, 0]`
4. The light only flashes during NAG checks (Branch 2), not during the initial evening reminder

### Light doesn't restore to previous state

**Solutions**:
1. The blueprint uses `scene.create` to snapshot the light state before flashing
2. If the light was OFF before flashing, it will be turned OFF after
3. If the scene creation fails (e.g., light entity changed), the restore may fail. Check logs for errors.

## General Debugging

### Checking automation traces

The most powerful debugging tool is the automation trace:

1. Go to **Settings → Automations & Scenes**
2. Find the Smart Bin Reminder automation
3. Click the three dots → **Traces**
4. Select a recent run to see exactly what happened at each step

### Manual testing workflow

1. Turn on `input_boolean.sbr_bin_night_active`
2. Set `input_text.sbr_bins_required_tonight` to your bin name(s)
3. Set `input_number.sbr_bin_nag_count` to `0`
4. Trigger the automation manually from the automation page
5. Check the trace and notification result

### Resetting everything

If the system gets stuck in a bad state:

1. Turn OFF all three `sbr_*` input_booleans
2. Clear both `sbr_*` input_texts (set to empty)
3. Set `sbr_bin_nag_count` to 0
4. Dismiss any persistent notifications: Developer Tools → Services → `persistent_notification.dismiss` with `notification_id: sbr_bin_night_reminder`

## Still Need Help?

[Open an issue](https://github.com/vlat0101/ha-smart-bin-reminder/issues) with:
- Your Home Assistant version
- The automation trace (screenshot or copy)
- Your bin schedule sensor format (from Developer Tools → States)
- Any relevant log entries
