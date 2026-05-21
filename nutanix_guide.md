# NUTANIX COMMUNITY EDITION GUIDE

---

## 1. THE 4-STEP DOWNLOAD PROCESS

### [Step 1: Identity Verification (The Gatekeeper)]
* [cite_start]You must be logged into a Nutanix ID account[cite: 1].
* [cite_start]If you aren't logged in, the link will either throw an error or redirect you to a generic page[cite: 2].
* [cite_start]**TIP:** Register at https://my.nutanix.com if you haven't already[cite: 3].

### [Step 2: Locating the Correct Thread]
* [cite_start]The discussion forum thread acts as the "official source." [cite: 4]
* [cite_start]You have to scroll through the first post to find the specific "Installer ISO" link[cite: 4].
* [cite_start]Nutanix updates this thread with the latest version (currently 2.0 or 2.1), so the download URL itself changes over time[cite: 5].
* [cite_start]**URL:** https://next.nutanix.com/discussion-forum-14/download-community-edition-38417 [cite: 6]

### [Step 3: The Browser-Only Download]
* [cite_start]This is an Authenticated Download[cite: 7].
* [cite_start]Because the server checks for your login "cookie," you cannot simply copy-paste the link into Proxmox's "Download from URL" tool[cite: 7].
* [cite_start]You must download the file directly to your personal computer's Downloads folder first[cite: 8].

### [Step 4: Verification (MD5 Checksum)]
* [cite_start]In that forum thread, Nutanix provides a long string of numbers and letters (the MD5 hash)[cite: 9].
* [cite_start]It is standard practice to check this after your download finishes to ensure the 5GB+ file wasn't corrupted during the transfer[cite: 10].

---

## 2. PROXMOX VM CPU CONFIGURATION (Nested Virtualization)

[cite_start]While setting up the VM, make sure to set the CPU type to "host" to enable nested virtualization[cite: 11]:

1. [cite_start]Select your Nutanix VM in the Proxmox sidebar[cite: 11].
2. [cite_start]Click the "Hardware" tab in the center panel[cite: 12].
3. [cite_start]Find the line labeled "Processors" and double-click it (or click Edit)[cite: 12].
4. [cite_start]Find the "Type" dropdown menu[cite: 13].
5. [cite_start]Scroll to the very top and select "host"[cite: 13].
6. [cite_start]Click OK[cite: 13].

---

## 3. MINIMUM SYSTEM REQUIREMENTS

| Component | Minimum Requirement |
| :--- | :--- |
| [cite_start]**CPU** [cite: 15] | [cite_start]4 vCPUs [cite: 18] |
| [cite_start]**RAM** [cite: 19] | [cite_start]20 GB RAM [cite: 20] |
| [cite_start]**Network** [cite: 21] | [cite_start]2x Network Adapters [cite: 22] |
| [cite_start]**Storage (3 HDDs)** [cite: 23] | - [cite_start]Host Boot Disk : 30 GB [cite: 23][cite_start]<br>- CVM Boot Disk  : 200 GB [cite: 24][cite_start]<br>- Data Disk      : 200 GB [cite: 25] |

+--------------------+--------------------------------------------------+
   | Component          | Minimum Requirement                              |
   +--------------------+--------------------------------------------------+
   | CPU                | 4 vCPUs                                          |
   | RAM                | 20 GB RAM                                        |
   | Network            | 2x Network Adapters                              |
   | Storage (3 HDDs)   | - Host Boot Disk : 30 GB                         |
   |                    | - CVM Boot Disk  : 200 GB                        |
   |                    | - Data Disk      : 200 GB                        |
   +--------------------+--------------------------------------------------+
---

## 4. DEFAULT CREDENTIALS Reference

### Default Installer/AHV Credentials
* **User:** `root`
* **Password:** `nutanix/4u`

### Default Prism Element (Web GUI) Credentials
* **User:** `admin`
* **Password:** `nutanix/4u`

---

## 5. VERIFYING COMPONENT & CVM STATUS

1. Attach Controller & Disk:
      
      qm set 126 --scsihw virtio-scsi-pci
      qm set 126 --scsi0 local-lvm:vm-126-disk-0

   2. Configure Boot Mode (Depending on Phase 1 finding):

      [IF ORIGINAL VM WAS UEFI]
      qm set 126 --machine q35 --bios ovmf
      qm set 126 --efidisk0 local-lvm:1,format=raw

      [IF ORIGINAL VM WAS LEGACY BIOS]
      (Skip this step entirely. Proxmox defaults to SeaBIOS).

   3. Set Boot Priority:
      
      qm set 126 --boot order=scsi0


--------------------------------------------------------------------------------
PHASE 6: CLEANUP
--------------------------------------------------------------------------------

   1. Start the VM via the Proxmox Web UI and verify it boots.
   2. Delete the temporary migration file to reclaim space:
      
      rm /var/lib/vz/images/ubuntu_export.qcow2
