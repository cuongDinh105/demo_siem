# Wazuh AI Dashboard Operation Guide

## Service Commands

Check dashboard:

```bash
systemctl status wazuh-ai-dashboard --no-pager
curl -I http://127.0.0.1:8088
```

Restart dashboard:

```bash
systemctl restart wazuh-ai-dashboard
```

Check local AI workers:

```bash
systemctl status wazuh-local-ai.service --no-pager
systemctl status wazuh-local-ai-full-log.service --no-pager
```

Check Telegram SOC chat bot:

```bash
systemctl status wazuh-telegram-ai-chat.service --no-pager
```

Restart local AI workers:

```bash
systemctl restart wazuh-local-ai.service
systemctl restart wazuh-local-ai-full-log.service
```

Restart Telegram SOC chat bot:

```bash
systemctl restart wazuh-telegram-ai-chat.service
```

Check Ollama:

```bash
systemctl status ollama --no-pager
curl http://127.0.0.1:11434/api/tags
```

## Important Files

Application:

```text
/opt/wazuh-ai-dashboard/app.py
/opt/wazuh-ai-dashboard/requirements.txt
/opt/wazuh-ai-dashboard/README.md
/etc/systemd/system/wazuh-ai-dashboard.service
```

Project source:

```text
/root/project/wazuh-ai-dashboard/app.py
/root/project/wazuh-ai-dashboard/README.md
```

Input JSONL:

```text
/var/log/wazuh-local-ai.jsonl
/var/log/wazuh-local-ai-full-log.jsonl
```

Telegram files:

```text
/root/project/.env.telegram
/var/log/wazuh-local-ai-telegram-sent.txt
/root/project/scripts/wazuh_telegram_ai_chat.py
/etc/systemd/system/wazuh-telegram-ai-chat.service
```

## Verify Input Data

Alert AI output:

```bash
tail -n 20 /var/log/wazuh-local-ai.jsonl
```

Full-log observer output:

```bash
tail -n 20 /var/log/wazuh-local-ai-full-log.jsonl
```

Demo rule check:

```bash
grep -E '100503|100504|100505' /var/log/wazuh-local-ai.jsonl | tail -n 20
```

Telegram dedup cache:

```bash
tail -20 /var/log/wazuh-local-ai-telegram-sent.txt
```

## Expected Worker Behavior

`wazuh-local-ai.service`:

- Reads Wazuh alerts.
- Focuses on demo rule `100503`, `100504`, `100505`.
- Uses local demo fast path for demo events.
- Writes `pending` and final `ok` records when pending output is enabled.
- Sends Telegram notifications only for `status=ok`.
- Uses 60-second burst dedup by `rule.id + srcip + dstip + dstport`.

`wazuh-local-ai-full-log.service`:

- Reads Wazuh full-log/archive style output.
- Writes observer records.
- Uses risk-only AI behavior so normal logs can be observed without forcing AI analysis.
- Does not send Telegram notifications.

`wazuh-telegram-ai-chat.service`:

- Polls Telegram for `/latest`, free-form questions, and quick button prompts.
- Reads JSONL context.
- Calls Ollama only when the user asks.
- Does not execute shell commands, block IPs, or modify Wazuh.

## Troubleshooting

Dashboard is not reachable:

```bash
systemctl status wazuh-ai-dashboard --no-pager
ss -ltnp | grep 8088
journalctl -u wazuh-ai-dashboard -n 80 --no-pager
```

Dashboard has no demo alerts:

```bash
tail -n 20 /var/log/wazuh-local-ai.jsonl
systemctl status wazuh-local-ai.service --no-pager
```

Full Log Mode has no data:

```bash
tail -n 20 /var/log/wazuh-local-ai-full-log.jsonl
systemctl status wazuh-local-ai-full-log.service --no-pager
```

Ollama unavailable:

```bash
systemctl status ollama --no-pager
curl http://127.0.0.1:11434/api/tags
```

Telegram chat not responding:

```bash
systemctl status wazuh-telegram-ai-chat.service --no-pager
journalctl -u wazuh-telegram-ai-chat -n 100 --no-pager
```

Telegram notification not sent:

```bash
journalctl -u wazuh-local-ai -n 100 --no-pager
tail -20 /var/log/wazuh-local-ai-telegram-sent.txt
```

Common reasons:

```text
record is still pending
risk is below TELEGRAM_MIN_RISK
rule is not in TELEGRAM_RULES
same rule/source/target/port was sent within the 60-second TTL
Telegram token or chat id is invalid
```

## Deployment After Editing Source

After editing `/root/project/wazuh-ai-dashboard/app.py`:

```bash
python3 -m py_compile /root/project/wazuh-ai-dashboard/app.py
cp /root/project/wazuh-ai-dashboard/app.py /opt/wazuh-ai-dashboard/app.py
systemctl restart wazuh-ai-dashboard
curl -I http://127.0.0.1:8088
```
