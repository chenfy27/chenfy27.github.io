---
layout: post
title: 正则匹配与替换
category: 平台
keywords:
---

正则表达式是字符串匹配的强力工具，被广泛应用到各种文字处理程序中。

### Linux系统工具

- grep可用于检查字符串是否满足特定的模式，-i选项可指定匹配时不区分大小写；
- sed可将满足模式的字符串替换成指定的串，I选项可指定匹配时不区分大小写；
- awk也可实现字符串替换，不区分大小写需要在BEGIN中设置IGNORECASE变量；

### 编程语言

python提供了正则表达式库re，函数sub可完成字符串替换工作，其原型为：`re.sub(pattern, newstr, string)`，可在后面加参数指定替换次数与是否区分大小写，如count=1, flags=re.I表示最多替换1次，不区分大小写。默认全部替换，区分大小写。

```python
import sys
import re
for line in sys.stdin:
    ret = re.sub("o(go)+", "***", line)
    print ret,
```

C++11标准提供了正则表达式接口，以下是同样功能的C++实现。

```cpp
#include <iostream>
#include <string>
#include <regex>
using namespace std;
int main() {
    string line;
    regex pattern("o(go)+");
    while (getline(cin, line))
        cout << regex_replace(line, pattern, "***") << endl;
    return 0;
}
```

Linux环境下C语言也有regex接口，但功能偏弱，不支持中文。

```c
#include <regex.h>
int RegMatch(const char *str, const char *pat) {
    regex_t reg;
    char err[128] = {0};
    int ret = 0;

    if (0 != (ret = regcomp(&reg, pat, REG_EXTENDED | REG_NOSUB))) {
        regerror(ret, &reg, err, sizeof(err));
        printf("regcomp failed:[%s]\n", err);
        return 0;
    }
    ret = regexec(&reg, str, 0, NULL, 0);
    regfree(&reg);
    return ret == 0;
}
```

