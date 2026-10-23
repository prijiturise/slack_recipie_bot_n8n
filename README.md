# 🍲 Slack Recipe Ingredient Bot

An AI-powered Slack bot built using **n8n, Slack, and OpenAI** that generates ingredient shopping lists based on recipe names shared in a Slack channel.

Users simply send a recipe name, such as `Paneer Butter Masala` or `Vegetable Biryani`, and the bot automatically generates a practical shopping list with approximate quantities, grouped by ingredient category.

## 📌 Project Overview

The Slack Recipe Ingredient Bot automates recipe-based shopping list generation using a workflow in n8n.

The bot listens for new messages in a designated Slack channel, filters out messages sent by itself, processes the recipe name using OpenAI, and posts the generated shopping list back to the same channel.

### ✨ Key Features

- **Automated Slack integration:** Listens for new recipe messages in a Slack channel.
- **AI-powered ingredient generation:** Uses OpenAI to generate ingredient lists.
- **Practical quantities:** Provides approximate ingredient quantities for 2–3 people.
- **Categorized ingredients:** Groups ingredients into categories such as vegetables, dairy, spices, and pantry items.
- **Bot-message filtering:** Prevents the bot from responding to its own messages and creating an infinite loop.
- **Automatic replies:** Sends the shopping list directly to the Slack channel.
- **No manual execution required:** Runs automatically when the workflow is active.

## 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| n8n | Workflow automation |
| Slack | User interaction and messaging |
| OpenAI API | AI-powered shopping list generation |
| Slack Trigger | Detects new channel messages |
| IF Node | Filters bot-generated messages |
| Edit Fields Node | Extracts recipe name and channel ID |
| Basic LLM Chain | Processes the prompt and generates the response |
| OpenAI Chat Model | Provides language model capabilities |
| Slack Send Message | Posts the shopping list |

## ⚙️ Workflow Architecture

```text
User sends a recipe name in Slack
              |
              v
        Slack Trigger
              |
              v
       IF: Bot Filter
         /       \
      True       False
        |           |
        v           v
   Edit Fields    Stop
        |
        v
   Basic LLM Chain
        |
        v
   OpenAI Chat Model
        |
        v
   Slack Send Message
        |
        v
Shopping list posted to Slack
```

**Note:** The True branch processes user messages. The False branch stops bot-generated messages to prevent repeated replies.

## 📋 Prerequisites

Before setting up the project, ensure you have:

- An n8n account (n8n Cloud or a self-hosted instance).
- A Slack workspace where you can install an app.
- A Slack app with the required bot permissions.
- An OpenAI API key and access to a supported model.
- Access to a Slack channel where the bot can read and post messages.

## 🚀 Setup Instructions

### Step 1: Create a Slack App

1. Open the [Slack API website](https://api.slack.com/apps).
2. Create a new Slack app.
3. Add a bot user and configure the required permissions.
4. Enable Event Subscriptions.
5. Subscribe to the `message.channels` bot event for public channel messages.
6. Install or reinstall the app in your Slack workspace after changing permissions.
7. Invite the bot to the target channel, such as `#n8n-automation`.

The precise permissions required depend on your workspace and whether you use public or private channels.

### Step 2: Configure Slack Credentials in n8n

1. Open n8n.
2. Create Slack credentials using the bot's OAuth token.
3. Configure the Slack Trigger credentials.
4. Configure the Slack Send Message credentials.
5. Add the signing secret if required by your chosen trigger configuration.

Keep all tokens and secrets private. Never commit them to GitHub.

### Step 3: Create the n8n Workflow

Create a workflow named:

`Slack Recipe Ingredient Bot`

Add and connect these nodes in order:

1. Slack Trigger
2. IF
3. Edit Fields
4. Basic LLM Chain
5. OpenAI Chat Model (connected to the Basic LLM Chain's model input)
6. Slack Send Message

### Step 4: Configure the Slack Trigger

Configure the Slack Trigger to listen for new messages posted in your chosen channel.

Example:

- Event: On new message posted to channel
- Channel: `n8n-automation`

Complete any required event subscription and request URL verification steps, then activate the workflow.

### Step 5: Configure the IF Node

Use the following condition to allow messages without a bot ID to continue:

```javascript
{{ $json.bot_id }}
```

Operator: **is empty**

- **True:** Continue to Edit Fields.
- **False:** Stop processing.

This prevents messages containing a bot ID from being processed by the AI chain again.

### Step 6: Configure the Edit Fields Node

Create these two fields:

| Field | Expression |
|---|---|
| `recipe` | `{{ $json.text }}` |
| `channelId` | `{{ $json.channel }}` |

These fields extract the recipe name and the channel ID from the incoming Slack message.

### Step 7: Configure the Basic LLM Chain

Use the following prompt:

```text
You are a helpful recipe shopping assistant.

Recipe name: {{ $json.recipe }}

Create a practical shopping list for preparing this recipe
for 2–3 people.

Instructions:
- List the ingredients and approximate quantities.
- Group ingredients into categories such as vegetables,
  dairy, spices, and pantry items.
- Use clear, concise bullet points suitable for a Slack message.
- If the recipe name is empty or unclear, politely ask
  the user to clarify.
- Do not invent ingredients for an unrecognized recipe.
- Return plain text, not JSON.

Start with: "Here is your shopping list:"
```

Connect the OpenAI Chat Model node to the model input of the Basic LLM Chain and select your configured model.

### Step 8: Configure Slack Send Message

Configure the Slack node as follows:

- Resource: Message
- Operation: Send
- Send Message To: Channel

Channel expression:

```javascript
{{ $('Edit Fields').item.json.channelId }}
```

For the message text, select the actual text output field from the Basic LLM Chain node. In this workflow, it may be:

```javascript
{{ $('Basic LLM Chain').item.json.text }}
```

Verify the output field against your node's execution data because the field name can vary with the node version or configuration.

### Step 9: Activate and Test

1. Save the workflow.
2. Activate or publish it.
3. Open the configured Slack channel.
4. Send a recipe name, such as:

   `Paneer Butter Masala`

5. Wait for the bot to generate and post the shopping list.
6. Open the n8n Executions page and confirm that the workflow succeeded.
7. Confirm that the bot's own response is routed to the IF node's False branch.

## 💬 Example Interaction

**User message in Slack:**

```text
Paneer Butter Masala
```

**Example bot response:**

```text
Here is your shopping list:

Vegetables
- Onions: 2 medium
- Tomatoes: 3 medium
- Green chillies: 1–2

Dairy
- Paneer: 250–300 g
- Fresh cream: 100 ml
- Butter: 2 tbsp

Spices
- Ginger-garlic paste: 1 tbsp
- Garam masala: 1 tsp
- Turmeric powder: 1/4 tsp
- Red chilli powder: 1/2 tsp

Pantry Items
- Cooking oil: 2 tbsp
- Salt: as required
```

*This is an illustrative response. Actual ingredients and quantities may vary depending on the recipe and model output.*

## 🔄 How It Works

1. A user sends a recipe name to the Slack channel.
2. The Slack Trigger captures the new message.
3. The IF node checks whether the message contains a bot ID.
4. If the message is from a user, Edit Fields extracts the recipe and channel ID.
5. The Basic LLM Chain sends the recipe and instructions to OpenAI.
6. OpenAI generates a categorized shopping list.
7. The Slack Send Message node posts the result to the original channel.
8. If the message comes from a bot, the IF node stops it from entering the processing chain.

## 🧪 Test Cases

| Test | Expected Result |
|---|---|
| Send `Paneer Butter Masala` | Generates a shopping list |
| Send `Vegetable Biryani` | Generates ingredients for that recipe |
| Send `Masala Dosa` | Generates a dosa shopping list |
| Send an empty or unclear recipe name | Requests clarification |
| Bot posts its own response | IF node stops reprocessing |
| Send another recipe | Generates a new response |
| Temporarily disable the workflow | No production response until reactivated |

## 🔐 Security Best Practices

- Store Slack tokens and OpenAI API keys in n8n credentials.
- Never expose secrets in workflow screenshots, source code, or GitHub repositories.
- Grant only the Slack permissions required by the workflow.
- Monitor OpenAI API usage and costs.
- Avoid logging sensitive information unnecessarily.

## 🛠️ Troubleshooting

### Bot does not respond

- Confirm that the workflow is active.
- Check the latest execution in n8n.
- Verify Slack event subscriptions and credentials.
- Ensure the bot is a member of the target channel.

### Bot replies repeatedly

- Confirm that the IF node checks `{{ $json.bot_id }}` for emptiness.
- Verify that the True branch leads to Edit Fields.
- Ensure the False branch is unconnected.
- Inspect the Slack Trigger output for bot-related fields if necessary.

### OpenAI node fails

- Check the OpenAI credentials.
- Confirm the selected model is available to your API account.
- Review the node's error details and API usage limits.

### Slack message is empty

- Open the Basic LLM Chain execution output.
- Verify the actual response field name.
- Confirm the Slack Send Message text expression references that field.

### Slack channel is incorrect

- Verify the Edit Fields expression:

```javascript
{{ $json.channel }}
```

- Confirm the Slack Send Message node uses the extracted `channelId`.

## 🚧 Future Enhancements

Possible improvements include:

- Estimated ingredient prices and total shopping cost.
- Support for different serving sizes.
- Vegetarian, vegan, and dietary preference options.
- Combining shopping lists for multiple recipes.
- Tamil-language recipe support.
- Exporting shopping lists as CSV or PDF.
- Adding buttons for recipe suggestions and serving-size adjustments.

## 📌 Project Status

**Status: Completed — Core workflow tested successfully.**

The workflow successfully receives recipe names, generates shopping lists using OpenAI, sends responses to Slack, and filters bot-generated messages to prevent repeated replies.

## 👩‍💻 Author

Created as an automation project using n8n, Slack, and OpenAI.

---

**Disclaimer:** Ingredient quantities are estimates generated by an AI model. Review the list and adjust quantities based on your recipe, preferences, and serving requirements.
