# Lab Build Notes

## Environment

- Hypervisor: Microsoft Hyper-V
- VM: `VetClinic-Wazuh-Server`
- Guest: Ubuntu Server 24.04 LTS
- Wazuh: 4.14.7 all-in-one deployment
- Endpoint: Windows 11 Pro
- Agent: `vetclinic-windows-01`, ID `001`

## Resource checks

The Ubuntu VM was validated with approximately 8 GB of assigned memory, a 4 GB swap allocation, and sufficient free disk space for the lab workload.

## Core service validation

```bash
sudo systemctl is-active wazuh-manager wazuh-indexer wazuh-dashboard filebeat
```

Expected result: four `active` lines.

## Agent service validation

Run in Administrator Windows PowerShell:

```powershell
Get-Service WazuhSvc
```

Expected result: `Running`.

## Manager-address check

```powershell
Select-String -Path "C:\Program Files (x86)\ossec-agent\ossec.conf" -Pattern "<address>"
```

Because Hyper-V's Default Switch uses dynamic addressing, compare this value with `hostname -I` on Ubuntu after every host restart.

## Custom rule validation

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
echo $?
```

An exit code of `0` indicates successful validation.

```bash
sudo systemctl restart wazuh-manager
sudo systemctl is-active wazuh-manager
```

## Authorised test command

Run only in Administrator Windows PowerShell:

```powershell
Add-Content "C:\VetClinic-Lab\Patient-Records\readme.txt" "Verified custom rule 100100 test - $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')"
```

The custom alert can then be found in Wazuh with:

```text
rule.id:100100
```

## Troubleshooting lesson

PowerShell commands such as `Get-Item`, `Add-Content`, and `Select-String` must be run at a Windows `PS C:\...>` prompt. They are not Ubuntu commands. Ubuntu commands such as `sudo`, `systemctl`, and `/var/ossec/bin/wazuh-analysisd` must be run at the `drakano@vetclinic-wazuh:~$` prompt.

