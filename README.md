---
description: Linux Directory
---

# Directory Structure

/bin:  store some common command&#x20;

/boot: store some of the core files to start Linux, includes some connection file and image files

/dve: the abbreviation of the "device", store some external device of Linux

/etc: store all of configuration files and sub directories required for system management

/home: the main directory of user, each user has its own home directory in Linux

/lib: store the most basic dynamic connection shared library, which acts like a DLL file in Windows

/media: Linux system will identify some devices automatically, such as U disk, CD-ROM drive, and so on, when it was identified , Linux will mount the device to this directory.

/mnt: user can mount other file system temporarily, we can mount CD-ROM drive on /mnt/, and enter the directory to view the contents of CD-ROM drive

/opt: the abbreviation of the "optional", and is the directory where additional software can be installed on host

/proc: the abbreviation of the "processes", store a series of special files about the current state of the kernel, and we can get system information by directly accessing this directory.

/root: the home of system administrator, also known as the main directory of super user

/sbin: store the system management program of system administrator

/srv:  store the data which needs to be extracted after service is started

/tmp: the abbreviation of the "temporary", store some temporary files

/usr: store many applications and files of user

/var: the abbreviation of "variable", store something which needs continue to expand, and we are used to putting those directories that are frequently modified in this directory. Includes various log files

/run: a temporary file systems, store some infomation since system started, when system restarts, this directory needs to be cleaned or deleted, and if you have the directory /var/run, you should make it point to /run
