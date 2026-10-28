Linux Disk Management is a common task in system administration and DevOps, especially when managing production servers.
When a new disk is attached to a Linux server, the typical process is to identify the disk, partition it, create a filesystem, 
mount it, and configure it for persistent mounting after a reboot.
For example, lsblk can be used to identify the newly attached disk:
lsblk
A GPT partition table and partition can then be created using parted:
sudo parted /dev/nvme1n1 --script mklabel gpt mkpart primary 0% 100%
Once the partition is created, a filesystem such as XFS can be created:
sudo mkfs.xfs -f /dev/nvme1n1p1
A mount point can then be created and the filesystem mounted:
sudo mkdir -p /data
sudo mount /dev/nvme1n1p1 /data
The mounted filesystem can be verified using:
df -hT | grep data
For persistent mounting across server reboots, the filesystem UUID can be retrieved using:
sudo blkid /dev/nvme1n1p1
The UUID can then be added to /etc/fstab:
UUID=d68b606a-4df3-44c2-bd74-418fa36e9fa2 /data xfs defaults,nofail 0 2
Using the UUID in /etc/fstab is generally preferred over relying directly on device names, 
as device naming can change depending on how storage devices are detected.
After updating /etc/fstab, the configuration can be validated with:
sudo mount -a
The mount can then be verified again using df -hT.
Another common production requirement is expanding existing storage. With an LVM-based setup, 
the partition can first be extended using growpart, followed by extending the required logical volume using lvextend.
For example:
sudo growpart /dev/nvme0n1 4
sudo lvextend -L +10G /dev/RootVG/rootVol
sudo lvextend -L +10G /dev/RootVG/varVol
One important point is that extending the logical volume does not always mean the filesystem has automatically been expanded. 
Depending on the filesystem, an additional filesystem resize may be required, such as xfs_growfs for XFS or resize2fs for ext4.
Some useful commands for Linux storage and system troubleshooting:
lsblk — view disks and partitions
df -hT — check filesystem type and available disk space
du -sh * — identify disk usage by files and directories
free -m — check memory utilization
Understanding the relationship between disk → partition → filesystem → mount point → LVM is important for 
troubleshooting storage issues and performing disk expansion safely in production environments.