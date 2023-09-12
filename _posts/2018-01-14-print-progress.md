---
layout: post
title: 打印进度条
keywords:
category: 平台
---

在执行耗时较长的任务时，有必要显示其工作进度，以便清楚地知道是否卡住或出错了。以下是一种简易实现，效果与Linux字符界面的进度条类似，但进度只有符号，没有数字。

```
void showpos(int ppos, int cpos) {
    for (int i = 0; i + ppos < cpos; i++)
        putchar('#');
    if (cpos == 100) puts(" ok");
}
int progress(int ppos, int done, int total) {
    int pos = 100;
    if (done < total)
        pos = done * 100 / total;
    if (ppos < 100)
        showpos(ppos, pos);
    return pos;
}
```

测试代码：

```
int randn(int lo, int hi) {
    return lo + rand() % (hi - lo + 1);
}
int main() {
    setbuf(stdout, 0);
    for (int kase = 1; kase <= 5; kase++) {
        int done = 0, pos = 0;
        int total = randn(5555, 666666);
        printf("case %d: total amount=%d\n", kase, total);
        while (done < total) {
            int step = randn(1, 10);
            done += step;
            pos = progress(pos, done, total);
            usleep(10);
        }
    }
    return 0;
}
```

某次运行结果：

```
case 1: total amount=120290
#################################################################################################### ok
case 2: total amount=471798
#################################################################################################### ok
case 3: total amount=643116
#################################################################################################### ok
case 4: total amount=432571
#################################################################################################### ok
case 5: total amount=546023
#################################################################################################### ok
```
