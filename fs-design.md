# Filesystem design

## 前言

Orange FS v1.0为我们提供了一个比较基础的文件系统,根据原书中的描述

> superblock通常也叫超级块关于文件系统的Metadata我们统统记在这里。sector map是一个位图它用来映射扇区的使用情况用1表示扇区已被使用0表示未使用。i-node是UNIX世界各种文件系统的核心数据结构之一我们把它借用过来。每个i-node对应一个文件用于存放文件名、文件属性等内容inode_array就是把所有i-node都放在这里形成一个较大的数组。而inode map就是用来映射inode_array这个数组使用情况的一个位图用法跟sector map类似。root数据区类似于FAT12的根目录区但本质上它也是个普通文件由于它是所有文件的索引所以我们把它单独看待。为了简单起见我们的文件系统暂不支持文件夹也就是说用来表示目录的特殊文件只有这么一个。这种不支持文件夹的文件系统其实也不是我们的首创历史上曾经有过而且这种文件系统还有个名字叫做扁平文件系统Flat File System。

另外在实现时,原书有描述

> 我们的inode结构体目前很简单其中i_start_sect代表文件的起始扇区i_nr_sects代表总扇区数i_size代表文件大小。在这里我们使用一个很笨拙的方法来存储文件那就是事先为文件预留出一定的扇区数i_nr_sects以便让文件能够追加数据i_nr_sects一旦确定就不再更改。这样做的优点很明显那就是分配扇区的工作在建立文件时一次完成从此再也不用额外分配或释放扇区。缺点也很明显那就是文件大小范围在文件建立之后就无法改变了i_size满足i_size ∊ [0, i_nr_sects × 512]。
>
> 笔者最终决定使用这种有明显缺陷的i-node原因还是为了简单即便它有千般不是它实在是太简单了以至于可以大大简化我们的代码而且它还是能用的。等到我们有很多文件需要灵活性时我们再推出一个v2.0那时需要改进的怕也不仅仅是一个i-node而已所以在做第一步时就让它傻一点吧。

可惜原书始终没有给出一个更现代的v2.0,目前的文件系统显然无法支持诸如多层目录路径以及灵活可变的文件大小来满足正常的需求.

![image-20241204154805036](/home/scarletborder/Codes/c/whu-os-final-exp/assets/image-20241204154805036.png)

## 我们的工作

在Orange FS v1.0的基础上改进为Orange FS fork v2.0

目前支持多层目录路径以及重新实现配合的创建文件,打开文件,读取文件,stat文件,删除文件等方法,更针对于实验课任务来说

- 列出当前目录文件`ls`,支持`-l`以显示文件属性,`-a`显示隐藏文件(`.`开头的文件).

- 创建文件,

  - `touch`命令创建空白文件,目前不支持任何flag;
  - `mkdir`命令创建空白文件夹,同样不支持任何flag

- 打开或编辑指定文件

  - `./executable`执行一个可执行文件
  - `si ./some.txt`一个模仿vi的超简单cli文件编辑器.

  注意要cursor,改一个叫做disp_pos的(0<= value<80*25)的东西,写几个heade.h支持一下自定义的vi-min

- 删除文件,`rm`命令额外新增`-r`,`-f`flag,效果和通用Linux中的同名flag相同.

