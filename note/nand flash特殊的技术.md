# Nand Flash特殊的技术

> Nand Flash被我们广泛使用，如SSD，SD卡，智能手机的外部存储等。
>
> 基于Nand Flash的存储设备，涉及到一系列有趣的技术（点），如：ECC，坏块管理，页寄存器，映射表，垃圾回收，磨损平衡，断电保护，写入放大，预留空间，寿命....

聪明的你们是不是对这些技术点了如指掌呢？

## Nand Flash硬件技术点

### Nand结构

Nand Flash ⇒ Chip ⇒ Plane ⇒ Block ⇒ Page ⇒ oob

### 页寄存器（页缓存）（Page Register）

对于Nand的每一片（Plane），都有一个对应的区域专门用于存放，将要写入到物理存储单元中去的或者刚从存储单元中读取出来的，一页的数据，这个数据缓存区，本质上就是一个缓存buffer，但是只是此处datasheet里面把其叫做页寄存器page register而已，实际将其理解为页缓存，更贴切原意。

### 不能随机访问

在一个块内，对每一个页进行编程的话，必须是顺序的，而不能是随机的。比如，一个块中有128个页，那么你只能先对page0编程，再对page1编程，。。。。，而不能随机的，比如先对page3，再page1，page2，page0，page4，。。。

> 关于此处对于只能顺序给页编程的说法，只是翻译自datasheet，但是实际情况却发现是，程序中没有按照此逻辑处理，可以任意对某Block内的Page去做Program的动作，而不必是顺序的。但是datasheet为何如此解释，原因未知，有待知情者给解释一下。



## Nand Flash文件系统技术点

在嵌入式系统开发中，经常使用Nand Flash的裸片，通过MTD子系统，文件系统（JFFS2，UBIFS）对Flash进行使用。

> 补充：Ext2/2/4文件系统可以通过FTL实现对flash的支持，FTL可以将闪存flash模拟成磁盘结构，从而实现对基于磁盘的文件系统的支持。

众所周知，Nand擦写具有如下特点：

- Nand写入操作只能在空或已擦除的单元内进行，更改数据时，将整页拷贝到缓存（Cache）中修改对应页，再把更改后的数据挪到新的页中保存，将原来位置的页标记为无效页；

- 以page为单位写入，以Block为单位擦除；擦除Block前需要先对里面的有效页进行搬迁。

  文件系统通常实现了垃圾回收和磨损均衡，保证Nand在有限的擦写周期内的正常使用。

- 每个Block都有擦除次数限制（有寿命），擦除次数过多会成为坏块（bad block）。

> 思考题：Linux MTD子系统提供磨损均衡，垃圾回收，坏块管理吗？

### 磨损均衡

参考具体文件系统实现。磨损均衡可以使得每个Block均匀被擦写使用，避免坏块产生。

有很多不同的磨损平衡机制，大体可以分为两大类：动态WL、静态WL。

- 动态WL：使用Block进行擦写时，优先挑选P/E值低的Block。
- 静态WL：把P/E值低的Block中的数据挪到P/E值高的Block中存放。

### 垃圾回收

参考具体文件系统实现。垃圾回收就是把几个Block中的有效数据集中搬移到新的Block上去， 然后再把这几个Block擦除掉。这样做的好处一方面减少寻址负担，另一方面留出更多的空闲 Block。

### ECC

当往NAND Flash的page中写入数据的时候，每256字节我们生成一个ECC校验和，称之为原ECC校验和，保存到PAGE的OOB（out-of-band）数据区中。当从NAND Flash中读取数据的时候，每256字节我们生成一个ECC校验和，称之为新ECC校验和。

将从OOB区中读出的原ECC校验和新ECC校验和按位异或，若结果为0，则表示不存在错（或是出现了 ECC无法检测的错误）；若3个字节异或结果中存在11个比特位为1，表示存在一个比特错误，且可纠正；若3个字节异或结果中只存在1个比特位为1，表示 OOB区出错；其他情况均表示出现了无法纠正的错误。

### 坏块管理

如果在对一个块的某个page进行编程的时候发生了错误就要把这个块标记为坏块。

一般在OOB标记是否是坏块。

> 坏块管理策略
>
> （1）坏块跳过策略：遇到坏块跳过，存放进好块里。
>
> （2）坏块替换策略：将NAND存储空间中预留出一些块作为保留块，再用替换表管理。

> 【思考题】Uboot采用了什么样的Nand坏块管理方法？
>
> 提示：BBT

### JFFS2文件系统

JFFS2文件系统是基于MTD设备而设计的，实现了垃圾回收、损耗均衡等机制。

JFFS2不支持write-back, JFFS2文件系统的所有变化都是立刻同步到flash介质上。事实上，JFFS2有一个很小的buffer大小是NAND page size（如果是 NAND flash）。这个buffer保存这最后要写的数据，一旦buffer写满立刻就会执行flush。



### UBIFS文件系统

UBIFS是运行于raw flash之上。

UBIFS是一个异步文件系统，和其他文件系统一样，UBIFS也利用了**page cache**.  page cache是一个通用的linux内存管理机制。page cache可以非常大以cache更多的数据。当你写一个文件时，首先写到page cache中，置为dirty, 然后write操作返回（特例是文件是同步的）。 后面数据会通过write-back机制写回。

**write-buffer**是UBIFS特定的buffer, 它位于page cache和flash介质之间。



思考题1：fsync()和fdatasync()的区别？

思考题2：总所周知，fsync()有时候执行的时间很长，如何优化？

> Linux有几个内核参数可以用来调整write-back。
>
> 1. dirty_writeback_centisecs: linux周期性write-back线程写出dirty数据的周期，这个机制可以确保所有的脏数据在某个时间点都可以写入介质
> 2. dirty_expire_centisecs: dirty数据的过期周期，这是数据为dirty的最大时间。 过了这个时间，dirty数据会被周期性write-back线程写回介质。
> 3. dity_background_ratio: dirty数据与全部内存的最大百分比。 当脏数据变多的时候，周期性的write-back线程开始同步数据使得脏数据比例变小。
> 4. dirty_ratio: dirty数据与全部内存的最大百分比

思考题3：UBIFS是一个异步文件系统，因此有掉电一致性问题，JFFS2有掉电一致性问题吗？

思考题4：UBIFS同步模式？

> UBIFS支持标准的sync mount 选项，可以用来disable UBIFS write-back和write-buffer，使得写过程完全同步。

思考题5：Linux下通过同步的方式打开文件（O_SYNC ），能保证一致性吗？



## SSD固件技术点

SSD主控和固件完成了GC，Trim，磨损均衡（一个SSD上有多个Nand颗粒的话也要均衡起来），断电保护等功能。

为了提升读写性能，SSD通常使用SDRAM做缓存。SSD板上还会加上钽电容或者超级电容，以保证断电的时候数据还能写入Nand。

为了提升写入性能，一些TLC固态会包含一定容量的SLC缓存。

> 一块250G的TLC固态硬盘，进行大文件写入，前面15G数据写入速度为1500M/s，此后写入速度骤降到150M/s，请思考是什么原因？





参考资料：

https://blog.csdn.net/kunkliu/article/details/78401902

https://www.ibm.com/developerworks/cn/linux/l-jffs2/

[Linux UBI子系统设计初探](https://www.cnblogs.com/wahaha02/p/4814698.html)

[SSD固态硬盘的GC与Trim](https://www.cnblogs.com/Christal-R/p/6846829.html)

https://www.cnblogs.com/Christal-R/p/7230304.html

https://blog.csdn.net/lights_joy/article/details/51649765