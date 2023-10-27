---
title: "How to Upgrade Dotnet Core Runtime With Ansible"
date: 2023-10-27T17:33:50+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=945df51e
categories:
  - Blog
tags:
  - ansible
  - dotnetcore
  - upgrade
slug:
  - ansible
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---


```yaml
---
- name: Upgrade .NET and .NET CORE VERSION ON WINDOWS SERVER
  hosts: "windows"
  gather_facts: no
  vars:
    #User Define
    # Find the information asp .net core hosting bundle from here https://dotnet.microsoft.com/en-us/download
    # If you want to upgrade .net 7 or 8. It will support. Change download_url, checksum and latest_dotnet_core_full_version
    download_url: https://download.visualstudio.microsoft.com/download/pr/34343c71-eb52-4537-b2b9-f25bc8b6c894/c6a39b3b387ad3f9662cd77c220902f5/dotnet-hosting-6.0.23-win.exe
    checksum: b0fab7d01aac899623fa5dc79940749ee58b29980d82d16e6aca794e323de46234b2263d70ba7e83cf989cb6c48d07e293f25a39b748e5a88874791566d38616
    latest_dotnet_core_full_version: 6.0.23
    
    # Don't change anything here,please.
    latest_dotnet_core_major_version: "{{ latest_dotnet_core_full_version.split('.')[0] }}"
    temp_dir: D:\installer
    checksum_algorithm: sha512
    installer: "{{ temp_dir }}\\dotnet-hosting-win.exe" 
    log_path: "{{ temp_dir }}\\dotnet-hosting-win.exe.log"    

  tasks:
    - name: Check if .NET Core is installed
      win_reg_stat:
        path: HKLM:\SOFTWARE\dotnet\Setup\InstalledVersions\x64\sharedhost
        name: Version
      register: dotnet_registry_key
      ignore_errors: yes

    - name: Extract .NET Core version if installed
      set_fact:
        installed_dotnet_core_full_version: "{{ dotnet_registry_key.value }}"
      when: dotnet_registry_key.exists

    - name: display installed_dotnet_core_full_version
      debug:
        msg: "{{installed_dotnet_core_full_version}}"
      when: dotnet_registry_key.exists

    - name: Extract installed dotnet core major version if installed 
      set_fact:
        installed_dotnet_core_major_version: "{{ installed_dotnet_core_full_version.split('.')[0] }}"
      when: dotnet_registry_key.exists

    - name: display installed dotnet core major version if installed 
      debug:
        msg: "{{installed_dotnet_core_major_version}}"
      when: dotnet_registry_key.exists

    - name: Check if installed .NET Core major version is {{ latest_dotnet_core_major_version }} 
      debug:
        msg: "Installed .NET Core major version is {{ latest_dotnet_core_major_version }}"
      when: 
        - dotnet_registry_key.exists
        - installed_dotnet_core_major_version is version(latest_dotnet_core_major_version, '=' ) 

    - name: .NET Core is not installed or not major version {{ latest_dotnet_core_major_version }} 
      debug:
        msg: ".NET Core is not installed or not  major version {{latest_dotnet_core_major_version}}"
      when: not(dotnet_registry_key.exists  and installed_dotnet_core_major_version is version(latest_dotnet_core_major_version, '=' ) )


    - name: Current installed .NET Core Version
      debug:
        msg: "Currently Install .net core version is {{ installed_dotnet_core_full_version}}"
      when:
        - dotnet_registry_key.exists
        - installed_dotnet_core_full_version is version(latest_dotnet_core_full_version, "<")

    - name: Download  dotnet  {{ latest_dotnet_core_full_version }} hosting bundle installer and checksum downloaded file
      win_get_url:
        url: "{{ download_url }}"
        dest: "{{ installer }}"
        checksum: "{{ checksum }}"
        checksum_algorithm: "{{ checksum_algorithm }}"
        force: True
      when:
        - dotnet_registry_key.exists
        - installed_dotnet_core_full_version is version(latest_dotnet_core_full_version, "<")

    - name: Install dotnet {{ latest_dotnet_core_full_version }} hosting bundle  installer 
      win_command: 
        "{{ installer }}  /install /quiet  /log  {{ log_path }}"
      become: yes
      when:
        - dotnet_registry_key.exists
        - installed_dotnet_core_full_version is version(latest_dotnet_core_full_version, "<")

    - name: Clean up dotnet  {{ latest_dotnet_core_full_version }} hosting bundle installer
      win_file:
        path:  "{{ installer }}"
        state: absent
      become: yes
      when:
        - dotnet_registry_key.exists
        - installed_dotnet_core_full_version is version(latest_dotnet_core_full_version, "<")

    - name: Check .NET Core Version in Registry
      ansible.windows.win_reg_stat:
        path: 'HKLM:\SOFTWARE\dotnet\Setup\InstalledVersions\x64\sharedhost'
        name: "Version"
      register: check_dotnetcore_version
      when:
        - dotnet_registry_key.exists
      
    - name: Check .NET Core Version after install finished
      debug:
        msg: "Currently Installed .net core version is {{ check_dotnetcore_version.value}}"
      when:
        - dotnet_registry_key.exists
```