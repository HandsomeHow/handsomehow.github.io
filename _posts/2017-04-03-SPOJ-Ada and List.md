---
layout: post
title:  SPOJ Ada and List 离线二分
date:   2017-04-03 22:55:16
category: "二分"
---
[题目链接](http://www.spoj.com/problems/ADALIST/en/)

# 题意：

给你一个矩形,每次横的或者竖的加一条先,然后问你每次加完线以后面积最大的小矩形的面积是多少.

# 题解:

面积S=x*y,x和y是独立的,所以我们要找到被切割以后最长的一段x和最长的一段y.

如果每加一次遍历一遍的话复杂度是O(q^2),于是考虑倒着考虑,问题就变成了有一堆线的矩形,每次拿掉一根线,然后求最长的一段x和最长的一段y.

然后我们只需要二分找比他大的数就可以了,维护一下maxx和maxy,然后删掉一根线以后把他上下两根线的距离算一下然后更新一下maxx和maxy就可以了.

复杂度O(qlogq)

```c++
/*************************************************************************
    > File Name: C.cpp
    > Author: HandsomeHow
    > Mail: handsomehowyxh@gmail.com 
    > Created Time: 2017/4/3 22:21:06
 ************************************************************************/

#include <bits/stdc++.h>
using namespace std;

#define rep(i,a,b) for(int i=(a);i<=(b);i++)
#define per(i,a,b) for(int i=(a);i>=(b);i--)
#define pb push_back
#define mp make_pair
#define cl(a) memset((a),0,sizeof(a))
#ifdef HandsomeHow
#define dbg(x) cerr << #x << " = " << x << endl
#else
#define dbg(x)
#endif
typedef long long ll;
typedef unsigned long long ull;
typedef pair <int, int> pii;
const int inf=0x3f3f3f3f;
const double eps=1e-8;
const int mod=1000000007;
const double pi=acos(-1.0);
inline void gn(long long&x){
    int sg=1;char c;while(((c=getchar())<'0'||c>'9')&&c!='-');c=='-'?(sg=-1,x=0):(x=c-'0');
    while((c=getchar())>='0'&&c<='9')x=x*10+c-'0';x*=sg;}
inline void gn(int&x){long long t;gn(t);x=t;}
inline void gn(unsigned long long&x){long long t;gn(t);x=t;}
ll gcd(ll a,ll b){return a? gcd(b%a,a):b;}
ll powmod(ll a,ll x,ll mod){ll t=1ll;while(x){if(x&1)t=t*a%mod;a=a*a%mod;x>>=1;}return t;}
// (づ°ω°)づe★------------------------------------------------

const int maxn = 1e6+5;

int op[maxn], v[maxn];

map<int,int>x,y;

ll ans[maxn];

int main(){
#ifdef HandsomeHow
	freopen("data.in","r",stdin);
	//freopen("data.out","w",stdout);
#endif
	int T;gn(T);
	int n;
	while(T--){
		x.clear();y.clear();
		int foo, bar;
		gn(foo); gn(bar); gn(n);
		x[foo]++;x[0]++;
		y[bar]++;y[0]++;
		rep(i,1,n){
			gn(op[i]);gn(v[i]);
			if(op[i] == 0)
				x[v[i]]++;
			else
				y[v[i]]++;
		}
		int mxx = -1, mxy = -1;
		auto it1 = x.begin();
		auto it2 = it1;
		it1++;		//begin->(*it2)->it1->end;
		while(it2 != x.end()){
			int nn = (*it1).first;
			int mm = (*it2).first;
			mxx = max(mxx,(*it1).first-(*it2).first);
			it2++;it1++;
		}
		it1 = y.begin();
		it2 = it1;
		it1++;		//begin->(*it2)->it1->end;
		while(it2 != y.end()){
			mxy = max(mxy,(*it1).first-(*it2).first);
			it2++;it1++;
		}
		per(i,n,1){
			ans[i] = 1ll * mxx * mxy;
			if(op[i] == 0){
				x[v[i]]--;
				if(x[v[i]] == 0){
					x.erase(v[i]);
					auto ite1 = x.upper_bound(v[i]);
					auto ite2 = ite1; --ite2;
					mxx = max(mxx, (*ite1).first - (*ite2).first);
				}else{
					auto ite1 = x.upper_bound(v[i]);
					auto ite2 = ite1; --ite2; --ite2;
					mxx = max(mxx,max((*ite1).first - v[i], v[i] - (*ite2).first));
				}
			}else{
				y[v[i]]--;
				if(y[v[i]] == 0){
					y.erase(v[i]);
					auto ite1 = y.upper_bound(v[i]);
					auto ite2 = ite1; --ite2;
					mxy = max(mxy, (*ite1).first - (*ite2).first);
				}else{
					auto ite1 = y.upper_bound(v[i]);
					auto ite2 = ite1; --ite2; --ite2;
					mxy = max(mxy,max((*ite1).first - v[i], v[i] - (*ite2).first));
				}
			}
		}
		rep(i,1,n)
			printf("%lld\n",ans[i]);
	}
	return 0;
}
```
