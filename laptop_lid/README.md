# Disable Suspend on Lid Close for laptops

To do this, we will need to edit the Systemd login configuration file

```bash
sudo vi /etc/systemd/logind.con
```

**Uncomment** the following keys and change it's values to **ignore**

```text
HandleSuspendKey=ignore
HandleLidSwitch=ignore
HandleLidSwitchDocked=ignore
HandleLidSwitchExternalPower=ignore
```

After saving the changes, we need to run the below command for these changes to take place

```bash
sudo systemctl restart systemd-logind
```

Now you can close the lid of your laptop and it won't suspend
