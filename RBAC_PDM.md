# Proxmox Datacenter Manager (PDM) Installation and Proxmox VE Developer Isolation Guide

## Installing Proxmox Datacenter Manager (PDM)

You can either install the PDM ISO or install the package directly on an existing Proxmox VE host for co-hosting on port `8443`.

This procedure resolves repository signing and package authentication issues and installs Proxmox Datacenter Manager on an existing Proxmox VE host. Since PVE already includes the trusted Proxmox archive keyring, no additional GPG key import is required.

### Step 1: Configure the PDM Repository

Log in to the Proxmox VE host shell and create the repository configuration file:

```bash
cat <<EOF > /etc/apt/sources.list.d/proxmox-pdm.list
# Proxmox Datacenter Manager repository for Debian Trixie
deb [signed-by=/usr/share/keyrings/proxmox-archive-keyring.gpg] http://download.proxmox.com/debian/pdm trixie pdm-no-subscription
EOF
```

### Step 2: Refresh Package Metadata

Clear cached package data and update repository indexes:

```bash
apt clean
apt update
```

### Step 3: Install PDM and the Web UI

Install the Datacenter Manager packages:

```bash
apt install proxmox-datacenter-manager proxmox-datacenter-manager-ui -y
```

After installation, the PDM web interface will be available on port `8443`.

---

# Proxmox VE (PVE) Web GUI Guide: Isolating Developers to Individual VMs

This guide demonstrates how to configure developer isolation entirely through the Proxmox VE web interface, allowing developers to manage only their assigned virtual machines while preventing access to node-level or cluster-wide resources.

---

![Create role] (images/PVE_role.png)

## Step 1 — Create a Custom Developer Role

Create a role that grants VM management capabilities without administrative access. You can also use prebuilt role like PVEAudior.

1. Navigate to **Datacenter**.
2. Open **Permissions** → **Roles**.
3. Click **Create**.
4. Enter the role name:

   ```text
   Developer
   ```

5. Select the following privileges:

   - VM.Allocate
   - VM.Clone
   - VM.Config.CDROM
   - VM.Config.CPU
   - VM.Config.Cloudinit
   - VM.Config.Disk
   - VM.Config.Memory
   - VM.Config.Network
   - VM.Console
   - VM.Migrate
   - VM.Monitor
   - VM.PowerMgmt
   - VM.Snapshot
   - VM.Snapshot.Rollback
   - Datastore.AllocateSpace

6. Click **Create**.

### Security Note

Do not assign any of the following privilege groups:

- `Sys.*`
- `Node.*`
- `Permissions.Modify`

These permissions provide cluster-wide administrative control and should not be granted to developers. But needs Sys.Audit to see the nodes ram and disk usage.
---

## Step 2 — Create Resource Pools

Resource pools provide logical boundaries for delegated administration.

1. Navigate to **Datacenter** → **Resource Pools**.
2. Click **Create**.
3. Create the first pool:

   - **Pool ID:** `alice-vms`
   - **Comment:** `Alice's dev VMs`

4. Click **Create**.
5. Repeat for additional developers:

   - `bob-vms`
   - `carol-vms`

---
![Create user] (images/PVE_user.png)

## Step 3 — Create Developer User Accounts

If external authentication is not being used, create local Proxmox user accounts.

1. Navigate to **Datacenter** → **Permissions** → **Users**.
2. Click **Add**.
3. Configure the user:

   - **User name:** `alice`
   - **Realm:** `pve`
   - **Password:** Choose a secure password
   - **Comment:** `Developer Alice`

4. Click **Add**.
5. Repeat for:

   - `bob`
   - `carol`

### Optional: LDAP or Active Directory Integration

To use centralized authentication:

1. Navigate to **Datacenter** → **Permissions** → **Authentication**.
2. Click **Add**.
3. Select either:

   - LDAP
   - Active Directory

4. Configure your directory server settings and complete the setup.

---

## Step 4 — Assign ACLs to Resource Pools

Grant each developer access only to their own resource pool.

1. Navigate to **Datacenter** → **Permissions**.
2. Click **Add** → **User Permission**.
3. Create the ACL for Alice:

   - **Path:** `/pool/alice-vms`
   - **User:** `alice@pve`
   - **Role:** `Developer`
   - **Propagate:** Enabled

4. Click **Add**.
5. Repeat for other developers:

| Pool Path | User | Role |
|------------|------|------|
| `/pool/bob-vms` | `bob@pve` | `Developer` |
| `/pool/carol-vms` | `carol@pve` | `Developer` |

Enabling **Propagate** ensures that any VM added to the pool automatically inherits the assigned permissions. If you 

---

## Step 5 — Assign VMs to the Appropriate Pool

Existing virtual machines must be placed into their corresponding resource pools.

1. Select the VM from the left navigation tree.
2. Open the **Options** tab.
3. Double-click **Pool**.
4. Select the appropriate pool (for example, `alice-vms`).
5. Click **OK**.

### Best Practice for New VMs

When a developer creates a new VM using the **Create VM** wizard:

1. On the **General** tab, locate the **Pool** field.
2. Select the developer's designated pool.
3. Complete the VM creation process.

If no pool is selected, the developer may lose administrative access to the VM because the ACL inheritance will not apply.

---

## Step 6 — Verify Isolation

Validate the configuration from a developer's perspective.

1. Open a private/incognito browser session or log out of the administrator account.
2. Sign in as:

   ```text
   alice@pve
   ```

# Proxmox Datacenter Manager (PDM) Access Control and Delegated Visibility

Once your developers have been isolated within Proxmox VE resource pools, you can use Proxmox Datacenter Manager (PDM) to provide centralized visibility and delegated access across one or more clusters.

## Phase 1 — Connect Proxmox VE to PDM

Before configuring permissions in PDM, ensure the PVE API token used for integration has the `PVEAuditor` role assigned at the root path (`/`) within the Proxmox VE cluster.

After the token has been created and assigned:

1. Log in to the PDM Web UI:

   ```text
   https://YOUR-PDM-IP:8443
   ```

   Sign in as:

   ```text
   root@pam
   ```

2. From the left navigation menu, select **Remotes**.
3. Click **Add**.
4. Configure the connection:

   - **Remote ID:** `dev-cluster`
   - **Host:** IP address or FQDN of the Proxmox VE node

5. Under **Authentication**, select **Token**.
6. Enter the API token details:

   - **Token ID:**

     ```text
     pdm-monitor@pve!pdm-token
     ```

   - **Secret:**

     ```text
     <PVE API Token Secret>
     ```

7. Click **Add**.
8. If prompted about a self-signed SSL certificate, review the fingerprint and click **Accept**.

![Create role] (images/PDM_fullview.png)


## Phase 2 — Create Users in PDM

Next, create the users who will access the PDM interface.

1. Navigate to:

   **Access Control** → **Users**

2. Click **Add**.
3. Enter the user information:

   - **User name:** `alice`
   - **Realm:** `pdm`

   Alternatively, select your synchronized LDAP or Active Directory realm.

4. Set a password.
5. Click **Add**.

Repeat this process for any additional users.

---
![Create role] (images/PDM_fullview.png)

## Phase 3 — Configure Targeted Access Control

PDM permissions determine what portions of the connected infrastructure users can see and manage.

### Option A: Restrict Access to a Single VM

1. Navigate to:

   **Access Control** → **Permissions**

2. Click **Add** → **User Permission**.
3. Configure the permission:

   - **Path:**

     ```text
     /remotes/dev-cluster/guests/101
     ```

   - **User:**

     ```text
     alice@pdm
     ```

   - **Role:** Select one of the following:

     - `PDMReader`
     - `PVEDevOwner`

   - **Propagate:** Enabled

4. Click **Add**.

### Option B: Restrict Access to a Specific Node

To limit visibility to a single node:

```text
/remotes/dev-cluster/nodes/pve-node-01
```

Configure the same user, role, and propagation settings as above.

### Role Selection

#### PDMReader

Use this role when users should only be able to monitor resources.

Capabilities:

- View dashboards
- View VM details
- View metrics and performance graphs
- View resource utilization

Restrictions:

- No console access
- No power operations
- No configuration changes
- No VM management actions

#### PVEDevOwner

Use this role when users require operational control over their assigned resources.

Capabilities:

- VM console access
- Start and stop VMs
- Reboot VMs
- View metrics and monitoring data
- Manage resources under the assigned path

Restrictions:

- No cluster-wide administration
- No global configuration changes
- No access outside assigned resources

### Importance of Propagation

Always enable **Propagate** when assigning permissions.

This ensures access automatically extends to all child objects beneath the selected path, such as:

```text
/remotes/dev-cluster/guests/101
```

or

```text
/remotes/dev-cluster/nodes/pve-node-01
```

Without propagation enabled, users may see incomplete resource trees or lose access to nested objects.

---
![Create role] (images/PDM_test.png)
## Phase 4 — Verify Access Restrictions

1. Log out of the PDM administrator account:

   ```text
   root@pam
   ```

2. Log back in using the delegated user account:

   ```text
   alice@pdm
   ```

