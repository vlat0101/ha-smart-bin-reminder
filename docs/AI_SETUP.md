# AI Camera Verification Setup

The Smart Bin Reminder can optionally use AI vision to verify bins via a security camera. This works with any AI provider that supports the Home Assistant **AI Task** integration.

## How It Works

1. The system turns on a floodlight (if configured) for visibility
2. Takes a snapshot from your camera
3. Sends the image to your AI provider with a prompt
4. AI returns structured data: which bins are visible, confidence level, and notes
5. If bins are NOT visible in their normal spot → they've been taken out (confirmed!)

## Supported AI Providers

Any provider that supports the `ai_task` domain in Home Assistant:

| Provider | Integration | Vision Support | Notes |
|---|---|---|---|
| **OpenAI** (GPT-4o) | [OpenAI Conversation](https://www.home-assistant.io/integrations/openai_conversation/) | Yes | Recommended. Excellent at image analysis. |
| **Google Gemini** | [Google Generative AI](https://www.home-assistant.io/integrations/google_generative_ai_conversation/) | Yes | Good alternative. |
| **Anthropic Claude** | [Anthropic](https://www.home-assistant.io/integrations/anthropic/) | Yes | Strong vision capabilities. |
| **Ollama** (local) | [Ollama](https://www.home-assistant.io/integrations/ollama/) | Varies | Use a vision model like LLaVA. No API costs. |
| **OpenRouter** | Via OpenAI-compatible setup | Varies | Access to many models. |

## Setup Instructions

### 1. Install an AI Integration

Go to **Settings → Devices & Services → Add Integration** and install one of the supported AI integrations.

For **OpenAI** (recommended):
- Get an API key from [platform.openai.com](https://platform.openai.com)
- Add the OpenAI Conversation integration
- Enter your API key
- Select a vision-capable model (GPT-4o recommended)

### 2. Verify AI Task Entity

After adding the integration, check that an `ai_task` entity was created:

1. Go to **Developer Tools → States**
2. Search for `ai_task`
3. You should see an entity like `ai_task.openai_ai_task` or similar

### 3. Configure the Blueprint

In the blueprint configuration:

1. Set **Enable AI Vision Checks** to `On`
2. Select your **Camera Entity** (the camera that can see the bins)
3. Optionally select a **Floodlight Entity** (for night visibility)
4. Optionally select the specific **AI Task Entity** (or leave blank to use system default)

### 4. Customize the AI Prompts

**This is the most important step!** The default prompts are generic. You MUST customize them to describe YOUR specific setup:

- Where are the bins normally stored? (left side, right side, against a wall, etc.)
- What do the bins look like? (colors, sizes, lid colors)
- What does the camera show? (driveway, backyard, alley, etc.)
- Any landmarks to reference? (fence, wall, garage, garden)

#### Example Custom Prompt (Bins Taken Out Check)

```
Analyze this security camera image showing my driveway.

The bins are normally stored against the LEFT FENCE, near the garage door.
There are two bins:
- A RED-lidded bin (general waste) - smaller
- A YELLOW-lidded bin (recycling) - taller

A floodlight is on for visibility. If a bin is NOT visible in its usual
spot by the left fence, it has been moved to the curb for collection.

Report whether each bin is visible (still needs to go out) or missing
(has been taken out).
```

#### Example Custom Prompt (Bins Returned Check)

```
Analyze this security camera image of my driveway.

Check if the wheelie bins are visible in their normal spot against the
LEFT FENCE near the garage. If visible, they have been brought back from
the curb. If missing, they are still out on the street.
```

## Camera Tips

- **Night vision**: Use a floodlight for color accuracy. IR/night mode cameras show everything in grayscale, making it hard for AI to distinguish bin colors.
- **Camera angle**: The camera should have a clear view of the bin storage area. Avoid extreme angles where bins might be hidden behind objects.
- **Resolution**: Higher resolution helps AI accuracy. 1080p or better recommended.
- **Snapshot path**: The blueprint saves snapshots to `/config/www/sbr_bin_snapshot.jpg`. Ensure the `/config/www/` directory exists and is writable.

## Costs

AI vision API calls cost money (except for local models like Ollama):

- **OpenAI GPT-4o**: ~$0.01-0.02 per image analysis
- **Google Gemini**: Similar pricing
- Typical usage: 5-15 checks per bin night = $0.05-0.30 per bin night

To minimize costs:
- Set a reasonable nag interval (15-30 minutes instead of 5)
- Set a reasonable max nags limit
- The system stops checking once bins are confirmed out

## Troubleshooting AI Issues

**AI returns incorrect results:**
- Customize the prompt to better describe your specific setup
- Try a more capable model (GPT-4o > GPT-4o-mini)
- Ensure the floodlight provides adequate illumination
- Check the snapshot quality in `/config/www/sbr_bin_snapshot.jpg`

**AI task fails:**
- Check the Home Assistant logs for error details
- Verify your API key is valid and has sufficient credits
- Ensure the AI integration is properly configured
- Check that the `ai_task` entity exists in Developer Tools → States

**Bins are always "detected" as out (false positives):**
- The AI might be too conservative. Adjust the prompt to be more specific.
- Add landmarks in the prompt so the AI knows exactly where to look.
- Consider the camera angle - can you actually see the bins clearly?

**Bins are never detected as out (false negatives):**
- The AI might not recognize your bins. Add color/size descriptions.
- Ensure floodlight is on for night checks.
- Try taking a manual snapshot and sending it to the AI to test.
