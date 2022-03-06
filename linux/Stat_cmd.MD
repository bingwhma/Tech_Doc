在linux系统下，使用stat（显示inode信息）命令可以查看一个文件的某些信息

stat displays the following file information:

    File - The name of the file.
    Size - The size of the file in bytes.
    Blocks - The number of allocated blocks the file takes.
    IO Block - The size in bytes of every block.
    File type - (ex. regular file, directory, symbolic link.)
    Device - Device number in hex and decimal.
    Inode - Inode number.
    Links - Number of hard links.
    Access - File permissions in the numeric and symbolic methods.
    Uid - User ID and name of the owner .
    Gid - Group ID and name of the owner.
    Context - The SELinux security context.
    Access - The last time the file was accessed.
    Modify - The last time the file’s content was modified.
    Change - The last time the file’s attribute or content was changed.
    Birth - File creation time (not supported in Linux).
