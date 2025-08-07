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

•importand command

```bash





