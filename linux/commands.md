---
description: Linux的一些些命令
---

# Commands

## Directory Command

{% code fullWidth="false" %}
```sh
pwd: display the full path of the current location

mkdir: create a new directory
    -p: create multilevel directories
    
cd: swicth directories

ls: show the information about directory or files
    -a: display all folder information (include hiden)
    -l: display the detailed information of folder, ll for short
    -h: used in conjunction with -l, display the size of files
        (adaptive units b,k,m,g...), ls -lh for short
    -d: when follow the specified directory, can view the information about current
        directory itself
        
cp: copy file from source to target
    -r: copy directory

mv: mv [path]/file [anotherpath] for cut file
    mv fileName1 fileName2 for rename
    
touch: create a blank text file

rm: delete files or directory
    -r: recursive process
    -f: force execute
```
{% endcode %}

## File Command

```
view small file: cat head tail
    cat: suitable for viewing data within one screen
        -n: number all output lines, starts with 1
    
    head: view the first few lines of the file, default is the first 10 lines
        -n: view the first n lines
    
    tail: view the last few lines of the file, default is the last 10 lines
        -f: monitor file content updates in real time, ctrl+v to end

view large file: less
     less: the tool for paginated display the files or other output, is the linux
         orthodox view files content tools, extrmely powerful
         -m: display the precentage of the file
         -N: display the line number for each line
         /string: search down for "string"
         ?string: search up for "string"
         n: search for next one(related to / or ?)
         N: search for the next one in the oppsite direction of n
         j: scroll down a line
         k: scroll up a line
         G: jump to the end of the file
         g: jump to the file header
         q: exit
         
grep: row filter, it can use regular expressions to search for text and print out
    mathcing lines
    -i: case insensitive
    -v: reverse selection, displays the line that doesn't contain the 'search string'
        content
    main parameters of the regular expression
        \: ignore the original meaning of special characters in regular expression
        ^: match the beginning of the regular expression
        $: matchs the tail of the regular expression
        [-]: scope, such as [A-Z] will match from A to Z
        .: represent any single character, such as a..b will match a12b or aabb
        *: wildcard character
        
awk: column filter
    format: awk [-F 'separator'] '{print $n[,$n]}' filename, default separator is
    "blank key" or "tab key", which can be specified without -F, $n is used to get
    the data for the specified column
    $0: represent all columns
    $1: represent first column
    $n: represent the nth column
    example: filter all columns in text.txt with the command "awk '{print $0}' text"
    
sed: search and replace file contents
    sed -n 'Np' filename: search for data in line N of the file
    sed -n 'N1,N2p' filename: search for data in line N1 to N2 of the file
    sed -n '/sss/p' filename: search the file for the rows contain the string 'sss'
    
    sed 's/the string for search/the string for replace/g' filename: search the
        string globally for replacement, and the replaced content is output to the 
        screen(don't modify the file content)
    sed 'N1,N2 s/the string for search/the string for replace/g' filename: seach the
        string from line N1 to N2 for replacement, and the replaced content is output
        to the screen(don't modify the file content)
    sed -i 's/the string for search/the string for replace/g' filename: search the
        string globally for replacement(file content will be modified)
    sed -i 'N1,N2 s/the string for search/the string for replace/g' filename: seach 
        the string from line N1 to N2 for replacement(file content will be modified)
        
|: pipe character
    command1 | command2: the output of command1 is taken as the input of command2
    note: pipe character handles only the correct output of the previous command
        and not handle the incorrect output
        
find: search for file or directory
    note: the directory uses find command can not be a link
    
```

## Output and Input redirection

```
>: standard output overwrites wirte file
>>: standard output appends to file

cat: connect file or standard input and print, this command is usually used to
    display the content of file
    cat << EOF: end with EOF input character as standard input
    cat > filename: create new file and put standard input and output into the
        filename file, end with 'ctrl+d'
```

## Remote network Command

```
scp: copy data from local host to remote server
    format: scp [-r] file_To_Copy remote_UserName@remote_Host_Ip:targetLocation
    
ssh: login remote server from localhost
    format: ssh userName@Host_Ip
    ssh-copy-id userNmae@Host_Ip: register the local public key certificate file with
        the remote server, and then we can login using the private key certificate
```

## Process Command

```
ps: the abbreviation of Process status, this command will list a snapshot of the
    current processes that were running at the time the ps command was executed
    -aux: view all processes on the system,using BS operating system format
        (with CPU and memory information)
        Decription of the command execution result:
            USER:owner of process PID:id of process PPID:parent process id
            %CPU:cpu percentage used by a process  %MEM: percentage of memory usgae
            VSZ:the amount of virtual memory used by the process(KB)
            RSS:the process uses physical memory
            TTY:if it is "pts/0", means the host process is connected to the network
            stat:process status
            Z:zombie process   <:high priority process  N:low priority process
            L:some page locked into memory
            s:the leader of process(with child processes below it)
            START:start time when process is triggered
            TIME:the actual cpu usage time of the process
            COMMAND: name and parameter of command
    -ef: view all processes on the system, using Liunx standard command format
        (with PID)
            Description of the command execution result:
                UID: id of user,but actual is user name
                PID: id of process PPID:praent process id
                C:percentage of cpu used by process
                STIME:start time of process
                TTY:which terminal is the process running on, if the process is
                independent of the terminal, the system will display "?". if the
                value is pts/0, it indicates that the host proess is connected by
                the network
                TIME:the time of cpu already be occupied by the process
                CMD: name and parameter of command
                
    ps often used with grep to find specific process, like ps -aux | grep sshd
    
kill: terminate process
     kill pid: wait for execution to finish and then terminate process(not mandatory)
     kill -9 pid: force to terminate process
     
     
top: real-time monitor process, will display the information about the processed
    running in the current system, includes pid, memory usage and cpu usage
    b:type keyboard "b" will highlight current running process 
    x:type keyboard "x" will sort process with the percentage of cpu
    
free: view the usage of system memeroy and the swap partition, and the output is very
    similar to the memory portion of the command top
    -b: display with byte
    -k: display with KB, default value
    -m: display with MB
    -g: display with GB
    Description of the command execution result:
        used: the size of the used physical memory(swap partition)
        free: the available physical memory size(swap partition)
        shared: the amount of shared memory by multiple processes
        buff/cache: the buffer size of disk
        avaiable: the size of memory can be used by new applictaions
    note: normally, the memory in swap partition is not be used unless the physical
        memory is full
        
netstat: view the usage of port
    netstat -apn | grep xx: find which process use the port xx
```

## Disk command

```
df: display the usage of each file system in Linux, includes the total capacity, 
    used capacity, and remaining capacity of the disk partition
    format: df -h

du: view the disk space occupied by specific directory or file
    format: du -s -h file/dir
```

## User or UserGroup Command

<pre><code>su username: swicth user and keep current dirctory and enviroment variables
<strong>
</strong><strong>su - username: switch user and go to the username's home directory and switch
</strong>    enviroment varibles
    
cat /etc/passwd: view the information of all users

useradd [option] username: create a new user
    ig: useradd -d /home/jerry  -g root -G tom,adm jerry :add user jerry and 
        indicate /home/jerry as jerry's home dirctory, the main group is root,
        additional groups are tom and jerry
        useradd -s /bin/sh -u 80000 test1: add a new user test1 and specify the
        shell program is /bin/sh and the user id is 80000 

usermod [option] username: modify user, parameter usage is similar to xx

userdel [option] username: delete user
    -r: delte user and user's home directory at the same time
    
passwd: manage user's password
    -l: lock account
    -u: unlock account
    -d: delete user's password
    format: passwd [option] username
    
cat /etc/group: view the information of user group
</code></pre>

## Authority Commad

```
chmod: modify the authority of file or directory
    -R: recursive process
    
chattr: add physical permissions, modify a file/folder is not allowed to be modified
    +i: can not be modified in any way
    +a: can only be appended, not be modified or deleted
    + <attribute>: open permission to file or folder
    - <attribute>: close permission to file or folder
    +R: recursive process
    
lsattr: view the authority of file/folder
    +a: view all files's attributes, includes hiding
    +d: display folder's attributes, not the attributes of files in the directory
    +R: recursive process
    
chown: used to manage which users and user groups own files
    format: chown owner:group filename
    
sudo: sudo means super user do, it can allow common user to temporarily perform root
    privileges, sudo also can restrict users from executing partial root permissions
```

## Package compression and decompression

```
common compressed file suffix:
    *.gz: gzip compressed files
    *.tar: tar command packed data,and didn't compress
    *.tar.gz: tar command packed and gzip compressed
    
gzip:
    filename: compression of source files is not retained
    -c filename > compressed_filename(.gz): keep the source file compressed
    -d compressed_filename(.gz): didn't keep source files decompressed
    -cd compressed_filename(.gz) decompressed_filename: keep source files 
        decompressed

tar:
    -cvf packaged_filename(.tar) need_packaged_filename: package files
    -tvf packaged_filename(.tar): view the packaged files
    -xvf packaged_filename(.tar) -C decompressed_dirname: decompress to 
        specified dir
    -czvf packaged_filename(.tar.gz) need_packaged_filename: package and 
        compress file
    -tzvf packaged_filename(.tar.gz): view the packaged files
    -xzvf packaged_filename(.tar.gz) -C decompressed_dirname: decompress to 
        specified dir
```

## Package Management tool

```
rpm: Redhat package management, used to manage software package under Linux
    advantage:
        rpm contains the data such as compiled programs and configuration files
            which allow user to avoid the recompilation
        before install rpm package, will check the system's disk capacity and
            os version to avoid incorrect installation
        the rpm file provides information about the software version, dependecy
            properties, software name, software usage and files contained in the
            software, let custmore easy to understand software
        due to software's informations are recorded in the Linux host database, 
            it's easy to query, upgrade and uninstall
    disadvantage:
        the environment in which the software are installed must be consistent with
            or equivalent to the enviroment requirements at the time of packaging
        need to meet the software dependency attribute requirements
        need to be careful when uninstall, don't remove the lowest layer of software
            first, otherwise the whole system will be affected
    note: rmp only records the dependency information, doesn't install dependent
        software automatically
    -ivh: install a package
    -Uvh: upgrade the installation package
    -e: uninstall a package
    -qa: query installation package
    -qR package_name: query which packages a package depends on
    -e --test package_name: query which package depends on the package
    -ql package_name: query rpm package path of installation
    -qa --queryformat "%{NAME} %{INSTALLTIME}\n" | sort: distinguish rpm package is
        installed by os or customer, if by os,installtime is 0
    -qf command_path: find which rpm the command belongs to
    
    
yum: base on rpm, and can download and install rpm package from specified server
    and can handle dependencies and install all dependent software packages
    automatically at once,without having to download and install them again and again
    note:
        yum has its own repository, also called source, download all rpm software to
            the yum repo
        rpm provides every software's information and dependency relationship, yum
            analyze the information and dependencies to generate a list of installed
            software
        when you want to install a software, it will install dependencies first and
            then install the software according to the list
        all of the yum source is restored in dir /etc/yum.repos.d
    check-update: list all of software which can be upgraded
    update: update all of software
    install: install specified software
    list: list all of the softwares which can be installed
    remove: delete the software package
    search: search a software package
    clean all && makecache: clean cache and then create new cache
    -y: when prompted to select "yes" for all during installation
    -q: doesn't shown the process during the installation
            
```

## Vim Editor

```
yy + p: copy the content of line where the cursor is located and paste it to the next
    line
yy + P: copy the content of the line where the cursor is located and paste it to the
    last line
3yy + P: copy the 3 lines of contents that contains the row where the cursor is
    located and paste it to the next line
dd: delete the row where the cursor is located
3dd: delete 3 rows contains the row where the cursor is located
10 x: delete the character after the cursor
20 x: delete the character before the cursor
G: enter the laste line of file
gg: enter the first line of file

:s/target/replace/g: replace the current line
:%s/target/replace/g: full file replacement
:set nu: enable line number display
:set nonu: disable line number display

i: insert data from the cursor
o: insetr data from the next line where the cursor is located
```
