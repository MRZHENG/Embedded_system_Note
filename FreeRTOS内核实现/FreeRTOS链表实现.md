### FreeRTOS 链表实现

#### FreeRTOS 链表描述

​	FreeRTOS 链表 是 一种 **双向环形升序链表** , 由 普通节点 和 特殊的根节点 组成,其中链表根节点在 链表类实现

#### 实现链表节点

```C
//链表节点数据结构
struct xLIST_ITEM
{
	TickType_t xItemValue;  //辅助值，用于帮助节点做顺序排列
	struct xLIST_ITEM *  pxNext;//指向链表的下一个节点
	struct xLIST_ITEM *  pxPrevious; //指向链表的前一个节点
	void * pvOwner;	//指向拥有当前节点的内核对象，通常是TCB结构体，也是C语言一种泛型的体现是
	void * pvContainer;	//指向当前节点所在的链表
};
typedef struct xLIST_ITEM ListItem_t ;//节点数据类型的重定义

```

![image-20250511165017110](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250511165017110.png)

#### 链表节点初始化

```c
//节点初始化
void vListInitialiseItem(ListItem_t * const pxItem)
{	
	//初始化当前节点 不属于任何链表
	pxItem->pvContainer = NULL;
}

```



#### 链表精简节点定义

```c
//链表精简节点结构体定义
struct xMINI_LIST_ITEM
{
	TickType_t xItemValue;  //辅助值，用于帮助节点做升序排列
	struct xLIST_ITEM *  pxNext;//指向链表的下一个节点
	struct xLIST_ITEM *  pxPrevious;//指向链表的前一个节点
};
typedef struct xMINI_LIST_ITEM MiniListItem_t;//链表精简节点重定义
```

描述：这个精简节点是嵌套在 链表类内部的，用于作为根节点



#### 链表类定义

```c
struct xLIST
{
	UBaseType_t  uxNumberOfItems;//节点计数器 记录节点 根节点除外
	ListItem_t * pxIndex;//链表节点索引指针
	MiniListItem_t xListEnd ;//链表的根节点,既是头节点又是尾节点
};
typedef struct xLIST List_t; //链表根节点重定义
```

![image-20250511165645151](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250511165645151.png)