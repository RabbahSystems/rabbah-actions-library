# Discord Deployment Notification

A GitHub Action that sends messages to Discord channels via webhooks.

## Description

This action allows you to send custom messages to Discord channels during your GitHub workflows, making it easy to notify teams about deployments, build statuses, or other workflow events.

## Inputs

| Input             | Description                                | Required |
| ----------------- | ------------------------------------------ | -------- |
| `discord_webhook` | Discord Webhook URL to authenticate sender | Yes      |
| `message`         | Message to send to the Discord channel     | Yes      |

## Usage

```yaml
- name: Notify Discord
  uses: ./discord-actions
  with:
    discord_webhook: ${{ secrets.DISCORD_WEBHOOK }}
    message: "🚀 Deployment completed successfully!"
```

## Setting up Discord Webhook

1. Go to your Discord server settings
2. Navigate to Integrations → Webhooks
3. Create a new webhook or use an existing one
4. Copy the webhook URL
5. Add it to your GitHub repository secrets as `DISCORD_WEBHOOK`

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

