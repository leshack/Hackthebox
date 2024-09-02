![logo](/logo.png)

# [Photon Lockdown- Challange]  
Hi folks, today I am going to solve an very easy rated hack the box challange which was released on as the hardware challange on HTB,Photon Lockdown created by [4594485](https://app.hackthebox.com/users/459485).So without any further intro, let'sf jump in.

# CHALLENGE DESCRIPTION

We’ve located the adversary’s location and must now secure access to their Optical Network Terminal to disable their internet connection. Fortunately, we’ve obtained a copy of the device’s firmware, which is suspected to contain hardcoded credentials. Can you extract the password from it?

Download the Files To play the challenge.When trying to uzip it requires a password `default password: hackthebox`

![](/Challanges/Hardware/Photon-Lockdown/Screenshots/downloads.png)

check the `file` info

```sh
file rootfs
```

![](/Challanges/Hardware/Photon-Lockdown/Screenshots/filesystem.png)

doing some search and this is what i get

Squashfs is a compressed read-only filesystem for Linux. It’s used to create highly compressed file systems that are typically mounted read-only. It’s popular for creating live CDs, embedded systems, and situations where conserving space is important.

Here are some key points about Squashfs:

1. **Compression**: Squashfs uses various compression algorithms, including gzip, LZMA, LZO, and XZ, to reduce the size of files within the filesystem.

2. **Read-Only**: The filesystem is read-only, meaning that you cannot modify files directly once they are on the Squashfs filesystem. This makes it ideal for scenarios where you want to ensure the integrity of the data.

3. **Efficiency**: Squashfs is designed to be efficient in both space and performance. It combines compression with features like file deduplication and a mechanism for quickly accessing data.

4. **Usage**: Commonly used in live distributions of Linux, embedded systems, and recovery environments. It’s often used in conjunction with initramfs (initial RAM filesystem) to provide a compressed root filesystem.

5. **Compatibility**: Squashfs is supported by the Linux kernel and can be managed using tools like `mksquashfs` for creating Squashfs filesystems and `unsquashfs` for extracting files from them.

6. **File Systems**: Squashfs can handle various file types and attributes, including symbolic links, device nodes, and special files, though it does not support file system features like journaling.

now lets extract files 

```sh
sudo unsquashfs rootfs
```

![](/Challanges/Hardware/Photon-Lockdown/Screenshots/unsquashfs.png)

we find this directories 

![](/Challanges/Hardware/Photon-Lockdown/Screenshots/directories.png)

now lest search for the flag using this command

```sh
grep -r -i HTB . 2>/dev/null
```

and we find the flag

![](/Challanges/Hardware/Photon-Lockdown/Screenshots/flag.png)

	-------------------------END successful attack @lesley----------------------



