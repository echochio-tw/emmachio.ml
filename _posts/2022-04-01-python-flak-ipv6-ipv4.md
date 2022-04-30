---
layout: post
title: flask ipv6 ipv4 
date: 2022-04-01
tags: python
---

flask 沒法用 try 只能用 re 判斷 ipv6 與 ipv4 紀錄一下

```
import re 
def validate_ip_address(address):
    match = re.match(r"^(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})$", address)
    if bool(match) is False:
        return validate_ipv6_address(address)
    for part in address.split("."):
        if int(part) < 0 or int(part) > 255:
            return validate_ipv6_address(address)
    return True

def validate_ipv6_address(address):
    ipv6match = re.match(r"^(([0-9a-fA-F]{1,4}:){7,7}[0-9a-fA-F]{1,4}|([0-9a-fA-F]{1,4}:){1,7}:|([0-9a-fA-F]{1,4}:){1,6}:[0-9a-fA-F]{1,4}|([0-9a-fA-F]{1,4}:){1,5}(:[0-9a-fA-F]{1,4}){1,2}|([0-9a-fA-F]{1,4}:){1,4}(:[0-9a-fA-F]{1,4}){1,3}|([0-9a-fA-F]{1,4}:){1,3}(:[0-9a-fA-F]{1,4}){1,4}|([0-9a-fA-F]{1,4}:){1,2}(:[0-9a-fA-F]{1,4}){1,5}|[0-9a-fA-F]{1,4}:((:[0-9a-fA-F]{1,4}){1,6})|:((:[0-9a-fA-F]{1,4}){1,7}|:)|fe80:(:[0-9a-fA-F]{0,4}){0,4}%[0-9a-zA-Z]{1,}|::(ffff(:0{1,4}){0,1}:){0,1}((25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])\.){3,3}(25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])|([0-9a-fA-F]{1,4}:){1,4}:((25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])\.){3,3}(25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9]))", address)
    if bool(ipv6match) is False:
        return False
    else:
        return True

print (validate_ip_address("1.1.1.1"))
print (validate_ip_address("2402:7500:901:af35:a574:c79b:bab1:3e4e"))
print (validate_ip_address('fe80::70ef:bfb0:6ddc:a150'))

```
