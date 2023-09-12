---
layout: post
title: 双指针
category: 算法
keywords:
---

尺取法通常设置前后两个指针，不断地按照某种策略移动前后指针，考察前后指针所限定的区间。

#### 例1: [CF279B](http://codeforces.com/contest/279/problem/B)求最长区间使元素之和不超过t。

```
int n, t, a[100005], i, j, sum, ans;
int main() {
    scanf("%d%d", &n, &t);
    for (i = 0; i < n; i++)
        scanf("%d", &a[i]);
    for (i = j = 0; i < n; i++) {
        sum += a[i];
        while (sum > t) {
            sum -= a[j];
            j++;
        }
        ans = max(ans, i - j + 1);
    }
    printf("%d\n", ans);
    return 0;
}
```

#### 例2：[POJ3320](http://poj.org/problem?id=3320)求最短区间包含所有出现过的数。

```
int i, j, n, a[1000005], ans;
set<int> st;
map<int,int> mp;
int main() {
    scanf("%d", &n);
    for (i = 0; i < n; i++) {
        scanf("%d", &a[i]);
        st.insert(a[i]);
    }
    ans = n;
    for (i = j = 0; i < n; i++) {
        mp[a[i]] += 1;
        while (mp.size() == st.size()) {
            ans = min(ans, i - j + 1);
            mp[a[j]] -= 1;
            if (mp[a[j]] == 0) mp.erase(a[j]);
            j++;
        }
    }
    printf("%d\n", ans);
    return 0;
}
```

#### 例3：[CF660C](http://codeforces.com/contest/660/problem/C)在01串中选出最长的子串，使它所包含0的个数不超过k。

```
int n, k, i, j, a[300005], len, cnt, u, v = -1;
int main() {
    scanf("%d%d", &n, &k);
    for (i = 0; i < n; i++)
        scanf("%d", &a[i]);
    for (i = j = 0; i < n; i++) {
        if (a[i] == 0) {
            cnt++;
            while (cnt > k) {
                if (a[j] == 0) cnt--;
                j++;
            }
        }
        if (len < i-j+1) {
            u = j; v = i; len = i-j+1;
        }
    }
    for (i = u; i <= v; i++)
        a[i] = 1;
    printf("%d\n", len);
    for (i = 0; i < n; i++)
        printf("%d%c", a[i], i+1 == n ? '\n' : ' ');
    return 0;
}
```

