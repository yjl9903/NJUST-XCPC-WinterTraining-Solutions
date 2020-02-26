---
mathjax: true
title: CCF CSP 选拔模拟赛 Round 3
date: 2020-02-26 15:50:31
tags:
---

恭喜杨永健，陆宗泽，刘宇轩分别冠亚季军，？获得集训队外第一。

## A 快乐水

根据题意直接做就行，每次将剩余的钱除以3 得到可以买的快乐水瓶数，减去买水的钱
再加上用瓶子换回来的钱，直到剩下的钱不够买一瓶快乐水，即可求出答案。

## B A+B

首先用字符串进行一个模拟大数加法。

实现方法：将两个字符串进行反转，然后从低位往高位模拟，注意进位问题。

对数字进行打印则首先将每个数字对应的图形存起来，然后对应输出就可以了。

## C 让圣光净化一切

单调队列。如果直接找区间最大和并不是很好做，所以我们将数组转化为前缀和，那么只要求出最大的 $s_y  - s_x (y - x \le m)$。

然后我们用一个 `for` 循环来枚举右端点，每次枚举固定住右端点，然后枚举左端点，使得区间内骷髅数量不超过 $m$ 个 。

当右端点固定，即 $s_y$ 不变，枚举左端点，对于两个左端点 $j,k$，若有 $k > j$ 且 $s_k \ge s_j$，那么我们肯定不会选择 $k$ ，因为我们要求出 $s_y - s_x$ 最大， $s_y -  s_j \ge s_y - s_k$，且因为 $k$ 更加靠近 $y$ ，意味着我们可以选到更多 $y$ 后 面的骷髅，这就是单调性。

我们用双端队列 `std::deque` 来实现这个功能，每次入队一个 $i$ ，就删除前面所有不符合要求的骷髅（队列中骷髅序号 $k$ 当前序号 $i - m$，队列中骷髅序号 $k < m$ 且 $s_k \ge s_i$）。

## D 简单打牌

可以对每个玩家都记录着十种情况的其中的每种情况中的最大值，注意顺子的问题。然后分情况讨论就行了。

## E 魔法阵

AC 自动机。在建立字典树的时候记录每个字符串长度，即每个 now 的位置对应字符串的长度。然后直接匹配，在匹配到相应的字符串时记录与当前字符相匹配的所有字符串起始位置和终点位置。将所有取得的位置按末尾位置从小到大排序，贪心取出末尾位置更小的字符串，记录 共能取出的数量即可。

## F 分糖果

我们使用前缀和，则从第 $i$ 个同学开始的 $b$ 个同学的糖果的数量和为 $pre(i+b-1)-pre(i-1)$。

因为是首尾相连的，所以分为两种情况。

1. $i+b-1 \le n$，则满足第 $i$ 个小朋友的要求的限制条件为 $pre(i+b-1)-pre(i-1) \ge c$；

2. $i+b-1 > n$，则满足第 $i$ 个小朋友的要求的限制条件为 $pre(n)+pre(i+b-1-n)-pre(i-1) \ge c$，移项：$pre(i+b-1-n)-pre(i-1) \ge c-pre(n)$。

可以看出这个是一个约束差分问题了，因为第二种情况的右边的值是不确定的。

所以我们二分枚举 $pre(n)$ 的值来判断是否有解，即判断是否能够满足所有小朋友的条件，同时 $pre(n)$ 也表示当前的答案。

<!--more-->

## 代码

### A

```c++
#include <bits/stdc++.h>
using namespace std;

int t,n,ans;

int main(){
  scanf("%d",&t);
  while (t--){
    scanf("%d",&n);
    ans=0;
    while (n>2){
      int tmp=n/3;
      ans+=tmp;
      n-=tmp*2;
    }
    printf("%d\n",ans);
  }
  return 0;
}
```

### B

```c++
#include<bits/stdc++.h>
using namespace std;
//7行 4列
//存储十个数字的形状
char num[10][7][5]={
  {
  " -- ",
  "|  |",
  "|  |",
  "    ",
  "|  |",
  "|  |",
  " -- "
  },
  {
  "    ",
  "   |",
  "   |",
  "    ",
  "   |",
  "   |",
  "    "
  },
  {
  " -- ",
  "   |",
  "   |",
  " -- ",
  "|   ",
  "|   ",
  " -- "
  },
  {
  " -- ",
  "   |",
  "   |",
  " -- ",
  "   |",
  "   |",
  " -- "
  },
  {
  "    ",
  "|  |",
  "|  |",
  " -- ",
  "   |",
  "   |",
  "    "
  },
  {
  " -- ",
  "|   ",
  "|   ",
  " -- ",
  "   |",
  "   |",
  " -- "
  },
  {
  " -- ",
  "|   ",
  "|   ",
  " -- ",
  "|  |",
  "|  |",
  " -- "
  },
  {
  " -- ",
  "   |",
  "   |",
  "    ",
  "   |",
  "   |",
  "    "
  },
  {
  " -- ",
  "|  |",
  "|  |",
  " -- ",
  "|  |",
  "|  |",
  " -- "
  },
  {
  " -- ",
  "|  |",
  "|  |",
  " -- ",
  "   |",
  "   |",
  " -- "
  }
};
const int maxn=1e5+100;
char ans[7][maxn*4+10];
//通过给定x数字和当前结果所在的位置 把x的形状复制进去
void addnum(int x,int pos)
{
  for (int i=0;i<7;i++)
  {
    for (int j=0;j<4;j++)
    {
      ans[i][j+pos]=num[x][i][j];
    }
  }
}
char s1[maxn],s2[maxn],sum[maxn+1];
//数组模拟大数加法
int add(char *a,char *b,char *sum)
{
  int lena=strlen(a),lenb=strlen(b);
  reverse(a,a+lena);
  reverse(b,b+lenb);
  int c=0;
  int len=max(lena,lenb);
  int pos=0;
  while (true)
  {
    if (pos>=lena && pos>=lenb && c==0)
    break;
    int tmp=c;
    c=0;
    if (pos<lena)
    tmp+=a[pos]-'0';
    if (pos<lenb)
    tmp+=b[pos]-'0';
    if (tmp>9)
    {
      c=1;
      tmp-=10;
    }
    sum[pos++]=tmp+'0';
  }
  reverse(sum,sum+pos);
  return pos;
}
int main()
{
  while (scanf("%s%s",s1,s2)!=EOF)
  {
    memset(ans,0,sizeof(ans));
    int len_sum=add(s1,s2,sum);
    int pos=0;
    for (int i=0;i<len_sum;i++,pos+=4)
    addnum(sum[i]-'0',pos);
    for (int i=0;i<7;i++)
    cout<<ans[i]<<endl;
  }
}
```

### C

```c++
#include<bits/stdc++.h>
using namespace std;
#define ll long long
int n,m;
ll num[300010],s[300010];
deque<ll>Q;

ll FindMax(){
  ll ans=-1e18+7;
  Q.clear();
  Q.push_back(0);
  for(int i=1;i<=n;i++){
    while(!Q.empty()&&Q.front()<i-m)Q.pop_front();
    ans=max(ans,s[i]-s[Q.front()]);
    while(!Q.empty()&&s[Q.back()]>=s[i])Q.pop_back();
    Q.push_back(i);
  }
  return ans;
}

int main(){
  //freopen("11.in","r",stdin);
  //freopen("11.out","w",stdout);
  int t;
  scanf("%d",&t);
  while (t--){
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;i++){
      scanf("%lld",&num[i]);
      s[i]=s[i-1]+num[i];
    }
    printf("%lld\n",FindMax());
  }
  return 0;
}
```

### D

```c++
#include<bits/stdc++.h>
using namespace std;
#define debug(x) cout<<#x<<" "<<x<<endl;
int pai[3][20];
char s[20],t[20],t1[20];
int change(char a)
{
  if (a=='T') return 10;
  if (a=='J') return 11;
  if (a=='Q') return 12;
  if (a=='K') return 13;
  if (a=='A') return 14;
  if (a=='2') return 15;
  if (a=='X') return 16;//小王
  if (a=='Y') return 17;//大王
  return a-'3'+3;
}
//玩家的结构体，其中包括所有手牌的每种类型的最大值
struct Player{
  int dan,dui,t3,t3_1,t3_2,t4,t4_2,t4_4;
  int shun[8];//5~12
  Player(){
    dan=dui=t3=t3_1=t3_2=t4=t4_2=t4_4=0;
    memset(shun,0,sizeof(shun));
  }
  void clear()
  {
    dan=dui=t3=t3_1=t3_2=t4=t4_2=t4_4=0;
    memset(shun,0,sizeof(shun));
  }
}play[3];
//对第id个玩家的手牌进行处理
void solve(int id)
{
  Player &p = play[id];
  //单牌
  for (int i=17;i>=3;i--)
  {
    if (pai[id][i])
    {
      p.dan=i;
      break;
    }
  }
  //对子
  for (int i=15;i>=3;i--)
  {
    if (pai[id][i]>=2)
    {
      p.dui=i;
      break;
    }
  }
  //3个
  for (int i=15;i>=3;i--)
  {
    if(pai[id][i]==3)
    {
      p.t3=i;
      break;
    }
  }
  //3带1
  for (int i=3;i<=17 && p.t3>0;i++)
  {
    if (i==p.t3)
    continue;
    if (pai[id][i])
    {
      p.t3_1=p.t3;
      break;
    }
  }
  //3带1对
  for (int i=3;i<=15 && p.t3>0;i++)
  {
    if (i==p.t3)
    continue;
    if (pai[id][i]>=2)
    {
      p.t3_2=p.t3;
      break;
    }
  }
  //4个
  for (int i=15;i>=3;i--)
  {
    if (pai[id][i]==4)
    {
      p.t4=i;
      break;
    }
  }
  //4带2
  int c=0;
  for (int i=3;i<=17 && p.t4;i++)
  {
    if (i==p.t4)
    continue;
    c+=pai[id][i];
    if (c>=2)
    {
      p.t4_2=p.t4;
      break;
    }
  }
  //4带2对
  c=0;
  for (int i=3;i<=15 && p.t4 ;i++)
  {
    if (i==p.t4)
    continue;
    c+=pai[id][i]/2;
    if (c>=2)
    {
      p.t4_4=p.t4;
    }
  }
  //顺子
  for (int i=14;i>=7;i--)
  {
    int len_s=0;
    for (int j=i;j>=3;j--)
    {
      if(pai[id][j])
      {
        len_s++;
        if (len_s>=5)
        p.shun[len_s-5]=max(p.shun[len_s-5],i);
      }
      else
      break;
    }
  }
}
//判断是否可以一次出完
bool once(int cur)
{
  Player &p = play[cur];
  int len=strlen(s);
  if (len>=5 && len<=12 && p.shun[len-5])
  return true;
  if (len==8 && p.t4_4)
    return true;
  if (len==6 && p.t4_2)
    return true;
  if (len==5 && p.t3_2)
    return true;
  if (len==4 && (p.t3_1 || p.t4))
    return true;
  if (len==3 && p.t3)
    return true;
  if (len==2 && p.dui)
    return true;
  if (len==1)
    return true;
  return false;
}
int main()
{
  //freopen("3.in","r",stdin);
  //freopen("3.out","w",stdout);
  int cas;
  cin>>cas;
  for (int ca=1;ca<=cas;ca++)
  {
    play[0].clear();
    play[1].clear();
    play[2].clear();
    memset(pai,0,sizeof(pai));
    scanf("%s%s%s",s,t,t1);
    int lens=strlen(s),lent=strlen(t),lent1=strlen(t1);
    for (int i=0;i<lens;i++)
    pai[0][change(s[i])]++;
    for (int i=0;i<lent;i++)
    pai[1][change(t[i])]++;
    for (int i=0;i<lent1;i++)
    pai[2][change(t1[i])]++;
    solve(0);
    solve(1);
    solve(2);
    if (once(0))
    {
      printf("Winner\n");
      continue;
    }
    else
    {
      //王炸
      if (pai[0][16] && pai[0][17])
      {
        printf("Yes\n");
        continue;
      }
      if ((pai[1][16] && pai[1][17]) || (pai[2][16] && pai[2][17]))
      {
        printf("No\n");
        continue;
      }
      //第2或3名玩家有炸弹
      if (play[1].t4 || play[2].t4)
      {
        if (play[0].t4>play[1].t4 && play[0].t4>play[2].t4)
          printf("Yes\n");
        else
          printf("No\n");
        continue;
      }
      Player &p1=play[0];
      Player &p2=play[1];
      Player &p3=play[2];
      bool yes=false;
      if (p1.t4 || (p1.t3>p2.t3 && p1.t3>p3.t3) || (p1.t3_1>p2.t3_1 && p1.t3_1>p3.t3_1) || (p1.t3_2>p2.t3_2 && p1.t3_2>p3.t3_2))
      yes=true;
      if ((p1.dui && p1.dui>=p2.dui && p1.dui>=p3.dui) || (p1.dan && p1.dan>=p2.dan && p1.dan>=p3.dan))
      yes=true;
      for (int i=0;i<8;i++)
      {
        if (p1.shun[i] && p1.shun[i]>=p2.shun[i] && p1.shun[i]>=p2.shun[i])
        {
          yes=true;
          break;
        }
      }
      if (yes)
      printf("Yes");
      else
      printf("No");
      if (ca!=cas)
      printf("\n");
    }
  }
}
```

### E

```c++
#include <bits/stdc++.h>
using namespace std;
#define mp make_pair
#define fi first
#define se second
const int maxn=1000010;

vector<int> wordlen[1000010];
//int wordlen[1000010];
vector<pair<int,int> > word;

int cmp(pair<int,int> a,pair<int,int> b){
  if (a.se==b.se){
    return a.fi<b.fi;
  }
  return a.se<b.se;
}

void init1(){
  for (int i=0;i<1000005;i++) wordlen[i].clear();
  //memset(wordlen,0,sizeof wordlen);
  word.clear();
}
struct trie{
  int nex[maxn][26],fail[maxn],end[maxn];
  int root,L;
  int newnode(){
    for (int i=0;i<26;i++){
      nex[L][i]=-1;
    }
    end[L++]=0;
    return L-1;
  }
  void init(){
    L=0;
    root=newnode();
  }
  
  void insert(char buf[]){
    int len=strlen(buf);
    int now=root;
    for (int i=0;i<len;i++){
      if (nex[now][buf[i]-'a']==-1){
        nex[now][buf[i]-'a']=newnode();
      }
      now=nex[now][buf[i]-'a'];
    }
    end[now]++;
    //wordlen[now]=len;
    wordlen[now].push_back(len);
  }
  
  void build(){
    queue<int> Q;
    fail[root]=root;
    for (int i=0;i<26;i++){
      if (nex[root][i]==-1){
        nex[root][i]=root;
      }
      else{
        fail[nex[root][i]]=root;
        Q.push(nex[root][i]);
      }
    }
    while (!Q.empty()){
      int now=Q.front();
      Q.pop();
      for (int i=0;i<26;i++){
        if (nex[now][i]==-1){
          nex[now][i]=nex[fail[now]][i];
        }
        else{
          fail[nex[now][i]]=nex[fail[now]][i];
          Q.push(nex[now][i]);
        }
      }
    }
  }
  
  void query(char buf[]){
    int len=strlen(buf);
    int now=root;
    for (int i=0;i<len;i++){
      now=nex[now][buf[i]-'a'];
      int temp=now;
      while (temp!=root){
        int si=wordlen[temp].size();
        for (int j=0;j<si;j++){
          word.push_back(mp(i-wordlen[temp][j]+1,i));
        }
        end[temp]=0;
        temp=fail[temp];
      }
    }
  }
};

char buf[55];
char s[100010];
trie ac;

int main(){
  //freopen("11.in","r",stdin);
  //freopen("11.out","w",stdout);
  int T;
  int n;
  scanf("%d",&T);
  while (T--){
    scanf("%s",s);
    scanf("%d",&n);
    ac.init();
    init1();
    for (int i=0;i<n;i++){
      scanf("%s",buf);
      ac.insert(buf);
    }
    ac.build();
    //ac.debug();
    ac.query(s);
    sort(word.begin(),word.end(),cmp);
    int tmp=-1;
    int si=word.size();
    int ans=0;
    for (int i=0;i<si;i++){
      //printf("%d %d\n",word[i].fi,word[i].se);
      if (word[i].fi>tmp){
        tmp=word[i].se;
        ans++;
      }
    }
    printf("%d\n",ans);
  }
  
  return 0;
}
```

### F

```c++
#include<iostream>
#include<queue>
#include<cstring>
#include<cstdio>
#include<ctime>
using namespace std;
//#define DEBUG
int fileid=1;
const int maxn=1005;
const int inf=1e8+7;
struct Edge{
  int to,w,nxt;
  Edge(){};
  Edge(int to1,int w1,int nxt1)
  {
    to=to1;w=w1;nxt=nxt1;
  }
}e[maxn*3];
int head[maxn],tot,n;
//给图加边
void adde(int u,int v,int w)
{
  tot++;
  e[tot]=Edge(v,w,head[u]);
  head[u]=tot;
}
bool visit[maxn];
int dis[maxn],Count[maxn];
queue<int> q;
//spfa跑最长路判断是否有解
bool spfa()
{
  while (!q.empty())
  q.pop();
  for (int i=0;i<=n;i++)
  {
    dis[i]=0;
    q.push(i);
    Count[i]=0;
    visit[i]=true;
    //cout<<"cas:"<<i<<endl;
  }
  while (!q.empty())
  {
    int tmp=q.front();
    q.pop();
    visit[tmp]=false;
    Count[tmp]++;
    if (Count[tmp]>=n+1)
    return false;
    for (int i=head[tmp];i;i=e[i].nxt)
    {
      if (dis[e[i].to] < dis[tmp]+e[i].w)
      {
        dis[e[i].to]=dis[tmp]+e[i].w;
        if (!visit[e[i].to])
        {
          visit[e[i].to]=true;
          q.push(e[i].to);
        }
      }
    }

  }
  return true;
}
struct P{
  int b,c;
}res[1005];
//根据不同的pre[n]进行构图
void make_map(int pre_n)
{
  tot=0;
  memset(head,0,sizeof(head));
  for (int i=1;i<=n;i++)
  {
    int tmp=i+res[i].b-1,c=res[i].c;
    if (tmp>n)
    {
      tmp-=n;
      c-=pre_n;
    }
    //每个人的要求
    adde(i-1,tmp,c);
    //前缀和的性质 pre[i]-pre[i-1]≥0
    adde(i-1,i,0);
  }
  //pre[n]-pre[0]<=pre[n]
  adde(n,0,-pre_n);
} 
int main()
{
  while (scanf("%d",&n)!=EOF)
  {
    for (int i=1;i<=n;i++)
    scanf("%d%d",&res[i].b,&res[i].c);
    int ans=inf;
    int l=0,r=1e6+10;
    while (l<=r)
    {
      int mid=(l+r)>>1;
      make_map(mid);
      if (spfa())
      {
        ans=mid;
        r=mid-1;
      }
      else
      {
        l=mid+1;
      }
    }
    printf("%d\n",ans);
  }
}
```
