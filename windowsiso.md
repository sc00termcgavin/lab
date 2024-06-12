
### Step 1: [Download Windows 10 ISO image](https://www.microsoft.com/en-us/software-download/windows10ISO)

### Step 2: Find USB identifier 
> Run: `diskutil list` to find disk ID of microSD card
> Replace: `disk>N<`  with disk ID (`/dev/disk4` in my case)

```shell
diskutil list 
```

### Step 3: Format Drive
> format drive with GPT partition table under MS-DOS FAT32 file system
> Replace: `/dev/disk4` with your disk ID

```shell
diskutil eraseDisk MS-DOS "win10Drive" GPT /dev/disk4
```

### Step 4: Mount Windows 10 ISO as virtual volume

```shell
hdiutil mount ~/Downloads/Win10_22H2_English_x64v1.iso 
```

### Step 5: Copy files except for `install.wim` file as its too large for FAT32 file system
> `CCCOMA_X64_EN-US _DV9` is the mounted volume name of Windows 10 ISO

```shell
rsync -vha — exclude=sources/install.wim /Volumes/CCCOMA_X64_EN-US _DV9/* /Volumes/WIN10`
```

### Step 6: Install utility `wimlib` via homebrew, install [homebrew](https://brew.sh/) if needed

```shell
brew install wimlib
```

### Step 7: Split the `install.wim` file into pieces (<4[GB]) and copy to USB drive
> Replace `WIN10USB` with your USB drive name

- using the wimlib library to split install.wim into two pieces (<4GB) then copy it into the usb drive
```shell
wimlib-imagex split /Volumes/CCCOMA_X64_EN-US_DV9/sources/install.wim /Volumes/WIN10USB/sources/install.swm 2700
```

### Step 8: Eject USB and use as a bootable USB to install Windows 10 on a PC
> Replace `disk>N<` with your disk ID

```
sudo diskutil unmountDisk /dev/disk>N<
```