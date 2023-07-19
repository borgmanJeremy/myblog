+++ 
title = "UFW Monitoring in Grafana"
date = 2023-07-18
author = "Jeremy Borgman"
tags = ['Telegraf', 'Grafana', 'UFW', 'Ansible']
+++


# Introduction
My home network no longer exposes any ports thanks to tailscale. However, I have some services that family members need to access without tailscale. To support this I have a linode running tailscale + Caddy that proxies these requests to the home network. 

I use UFW as the firewall on this linode and wanted a way to gain insights into how many, and what kind of requests it was blocking.

I already use a telegraf + influxdb + grafana monitoring stack so the goal was to integrate into that.


# Setup
My setup was inspired by this [blog post](https://www.eliastiksofts.com/blog/2022/04/monitorer-son-serveur-avec-grafana-monitorer-le-pare-feu-ufw). This is only available in French so I think it's worth repeating some of the details. 

## Gather UFW statistics

UFW statistics can be gathered with the inputs.tail plugin of telegraf. Then the raw log can be parsed with grok before uploading into influx:

```
[[inputs.tail]]
    files = ["/var/log/ufw.log"]
    from_beginning = true
    name_override = "ufw_log"
    watch_method = "poll"
    grok_patterns = ["%{CUSTOM_LOG_FORMAT}"]
    grok_custom_patterns = '''
         CUSTOM_LOG_FORMAT %{SYSLOGTIMESTAMP:ufw_timestamp:ts-syslog} %{SYSLOGHOST:ufw_hostname} %{DATA:ufw_program}: \[%{DATA}\] \[UFW %{WORD:ufw_action}\] IN=%{DATA:ufw_interface} OUT=%{DATA:ufw_interface_out}( (MAC|PHYSIN)=%{DATA:ufw_mac})?SRC=%{IP:ufw_src_ip} DST=%{IP:ufw_dest_ip}( LEN=%{NUMBER:ufw_packet_len})? %{GREEDYDATA:ufw_tcp_opts} PROTO=%{WORD:ufw_protocol}( SPT=%{NUMBER:ufw_source_port})?( DPT=%{NUMBER:ufw_dest_port})?%{GREEDYDATA:ufw_tcp_opts}
    '''
    data_format = "grok"
    grok_timezone = "UTC"
    grok_custom_pattern_files = []
```

That looks like a mess but its straightforward once its broken down. The ufw log file is located at /var/log/ufw.log. We are going to watch that file and then parse the output using grok.

{{% notices info %}}
I lost a lot of time here. The ufw.log file may have unique permissions. On Ubuntu for instance its owned by user syslog and group adm. I added the telegraf user to the adm group so it could read this file.
{{% /notices %}}