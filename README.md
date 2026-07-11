# Wazuh Local AI Realtime SOC Dashboard

NiceGUI dashboard for viewing local AI analysis generated from Wazuh realtime JSONL outputs.

The dashboard is a read-only SOC UI. It does not modify Wazuh, does not run active response, and does not block IP addresses. The SOC AI Chat only calls the local model when the user asks a question.

The broader project also includes Telegram notification and a Telegram SOC AI chat bot. Those are separate from the HTTP dashboard and run as separate services.

## Input Files

```text
/var/log/wazuh-local-ai.jsonl
/var/log/wazuh-local-ai-full-log.jsonl
```

## Dashboard Modes

- `Demo Mode`: default mode. Shows only demo rules `100503`, `100504`, and `100505`.
- `SOC Mode`: shows alert AI output from `/var/log/wazuh-local-ai.jsonl`, grouped by source type: Windows, Linux, OPNsense, Wazuh, Unknown.
- `Full Log Mode`: shows observer data from `/var/log/wazuh-local-ai-full-log.jsonl` for hunting. It is not the primary alert queue.

## SOC AI Chat

The right side of the dashboard includes `SOC AI Chat`.

- Uses the selected alert/log as context.
- Can explain the event, recommend next steps, write a short report conclusion, and suggest related hunting.
- Calls Ollama only when the user clicks `Ask` or a quick prompt.
- Does not change the AI pipeline or re-process Wazuh logs.
- Shows a `Local AI đang suy luận...` state while waiting for Ollama.

If a selected event has both `pending` and final `ok` records with the same `event_id`, the dashboard prefers the final `ok` record.

## Telegram Integration

Telegram alerting and Telegram chat are handled outside this dashboard:

```text
wazuh-local-ai.service              sends deduplicated alert notifications
wazuh-telegram-ai-chat.service      provides /latest and SOC AI chat prompts
```

See:

```text
/root/project/telegram-ai-chat-guide.md
```

## Install

```bash
python3 -m venv /opt/wazuh-ai-dashboard/.venv
source /opt/wazuh-ai-dashboard/.venv/bin/activate
pip install -r /opt/wazuh-ai-dashboard/requirements.txt
```

## Run As Service

```bash
cp /opt/wazuh-ai-dashboard/wazuh-ai-dashboard.service /etc/systemd/system/wazuh-ai-dashboard.service
systemctl daemon-reload
systemctl enable --now wazuh-ai-dashboard.service
systemctl status wazuh-ai-dashboard --no-pager
```

## Open Dashboard

```text
http://<proxmox-host-ip>:8088
```

Known lab URLs:

```text
http://192.168.3.208:8088
http://192.168.7.2:8088
```

Local test:

```bash
curl -I http://127.0.0.1:8088
```

## Demo Nmap Flow

From Kali:

```bash
sudo nmap -sT -Pn -p 135,139,445 192.168.7.151
```

Expected Wazuh rules:

```text
100503 - Port 135 probe detected from Kali to Win11
100504 - Port 139 probe detected from Kali to Win11
100505 - Port 445 probe detected from Kali to Win11
```

## Documentation

- `SUMMARY.md`: short functional summary.
- `USER_GUIDE.md`: dashboard usage guide.
- `OPERATION_GUIDE.md`: service, troubleshooting, and verification commands.
- `DEMO_CHECKLIST.md`: pre-demo and screenshot checklist.
- `../telegram-ai-chat-guide.md`: Telegram notification and SOC AI chat bot guide.
