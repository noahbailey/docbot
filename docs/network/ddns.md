# Dynamic DNS with CloudFlare

A simple bash script that can be run on a linux router to update a Cloudflare DDNS host record. 

```
#!/bin/bash
set -e

ZONE="XXXXX"
RECORD="YYYYY"
TOKEN="ZZZZZ"

ACTUAL_IP=$(ip --json -4 a show dev enp1s0 up | jq .[0].addr_info[].local)
CURRENT_IP=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones/$ZONE/dns_records/$RECORD" \
    -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    | jq .result.content)

BODY='{"type": "A", "name": "example.com", "content": '$ACTUAL_IP', "proxied": false, "ttl": "300"}'

if [ "$CURRENT_IP" != "$ACTUAL_IP" ]; then
    echo "IP requires update."
    curl -s -X PUT "https://api.cloudflare.com/client/v4/zones/$ZONE/dns_records/$RECORD" \
        -H "Authorization: Bearer $TOKEN" \
        -H "Content-Type: application/json" \
        --data "$BODY"
fi
```

A cron job triggers this every 15 minutes:

```
MAILTO="root"
*/15 * * * *   root   /opt/ddns.sh
```

If the IP is updated, an email will be sent to the root user. 
