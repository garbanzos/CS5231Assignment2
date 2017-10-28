# CS5231 Assignment 2

| Name        | Matriculation No.| Email  |
| ------------- |-------------| -----|
| Tan Yi Yan      | A0127051U | yiyan@u.nus.edu |

## Task 0
### Differences between Android VM and actual Android devices
One difference between them is the recovery OSes installed on them. An actual Android device's recovery OS is the recovery OS placed by vendor. This recovery OS placed by the vendor does not give users a shell prompt so users are unable to run arbitrary commands. Furthermore, the recovery OS placed by the vendor only accepts OTA packages signed by the vendor. In contrast, the Android VM has a general-purpose OS installed as the "recovery OS" - Ubuntu 15.10 is installed as the "recovery OS". To simulate what an actual recovery OS does when installing an OTA update, a recovery program (Neko Android) is installed on the recovery OS. This recovery program allows users access to both a normal user shell and a root shell so users are able to run arbitrary commands and make arbitrary changes to the Android partition. Moreover, the recovery program in the Android VM also allows users to disable digital signature verification of the OTA packages - this functionality is not available in the recovery OS of an actual Android device.

Another difference is location where the `rc` script files imported by `init.rc` are stored. In an actual Android device, these script files are inside `boot.img`, which contains `ramdisk.img` and the kernel. On the other hand, these
script files are are stored in the `ramdisk.img` image file on the Android VM.

In addition, the state of the bootloader may also be different. In an actual Android device, the bootloader is usually "locked". With a locked bootloader, any attempt to flash the installed OS will be denied by the bootloader. The manufacturers may or may not choose to permanently lock the bootloader so the bootloader cannot be unlocked with normal means. For the lab, it is assumed that the bootloader on the Android VM can be unlocked and it was "unlocked" at some point to replace the stock recovery OS with Ubuntu 15.10.

<br>

## Task 1

### Problematic component of signature verification process

> Identify which component in the signature verification process is problematic and explain why is it so. (10 Marks)

### Verifying our custom OTA file

> Show and explain how the OTA update service can be made to verify our custom ota file.

### Structure of OTA file

> **NOTE: may change after doing signature verification.** Show the structure of OTA file.

![](task1.3_ota_file_structure.png)
*Figure 1. Structure of OTA file in task 1*

### Dummy file within the Android system

> Show that the dummy file is created within the Android system.

![](task1.4_dummy_file.png)
*Figure 2. dummy file created within Android system for task 1*

## Task 2

### Structure of OTA file

> **NOTE: may change after doing signature verification.** Show the structure of OTA file.

![](task2.1_ota_file_structure.png)
*Figure 3. Structure of OTA file in task 2*

### Dummy file within the Android system

![](task2.2_dummy2_file.png)
*Figure 4. dummy2 file created within Android system for task 2*

### Differences between the dummy files are created in Task 1 and 2

- file permissions
- when it is created?? init vs app_process
- contents LOL

## Task 3

### The root shell

> Show that root shell can be obtained.

![](task3.1_root_shell.png)
*Figure 5. The root shell obtained in task 3*

### File descriptors of the client and shell processes

> Show that the client and shell processes share the same file descriptors.

![](task3.2_client_fd.png)
*Figure 6. File descriptors of client process*

![](task3.2_shell_fd.png)
*Figure 7. File descriptors of shell process*

### Source file

#### Server launches the original app process binary
In *mydaemonsu.c*'s `main()` function, lines 252 to 253:
```
argv[0] = APP_PROCESS;
execve(argv[0], argv, environ);
```

#### Client sends its FDs
In *mysu.c*'s `connect_daemon()` function, lines 101 to 103:
```
send_fd(socket, STDIN_FILENO);      //STDIN_FILENO = 0
send_fd(socket, STDOUT_FILENO);     //STDOUT_FILENO = 1
send_fd(socket, STDERR_FILENO);     //STDERR_FILENO = 2
```

#### Server forks to a child process
In *mydaemonsu.c*'s `run_daemon()` function, line 189:
```
if (0 == fork()) {
```

#### Child process receives client’s FDs
In *mydaemonsu.c*'s `child_process()` function, lines 147 to 149:
```
int client_in = recv_fd(socket);
int client_out = recv_fd(socket);
int client_err = recv_fd(socket);
```

#### Child process redirects its standard I/O FDs
In *mydaemonsu.c*'s `child_process()` function, lines 151 to 153:
```
dup2(client_in, STDIN_FILENO);      //STDIN_FILENO = 0
dup2(client_out, STDOUT_FILENO);    //STDOUT_FILENO = 1
dup2(client_err, STDERR_FILENO);    //STDERR_FILENO = 2
```

#### Child process launches a root shell
In *mydaemonsu.c*'s `child_process()` function, line 162:
```
execve(shell[0], shell, env);
```
