# 基础数据结构

## 并查集

最朴素的并查集，查找根节点，合并，查询都是 $O(n)$。

我们分别有路径压缩，启发式合并 / 按秩合并来优化。

启发式合并：按照节点数小的连向节点数大的。

按秩合并：按照最大深度小的连向最大深度大的。

一般启发式合并比按秩合并好写，往往可以用启发式合并来代替按秩合并。

!!! note "时间复杂度分析"
    同时使用启发式合并和路径压缩的时间复杂度是 $O(m\alpha (m, n))$。

    不使用启发式合并、只使用路径压缩的最坏时间复杂度是 $O(m\log n)$，在平均情况下，时间复杂度依然是 $O(m\alpha (m, n))$。

    如果只使用启发式合并，而不使用路径压缩，时间复杂度为 $O(m\log n)$。由于路径压缩单次合并可能造成大量修改，有时路径压缩并不适合使用。例如，在可持久化并查集、线段树分治 + 并查集中，一般使用只启发式合并的并查集。


### 代码实现

- 启发式合并

```cpp
struct DSU {
    std::vector<int> fa, size;

    int find(int x) { return fa[x] == x ? x : find(fa[x]); }

    explicit DSU(int size_) : fa(size_), size(size_, 1) {
        std::iota(fa.begin(), fa.end(), 0);
    }

    bool query(int x, int y) {
    	return (find(x) == find(y));
    }

    void merge(int x, int y) {
        x = find(x), y = find(y);
        if (x == y) return;
        if (size[x] < size[y]) std::swap(x, y);
        fa[y] = x;
        size[x] += size[y];
    }

    void move(int x, int y) {
        auto fx = find(x), fy = find(y);
        if (fx == fy) return;
        fa[x] = fy;
        --size[fx], ++size[fy];
    }
};
```

- 按秩合并

```cpp
struct DSU {
    std::vector<int> fa, rk;

    int find(int x) { return fa[x] == x ? x : find(fa[x]); }

    explicit DSU(int size_) : fa(size_), rk(size_, 1) {
        std::iota(fa.begin(), fa.end(), 0);
    }

    bool query(int x, int y) {
    	return (find(x) == find(y));
    }

    void merge(int x, int y) {
        x = find(x), y = find(y);
        if (rk[x] <= rk[y])
            fa[x] = y;
        else
            fa[y] = x;
        if (rk[x] == rk[y] && x != y)
            rk[y]++;
    }
};
```

- 可撤回并查集

这里使用的启发式合并，注意这里不可以路径压缩。

```cpp
struct DSU {
    std::vector<int> siz;
    std::vector<int> f;
    std::vector<std::array<int, 2>> his;
    
    DSU(int n) : siz(n + 1, 1), f(n + 1) {
        std::iota(f.begin(), f.end(), 0);
    }
    
    int find(int x) {
        while (f[x] != x) {
            x = f[x];
        }
        return x;
    }
    
    bool query(int x, int y) {
    	return find(x) == find(y);
    }

    bool merge(int x, int y) {
        x = find(x);
        y = find(y);
        if (x == y) {
            return false;
        }
        if (siz[x] < siz[y]) {
            std::swap(x, y);
        }
        his.push_back({x, y});
        siz[x] += siz[y];
        f[y] = x;
        return true;
    }
    
    int time() {
        return his.size();
    }
    
    void revert(int tm) {
        while (his.size() > tm) {
            auto [x, y] = his.back();
            his.pop_back();
            f[y] = y;
            siz[x] -= siz[y];
        }
    }
};
```

### 扩展域并查集

又称为种类并查集。

本质上是拆点找关系，一般的并查集都是找**朋友的朋友也是朋友**这层关系，但是带权并查集是找**敌人的敌人就是朋友**这层关系。我们把 $1 \sim n$ 扩展为 $1 \sim 2n$，其中 $1$ 被拆为了 $1$ 和 $1 + n$。和 $1$ 在一个连通块表示和他关系好，和 $n + 1$ 在一个连通块表示和他关系不好。

- 例题

[**P2024 [NOI2001] 食物链**](https://www.luogu.com.cn/problem/P2024)

**Solution**

经典扩展并查集的题目了。

我们开三倍空间，$\{i + n\}$ 表示 $\{i\}$ 的捕食者，$\{i + 2n\}$ 表示 $\{i + n\}$ 的捕食者，$\{i\}$ 表示 $\{i + 2n\}$ 的捕食者。然后分别建立连接关系即可。

```cpp
struct DSU {
	std::vector<size_t> fa, size;

	size_t find(size_t x) { return fa[x] == x ? x : find(fa[x]); }

    explicit DSU(size_t size_) : fa(size_), size(size_, 1) {
        std::iota(fa.begin(), fa.end(), 0);
    }

    bool query(size_t x, size_t y) {
    	return (find(x) == find(y));
    }

    void merge(size_t x, size_t y) {
        x = find(x), y = find(y);
        if (x == y) return;
        if (size[x] < size[y]) std::swap(x, y);
        fa[y] = x;
        size[x] += size[y];
    }
};


void solve() {
	int n, k;
	std::cin >> n >> k;
	DSU dsu(3 * n + 10);
	int ans = 0;
	while (k--) {
		int op, x, y;
		std::cin >> op >> x >> y;
		if (x > n || y > n) {
			ans++;
			continue;
		}
		if (op == 1) {
			if (dsu.query(x + 2 * n, y + n) || dsu.query(x + n, y + 2 * n)) {
				ans++;
			}
			else {
				dsu.merge(x, y);
				dsu.merge(x + n, y + n);
				dsu.merge(x + 2 * n, y + 2 * n);
			}
		}
		else {
			if (dsu.query(x + n, y + n) || dsu.query(x + n, y + 2 * n)) {
				ans++;
			}
			else {
				dsu.merge(x + 2 * n, y + n);
				dsu.merge(x + n, y);
				dsu.merge(x, y + 2 * n);
		    }
	    }
    }
    std::cout << ans << '\n';
}
```

本题也可以用带权并查集去做。

我们记点到根节点的距离为 $d_i$，我们在一个模 $3$ 的域下看点之间的关系。

若 $d_x - d_y \equiv 0(\mod 3)$ 则说明 $x$ 和 $y$ 是同类。

若 $d_x - d_y \equiv 1(\mod 3)$ 则说明 $x$ 是 $y$ 的天敌。

若 $d_x - d_y \equiv 2(\mod 3)$ 则说明 $x$ 是 $y$ 的猎物。


### 带权并查集

- 例题

带权并查集，顾名思义，连边的时候是带有权值的，往往是要维护每个元素到根节点的某一属性值。需要改变 `find` 函数。

[**P1196 [NOI2002] 银河英雄传说**](https://www.luogu.com.cn/problem/P1196)

**Solution**

我们要维护每个点到根节点的距离，此外两个队头合并的时候还要记录每个队伍的长度，所以用 `dis` 和 `num` 记录。

使用带权并查集维护即可。

```cpp
struct DSU {
    std::vector<size_t> fa;
    std::vector<int> dis, num;

    size_t find(size_t x) { 
    	if (fa[x] == x) return x;
    	int fx = find(fa[x]);
    	dis[x] += dis[fa[x]];
    	return fa[x] = fx;
    }

    explicit DSU(size_t size_) : fa(size_), dis(size_, 0), num(size_, 1) {
        std::iota(fa.begin(), fa.end(), 0);
    }

    bool query(size_t x, size_t y) {
    	return (find(x) == find(y));
    }

    void merge(size_t x, size_t y) {
        x = find(x), y = find(y);
        if (x == y) return;
        fa[x] = y;
        dis[x] = num[y];
        num[y] += num[x];
    }
};

void solve() {
	DSU dsu(30010);
	int t;
	std::cin >> t;
	int n = 4;
	while (t--) {
		char op;
		int x, y;
		std::cin >> op >> x >> y;
		if (op == 'M') {
			dsu.merge(x, y);
		}
		else {
			if (dsu.query(x, y)) {
				std::cout << std::abs(dsu.dis[x] - dsu.dis[y]) - 1 << '\n';
			}
			else {
				std::cout << "-1\n";
			}
		}
	}
}
```

## 树状数组

### 代码实现

```cpp
template<typename T>
struct Fenwick{
    int n;
    vector<T> tr;
 
    Fenwick(int n) : n(n), tr(n + 1, 0){}
 
    int lowbit(int x) {
        return x & -x;
    }
 
    void modify(int x, T c) {
        for(int i = x; i <= n; i += lowbit(i)) tr[i] += c;
    }
 
    void modify(int l, int r, T c) {
        modify(l, c);
        if (r + 1 <= n) modify(r + 1, -c);
    }
 
    T query(int x) {
        T res = T();
        for(int i = x; i; i -= lowbit(i)) res += tr[i];
        return res;
    }
 
    T query(int l, int r) { 
        return query(r) - query(l - 1);
    }
 
    int find_first(T sum) {
        int ans = 0; T val = 0;
        for(int i = __lg(n); i >= 0; i--) {
            if ((ans | (1 << i)) <= n && val + tr[ans | (1 << i)] < sum){
                ans |= 1 << i;
                val += tr[ans];
            }
        }
        return ans + 1;
    }
 
    int find_last(T sum) {
        int ans = 0; T val = 0;
        for(int i = __lg(n); i >= 0; i--) {
            if ((ans | (1 << i)) <= n && val + tr[ans | (1 << i)] <= sum){
                ans |= 1 << i;
                val += tr[ans];
            }
        }
        return ans;
    }
 
};
using BIT = Fenwick<int>;
```



## 习题


[**CF2018D Max Plus Min Plus Size**](https://codeforces.com/contest/2018/problem/D)

> 给定长度 $n$ 数组，不能选择相邻的元素，要求选择的最大元素，最小元素，选择个数之和最大。 

**Solution**

挺巧妙的题，并查集做法。略微有点细节。

不难发现，如果最终选择的元素没有整个数组的最大值，那么我们可以换一下选择的奇偶选择最大值，这样答案是不减的。

所以最大值一定会选上。这下我们可以从大到小枚举最小值，这是相当于每次加入若干新的数，将整个数组分成若干段。用并查集来维护统计个数。

注意的是，我们还要记录每段取最大情况的时候是否能取到最大值。对于每一段，我们可能尽量取 $\left \lceil \dfrac{len}{2} \right \rceil$，所以如果我们每一段都这样取的话可能会出现没取最大值的情况，所以要判断一下每段奇数位和偶数位能不能取到最大值。如果全都没取到最后答案要减一（少取一个数来取最大值）。

以及，我们不用考虑当前枚举的最小值是否真的会被取到。假设没被取到，那么一定没有之前的优，所以不会对答案产生影响。



## 参考资料

[OI-Wiki 并查集](https://oi-wiki.org/ds/dsu/)

[算法学习笔记(7)：种类并查集](https://zhuanlan.zhihu.com/p/97813717)
