---
title: "How to Generate Windows Reboot Task Scheduler"
date: 2023-12-22T22:55:42+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=78edc0d0
categories:
  - windows
tags:
  - powershell
  - ansible
  - task scheduler
  - windows
  - reboot
slug:
  - post
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---


## Powershell

```powershell
#create-reboot-task-scheduler.ps1
$ErrorActionPreference = 'Stop'

# Define the trigger (every Sunday at 6:00 AM)
$trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Sunday -At 6am

# Define the action to be performed (in this case, a system restart)
$action = New-ScheduledTaskAction -Execute 'shutdown' -Argument '/r /f /t 0'


# Register the task with the Task Scheduler
Register-ScheduledTask -TaskName "RebootTask" `
                       -Action $action `
                       -Trigger $trigger `
                       -Settings (New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries) `
                       -RunLevel Highest `
                       -User "NT AUTHORITY\SYSTEM" `
                       -Description "Reboot server every Sunday at 6am"

# Get task to confirm it was created
Get-ScheduledTask -TaskName "RebootTask"

```

## Ansible

```yaml
---
- name: Create Reboot Server Scheduler at every 5:00AM Sunday ON WINDOWS SERVER
  hosts: "windows"
  gather_facts: no

  tasks:
    - name: Create scheduled task
      win_scheduled_task:
        name: Reboot_Task
        description: "Reboot every Sunday at 6:00 AM"
        actions:
          - path: "C:\\Windows\\System32\\shutdown.exe"
            arguments: "/r /f /t 0"
        triggers:
          - type: weekly
            weeks_interval: 1
            days_of_week: SUNDAY
            start_boundary: '2023-12-17T06:00:00'
        enabled: yes
        state: present
        run_level: highest
        run_as_user: SYSTEM
        force: yes
        username: System
        password:
        logon_type: SERVICE_ACCOUNT
        disallow_start_if_on_batteries: false

    - name: Get details of the scheduled task status
      win_shell: Get-ScheduledTask -TaskName "Reboot_Task"  | Select-Object TaskName,State
      register: task_status

    - name: Dispaly Task Info
      debug:
        var: task_status.stdout_lines
      

```
