# Discord Deployment Notification

A GitHub Action that sends messages to Discord channels via webhooks with flexible embed support.

## Description

This action allows you to send custom messages or rich embeds to Discord channels during your GitHub workflows, making it easy to notify teams about deployments, build statuses, or other workflow events. It supports both simple text messages and advanced embed formatting with custom fields, colors, and templates.

## Inputs

| Input             | Description                                         | Required | Default        |
| ----------------- | --------------------------------------------------- | -------- | -------------- |
| `discord_webhook` | Base64 encoded Discord Webhook URL                 | Yes      | -              |
| `message`         | Message to send to the Discord channel             | No       | ""             |
| `embed_template`  | Discord embed template with placeholders           | No       | ""             |
| `embed_title`     | Title for the embed                                 | No       | "Notification" |
| `embed_description` | Description for the embed                         | No       | ""             |
| `embed_color`     | Color for the embed (decimal)                       | No       | "3066993"      |
| `embed_fields`    | Base64 encoded JSON array of fields for the embed  | No       | ""             |

## Usage

### Simple Message

```yaml
- name: Notify Discord
  uses: ./discord-actions
  with:
    discord_webhook: ${{ secrets.DISCORD_WEBHOOK }}
    message: "🚀 Deployment completed successfully!"
```

### Rich Embed

```yaml
- name: Notify Discord with Embed
  uses: ./discord-actions
  with:
    discord_webhook: ${{ secrets.DISCORD_WEBHOOK }}
    embed_title: "Deployment Status"
    embed_description: "Deployment to production completed successfully"
    embed_color: "65280"
    embed_fields: ${{ secrets.EMBED_FIELDS }}
```

## Setting up Discord Webhook

1. Go to your Discord server settings
2. Navigate to Integrations → Webhooks
3. Create a new webhook or use an existing one
4. Copy the webhook URL
5. Base64 encode the webhook URL:
   ```bash
   echo -n "your-webhook-url" | base64
   ```
6. Add the encoded value to your GitHub repository secrets as `DISCORD_WEBHOOK`

## Examples

### Basic notification

```yaml
- uses: ./discord-actions
  with:
    discord_webhook: ${{ secrets.DISCORD_WEBHOOK }}
    message: "Build completed"
```

### Deployment notification with context

```yaml
- uses: ./discord-actions
  with:
    discord_webhook: ${{ secrets.DISCORD_WEBHOOK }}
    message: "🚀 Deployed ${{ github.ref_name }} to production"
```

### Error notification

```yaml
- uses: ./discord-actions
  if: failure()
  with:
    discord_webhook: ${{ secrets.DISCORD_WEBHOOK }}
    message: "❌ Build failed on ${{ github.ref_name }}"
```

### Rich embed with fields

```yaml
- uses: ./discord-actions
  with:
    discord_webhook: ${{ secrets.DISCORD_WEBHOOK }}
    embed_title: "Deployment Complete"
    embed_description: "Successfully deployed to production environment"
    embed_color: "65280"
    embed_fields: "W3tcIm5hbWVcIjpcIkVudmlyb25tZW50XCIsXCJ2YWx1ZVwiOlwicHJvZHVjdGlvblwiLFwiaW5saW5lXCI6dHJ1ZX0se1wibmFtZVwiOlwiVmVyc2lvblwiLFwidmFsdWVcIjpcInYxLjIuM1wiLFwiaW5saW5lXCI6dHJ1ZX1d"
```

## Important Notes

- You must provide either `message` OR `embed_template`, but not both
- If neither is provided, the action will fail
- The webhook URL must be base64 encoded for security
- Embed fields must be base64 encoded JSON array
- The action automatically adds timestamps to embeds

## Embed Fields Format

The `embed_fields` input expects a base64 encoded JSON array:

```json
[
  {
    "name": "Environment",
    "value": "production",
    "inline": true
  },
  {
    "name": "Version",
    "value": "v1.2.3",
    "inline": true
  }
]
```

To encode this:
```bash
echo -n '[{"name":"Environment","value":"production","inline":true},{"name":"Version","value":"v1.2.3","inline":true}]' | base64
```

