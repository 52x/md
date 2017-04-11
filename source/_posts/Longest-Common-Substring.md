title: 最长公共子串问题的几种算法
date: 2015-04-21 23:19:32
categories: Algorithms
tags: [LCS,DP,HASH,suffix array,SAM]
---
# 一. 最长公共子串
**最长公共子串(Longest Common Substring ,简称LCS)**问题，是指求给定的一组字符串	长度最大的共有的子串的问题。例如字符串"abcb","bca","acbc"的LCS就是"bc"。`LCS`同时也是`Longest Common Subsequence`的缩写，注意区分`子串(Substring)`和`子序列(Subsequence)`的区别，子串必须连续，而子序列可以不连续。

本文首先从求解*两个字符串*的LCS入手，介绍暴力穷举、DP、字符串哈希等算法在处理该问题的应用，并分析复杂度。之后扩展至求*多个字符串*的LCS，引入后缀数组、后缀自动机等工具，并分析复杂度。

<!-- more -->

*变量说明*
`L`: 字符串的最大长度
`K`: 最长公共子串的长度
`N`: N个字符串

# 二. 两个字符串的LCS
## 2.1 朴素穷举算法
显然该算法极端的低效，可以枚举A串以i开始，B串以j开始，向后最长有多少相同字符。算法时间复杂度O(L^2*K)。
```cpp
int comlen(char * p, char * q) {
    int len = 0;
    while(*p && *q && *p++ == *q++) {
        ++len;
    }
    return len;
}

int LCS_base(char * X, int xlen, char * Y, int ylen) {
    for(int i = 0; i < xlen; ++i) {
        for(int j = 0; j < ylen; ++j) {
            int len = comlen(&X[i],&Y[j]);
            if(len > maxlen) {
                maxlen = len;
                maxindex = i;
            }
        }
    }
    return maxlen;
}
```

## 2.2 动态规划算法
最长公共子序列(Longest Common Subsequence)是一个经典的DP，本文谈及的最长公共子串也可以用类似的方法解决。DP状态f[i,j]代表A[0..i-1]和B[0..j-1]的最长公共子串长度。f[lenA-1,lenB-1]为所求。状态转移为:
```bash
f[i,j] = f[i-1,j-1]+1 (A[i]==B[j])
		0 	ohterwise
```
时间复杂度为O(L^2)，空间复杂度也是O(L^2)。如果使用滚动数组或者降维的方法，可以将空间复杂度优化至O(L)，时间复杂度不变。

```cpp
int LCS_dp(char * X, int xlen, char * Y, int ylen) {
    memset(dp,0,sizeof(dp));
    int maxlen =0, maxindex = 0;
    for(int i = 0; i < xlen; ++i) {
        for(int j = ylen-1; j >= 0; --j) {
            if(X[i] == Y[j]) {
                if(i && j) {
                    dp[j] = dp[j-1] + 1;
                }
                if(i == 0 || j == 0) {
                    dp[j] = 1;
                }
                if(dp[j] > maxlen) {
                    maxlen = dp[j];
                    maxindex = i + 1 - maxlen;
                }
            } else {
                dp[j] = 0;
            }
        }
    }
    int maxlen;
}
```

## 2.3 字符串哈希
字符串hash的方法有许多种，对一个长度为L的字符串，进行O(L)时间的预处理后，可以常数时间获得该字符串某个子串的hash值。

对于LCS问题，可以二分枚举答案K，假设有长度为K的公共子串，并对K的可行性进行验证。如果验证K可行，K'(K'<K)也一定可行，尝试增加K，反之尝试缩小K，至多只会尝试O(LogL)次。

接下来的问题集中到，如果应用字符串哈希的算法尽可能高效的验证是否存在长度为K的公共子串。可以将A串的所有长度为K的子串hash值存入set容器，之后遍历B串中所有长度为K的子串，如果hash值在set中出现，则说明存在长度K的公共子串。长度为K的子串至多有L个，set插入和查询的复杂度为log。因此每次验证K的时间复杂度为LlogK，因此该算法总的时间复杂度为O(L*LogK*logL)。set可以用hashtable代替，时间复杂度可达到O(L*logL)。
```cpp
const ull B = 1e8+7;	/*according to the book*/
const int MAXN = 100000+100;
ull ah[MAXN],base[MAXN];

int C(int len) {
    int pos=m-len+1;
    ull bh=0,tmp=0;
    for(int i=0; i<len; i++)
        tmp=tmp*B+a[i];
    ah[0]=tmp;
    for(int i=0; i+len<=n; i++) /////////
        ah[i+1]=ah[i]*B+a[i+len]-a[i]*base[len];
    sort(ah,ah+n-len+1);
    for(int i=0; i<len; i++)
        bh=bh*B+b[i];
    for(int k=0; k<pos; k++) {
        if(binary_search(ah,ah+n-len+1,bh)) {
            return 1;
        }
        bh=bh*B+b[k+len]-b[k]*base[len];
    }
    return 0;
}

int solve() {
    n=strlen(a),m=strlen(b);// a--long b-short
    if(n<m) {
        swap(n,m);
        strcpy(tmp,a);
        strcpy(a,b);
        strcpy(b,tmp);
    }
    int d=0,up=m+1,mid;
    while(up>d+1) {
        mid=(d+up)/2;
        if(C(mid))d=mid;
        else up=mid;
    }
    return d;
}
```

# 三. N个字符串的LCS
将问题扩展至N个字符串的最长公共子串后，DP算法时间复杂度会陡然变高。而hash算法的时间复杂度变为O(NLlogL)，仅是随着N线性增加，不过hash算法可能会损失一定的正确性。下面再介绍几种可以处理N个字符串LCS的算法，利用扩展KMP算法，时间复杂度是O(NL^2)；后缀数组的时间复杂度是O(NLlogL)；后缀自动机的复杂度甚至可以做到O(NL)，因此对于两个串的LCS，后缀自动机是线性时间复杂度算法，非常巧妙。

## 3.1 扩展KMP
扩展KMP，顾名思义是KMP算法的一种扩展，它可以解决这样的问题：给出串A和串B，长度分别为lenA和lenB，要求在线性时间内，对于每个A[i]（0<=i<lenA)，求出A[i..lenA-1]与B的最长公共前缀长度，记为ex[i]（或者说，ex[i]为满足A[i..i+z-1]==B[0..z-1]的最大的z值）。扩展KMP的时间复杂度是线性的，只是常数会比KMP更大。

利用扩展KMP解决LCS问题时，假设有A、B两个字符串。可以枚举B串的每一个后缀，用这个后缀与A串做扩展KMP，遍历A[i]和这个后缀的最长公共前缀，最大值就是LCS的解。B串每个后缀做扩展KMP的时间复杂度是O(L),一共L个后缀，因此算法时间复杂度是O(L^2)，这个算法可以直接应用于N个字符串的LCS，时间复杂度为O(N*L^2)。

```cpp
int next[N],extand[N];
void getnext(char *T) { // next[i]: 以第i位置开始的子串 与 T的公共前缀
    int i,length = strlen(T);
    next[0] = length;
    for(i = 0; i<length-1 && T[i]==T[i+1]; i++);
    next[1] = i;
    int a = 1;
    for(int k = 2; k < length; k++) {
        int p = a+next[a]-1, L = next[k-a];
        if( (k-1)+L >= p ) {
            int j = (p-k+1)>0? (p-k+1) : 0;
            while(k+j<length && T[k+j]==T[j]) j++;// 枚举(p+1，length) 与(p-k+1,length) 区间比较
            next[k] = j, a = k;
        } else next[k] = L;
    }
}
//求出S[i..lenS-1]与T的最长公共前缀长度，记为extand[i]
void getextand(char *S,char *T) {
    memset(next,0,sizeof(next));
    getnext(T);
    int Slen = strlen(S), Tlen = strlen(T), a = 0;
    int MinLen = Slen>Tlen?Tlen:Slen;
    while(a<MinLen && S[a]==T[a]) a++;
    extand[0] = a, a = 0;
    for(int k = 1; k < Slen; k++) {
        int p = a+extand[a]-1, L = next[k-a];
        if( (k-1)+L >= p ) {
            int j = (p-k+1)>0? (p-k+1) : 0;
            while(k+j<Slen && j<Tlen && S[k+j]==T[j] ) j++;
            extand[k] = j;
            a = k;
        } else extand[k] = L;
    }
}

char a[3010],b[3010];
void LCS(char *a, char *b) {
    int ans = 0;
    int lena = strlen(a), lenb = strlen(b);
    for(int pb=0; pb<lenb; ++pb) {
        getextand(a,b+pb);
        for(int j=0; j<lena; ++j)
            ans = max(ans, extand[j]);
    }
    printf("%d\n",ans);
}
```

## 3.2 后缀数组
后缀数组是解决字符串问题的有力工具！IOI国家集训队的两篇论文非常值得一读*《后缀数组》(2004年许智磊)*和*《后缀数组——处理字符串的有力工具》(2009年罗穗骞)*。尤其是2009年的论文，里面给出了很多经典的例题。关于[最长公共子串问题的后缀数组解法](https://www.byvoid.com/blog/lcs-suffix-array/)，byvoid在他的文章中也有详细的讲解。本文再次简要复述算法，SA[i]表示排名第i的后缀的位置，Height[i]表示后缀SA[i]和SA[i-1]的最长公共前缀(Longest Common Prefix,LCP)，简记为Height[i]=LCP(SA[i],SA[i-1])。在本文2.3节已经说明LCS问题可以二分答案，因此我们只需要讨论如何用后缀数组验证是否存在长度为K的公共子串。设N个串分别为S1,S2,S3,...,SN，首先建立一个串S，把这N个串用不同的分隔符连接起来。验证时，找出出在Height数组中找出连续的一段Height[i..j]，使得i<=k<=j均满足Height[k]>=K，并且i-1<=k<=j中，SA[k]分属于原有N个串S1..SN。如果能找到这样的一段，那么K就是可行解。算法时间复杂度O(N*L*logL)。
```cpp
//以下为二分验证函数
bool check(int K)
{
    int i,j,k;
    bool ba[MAXN];
    for (i=1;i<=SA.N;i++)
    {
        if (SA.Height[i]>=K)
        {
            for (j=i;SA.Height[j]>=K && j<=SA.N;j++);
            j--;
            memset(ba,0,sizeof(ba));
            for (k=i-1;k<=j;k++)
                ba[Bel[SA.SA[k]]]=true;
            for (k=1;ba[k] && k<=N;k++);
            if (k==N+1)
                return true;
            i=j;
        }
    }
    return false;
}
```

## 3.3 后缀自动机
关于后缀自动机(SAM)的教程，可以参考[2012年noi冬令营陈立杰讲稿](http://wenku.baidu.com/link?url=cZvtkdkd8D1FYn6a6BTa8glhSwdCVlSoFvNfZbGCS5yp-Jt9ZK0A9WNZkQbe0WrCV3yBzYDloATSa1S1pOZWoYtvhyiX25Fgj3HFKf7IAnm)。SAM的内容相对复杂，若要展开讨论需要较长篇幅，在此不赘述SA性质和构造方法。对于串A和B，我们先构造出串A的后缀自动机，那么然后用B串去匹配，对于B，我们一位一位地扫描，维护一个ans值，表示从B串的开始到B[i]的这个子串与A的最长公共子串。假设现在到B[i-1]的最长公共子串长度为ans，然后我们来看B[i]，如果当前节点有B[i]这个孩子，那么直接就len++即可。如果没有就找一直向前找pre，直到找到有B[i]这个孩子的节点。SAM可以在线性时间内构造，因此时间复杂度为O(N*L)。虽然SAM算法较为复杂，但是代码却非常简短。
```cpp
struct SAM {
    SAM *pre,*son[26];
    int len,g;
} que[N],*root,*tail;
int tot;
void add(int c,int l) {
    SAM *p=tail,*np=&que[tot++];
    np->len=l;
    tail=np;
    while(p&&p->son[c]==NULL) p->son[c]=np,p=p->pre;
    if(p==NULL) np->pre=root;
    else {
        SAM *q=p->son[c];
        if(p->len+1==q->len) np->pre=q;
        else {
            SAM *nq=&que[tot++];
            *nq=*q;
            nq->len=p->len+1;
            np->pre=q->pre=nq;
            while(p&&p->son[c]==q) p->son[c]=nq,p=p->pre;
        }
    }
}
char a[N/2],b[N/2];
int lcs(char a[],char b[]) {
    memset(que,0,sizeof(que));
    tot=0;
    root=tail=&que[tot++];
    for(int i=0; a[i]; i++) add(a[i]-'a',i+1);
    SAM *p=root;
    int ans=0;
    for(int i=0,l=0; b[i]; i++,ans=max(ans,l)) {
        int c=b[i]-'a';
        if(p->son[c]) p=p->son[c],l++;
        else {
            while(p&&p->son[c]==NULL) p=p->pre;
            if(p==NULL) p=root,l=0;
            else l=p->len+1,p=p->son[c];
        }
    }
    return ans;
}
```
# 四. 总结
本文利用几种常见的字符串工具实现了LCS问题，从暴力穷举算法逐步优化，直至线性时间复杂度的SAM算法。逐步寻找最优解是本文的初衷，同时也可以感知到高效的算法往往简洁而优美。

本文提及的算法复杂度总结如下：
![LCS各算法复杂度分析](http://7tsysl.com1.z0.glb.clouddn.com/LCS Complexity.png)
`L`: 字符串的最大长度
`K`: 最长公共子串的长度
`N`: N个字符串
