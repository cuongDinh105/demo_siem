# Wazuh AI Dashboard User Guide

## Open The Dashboard

Use one of the lab URLs:

```text
http://192.168.3.208:8088
http://192.168.7.2:8088
```

## Header

The header shows:

- Dashboard title
- Current pipeline text
- Local model name
- Service status for:
  - `wazuh-local-ai.service`
  - `wazuh-local-ai-full-log.service`
  - Ollama API

## Mode Selector

Use the mode buttons near the top:

```text
Demo Mode
SOC Mode
Full Log Mode
```

### Demo Mode

Use this for the main nmap demo. It is the default mode.

Visible content:

- Port 135 Probe card
- Port 139 Probe card
- Port 445 Probe card
- Realtime AI Alerts for rule `100503`, `100504`, `100505`
- Other AI Observations
- AI Detail Panel

Useful filters:

```text
All Demo Rules
100503 Port 135
100504 Port 139
100505 Port 445
Medium+
High only
```

### SOC Mode

Use this to review all alert AI output.

SOC Mode reads:

```text
/var/log/wazuh-local-ai.jsonl
```

Alerts are grouped by:

```text
Windows
Linux
OPNsense
Wazuh
Unknown
```

Click any row to inspect the selected event in the AI Detail Panel.

### Full Log Mode

Use this for hunting and broader context.

Full Log Mode reads:

```text
/var/log/wazuh-local-ai-full-log.jsonl
```

Each row can be clicked to open the AI Detail Panel. This view is for observation and hunting, not the primary alert queue.

## NEW Badge

Rows with `NEW` have not been clicked yet in the current dashboard session.

Clicking a row:

- marks it as viewed
- removes the `NEW` badge
- loads the event into the AI Detail Panel

## AI Detail Panel

The detail panel shows:

- Detection
- Source and target details for demo alerts
- AI Risk
- AI Summary
- AI Why
- AI Next Steps
- AI Report Conclusion
- Raw JSON

Use `Auto select latest` to return the panel to the newest demo event, prioritizing rule `100505`.

## SOC AI Chat

The `SOC AI Chat` panel is under the AI Detail Panel.

Use it to ask the local model about the selected alert or full-log record. Quick prompts are available for:

- Explain Alert
- Next Steps
- Report Conclusion
- Compromise?
- Hunt Related

The chat uses the selected event as context. If no event is selected, it uses the latest records from the current mode. The chat calls Ollama only when you submit a question or click a quick prompt.

While Ollama is working, the panel shows:

```text
Local AI đang suy luận...
```

Buttons and input are temporarily disabled to avoid duplicate requests.

## Telegram SOC Chat

The project also includes a Telegram chat bot with similar analysis actions.

Service:

```text
wazuh-telegram-ai-chat.service
```

Useful commands:

```text
/start
/help
/latest
```

Quick buttons:

```text
Latest Alert
Explain Alert
Next Steps
Report Conclusion
Compromise?
Hunt Related
```

The Telegram bot is useful when presenting without staying on the HTTP dashboard. It reads the same JSONL outputs and calls the same local Ollama backend.

## Refresh

The dashboard auto-refreshes every 2 seconds.

Use `Refresh now` for manual refresh.

If a record appears first as `pending` and later as `ok`, the dashboard keeps the final `ok` version for the same `event_id`.
