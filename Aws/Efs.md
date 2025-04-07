## 📦 AWS EFS (Elastic File System) - Step-by-Step Notes with Diagram and Explanation

---

### 🔍 What is EFS?

**Elastic File System (EFS)** is a scalable, fully-managed, NFS (Network File System) file system by AWS that can be mounted on multiple EC2 instances at once.

**Key Features:**

- Shared file system (like Google Drive for EC2s)
- Scalable storage
- Mountable across multiple AZs and EC2s
- Pay-as-you-use model

---

### 🧰 Use Case:

When you want to share files between multiple EC2 instances – like logs, media files, app data etc.

---

### 🧱 Step-by-Step Setup:

### ✅ Step 1: Create an EFS

1. Go to AWS Console → Search for **EFS** → Click **Create file system**
2. **Name** your EFS
3. Choose **VPC** (same VPC as EC2)
4. Add **mount targets** in *each Availability Zone* where your EC2s are running
5. Attach appropriate **Security Groups** (should allow NFS traffic - port 2049)

---

### ✅ Step 2: Create Security Group Rule

Make sure the EC2 instance's security group allows **inbound NFS (2049)** from the EFS security group.

---

### ✅ Step 3: Install NFS client in EC2 instance

Amazon Linux:

```bash
sudo yum install -y nfs-utils
```

Ubuntu:

```bash
sudo apt update
sudo apt install -y nfs-common
```

---

### ✅ Step 4: Create a Mount Point

```bash
sudo mkdir /mnt/efs
```

---

### ✅ Step 5: Mount EFS to EC2 (Using DNS)

Find EFS DNS:

```
<file-system-id>.efs.<region>.amazonaws.com
```

Mount using:

```bash
sudo mount -t nfs -o vers=4.1 <EFS-DNS>:/ /mnt/efs
```

📌 **Explanation:**

- `mount` → mounts file systems
- `-t nfs` → file system type is NFS
- `-o vers=4.1` → option to use NFS version 4.1
- `<EFS-DNS>:/` → remote EFS root directory
- `/mnt/efs` → local mount point

---

### ✅ Step 5.1: Mount EFS to EC2 (Using IP Address)

Find EFS IP from VPC > Network Interface linked to Mount Target.

Example:

```bash
sudo mount -t nfs -o vers=4.1 10.0.0.123:/ /mnt/efs
```

📌 **Note:** DNS is preferred. Use IP only if DNS fails or for troubleshooting.

---

### ✅ Step 5.2: Mount EFS from AWS Console (using EC2 Connect)

1. Go to EFS Console → Click your File System → Click **Attach**
2. Select **Mount via DNS** or **Mount via IP** (option depends on setup)
3. Copy the mount command shown (it auto-fills your region & file system ID)
4. Go to EC2 Console → Connect → **EC2 Instance Connect** (browser-based terminal)
5. Paste and run the mount command in the terminal

---

### ✅ Step 6: Test Sharing

Try creating a file in `/mnt/efs` from one instance, then SSH into another instance (same VPC) and check that file in the same mount point.

```bash
echo "Hello from EC2" | sudo tee /mnt/efs/hello.txt
```

---

### ✅ Step 7: (Optional) Mount on Reboot

Edit `/etc/fstab`:

```bash
echo '<EFS-DNS>:/ /mnt/efs nfs4 defaults,_netdev 0 0' | sudo tee -a /etc/fstab
```

📌 `_netdev` ensures mount happens only after networking is up.

---

### 🗄️ Diagram - EFS Flow

```plaintext
       +-----------+          +------------+        +------------+
       |  EC2 A    |  <--->   |   AWS EFS  |  <-->  |  EC2 B     |
       +-----------+   NFS    +------------+   NFS  +------------+
           |                          |                  |
     /mnt/efs                    Shared FS         /mnt/efs
```

---

### 💡 Notes:

- EFS is region-specific, not global
- High availability & scalability
- You pay per GB stored + GB transferred
- Best for shared access, not block storage

---

### 💼 Common Interview Questions (with English Answers):

---

**1. What is the difference between EFS and EBS?**\
📌 *EBS is block-level storage and can be attached to only one EC2 at a time. EFS is a shared file storage system that can be accessed by multiple EC2 instances simultaneously.*

**2. How does EFS achieve high availability?**\
📌 *EFS stores data redundantly across multiple Availability Zones (AZs), which ensures high availability and durability.*

**3. Can EFS be mounted across regions?**\
📌 *No, EFS is region-specific. It can only be mounted within the same region where it is created.*

**4. What protocol does EFS use?**\
📌 *EFS uses the NFS (Network File System) protocol, specifically version 4.1.*

**5. How do you persist EFS mount across reboots?**\
📌 *You can add an entry in the **`/etc/fstab`** file using the **`_netdev`** option to ensure the mount happens after the network is available.*

**6. What is the use of the **_netdev** option in **/etc/fstab**?**\
📌 *It tells the OS to wait for network services before attempting to mount the EFS file system.*

**7. How does EFS handle scalability?**\
📌 *EFS automatically scales as you add or remove files—no need to provision storage ahead of time.*

**8. What is the pricing model for EFS?**\
📌 *You pay for the storage used (per GB/month) and data transferred (read/write operations).*

**9. Is EFS encrypted?**\
📌 *Yes, EFS supports encryption at rest and in transit using AWS KMS.*

**10. Can EFS be used with containers like ECS or EKS?**\
📌 *Yes, EFS can be mounted to containers using EFS CSI driver in EKS or via ECS Task Definitions.*

**11. Difference between EFS and S3?**\
📌 *EFS is a file system used for real-time access by EC2 instances. S3 is object storage used to store and retrieve any amount of data anytime.*

**12. What are the throughput and performance modes in EFS?**

📌 *EFS offers two throughput modes (Bursting and Provisioned) and two performance modes (General Purpose and Max I/O). Bursting mode adjusts automatically based on usage, while Provisioned lets you specify throughput. General Purpose is suitable for latency-sensitive workloads, and Max I/O supports higher aggregate throughput but with higher latency.*

---

### ♻️ EFS Lifecycle Policies

**What are they?**  
EFS Lifecycle Policies automatically move files that haven’t been accessed for a set period to **Infrequent Access (IA)** storage class to save cost.

**Available transition options:**

- 7 days
- 14 days
- 30 days
- 60 days
- 90 days

**How to enable:**

1. While creating the EFS, enable **Lifecycle Management**
2. Choose transition period (like “30 days of no access”)
3. AWS will move the files to IA tier automatically

**Pricing Benefit:**  
IA storage is **cheaper** than Standard storage, great for files that aren’t frequently used.

**Access:**  
Files stay visible in the same mount point—no changes needed in your app logic.

**Note:**  
If accessed again, files are automatically pulled back to Standard storage.

---

### 📊 EFS vs EBS - Tabular Comparison

| Feature           | EFS (Elastic File System)                    | EBS (Elastic Block Store)                      |
| ----------------- | -------------------------------------------- | ---------------------------------------------- |
| Type              | File Storage (NFS)                           | Block Storage                                  |
| Mountable on      | Multiple EC2s (same VPC & AZs)               | One EC2 instance at a time                     |
| Scalability       | Automatically scales                         | Needs manual provisioning                      |
| Shared Access     | Yes                                          | No                                             |
| Protocol          | NFS (v4.1)                                   | Block-level via device mapping                 |
| Performance Modes | General Purpose & Max I/O                    | SSD (gp2, gp3, io1, io2) & HDD (st1, sc1)      |
| Backup & Snapshot | Manual via AWS Backup                        | Snapshots supported                            |
| Pricing           | Per GB + operations (read/write)             | Per GB provisioned                             |
| Use Case          | Shared content, web server assets, logs etc. | Databases, boot volumes, low-latency workloads |
| Mount Persistence | Requires `/etc/fstab` + _netdev              | Mounts like a disk                             |

---

