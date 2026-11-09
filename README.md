# Zabbix 7 LTS

## zbx_export_templates7_0.yaml
Template name: **SMART by Zabbix agent 2 by Silureth**

Monitors S.M.A.R.T. data of physical disks (HDD, SSD, NVMe, including disks behind RAID controllers)
with the built-in smart plugin of the **stock** Zabbix agent 2 (7.0.x, tested on 7.0.32).
No external scripts or custom agent builds are needed: the built-in plugin now returns `normalized_value` itself.
Based on the official "SMART by Zabbix agent 2" template; it has its own UUIDs, so it can be imported next to the official one.

### Supported disks
Every disk that smartctl can read gets the base items: temperature, power-on hours, model, serial number,
smartctl exit status and self-test result (SATA/SAS); NVMe disks also get percentage used, critical warning and media errors.
Vendor attributes are added on top when the disk reports them:

| Brand | Endurance item | Other vendor items |
|---|---|---|
| Samsung SATA SSD | Wear_Leveling_Count | Total bytes written, Runtime_Bad_Block, Uncorrectable_Error_Cnt, CRC_Error_Count |
| Micron / Crucial SATA SSD | Percent_Lifetime_Remain | Total bytes written, Current_Pending_Sector, Offline_Uncorrectable, UDMA_CRC_Error_Count, Reported_Uncorrect |
| WD (Red SA500 and others) | Media_Wearout_Indicator | Total bytes written (Total_LBAs_Written or Host_Writes_GiB), UDMA_CRC_Error_Count, Reported_Uncorrect |
| Kingston SSD | SSD_Life_Left | Total bytes written if reported |
| NVMe (any brand) | Percentage used | Critical warning, media errors |
| HDD (any brand) | - | Raw_Read_Error_Rate, Seek_Error_Rate, Spin_Up_Time, Start_Stop_Count, Reallocated_Sector_Ct, Current_Pending_Sector, Offline_Uncorrectable |

Tested with the stock 7.0.32 agent, SATA disks behind an HP Smart Array controller (`cciss`):
- Samsung SSD 860 EVO 500GB / 1TB / 2TB, 870 EVO 500GB / 1TB / 2TB
- Micron MTFDDAK960TCB
- WD Red SA500 1TB (WDS100T1R0A) and 2TB
- NVMe: Samsung SSD 980, 980 PRO, MZVL8512HFLU

Kingston and HDD support comes from the official template's attribute list and has not been tested here.

### Setup
1. Install Zabbix agent 2 (7.0 or newer) and smartmontools 7.1 or newer.
2. Make sure the agent can find `smartctl`: either have its folder in the agent's PATH, or set the full path in the agent configuration:
   ```
   Plugins.Smart.Path=/usr/sbin/smartctl
   ```
3. Allow the zabbix user to run smartctl as root without a password (Linux). Run `visudo` and add:
   ```
   zabbix ALL=(ALL) NOPASSWD:/usr/sbin/smartctl
   ```
4. Check on the host that the agent sees the disks:
   ```
   zabbix_agent2 -t smart.disk.discovery
   ```
5. Import `zbx_export_templates7_0.yaml` and link the template to the host.
   Disks are discovered every hour; SMART data is polled every `{$SMART.DISK.GET.INTERVAL}` (default 1h).

### Per-disk settings
Use macro context on the host. The context is the disk name shown in the item names, e.g. `sda cciss,3` or `nvme0`:
- `{$SMART.WEAROUT.INVERT:"<disk>"}` = `1` for disks whose Media_Wearout_Indicator counts up from 0 (new) to 100 (worn out), mostly newer WD models
- `{$SMART.TBW.UNIT.BYTES:"<disk>"}` = `1073741824` for disks that report Total_LBAs_Written in GiB instead of 512-byte sectors (e.g. WD Red SA500)
- `{$SMART.TBW.SUSPECT:"<disk>"}` = a higher limit (e.g. `5000T`) for enterprise SSDs rated for many petabytes, so they don't trip the "shows no wear after heavy writes" alert

On HP Smart Array (cciss) hosts set `{$SMART.DISK.NAME.NOT_MATCHES}` to ` scsi$` to skip the controller's logical volume, which has no SMART data.

### Changes compared to the official template
- SSD endurance items: Wear_Leveling_Count (Samsung), SSD_Life_Left (Kingston), Media_Wearout_Indicator (WD), Percent_Lifetime_Remain (Micron/Crucial).
  All are stored as "% life remaining" (100 = new) and alert below `{$SMART.WEAR_LEVEL.WARN}` / `AVG` / `CRIT` (default 30 / 20 / 10 %).
- Total bytes written per disk (from Total_LBAs_Written or Host_Writes_GiB). There are no write-limit triggers:
  SSD wear is alerted by the endurance items above, and fixed limits only made noise on large disks and HDDs.
  Instead it flags a wear counter that does not move: more than `{$SMART.TBW.SUSPECT}` (default `500T`) written while the
  endurance item still shows 99-100 % remaining (Average, can be closed manually).
- Failure attribute items and triggers: Current_Pending_Sector, Offline_Uncorrectable, Uncorrectable_Error_Cnt, Runtime_Bad_Block,
  UDMA_CRC_Error_Count / CRC_Error_Count (interface errors, usually cable or backplane), plus alerts when Reallocated_Sector_Ct
  or Reported_Uncorrect increase. "Increased" alerts stay open until closed manually or until the disk is replaced (new serial number).
- Disk age triggers on power-on hours (`{$SMART.OLDAGE.WARN}` / `AVG` / `CRIT` = Warning / Average / High), with the disk type as macro context:
  8 / 10 / 12 years for HDDs, 6 / 7 / 8 years for SSD and NVMe. Age is a secondary indicator: it still alerts when a disk's
  wear counter is wrong (e.g. an old SSD that reports 0 % wear after a petabyte of writes).
  Override on a host per type, e.g. `{$SMART.OLDAGE.WARN:"ssd"}` = `61320`.
- Configurable polling interval (`{$SMART.DISK.GET.INTERVAL}`) and triggers that use the latest value, suited to hourly polling.
- Vendor attribute items are created only when the disk reports that attribute.

# Zabbix 6 LTS

## zabbix-agent2-smart-raw
zabbix-agent2 with raw data , precompiled on DEBIAN 10 
 - go version go1.22.3 linux/amd64
 - libc6 (2.28-10+deb10u3)

## ZIPPED CHANGES to smart plugin in smart.zip
src/go/plugins/smart

## PRECOMPILED agent(s) is based on
zabbix_agent2 (Zabbix) 6.0.31rc1
zabbix_agent2 (Zabbix) 6.0.37
zabbix_agent2 (Zabbix) 7.0.8  //with old template and 
zabbix_agent2 (Zabbix) 7.2.2  //with old template and 


## zbx_export_templates6_0.yaml
for LTS 6.0 with the custom agents above, included changes for "normalized values"
MACROS for it
{$SMART.WEAR_LEVEL.CRIT}
{$SMART.WEAR_LEVEL.WARN}
{$SMART.WEAR_LEVEL.AVG}

added TBW to item prototypes the value is calculated on the assumtion that the sector size is 512
from "total_lbas_written"

added newser WDC disk media_wearout goes from 1->100 old WD Red are 100->1, added normalization to Trigger Prototype

# USED
changes made by this pull:
https://github.com/zabbix/zabbix/pull/95
# BUILD INSTRUCTIONS
https://medium.com/@zhuravlev.vitaly/writing-watcher-zabbix-agent2-mqtt-plugin-in-go-e00067ea1990