# Start Here: Finish the VetClinic Wazuh Portfolio

This checklist begins immediately after capturing the successful custom-rule alert (`04-custom-patient-record-alert.png`). Complete it in order when you return.

## Current completed evidence

- `01-wazuh-dashboard-overview.png` — Wazuh overview and active endpoint
- `02-windows-agent-details.png` — enrolled Windows 11 agent
- `03-fim-file-change-detected.png` — standard FIM rule 550 detection
- `04-custom-patient-record-alert.png` — custom rule 100100, level 12
- `05-vulnerability-detection.png` — vulnerability severity and affected packages

Your custom server rule is stored at:

```text
/var/ossec/etc/rules/vetclinic_rules.xml
```

The monitored Windows file is:

```text
C:\VetClinic-Lab\Patient-Records\readme.txt
```

## 1. Resume the lab after another shutdown

1. Start `VetClinic-Wazuh-Server` in Hyper-V.
2. Sign in through the Ubuntu console and run:

   ```bash
   hostname -I
   ```

3. Note the current `172.30.x.x` address. Hyper-V's Default Switch may assign a different address after a restart.
4. From **Windows PowerShell**, connect with:

   ```powershell
   ssh drakano@CURRENT_VM_IP
   ```

5. At the Ubuntu prompt (`drakano@vetclinic-wazuh:~$`), verify the Wazuh services:

   ```bash
   sudo systemctl is-active wazuh-manager wazuh-indexer wazuh-dashboard filebeat
   ```

   All four lines should say `active`.

6. In **Administrator: Windows PowerShell**, check the manager address used by the agent:

   ```powershell
   Select-String -Path "C:\Program Files (x86)\ossec-agent\ossec.conf" -Pattern "<address>"
   ```

7. If the displayed address is not the current VM address, replace `CURRENT_VM_IP` below and run:

   ```powershell
   $configPath = "C:\Program Files (x86)\ossec-agent\ossec.conf"
   $newManagerIp = "CURRENT_VM_IP"
   Copy-Item $configPath "$configPath.backup" -Force
   $configText = Get-Content $configPath -Raw
   $configText = $configText -replace '<address>[^<]+</address>', "<address>$newManagerIp</address>"
   Set-Content $configPath $configText -Encoding UTF8
   Restart-Service WazuhSvc
   Get-Service WazuhSvc
   ```

8. Open `https://CURRENT_VM_IP`, sign in to Wazuh, and confirm `Active (1)` and `Disconnected (0)`.

Never include passwords or login screens in portfolio screenshots.

## 2. Save screenshot 05 if it was not saved before shutdown

Open **Vulnerability Detection > Dashboard** and save the view showing severity counts, CVEs, the Windows endpoint, and affected packages as:

```text
C:\VetClinic-Lab\Portfolio\screenshots\05-vulnerability-detection.png
```

## 3. Capture CIS configuration-assessment evidence

1. In Wazuh, open the top-left menu.
2. Select **Configuration Assessment**.
3. Select `vetclinic-windows-01` if requested.
4. Open **Dashboard** or **Overview**.
5. Make sure the screen shows the CIS Microsoft Windows 11 policy, passed checks, failed checks, not-applicable checks, and overall score.
6. Save the screenshot as:

   ```text
   C:\VetClinic-Lab\Portfolio\screenshots\06-security-configuration-assessment.png
   ```

Portfolio meaning: this demonstrates CIS benchmark assessment and security-baseline gap analysis.

## 4. Capture a Windows Defender security event

1. Open **Administrator: Windows PowerShell** (`PS C:\...>`).
2. Start an authorised Defender quick scan:

   ```powershell
   Start-MpScan -ScanType QuickScan
   ```

3. When it finishes, verify its status:

   ```powershell
   Get-MpComputerStatus | Select-Object AntivirusEnabled, RealTimeProtectionEnabled, QuickScanStartTime, QuickScanEndTime
   ```

4. Wait up to two minutes for Wazuh to ingest the event.
5. In Wazuh, open **Threat Hunting > Events**.
6. Search for:

   ```text
   rule.id:62108
   ```

7. If that returns nothing, use:

   ```text
   rule.groups:windows_defender
   ```

8. Set the time range to **Last 24 hours**, click **Refresh**, and show the event row with the agent name, description, level, and rule ID.
9. Save it as:

   ```text
   C:\VetClinic-Lab\Portfolio\screenshots\07-windows-defender-event.png
   ```

Portfolio meaning: this demonstrates Windows security-event collection and SIEM investigation.

## 5. Capture the MITRE ATT&CK mapping

1. In **Threat Hunting > Events**, search for:

   ```text
   rule.id:100100
   ```

2. Expand the custom-rule event using the icon at the far left of its row.
3. Show these fields where possible:
   - `agent.name: vetclinic-windows-01`
   - `rule.id: 100100`
   - `rule.level: 12`
   - `rule.description`
   - `rule.mitre.id: T1565.001`
   - `syscheck.path`
4. Save it as:

   ```text
   C:\VetClinic-Lab\Portfolio\screenshots\08-mitre-mapping-custom-alert.png
   ```

This screenshot is recommended but optional if the event-detail layout does not show the fields clearly. Screenshot 04 already proves the custom alert.

## 6. Export the live custom rule

At the **Ubuntu SSH prompt**, run:

```bash
sudo cp /var/ossec/etc/rules/vetclinic_rules.xml /home/drakano/vetclinic_rules.xml
sudo chown drakano:drakano /home/drakano/vetclinic_rules.xml
```

Then, from **Windows PowerShell**, replace `CURRENT_VM_IP` and run:

```powershell
New-Item -ItemType Directory -Force -Path "C:\VetClinic-Lab\Portfolio\rules" | Out-Null
scp drakano@CURRENT_VM_IP:/home/drakano/vetclinic_rules.xml "C:\VetClinic-Lab\Portfolio\rules\vetclinic_rules.xml"
```

Validate the copied file:

```powershell
Get-Content "C:\VetClinic-Lab\Portfolio\rules\vetclinic_rules.xml"
```

## 7. Assemble the GitHub repository

Use this final structure:

```text
vetclinic-wazuh-siem-lab/
├── README.md
├── START-HERE.md
├── docs/
│   ├── findings-and-remediation.md
│   └── lab-build-notes.md
├── rules/
│   └── vetclinic_rules.xml
└── screenshots/
    ├── 01-wazuh-dashboard-overview.png
    ├── 02-windows-agent-details.png
    ├── 03-fim-file-change-detected.png
    ├── 04-custom-patient-record-alert.png
    ├── 05-vulnerability-detection.png
    ├── 06-security-configuration-assessment.png
    ├── 07-windows-defender-event.png
    └── 08-mitre-mapping-custom-alert.png
```

The downloadable project package already contains the README, documentation, rule, and screenshots available so far. Copy newly captured screenshots 06–08 into its `screenshots` folder.

## 8. Final privacy and quality check

Before publishing, confirm:

- No Wazuh, Ubuntu, email, GitHub, or Windows passwords appear anywhere.
- No personal email address, product key, or activation-settings screen is included.
- The screenshots show only authorised lab activity.
- Screenshot filenames match the README exactly.
- Screenshot 04 visibly shows rule 100100, the monitored path, modified event, and level 12.
- Screenshot 05 shows the severity counts and affected endpoint/packages.
- The custom rule file contains no credentials.
- Any Windows activation watermark is cropped out where practical; it does not invalidate the technical evidence.

## 9. Publish to GitHub

1. Sign in to GitHub as `drakanor98`.
2. Create a public repository named:

   ```text
   vetclinic-wazuh-siem-lab
   ```

3. Do not initialise it with another README, licence, or `.gitignore`.
4. Open **Windows PowerShell** in the extracted project folder and run:

   ```powershell
   git init
   git branch -M main
   git add .
   git commit -m "Publish VetClinic Wazuh SIEM monitoring lab"
   git remote add origin https://github.com/drakanor98/vetclinic-wazuh-siem-lab.git
   git push -u origin main
   ```

5. If Git asks you to sign in, complete the browser authentication prompt. Never paste your GitHub password into the repository.
6. On GitHub, add this description:

   ```text
   Wazuh SIEM lab demonstrating Windows endpoint monitoring, custom FIM detection, MITRE ATT&CK mapping, vulnerability management and CIS assessment.
   ```

7. Add these repository topics:

   ```text
   wazuh, siem, cybersecurity, blue-team, detection-engineering, file-integrity-monitoring, vulnerability-management, mitre-attack, windows-security, hyper-v
   ```

8. Pin the repository on your GitHub profile.

## 10. Final portfolio wording

### CV project entry

**VetClinic Wazuh SIEM and Endpoint Monitoring Lab**  
Built an all-in-one Wazuh SIEM environment on an Ubuntu Hyper-V virtual machine, onboarded a Windows 11 endpoint, configured file-integrity monitoring, and engineered a severity-12 custom rule mapped to MITRE ATT&CK T1565.001. Investigated endpoint alerts, Windows Defender telemetry, CIS benchmark gaps, and 721 vulnerability findings to produce prioritised remediation recommendations.

### Short portfolio description

Designed a defensive-security lab that monitors a Windows veterinary-clinic endpoint using Wazuh. The project demonstrates endpoint onboarding, FIM, a custom high-severity patient-record modification alert, MITRE ATT&CK mapping, vulnerability detection, security configuration assessment, and evidence-led remediation.

## Completion definition

The project is finished when:

- Screenshots 01–07 are present; screenshot 08 is strongly recommended.
- The custom XML rule is included.
- The README images display correctly on GitHub.
- The repository contains no credentials or sensitive information.
- The public repository is pinned to the `drakanor98` profile.

