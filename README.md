# Attach EBS Volume to EC2 Instance and Provision Snapshots

This project creates an Amazon EBS volume, attaches it to a running EC2 instance, formats and mounts it as a persistent file system, and creates a snapshot of the volume for backup.

**Skill:** Backup for Instances
**Course:** IT 442 Cloud Infrastructure, George Mason University

## Architecture

EC2 instance (`lab4`, `t2.micro`, Amazon Linux 2023, us-east-1c) → EBS volume (`1 GiB`, gp3, us-east-1c) attached as `/dev/xvdbf` → mounted at `/mnt/data-store` → EBS snapshot of the volume

## Steps

### 1. Create an EC2 instance
- Name: `lab4`
- Instance type: `t2.micro`
- Availability Zone: us-east-1c
- Launched in N. Virginia (us-east-1)

### 2. Create the EBS volume
- Volume type: General Purpose SSD (gp3)
- Size: 1 GiB
- IOPS: 3000 (default)
- Throughput: 125 MiB/s (default)
- Availability Zone: same as the instance (us-east-1c)
- Tag: key `madeleine`, value `My Volume`

### 3. Attach the volume to the instance
- Attached the volume to `lab4` from the Volumes console. The console confirmed the attachment as successful.

### 4. Connect to the instance
- Opened the instance in a browser terminal with **Connect → EC2 Instance Connect**.

### 5. Check the existing file systems
```bash
df -h
```
The root volume (`/dev/xvda1`) was the only data disk. The new volume (`/dev/xvdbf`) did not appear yet because it had not been formatted or mounted.

### 6. Format the new volume
```bash
sudo mkfs -t ext3 /dev/xvdbf
```
The command created an ext3 file system with 262,144 4 KB blocks.

### 7. Mount the volume
```bash
sudo mkdir /mnt/data-store
sudo mount /dev/xvdbf /mnt/data-store
```

### 8. Make the mount persist across reboots
```bash
echo "/dev/xvdbf /mnt/data-store ext3 defaults,noatime 1 2" | sudo tee -a /etc/fstab
cat /etc/fstab
```
The new line appears at the bottom of `/etc/fstab`, so the volume mounts automatically when the instance starts.

### 9. Verify the mount
```bash
df -h
```
`/dev/xvdbf` (about 975 MB) is mounted on `/mnt/data-store`.

### 10. Write a test file
```bash
sudo sh -c "echo some text has been written > /mnt/data-store/file.txt"
cat /mnt/data-store/file.txt
```
The output showed `some text has been written`, confirming the file is stored on the mounted volume.

### 11. Create a snapshot
- Source volume: the 1 GiB volume created in step 2
- Description: "my volume snapshot for lab 4"
- Encryption: not encrypted
- Tag: key `madeleine`, value `my snap shot`

## Verification

| Check | Result |
|---|---|
| Volume attached | Attachment confirmed in the Volumes console |
| File system created | ext3 created on `/dev/xvdbf` |
| Mount point | `/dev/xvdbf` mounted on `/mnt/data-store` (confirmed with `df -h`) |
| Persistent mount | Entry added to `/etc/fstab` |
| Test file | `file.txt` written and read back from the mounted volume |
|
