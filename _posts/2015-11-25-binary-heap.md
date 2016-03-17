---

layout: post
title: 二叉堆(binary heap)
category : C
tags : [C, data structure]

---

实现优先队列的其中一种普遍方式：二叉堆。而且堆（heap）一般也是值这种
数据结构。

一个堆的数据结构将由一个数组、一个代表最大值的整数及当前堆的大小组成。

堆的声明：

```c
#ifndef _BinHeap_H

struct HeapStruct;
typedef struct HeapStruct * PriorityQueue;

PriorityQueue Initialize(int MaxElements);
void Destroy(PriorityQueue H);
void MakeEmpty(PriorityQueue H);
void Insert(ElementType X, PriorityQueue H);
ElementType DeleteMin(PriorityQueue H);
ElementType FindMin(PriorityQueue H);
int IsEmpty(PriorityQueue H);
int IsFull(PriorityQueue H);

#endif

struct HeapStruct{
    int Capacity;
    int Size;
    ElementType *Elements;
};
```

堆初始化：

```c
PriorityQueue Initialize(int MaxElements)
{
    PriorityQueue H;

    if(MaxElements < MinPQSize)
        Error("PriorityQueue size is too small");

    H = malloc(sizeof(struct HeapStruct));
    if(H == NULL)
        FatalError("Out of space");

    /* Allocate the array plus one extra for sentinel */
    H->Elements = malloc((MaxElements + 1) * sizeof(ElementType));
    if(H->Elements == NULL)
        FatalError("Out of space");

    H->Capacity = MaxElements;
    H->Size = 0;
    H->Elements[0] = MinData;

    return H;
}
```

插入操作。为了保证插入不破坏堆序（heap order），我们需要用上滤(percolate up)
策略来找到插入点。

    {% capture images %} /images/heap-percolate-up.png {% endcapture %} {% include gallery images=images caption="上滤(percolate up)" cols=1 %}

```c
/* H->Element[0] is a sentinel */

void Insert(ElementType X, PriorityQueue H)
{
    int i;

    if(IsFull(H)) {
        Error("PriorityQueue is full");
        return;
    }

    for(i = ++H->Size; H->Elements[i/2] > X; i/=2)
        H->Elements[i] = H->Elements[i/2];
    H->Elements[i] = X;
}
```

DeleteMin类似插入函数的处理。找出最小元素（root节点）删除。但是会
产生一个空穴。为了保证完全树，需要将空穴填补，并且将最后一个元素
挪到一个合适的位置。

如下如，要删除13，同时要把书重新变完全，需要把31挪位。一般用下滤
（percolate down)策略：

    {% capture images %} /images/heap-percolate-down.png {% endcapture %} {% include gallery images=images caption="下滤(percolate down)" cols=1 %}

```c
ElementType DeleteMin(PriorityQueue H)
{
    int i, Child;
    ElementType MinElement, LastElement;

    if(IsEmpty(H)) {
        Error("PriorityQueue is empty");
        return H->Elements[0];
    }
    MinElement = H->Elements[1];
    LastElement = H->Elements[H->Size--];

    for(i = 1; i * 2 <= H->Size; i = Child) {
        /* Find smaller child */
        Child = i * 2;
        if(Cild != H->Size && H->Elements[Child + 1] > H->Elements[Child])
            Child++;

        /* Percolate one level */
        if(LastElement > H->Elements[Child])
            H->Elements[i] = H->Elements[Child];
        else
            break;
    }
    H->Elements[i] = LastElement;
    return MinElement;
}
```
