# Linux

## [安装与配置](./install/README.md)

## 文本编辑器

### [Vim](./vim.md)

## Shell

### 数据流重定向

|   名称   |   覆盖   |   追加    |
| :------: | :------: | :-------: |
| `stdout` | `1> dst` | `1>> dst` |
| `stderr` | `2> dst` | `2>> dst` |

示例：

```shell
# 正常信息、错误信息 均输出到 屏幕
$ find ~/.. -name .bash_history
# 正常信息 输出到 stdout.txt 文件，错误信息 输出到 屏幕
$ find ~/.. -name .bash_history > stdout.txt
# 正常信息 输出到 stdout.txt 文件，错误信息 输出到 stderr.txt 文件
$ find ~/.. -name .bash_history > stdout.txt 2> stderr.txt
# 正常信息、错误信息 均输出到 stdout_stderr.txt 文件
$ find ~/.. -name .bash_history 2>&1 stdout_stderr.txt
```

### [SSH](./ssh.md)

## 进程管理
### 基本概念
本节的 ***程序 (program)*** 特指（存储在磁盘中的）可执行文件，而 ***进程 (process)*** 则是（被操作系统加载到内存中的）某个程序的运行实例。
操作系统在加载一个程序使其成为一个进程时，会为其分配一个 ***Process ID (PID)*** 并附上进程触发者的 ***User ID (UID)*** 及 ***Group ID (GID)***。

进程之间可能存在依赖关系，即一个进程由另一个进程触发。
- 被依赖者（即触发者）被称为 ***亲进程 (parent process)***。
- 依赖者（即被触发者）被称为 ***子进程 (child process)***。
- 子进程将亲进程的 UID、GID 继承下来，并以亲进程的 PID 作为自己的 ***Parent Process ID (PPID)***。

### 查看进程

|          命令           |                     功能                     |
| :---------------------: | :------------------------------------------: |
|         `ps -l`         |            只显示与自己相关的进程            |
|        `ps aux`         |                 显示所有进程                 |
|        `pstree`         |               显示进程依赖关系               |
|    `pstree -p 12345`    | 只显示 PID 为 `12345` 的那个进程的亲代及后代 |
|    `pstree -u user`     | 只显示用户名为 `user` 的那些进程的亲代及后代 |
|          `top`          |     动态显示所有进程状态（按 `q` 退出）      |
| `top -o +cpu -s 3 -n 4` |  CPU 升序、采样周期 `3` 秒、最多 `4` 个进程  |

### 管理进程

|          命令           |                     功能                     |
| :---------------------: | :------------------------------------------: |
|     `kill -9 12345`     |      强制关闭 PID 为 `12345` 的那个进程      |
|    `kill -15 12345`     |      正常关闭 PID 为 `12345` 的那个进程      |
|  `killall -9 process`   |      强制关闭名为 `process` 的那些进程       |
| `killall -s -9 process` |               显示但不执行操作               |


### 管理任务
由同一个 shell 进程触发的子进程称为 ***任务 (job)***。若系统只提供了一个 shell 进程，则用户通常需要将 ***任务 (job)*** 在 ***前台 (foreground)*** 与 ***后台 (background)*** 之间来回切换，以便让多个任务同时运行。

⚠️ 这里的 *前台*、*后台* 是相对于当前 shell 进程而言的；若这个 shell 进程是远程主机提供的，则退出 shell 意味着退出所有后台任务（仍在后台的任务会阻断 shell 的退出）。《[利用 SSH 访问远程 Linux 主机](./ssh.md)》介绍了如何访问远程主机以及在 *远程主机的后台* 运行任务。

|     命令      |             功能              |
| :-----------: | :---------------------------: |
|    `top &`    |    在后台启动并运行 `top`     |
|  `Ctrl + Z`   |   将当前任务暂停并移入后台    |
|    `jobs`     |       显示当前后台任务        |
|    `fg %2`    | 将任务 `2` 从后台移到前台运行 |
|    `bg %2`    |     令任务 `2` 在后台运行     |
|   `kill -l`   |        列出可用的信号         |
| `kill -2 %2`  |  相当于在后台按下 `Ctrl + C`  |
| `kill -9 %2`  |    强制结束并删除任务 `2`     |
| `kill -15 %2` |    按正常步骤结束任务 `2`     |
| `kill -19 %2` |  相当于在后台按下 `Ctrl + Z`  |

以下示例演示了表中主要命令的用法：
1. 在后台启动并运行一个任务：
   ```shell
   $ top &
   [1] 60137
   ```
    返回值 `[1]` 为其 ***任务编号 (job number)***，`60137` 为 PID。
1. 启动一个需要运行很长时间的任务，按 `Ctrl + Z` 将其暂停并移入后台：
   ```shell
   $ find / > temp.txt
   # 需要运行很长时间（除非该系统几乎没有被使用过）
   # 按 `Ctrl + Z` 将进程暂停并移入后台
   [2]+  Stopped                 find / > temp.txt
   ```
1. 此时已有两个后台任务，可用 `jobs` 查看：
   ```shell
   $ jobs
   [1]-  Stopped                 vim
   [2]+  Stopped                 find / > temp.txt
   $ jobs -l
   [1]- 60137 Stopped (tty output): 22top
   [2]+ 60141 Suspended: 18           find / > temp.txt
   ```
   其中 `+` 与 `-` 分别表示 *最后一个* 与 *倒数第二个* 被移入后台的任务。
1. 令（停止的）任务在前台或后台运行：
   ```shell
   $ fg %2  # `find ~ > temp.txt` 在前台恢复运行
   # 输出几乎不会停止，按 `Ctrl + Z` 移入后台
   $ bg %2  # `find ~ > temp.txt` 在后台恢复运行
   $ jobs
   [1]+  Stopped                 top
   [2]-  Running                 find ~ > temp.txt &
   ```
1. 向后台任务发送信号：
   ```shell
   $ kill %1  # 正常结束 任务[1]
   $ jobs  # 短时间内 任务[1] 的状态变为 Terminated
   [1]-  Terminated: 15          top
   [2]+  Stopped                 find / > temp.txt
   $ jobs  # 较长时间后，只剩下 任务[2]
   [2]+  Stopped                 find / > temp.txt
   $ kill -2 %2  # 在后台 Ctrl + C 任务[2]
   $ jobs  # 短时间内 任务[2] 的状态变为 Interrupt
   [2]+  Interrupt: 2            find / > temp.txt
   ```
