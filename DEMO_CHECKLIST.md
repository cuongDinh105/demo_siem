# Wazuh AI Dashboard Demo Checklist

## Before Demo

```text
[ ] Dashboard opens on port 8088
[ ] wazuh-ai-dashboard.service is active
[ ] wazuh-local-ai.service is active
[ ] wazuh-local-ai-full-log.service is active
[ ] wazuh-telegram-ai-chat.service is active
[ ] Ollama API is active
[ ] Demo Mode is selected
[ ] Agent Win11-VMatrix is active in Wazuh
[ ] /var/log/wazuh-local-ai.jsonl is being written
[ ] /var/log/wazuh-local-ai-full-log.jsonl is being written
[ ] /root/project/.env.telegram exists and is not shown on screen
[ ] Telegram /latest returns the latest AI alert
```

## Demo Attack Command

Run from Kali:

```bash
sudo nmap -sT -Pn -p 135,139,445 192.168.7.151
```

Expected rules:

```text
100503 - Port 135 probe detected from Kali to Win11
100504 - Port 139 probe detected from Kali to Win11
100505 - Port 445 probe detected from Kali to Win11
```

## Demo Flow To Explain

```text
Kali nmap
-> Win11 Firewall DROP
-> Wazuh Agent Win11-VMatrix
-> Wazuh Manager alert
-> Local AI triage
-> NiceGUI SOC Dashboard
-> Telegram notification
-> Telegram SOC AI Chat
```

## Screenshot List

```text
[ ] Dashboard overview in Demo Mode
[ ] Port 135/139/445 statistic cards
[ ] Realtime AI Alerts with rule 100503/100504/100505
[ ] Rule 100505 selected in AI Detail Panel
[ ] SOC AI Chat explains selected rule 100505
[ ] Telegram notification for rule 100503/100504/100505
[ ] Telegram /latest or Compromise? answer
[ ] SOC Mode grouped by source type
[ ] Full Log Mode observer view
[ ] Service status output
```

## Useful Verification Commands

Dashboard:

```bash
curl -I http://127.0.0.1:8088
systemctl status wazuh-ai-dashboard --no-pager
systemctl status wazuh-telegram-ai-chat.service --no-pager
```

AI output:

```bash
tail -n 20 /var/log/wazuh-local-ai.jsonl
tail -n 20 /var/log/wazuh-local-ai-full-log.jsonl
```

Demo alert output:

```bash
grep -E '100503|100504|100505' /var/log/wazuh-local-ai.jsonl | tail -n 20
tail -20 /var/log/wazuh-local-ai-telegram-sent.txt
```

## Final Demo Conclusion

```text
Kali 192.168.7.42 performed reconnaissance against Win11 192.168.7.151 on ports 135, 139, and 445. Windows Firewall recorded DROP events, Wazuh generated demo alerts 100503/100504/100505, Local AI presented the triage result on the dashboard, and Telegram provided deduplicated notification plus SOC chat analysis. The event is reconnaissance/port probing, with no evidence of successful exploitation.
```
