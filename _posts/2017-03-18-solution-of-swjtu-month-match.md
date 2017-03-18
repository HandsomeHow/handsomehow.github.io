---
layout: post
title: swjtu2017-3月月赛题解
data: 2017-03-18 16:15
category: "套题"
---

# Problem A How many different roads?
假设纸张被划分为n*m个小方格。

由于每次只能沿着边缘上移或右移一格，故从左下角移动到右上角共需要移动(n+m)次。

并且右移的次数为n，上移的次数为m。因此问题即转化为我们要在(n+m)次移动中选出n次用来右移,

m次用来上移，而当n次右移确定了之后，m次上移也确定了(反之亦是如此)。

因此题目的答案即为C(m+n,n)或者C(m+n,m)。

注：

1.在计算C(m+n,n)或者C(m+n,m)时，应选择循环次数小的算法，否则会超时。

2.这个题目的结果范围是2的64次方-1，所以用int和long long都是会炸的，要用unsigned long long才可以过。

```c++
#include <bits/stdc++.h>
using namespace std;

#define rep(i,a,b) for(int i=(a);i<=(b);i++)
#define per(i,a,b) for(int i=(a);i>=(b);i--)

unsigned long long C(int n, int m){
	unsigned long long ret = 1;
	rep(i,1,m) ret = ret * (n - i + 1);
	rep(i,1,m) ret /= i;
	return ret;
}
int main(){
	int T; scanf("%d",&T);
	while(T--){
		ll n, m; scanf("%lld%lld",&n,&m);
		cout<<C(n+m,min(n,m))<<endl;
	}
	return 0;
}
```



# Problem B Find the string
当m=1时，输出n个a即可。

当m>=3时，输出abcabcabcab....(此时最大回文子串长度为1，且字典序最小)

当m=2时，可列举n=1~20的结果，便能比较容易的发现babbaa是以循环的规律出现的，之后再多分几种情况讨论即可，详见代码。
```c++
#include <stdio.h>
#include <time.h>

int main() {
    int T;
    int m,n;
    scanf("%d",&T);
    while(T--){
        scanf("%d%d",&m,&n);
        if(m == 1){
            for(int i = 0;i < n;i++)printf("a");
            printf("\n");
            continue;
        }
        if(m >= 3){
            int idx = 0;
            for(int i = 0;i < n;i++){
                printf("%c",idx + 'a');
                idx = (idx + 1)%3;
            }
            printf("\n");
            continue;
        }
        if(n == 1)printf("a");
        else if(n == 2)printf("ab");
        else if(n == 3)printf("aab");
        else if(n == 4)printf("aabb");
        else if(n == 5)printf("aaaba");
        else if(n == 6)printf("aaabab");
        else if(n == 7)printf("aaababb");
        else if(n == 8)printf("aaababbb");
        else if(n == 9)printf("aaaababba");
        else{
            printf("aaaa");
            n -= 4;
            while(n >= 6){
                printf("babbaa");
                n -= 6;
            }
            if(n == 1)printf("a");
            else if(n == 2)printf("aa");
            else if(n == 3)printf("bab");
            else if(n == 4)printf("babb");
            else if(n == 5)printf("babba");
        }
        printf("\n");
    }
    return 0;
}
```


# Problem C 分狗粮
暴搜+裁剪即可

通过下列裁剪会加快运算速度

1.将狗粮总数/4，不能整除直接输出no

2.如果有比平均数大的,直接输出no

3.dfs时如果前3个满足,则最后一个一定满足
```c++
#include<cstdio>
#include<cstring>
#include<algorithm>
using namespace std;

int arr[25], vis[25], ave;

int search(int n,int sum,int package,int index) {
	if(ave==sum) {
		index=sum=0;
		package++;
		if(package==3)	return 1;
	}
	for(int i=index;i<n;i++) {
		if(!vis[i]) {
			if(sum+arr[i]<=ave) {
				vis[i]=1;
				if(search(n,arr[i]+sum,package,i+1))	return 1;
				vis[i]=0;
			}
		}
	}
	return 0;
}

int main() {
	int T,sum,n;
	scanf("%d",&T);
	while(T--) {
		sum=0;
		scanf("%d",&n);
		for(int i=0;i<n;i++)	{
			scanf("%d",&arr[i]);
			sum+=arr[i];
		}
		memset(vis,0,sizeof(vis));
		ave=sum/4;

		if(sum%4 || arr[0]>ave)	printf("no\n");
		else {
			if(search(n,0,0,0))	printf("yes\n");
			else	printf("no\n");
		}
	}
	return 0;
}
```

# Problem D SSR，我也一定能出的
首先这个东西肯定是随机数，但是无论怎么随机，你的随机数种子得不一样吧，所以肯定不能用时间戳time()来作为种子，因为这两个程序是被同时运行的。那么你可以用别的乱七八糟的且不会相同的东西来作为种子(比如地址)
```c++
#include <stdio.h>
#include <time.h>
#include <stdlib.h>
int main(){
	int* a = new int[5];
	srand((long long)a);
	if(rand() & 1)
		puts("bamboo gets SR!");
	else
		puts("bamboo gets SSR!");
	return 0;
}
```

# Problem E 银河铁道之夜
二分答案，O(n)判断是否满足即可。
```c++
#include<iostream>
#include<cstdio>
#include<map>
#include<set>
#include<vector>
#include<stack>
#include<queue>
#include<string>
#include<cstring>
#include<sstream>
#include<algorithm>
#include<cmath>
#define INF 0x3f3f3f3f
#define eps 1e-8
#define pi acos(-1.0)
using namespace std;
typedef long long LL;
int n,m;
int a[10010];
bool pass(int k) {
    int l = 0,cnt = m;
    for (int i = 1;i <= n; i++) {
        if (a[i] - a[i-1] > k) return false;
        if (a[i] - a[l] > k) {
            l = i-1;
            if (--cnt < 0) return false;
        }
    }
    if (--cnt < 0) return false;
    return true;
}
int main() {
    int t;
    scanf("%d",&t);
    while (t--) {
        scanf("%d%d",&n,&m);
        for (int i = 1;i <= n; i++) {
            scanf("%d",&a[i]);a[i] += a[i-1];
        }
        int l = 0,r = a[n];
        while (r - l > 1) {
            int mid = (l + r) >> 1;
            if (pass(mid)) r = mid;
            else l = mid;
        }
        printf("%d\n",r);
    }
	return 0;
}
```
# Problem F 巧克力小姐的大惊小怪
简单的多重背包问题，而且数据规模很小完全可以当做01背包来做。
```c++
#include<iostream>
#include<cstdio>
#include<map>
#include<set>
#include<vector>
#include<stack>
#include<queue>
#include<string>
#include<cstring>
#include<sstream>
#include<algorithm>
#include<cmath>
#define INF 0x3f3f3f3f
#define eps 1e-8
#define pi acos(-1.0)
using namespace std;
typedef long long LL;
int a[210],b[210],c[210],f[210];
int main() {
    int t;
    scanf("%d",&t);
    int n,m;
    while (t--) {
        scanf("%d%d",&n,&m);
        for (int i = 1;i <= n; i++) scanf("%d%d%d",&a[i],&b[i],&c[i]);
        memset(f,0,sizeof(f));
        for (int i = 1;i <= n; i++)
            for (int j = 1;j <= a[i]; j++)
                for (int k = m;k >= b[i]; k--)
                    f[k] = max(f[k],f[k-b[i]] + c[i]);
        printf("%d\n",f[m]);
    }
	return 0;
}
```

# Problem G 抽卡的秘诀
我们记pre[k]为前k个cute的异或前缀(cute[1] ^ cute[2].... ^ cute[k])

对于每个a[i]，如果有右端点r,那么l必须满足条件 cute[l] ^ cute[l+1] ^ .... ^ cute[r] = a[i]

即 pre[r] ^ pre[l-1] = a[i]

那么pre[	l-1] = a[i] ^ pre[r]

因此我们可以开一个数组记录每一个前缀出现的次数，然后从左到右枚举右端点，对于每一个右端点我们把满足条件的左端点个数加到答案上，并且更新这个次数即可。

要注意：

1.cute[i]<=1e6,但这个前缀会大于这个值，最大为1048575(大于1e6的第一个2的幂-1)，所以数组要开大点

2.如果对每一个a[i],你都memset这个记录出现次数的数组的话，会TLE,所以对这个数组的清零方法是，把每一个前缀都减回去。或者我们可以对k个询问并行处理。

3.这个题需要把复杂度优化到O(n*k),所以对于每个前缀出现的次数要用一般数组来记录而不能用std::map

```c++
#include <bits/stdc++.h>
using namespace std;

#define rep(i,a,b) for(int i=(a);i<=(b);i++)
#define per(i,a,b) for(int i=(a);i>=(b);i--)

const int maxn = 1e4+5;
const int maxv = 1000000 * 2;
int cnt[maxv],a[maxn],v[maxn],ans[maxn];

int main(){
	int n,k;
	int T;scanf("%d",&T);
	while(T--){
		memset(ans,0,sizeof(ans));
		memset(cnt,0,sizeof(cnt));
		scanf("%d",&n);
		rep(i,1,n) scanf("%d",&a[i]);
		scanf("%d",&k);
		rep(i,1,k) scanf("%d",&v[i]);
		int pr = 0;
		cnt[0] = 1;
		rep(i,1,n){
			pr ^= a[i];
			rep(j,1,k){
				int to = v[j] ^ pr;
				ans[j] += cnt[to];
			}
			cnt[pr]++;
		}
		rep(i,1,k)
			printf("%d\n",ans[i]);
	}
	return 0;
}
```
# Problem H 找众数

数据范围很小，我们记录每一个数出现的次数然后遍历一遍就可以了。

```c++
#include <stdio.h>
#include <string.h>

int cnt[105];

int main(){
	int T;
	scanf("%d",&T);
	while(T--){
		int n;
		scanf("%d",&n);
		memset(cnt,0,sizeof(cnt));
		for(int i = 0; i < n; ++i){
			int x;scanf("%d",&x);
			cnt[x]++;
		}
		int mx = -1, ans;
		for(int i = 0; i <= 100; ++i){
			if(cnt[i] > mx){
				mx = cnt[i];
				ans = i;
			}
		}
		printf("%d\n",ans);
	}
	return 0;
}
```
