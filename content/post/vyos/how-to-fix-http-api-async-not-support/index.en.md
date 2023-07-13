---
title: "How to Fix Http Api Async Not Support"
date: 2023-07-14T01:00:20+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=660e7c8f
categories:
  - vyos
tags:
  - vyos
  - api
  - shell
  - network
  - automation
slug:
  - vyos
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

For a long time ago, When I try to implement some automation job with vyos api, when mutiple client call a server side api, will make the server `config.boot` cut off.
When the server of vyos reboot, the configuration will broken and not working anymore.

After report the bug to vyos developer team. They tell me that this is the miss configration of http-api issue. But it can be patch by modify the file`/usr/libexec/vyos/services/vyos-http-api-server`.

After patch the api source code. it works well. Here is the solution.
## Create Script
```bash
tee  /config/scripts/patch_api_trunk_bug.sh << "EOF"
#!/bin/sh
set -xe
#Define the string which we want to search
string='^def retrieve_op(data: RetrieveModel):'

# Define the file path
file='/usr/libexec/vyos/services/vyos-http-api-server'

#To check the string is still there without modified
exist=$(grep -n "$string" ${file} | wc -l)

# Print exist value
echo ${exist}

# Function of find and modify
function find_and_modifiy {
  # Define line number of the string
  lineno=$(grep -n "$string" ${file} | cut -d: -f1)
  #Display line number
  echo ${lineno}
  #define commit string
  replace_string='#def retrieve_op(data: RetrieveModel):'
  echo ${replace_string}
  #Define the new string
  new_string='async def retrieve_op(data: RetrieveModel):'
  echo ${new_string}

  # Commit/replace the found line 
  sed -i "${lineno}c${replace_string}" ${file}

  # Insert new line below current line
  sed -i "${lineno}a${new_string}" ${file}
  # Print the lines which modified
  sed -n "${lineno},$(expr ${lineno} + 1)p" ${file}
  
  # Restart vyos-http-api service
  systemctl restart vyos-http-api

  # Check vyos-http-api serivce works or not
  systemctl status vyos-http-api
}

# Check the bug fixed or not
if [ ${exist} -ne 0 ] 
then
    echo 'find it!'
    find_and_modifiy

else
    echo 'not found!'
fi
EOF
```

## Run script to fix the bug
```
sudo bash /config/scripts/patch_api_trunk_bug.sh
```