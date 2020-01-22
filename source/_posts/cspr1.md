---
title: CCF CSP 选拔模拟赛 Round 1
date: 2020-01-22 19:19:25
categories:
- 2020 冬季
---

# A 莫纳卡的护身符

## 题面

给定一个序列 $a_1,a_2, \dots , a_n$，有 $q$ 次询问，每次询问从序列中选出一个大小最大的子集，满足子集元素和小于等于输入的 $s$。

其中 $1 \le n,q\le 2 \cdot 10^5,1 \le a_i \le 10^9$。

## 题解

容易观察发现肯定是尽量先选值比较小的。

### 算法一

使用萌萌哒冒泡排序，选择排序，插入排序，对数组排序后。

从小到大，依次选上每个数，直到总和大于 $s$，输出个数。

时间复杂度：$O(n^2)$。可以通过子任务 $1$。

### 算法二

在算法一的基础上，使用 `C++` 标准库中提供的 `std::sort` 函数进行排序。

时间复杂度：$O(n \log n)$。可以通过子任务 $2$。

### 算法三

在算法二的基础上，对于排序后的序列 $a_1,a_2,\dots,a_n$，令 $pre(i)=\sum_{j=1}^i a_j$，询问等价于找到最大的 $i$，使得 $pre(i)\le s$，使用二分查找即可。这里可以使用 `C++` 标准库中提供的函数 `std::upper_bound`。

时间复杂度：$O(n\log n+q\log n)$。可以通过子任务 $3$。

## 标程

```c++
#include <cstdio>
#include <algorithm>
const int maxn = 200000 + 5;

int n, q, a[maxn];
long long pre[maxn];

int main() {
  int T; scanf("%d", &T);
  while (T--) {
    scanf("%d%d", &n, &q);
    for (int i = 1; i <= n; i++) scanf("%d", a + i);
    std::sort(a + 1, a + 1 + n);
    for (int i = 1; i <= n; i++) pre[i] = a[i] + pre[i - 1];
    for (int T = 1, x; T <= q; T++) {
      scanf("%d", &x);
      printf("%d\n", std::upper_bound(pre + 1, pre + 1 + n, x) - pre - 1);
    }
  }
  return 0;
}
```

# B 普通上班族的观察喵

## 题面

实现解析给定文法的简单词法分析器和简单的给定观察者模式。

## 题解

细节参考完整题面。

想法来源：

+ [Vue.js](https://cn.vuejs.org/)
+ [观察者模式](https://zh.wikipedia.org/wiki/观察者模式)

### 子任务一

只需要解析四种代码 `observable 变量名 = 右值`，`变量名 = 右值`，`变量名 = 右值 + 右值`，`变量名 = 右值 - 右值`。需要注意等号左右的空格。

使用 `std::map<std::string,int>`，建立一个变量名到值的映射。

### 子任务二

使用 `std::map<std::string,std::vector<std::string>>`，存储每个变量的修改可能会触发哪些函数。

### 子任务三

在子任务二的基础上，把修改改成递归，并维护一个 `std::set<std::string>` 作为栈，防止死循环。

## 标程

```c++
#include <cassert>
#include <iostream>
#include <string>
#include <vector>
#include <map>
#include <set>
#include <functional>
using namespace std;

bool isLetter(char c) { return 'a' <= c && c <= 'z'; }
bool isNum(char c) { return '0' <= c && c <= '9'; }

vector<string> lexer(const string& s) {
  vector<string> v;
  for (int i = 0; i < (int)s.length(); i++) {
    if (s[i] == ' ') continue;
    if (isLetter(s[i])) {
      int j = i;
      while (j < (int)s.length() && isLetter(s[j])) j++;
      v.push_back(s.substr(i, j - i)); i = j - 1;
    } else if (isNum(s[i])) {
      int j = i;
      while (j < (int)s.length() && isNum(s[j])) j++;
      v.push_back(s.substr(i, j - i)); i = j - 1;
    } else if (s[i] == '=' || s[i] == '+' || s[i] == '-') {
      v.push_back(s.substr(i, 1));
    }
  }
  return v;
}

map<string,int> symbols;
int getVal(const string& s) {
  if (isNum(s[0])) {
    return atoi(s.c_str());
  } else {
    return symbols[s];
  }
}

struct Op {
  string target, a, b; char op;
  Op() {}
  Op(const vector<string>& tokens) {
    target = tokens[0];
    if (tokens.size() == 3) {
      op = '='; a = tokens[2];
    } else {
      op = tokens[3][0]; a = tokens[2]; b = tokens[4];
    }
  }
  int cal() const {
    if (op == '=') return getVal(a);
    else if (op == '+') return getVal(a) + getVal(b);
    else if (op == '-') return getVal(a) - getVal(b);
    assert(false);
  }
};

map<string,vector<Op> > func;
string inFunc; vector<Op> fcode;
map<string,vector<string> > deps;

string update(const Op& op) {
  string ans; set<string> stk;
  function<void(const Op&)> update = [&](const Op& op) {
    int val = op.cal();
    if (symbols[op.target] != val) {
      symbols[op.target] = val;
      ans += "update " + op.target + " " + to_string(val) + "\n";
      for (auto& dep: deps[op.target]) {
        if (stk.count(dep)) continue;
        ans += "enter " + dep + '\n';
        stk.insert(dep);
        for (auto& c: func[dep]) update(c);
        ans += "leave " + dep + '\n';
        stk.erase(dep);
      }
    }
  };
  return update(op), ans;
}

string recordDep() {
  string ans; set<string> from;
  ans += "enter " + inFunc + "\n";
  for (auto& c: fcode) {
    ans += update(c);
    if (!c.a.empty() && isLetter(c.a[0])) from.insert(c.a);
    if (!c.b.empty() && isLetter(c.b[0])) from.insert(c.b);
  }
  for (auto& s: from) deps[s].push_back(inFunc);
  ans += "leave " + inFunc + "\n";
  return ans;
}

string parser(const vector<string>& tokens) {
  string ans;
  if (tokens[0] == "observable") {
    int val = getVal(tokens[3]);
    ans += "observable " + tokens[1] + " " + to_string(val) + "\n";
    symbols[tokens[1]] = val;
  } else if (tokens[0] == "observe") {
    inFunc += tokens[1];
  } else if (tokens[0] == "end") {
    ans = recordDep(); func[inFunc] = vector<Op>(fcode);
    inFunc.clear(); fcode.clear();
  } else {
    if (inFunc.empty()) {
      ans = update(Op(tokens));
    } else {
      fcode.emplace_back(tokens);
    }
  }
  return ans;
}

int main() {
  ios::sync_with_stdio(false);
  int n; cin >> n; cin.ignore();
  for (int i = 1; i <= n; i++) {
    string code; getline(cin, code);
    cout << parser(lexer(code));
  }
  return 0;
}
```

# C 世界线变动率探测仪

## 题面

实现一个简单的可持久化并查集。

## 题解

你去抄一个洛谷的在线模板是无法通过的。在线的可持久化并查集，主要通过可持久化线段树实现一个可持久化数组，并通过按秩合并（或启发式合并），保证复杂度。时间复杂度：$O(n \log^2n)$，空间复杂度：$O(n \log n)$。

注意到，可持久的时间构成了一棵树形结构，在树上 dfs，并使用可撤销并查集即可通过本题，时间复杂度 $O(n \log n)$，空间复杂度：$O(n)$。

其他例题：[Extending Set of Points](https://codeforces.com/contest/1140/problem/F)

## 标程

```c++
#include <cstdio>
#include <vector>
const int maxn = 500000 + 5;

namespace DSU {
  int pre[maxn], siz[maxn];
  void init(int n) {
    for (int i = 0; i <= n; i++) {
      pre[i] = i; siz[i] = 1;
    }
  }
  int find(int x) {
    while (x != pre[x]) x = pre[x];
    return x;
  }
  std::vector<std::pair<int,int> > sta;
  bool join(int x, int y) {
    x = find(x); y = find(y);
    if (x == y) return 0;
    if (siz[x] > siz[y]) std::swap(x, y);
    pre[x] = y; siz[y] += siz[x];
    sta.push_back({x, y});
    return 1;
  }
  void undo() {
    auto tp = sta.back(); sta.pop_back();
    int x = tp.first, y = tp.second;
    pre[x] = x; siz[y] -= siz[x];
  }
}

int n, q, a[maxn], b[maxn]; bool ans[maxn];
std::vector<int> edge[maxn];

void dfs(int u) {
  if (u) ans[u] = DSU::join(a[u], b[u]);
  for (int v: edge[u]) dfs(v);
  if (ans[u]) DSU::undo();
}

int main() {
  int T; scanf("%d", &T);
  while (T--) {
    scanf("%d%d", &n, &q);
    for (int i = 1; i <= q; i++) edge[i].clear();
    DSU::init(n);
    for (int i = 1, k; i <= q; i++) {
      scanf("%d%d%d", &k, a + i, b + i);
      edge[k].push_back(i);
    }
    dfs(0);
    for (int i = 1; i <= q; i++) puts(ans[i] ? "No" : "Yes");
  }
  return 0;
}
```
