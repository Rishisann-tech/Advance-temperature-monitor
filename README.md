# Advance-temperature-monitor
• This the advanced temperature monitor use too measure the temperature of your phone (cpu,gpu,battery,over-all) using termux 
# Requirements
• Termux apk latest version

• root access required
# Installation-process
• paste this command in termux 

**•First command-**

`Inline code` 

```bash
pkg update -y && pkg upgrade -y
pkg install -y termux-api python
pip install --upgrade pip
mkdir -p ~/.termux
echo 'alias temp="python ~/.termux/temperature.py"' >> ~/.bashrc
source ~/.bashrc

```

• if the upper command process is done without any error you can skip the second or third command

• But if you are getting a error like this than paste the second & third command

**ERROR- "Installing pip is forbidden, this will break the python-pip package"**

**• Second command-**

• clearing the previous attempt:-

`Inline code`
```bash
pkg remove python-pip
pkg autoremove

```
**•Third command**

```bash
pkg install python
python -m ensurepip --upgrade

```
**•Fourth command**

• Main comand 🙂

```bash
cat > ~/.termux/temperature.py << 'EOF'
#!/data/data/com.termux/files/usr/bin/python3

import os
import glob
import subprocess
from time import strftime

def run_root_cmd(cmd):
    """Execute a command as root with proper error handling"""
    try:
        result = subprocess.run(
            ['su', '-c', cmd],
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            text=True,
            timeout=2
        )
        return result.stdout.strip()
    except:
        return ""

def get_cpu_temp():
    """Get CPU temperature from various possible locations"""
    paths = [
        "/sys/devices/virtual/thermal/thermal_zone*/temp",
        "/sys/class/thermal/thermal_zone*/temp",
        "/sys/devices/system/cpu/cpu*/thermal_throttle/*_temp",
        "/sys/devices/platform/soc/*/thermal*/*/temp"
    ]
    
    temps = []
    for path_pattern in paths:
        for path in glob.glob(path_pattern):
            temp = run_root_cmd(f"cat {path}")
            if temp.isdigit():
                temps.append(float(temp)/1000)
    
    if temps:
        return f"{max(temps):.1f}°C"  # Return hottest CPU core
    return "N/A"

def get_gpu_temp():
    """Get GPU temperature with more comprehensive path checking"""
    # Additional GPU temp paths for different devices
    paths = [
        # Qualcomm devices
        "/sys/class/thermal/thermal_zone*/temp",
        "/sys/devices/virtual/thermal/thermal_zone*/temp",
        "/sys/devices/platform/soc/*gpu*/thermal*",
        "/sys/devices/platform/soc/*qcom,gpu*/thermal*",
        
        # Mali GPUs
        "/sys/devices/platform/*mali*/temp",
        "/sys/devices/platform/*mali*/thermal*",
        "/sys/class/misc/mali0/device/temp",
        
        # Adreno GPUs
        "/sys/class/kgsl/kgsl-3d0/temp",
        
        # Generic fallbacks
        "/sys/kernel/debug/thermal/thermal_zone*/temp",
        "/sys/devices/*/thermal/temp"
    ]
    
    # First try to find GPU-specific thermal zones
    zone_types = run_root_cmd("find /sys/class/thermal -name 'type' -exec cat {} \\; -exec echo \\ {} \\;")
    gpu_zones = []
    if zone_types:
        for line in zone_types.split('\n'):
            if 'gpu' in line.lower() or 'mali' in line.lower():
                zone_path = line.split()[1].replace('type', 'temp')
                gpu_zones.append(zone_path)
    
    # Check identified GPU zones first
    for zone in gpu_zones:
        temp = run_root_cmd(f"cat {zone}")
        if temp.isdigit():
            return f"{float(temp)/1000:.1f}°C"
    
    # Check all possible paths
    for path_pattern in paths:
        for path in glob.glob(path_pattern):
            temp = run_root_cmd(f"cat {path}")
            if temp and temp.isdigit():
                temp_c = float(temp)
                if temp_c > 1000:  # Handle millidegree values
                    temp_c = temp_c / 1000
                return f"{temp_c:.1f}°C"
    
    return "N/A (No GPU sensor found)"
    
def get_battery_temp():
    """Get battery temperature from various possible locations"""
    paths = [
        "/sys/class/power_supply/battery/temp",
        "/sys/class/power_supply/bms/temp",
        "/sys/class/power_supply/battery/batt_temp"
    ]
    
    for path in paths:
        temp = run_root_cmd(f"cat {path}")
        if temp:
            return f"{float(temp)/10:.1f}°C" if len(temp) > 2 else f"{temp}°C"
    
    # Fallback method
    temp = run_root_cmd("dumpsys battery | grep temperature | awk '{print $2}'")
    if temp.isdigit():
        return f"{float(temp)/10:.1f}°C"
    
    return "N/A"

def get_overall_temp():
    """Calculate overall device temperature"""
    temps = []
    
    # Get CPU temp
    cpu_temp = get_cpu_temp().replace('°C','')
    if cpu_temp.replace('.','').isdigit():
        temps.append(float(cpu_temp))
    
    # Get GPU temp
    gpu_temp = get_gpu_temp().replace('°C','')
    if gpu_temp.replace('.','').isdigit():
        temps.append(float(gpu_temp))
    
    # Get Battery temp
    batt_temp = get_battery_temp().replace('°C','')
    if batt_temp.replace('.','').isdigit():
        temps.append(float(batt_temp))
    
    if temps:
        avg_temp = sum(temps) / len(temps)
        return f"{avg_temp:.1f}°C"
    return "N/A"

def display_header():
    """Display the script header with creator info"""
    print("\033[1;35m" + "="*50)
    print("  Advanced Device Temperature Monitor")
    print(f"  Time: {strftime('%Y-%m-%d %H:%M:%S')}")
    print("  Created by Rishi Sann")
    print("   Wait for few seconds")
    print("="*50 + "\033[0m")

def main():
    """Main function to display temperature readings"""
    display_header()
    print(f"\033[1;32mCPU Temperature:\033[0m    {get_cpu_temp()}")
    print(f"\033[1;31mGPU Temperature:\033[0m    {get_gpu_temp()}")
    print(f"\033[1;33mBattery Temperature:\033[0m {get_battery_temp()}")
    print(f"\033[1;36mOverall Temperature:\033[0m {get_overall_temp()}")
    print("\033[1;35m" + "="*50 + "\033[0m")
    print("\033[1;36mNote: Requires root access\033[0m")

if __name__ == "__main__":
    main()
EOF

```
# How-To-Use
• exit and open the termux app 

• Type **temp** and enter

• u'll get the temperatures 🌡️ 

• Make sure u gave the root access

# Show-Case

!![Image Alt](https://github.com/Rishisann-tech/Advance-temperature-monitor/blob/58c9a3a0a257787c7792118d7525aaf63ad83463/Screenshot_20250807-233710_Termux.png)






