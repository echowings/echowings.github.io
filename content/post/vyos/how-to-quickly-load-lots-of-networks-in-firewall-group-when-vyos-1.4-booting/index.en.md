---
title: "How to Quickly Load Lots of Networks in Firewall Group When Vyos 1.4 Booting"
date: 2024-03-11T16:16:58+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=d8d1c543
categories:
  - network
tags:
  - vyos
  - bash
  - pbr
  - nftable
  - shell
  - networking
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

## Download  file `/config/scripts/chinaipranges.txt`

```shell
curl https://raw.githubusercontent.com/pmkol/easymosdns/rules/china_ip_list.txt | sed -En '/([0-9]+\.){3}[0-9]+/{s,,&,p}' >  /config/scripts/chinaipranges.txt
```

## Create file `/config/scripts/add-firewall-group.py`

```shell
tee  /config/scripts/add-firewall-group.py << "EOF"
#!/usr/bin/env python3
import subprocess
import sys
import time

def add_networks_in_batches(set_name, filename, batch_size=1000):
    #table_name = "vyos_filter"  # Fixed table name
    table_names = ["vyos_filter", "vyos_mangle", "vyos_nat", "vyos_conntrack"]
    total_added = 0  # Initialize counter for total added IPs

    start_time = time.time()  # Record the start time

    try:
        with open(filename, 'r') as file:
            networks = [line.strip() for line in file if line.strip()]

        for table_name in table_names:
          for i in range(0, len(networks), batch_size):
            batch_networks = networks[i:i+batch_size]
            networks_string = ', '.join(batch_networks)
            command = f"nft add element ip {table_name} {set_name} {{ {networks_string} }}"

            subprocess.run(command, check=True, shell=True)
            print(f"Successfully added batch of networks to {set_name}")
            total_added += len(batch_networks)  # Update count of added IPs

    except subprocess.CalledProcessError as e:
        print(f"Error adding networks to {set_name}: {e}")
    except FileNotFoundError:
        print(f"File {filename} not found")
    except Exception as e:
        print(f"An error occurred: {e}")

    end_time = time.time()  # Record the end time
    elapsed_time = end_time - start_time  # Calculate elapsed time

    # Print summary
    print(f"\nTotal of {total_added} networks were added to {set_name}.")
    print(f"Process took {elapsed_time:.2f} seconds.")

def main():
    if len(sys.argv) != 3:
        print("Usage: python3 test.py <set name> <filename>")
        sys.exit(1)

    set_name = sys.argv[1]
    filename = sys.argv[2]

    add_networks_in_batches(set_name, filename)

if __name__ == "__main__":
    main()
EOF
```


## Add loading ip script in  `/config/scripts/vyos-postconfig-bootup.script`

```shell
if ! grep -q "cd /config/scripts &&python3 add-firewall-group.py  N_china-ip-ranges chinaipranges.txt"  /config/scripts/vyos-postconfig-bootup.script
then
  echo "Not existed, append file"
  echo "cd /config/scripts &&python3 add-firewall-group.py  N_china-ip-ranges chinaipranges.txt" >> /config/scripts/vyos-postconfig-bootup.script
else
  echo "Already existed"
fi
```
## Check  network group command

```shell
# List all tables has network-group
sudo nft list sets | grep -B  1  chatgpt-ip-ranges
```

```shell
# List 
sudo nft list set ip vyos_filter N_china-ip-ranges
sudo nft list set ip vyos_mangle N_china-ip-ranges
sudo nft list set ip vyos_nat N_china-ip-ranges
sudo nft list set ip vyos_conntrack N_china-ip-ranges
```

## Reference
  - [How to load networks from a file to add them into the firewall group in vyos 1.4 with ntf command to replace removed ipset ](https://forum.vyos.io/t/how-to-load-networks-from-a-file-to-add-them-into-the-firewall-group-in-vyos-1-4-with-ntf-command-to-replace-removed-ipset/13997/2)