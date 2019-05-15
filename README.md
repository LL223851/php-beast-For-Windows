**php-beast** windows版本自行编译指南

**php-beast**这个工具还是挺好用的，可惜作者没有提供windows版编译方法，其他人提供编译好的自己只能用资源编辑工具修改默认密码，不方便，我不是程序员，只是有些兴趣，经多次百度，总算自行编译出来了，md文件也是第一次写，凑合看吧。

我自己电脑上的PHP版本是php-7.3.5-NTS-Win32-VC15-x86的，本次教程目标是为这个版本编译php-beast

1、资源准备

a）下载**php-beast**  

下载地址：https://github.com/liexusong/php-beast

b）下载**php-sdk-binary-tools**   

下载地址：https://github.com/microsoft/php-sdk-binary-tools

c）下载PHP7.35

https://windows.php.net/downloads/releases/php-7.3.5-src.zip

c）下载并安装编译环境

PHP7是用VC15编译的，没有VC15，所以到微软的官网上下载visual
studio 2017 entrpise的试用版即可（根据PHP版本的编译环境选择安装扩展编译VC版本）。
地址：
<https://www.visualstudio.com/zh-hans/?rr=https%3A%2F%2Fwww.microsoft.com%2Fzh-cn%2F>

2、目录准备及编译

编译前请确认是否安装好vs2017

a）新建一个编译目录，我自己准备的是E盘根目录下解压**php-sdk-binary-tools**   并重命名为php-sdk，测试编译工作目录为E:\php-sdk，截图如下：

![1](\1.png)

b)在当前目录E:\php-sdk下按住shift键同时右击鼠标，选择“在此处打开命令窗口”打开命令行，phpsdk-vc15-x86.bat脚本

![2](2.png)

c)再运行：phpsdk_buildtree phpdev，结束后会在E:\php-sdk\下生成“phpdev\vc15\x86”目录结构，解压php-7.3.5-src.zip，把php-7.3.5-src放到X86目录下，同时在x86下手动创建“prcl”目录，解压php-beast-master.zip，复制php-beast-master到prcl目录下，并改名为beast。

d)在命令行下运行“cd phpdev\vc15\x86\php-7.3.5-src” 进入“E:\php-sdk\phpdev\vc15\x86\php-7.3.5-src” ,执行“phpsdk_deps --update --branch master”命令用来获取SDK的依赖包，最终目录结果如下图：

![3](3.png)

```
E:\php-sdk\phpdev\vc15\x86\php-7.3.5-src
$ phpsdk_deps --update --branch master
```

等待下载结束后返回到命令模式。

e)修改代码。进入“E:\php-sdk\phpdev\vc15\x86\pecl\beast”目录修改php-beast代码

（1）**添加“win95nt.h”**，作者的源代码包中没有提供这个头文件。

（2）**修改config.w32代码**，原作者提供的缺少一个配置项，导致无法使用“--enable-execute-normal-script=yes”配置项，没有这个配置项的话，模块默认会禁止执行未加密的php代码，这个是参照config.m4修改而来的，在之前的编译过程中我耽误了很长时间，明明每个步骤都正确，编译出来的模块就是没法正常运行，提示500错误，一直排查为什么，后来仔细看了作者的readme文件，发现还有一个“enable-execute-normal-script”配置项，用configure --help查看却没有，翻看config.m4文件却发现有这个配置项，加上去后再次编译测试正常了

```js
// $Id$
// vim:ft=javascript

// If your extension references something external
ARG_WITH("beast", "for beast support", "yes,shared");
ARG_ENABLE("beast", "enable beast support", "yes,shared");
ARG_ENABLE("beast-debug", "enable beast debug mode", "no");
ARG_ENABLE("execute-normal-script", "enable execute normal PHP script mode","yes");

if (PHP_BEAST != "no") {
	if (PHP_BEAST_DEBUG != "no") {
		AC_DEFINE('BEAST_DEBUG_MODE', 1, 'Debug support in beast');
	}
	if (PHP_EXECUTE_NORMAL_SCRIPT != "yes"){
		AC_DEFINE('BEAST_EXECUTE_NORMAL_SCRIPT', 0, 'disable execute normal PHP script mode');
	}else{
		AC_DEFINE('BEAST_EXECUTE_NORMAL_SCRIPT', 1, 'enable execute normal PHP script mode');
	}

	EXTENSION("beast", "beast.c aes_algo_handler.c des_algo_handler.c base64_algo_handler.c beast_mm.c spinlock.c cache.c beast_log.c global_algo_modules.c header.c networkcards.c tmpfile_file_handler.c file_handler_switch.c shm.c", true);
}

```

修改至此编译可以通过，但是会提示一些警告信息，自己自行修改吧

**编译的目的就是为了编译处属于自己修改了密码的dll文件，所以原作者说要大家修改的地方如下：**

> ## 制定自己的php-beast
>
> `php-beast` 有多个地方可以定制的，以下一一列出：
>
> *1.* 使用 `header.c` 文件可以修改 `php-beast` 加密后的文件头结构，这样网上的解密软件就不能认识我们的加密文件，就不能进行解密，增加加密的安全性。
>
> *2.* `php-beast` 提供只能在指定的机器上运行的功能。要使用此功能可以在 `networkcards.c` 文件添加能够运行机器的网卡号，例如：
>
> ```c
> char *allow_networkcards[] = {
> 	"fa:16:3e:08:88:01",
>     NULL,
> };
> ```
>
> 这样设置之后，`php-beast` 扩展就只能在 `fa:16:3e:08:88:01` 这台机器上运行。另外要注意的是，由于有些机器网卡名可能不一样，所以如果你的网卡名不是 `eth0` 的话，可以在 `php.ini` 中添加配置项： `beast.networkcard = "xxx"` 其中 `xxx` 就是你的网卡名，也可以配置多张网卡，如：`beast.networkcard = "eth0,eth1,eth2"`。
>
> *3.* 使用 `php-beast` 时最好不要使用默认的加密key，因为扩展是开源的，如果使用默认加密key的话，很容易被人发现。所以最好编译的时候修改加密的key，`aes模块` 可以在 `aes_algo_handler.c` 文件修改，而 `des模块` 可以在 `des_algo_handler.c` 文件修改。
>
> ------
>
> ## 

经过上面代码准备和代码修改，就可以执行编译了，三步走：

执行buildconf命令，并等待结束

```
E:\php-sdk\phpdev\vc15\x86\php-7.3.5-src
$ buildconf
```

执行configure命令并等地结束

```
E:\php-sdk\phpdev\vc15\x86\php-7.3.5-src
$ configure --enable-execute-normal-script=yes
```

PHP的configure有很多参数，自行百度添加，php默认是ts模式，使用nts需要加上“--disable-zts ”参数

注意：**--enable-execute-normal-script=yes** 用于关闭beast禁止执行非加密代码功能

```
E:\php-sdk\phpdev\vc15\x86\php-7.3.5-src
$ configure --enable-cli --disable-zts --enable-execute-normal-script=yes
```

执行后如下图

![4](4.png)

红框中就是beast模块，说明之前做的都正确了。

这面configure时我没有关闭其他模块，而是默认全开了，如果你电脑配置不高的话建议关闭不必要的模块，毕竟我们只是要编译php-beast这一个扩展，扩展编译全开的话最后编译时很耗时间，我的电脑配置可能高一些，几分钟就好了，结束后如下图

![5](5.png)

最后一步执行nmake命令

```
E:\php-sdk\phpdev\vc15\x86\php-7.3.5-src
$ nmake
```

![6](6.png)

编译成功

![7](7.png)

至此，属于自己修改过密码等定制的windows版的dll文件编译完毕。

**注意事项：**

1、目录结构要正确

2、版本要对应，自己用的PHP的版本是多少，X86还是X64，ts还是nts，运行库是VC哪个版本。

3、configure 配置选项要正确，是ts还是nts，是否允许执行未加密的php代码。

4、需要添加的文件“`win95nt.h`”头文件，需要修改的文件“`config.w32`”。

5、定制自己的加密模块可以修改的文件：

​    a、`header.c`文件修改encrypt_file_header_sign[]，数组里面随便修改一下，破解的人无法根据文件头判断是否是beast加密的。

​    b、`aes_algo_handler.c`文件修改key[]数组，这里面是aes加解密密码。

​    c、`des_algo_handler.c`文件修改key[8]数组，这里面是des加解密密码。

​    d、`networkcards.c`文件*allow_networkcards[]数组，这里面是添加网卡MAC地址，格式为"aa:bb:cc:dd:ee:ff"，其他要求见原作者说明。

**其他说明：**

​        有经验的人看到PHP扩展使用了“php_beast”模块就知道代码被php_beast加密了，即便是自己编译的修改了头文件和加密密码的或者是绑定了MAC地址的php_beast模块，也是可以通过WinHex这样的软件查看文件头、密码和绑定的MAC地址的，有了这些信息对会编译的人这样的加密也是形同虚设的。
