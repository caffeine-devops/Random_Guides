# Auto-Mount Drives in Linux

In many Linux Distros and servers, non-bootable drives are not mounted when you boot into the system. They mount when you open them. But it is helpful in many cases to already have the drive mounted when you first boot up your machine.
For instance, if you have multiple drives in your system and have installed linux in only one drive but want the other drives as storage. In this case, when you boot up your system, the extra drives are not mounted automatically unless you access and mount it manually first.
To make life easier, we will auto mount the drives during boot up.

> [!warning]
> Doing this requires editing the fstab file in /etc
> Be careful when doing this
> Mistakes may cause boot/other issues

---

## Identify the Drive and Filesystem

You need to know which partition you are mounting and it's filesystem type.

```bash
sudo fdisk -l       # to see all drives and their partitions
sudo blkid          # Display info about available block devices
```

> Use `blkid` output to get the ***UUID*** of the partition and its filesystem type (`ext4`, `ntfs`, `vfat`, etc)

## Create a Mount Point

Choose or create a directory where the drive should appear. It is common practice to do this in `/media` folder.

```bash
sudo mkdir -p /media/Data       # you can name the directory anything else as well instead of Data
```

## Update /etc/fstab

Open the file using a text editor

```bash
sudo vi /etc/fstab
```

Add a new entry at the end with a format similar to

```text
# /media/Data (/dev/sdb1)
UUID=your-partition-uuid       /media/Data   ext4    defaults        0       0
```

- Replace `your-partition-uuid` with the actual UUID
- Replace `ext4` with the correct filesystem type
- These are **tab seperated** not **space seperated**

Once you have updated the `/etc/fstab` file with the data like above, make sure to save and quit

## Apply and test the configuration

Test the new fstab entry without rebooting

```bash
sudo mount -a
```

> [!tip]
> You might see an output asking you to run `sudo systemctl daemon-reload`. Do it!
>
> If there are no errors, your mount should be accessible.

You can verify this with

```bash
df -h | grep Data       # replace Data with whatever you have used
```

You can now reboot the system and run the above command to verify if it automounts on boot up

---

## Make Mounted Drives Easier to Access using Symlinks

> [!note]
> This section is optional

When mounting external or secondary drives in Linux, they're often mounted under `/media` or `/mnt` (in this case `/media/Data`).
But navigating to `/media/Data` every time is a bit inconvinient -- especially if you're working from your home directory.

Let's fix this by:

1. Giving your user access to the mounter folder
2. Creating a symbolix link (shortcut) in your home directory for easy access

### Make Sure You Own the Mounted Directory

By default, drives mounted to `/media` are owned by `root`, making it read-only for some users.
To give yourself full access:

```bash
sudo chown -R $USER:$USER /media/Data
```

- `-R` applies the ownership recursively
- `$USER` refers to your currently logged-in user

### Create a Symlink in your Home Directory

Now that you have Permission to access the drive, create a symbolic link to it

```bash
ln -s /media/Data ~/Data        # If you gave some other name to the mount point instead of Data, use it
```

This creates a new `Data` link in your home folder (`~`), pointing directly to the mounted drive

### What it Looks Like

Now from your home directory, if you run `cd Data`, you're in the same location as `/media/Data`, but without the long path
To verify:

```bash
ls -l ~ | grep Data

# you should see:
# Data -> /media/Data               or the name you gave
```

### Cleanup (Optional)

If you accidentally created a `Data/Data` situation (symlink inside a directory), just clean it up with

```bash
rm -rf ~/Data
ln -s /media/Data ~/Data
```
