- [Block tor](#block-tor)
  - [Init script](#init-script)
  - [Firewall script](#firewall-script)
  - [Schedule script](#schedule-script)
- [Block shodan, stretchoid and binary-edge](#block-shodan-stretchoid-and-binary-edge)
  - [Init script](#init-script-1)
  - [Firewall script](#firewall-script-1)
  - [Schedule script](#schedule-script-1)
- [Verify / debug](#verify--debug)
- [Todo](#todo)
- [Acknowledgements](#acknowledgements)

# Block tor
## Init script
Under `Administration -> Scripts -> Init` add
```bash
modprobe -a ip_set ip_set_hash_ip xt_set
ipset create tor hash:ip
```

## Firewall script
Under `Administration -> Scripts -> Firewall` add
```bash
iptables -I FORWARD -m set --match-set tor src -j DROP
```

## Schedule script
Then to update populate the ipset with ips add the following script under `Administration -> Scripts -> Scheduler`
```bash
set -e
logger -p "info" "Fetching new ip's to add"

result="`wget -qO- https://check.torproject.org/exit-addresses`"
logger -p "info" "Fetched ${#result} bytes"

logger -p "info" "Removing all ip's from tor ipset"
ipset flush tor

echo "$result" |
while IFS= read -r line; do
    match=$(echo "$line" | sed -n 's/ExitAddress \([0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+\).*/\1/p')
    if [ ! -z "$match" ]
    then
        ipset -! add tor "$match"
    fi
done

logger -p "info" "Done creating tor ipset"
```




# Block shodan, stretchoid and binary-edge

## Init script
Under `Administration -> Scripts -> Init` add
```bash
modprobe -a ip_set ip_set_hash_ip xt_set
ipset create shodan hash:ip
ipset create stretchoid hash:ip
ipset create binary-edge hash:ip
ipset create other hash:ip
```

## Firewall script
Under `Administration -> Scripts -> Firewall` add
```bash
iptables -I FORWARD -m set --match-set shodan src -j DROP
iptables -I FORWARD -m set --match-set stretchoid src -j DROP
iptables -I FORWARD -m set --match-set binary-edge src -j DROP
iptables -I FORWARD -m set --match-set other src -j DROP
```

## Schedule script
Then to update populate the ipset with ips add the following script under `Administration -> Scripts -> Scheduler`
```bash
set -e
update_ipset()
{
    name=$1
    url=$2
    logger -p "info" "Fetching new ip's to add to $name"
    result="`wget -qO- $url`"
    logger -p "info" "Fetched ${#result} bytes"
    logger -p "info" "Removing all ip's from $name ipset"
    ipset flush tor
    echo "$result" |
    while IFS= read -r line; do
        match=$(echo "$line" | sed -n 's/\([0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+\).*/\1/p')
        if [ ! -z "$match" ]
        then
            ipset -! add "$name" "$match"
        fi
    done
}
update_ipset "shodan" "https://raw.githubusercontent.com/SilvrrGIT/IP-Lists/master/shodan"
update_ipset "stretchoid" "https://raw.githubusercontent.com/SilvrrGIT/IP-Lists/master/stretchoid"
update_ipset "binary-edge" "https://raw.githubusercontent.com/SilvrrGIT/IP-Lists/master/binary_edge"
update_ipset "other" "https://raw.githubusercontent.com/SilvrrGIT/IP-Lists/master/other"
```

# Verify / debug
To verify that the the scripts add log entries to `Status > Logs`, you can also `ssh` or `telnet` into the router and run `ipset list [name]`.

# Todo
* The ipset's currently get cleared when you reboot your router.

# Acknowledgements
Thanks to [SilvrrGIT](https://www.github.com/SilvrrGIT) for having awesome ip block lists on his github.




![](https://visitor-badge.glitch.me/badge?page_id=tomato-block-malicious-ips)