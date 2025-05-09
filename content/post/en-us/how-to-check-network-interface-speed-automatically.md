---
title: "How to Check Network Interface Speed Automatically"
date: 2025-05-09T18:37:13+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=6924806d
categories:
  - Blog
tags:
  - Hugo
slug:
  - test
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---


# **Keep Your Network Speeds Up to Par: Automated Monitoring and Correction on Linux**

Ever faced that frustrating moment when you realize your server's network interface has mysteriously dropped from a speedy 1Gbps or 2.5Gbps down to a sluggish 100Mbps, or even 10Mbps? This can happen for various reasons, from faulty cables and switch port issues to driver quirks. Manually checking and resetting these speeds is a pain and can lead to unnoticed performance degradation.

What if you could have a vigilant assistant that automatically monitors your critical network interfaces and, if their speed drops, not only alerts you but also attempts to set them back to their optimal performance?

Today, we're diving into a comprehensive bash script that does just that. This "NIC Speed Monitor" is designed to be set up as a systemd service on your Debian-based Linux system (and can be adapted for other distributions). It will periodically check your active network interfaces, and if it finds any running at an undesirable low speed, it will log the event, optionally send you an email alert, and try to reconfigure the interface to your desired speed.

## **What Does This Solution Do?**

The core of this solution is a master setup script that automates the deployment of:

1. **A Monitoring Script (monitor\_nic\_speeds.sh):** This is the workhorse. It scans active network interfaces (excluding virtual ones like lo, veth\*, and br-\*), checks their current speed using ethtool, and if an interface is running below a defined "OK" speed (e.g., 1000Mbps or 2500Mbps), it attempts to reconfigure it to a desired higher speed (e.g., 1000Mbps, full duplex, autonegotiation off). It also includes an alerting mechanism.  
2. **A systemd Service Unit (monitor\_nic\_speeds.service):** This defines how systemd should run the monitoring script.  
3. **A systemd Timer Unit (monitor\_nic\_speeds.timer):** This controls how often the service (and thus the monitoring script) is executed. You can configure it to run, for example, every 5 minutes.

## **Key Features**

* **Automated Detection:** Regularly checks the operational speed of physical network interfaces.  
* **Exclusion of Virtual Interfaces:** Intelligently ignores loopback, veth, and bridge interfaces where speed settings are not applicable.  
* **Configurable Speeds:** You can define what speeds are "OK" and what the "desired" speed should be if a correction is needed.  
* **Attempted Correction:** Uses ethtool to try and set the interface back to the desired speed and duplex settings.  
* **Alerting:** Can send an email alert (if configured) when an interface is found at an unexpectedly low speed (specifically 10/100 Mbps in the current version).  
* **Systemd Integration:** Runs reliably as a background service managed by systemd.  
* **Logging:** All actions and findings are logged to the systemd journal for easy review.  
* **Idempotent Setup:** The master setup script can be run multiple times, overwriting previous configurations safely.

## **Why Bother? The Benefits**

* **Proactive Issue Detection:** Catch speed drops before they cause significant performance impacts.  
* **Reduced Manual Intervention:** No more manually ssh-ing into servers to check and reset link speeds.  
* **Improved Network Reliability:** Helps maintain consistent network performance for critical services.  
* **Peace of Mind:** Get alerted if something goes wrong, allowing for quicker investigation of underlying hardware or configuration issues if the script can't permanently fix the speed (e.g., a bad cable).

## **The All-in-One Setup Script**

Below is the master script that will create the monitoring script and set up the systemd service and timer for you.

**Before running:**

* You'll run this script with sudo.  
* **Crucially, review the configuration variables** at the top of this setup script, especially MONITOR\_ALERT\_EMAIL, MONITOR\_ENABLE\_EMAIL\_ALERTS, and TIMER\_ON\_UNIT\_ACTIVE\_SEC.  
* If you want email alerts, ensure mailutils is installed (sudo apt install mailutils) and your system can send emails.

```bash
#!/bin/bash

# ==============================================================================
# MASTER SETUP SCRIPT
# Description:
#   This script automates the creation and setup of:
#     1. The NIC speed monitoring script (/usr/local/sbin/monitor_nic_speeds.sh)
#     2. A systemd service unit to run the monitoring script.
#     3. A systemd timer unit to trigger the service periodically.
#
# Usage:
#   Run this script with sudo: sudo ./setup_nic_monitor.sh
#
# Idempotency:
#   Running this script multiple times will overwrite existing files and
#   re-enable/restart the timer.
#
# Current Date for Reference: Friday, May 9, 2025
# ==============================================================================

# --- Configuration for the monitoring script (ADJUST THESE AS NEEDED) ---
MONITOR_DESIRED_SPEED_MBPS="1000"
MONITOR_DESIRED_DUPLEX="full"
MONITOR_AUTONEG_SETTING="off"
# Define OK_SPEEDS_MBPS as a bash array for the master script
MONITOR_OK_SPEEDS_MBPS_ARRAY=("1000" "2500")
MONITOR_FORCE_CHANGE_IF_UNSURE="false"
MONITOR_ALERT_EMAIL="your_email@example.com" # IMPORTANT: Change this!
MONITOR_ENABLE_EMAIL_ALERTS="false"         # Set to true to enable email alerts

# --- Paths for generated files ---
MONITOR_SCRIPT_PATH="/usr/local/sbin/monitor_nic_speeds.sh"
SYSTEMD_SERVICE_PATH="/etc/systemd/system/monitor_nic_speeds.service"
SYSTEMD_TIMER_PATH="/etc/systemd/system/monitor_nic_speeds.timer"
SYSTEMD_SERVICE_NAME="monitor_nic_speeds.service"
SYSTEMD_TIMER_NAME="monitor_nic_speeds.timer"

# --- Timer Configuration (how often the monitoring script runs) ---
# Runs 1 minute after boot, and then every 5 minutes.
# Adjust OnUnitActiveSec for different frequencies (e.g., 1min, 15min, 1h).
TIMER_ON_BOOT_SEC="1min"
TIMER_ON_UNIT_ACTIVE_SEC="5min"

# --- Helper function for logging within this master script ---
log_master() {
    echo "[SETUP_NIC_MONITOR] $(date -u '+%Y-%m-%d %H:%M:%S %Z') - $1"
}

# ==============================================================================
# MAIN SETUP FUNCTION
# ==============================================================================
main() {
    log_master "Starting automated setup of NIC speed monitor."

    # Check for root privileges
    if [[ $EUID -ne 0 ]]; then
       log_master "ERROR: This setup script must be run as root or with sudo."
       exit 1
    fi

    # Stop the timer if it's already running, to prevent issues during file overwrite
    log_master "Attempting to stop existing timer ($SYSTEMD_TIMER_NAME) if active..."
    if systemctl is-active --quiet "$SYSTEMD_TIMER_NAME"; then
        systemctl stop "$SYSTEMD_TIMER_NAME"
        log_master "$SYSTEMD_TIMER_NAME stopped."
    else
        log_master "$SYSTEMD_TIMER_NAME is not active or does not exist yet."
    fi

    # 1. Create the monitoring script
    create_monitoring_script

    # 2. Create the systemd service unit
    create_systemd_service_unit

    # 3. Create the systemd timer unit
    create_systemd_timer_unit

    # 4. Reload systemd, enable and start the timer
    setup_systemd_units

    log_master "----------------------------------------------------------------------"
    log_master "Setup complete!"
    log_master "The NIC speed monitor service is configured and should be active."
    log_master "To check timer status: sudo systemctl status $SYSTEMD_TIMER_NAME"
    log_master "To check service logs: sudo journalctl -u $SYSTEMD_SERVICE_NAME -f"
    log_master "Monitoring script is at: $MONITOR_SCRIPT_PATH"
    log_master "Service unit is at: $SYSTEMD_SERVICE_PATH"
    log_master "Timer unit is at: $SYSTEMD_TIMER_PATH"
    log_master "----------------------------------------------------------------------"
}

# ==============================================================================
# FUNCTION TO CREATE THE MONITORING SCRIPT
# (/usr/local/sbin/monitor_nic_speeds.sh)
# ==============================================================================
create_monitoring_script() {
    log_master "Creating monitoring script at $MONITOR_SCRIPT_PATH..."

    # Convert MONITOR_OK_SPEEDS_MBPS_ARRAY to a string format suitable for the script
    local ok_speeds_string_for_script
    ok_speeds_string_for_script=$(printf '"%s" ' "${MONITOR_OK_SPEEDS_MBPS_ARRAY[@]}")
    # Result: "1000" "2500" (note the trailing space)
    ok_speeds_string_for_script="(${ok_speeds_string_for_script% })" # Add parentheses and remove last space
    # Result: ("1000" "2500")

    # Using a heredoc to write the script content.
    # Variables from this master script are expanded into the heredoc.
    # Dollar signs ($) that should be literal in the final script are escaped (\$).
    cat > "$MONITOR_SCRIPT_PATH" << EOF
#!/bin/bash

# This script is automatically generated. Do not edit directly unless you know what you're doing.
# Source configuration is in the master setup script.

# --- Configuration (from master setup script) ---
DESIRED_SPEED_MBPS="${MONITOR_DESIRED_SPEED_MBPS}"
DESIRED_DUPLEX="${MONITOR_DESIRED_DUPLEX}"
AUTONEG_SETTING="${MONITOR_AUTONEG_SETTING}"
OK_SPEEDS_MBPS=${ok_speeds_string_for_script} # This will be like ("1000" "2500")
FORCE_CHANGE_IF_UNSURE="${MONITOR_FORCE_CHANGE_IF_UNSURE}"
ALERT_EMAIL="${MONITOR_ALERT_EMAIL}"
ENABLE_EMAIL_ALERTS="${MONITOR_ENABLE_EMAIL_ALERTS}"
# --- End of Configuration ---

# Function to log messages with a timestamp
log_msg() {
    # Using UTC for logs for consistency across timezones
    echo "\$(date -u '+%Y-%m-%d %H:%M:%S %Z') - \$1"
}

# Function to send an alert
send_alert() {
    local interface_name="\$1"
    local current_speed="\$2"
    local subject="Network Speed Alert: Interface \$interface_name at \${current_speed}Mb/s"
    local body="Interface \$interface_name on host \$(hostname) has dropped to \${current_speed}Mb/s at \$(date -u '+%Y-%m-%d %H:%M:%S %Z')."

    log_msg "ALERT_TRIGGERED: \$body"

    if \$ENABLE_EMAIL_ALERTS; then
        if command -v mail &> /dev/null; then
            echo "\$body" | mail -s "\$subject" "\$ALERT_EMAIL"
            log_msg "Email alert sent to \$ALERT_EMAIL for \$interface_name."
        else
            log_msg "Warning: 'mail' command not found. Cannot send email alert. Please install mailutils (sudo apt install mailutils)."
        fi
    fi
}

# 1. Pre-flight Checks
log_msg "Starting network interface speed check and adjustment process."
if ! command -v ethtool &> /dev/null; then
    log_msg "FATAL: ethtool is not installed. Please install it (e.g., sudo apt install ethtool)."
    exit 1
fi
if ! command -v ip &> /dev/null; then
    log_msg "FATAL: ip command (from iproute2) not found. This is highly unusual for a Debian system."
    exit 1
fi
# This script is run by systemd as root, so no EUID check needed here.

# 2. Get all network interfaces that are UP (excluding loopback, veth, and br-)
log_msg "Discovering active (UP) network interfaces (excluding lo, veth*, and br-*)..."
UP_INTERFACES=()
while IFS= read -r line; do
    IFACE=\$(echo "\$line" | awk '{print \$1}')
    STATE=\$(echo "\$line" | awk '{print \$2}')
    if [[ "\$IFACE" != "lo" && "\$STATE" == "UP" && "\$IFACE" != veth* && "\$IFACE" != br-* ]]; then
        UP_INTERFACES+=("\$IFACE")
    fi
done < <(ip -br link)

if [ \${#UP_INTERFACES[@]} -eq 0 ]; then
    log_msg "No suitable active (UP) network interfaces found (excluding lo, veth*, and br-*). Exiting."
    exit 0
fi
log_msg "Found suitable active interfaces to check: \${UP_INTERFACES[*]}"
echo

CHANGES_ATTEMPTED=false

# 3. Loop through each suitable UP interface and apply logic
for INTERFACE_NAME in "\${UP_INTERFACES[@]}"; do
    log_msg "--- Processing interface: \$INTERFACE_NAME ---"

    ETHTOOL_OUTPUT=\$(ethtool "\$INTERFACE_NAME" 2>/dev/null)
    if [ \$? -ne 0 ]; then
        log_msg "Warning: ethtool command failed for \$INTERFACE_NAME or it does not support ethtool queries. Skipping."
        log_msg "--- Finished processing interface: \$INTERFACE_NAME ---"; echo
        continue
    fi

    CURRENT_SPEED_FULL=\$(echo "\$ETHTOOL_OUTPUT" | grep "^\s*Speed:" | awk '{print \$2}')
    CURRENT_SPEED_NUMERIC=""

    if [[ -z "\$CURRENT_SPEED_FULL" ]]; then
        log_msg "Warning: Could not determine current speed line for \$INTERFACE_NAME."
        CURRENT_SPEED_NUMERIC="0"
    elif [[ "\$CURRENT_SPEED_FULL" == "Unknown!" ]]; then
        log_msg "Warning: Link speed for \$INTERFACE_NAME is Unknown! (Link likely down or not negotiated)."
        CURRENT_SPEED_NUMERIC="0"
    else
        CURRENT_SPEED_NUMERIC=\$(echo "\$CURRENT_SPEED_FULL" | sed 's/[^0-9]*//g')
        if [[ -z "\$CURRENT_SPEED_NUMERIC" ]]; then
            log_msg "Warning: Could not parse numeric speed from '\$CURRENT_SPEED_FULL' for \$INTERFACE_NAME."
            CURRENT_SPEED_NUMERIC="0"
        else
            log_msg "Current speed of \$INTERFACE_NAME: \${CURRENT_SPEED_NUMERIC}Mb/s"
        fi
    fi

    if [[ "\$CURRENT_SPEED_NUMERIC" == "10" || "\$CURRENT_SPEED_NUMERIC" == "100" ]]; then
        send_alert "\$INTERFACE_NAME" "\$CURRENT_SPEED_NUMERIC"
    fi

    IS_OK_SPEED=false
    for ok_speed in "\${OK_SPEEDS_MBPS[@]}"; do # Iterate over the array elements
        if [[ "\$CURRENT_SPEED_NUMERIC" == "\$ok_speed" ]]; then
            IS_OK_SPEED=true
            break
        fi
    done

    if ! \$IS_OK_SPEED; then
        log_msg "\$INTERFACE_NAME speed is \${CURRENT_SPEED_FULL:-Not determined/Link down}. This is not one of the desired speeds (\${OK_SPEEDS_MBPS[*]} Mb/s)."
        log_msg "Attempting to change \$INTERFACE_NAME to \$DESIRED_SPEED_MBPS Mb/s, \$DESIRED_DUPLEX duplex, autoneg \$AUTONEG_SETTING."
        CHANGES_ATTEMPTED=true

        SUPPORTED_LINK_MODES_OUTPUT=\$(echo "\$ETHTOOL_OUTPUT" | grep "Supported link modes:")
        DESIRED_MODE_PATTERN_FULL="\${DESIRED_SPEED_MBPS}baseT/Full"
        DESIRED_MODE_PATTERN_HALF="\${DESIRED_SPEED_MBPS}baseT/Half"
        SPEED_SUPPORTED_IN_MODES=false
        if echo "\$SUPPORTED_LINK_MODES_OUTPUT" | grep -qE "(\$DESIRED_MODE_PATTERN_FULL|\$DESIRED_MODE_PATTERN_HALF|\${DESIRED_SPEED_MBPS}base)"; then
            SPEED_SUPPORTED_IN_MODES=true
        elif echo "\$ETHTOOL_OUTPUT" | grep -q "\${DESIRED_SPEED_MBPS}Mb/s"; then
            SPEED_SUPPORTED_IN_MODES=true
        fi

        if ! \$SPEED_SUPPORTED_IN_MODES && [[ ! -z "\$SUPPORTED_LINK_MODES_OUTPUT" ]]; then
            log_msg "Warning: \$INTERFACE_NAME 'Supported link modes' do not clearly list \${DESIRED_SPEED_MBPS}Mb/s."
            if \$FORCE_CHANGE_IF_UNSURE; then
                log_msg "FORCE_CHANGE_IF_UNSURE is true. Proceeding with change attempt for \$INTERFACE_NAME."
            else
                log_msg "FORCE_CHANGE_IF_UNSURE is false. Skipping speed change for \$INTERFACE_NAME."
                log_msg "--- Finished processing interface: \$INTERFACE_NAME ---"; echo
                continue
            fi
        fi

        if ethtool -s "\$INTERFACE_NAME" autoneg "\$AUTONEG_SETTING" speed "\$DESIRED_SPEED_MBPS" duplex "\$DESIRED_DUPLEX"; then
            log_msg "Successfully changed settings for \$INTERFACE_NAME."
            log_msg "New settings for \$INTERFACE_NAME (verification):"
            ethtool "\$INTERFACE_NAME" | grep -E "^\s*Speed:|^\s*Duplex:|^\s*Auto-negotiation:"
        else
            log_msg "Error: Failed to change settings for \$INTERFACE_NAME."
        fi
    else
        log_msg "\$INTERFACE_NAME is already at an acceptable speed (\${CURRENT_SPEED_NUMERIC}Mb/s). No changes needed."
    fi
    log_msg "--- Finished processing interface: \$INTERFACE_NAME ---"; echo
done

log_msg "All interface processing complete."
if \$CHANGES_ATTEMPTED; then
    echo
    log_msg "############################################################################"
    log_msg "IMPORTANT: Changes made by this script are TEMPORARY and will be lost on"
    log_msg "reboot. To make them permanent, configure your system's network management"
    log_msg "tool (systemd-networkd, NetworkManager via nmcli, or /etc/network/interfaces)"
    log_msg "appropriately for Debian 12."
    log_msg "############################################################################"
fi

exit 0
EOF

    # Make the monitoring script executable
    chmod +x "$MONITOR_SCRIPT_PATH"
    log_master "Monitoring script created and made executable at $MONITOR_SCRIPT_PATH."
}

# ==============================================================================
# FUNCTION TO CREATE THE SYSTEMD SERVICE UNIT
# (/etc/systemd/system/monitor_nic_speeds.service)
# ==============================================================================
create_systemd_service_unit() {
    log_master "Creating systemd service unit at $SYSTEMD_SERVICE_PATH..."
    # Using 'EOF' (quoted) for systemd units to prevent shell expansion within the heredoc
    cat > "$SYSTEMD_SERVICE_PATH" << 'EOF'
[Unit]
Description=Network Interface Speed Monitor
Documentation=file:///usr/local/sbin/monitor_nic_speeds.sh
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/monitor_nic_speeds.sh
# Output will go to systemd journal by default

[Install]
WantedBy=multi-user.target
EOF
    log_master "Systemd service unit created at $SYSTEMD_SERVICE_PATH."
}

# ==============================================================================
# FUNCTION TO CREATE THE SYSTEMD TIMER UNIT
# (/etc/systemd/system/monitor_nic_speeds.timer)
# ==============================================================================
create_systemd_timer_unit() {
    log_master "Creating systemd timer unit at $SYSTEMD_TIMER_PATH..."
    # Using 'EOF' (quoted) for systemd units
    # The timer values are injected from master script variables
    cat > "$SYSTEMD_TIMER_PATH" << EOF
[Unit]
Description=Run Network Interface Speed Monitor periodically
Documentation=file:///usr/local/sbin/monitor_nic_speeds.sh

[Timer]
# Timer settings from master setup script:
OnBootSec=${TIMER_ON_BOOT_SEC}
OnUnitActiveSec=${TIMER_ON_UNIT_ACTIVE_SEC}

# AccuracySec=1s # Optional: How long the timer can be delayed
Persistent=true # Optional: Run missed jobs if system was down

Unit=${SYSTEMD_SERVICE_NAME}

[Install]
WantedBy=timers.target
EOF
    log_master "Systemd timer unit created at $SYSTEMD_TIMER_PATH."
}

# ==============================================================================
# FUNCTION TO RELOAD SYSTEMD, ENABLE AND START THE TIMER
# ==============================================================================
setup_systemd_units() {
    log_master "Reloading systemd daemon..."
    systemctl daemon-reload

    log_master "Enabling $SYSTEMD_TIMER_NAME to start on boot..."
    systemctl enable "$SYSTEMD_TIMER_NAME"

    log_master "Starting $SYSTEMD_TIMER_NAME now..."
    systemctl start "$SYSTEMD_TIMER_NAME"

    log_master "Systemd units reloaded. Timer $SYSTEMD_TIMER_NAME is enabled and started."
}

# --- Execute main setup function ---
main

```
## **How to Use the Setup Script**

1. **Save the Script:** Copy the entire "All-in-One Setup Script" above and save it to a file on your Linux server, for example, setup\_nic\_monitor.sh.  
2. **Make it Executable:**  
   chmod \+x setup\_nic\_monitor.sh

3. **Review Configuration:** Open the setup\_nic\_monitor.sh script with a text editor (nano setup\_nic\_monitor.sh) and **carefully adjust the configuration variables** at the top. The most important ones are:  
   * MONITOR\_ALERT\_EMAIL: Your email address for alerts.  
   * MONITOR\_ENABLE\_EMAIL\_ALERTS: Set to true to get emails.  
   * TIMER\_ON\_UNIT\_ACTIVE\_SEC: How often the check runs (e.g., 5min, 10min).  
4. **Run the Setup Script:**  
   sudo ./setup\_nic\_monitor.sh

   The script will output its progress.

## **After Setup**

Once the setup script completes, the monitor is active\!

* **Check Timer Status:** sudo systemctl status monitor\_nic\_speeds.timer  
* **View Logs:** sudo journalctl \-u monitor\_nic\_speeds.service \-f (this shows the output from the monitoring script itself each time it runs).

## **Important Note on Persistence of Fixes**

The monitoring script uses ethtool to attempt to correct the speed. These ethtool changes are generally **temporary** and might be lost on reboot or if a network management service (like NetworkManager) reasserts its own configuration.

For a truly permanent fix to recurring speed drops, you should investigate the root cause (cabling, switch port, NIC hardware/firmware). If the issue is a configuration one that can be set permanently via NetworkManager or systemd-networkd, that would be the preferred long-term solution. This script acts as a valuable watchdog and first-responder.




# Uninstalling the Service
If you no longer need the NIC speed monitor, you can easily remove it. The following script will stop the service, disable the timer, and remove all related files created by the setup script.

Uninstallation Script:

```bash
#!/bin/bash

# ==============================================================================
# UNINSTALL SCRIPT
# Description:
#   This script automates the uninstallation of the NIC speed monitoring
#   service and its related files.
#
# Usage:
#   Run this script with sudo: sudo ./uninstall_nic_monitor.sh
#
# Current Date for Reference: Friday, May 9, 2025
# ==============================================================================

# --- Paths for files to be removed ---
MONITOR_SCRIPT_PATH="/usr/local/sbin/monitor_nic_speeds.sh"
SYSTEMD_SERVICE_PATH="/etc/systemd/system/monitor_nic_speeds.service"
SYSTEMD_TIMER_PATH="/etc/systemd/system/monitor_nic_speeds.timer"
SYSTEMD_TIMER_NAME="monitor_nic_speeds.timer"

# --- Helper function for logging ---
log_uninstall() {
    echo "[UNINSTALL_NIC_MONITOR] $(date -u '+%Y-%m-%d %H:%M:%S %Z') - $1"
}

# ==============================================================================
# MAIN UNINSTALL FUNCTION
# ==============================================================================
main() {
    log_uninstall "Starting uninstallation of NIC speed monitor service."

    # Check for root privileges
    if [[ $EUID -ne 0 ]]; then
       log_uninstall "ERROR: This uninstallation script must be run as root or with sudo."
       exit 1
    fi

    # 1. Stop and Disable the systemd Timer
    log_uninstall "Stopping $SYSTEMD_TIMER_NAME..."
    if systemctl is-active --quiet "$SYSTEMD_TIMER_NAME"; then
        systemctl stop "$SYSTEMD_TIMER_NAME"
        log_uninstall "$SYSTEMD_TIMER_NAME stopped."
    else
        log_uninstall "$SYSTEMD_TIMER_NAME was not active."
    fi

    log_uninstall "Disabling $SYSTEMD_TIMER_NAME..."
    if systemctl is-enabled --quiet "$SYSTEMD_TIMER_NAME"; then
        systemctl disable "$SYSTEMD_TIMER_NAME"
        log_uninstall "$SYSTEMD_TIMER_NAME disabled."
    else
        log_uninstall "$SYSTEMD_TIMER_NAME was not enabled."
    fi

    # 2. Remove the systemd Unit Files
    log_uninstall "Removing systemd unit files..."
    if [ -f "$SYSTEMD_SERVICE_PATH" ]; then
        rm "$SYSTEMD_SERVICE_PATH"
        log_uninstall "Removed $SYSTEMD_SERVICE_PATH."
    else
        log_uninstall "$SYSTEMD_SERVICE_PATH not found."
    fi

    if [ -f "$SYSTEMD_TIMER_PATH" ]; then
        rm "$SYSTEMD_TIMER_PATH"
        log_uninstall "Removed $SYSTEMD_TIMER_PATH."
    else
        log_uninstall "$SYSTEMD_TIMER_PATH not found."
    fi

    # 3. Reload the systemd Daemon
    log_uninstall "Reloading systemd daemon..."
    systemctl daemon-reload
    log_uninstall "Systemd daemon reloaded."

    # 4. Remove the Monitoring Script
    log_uninstall "Removing the monitoring script..."
    if [ -f "$MONITOR_SCRIPT_PATH" ]; then
        rm "$MONITOR_SCRIPT_PATH"
        log_uninstall "Removed $MONITOR_SCRIPT_PATH."
    else
        log_uninstall "$MONITOR_SCRIPT_PATH not found."
    fi

    log_uninstall "----------------------------------------------------------------------"
    log_uninstall "Uninstallation complete."
    log_uninstall "The NIC speed monitor service and related files should be removed."
    log_uninstall "----------------------------------------------------------------------"
}

# --- Execute main uninstall function ---
main

```


### How to Use the Uninstallation Script:
  1. Save the Script: Copy the "Uninstallation Script" above and save it to a file, for example, uninstall_nic_monitor.sh.
  2. Make it Executable:
   ```shell
   chmod +x uninstall_nic_monitor.sh
   ```
  3. Run the Uninstallation Script:
   ```bash
   sudo ./uninstall_nic_monitor.sh
   ```
