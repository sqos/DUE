# 开放网络安装环境

开放网络安装环境基于Debian 8 (Jessie), Debian 9 (Stretch) 或者 Debian 10 (Buster)镜像创建。  
**Note:** 大部分编译目标使用Debian 9。Debian 10的支持目前处理beta测试阶段。  
  

## ONIE构建环境创建示例:  
创建基于Debian 11的构建环境: ./due --create --from debian:11 --description "ONIE Build Debian 11" --name onie-build-debian-11 --prompt ONIE-11 --tag onie --use-template onie  
**或者**  
创建基于Debian 10的构建环境: ./due --create --from debian:10 --description "ONIE Build Debian 10" --name onie-build-debian-10 --prompt ONIE-10 --tag onie --use-template onie  
**或者**  
创建基于Debian 9的构建环境: ./due --create --from debian:9  --description "ONIE Build Debian 9" --name onie-build-debian-9 --prompt ONIE-9 --tag onie --use-template onie  
**或者**   
创建基于Debian 8的构建环境: ./due --create --from debian:8  --description "ONIE Build Debian 8" --name onie-build-debian-8 --prompt ONIE-8 --tag onie-8 --use-template onie  

### 第一个例子的解释:  
  * 基于Debian 11镜像；  
  * 目标镜像名字为onie-build-debian-11；  
  * 目标镜像tag为onie；  
  * 设置镜像中用户的PS1提示为ONIE-11，以提供更友好的提示上下文；  
  * 在创建配置文件目录时，合并./templates/onie中的文件；  

(**译者注:** 此处原英文版并未根据*ONIE构建环境创建示例*进行同步修订，此处译者使用其第一个示例进行说明。)

## 附件配置
此处列出该容器独有的修订。  

### The oniebuild user
`post-install-config.sh.template`脚本安装所有的ONIE构建依赖，并且创建一个用来从DUE环境登陆到容器的名为oniebuild用户。  

`--username oniebuild`该用户添加/sbin和/usr/sbin到path路径，并作为git user.name和user.email配置防止执行git操作时，没有相关配置的告警。

使用 oniebuild 帐户登录时，您可能会注意到延迟，它会更改容器中的onebuild帐户以匹配正在执行用户的ID。  

### Python-sphinx
...它仅安装在Debian 9 (Stretch) 和 10 (Buster)的基础镜像中，目的时更新OINE文档，配置他用于ONIE的季度发布。用户可以通过编辑`post-install-config.sh.template`进行添加/删除。  

# 使用

## 作为本地用户构建

您可以使用 `due --run` 并选择在前面步骤中构建的镜像，这将：  

1.  挂载您的"home"目录（这不必一定是您本地主机的'~/'目录，请查阅`docs/GettingStarted.md`）；  
2.  使用您的用户名和用户 ID 在容器中为您创建一个帐户；  
3.  Source a .bashrc,并允许访问其他任何 . 开头的隐藏文件；  
4.  ...现在您可以切换到 onie 目录，从命令行构建；  


## 无交互构建

有多种方式可无需登录容器进行构建ONIE。  
**小贴士** 如果已知构建需要使用的镜像，使用`--run-image image-name:image-tag`参数跳过镜像选择交互界面。  


有关更多信息，请参阅`DUE/docs/Building.md`。  

开始: 首先进入onie/build-config目录。
如果DUE直接使用命令执行而不是登录到容器，DUE将自动挂载当前目录。  

然后执行以下任一流程：

### Using `--command`
**目的:** 在Bash shell中--command之后执行所有的其他内容。  
**描述:** 此处容器在 Bash shell 中执行 --command 之后执行所有的其他内容。  
**示例:** due --run --command export PATH="/sbin:/usr/sbin:\$PATH" \; make -j4 MACHINE=kvm\_x86\_64 all  
**示例:** due --run --command export PATH="/sbin:/usr/sbin:\$PATH" \; make MACHINEROOT=../machine/accton MACHINE=accton_as7112_54x all demo recovery-iso  

**NOTES:**  
1.  **\;** 用于分隔要在容器中运行的两个命令. 没有 **'\'** ，调用 shell 会将 **';'** 之后的所有内容解释为要在 _after_ 调用 DUE 后运行的命令。  
这可能会造成混乱并使调试复杂化，因为第二个命令在容器外失败并不明显。  
2.  只有在创建 recovery-iso 目标时，才需要将 /sbin 和 /usr/sbin 添加到路径中。


### Using `--build`  
**目的:** 使用容器的 `duebuild` 脚本执行附加配置。  
**描述:** 这里，`--build` 是调用容器中`/usr/local/bin/duebuild` 脚本的快捷方式，并提供一些抽象，以免打扰用户构建的细节。  
**小贴士:** 通过运行`due --run --build help`获取容器的 `duebuild` 脚本的帮助。  


#### Using `--build --default`  
**目的:** 这将构建一个目标，该目标应该始终用于对构建环境进行完整性检查。  
**描述:** 这将根据镜像的角色而有所不同，但在 ONIE 的情况下，它将构建 kvm-x86 虚拟机，因为该目标不需要切换硬件来运行，因此应该可以在任何地方使用。  


#### Using `--build --cbuild`  
**目的:** `--cbuild` 选项允许在构建之前对环境进行默认配置，但这确实不适用于 ONIE，所以这里它只是直接通过 make 字符串，`--command`具有同样的行为。  
**示例:** due --run --build --cbuild make -j4 MACHINE=kvm_x86_64 all demo recovery-iso  
**示例:** due --run --build --cbuild make -j4 MACHINEROOT=../machine/accton MACHINE=accton_as7112_54x all demo recovery-iso  


#### 为ONIE使用附加参数duebuild  
在这里，duebuild 脚本可以通过将构建详细信息指定为参数将其传递给 makefile，来为构建提供一些便利。益处是 MACHINEROOT 可以通过搜索 MACHINE 来确定。  
然而，这只是实现最终 makefile 调用的另一种方式。  

**示例:** due --run --build --jobs 4 --machine kvm_x86_64 --build-targets all demo recovery-iso  
**示例:** due --run --build --jobs 4 --machine accton_as7112_54x --build-targets all demo recovery-iso  

## 挂载主机目录  
ONIE 容器将尝试从主机系统挂载以下目录。不这样做可能会导致错误（操作绝对无法继续）或警告，只是为了让用户知道正在发生的事情。  

#### /home/*username*  
**If missing:** Error.    
**用途:** 访问用户的默认环境和配置。建议用户的主目录始终挂载。  

#### Current Working Directory  
**If missing** Error.  
**用途:** 当创建容器只是为了运行命令而不是支持交互式登录时，将始终挂载当前工作目录。  
 
#### /var/cache/onie  
**If missing:** Warning.  
**用途:** Download cache.   
不必总是从其原始站点或 [OpenCompute Mirror](http://mirror.opencompute.org/onie) 下载源包，ONIE 可以在 `/var/cache/onie/downloads` 中搜索源包，前提是 `make` 调用时设置了 `ONIE_USE_SYSTEM_DOWNLOAD_CACHE=TRUE`。如果主机系统上存在此目录，则所有运行的 ONIE 容器都可以访问它，从而节省时间和带宽。填充目录留给系统管理员作为练习，运行`wget –recursive –cut-dirs=2 –no-host-directories –no-parent –reject “index.html” http://mirror.opencompute.org/onie/`  
...可能是一个很好的起点。  


#### /dev  
**If missing:** Warning.  
**用途:** Loopback mounting filesystems.  
仅建议为需要loopback设备进行挂载文件系统的某些工作流挂载主机的 `/dev` 目录。由于主机系统的 `/dev` 目录可以通过容器中的操作进行修改，因此容器必须在运行时设置 Docker 的 `--privileged` 选项才能挂载（见下文）。  
请注意，如果不希望以特权运行，通常可以在容器外部进行loopback设备挂载操作的替代工作流程。  


### 运行带有参数--privileged的容器  
某些 ONIE 工作流程，例如为安全启动构建 KVM 目标，需要利用对主机系统的 /dev 目录的访问来环回挂载文件系统并减少构建中所需的用户交互量。  
**示例:** due --run --dockerarg --privileged  
**示例:** due --run-image due-onie-build-debian-10 --dockerarg --privileged  



## Debugging  
或者各种失败方式的描述性集合。估计此列表会持续增长。  

**错误:** 配置失败. In particular, "Can't find bash version > 3.1"  
**解决方案:** 这发生在我使用 ;而不是 \;使用 --command 构建 ONIE 目标。  
**解释:** 容器运行，获取 ; 之前的文本，然后 bash 在本地执行其余部分 - 在容器外部 - 并且在构建结束时，这种状态变化早已从屏幕上滚动。  


#  补充说明:
None.  


