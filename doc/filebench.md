# FileBench

用户可以从头开始描述所需的工作负载或使用Filebench 附带的（有或没有修改）工作负载特性（例如，mail-、web-、file-和database-server工作负载）。Filebench 同样适用于微观和宏观基准测试

## 安装

第一步：生成自动构建脚本

需要libtoolize和automake tools。

```shell
libtoolize
aclocal
autoheader
automake --add-missing
autoconf
```

第二步，编译安装
make sure **yacc** and **lex** are available in your system(这两个是语法分析工具)

```shell
./configure
make
sudo make install
```

Libraries have been installed in:
   /usr/local/lib/filebench

## 如何使用

### 使用Workload Model Language (WML)生成用户定义的负载

工作负载描述文件以 `.f` 作为文件扩展名。

```bash
01  define fileset name="testF",entries=1000,filesize=16k,prealloc,path="../tmp"
02
03  define process name="readerP",instances=2 {
04    thread name="readerT",instances=3 {
05      flowop openfile name="openOP",filesetname="testF"
06      flowop readwholefile name="readOP",filesetname="testF"
07      flowop closefile name="closeOP"
08    }
09  }
10
11  run 60
```

Filebench 中的四个主要实体是 `filesets`、由`processes` 组成的 `threads` 和 `flowops`。在第一行中，我们定义了一个 fileset，其中包含 10,000 个大小为 16KiB 的文件，每个文件位于 /tmp 目录中。Filebench 被指示在执行实际工作负载之前预先创建fileset中的所有文件。

在第三行和第四行中，我们定义了两个相同的进程，每个进程由三个相同的线程组成。Filebench 中的每个线程都会循环重复其中定义的流程（操作）。
线程：在“testF”文件集中打开一个文件，完全读取文件，然后关闭
它。最后，在第 11 行中，我们指示运行工作负载 60 秒。

在更复杂的工作负载中，可以定义任意数量的文件集、多个不同的进程和线程、使用各种流程和属性等等。有关详细信息，请参阅完整的 WML 词汇表 <https://github.com/filebench/filebench/wiki/Workload-model-language>

假设自定的负载保存在“readfiles.f”文件中，然后可以通过运行 `filebench -f readfiles.f` 命令生成相应的工作负载。

```shell
sudo filebench -f readfiles.f
```

### 示例 2：预定义的工作负载

Filebench 带有几个预定义的微观和宏观工作负载（例如，webserver、fileserver、mailserver），它们也在 WML 中描述，

与上面示例 1 中的工作负载没有什么不同。在源代码树中，工作负载位于 workloads/ 目录中，并且通常在“make install”期间安装在 /usr/local/share/filebench/workloads/ 中（尽管这可能因安装而异）。

我们*不*建议直接使用 workloads/ 或 /usr/local/share/filebench/workloads/ 目录中的工作负载文件。主要原因是这些工作负载*没有适合特定系统的大小*（例如，就数据集大小而言）。例如，网络服务器的初始数据集大小
工作负载仅略大于 16MiB，这通常不是您想要测试包含数 GB RAM 的系统的大小。

因此，改为将 webserver 工作负载复制到任何其他目录：

```shell
cp /usr/local/share/filebench/workloads/webserver.f mywebserver.f
```

增加文件集的数据集大小属性到适当的值。最后，运行工作负载：

```shell
filebench -f mywebserver.f
```

关于如何扩展 Filebench 工作负载的扩展讨论可以在 <https://github.com/filebench/filebench/wiki/Scaling-Filebench-workloads>
中找到。

## 常见问题

<https://github.com/filebench/filebench/issues/156>

```shell
sudo su
echo 0 > /proc/sys/kernel/randomize_va_space
exit
```

> proc/sys/kernel/randomize_va_space用于控制Linux下 内存地址随机化机制（address space layout randomization)，有以下三种情况(进程地址随机化，比如每一次运行的栈起始地址都不一样)
> 0 - 表示关闭进程地址空间随机化。
> 1 - 表示将mmap的基址，stack和vdso页面随机化。
> 2 - 表示在1的基础上增加栈（heap）的随机化。
