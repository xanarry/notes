```c
#include <linux/string.h>
#include <asm/page.h>
#include <linux/kernel.h>
#include "ToolFunctions.h"

unsigned long linearAddrToPhysicalAddr(unsigned long linearAddr)
{
	 /*
    一级页表：gdVal(39-47位)
    二级页表：udVal(30-38位)
    三级页表：mdVal(21-29位)
    四级页表：tVal(12-20位)
    页内偏移：offsetVal(0-11位)
    +-----------------+-+---------++---------++---------++---------+-+------------+
    |     reserved    | |globa dir||upper dir||midle dir||  table  | |   offset   |
    +-----------------+-+---------++---------++---------++---------+-+------------+
    |<------63:48---->| |<-47:39->||<-38:30->||<-29:21->||<-20:12->| |<---11:0--->|
    +-----------------+-+---------++---------++---------++---------+-+------------+
    1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111
    | 1    2    3    4    5    6    7    8    9   10   11   12   13   14   15   16|
    +-----------------+-+---------++---------++---------++---------+-+------------+
    对于每一级的页表项，第12-51位保存了下一级页表的地址。
    */	

	unsigned long cr3;
	unsigned long globalDirPhysAddr;
	unsigned long upperDirPhysAddr;
	unsigned long midleDirPhysAddr;
	unsigned long pageTablePhysAddr;
	unsigned long pagePhysAddr;

	unsigned long globalDirIndex;
	unsigned long upperDirIndex;
	unsigned long middleDirIndex;
	unsigned long pageTableIndex;
	unsigned long pageOffset;

	unsigned long globalDirItem;
	unsigned long upperDirItem;
	unsigned long middleDirItem;
	unsigned long pageTableItem;
	unsigned long page;
	unsigned long __force_order;

	unsigned long physAddr;

	globalDirIndex = (linearAddr & 0x0000FF8000000000UL) >> 38; //global dir index
	upperDirIndex  = (linearAddr & 0x0000007FC0000000UL) >> 29; //upper  dir index
	middleDirIndex = (linearAddr & 0x000000003FE00000UL) >> 21; //middle dir index
	pageTableIndex = (linearAddr & 0x00000000001FF000UL) >> 12; //table      index
	pageOffset     = (linearAddr & 0x0000000000000FFFUL) >>  0; //page offset

	//取出cr3寄存器的值
	asm volatile("mov %%cr3, %0\n\t" : "=r" (cr3), "=m" (__force_order));
	DEBUG_PRINT("cr3: %lx\n", cr3);

	//每级页表项的12-51位是下级表的物理基地址，& 0x0007FFFFFFFFF000UL取出12-51位

	/*处理一级页表*/
	//找出globaldir表的物理地址
	globalDirPhysAddr = cr3 & 0x0007FFFFFFFFF000UL; //cr3寄存器的低12为没有被使用，取高地址部分
	//通过偏移得到目标表项，并取值赋给globalDir
	globalDirItem = *((unsigned long *)(__va(globalDirPhysAddr + globalDirIndex * 8))); 

	DEBUG_PRINT("finish 处理1级页表\n");

	/*处理二级页表*/
	//找出upperDir表的物理地址
	upperDirPhysAddr = globalDirItem & 0x0007FFFFFFFFF000UL;//the right most bit is XD(eXecution Disable)
	//通过偏移得到目标表项，并取值赋给upperDir
	upperDirItem = *((unsigned long *)(__va(upperDirPhysAddr + upperDirIndex * 8)));

	DEBUG_PRINT("finish 处理2级页表\n");

	/*处理三级页表*/
	//找出middleDir表的物理地址
	midleDirPhysAddr = upperDirItem & 0x0007FFFFFFFFF000UL;
	//找出middleDir表的物理地址
	middleDirItem = *((unsigned long *)(__va(midleDirPhysAddr + middleDirIndex * 8))); 

	DEBUG_PRINT("finish 处理3级页表\n");

	/*处理四级页表*/
	//找出pageTable表的物理地址
	pageTablePhysAddr = middleDirItem & 0x0007FFFFFFFFF000UL;
	//找出pageTable表的物理地址
	pageTableItem = (unsigned long)(__va(pageTablePhysAddr + pageTableIndex * 8));

	DEBUG_PRINT("finish 处理4级页表\n");

	/*处理页内偏移*/
	//找出page表的基物理地址
	pagePhysAddr = (pageTableItem & 0x0007FFFFFFFFF000UL);
	//读出页的基地址+页内偏移=物理地址
	physAddr = pagePhysAddr + pageOffset;
	DEBUG_PRINT("linear: %lx physAddr: %lx\n", linearAddr, physAddr);
	return physAddr;
}

```

