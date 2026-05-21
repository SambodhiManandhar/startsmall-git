# NUTANIX COMMUNITY EDITION GUIDE

---

## 1. THE 4-STEP DOWNLOAD PROCESS

### Step 1: Identity Verification (The Gatekeeper)
* You must be logged into a Nutanix ID account.
* If you aren't logged in, the link will either throw an error or redirect you to a generic page.
* **TIP:** Register at https://my.nutanix.com if you haven't already.

### Step 2: Locating the Correct Thread
* The discussion forum thread acts as the "official source."
* You have to scroll through the first post to find the specific "Installer ISO" link.
* Nutanix updates this thread with the latest version (currently 2.0 or 2.1), so the download URL itself changes over time.
* **URL:** https://next.nutanix.com/discussion-forum-14/download-community-edition-38417

### Step 3: The Browser-Only Download
* This is an Authenticated Download.
* Because the server checks for your login "cookie," you cannot simply copy-paste the link into Proxmox's "Download from URL" tool.
* You must download the file directly to your personal computer's Downloads folder first.

### Step 4: Verification (MD5 Checksum)
* In that forum thread, Nutanix provides a long string of numbers and letters (the MD5 hash).
* It is standard practice to check this after your download finishes to ensure the 5GB+ file wasn't corrupted during the transfer.

---

## 2. PROXMOX VM CPU CONFIGURATION (Nested Virtualization)

While setting up the VM, make sure to set the CPU type to "host" to enable nested virtualization:

1. Select your Nutanix VM in the Proxmox sidebar.
2. Click the **Hardware** tab in the center panel.
3. Find the line labeled "Processors" and double-click it (or click Edit).
4. Find the **Type** dropdown menu.
5. Scroll to the very top and select **host**.
6. Click **OK**.

---

## 3. MINIMUM SYSTEM REQUIREMENTS

| Component | Minimum Requirement |
| :--- | :--- |
| **CPU** | 4 vCPUs |
| **RAM** | 20 GB RAM |
| **Network** | 2x Network Adapters |
| **Storage (3 HDDs)** | - Host Boot Disk: 30 GB <br> - CVM Boot Disk: 200 GB <br> - Data Disk: 200 GB |

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
