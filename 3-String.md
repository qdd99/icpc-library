## 字符串

### 哈希

```cpp
// open hack不要用哈希
using ull = unsigned long long;

const int x = 135, p1 = 1e9 + 7, p2 = 1e9 + 9;
const ull mask32 = ~(0u);

ull xp1[MAXN], xp2[MAXN];

void init_xp() {
    xp1[0] = xp2[0] = 1;
    for (int i = 1; i < MAXN; i++) {
        xp1[i] = xp1[i - 1] * x % p1;
        xp2[i] = xp2[i - 1] * x % p2;
    }
}

struct Hash {
    vector<ull> h;

    Hash() : h(1) {}

    void add(const string& s) {
        ull res1 = h.back() >> 32;
        ull res2 = h.back() & mask32;
        for (char c : s) {
            res1 = (res1 * x + c) % p1;
            res2 = (res2 * x + c) % p2;
            h.push_back((res1 << 32) | res2);
        }
    }

    ull get(int l, int r) {
        r++;
        int len = r - l;
        ull l1 = h[l] >> 32, r1 = h[r] >> 32;
        ull l2 = h[l] & mask32, r2 = h[r] & mask32;
        ull res1 = (r1 - l1 * xp1[len] % p1 + p1) % p1;
        ull res2 = (r2 - l2 * xp2[len] % p2 + p2) % p2;
        return (res1 << 32) | res2;
    }
};
```

+ 二维哈希

```cpp
const ll basex = 239, basey = 241, p = 998244353;

ll pwx[MAXN], pwy[MAXN];

void init_xp() {
    pwx[0] = pwy[0] = 1;
    for (int i = 1; i < MAXN; i++) {
        pwx[i] = pwx[i - 1] * basex % p;
        pwy[i] = pwy[i - 1] * basey % p;
    }
}

struct Hash2D {
    vector<vector<ll> > h;

    Hash2D(const vector<vector<int> >& a, int n, int m) : h(n + 1, vector<ll>(m + 1)) {
        for (int i = 0; i < n; i++) {
            ll s = 0;
            for (int j = 0; j < m; j++) {
                s = (s * basey + a[i][j] + 1) % p;
                h[i + 1][j + 1] = (h[i][j + 1] * basex + s) % p;
            }
        }
    }

    ll get(int x, int y, int xx, int yy) {
        ++xx; ++yy;
        int dx = xx - x, dy = yy - y;
        ll res = h[xx][yy]
            - h[x][yy] * pwx[dx]
            - h[xx][y] * pwy[dy]
            + h[x][y] * pwx[dx] % p * pwy[dy];
        return (res % p + p) % p;
    }
};
```

### Manacher

```cpp
// "aba" => "#a#b#a#"
string make(string& s) {
    string t = "#";
    for (int i = 0; i < s.size(); i++) {
        t.push_back(s[i]);
        t.push_back('#');
    }
    return t;
}

void manacher(string& s, vector<int>& d) {
    int n = s.size();
    d.resize(n);
    for (int i = 0, l = 0, r = -1; i < n; i++) {
        int k = (i > r) ? 1 : min(d[l + r - i], r - i);
        while (i - k >= 0 && i + k < n && s[i - k] == s[i + k]) k++;
        d[i] = --k;
        if (i + k > r) {
            l = i - k;
            r = i + k;
        }
    }
}
```

### KMP

```cpp
// 前缀函数（每一个前缀的最长公共前后缀）
void get_pi(const string& s, vector<int>& a) {
    int n = s.size(), j = 0;
    a.resize(n);
    for (int i = 1; i < n; i++) {
        while (j && s[j] != s[i]) j = a[j - 1];
        if (s[j] == s[i]) j++;
        a[i] = j;
    }
}

void kmp(const string& s, vector<int>& a, const string& t) {
    int j = 0;
    for (int i = 0; i < t.size(); i++) {
        while (j && s[j] != t[i]) j = a[j - 1];
        if (s[j] == t[i]) j++;
        if (j == s.size()) {
            // ...
            j = a[j - 1]; // 允许重叠匹配 j = 0 不允许
        }
    }
}

// Z函数（每一个后缀和该字符串的最长公共前缀）
void get_z(const string& s, vector<int>& z) {
    int n = s.size(), l = 0, r = 0;
    z.resize(n);
    for (int i = 1; i < n; i++) {
        if (i <= r) z[i] = min(r - i + 1, z[i - l]);
        while (i + z[i] < n && s[z[i]] == s[i + z[i]]) z[i]++;
        if (i + z[i] - 1 > r) {
            l = i;
            r = i + z[i] - 1;
        }
    }
}
```

### 最小表示法

```cpp
int get(const string& s) {
    int k = 0, i = 0, j = 1, n = s.size();
    while (k < n && i < n && j < n) {
        if (s[(i + k) % n] == s[(j + k) % n]) {
            k++;
        } else {
            s[(i + k) % n] > s[(j + k) % n] ? i = i + k + 1 : j = j + k + 1;
            if (i == j) i++;
            k = 0;
        }
    }
    return min(i, j);
}
```

### Trie

```cpp
// 01 Trie
struct Trie {
    int t[31 * MAXN][2], sz;

    void init() {
        memset(t, 0, 2 * (sz + 2) * sizeof(int));
        sz = 1;
    }

    void insert(int x) {
        int p = 0;
        for (int i = 30; i >= 0; i--) {
            bool d = (x >> i) & 1;
            if (!t[p][d]) t[p][d] = sz++;
            p = t[p][d];
        }
    }
};

// 正常Trie
struct Trie {
    int t[MAXN][26], sz, cnt[MAXN];

    void init() {
        memset(t, 0, 26 * (sz + 2) * sizeof(int));
        memset(cnt, 0, (sz + 2) * sizeof(int));
        sz = 1;
    }

    void insert(const string& s) {
        int p = 0;
        for (char c : s) {
            int d = c - 'a';
            if (!t[p][d]) t[p][d] = sz++;
            p = t[p][d];
        }
        cnt[p]++;
    }
};
```

### AC 自动机

```cpp
struct ACA {
    int t[MAXN][26], sz, fail[MAXN], nxt[MAXN], cnt[MAXN];

    void init() {
        memset(t, 0, 26 * (sz + 2) * sizeof(int));
        memset(fail, 0, (sz + 2) * sizeof(int));
        memset(nxt, 0, (sz + 2) * sizeof(int));
        memset(cnt, 0, (sz + 2) * sizeof(int));
        sz = 1;
    }

    void insert(const string& s) {
        int p = 0;
        for (char c : s) {
            int d = c - 'a';
            if (!t[p][d]) t[p][d] = sz++;
            p = t[p][d];
        }
        cnt[p]++;
    }

    void build() {
        queue<int> q;
        for (int i = 0; i < 26; i++) {
            if (t[0][i]) q.push(t[0][i]);
        }
        while (!q.empty()) {
            int u = q.front();
            q.pop();
            for (int i = 0; i < 26; i++) {
                int& v = t[u][i];
                if (v) {
                    fail[v] = t[fail[u]][i];
                    nxt[v] = cnt[fail[v]] ? fail[v] : nxt[fail[v]];
                    q.push(v);
                } else {
                    v = t[fail[u]][i];
                }
            }
        }
    }
};
```

### 回文自动机

```cpp
// WindJ0Y
struct Palindromic_Tree {
    static constexpr int MAXN = 300005;
    static constexpr int N = 26;

    int next[MAXN][N]; // next指针，next指针和字典树类似，指向的串为当前串两端加上同一个字符构成
    int fail[MAXN]; // fail指针，失配后跳转到fail指针指向的节点
    int cnt[MAXN]; // 表示节点i表示的本质不同的串的个数 aftre count()
    int num[MAXN]; // 表示以节点i表示的最长回文串的最右端点为回文串结尾的回文串个数。
    int len[MAXN]; // len[i]表示节点i表示的回文串的长度
    int lcnt[MAXN];
    int S[MAXN]; // 存放添加的字符
    int last; // 指向上一个字符所在的节点，方便下一次add
    int n; // 字符数组指针
    int p; // 节点指针

    int newnode(int l, int vc) { // 新建节点
        for (int i = 0; i < N; ++i) next[p][i] = 0;
        cnt[p] = 0;
        num[p] = 0;
        len[p] = l;
        lcnt[p] = vc;
        return p++;
    }

    void init() { // 初始化
        p = 0;
        newnode(0, 0);
        newnode(-1, 0);
        last = 0;
        n = 0;
        S[n] = -1; // 开头放一个字符集中没有的字符，减少特判
        fail[0] = 1;
    }

    int get_fail(int x) { // 和KMP一样，失配后找一个尽量最长的
        while (S[n - len[x] - 1] != S[n]) x = fail[x];
        return x;
    }

    void add(int c) {
        S[++n] = c;
        int cur = get_fail(last); // 通过上一个回文串找这个回文串的匹配位置
        if (!next[cur][c]) { // 如果这个回文串没有出现过，说明出现了一个新的本质不同的回文串
            int now = newnode(len[cur] + 2, lcnt[cur] | (1 << c)); // 新建节点
            fail[now] = next[get_fail(fail[cur])][c]; // 和AC自动机一样建立fail指针，以便失配后跳转
            next[cur][c] = now;
            num[now] = num[fail[now]] + 1;
        }
        last = next[cur][c];
        cnt[last]++;
    }

    void count() {
        for (int i = p - 1; i >= 0; --i) cnt[fail[i]] += cnt[i];
        // 父亲累加儿子的cnt，因为如果fail[v]=u，则u一定是v的子回文串
    }
} pt;
```

### 后缀数组

```cpp
// 下标从1开始
// sa[i]: 排名为i的后缀位置
// rk[i]: 第i个后缀的排名
// ht[i]: LCP(sa[i], sa[i - 1])
struct SA {
    int n, m;
    vector<int> a, d, sa, rk, ht;

    void rsort() {
        vector<int> c(m + 1);
        for (int i = 1; i <= n; i++) c[rk[d[i]]]++;
        for (int i = 1; i <= m; i++) c[i] += c[i - 1];
        for (int i = n; i; i--) sa[c[rk[d[i]]]--] = d[i];
    }

    SA(const string& s) : n(s.size()), m(128), a(n + 1), d(n + 1), sa(n + 1), rk(n + 1), ht(n + 1) {
        for (int i = 1; i <= n; i++) { rk[i] = a[i] = s[i - 1]; d[i] = i; }
        rsort();
        for (int j = 1, i, k; k < n; m = k, j <<= 1) {
            for (i = n - j + 1, k = 0; i <= n; i++) d[++k] = i;
            for (i = 1; i <= n; i++) if (sa[i] > j) d[++k] = sa[i] - j;
            rsort(); swap(rk, d); rk[sa[1]] = k = 1;
            for (i = 2; i <= n; i++) {
                rk[sa[i]] = (d[sa[i]] == d[sa[i - 1]] && d[sa[i] + j] == d[sa[i - 1] + j]) ? k : ++k;
            }
        }
        int j, k = 0;
        for (int i = 1; i <= n; ht[rk[i++]] = k) {
            for (k ? k-- : k, j = sa[rk[i] - 1]; a[i + k] == a[j + k]; ++k);
        }
    }
};
```
