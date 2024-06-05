# JdmTool

A command-line tool for downloading and transferring Jeppesen databases aiming to be compatible with [Jeppesen Distribution Manager](https://ww2.jeppesen.com/data-solutions/jeppesen-distribution-manager/).

It requires a Jeppesen subscription, and currenty supports the following services:
- NavData for Garmin GNS 400/500 Series
  - Requires a Skybound data card programmer (USB ID `0e39:1250`)
  - Requires a 16MB NavData WAAS card or a 4MB NavData non-WAAS card
    - If you have an 8MB data card, please [file a bug](https://github.com/dimaryaz/jdmtool/issues/)!
- NavData and Obstacles for Avidyne IFD 400 Series
- (Very experimental) Garmin G1000 support, except for Electronic Charts
  - If you try it, please [file a bug](https://github.com/dimaryaz/jdmtool/issues/) to report your results - even just to say "it worked".

It is mainly tested on Linux, but should work on OS X and Windows.

## Installing

You may want to create a Python virtual environment using e.g. [virtualenvwrapper](https://pypi.org/project/virtualenvwrapper/).

Install the latest `jdmtool` release:

```
pip3 install jdmtool
```

Or install the latest code from GitHub:

```
pip3 install "git+https://github.com/dimaryaz/jdmtool.git#egg=jdmtool"
```

### IFD 400 and G1000

You should install an optional Just-in-Time compiler by running:
```
pip3 install jdmtool[jit]
```

This should significantly improve transfer speeds.

### GNS 400/500

Make sure you have access to the USB device. On Linux, you should copy `udev/50-garmin.rules` to `/etc/udev/rules.d/` and possibly reload the rules.

## Basic Usage

### Log in

You only need to run this once (unless you change your password).

```
$ jdmtool login
Username: test@example.com
Password: 
Logged in successfully
```

### Refresh the list of available downloads

Run this every time you want to download updates.

```
$ jdmtool refresh
Downloading services...
Downloading keychain...
Success
```

### View available downloads

```
$ jdmtool list
ID  Name                                                                    Coverage              Version   Start Date  End Date    Downloaded
 0  Garmin GNS 400/500 Series WAAS - NavData                                Americas              2303      2023-03-23  2023-04-20            
 1  Garmin GNS 400/500 Series WAAS - NavData                                Americas              2304      2023-04-20  2023-05-18            
```

### View detailed info

```
$ jdmtool info 0
Aircraft Manufacturer:        LOCKHEED
Aircraft Model:               SR-71
Aircraft Tail Number:         N12345

Avionics:                     Garmin GNS 400/500 Series WAAS
Coverage:                     Americas
Service Type:                 NavData
Service Code:                 DGRW7253
Service ID:                   12345678
Service Renewal Date:         2024-01-01 00:00:00

Version:                      2303
Version Start Date:           2023-03-23 06:00:00
Version End Date:             2023-04-20 06:00:00

Next Version:                 2304
Next Version Available Date:  2023-04-10 06:00:00
Next Version Start Date:      2023-04-20 06:00:00

File Name:                    dgrw72_2303_eceb0273.bin
File Size:                    8443904
File CRC32:                   eceb0273
Serial Number:                
System ID:                    

Downloads:
  /home/user/.local/share/jdmtool/downloads/dgrw72_2303_eceb0273.bin  (missing)
```

### Download the database

```
$ jdmtool download 0
Downloading: 100%|█████████████████████████████████████████████████| 8.44M/8.44M [00:03<00:00, 2.15MB/s]
Downloaded to /home/user/.local/share/jdmtool/downloads/dgrw72_2303_eceb0273.bin
```

### Transfer the database to the data card (GNS 400/500)

```
$ jdmtool transfer 0
Found device: Bus 001 Device 052: ID 0e39:1250
Detected data card: 16MB WAAS
Transfer /home/user/.local/share/jdmtool/downloads/dgrw72_2303_eceb0273.bin to the data card? (y/n) y
Erasing the database: 100%|████████████████████████████████████████| 8.59M/8.59M [02:15<00:00, 63.1KB/s]
Writing the database: 100%|████████████████████████████████████████| 8.59M/8.59M [04:14<00:00, 40.5KB/s]
Verifying the database: 100%|██████████████████████████████████████| 8.59M/8.59M [01:32<00:00, 92.5KB/s]
Writing new metadata: {2303~12345678}
Done
```

### Transfer the database to the USB drive (IFD 440 or G1000)

Note: the final database file requires the FAT32 volume ID of the USB drive. `jdmtool` will attempt to find it automatically - which requires the destination to be an actual FAT32-formatted device, not any random directory. Alternatively, you may set the volume ID manually using the `--vol-id` parameter.

> Getting the volume ID automatically is currently not supported on Mac OS, so you will need to use the `--vol-id` parameter. You can try [these instructions](https://apple.stackexchange.com/questions/408562/how-can-i-get-the-volume-serial-number-of-a-fat-volume) for finding the volume ID.

```
$ jdmtool transfer 0 /run/media/user/USB/
Transfer 'Avidyne IFD 400 Series, Bendix King AeroNav Series - NavData' to /run/media/user/USB/? (y/n) y
Found volume ID: 1234abcd
Writing to /run/media/user/USB/navdata.dsf: 100%|██████████████████| 38.0M/38.0M [00:15<00:00, 2.49MB/s]
Updating .jdm...
Done
```

## Advanced Features (GNS 400/500)

### Check that the tool can detect the device and the data card:

```
$ jdmtool detect
Found device: Bus 001 Device 049: ID 0e39:1250
Firmware version: 20071203
Card inserted:
  IID: 0x1004100
  Unknown identifier: 0x38001000
```

("Unknown identifier" likely contains the information about what type of card this is, but
I don't have enough information to decode it.)


### Read the metadata (should contain the cycle and the service ID):

JDM seems to only write it to 16MB cards. Not clear if it's actually used for anything.

```
$ jdmtool read-metadata
Found device: Bus 001 Device 045: ID 0e39:1250
Detected data card: 16MB WAAS
Database metadata: {2303~12345678}
```

### Write the metadata (should probably keep the same format):

JDM seems to only write it to 16MB cards. Not clear if it's actually used for anything.

```
$ jdmtool write-metadata '{2303~12345678}'
Found device: Bus 001 Device 045: ID 0e39:1250
Detected data card: 16MB WAAS
Done
```

### Read the current database from the data card:

```
$ jdmtool read-database db.bin
Found device: Bus 001 Device 044: ID 0e39:1250
Detected data card: 16MB WAAS
Reading the database: 100%|████████████████████████████████████████| 8.59M/8.59M [01:33<00:00, 91.6KB/s]
Truncating the file...
Done
```

You should now have the database in `db.bin`:

```
$ file db.bin
db.bin: DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "GARMIN10", sectors/cluster 8, FAT  1, root entries 512, sectors 32768 (volumes <=32 MB), sectors/FAT 16, sectors/track 63, heads 255, hidden sectors 63, serial number 0x1102, label: "GARMIN AT  ", FAT (16 bit)
```

It may not match the original downloaded file exactly. There is no way to know the size of the database on the data card, so either `db.bin` or the original file will likely contain extra `\xFF` bytes at the end.

### Write a new database to the data card:

```
$ jdmtool write-database dgrw72_2303_eceb0273.bin
Found device: Bus 001 Device 045: ID 0e39:1250
Detected data card: 16MB WAAS
Transfer dgrw72_2303_eceb0273.bin to the data card? (y/n) y
Erasing the database: 100%|████████████████████████████████████████| 8.59M/8.59M [02:15<00:00, 63.1KB/s]
Writing the database: 100%|████████████████████████████████████████| 8.59M/8.59M [04:14<00:00, 40.5KB/s]
Verifying the database: 100%|██████████████████████████████████████| 8.59M/8.59M [01:32<00:00, 92.5KB/s]
Done
```


## Bugs

Please [file a bug](https://github.com/dimaryaz/jdmtool/issues/) if you run into problems, or if you have a device/service that is not currently supported.
