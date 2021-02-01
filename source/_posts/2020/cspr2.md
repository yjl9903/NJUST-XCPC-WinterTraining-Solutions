---
mathjax: true
title: CCF CSP 选拔模拟赛 Round 2
date: 2020-02-5 21:10:03
tags:
---

恭喜陶俊天，陆宗泽，龚敏分别冠亚季军，龚敏获得集训队外第一。

## A 数糖果

申请 $10000+$ 大小的结构数组 A，有颜色编号、水果糖和巧克力三属性，开始用数组的下标充当颜色标号计数，即当输入为 `s 0`，则 `A[s].f++`，`s 1` 则 `A[s].q++`。

计数完毕后，根据颜色编号属性 `std::sort` 排序即可。

## B 吃糖果

很容易想到根据每种颜色糖果总数量从多到少排序，若总数量相同，则按水果糖数量从
多到少排序计数。但是当出现三组数据 `5 5、4 6、3 6` 时，三组总数量依次为 `10、10、9`，先吃完第一组 $5$ 颗水果糖和 $5$ 颗巧克力后，注意此时虽然第二组的总数量也为 $10$，但是可以选择吃第二组的 $4$ 颗水果糖和 $5$ 颗巧克力，这样既避免与第一组总数量相同的冲突，又最大化糖果总数量且吃了尽可能少的巧克力。接着再吃第三组的 $3$ 颗水果糖与 $5$ 颗巧克力。

所以，可以建立一个优先队列来存储糖果数量信息，还是按照之前的规则排序，当数量出现冲突的时候，将总数量减一，再压入队列里，重新参与排序。数量不冲突的时候，即可计数。

## C 炸金花

考虑采用结构体存储手牌信息，用一个 `prio` 优先级 $1$ 到 $6$，分别表示单牌->豹子，再用一个数组 `poker[3]`，存储手牌点数信息，最后用一个 `seq` 表示玩家序号。

当记录完所有玩家手牌信息后，先按照prio 优先级排列，相等则进行点数排列，再相等则按照 `seq` 序号排列。最后输出第一个结构体里的序号即可。

## D Miaojie 学指令

按照题意模拟。

## E Miaojie 学图论

$dp(i,a,b,c,d)$ 表示前 $i$ 个点，有 $a$ 个红色点是偶数条美丽的路径的终点，有 $b$ 个黑色色点是偶数条美丽的路径的终点，有 $c$ 个红色点是奇数条美丽的路径的终点，有 $d$ 个黑色点是奇数条美丽的路径的终点，那么显然 $i=a+b+c+d$，所以可以缩减一维变成 $dp(i,a,b,c)$。

如果某个点是红色的，那么这个点的美丽路径一定是来自黑色的点，并且黑色节点为终点且
以这个节点为终点美丽路径的条数是偶数的话，那么这个节点对于红色节点的奇偶性是没有
影响的。综上，对于一个红色节点，只有上一个节点是黑色且以它为终点的路径是奇数的点
才会有影响，连边方案数为 $\sum C(d, 1) + C(d, 3) + \dots = 2^{d-1}$。

对于黑色节点同理。转移方程为：

$$
dp(i,a,b,c)=2^{a+b+c-1} \times (2^{d-1} \times dp(i-1,a-1,b,c) + 2^{d-1} \times dp(i,a,b,c-1)) +
2^{a+b+d-1}\times (2^{c-1} \times dp(i-1,a,b,c) + 2^{c-1} \times dp(i-1,a,b-1,c))
$$

每一次求完 $dp(i,a,b,c)$ 之后判断是否合法并加入答案即可。

<!--more-->

## 代码

### A

```c++
# include <iostream>
# include <fstream>
# include <cstdio>
# include <cstring>
# include <algorithm>
using namespace std;

const int maxn = 10005;
int f[maxn];

struct Suger
{
  int seq, f, q;
}s[maxn];

bool cmp(Suger x, Suger y)
{
  return x.seq < y.seq;
}

int main()
{
  int T;
//  freopen("1.in", "r", stdin);
//  freopen("1.out", "w", stdout);
  cin >>T;
  while(T--)
  {
    int n, seq, p, cnt;
    cin >>n;
    cnt = 0;
    memset(f, 0, sizeof(f));
    for(int i = 1; i <= n; i++)
    {
      s[i].seq = 0;
      s[i].f = 0;
      s[i].q = 0;
    }
    for(int i = 1; i <= n; i++)
    {
      scanf("%d%d", &seq, &p);
      s[seq].seq = seq;
      if(!f[seq]) f[seq] = 1, cnt++;
      if(!p) s[seq].f++;
      else s[seq].q++;
    }
    sort(s+1, s+n+1, cmp);
    printf("%d\n", cnt);
    //outfile<<cnt<<endl;
    for(int i = 1; i <= n; i++)
    {
      if(s[i].seq)
      {
        printf("%d %d %d", s[i].seq, s[i].f, s[i].q);
        printf("\n");
      }
    }
  }
  return 0;
}
```

### B

```c++
# include <iostream>
# include <cstdio>
# include <queue>
# include <cstring>
# include <algorithm>
using namespace std;

struct Suger
{
  int sum, f;
  bool operator < (const Suger &s) const
  {
    return sum!=s.sum ? sum<s.sum : f<s.f;
  }
};

int main()
{
  //freopen("2.in", "r", stdin);
  int T;
  cin >>T;
  
  while(T--)
  {
    int n;
    cin >>n;
  
    int ans1=0, ans2=0, maxn=10001;
    priority_queue<Suger> Q;
  
    for(int i = 1; i <= n; i++)
    {
      Suger S;
      int f, q;
      scanf("%d%d",&f,&q);
      S.sum = f+q;
      S.f = f;
      Q.push(S);
    }
    while(!Q.empty())
    {
      Suger S = Q.top();
      int x = min(maxn, S.sum);
      int y = min(x, S.f);
      Q.pop();
      if(x < S.sum)
      {
        S.sum--;
        Q.push(S);
        continue;
      }
      if(x > 0)
      {
        ans1 += x;
        ans2 += y;
        maxn = x-1;
      }
      else
      break;
    }
    printf("%d\n",ans1-ans2);  
  }
  return 0;
}
```

### C

```c++
# include <iostream>
# include <cstring>
# include <algorithm>
using namespace std;

string p[7];
struct Poker
{
  int seq;
  int prio;
  int po[4];
}poker[15];

bool cmp(Poker p1, Poker p2)
{
  if(p1.prio != p2.prio) return p1.prio > p2.prio;
  else
  {
    if(p1.prio == 6)
    {
      if(p1.po[1] != p2.po[1]) return p1.po[1] > p2.po[1];
      else return p1.seq < p2.seq;
    }
    else if(p1.prio == 5)
    {
      if(p1.po[1] != p2.po[1]) return p1.po[1] > p2.po[1];
      else return p1.seq < p2.seq;
    }
    else if(p1.prio == 4)
    {
      if(p1.po[3] != p2.po[3]) return p1.po[3] > p2.po[3];
      else if(p1.po[2] != p2.po[2]) return p1.po[2] > p2.po[2];
      else if(p1.po[1] != p2.po[1]) return p1.po[1] > p2.po[1];
      else return p1.seq < p2.seq;
    }
    else if(p1.prio == 3)
    {
      if(p1.po[1] != p2.po[1]) return p1.po[1] > p2.po[1];
      else return p1.seq < p2.seq;
    }
    else if(p1.prio == 2)
    {
      if(p1.po[1] != p2.po[1]) return p1.po[1] > p2.po[1];
      else if(p1.po[2] != p2.po[2]) return p1.po[2] > p2.po[2];
      else return p1.seq < p2.seq;
    }
    else
    {
      if(p1.po[3] != p2.po[3]) return p1.po[3] > p2.po[3];
      else if(p1.po[2] != p2.po[2]) return p1.po[2] > p2.po[2];
      else if(p1.po[1] != p2.po[1]) return p1.po[1] > p2.po[1];
      else return p1.seq < p2.seq;
    }
  }
}

int ret(string s)
{
  if(s == "2") return 2;
  else if(s == "3") return 3;
  else if(s == "4") return 4;
  else if(s == "5") return 5;
  else if(s == "6") return 6;
  else if(s == "7") return 7;
  else if(s == "8") return 8;
  else if(s == "9") return 9;
  else if(s == "10") return 10;
  else if(s == "J") return 11;
  else if(s == "Q") return 12;
  else if(s == "K") return 13;
  else return 14;
}

void solve(int x)
{
  int pp[4];
  pp[1] = ret(p[2]);
  pp[2] = ret(p[4]);
  pp[3] = ret(p[6]);
  sort(pp+1, pp+4);
  poker[x].seq = x;
  if(pp[1] == pp[2] && pp[2] == pp[3])
  {
    poker[x].prio = 6;
    poker[x].po[1] = pp[1];
  }
  else if(p[1] == p[3] && p[3] == p[5])
  {
    if((pp[1] == pp[2]-1&& pp[2] == pp[3]-1))
    {
      poker[x].prio = 5;
      poker[x].po[1] = pp[3];
    }
    else if(pp[1] == 2 && pp[2] == 3&&pp[3] == 14)
    {
      poker[x].prio = 5;
      poker[x].po[1] = 3;
    }
    else
    {
      poker[x].prio = 4;
      poker[x].po[1] = pp[1];
      poker[x].po[2] = pp[2];
      poker[x].po[3] = pp[3];
    }
  }
  else if(pp[1] == pp[2]-1&& pp[2] == pp[3]-1)
  {
    poker[x].prio = 3;
    poker[x].po[1] = pp[3];
  }
  else if(pp[1] == 2 && pp[2] == 3&&pp[3] == 14)
  {
    poker[x].prio = 3;
    poker[x].po[1] = 3;
  }
  else if(pp[1] == pp[2] || pp[1] == pp[3] ||pp[2] == pp[3])
  {
    poker[x].prio = 2;
    if(pp[1] == pp[2]) poker[x].po[1] = pp[1], poker[x].po[2] = pp[3];
    else if(pp[1] == pp[3]) poker[x].po[1] = pp[1], poker[x].po[2] = pp[2];
    else if(pp[2] == pp[3]) poker[x].po[1] = pp[2], poker[x].po[2] = pp[1];
  }
  else
  {
    poker[x].prio = 1;
    poker[x].po[1] = pp[1];
    poker[x].po[2] = pp[2];
    poker[x].po[3] = pp[3];
  }
}

int main()
{
  int T;
  cin >>T;
  
  while(T--)
  {
    int n;
    cin >>n;
    for(int i = 1; i <= n; i++)
    {
      cin >>p[1] >>p[2] >>p[3] >>p[4] >>p[5] >>p[6];
      solve(i);
    }
    sort(poker+1, poker+n+1, cmp);
    cout<<poker[1].seq<<endl ;
  return 0;  
}
```

### D

```c++
#include<bits/stdc++.h>
#define LL longlong
using namespace std;
struct node{
    string s;
    string si;
    string i;
}a[40];
int pd(string s){
    if(s=="ADD") return 1;
    if(s=="SUB") return 2;
    if(s=="DIV") return 3;
    if(s=="MUL") return 4;
    if(s=="MOVE") return 5;
    if(s=="SET") return 6;
    return 0;
}
void init(){
  a[0].s="000";a[1].s="ADD";a[2].s="SUB";a[3].s="DIV";a[4].s="MUL";a[5].s="MOVE";a[6].s="SET";
    a[0].si="000000";a[1].si="111111";a[2].si="111110";a[3].si="111101";a[4].si="111100";a[5].si="111011";a[6].si="111010";
    a[0].i="00000";a[1].i="00001"; a[2].i="00010"; a[3].i="00011"; a[4].i="00100"; a[5].i="00101"; a[6].i="00110"; a[7].i="00111";
  a[8].i="01000";a[9].i="01001";a[10].i="01010";a[11].i="01011";a[12].i="01100";a[13].i="01101";a[14].i="01110";a[15].i="01111";
  a[16].i="10000";a[17].i="10001";a[18].i="10010";a[19].i="10011";a[20].i="10100";a[21].i="10101";a[22].i="10110";a[23].i="10111";
  a[24].i="11000";a[25].i="11001";a[26].i="11010";a[27].i="11011";a[28].i="11100";a[29].i="11101";a[30].i="11110";a[31].i="11111";
}
int main(){
  init();
  int T;
  cin>>T;
  while(T--){
    int op;
    string s1,s2,s3;
    cin>>op;
    if(op==1){
      cin>>s1>>s2;
            if(s1=="SET"){
                string c1;
                for(int i=0;i<s2.size();i++){
                    if(s2[i]<='9'&&s2[i]>='0') c1=c1+s2[i];
                }
                int k=c1.size();
                int num=1,x=0;
                while(k--){
                    x+=(c1[k]-'0')*num;
                    num*=10;
                }
                cout<<"111010";
                cout<<a[x].i<<"00000"<<endl;
            }
            else{
                int x=pd(s1),y,z,num;
                string c2,c3;
                int flag=0;
                for(int i=0;i<s2.size();i++){
                    if(flag==0&&s2[i]>='0'&&s2[i]<='9') c2=c2+s2[i];
                    if(s2[i]==',') flag=1;
                    if(flag==1&&s2[i]>='0'&&s2[i]<='9') c3=c3+s2[i];
                }
                int k=c2.size();
                num=1,y=0;
                while(k--){
                    y+=(c2[k]-'0')*num;
                    num*=10;
                }
                k=c3.size();
                num=1,z=0;
                while(k--){
                    z+=(c3[k]-'0')*num;
                    num*=10;
                }
                cout<<a[x].si<<a[y].i<<a[z].i<<endl;
            }
    }
    else{
            cin>>s3;
            int x=(s3[0]-'0')*32+(s3[1]-'0')*16+(s3[2]-'0')*8+(s3[3]-'0')*4+(s3[4]-'0')*2+(s3[5]-'0')*1;
            int y=(s3[6]-'0')*16+(s3[7]-'0')*8+(s3[8]-'0')*4+(s3[9]-'0')*2+(s3[10]-'0')*1;
            int z=(s3[11]-'0')*16+(s3[12]-'0')*8+(s3[13]-'0')*4+(s3[14]-'0')*2+(s3[15]-'0')*1;
            x=64-x;
            if(x<1||x>6||y<1||y>31||z<0||z>31) printf("Error\n");
            else{
                if((z==0&&x!=6)||(x==6&&z!=0))
                    printf("Error\n");
                else{
                    if(x==6&&z==0) cout<<a[x].s<<" "<<"R"<<y<<endl;
                    else cout<<a[x].s<<" "<<"R"<<y<<","<<"R"<<z<<endl;
                }
            }
        }
  }
    return 0;
}
```

### E

```c++
#include<bits/stdc++.h>
#define LL long long
using namespace std;
const int mod=1e9+7;
const int N=60;
int n,p,s[N];
LL f[N][N][N][N],pw[N],ans;
void add(LL &a,LL b){
  a+=b;
  while(a>=mod) a-=mod;
}
int main(){
  pw[0]=1;
  for(int i=1;i<=50;i++){
    pw[i]=pw[i-1]*2%mod;
  }
  int T;
  scanf("%d",&T);
  while(T--){
    scanf("%d %d",&n,&p);
    for(int i=1;i<=n;i++){
      scanf("%d",&s[i]);
    }
    memset(f,0,sizeof f);
    ans=0;
    f[0][0][0][0]=1;
    for(int i=1;i<=n;i++){
      for(int a=0;a<=i;a++){
          for(int b=0;b+a<=i;b++){
            for(int c=0;c+a+b<=i;c++){
                int d=i-a-b-c;
                if(c+a&&s[i]){
                    LL res=0;
                    if(c){
                      if(d) add(res,f[i-1][a][b][c-1]*pw[d-1]%mod);
                      else add(res,f[i-1][a][b][c-1]);
                    }
                    if(a&&d) add(res,f[i-1][a-1][b][c]*pw[d-1]%mod);
                    add(f[i][a][b][c],res*pw[c+a+b-1]%mod);
                  }
                  if(d+b&&s[i]<1){
                    LL res=0;
                    if(d){
                        if(c)add(res,f[i-1][a][b][c]*pw[c-1]%mod);
                        else add(res,f[i-1][a][b][c]);
                    }
                    if(b&&c)add(res,f[i-1][a][b-1][c]*pw[c-1]%mod);
                    add(f[i][a][b][c],res*pw[b+d+a-1]%mod);
                  }
                  if(i==n&&(c+d)%2==p)add(ans,f[i][a][b][c]);
             }
        }
      }
    }
      printf("%d\n",ans);
  }
  return 0;
}
```
