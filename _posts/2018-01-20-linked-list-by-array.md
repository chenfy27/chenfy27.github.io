---
layout: post
title: 链表的数组表示
category: 算法
tag:
---

除了指针表示法外，链表还可以用数组来表示，优点是编码简单，调试方便，缺点是要事先定义最大节点数。

与指针表示法类似，数组表示法的节点也由两部分构成：数据本身和下一个节点的位置。

```cpp
TYPE a[n];
int nxt[n];
```

为操作方便，一般以1为下标起始点，而把0作为链表的表头。nxt[i]表示第i个节点的下一个节点的下标，若为0则表示当前已经是最后一个节点。初始时链表是空的，因此设置`nxt[0] = 0`即可。

下面以[题目Broken Keyboard](https://vjudge.net/problem/UVA-11988)为例，展示其用法，需要重点关注以下几方面：

- 链表初始化
- 添加节点
- 遍历链表

```cpp
#include <bits/stdc++.h>
using namespace std;
char s[100005];
int nxt[100005];
int main() {
    while (~scanf("%s", s+1)) {
        int cur = 0, tail = 0;
        nxt[0] = 0;
        for (auto i = 1; s[i]; i++) {
            if (s[i] == '[')
                cur = 0;
            else if (s[i] == ']')
                cur = tail;
            else {
                nxt[i] = nxt[cur];
                nxt[cur] = i;
                if (cur == tail)
                    tail = i;
                cur = i;
            }
        }
        for (auto i = nxt[0]; i; i = nxt[i])
            putchar(s[i]);
        puts("");
    }
    return 0;
}
```

对应的链式写法参考如下：

```cpp
#include <bits/stdc++.h>
using namespace std;
struct Node {
    char ch;
    struct Node *nxt;
    Node(char c) : ch(c), nxt(nullptr) {}
};
char s[100005];
int main() {
    while (~scanf("%s", s)) {
        Node head(0);
        Node *cur = &head, *tail = &head;
        for (int i = 0; s[i]; i++) {
            if (s[i] == '[') {
                cur = &head;
            } else if (s[i] == ']') {
                cur = tail;
            } else {
                Node *p = new Node(s[i]);
                p->nxt = cur->nxt;
                cur->nxt = p;
                if (tail == cur)
                    tail = p;
                cur = p;
            }
        }
        for (auto p = head.nxt; p; p = p->nxt)
            putchar(p->ch);
        puts("");
    }
    return 0;
}
```
