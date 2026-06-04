# above-and-beeond 

`above-and-beeond` is a modified `beeond` command that allows you to create smarter and better BeeOND clusters:

- Proper segregation between data and metadata volumes
- Proper formatting of each device type
- Enables asymmetric BeeOND clusters
- With minor modifications, constituent devices may be formatted with non-default filesystems

Additional features may be added, but my primary focus is NetApp E-Series-specific use cases that interest me. You're free to copy and change the code within the license (I don't mean mine, which is very permissive, but ThinkParQ's).

## How to use

If your BeeGFS version is > 8.3, first try the script out on a test VM to make sure it works correctly. The initial release was created from beeond in v8.3.1.

- Copy the two scripts to `/opt/beegfs/lib/` and make them executable
- Create a config file with **deterministic** device names. Example with non-deterministic device names, `ab.yaml`:
```yaml
node1:
 - meta: [/dev/vdb]
 - data: [/dev/vdc,/dev/vdd]
node2:
 - meta: [/dev/vdc]
 - data: [/dev/vdg,/dev/vdz]
```
- Deploy:
  
```sh
sudo /usr/bin/beeond start -y ab.yaml -c /mnt/beeond
```
- End result might look like something like this (view from `node1`):
```sh
$ df
Filesystem                        1K-blocks    Used Available Use% Mounted on
/dev/vdb                            3484948  264280   2942892   9% /mnt/beeond_internal_meta_vdb
/dev/vdc                            7798784  182516   7616268   3% /mnt/beeond_internal_data_vdc
/dev/vdd                            7798784  182516   7616268   3% /mnt/beeond_internal_data_vdc
beegfs_ondemand                    31195136  182272  31195136   3% /mnt/beeond
```

- The stop command works as expected; if `-d` is passed to `stop`, `wipefs` is used to nuke data on the constituent volumes

## Tips for NetApp E-Series users

- Create a RAID 0 disk group
- Split disk group capacity among `N` hosts (e.g. 4 x 3.84 TB / 5 = 3.07 TB per host)
- Leave an extra 5% for better wear-leveling (3.07 x 0.95 = 2.85 TB per host)
- Create MD and data disks and present them individually to each host
  - 5% for metadata - 2.85 TB x 0.05 = 140 GiB
  - 95% for data  - 2.85 TB x 0.95 = 2.7 TiB. Split in 2 volumes x 1.35 TB
- Rescan and validate access
- Create a YAML file using unique device names and run `above-and-beeond`

```yaml
host1:
  - meta: [/dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab26]
  - data: [/dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab27, /dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab28]
host2:
  - meta: [/dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab20]
  - data: [/dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab21, /dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab22]
host3:
  - meta: [/dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab23]
  - data: [/dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab24, /dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab25]
host4:
  - meta: [/dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab11]
  - data: [/dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab12, /dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab13]
host5:
  - meta: [/dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab14]
  - data: [/dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab15, /dev/disk/by-id/scsi-3600605b00f3eb5202516998299f4ab16]  
```

This workflow can be easily automated with my SANtricity client libraries (Go, Python, PowerShell) or Terraform Provider for SANtricity.

## License

- See LICENSE.txt for original (upstream) `beeond` scripts
- New code added in this repository: the MIT License (c) github.com/scaleoutsean, 2026
- Documentation: CC BY 4.0, https://scaleoutsean.github.io
