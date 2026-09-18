---
title: "Write Your Own Traceroute Tool"
author: "Mikey San"
date: 2023-05-04T13:05:58.243Z
lastmod: 2026-09-18T13:13:39+01:00

description: ""
subtitle: ""

image: "1.jpg" 
images:
 - "1.jpg"

aliases:
  - "/write-your-own-traceroute-tool-8f6b162a530e"

---


![](1.jpg)

Photo by [Radek Kilijanek](https://unsplash.com/@radek_blackseven?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

### **Preface:**

You’re working on a Linux-based system and you need to diagnose some network issues. What’s the first tool, well, one of the first, you think of? Traceroute, right? But there’s a catch — you can’t install it! Yes. True story. However, fear not, as I’ll walk you through the process of writing your own traceroute tool using Bash. It will provide you with a customisable, lightweight alternative.

Traceroute is a widely used network diagnostic tool that maps the route taken by packets across the network and the Internet to reach a target IP address. We’ll cover the fundamentals of how traceroute operates and teach you how to create a basic, yet functional, traceroute tool in Bash. We’ll then discuss testing and analysing the output, as well as potential improvements to make the tool even more powerful.

**Understanding the Basics of Traceroute**

Traceroute leverages Time-to-Live (TTL) values in IP packets to map the route taken by packets across the Internet. When TTL reaches zero, the router drops the packet and sends an ICMP Time Exceeded message. Traceroute manipulates this behaviour by sending packets with increasing TTL values, revealing the sequence of routers along the path. When the target is reached, it sends an ICMP Echo Reply message.

There are two common traceroute implementations: ICMP-based and UDP-based. While both are similar, routers and firewalls might treat ICMP and UDP packets differently, resulting in variations in the output.

**Another reason you may want your own Traceroute**

Working on systems with limited privileges can be challenging, especially when you need to diagnose network issues. By creating your own traceroute tool, you are able to work around these limitations and customise the tool to your liking, enhancing your knowledge of network protocols in the process.

**Enough talk, let’s write some code**

- Create a new Bash script file (e.g., simple\_trace.sh) and add the shebang at the beginning:

```bash
#!/bin/bash
```

- Check for correct arguments and provide usage messages:

```bash
if [ "$#" -ne 1 ]; then
echo "Usage: $0 target_ip_address"
exit 1
fi
```

- Define the target IP address, maximum number of hops, and initial TTL values:

```ini
TARGET_IP="$1"
MAX_HOPS=30
TTL=1
```

- Write a loop to send ICMP Echo Request packets with increasing TTL values:

```bash
while [ $TTL -le $MAX_HOPS ]; do
    OUTPUT=$(ping -c 1 -t $TTL $TARGET_IP)
    IP=$(echo "$OUTPUT" | grep -Eo "From [0-9.]* " | awk '{print $2}')
    RTT=$(echo "$OUTPUT" | grep -Eo "time=[0-9.]* ms" | awk -F= '{print $2}')

    if [ -n "$IP" ]; then
        echo "$TTL: $IP $RTT"
    else
        echo "$TTL: *"
    fi

    if [ "$IP" == "$TARGET_IP" ]; then
        break
    fi

    TTL=$((TTL + 1))
done
```

- Save and execute the script with:

```bash
chmod +x simple_trace.sh
./simple_trace.sh target_ip_address
```

**Testing and Analysing the Output**

- Run the custom traceroute tool using the command:

```bash
./simple_trace.sh target_ip_address
```

- Compare the output with existing traceroute tools. The results should be similar, but there might be slight differences due to ICMP and UDP packet handling by routers and firewalls.
- Troubleshoot common issues:
- Incomplete or missing output: Some routers or firewalls might filter ICMP packets, leading to gaps in the output. Try the built-in traceroute with the “-I” option or use a different implementation, like tcptraceroute.
- Timeouts and unreachable destinations: Increase the maximum number of hops (MAX\_HOPS) in the script or check for any network connectivity issues.
- Improve your custom traceroute tool by enhancing error handling, adding parallel probing, and supporting IPv6.

**Closing:**

Writing your own traceroute tool in Bash empowers you to overcome installation limitations while providing a customisable, lightweight alternative. This enables you to work around restrictions and also deepens your understanding of network protocols. So, embrace this opportunity to expand your skill set and develop a valuable diagnostic tool when traditional installations are off the table.
