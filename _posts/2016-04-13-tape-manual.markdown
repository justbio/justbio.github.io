---
layout: post
category: "read"
title:  "磁带机的手动操作"
---

Netvault太贵，amada太复杂，所以手动操作磁带。。。

### 磁带机设备查看  
设备名一般是/dev/st0和/dev/nst0

st0是自动回卷设备，nst0是非回卷设备。
<!-- more --> 

### 磁带的数据结构和数据操作   
![](../assets/739083-20160413101306566-1430165615.png)

BOT:Begin of Tape,磁带开头的标记  
FileX:磁带中写入的文件数据,号码标记从0开始  
EOF:End of File,文件块结束的标记  
blockX:磁带的块,文件是以区块为单位来写入的  
IRG:Inter Record Gap,块和块之间的分隔标记  

文件首次写入 tar -cvf 磁带装置名 文件名  
文件继续写入 tar -rvf 磁带装置名 文件名  
文件读出 tar -xvf 磁带装置名 文件名  
列出文件列表 tar -tvf 磁带装置名  

*仅能列出磁头当前所在位置的File Number中的文件

### 回卷设备的操作和磁头活动

文件首次写入  
![](../assets/739083-20160413101306895-285270777.png)

磁头从①位置开始写入文件A，到②位置写入完毕，打上EOF标签。  
然后磁头回卷，回到①位置。
 
文件继续写入  
![](../assets/739083-20160413101307176-1452372608.png)

磁头从①位置开始寻址，到②位置去掉EOF标签，开始继续写入文件B,  
到③位置写入完毕,打上EOF标签。然后磁头回卷，回到①位置。  
由于写入之前要进行寻址，所以从命令执行到开始写入之间有一段寻址的时间，文件越大寻址越慢。

文件读出  
![](../assets/739083-20160413101307176-1452372608.png)

磁头从①位置开始寻址，到②位置后开始读出文件，到③位置读出完毕。  
然后磁头回卷，回到①位置。读出的文件可以在当前目录找到。  
由于写入之前要进行寻址，所以从命令执行到开始写入之间有一段寻址的时间，文件越大寻址越慢。

注意

一般来说回卷设备只有一个FileNumber，在磁带有文件的情况下继续写入数据必须用tar -rvf 命令。  
如果错用了tar –cvf的话磁带中所有的数据会被覆盖

### 非回卷设备的操作和磁头活动
文件第一次写入  
![](../assets/739083-20160413101307738-509828293.png)
磁头从①位置开始写入文件A，写入完毕后打上EOF标签。  
然后磁头停留在EOF之后②位置。

文件第二次写入  
![](../assets/739083-20160413101307973-936780773.png)
磁头刚才停留在②的位置，直接开始写入文件B,写入完毕后打上EOF标签。  
然后磁头停留在EOF之后③位置。

注意

非回卷设备原则上不可以用tar –rvf命令。   
因为如果磁头不是在最后一个File中的情况下使用tar –rvf命令，那么这个File之后的所有文件就会被抹去。

### 非回卷设备的基本概念
非回卷设备有很重要的概念：
File Number 和 Block Number  
![](../assets/739083-20160413101308285-1120443866.png)
File Number是指磁头现在在哪个File中
Block Number是指磁头在本File的哪个Block开始处

例：  
File Number=0 Block Number=0 是指磁头在上图的1处  
File Number=0 Block Number=1 是指磁头在上图的2处

另外有Number为-1的情况  
File Number= -1是指磁头在BOT（3处）  
Block Number= -1是指磁头不在Block的开头处  

例：  
File Number=-1 Block Number=-1  
是指磁头在上图的3处,一般只有在操作错误的时候才会出现  
File Number=0 Block Number=-1  
是指磁头在上图的4处或5处等等，处于不可用的位置  

### 非回卷设备的常用磁头操作
命令（设备统一为nst0）|说明|完成后磁头位置变化|
:-|:-|:-
|tar -cvf /dev/nst0 文件名|写入文件|File Number加1，Block Number=0
|tar -xvf /dev/nst0 文件名|读出文件|File Number不变，Block Number变为文件末尾的block号码|
|tar -tvf /dev/nst0|列出文件列表|File Number不变，Block Number变为文件末尾的block号码|
|mt -f /dev/nst0 status|查看磁头位置|不变|
|mt -f /dev/nst0 rewind|回卷磁头|File Number=0，Block Number=0|
|mt -f /dev/nst0 eod|移到数据末尾|File Number为最大加1，Block Number=-1|
|mt -f /dev/nst0 fsf 数字|前进磁头，定位在后一个文件的第一块上|File Number为当前File号码+数字，Block Number=0
|mt -f /dev/nst0 fsfm 数字|前进磁头，定位在前一个文件的最后块上|File Number为当前File号码+数字-1，Block Number=-1|
|mt -f /dev/nst0 bsf 数字|倒退磁头，定位在前一个文件的最后块上|File Number为当前File号码-数字，Block Number=-1|
|mt -f /dev/nst0 bsfm 数字|倒退磁头，定位在后一个文件的第一块上|File Number为当前File号码-数字+1，Block Number=0|