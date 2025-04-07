## 🪵 EBS (Elastic Block Store) Volume Mount Karne ke Steps – Full Guide

### 🔰 Step 0: EBS kya hota hai?

EBS ek type ka storage hai jo EC2 instance ke saath attach kiya ja sakta hai. Ye basically ek virtual hard drive hai jise hum EC2 instance me extra space ke liye use karte hain.

---

### 📦 Step 1: EBS Volume Create Karo

1. AWS Console kholo → EC2 service pe jao.
2. Left sidebar me **"Elastic Block Store"** → **"Volumes"**.
3. Click **“Create Volume”**.
4. Size select karo (jaise 8 GiB), Availability Zone wahi rakho jo EC2 ka hai.
5. Volume type (gp2/general purpose) aur baaki defaults theek hain → **Create Volume** dabao.

📌 *Note: Volume tabhi attach hoga jab EC2 aur EBS dono same AZ me ho.*

---

### 🖇️ Step 2: Volume ko EC2 se Attach Karo

1. Abhi jo volume create kiya us par jao → **Actions > Attach Volume**.
2. Instance select karo (jo tumhara EC2 instance hai).
3. Device name default hi rehne do (`/dev/xvdf` ya similar).
4. **Attach** dabao.

---

### 🧼 Step 3: EC2 me Login Karke Check Karo

1. EC2 instance me SSH karo:
   ```bash
   ssh -i "keypair.pem" ec2-user@<public-ip>
   ```
2. Check karo device dikh raha hai ya nahi:
   ```bash
   lsblk
   ```
   Output me `xvdf` dikhna chahiye. Agar `xvdf` ke neeche `xvdf1` nahi hai, toh iska matlab partition abhi bana nahi hai.

---

### 🧱 Step 4: Partition Banana (Agar Required Ho)

📌 *Partition ek logical divide hota hai storage device ka, jaise hum ek badi almirah ko alag-alag compartments me baant dete hain. Isse hum data ko alag organize kar sakte hain. Naya EBS volume by default blank hota hai aur usme koi partition nahi hota, isliye use file system dene se pehle partition banana zaruri hota hai.*

1. `fdisk` tool open karo:

   ```bash
   sudo fdisk /dev/xvdf
   ```

2. Andar jaake:

   - `n` → new partition banane ke liye
   - `p` → primary partition banane ke liye
   - `1` → partition number (first partition ke liye 1)
   - `Enter` → default start sector
   - `Enter` → default end sector
   - `w` → changes save karne ke liye

3. Partition table update karne ke liye:

   ```bash
   sudo partprobe
   ```

4. Dubara check karo:

   ```bash
   lsblk
   ```

   Ab `xvdf1` dikhna chahiye.

📌 `fdisk` partition banata hai, `partprobe` OS ko update karta hai ki partition ban gaya hai.

1. `fdisk` tool open karo:

   ```bash
   sudo fdisk /dev/xvdf
   ```

2. Andar jaake:

   - `n` → new partition banane ke liye
   - `p` → primary partition banane ke liye
   - `1` → partition number (first partition ke liye 1)
   - `Enter` → default start sector
   - `Enter` → default end sector
   - `w` → changes save karne ke liye

3. Partition table update karne ke liye:

   ```bash
   sudo partprobe
   ```

4. Dubara check karo:

   ```bash
   lsblk
   ```

   Ab `xvdf1` dikhna chahiye.

📌 `fdisk` partition banata hai, `partprobe` OS ko update karta hai ki partition ban gaya hai.

---

### 🧃 Step 5: Format Karo Partition Ko

1. File system bana do (ext4 format):
   ```bash
   sudo mkfs -t ext4 /dev/xvdf1
   ```
   - `mkfs` = Make file system
   - `-t ext4` = ext4 file system ka type use karo
   📌 *File system zaruri hai data store karne ke liye – bina iske volume blank hai.*

---

### 📁 Step 6: Mount Point Banao aur Mount Karo

1. Directory banao jahan mount karna hai:
   ```bash
   sudo mkdir /mnt/myebs
   ```
2. Mount karo partition ko:
   ```bash
   sudo mount /dev/xvdf1 /mnt/myebs
   ```

---

### ✅ Step 7: Verify Karo

```bash
df -h
```

Output me `/mnt/myebs` dikhna chahiye with size of volume (e.g., 8G).

---

### ♻️ Step 8: Permanently Mount Karna (Reboot ke baad bhi)

1. UUID lo partition ka:

   ```bash
   sudo blkid /dev/xvdf1
   ```

   Output kuch aisa dikhega:

   ```
   UUID="abcd-1234" TYPE="ext4"
   ```

2. fstab file edit karo:

   ```bash
   sudo nano /etc/fstab
   ```

3. Add karo line:

   ```
   UUID=abcd-1234 /mnt/myebs ext4 defaults,nofail 0 2
   ```

   🔍 Breakdown of fstab fields:

   - `UUID=abcd-1234` → jise mount karna hai (unique identifier)
   - `/mnt/myebs` → mount hone wali jagah
   - `ext4` → file system type
   - `defaults,nofail` → mount options (defaults = normal behavior, nofail = agar volume na mile toh boot fail na ho)
   - `0` → dump (backup ke liye; 0 = disable)
   - `2` → fsck order (root ke liye 1, others ke liye 2 ya 0)

4. Test karo:

   ```bash
   sudo mount -a
   ```

   Agar error nahi aaya toh permanent mount successful hai 🎉

---

### 🗺️ Diagram: EBS Mounting Flow

```
  +--------------------+
  | Create EBS Volume  |
  +---------+----------+
            |
            v
  +---------------------+
  | Attach to EC2       |
  +---------+-----------+
            |
            v
  +----------------------+
  | SSH into EC2         |
  +----------+-----------+
             |
             v
  +-----------------------------+
  | Partition (fdisk + partprobe) |
  +----------+------------------+
             |
             v
  +-------------------+
  | Format (mkfs)     |
  +--------+----------+
           |
           v
  +--------------------+
  | Mount (mkdir + mount) |
  +--------+-----------+
           |
           v
  +-------------------+
  | Permanent Mount   |
  +-------------------+
```

---

### 💼 Interview Questions (with Answers):

---

**1. What is EBS and how does it work with EC2?**\
📌 *EBS (Elastic Block Store) is a block-level storage service provided by AWS. It acts like a virtual hard drive that you can attach to EC2 instances. It provides persistent, durable storage and can be used just like a physical disk.*

---

**2. What are the steps required to use a newly created EBS volume?**\
📌 *After creating the volume, attach it to an EC2 instance, create a partition using **`fdisk`**, format the partition with a file system like **`ext4`** using **`mkfs`**, and then mount it to a directory. Optionally, update the **`/etc/fstab`** file to enable automatic mounting on reboot.*

---

**3. Why is formatting a file system necessary?**\
📌 *Formatting organizes the storage space with a file system so that the operating system can read and write data efficiently. Without a file system, the OS cannot understand how to store or retrieve files from the volume.*

---

**4. What is the fstab file and what does it contain?**\
📌 *The **`/etc/fstab`** file is used to define how disk partitions, storage volumes, and remote filesystems should be mounted during system boot. It contains six fields: device identifier (like UUID), mount point, filesystem type, mount options, dump option, and fsck order.*

---

**5. What does the **``** command do?**\
📌 *The **`mount -a`** command mounts all filesystems mentioned in the **`/etc/fstab`** file that are not currently mounted. It's commonly used to test fstab entries without restarting the system.*

---

### 📋 Summary (Ek Nazar me):

- Create volume
- Attach volume
- SSH into instance
- Partition banao (agar needed ho)
- Format karo
- Mount karo
- Verify karo
- fstab me add karo for permanent mount

---

Batado Bhoomika agar koi aur cheez add karni ho ya aur funny bana doon toh 😄❤️

