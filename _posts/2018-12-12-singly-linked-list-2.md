---
layout: post
title: '再谈单向链表操作'
subtitle: '单向链表的访问与修改'
date: 2018-12-12
categories: 计算机
cover: '../../../assets/img/singly-linked-list-2-cover.jpg'
tags: 计算机 C/C++
---

## 链表的访问

既然理解了链表的构成，对链表的访问也就不成问题了。不过要特别注意的是，对于数组，我们可以利用下标直接访问任何一个元素（这被称为**“随机访问”**），而对于单向链表，只能从头结点开始，依次进行**“顺序访问”**。

假设我们已经按照前文创建了一个链表，下面我们设计一个函数，查找是否存在某数据：

```cpp
//参数说明：p为头指针，n为所查找的数据
void search_node(l* p, int n) {
    //若非尾结点，则继续循环
    while(p->next != NULL) {
        //将指针指向下一个结点
        p = p->next;
        if(p->num == n) {
            printf("找到该数据！");
            return;
        }
    }
    printf("未找到该数据！");
    return;
}
```

尤其要注意第3行，由于我们的链表有不存储数据的头结点，要先将指针指向下一个结点，再访问数据。读者不妨想一想，如果使用无头结点的链表，指针后移的操作应该在何处进行。

## 结点的插入

假如我们需要将一个新结点（记为M）插入结点N后面，只需先将M的指针域指向N后面的结点（也就是N指针域所指的结点），再将N结点的指针域指向M即可。假设有一个指针p指向N结点，又有一个指向新结点M的指针p_new，那么只需进行如下操作：

```cpp
p_new->next = p->next;
p->next = p_new;
```

下面我们设计一个函数，查找某数据并将新数据插入其后：

```cpp
//参数说明：p为头指针，n为所查找的数据
void insert_node(l* p, int n) {
    l* p_new;
    while(p->next != NULL) {
        p = p->next;
        if(p->num == n) {
            //找到数据后，为新结点分配空间并输入要插入的数据
            printf("请输入要插入的数据：");
            p_new = (l*)malloc(sizeof(l));
            scanf("%d", &p_new->num);
            //插入新结点
            p_new->next = p->next;
            p->next = p_new;
        printf("插入成功！");
        return;
        }
    }
    printf("未找到该数据！");
    return;
}
```

## 结点的删除

相对插入而言，结点的删除就是小菜一碟了。不过细细想来，似乎有一些问题——我们在找到要删除的数据时，指针p已经指向了要删除的结点，但是要进行的操作是：改变它的前一个结点（称为**“前驱结点”**）指针域的指向，而我们的链表是单向的，无法将指针向前移动，这该怎么办呢？

因此，我们需要使用一个指向前驱结点的指针，辅助进行删除操作。

下面我们设计一个函数，查找某数据并将其结点删除：

```cpp
//参数说明：p为头指针，n为所查找的数据
void delete_node(l* p, int n) {
    l* predecessor;
    while(p->next != NULL) {
        predecessor = p;
        p = p->next;
        if(p->num == n) {
            if(p->next == NULL) {
                predecessor->next = NULL;
            } else {
                predecessor->next = p->next;
            }
            //释放要删除的结点的内存
            free(p);
            printf("删除成功！");
            return;
        }
    }
    printf("未找到该数据！");
    return;
}
```

特别注意，最后一定不要忘记使用free()函数释放所删除结点的内存！否则会导致内存泄漏。

## 总结

读完上面的例子，对链表的基本操作便理解完成了，现在，不妨回过头来，看看能否理解全部代码：

```cpp
#include<cstdio>
#include<cstdlib>

typedef struct linked_list {
    int num;
    struct linked_list* next;
} l;

l* create(int n) {
    l * head, * new_node, * tail;
    int i = 0;
    head = (l*)malloc(sizeof(l));
    tail = head;
    for(i = 0; i < n; ++i) {
        new_node = (l*)malloc(sizeof(l));
        scanf("%d", &new_node->num);
        tail->next = new_node;
        tail = new_node;
    }
    tail->next = NULL;
    return head;
}

void search_node(l* p, int n) {
    while(p->next != NULL) {
        p = p->next;
        if(p->num == n) {
            printf("找到该数据！");
            return;
        }
    }
    printf("未找到该数据！");
    return;
}

void insert_node(l* p, int n) {
    l* p_new;
    while(p->next != NULL) {
        p = p->next;
        if(p->num == n) {
            printf("请输入要插入的数据：");
            p_new = (l*)malloc(sizeof(l));
            scanf("%d", &p_new->num);
            if(p->next == NULL) {
                p_new->next = NULL;
                p->next = p_new;
            } else {
                p_new->next = p->next;
                p->next = p_new;
            }
            printf("插入成功！");
            return;
        }
    }
    printf("未找到该数据！");
    return;
}

void delete_node(l* p, int n) {
    l* predecessor;
    while(p->next != NULL) {
        predecessor = p;
        p = p->next;
        if(p->num == n) {
            if(p->next == NULL) {
                predecessor->next = NULL;
            } else {
                predecessor->next = p->next;
            }
            free(p);
            printf("删除成功！");
            return;
        }
    }
    printf("未找到该数据！");
    return;
}

int main() {
    l* head_pointer;
    int n;
    printf("请输入数据个数：");
    scanf("%d", &n);
    printf("请依次输入数据：");
    head_pointer = create(n);
    /*
    下面可以调用函数进行访问和修改操作，我懒得写了
    */
    return 0;
}
```
