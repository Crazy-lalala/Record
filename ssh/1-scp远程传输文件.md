## 目录(Directory)

- [1 背景及目的(Background and Objectives)](#1-背景及目的(Background and Objectives))

- [2 过程(The Process)](#2-过程(The Process))
  - [2.1 编写脚本(Script Writing)](#21-编写脚本(Script Writing))
  
  - [2.2 设置ssh密钥对登录(Setting up SSH key-based login)](#22-设置ssh密钥对登录(Setting up SSH key-based login)))
  
- [3 建议(Suggestion)](#3-建议(Suggestion))

## 1 背景及目的(Background and Objectives)

&emsp;&emsp;开发编译好的代码不能在本地运行，需要通过scp传输到实验板上跑。且中间还要经过一次跳板机的中转传输，过于繁琐，浪费一部分开发时间。尝试编写脚本以减少开发中通过scp命令传输时的工作。

&emsp;&emsp;The compiled code cannot be run locally and needs to be transferred to the experimental board via scp, with an additional transfer through a jump server, which is cumbersome and wastes part of the development time. An attempt is made to write a script to reduce the work involved in transferring via the scp command during development.

## 2 过程(The Process)

### 2.1 编写脚本(Script Writing)

&emsp;&emsp;编写脚本如下：

&emsp;&emsp;Writing a Script:

```bash
#!/bin/bash

usr_target_host="U1@I1"

local_dir_dir="P1"
target_dir_dir="P2"

local_dir_lib="P3"
target_dir_lib="P4"

if [ "$#" -lt 2 ]; then
    echo "Usage: $0 <dir_flag> <lib_flag>"
    echo "  dir_flag: 1 to copy directory"
    echo "  lib_flag: 1 to copy library"
    exit 1
fi

dir_flag=$1
lib_flag=$2

if [ "$dir_flag" == "1" ]; then
    scp -r $local_dir_dir/D1 $usr_target_host:$target_dir_dir
fi

if [ "$lib_flag" == "1" ]; then
    scp $llocal_dir_lib/L1 $usr_target_host:$target_dir_lib
fi
```
&emsp;&emsp;解释每一段的含义：

&emsp;&emsp;Explain the meaning of each section：

***
```bash
#!/bin/bash
```
&emsp;&emsp;表示这是一个Bash脚本文件，让计算机使用Bash脚本文件的规则来理解此文件。

&emsp;&emsp;Indicate that this is a Bash script file, instructing the computer to  interpret this file according to the rules of Bash script files.

***
```bash
usr_target_host="U1@I1"
```
&emsp;&emsp;目标host的用户名及IP，将`U1`替换为你要传输到的远端的用户名，`I1`替换对需要传输到的远端的IP。

&emsp;&emsp;The username and IP of the target host, replace `U1` with the username of the remote host you want to transfer to, and replace `I1` with the IP address of the remote host you need to transfer to.

***
```bash
local_dir_dir="P1"
target_dir_dir="P2"

local_dir_lib="P3"
target_dir_lib="P4"
```
&emsp;&emsp;`local_XX`：要传输的文件来源的路径。
&emsp;&emsp;`target_XX`：要传输的文件去向的路径。
&emsp;&emsp;因为使用编写的动态库，所以要同时传输不同路径下的生成文件夹和库到不同的路径。如果一次只需要传输一份文件，一个路径即可。

&emsp;&emsp;`local_XX`: The path from which the files to be transferred originate.
&emsp;&emsp;`target_XX`: The path to which the files will be transferred.
&emsp;&emsp;Because a dynamically linked library is being used, it is necessary to transfer generated folders and libraries from different paths to different destinations simultaneously. If only one file needs to be transferred at a time, one path is sufficient.	

***
```bash
if [ "$#" -lt 2 ]; then
    echo "Usage: $0 <dir_flag> <lib_flag>"
    echo "  dir_flag: 1 to copy directory"
    echo "  lib_flag: 1 to copy library"
    exit 1
fi
```
&emsp;&emsp;运行时的输入选项，接受两个参数。
&emsp;&emsp;&emsp;&emsp;第一个参数为1代表需要传输文件夹。
&emsp;&emsp;&emsp;&emsp;第二个参数为1代表需要传输文件。
&emsp;&emsp;如果参数不满足则退出。
&emsp;&emsp;添加输入参数功能是因为，文件夹传输的时间花费远大于库传输的时间，当只需要传输库时会浪费大量的事件在传输文件夹上。如果无此需求，可省略。

&emsp;&emsp;The input options during runtime accept two arguments.
&emsp;&emsp;&emsp;&emsp;The first argument, when set to 1, indicates that a folder needs to be transferred.
&emsp;&emsp;&emsp;&emsp;The second argument, when set to 1, indicates that a file needs to be transferred.
&emsp;&emsp;If the arguments are not met, the script will exit. The reason for adding input argument functionality is that the time  spent transferring folders is significantly greater than the time spent  transferring libraries. When only libraries need to be transferred, a  considerable amount of time can be wasted on transferring folders. If  this need does not exist, it can be omitted.

***
```bash
dir_flag=$1
lib_flag=$2
```
&emsp;&emsp;用两个变量接收输入的参数。如果无输入参数的需求，可省略。

&emsp;&emsp;Use two variables to receive input arguments. If there is no need for input arguments, they can be omitted.

***
```bash
if [ "$dir_flag" == "1" ]; then
    scp -r $local_dir_dir/D1 $usr_target_host:$target_dir_dir
fi

if [ "$lib_flag" == "1" ]; then
    scp $llocal_dir_lib/L1 $usr_target_host:$target_dir_lib
fi
```
&emsp;&emsp;通过对输入参数的判断，选择是否执行对应的scp命令。
&emsp;&emsp;将`D1`或`L1`替换为需要传输文件的具体的文件名。
&emsp;&emsp;通过scp传输文件夹需要增加`-r`选项。
&emsp;&emsp;如果一次只需要传输一份文件，一个scp命令即可。

&emsp;&emsp;Based on the judgment of input parameters, decide whether to execute the corresponding scp command.
&emsp;&emsp;Replace `D1` or `L1` with the specific filenames that need to be transferred.
&emsp;&emsp;To transfer a folder via scp, you need to add the `-r` option.
&emsp;&emsp;If only one file needs to be transferred at a time, a single scp command is sufficient.

***
&emsp;&emsp;当需求不复杂，只需要通过脚本一次传输一份文件，可简化为：

&emsp;&emsp;When the requirement is not complex and only involves transferring a single file through the script at a time, it can be simplified to:

```bash
#!/bin/bash

usr_target_host="U1@I1"

local_dir="P1"
target_dir="P2"

scp $local_dir/F1 $usr_target_host:$target_dir
```
### 2.2 设置ssh密钥对登录(Setting up SSH key-based login)

&emsp;&emsp;在编写好脚本后运行，会需要登录密码。可以通过设置密钥对登录的方式来规避对密码的输入。

&emsp;&emsp;After the script is written and run, a login password will be required. This can be avoided by setting up key-based login with a pair of keys.

***

#### 客户端设置(Client-side setup)

&emsp;&emsp;生成ssh密钥对，在终端输入：

&emsp;&emsp;To generate an SSH key pair, enter the following command in the terminal:

```shell
ssh-keygen -t rsa
```
&emsp;&emsp;使用默认文件夹，安全要求不高的环境，建议不设置密码。
&emsp;&emsp;将公钥复制到远程服务器。在终端输入：

&emsp;&emsp;For environments with low security requirements and when using the default directory, it is recommended not to set a passphrase.
&emsp;&emsp;Copy the public key to the remote server. Enter the following in the terminal:

```shell
ssh-copy-id U@I
```
&emsp;&emsp; `U`代表远端服务器用户名，`I`代表远端服务器IP。
&emsp;&emsp;这里需要输入一次远端服务器的密码。
&emsp;&emsp;此时客户端的设置就完成了。
&emsp;&emsp;有设置成功的提示后，可尝试ssh指令进行登录。如果登录不成功，说明还需要对远端服务器端进行修改。

&emsp;&emsp;`U` represents the username of the remote server, and `I` represents the IP address of the remote server. You will need to enter the password of the remote server once here.  This completes the setup on the client side.  After receiving a success message, you can try to log in with the ssh  command. If the login is unsuccessful, it indicates that further  modifications may be needed on the remote server side.

***

#### 服务器设置(Server-side setup)

&emsp;&emsp;登录远端服务器，修改`/etx/ssh`下的`sshd_config`文件。
&emsp;&emsp;添加：

&emsp;&emsp;Log in to the remote server and modify the `sshd_config` file located in the `/etc/ssh` directory.
&emsp;&emsp;Add:

```bash
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```
&emsp;&emsp;检查`~/.ssh`下的`authorized_keys`文件夹是否存在客户端添加好的密钥。
&emsp;&emsp;服务器端的设置也完成了。
&emsp;&emsp;可以尝试运行脚本传输文件。

&emsp;&emsp;Check if the `authorized_keys` file containing the added key from the client exists in the `~/.ssh` directory.
&emsp;&emsp;&emsp;&emsp;The server-side setup is also complete.
&emsp;&emsp;&emsp;&emsp;You can now try running the script to transfer files.

## 3 建议(Suggestion)

### 建议1(Suggestion 1)

&emsp;&emsp;在对服务器端进行修改时，将原`sshd_config`进行一次备份，且最好保留一个连接上远端服务器的终端作为备用。同时与同事或同学沟通一下，确保在修改的过程中远端服务器不会被关闭或重启（如果使用了跳板机，也需要保证跳板机不会被关闭或者重启），防止在设置的过程中，因`sshd_config`修改不当，导致无法通过ssh登录再次修改`sshd_config`的问题。每次修改完`sshd_config`后，开启新的终端尝试ssh能否不需要密码进行登录。若无法登录，则赶紧将`sshd_config`还原，再去排查问题。

&emsp;&emsp;When making changes to the server side, back up the original `sshd_config` file, and it's best to keep a terminal connected to the remote server as a backup. Also, communicate with colleagues or classmates to ensure that the remote server will not be shut down or restarted during the modification process (if you are using a jump server, make sure it will not be shut down or restarted as well), to prevent the issue of being unable to log in via SSH again to modify the `sshd_config` due to improper changes. After each modification to the `sshd_config`, open a new terminal and try to SSH to see if you can log in without a password. If you cannot log in, quickly restore the `sshd_config` to its original state and then troubleshoot the issue.

### 建议2(Suggestion 2)

&emsp;&emsp;在对服务器端进行修改后，确保按照安全最佳实践调整文件夹权限：
- 修改`.ssh/authorized_keys`文件权限不高于600。这样可以防止其他用户读取你的私钥，增加安全性。
- 修改`.ssh`目录权限不高于700。这意味着只有文件所有者可以读取、写入和执行目录中的文件。
- 修改`.ssh`上级目录权限不高于755。这允许所有者和组用户读取和执行，同时只允许所有者写入。

&emsp;&emsp;After making changes to the server side, ensure that you adjust the folder permissions according to security best practices:
&emsp;&emsp;Modify the permissions of the `.ssh/authorized_keys` file to not exceed 600. This prevents other users from reading your private key, increasing security.
&emsp;&emsp;Modify the permissions of the `.ssh` directory to not exceed 700. This means that only the file owner can read, write, and execute files in the directory.
&emsp;&emsp;Modify the permissions of the parent directory of `.ssh` to not exceed 755. This allows the owner and group users to read and execute, while only allowing the owner to write.
