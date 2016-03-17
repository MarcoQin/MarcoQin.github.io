---

layout: post
title: 单链表C实现
category : C
tags : [C, data structure]

---

单链表的C实现。网上有一坨又一坨的文章来描述，所以这里就不赘述了（懒）。

以下代码均来自《数据结构与算法分析》，我只是一个勤劳的搬运工。

不过就目前来看，对于我最有效的学习方式居然是抄写代码->_->也是醉了。

```c
/**
* FileName: list.h
*/
#ifndef _List_H

struct Node;
typedef struct Node *PtrToNode;
typedef PtrToNode List;
typedef PtrToNode Position;

List MakeEmpty(List L);
int IsEmpty(List L);
int IsLast(Position P, List L);
Position Find(ElementType X, List L);
void Delete(ElementType X, List L);
Position FindPrevious(ElementType X, List L);
void Insert(ElementType X, List L, Position P);
void DeleteList(List L);
Position Header(List L);
Position First(List L);
Position Advance(Position P);
ElementType Retrive(Position P);

#endif

struct Node{
    ElementType Element;
    Position Next;
};
```


```c
/**
* FileName: list.c
*/
#include <stdio.h>
#include <string.h>

#include "list.h"

/* Return true if is empty */
int IsEmpty(List L)
{
    return L->Next == NULL;
}

int IsLast(Position P, List L)
{
    return P->Next == NULL;
}

Position Find(ElementType X, List L)
{
    Position P;

    P = L->Next;
    while(P != NULL && P->Element != x)
        P = P->Next;

    return P;
}

Position FindPrevious(ElementType X, List L)
{
    Position P;

    P = L;
    while(P->Next != NULL && P->Next->Element != X)
        P = P->Next;

    return P;
}

void Delete(ElementType X, List L)
{
    Position P, TmpCell;

    P = FindPrevious(X, L);

    if(!IsLast(P, L)) {
        TmpCell = P->Next;
        P->Next = TmpCell->Next;
        free(TmpCell);
    }
}

void Insert(ElementType X, List L, Position P)
{
    Position TmpCell;

    TmpCell = malloc(sizeof(struct Node));
    if(TmpCell == NULL)
        FatalError("Out of space.");

    TmpCell->Element = X;
    TmpCell->Next = P->Next;
    P->Next = TmpCell;
}

void DeleteList(List L)
{
    Position P, Tmp;

    P = L->Next;
    L->Next = NULL;
    while(P != NULL)
    {
        Tmp = P->Next;
        free(P);
        P = Tmp;
    }
}
```
