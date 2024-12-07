## Miscellaneous

### Sliding Window

```cpp
// [start, end)
// [l, r)
void sliding_window(int start, int end,
                    function<void(int)> add,
                    function<void(int)> del,
                    function<bool()> f,
                    function<void(int, int)> callback) {
  for (int l = start, r = start; r != end; callback(l, r))
    for (add(r++); l != r && f(); del(l++));
}
```

### Binary Search

```cpp
// [l, r]
template <class T, class F>
T min_left(T l, T r, F f) {
  while (l < r) {
    T p = l + (r - l) / 2;
    f(p) ? r = p : l = p + 1;
  }
  return l;
}

template <class T, class F>
T max_right(T l, T r, F f) {
  while (l < r) {
    T p = l + (r - l + 1) / 2;
    f(p) ? l = p : r = p - 1;
  }
  return l;
}
```

### Ternary Search

```cpp
// for real numbers
template <class F, class C = less<>>
double ternary_search(double l, double r, F f, C cmp = C()) {
  for (int i = 0; i < 75; i++) {
    double x1 = (l * 5 + r * 4) / 9;
    double x2 = (l * 4 + r * 5) / 9;
    cmp(f(x1), f(x2)) ? r = x2 : l = x1;
  }
  return l;
}

// for integers
template <class T, class F, class C = less<>>
T ternary_search(T l, T r, F f, C cmp = C()) {
  while (l < r - 2) {
    T x = l + (r - l) / 2;
    cmp(f(x), f(x + 1)) ? r = x + 1 : l = x;
  }
  auto bestval = f(l);
  T ans = l;
  for (T i = l + 1; i <= r; i++) {
    if (cmp(f(i), bestval)) bestval = f(i), ans = i;
  }
  return ans;
}
```

### Date

```cpp
// 0 ~ 6 corresponds to Monday ~ Sunday
int zeller(int y, int m, int d) {
  if (m <= 2) m += 12, y--;
  return (d + 2 * m + 3 * (m + 1) / 5 + y + y / 4 - y / 100 + y / 400) % 7;
}

// date_to_int(1, 1, 1) = 1721426
// date_to_int(2019, 10, 27) = 2458784
int date_to_int(int y, int m, int d) {
  return
  1461 * (y + 4800 + (m - 14) / 12) / 4 +
  367 * (m - 2 - (m - 14) / 12 * 12) / 12 -
  3 * ((y + 4900 + (m - 14) / 12) / 100) / 4 +
  d - 32075;
}

void int_to_date(int jd, int &y, int &m, int &d) {
  int x, n, i, j;

  x = jd + 68569;
  n = 4 * x / 146097;
  x -= (146097 * n + 3) / 4;
  i = (4000 * (x + 1)) / 1461001;
  x -= 1461 * i / 4 - 31;
  j = 80 * x / 2447;
  d = x - 2447 * j / 80;
  x = j / 11;
  m = j + 2 - 12 * x;
  y = 100 * (n - 49) + i + x;
}
```

### Subset Enumeration

```cpp
// all strict subsets
for (int t = (x - 1) & x; t; t = (t - 1) & x)

// subsets of size k
// Note: k cannot be 0
void subset(int k, int n) {
  int t = (1 << k) - 1;
  while (t < (1 << n)) {
    // do something
    int x = t & -t, y = t + x;
    t = ((t & ~y) / x >> 1) | y;
  }
}

// enumerate supersets
for (int t = (x + 1) | x; t < (1 << n); t = (t + 1) | x)
```

### Sum Over Subsets DP

```cpp
// sum over subsets
for (int i = 0; i < k; i++)
  for (int s = 0; s < (1 << k); s++)
    if ((s >> i) & 1) dp[s] += dp[s - (1 << i)];

// sum over supersets
for (int i = 0; i < k; i++)
  for (int s = 0; s < (1 << k); s++)
    if (!((s >> i) & 1)) dp[s] += dp[s + (1 << i)];
```

### Longest Increasing Subsequence

```cpp
// dp[i]: the minimum value of the last element of a non-decreasing subsequence with length i+1
template <class T>
int lis(const vector<T>& a) {
  vector<T> dp(a.size() + 1, numeric_limits<T>::max());
  T mx = dp[0];
  for (auto& x : a) *upper_bound(dp.begin(), dp.end(), x) = x;  // use lower_bound for increasing
  int ans = 0;
  while (dp[ans] != mx) ++ans;
  return ans;
}

// output actual solution
template <class T>
vector<int> lis(const vector<T>& S) {
  if (S.empty()) return {};
  vector<int> prev(S.size());
  using P = pair<T, int>;
  vector<P> res;
  for (int i = 0; i < int(S.size()); i++) {
    // use lower_bound P{S[i], 0} for increasing
    auto it = upper_bound(res.begin(), res.end(), P{S[i], INT_MAX});
    if (it == res.end()) res.emplace_back(), it = res.end() - 1;
    *it = {S[i], i};
    prev[i] = it == res.begin() ? 0 : (it - 1)->second;
  }
  int L = res.size(), cur = res.back().second;
  vector<int> ans(L);
  while (L--) ans[L] = cur, cur = prev[cur];
  return ans;
}
```

### Divide and Conquer DP

```cpp
vector<i64> dp(n), new_dp(n);

// compute new_dp[l], ... new_dp[r] (inclusive)
function<void(int, int, int, int)> compute = [&](int l, int r, int optl, int optr) {
  if (l > r) return;
  int mid = (l + r) >> 1;
  pair<i64, int> best = {LLONG_MAX, -1};
  for (int j = optl; j <= min(mid, optr); j++) {
    best = min(best, {(j ? dp[j - 1] : 0) + C(j, mid), j});
  }
  new_dp[mid] = best.first;
  int opt = best.second;
  compute(l, mid - 1, optl, opt);
  compute(mid + 1, r, opt, optr);
};

for (int i = 0; i < n; i++) {
  dp[i] = C(0, i);
}
for (int i = 1; i <= k; i++) {
  compute(0, n - 1, 0, n - 1);
  swap(dp, new_dp);
}
// return dp[n - 1];
```

### Gray Code

```cpp
int g(int n) { return n ^ (n >> 1); }

int rev_g(int g) {
  int n = 0;
  for (; g; g >>= 1) n ^= g;
  return n;
}
```

### Digit DP

```cpp
struct DigitDP {
  int n;
  vector<int> a, b;
  vector<i64> dp;

  DigitDP(i64 x, i64 y) {
    while (x) {
      a.push_back(x % 10);
      x /= 10;
    }
    while (y) {
      b.push_back(y % 10);
      y /= 10;
    }
    n = b.size();
    while (a.size() < n) a.push_back(0);
    dp.assign(n, -1);
  }

  i64 dfs(int i, bool tight_lo, bool tight_hi, bool zero) {
    if (i == -1) return 1;
    if (!tight_lo && !tight_hi && !zero && dp[i] != -1) return dp[i];
    i64 res = 0;
    int lo = tight_lo ? a[i] : 0;
    int hi = tight_hi ? b[i] : 9;
    for (int d = lo; d <= hi; d++) {
      res += dfs(i - 1, tight_lo && d == lo, tight_hi && d == hi, zero && d == 0);
    }
    return tight_lo || tight_hi || zero ? res : dp[i] = res;
  }

  i64 count() { return dfs(n - 1, true, true, true); }
};
```

### Random Set

```cpp
vector<int> randset(int l, int r, int k) {
  // assert(l <= r && k <= r - l + 1);
  unordered_map<int, int> p;
  for (int i = l; i < l + k; i++) p[i] = i;
  for (int i = l; i < l + k; i++) {
    int j = randint(i, r);
    if (!p.count(j)) p[j] = j;
    swap(p[i], p[j]);
  }
  vector<int> a(k);
  for (int i = 0; i < k; i++) a[i] = p[i + l];
  return a;
}
```

### Expression Evaluation

```py
print(input()) # Python2
print(eval(input())) # Python3
```

### Regex

```py
import re

pattern = r'hello'
text = 'hello world'
match = re.search(pattern, text) # or re.match(pattern, text)
if match:
    print('Match found:', match.group(), match.start(), match.end())
else:
    print('No match')

# identifier
r'^[a-zA-Z_]\w*$'

# email
r'^[\w\.-]+@[\w\.-]+\.\w+$'

# URL
r'^(https?|ftp)://[^\s/$.?#].[^\s]*$'
```

### Stress Test

+ *unix

```bash
#!/bin/bash
cd "$(dirname "${BASH_SOURCE[0]}")"

g++ gen.cpp -o gen -O2 -std=c++17
g++ my.cpp -o my -O2 -std=c++17
g++ std.cpp -o std -O2 -std=c++17

while true
do
    ./gen > in.txt
    ./std < in.txt > stdout.txt
    ./my < in.txt > myout.txt

    if test $? -ne 0
    then
        printf "RE\n"
        exit 0
    fi

    if diff stdout.txt myout.txt
    then
        printf "AC\n"
    else
        printf "WA\n"
        exit 0
    fi
done
```

+ Windows

```
@echo off

g++ gen.cpp -o gen.exe -O2 -std=c++17
g++ my.cpp -o my.exe -O2 -std=c++17
g++ std.cpp -o std.exe -O2 -std=c++17

:loop
    gen.exe > in.txt
    std.exe < in.txt > stdout.txt
    my.exe < in.txt > myout.txt
    if errorlevel 1 (
        echo RE
        pause
        exit
    )
    fc stdout.txt myout.txt
    if errorlevel 1 (
        echo WA
        pause
        exit
    )
goto loop
```

### Safe std::vector

```cpp
namespace std {
template<class T>
class vector_s : public vector<T> {
public:
  vector_s(size_t n = 0, const T& x = T()) : vector<T>(n, x) {}
  T& operator [] (size_t n) { return this->at(n); }
  const T& operator [] (size_t n) const { return this->at(n); }
};
}

#define vector vector_s
```

### More Hash

```cpp
template<class T1, class T2>
struct pair_hash {
  size_t operator () (const pair<T1, T2>& p) const {
    return hash<T1>()(p.first) * 19260817 + hash<T2>()(p.second);
  }
};

unordered_set<pair<int, int>, pair_hash<int, int>> st;
unordered_map<pair<int, int>, int, pair_hash<int, int>> mp;

struct custom_hash {
  static uint64_t splitmix64(uint64_t x) {
    // http://xorshift.di.unimi.it/splitmix64.c
    x += 0x9e3779b97f4a7c15;
    x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9;
    x = (x ^ (x >> 27)) * 0x94d049bb133111eb;
    return x ^ (x >> 31);
  }

  size_t operator()(uint64_t x) const {
    static const uint64_t FIXED_RANDOM = chrono::steady_clock::now().time_since_epoch().count();
    return splitmix64(x + FIXED_RANDOM);
  }
};

unordered_map<i64, int, custom_hash> safe_map;
```

### Max/Min with Assignment

```cpp
template <class T, class U> bool umax(T& a, U b) { return a < b ? a = b, 1 : 0; }
template <class T, class U> bool umin(T& a, U b) { return a > b ? a = b, 1 : 0; }
```

### Split and Join

```cpp
vector<string> split(const string& s, string sep) {
  size_t b = 0, e;
  vector<string> res;
  while ((e = s.find(sep, b)) != string::npos) {
    res.push_back(s.substr(b, e - b));
    b = e + sep.size();
  }
  res.push_back(s.substr(b));
  return res;
}

template <class T>
string join(const vector<T>& v, string sep) {
  stringstream ss;
  bool f = 0;
  for (const T& x : v) (f ? ss << sep : ss) << x, f = 1;
  return ss.str();
}
```

### Coordinate Compressor

```cpp
template <class T>
struct CoordinateCompressor {
  vector<T> v;
  CoordinateCompressor(const vector<T>& a) : v(a) {
    sort(v.begin(), v.end());
    v.erase(unique(v.begin(), v.end()), v.end());
  }
  int operator()(const T& x) const {
    return lower_bound(v.begin(), v.end(), x) - v.begin();
  }
};
```

### Merge Same Items

```cpp
template <class T>
vector<pair<T, int>> norm(vector<T>& v) {
  // sort(v.begin(), v.end());
  vector<pair<T, int>> p;
  for (int i = 0; i < (int)v.size(); i++) {
    if (i == 0 || v[i] != v[i - 1])
      p.emplace_back(v[i], 1);
    else
      p.back().second++;
  }
  return p;
}
```

### 2D Transformation

```cpp
struct Transformation {
  int n, m;
  Transformation(int n, int m) : n(n), m(m) {}
  pair<int, int> rotate_90(int x, int y) const { return {y, n - x - 1}; }
  pair<int, int> rotate_180(int x, int y) const { return {n - x - 1, m - y - 1}; }
  pair<int, int> rotate_270(int x, int y) const { return {m - y - 1, x}; }
  pair<int, int> flip_horizontal(int x, int y) const { return {x, m - y - 1}; }
  pair<int, int> flip_vertical(int x, int y) const { return {n - x - 1, y}; }
  pair<int, int> flip_diagonal(int x, int y) const { return {y, x}; }
  pair<int, int> flip_antidiagonal(int x, int y) const { return {m - y - 1, n - x - 1}; }
};
```

### Priority Queue with Erase

```cpp
template <class T, class C = less<T>>
struct heap {
  priority_queue<T, vector<T>, C> q1, q2;
  void push(const T& x) { q1.push(x); }
  void erase(const T& x) { q2.push(x); }
  T top() {
    while (q2.size() && q1.top() == q2.top()) q1.pop(), q2.pop();
    return q1.top();
  }
  void pop() {
    while (q2.size() && q1.top() == q2.top()) q1.pop(), q2.pop();
    q1.pop();
  }
  int size() const { return q1.size() - q2.size(); }
};
```

### Fractions

```cpp
template <class T>
struct Frac {
  T x, y;

  Frac(const T& p = 0, const T& q = 1) {
    T d = gcd(p, q);
    x = p / d;
    y = q / d;
    if (y < 0) {
      x = -x;
      y = -y;
    }
  }

  Frac operator+(const Frac& b) { return Frac(x * b.y + y * b.x, y * b.y); }
  Frac operator-(const Frac& b) { return Frac(x * b.y - y * b.x, y * b.y); }
  Frac operator*(const Frac& b) { return Frac(x * b.x, y * b.y); }
  Frac operator/(const Frac& b) { return Frac(x * b.y, y * b.x); }

  bool operator==(const Frac& b) { return x * b.y == y * b.x; }
  bool operator!=(const Frac& b) { return !(*this == b); }
  bool operator<(const Frac& b) { return x * b.y < y * b.x; }
  bool operator>(const Frac& b) { return x * b.y > y * b.x; }
  bool operator<=(const Frac& b) { return !(*this > b); }
  bool operator>=(const Frac& b) { return !(*this < b); }

  Frac& operator+=(const Frac& b) { *this = *this + b; return *this; }
  Frac& operator-=(const Frac& b) { *this = *this - b; return *this; }
  Frac& operator*=(const Frac& b) { *this = *this * b; return *this; }
  Frac& operator/=(const Frac& b) { *this = *this / b; return *this; }
};

template <class T>
ostream& operator<<(ostream& os, const Frac<T>& f) {
  if (f.y == 1) return os << f.x;
  else return os << f.x << '/' << f.y;
}
```

### ModInt

+ Industrial ModInt

```cpp
// tourist
template <typename T>
T inverse(T a, T m) {
  T u = 0, v = 1;
  while (a != 0) {
    T t = m / a;
    m -= t * a; swap(a, m);
    u -= t * v; swap(u, v);
  }
  assert(m == 1);
  return u;
}

template <typename T>
class Modular {
 public:
  using Type = typename decay<decltype(T::value)>::type;

  constexpr Modular() : value() {}
  template <typename U>
  Modular(const U& x) {
    value = normalize(x);
  }

  template <typename U>
  static Type normalize(const U& x) {
    Type v;
    if (-mod() <= x && x < mod()) v = static_cast<Type>(x);
    else v = static_cast<Type>(x % mod());
    if (v < 0) v += mod();
    return v;
  }

  const Type& operator()() const { return value; }
  template <typename U>
  explicit operator U() const { return static_cast<U>(value); }
  constexpr static Type mod() { return T::value; }

  Modular& operator+=(const Modular& other) { if ((value += other.value) >= mod()) value -= mod(); return *this; }
  Modular& operator-=(const Modular& other) { if ((value -= other.value) < 0) value += mod(); return *this; }
  template <typename U> Modular& operator+=(const U& other) { return *this += Modular(other); }
  template <typename U> Modular& operator-=(const U& other) { return *this -= Modular(other); }
  Modular& operator++() { return *this += 1; }
  Modular& operator--() { return *this -= 1; }
  Modular operator++(int) { Modular result(*this); *this += 1; return result; }
  Modular operator--(int) { Modular result(*this); *this -= 1; return result; }
  Modular operator-() const { return Modular(-value); }

  template <typename U = T>
  typename enable_if<is_same<typename Modular<U>::Type, int>::value, Modular>::type& operator*=(const Modular& rhs) {
#ifdef _WIN32
    uint64_t x = static_cast<int64_t>(value) * static_cast<int64_t>(rhs.value);
    uint32_t xh = static_cast<uint32_t>(x >> 32), xl = static_cast<uint32_t>(x), d, m;
    asm(
      "divl %4; \n\t"
      : "=a" (d), "=d" (m)
      : "d" (xh), "a" (xl), "r" (mod())
    );
    value = m;
#else
    value = normalize(static_cast<int64_t>(value) * static_cast<int64_t>(rhs.value));
#endif
    return *this;
  }
  template <typename U = T>
  typename enable_if<is_same<typename Modular<U>::Type, long long>::value, Modular>::type& operator*=(const Modular& rhs) {
#ifdef __SIZEOF_INT128__
    value = normalize(static_cast<__int128>(value) * static_cast<__int128>(rhs.value));
#else
    long long q = static_cast<long long>(static_cast<long double>(value) * rhs.value / mod());
    value = normalize(value * rhs.value - q * mod());
#endif
    return *this;
  }
  template <typename U = T>
  typename enable_if<!is_integral<typename Modular<U>::Type>::value, Modular>::type& operator*=(const Modular& rhs) {
    value = normalize(value * rhs.value);
    return *this;
  }

  Modular& operator/=(const Modular& other) { return *this *= Modular(inverse(other.value, mod())); }

  friend const Type& abs(const Modular& x) { return x.value; }

  template <typename U>
  friend bool operator==(const Modular<U>& lhs, const Modular<U>& rhs);

  template <typename U>
  friend bool operator<(const Modular<U>& lhs, const Modular<U>& rhs);

  template <typename V, typename U>
  friend V& operator>>(V& stream, Modular<U>& number);

 private:
  Type value;
};

template <typename T> bool operator==(const Modular<T>& lhs, const Modular<T>& rhs) { return lhs.value == rhs.value; }
template <typename T, typename U> bool operator==(const Modular<T>& lhs, U rhs) { return lhs == Modular<T>(rhs); }
template <typename T, typename U> bool operator==(U lhs, const Modular<T>& rhs) { return Modular<T>(lhs) == rhs; }

template <typename T> bool operator!=(const Modular<T>& lhs, const Modular<T>& rhs) { return !(lhs == rhs); }
template <typename T, typename U> bool operator!=(const Modular<T>& lhs, U rhs) { return !(lhs == rhs); }
template <typename T, typename U> bool operator!=(U lhs, const Modular<T>& rhs) { return !(lhs == rhs); }

template <typename T> bool operator<(const Modular<T>& lhs, const Modular<T>& rhs) { return lhs.value < rhs.value; }

template <typename T> Modular<T> operator+(const Modular<T>& lhs, const Modular<T>& rhs) { return Modular<T>(lhs) += rhs; }
template <typename T, typename U> Modular<T> operator+(const Modular<T>& lhs, U rhs) { return Modular<T>(lhs) += rhs; }
template <typename T, typename U> Modular<T> operator+(U lhs, const Modular<T>& rhs) { return Modular<T>(lhs) += rhs; }

template <typename T> Modular<T> operator-(const Modular<T>& lhs, const Modular<T>& rhs) { return Modular<T>(lhs) -= rhs; }
template <typename T, typename U> Modular<T> operator-(const Modular<T>& lhs, U rhs) { return Modular<T>(lhs) -= rhs; }
template <typename T, typename U> Modular<T> operator-(U lhs, const Modular<T>& rhs) { return Modular<T>(lhs) -= rhs; }

template <typename T> Modular<T> operator*(const Modular<T>& lhs, const Modular<T>& rhs) { return Modular<T>(lhs) *= rhs; }
template <typename T, typename U> Modular<T> operator*(const Modular<T>& lhs, U rhs) { return Modular<T>(lhs) *= rhs; }
template <typename T, typename U> Modular<T> operator*(U lhs, const Modular<T>& rhs) { return Modular<T>(lhs) *= rhs; }

template <typename T> Modular<T> operator/(const Modular<T>& lhs, const Modular<T>& rhs) { return Modular<T>(lhs) /= rhs; }
template <typename T, typename U> Modular<T> operator/(const Modular<T>& lhs, U rhs) { return Modular<T>(lhs) /= rhs; }
template <typename T, typename U> Modular<T> operator/(U lhs, const Modular<T>& rhs) { return Modular<T>(lhs) /= rhs; }

template<typename T, typename U>
Modular<T> power(const Modular<T>& a, const U& b) {
  assert(b >= 0);
  Modular<T> x = a, res = 1;
  U p = b;
  while (p > 0) {
    if (p & 1) res *= x;
    x *= x;
    p >>= 1;
  }
  return res;
}

template <typename T>
bool IsZero(const Modular<T>& number) {
  return number() == 0;
}

template <typename T>
string to_string(const Modular<T>& number) {
  return to_string(number());
}

// U == std::ostream? but done this way because of fastoutput
template <typename U, typename T>
U& operator<<(U& stream, const Modular<T>& number) {
  return stream << number();
}

// U == std::istream? but done this way because of fastinput
template <typename U, typename T>
U& operator>>(U& stream, Modular<T>& number) {
  typename common_type<typename Modular<T>::Type, long long>::type x;
  stream >> x;
  number.value = Modular<T>::normalize(x);
  return stream;
}

/* using ModType = int;

struct VarMod { static ModType value; };
ModType VarMod::value;
ModType& md = VarMod::value;
using Mint = Modular<VarMod>; */

constexpr int md = (int) 1e9 + 7;
using Mint = Modular<std::integral_constant<decay<decltype(md)>::type, md>>;

/* const int N = 2e5 + 10;
Mint fac[N], ifac[N];

void init_inv() {
  fac[0] = 1;
  for (int i = 1; i < N; i++) {
    fac[i] = fac[i - 1] * i;
  }
  ifac[N - 1] = 1 / fac[N - 1];
  for (int i = N - 2; i >= 0; i--) {
    ifac[i] = ifac[i + 1] * (i + 1);
  }
}

Mint C(int n, int m) {
  if (n < m || m < 0) return 0;
  return fac[n] * ifac[m] * ifac[n - m];
}

Mint H(int n, int m) { return C(n + m - 1, m); } */
```

### BigInt

```cpp
// wxh
class BigInt {
#define w size()

  static constexpr int base = 1000000000;
  static constexpr int base_digits = 9;

  using vi = vector<int>;
  using vll = vector<long long>;

  vi z;
  int f;

  void trim() {
    while (!z.empty() && z.back() == 0) {
      z.pop_back();
    }
    if (z.empty()) {
      f = 1;
    }
  }

  void read(const string& s) {
    f = 1;
    z.clear();
    int pos = 0;
    while (pos < (int)s.w && (s[pos] == '-' || s[pos] == '+')) {
      if (s[pos] == '-') {
        f = -f;
      }
      ++pos;
    }
    for (int i = s.w - 1; i >= pos; i -= base_digits) {
      int x = 0;
      for (int j = max(pos, i - base_digits + 1); j <= i; j++) {
        x = x * 10 + s[j] - '0';
      }
      z.push_back(x);
    }
    trim();
  }

  static vi convert_base(const vi& a, int old_digits, int new_digits) {
    vll p(max(old_digits, new_digits) + 1);
    p[0] = 1;
    for (int i = 1; i < (int)p.w; i++) {
      p[i] = p[i - 1] * 10;
    }
    vi res;
    long long cur = 0;
    int cur_digits = 0;
    for (int i = 0; i < (int)a.w; i++) {
      cur += a[i] * p[cur_digits];
      cur_digits += old_digits;
      while (cur_digits >= new_digits) {
        res.push_back(cur % p[new_digits]);
        cur /= p[new_digits];
        cur_digits -= new_digits;
      }
    }
    res.push_back(cur);
    while (!res.empty() && res.back() == 0) {
      res.pop_back();
    }
    return res;
  }

  static vll karatsuba(const vll& a, const vll& b) {
    int n = a.w;
    vll res(n + n);
    if (n <= 32) {
      for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
          res[i + j] += a[i] * b[j];
        }
      }
      return res;
    }
    int k = n >> 1;
    vll a1(a.begin(), a.begin() + k);
    vll a2(a.begin() + k, a.end());
    vll b1(b.begin(), b.begin() + k);
    vll b2(b.begin() + k, b.end());
    vll a1b1 = karatsuba(a1, b1);
    vll a2b2 = karatsuba(a2, b2);
    for (int i = 0; i < k; i++) {
      a2[i] += a1[i];
    }
    for (int i = 0; i < k; i++) {
      b2[i] += b1[i];
    }
    vll r = karatsuba(a2, b2);
    for (int i = 0; i < (int)a1b1.w; i++) {
      r[i] -= a1b1[i];
    }
    for (int i = 0; i < (int)a2b2.w; i++) {
      r[i] -= a2b2[i];
    }
    for (int i = 0; i < (int)r.w; i++) {
      res[i + k] += r[i];
    }
    for (int i = 0; i < (int)a1b1.w; i++) {
      res[i] += a1b1[i];
    }
    for (int i = 0; i < (int)a2b2.w; i++) {
      res[i + n] += a2b2[i];
    }
    return res;
  }

public:
  BigInt() : f(1) {}
  BigInt(long long v) { *this = v; }
  BigInt(const string& s) { read(s); }

  void operator=(const BigInt& v) {
    f = v.f;
    z = v.z;
  }

  void operator=(long long v) {
    f = 1;
    if (v < 0) {
      f = -1, v = -v;
    }
    z.clear();
    for (; v > 0; v = v / base) {
      z.push_back(v % base);
    }
  }

  BigInt operator+(const BigInt& v) const {
    if (f == v.f) {
      BigInt res = v;
      for (int i = 0, carry = 0; i < (int)max(z.w, v.z.w) || carry; ++i) {
        if (i == (int)res.z.w) {
          res.z.push_back(0);
        }
        res.z[i] += carry + (i < (int)z.w ? z[i] : 0);
        carry = res.z[i] >= base;
        if (carry) {
          res.z[i] -= base;
        }
      }
      return res;
    } else {
      return *this - (-v);
    }
  }

  BigInt operator-(const BigInt& v) const {
    if (f == v.f) {
      if (abs() >= v.abs()) {
        BigInt res = *this;
        for (int i = 0, carry = 0; i < (int)v.z.w || carry; ++i) {
          res.z[i] -= carry + (i < (int)v.z.w ? v.z[i] : 0);
          carry = res.z[i] < 0;
          if (carry) {
            res.z[i] += base;
          }
        }
        res.trim();
        return res;
      } else {
        return -(v - *this);
      }
    } else {
      return *this + (-v);
    }
  }

  void operator*=(int v) {
    if (v < 0) {
      f = -f, v = -v;
    }
    for (int i = 0, carry = 0; i < (int)z.w || carry; ++i) {
      if (i == (int)z.w) {
        z.push_back(0);
      }
      long long cur = (long long)z[i] * v + carry;
      carry = cur / base;
      z[i] = cur % base;
      // asm("divl %%ecx" : "=a"(carry), "=d"(a[i]) : "A"(cur), "c"(base));
    }
    trim();
  }

  BigInt operator*(int v) const {
    BigInt res = *this;
    res *= v;
    return res;
  }

  friend pair<BigInt, BigInt> divmod(const BigInt& a1, const BigInt& b1) {
    int norm = base / (b1.z.back() + 1);
    BigInt a = a1.abs() * norm;
    BigInt b = b1.abs() * norm;
    BigInt q, r;
    q.z.resize(a.z.w);
    for (int i = a.z.w - 1; i >= 0; i--) {
      r *= base;
      r += a.z[i];
      int s1 = b.z.w < r.z.w ? r.z[b.z.w] : 0;
      int s2 = b.z.w - 1 < r.z.w ? r.z[b.z.w - 1] : 0;
      int d = ((long long)s1 * base + s2) / b.z.back();
      r -= b * d;
      while (r < 0) {
        r += b, --d;
      }
      q.z[i] = d;
    }
    q.f = a1.f * b1.f;
    r.f = a1.f;
    q.trim();
    r.trim();
    return make_pair(q, r / norm);
  }

  friend BigInt sqrt(const BigInt& a1) {
    BigInt a = a1;
    while (a.z.empty() || (int)a.z.w % 2 == 1) {
      a.z.push_back(0);
    }
    int n = a.z.w;
    int firstDigit = sqrt((long long)a.z[n - 1] * base + a.z[n - 2]);
    int norm = base / (firstDigit + 1);
    a *= norm;
    a *= norm;
    while (a.z.empty() || (int)a.z.w % 2 == 1) {
      a.z.push_back(0);
    }
    BigInt r = (long long)a.z[n - 1] * base + a.z[n - 2];
    firstDigit = sqrt((long long)a.z[n - 1] * base + a.z[n - 2]);
    int q = firstDigit;
    BigInt res;
    for (int j = n / 2 - 1; j >= 0; j--) {
      for (;; --q) {
        BigInt r1 = (r - (res * 2 * base + q) * q) * base * base +
              (j > 0 ? (long long)a.z[2 * j - 1] * base + a.z[2 * j - 2] : 0);
        if (r1 >= 0) {
          r = r1;
          break;
        }
      }
      res *= base;
      res += q;
      if (j > 0) {
        int d1 = res.z.w + 2 < r.z.w ? r.z[res.z.w + 2] : 0;
        int d2 = res.z.w + 1 < r.z.w ? r.z[res.z.w + 1] : 0;
        int d3 = res.z.w < r.z.w ? r.z[res.z.w] : 0;
        q = ((long long)d1 * base * base + (long long)d2 * base + d3) / (firstDigit * 2);
      }
    }
    res.trim();
    return res / norm;
  }

  BigInt operator/(const BigInt& v) const { return divmod(*this, v).first; }
  BigInt operator%(const BigInt& v) const { return divmod(*this, v).second; }

  void operator/=(int v) {
    if (v < 0) {
      f = -f, v = -v;
    }
    for (int i = z.w - 1, rem = 0; i >= 0; --i) {
      long long cur = z[i] + (long long)rem * base;
      z[i] = cur / v;
      rem = cur % v;
    }
    trim();
  }

  BigInt operator/(int v) const {
    BigInt res = *this;
    res /= v;
    return res;
  }

  int operator%(int v) const {
    if (v < 0) {
      v = -v;
    }
    int m = 0;
    for (int i = z.w - 1; i >= 0; --i) {
      m = ((long long)m * base + z[i]) % v;
    }
    return m * f;
  }

  void operator+=(const BigInt& v) { *this = *this + v; }
  void operator-=(const BigInt& v) { *this = *this - v; }
  void operator*=(const BigInt& v) { *this = *this * v; }
  void operator/=(const BigInt& v) { *this = *this / v; }

  bool operator<(const BigInt& v) const {
    if (f != v.f) {
      return f < v.f;
    }
    if (z.w != v.z.w) {
      return z.w * f < v.z.w * v.f;
    }
    for (int i = z.w - 1; i >= 0; i--) {
      if (z[i] != v.z[i]) {
        return z[i] * f < v.z[i] * f;
      }
    }
    return false;
  }

  bool operator>(const BigInt& v) const { return v < *this; }
  bool operator<=(const BigInt& v) const { return !(v < *this); }
  bool operator>=(const BigInt& v) const { return !(*this < v); }
  bool operator==(const BigInt& v) const { return !(*this < v) && !(v < *this); }
  bool operator!=(const BigInt& v) const { return *this < v || v < *this; }

  bool is_zero() const { return z.empty() || ((int)z.w == 1 && !z[0]); }

  BigInt operator-() const {
    BigInt res = *this;
    res.f = -f;
    return res;
  }

  BigInt abs() const {
    BigInt res = *this;
    res.f *= res.f;
    return res;
  }

  long long long_value() const {
    long long res = 0;
    for (int i = z.w - 1; i >= 0; i--) {
      res = res * base + z[i];
    }
    return res * f;
  }

  friend BigInt gcd(const BigInt& a, const BigInt& b) { return b.is_zero() ? a : gcd(b, a % b); }
  friend BigInt lcm(const BigInt& a, const BigInt& b) { return a / gcd(a, b) * b; }

  friend istream& operator>>(istream& is, BigInt& v) {
    string s;
    is >> s;
    v.read(s);
    return is;
  }

  friend ostream& operator<<(ostream& os, const BigInt& v) {
    if (v.f == -1) {
      os << '-';
    }
    os << (v.z.empty() ? 0 : v.z.back());
    for (int i = v.z.w - 2; i >= 0; --i) {
      os << setw(base_digits) << setfill('0') << v.z[i];
    }
    return os;
  }

  BigInt operator*(const BigInt& v) const {
    vi a6 = convert_base(this->z, base_digits, 6);
    vi b6 = convert_base(v.z, base_digits, 6);
    vll a(a6.begin(), a6.end());
    vll b(b6.begin(), b6.end());
    while (a.w < b.w) {
      a.push_back(0);
    }
    while (b.w < a.w) {
      b.push_back(0);
    }
    while (a.w & (a.w - 1)) {
      a.push_back(0);
      b.push_back(0);
    }
    vll c = karatsuba(a, b);
    BigInt res;
    res.f = f * v.f;
    for (int i = 0, carry = 0; i < (int)c.w; i++) {
      long long cur = c[i] + carry;
      res.z.push_back(cur % 1000000);
      carry = cur / 1000000;
    }
    res.z = convert_base(res.z, 6, base_digits);
    res.trim();
    return res;
  }

#undef w
};
```


### Java

+ Main

```java
import java.io.*;
import java.util.*;

public class Main {
  public static void main(String[] args) {
    Scanner in = new Scanner(System.in);
    PrintStream out = System.out;

  }
}
```

+ Petr's ultimate fast input

```java
public class Main {
  public static void main(String[] args) {
    InputStream inputStream = System.in;
    OutputStream outputStream = System.out;
    InputReader in = new InputReader(inputStream);
    PrintWriter out = new PrintWriter(outputStream);

    out.close();
  }

  static class InputReader {
    public BufferedReader reader;
    public StringTokenizer tokenizer;

    public InputReader(InputStream stream) {
      reader = new BufferedReader(new InputStreamReader(stream), 32768);
      tokenizer = null;
    }

    public String next() {
      while (tokenizer == null || !tokenizer.hasMoreTokens()) {
        try {
          tokenizer = new StringTokenizer(reader.readLine());
        } catch (IOException e) {
          throw new RuntimeException(e);
        }
      }
      return tokenizer.nextToken();
    }

    public int nextInt() {
      return Integer.parseInt(next());
    }
  }
}
```

+ BigInteger

```java
import java.math.BigInteger;

BigInteger.ZERO
BigInteger.ONE
BigInteger.TWO // since Java 9
BigInteger.TEN
BigInteger.valueOf(2)

BigInteger abs()
BigInteger negate() // -this

BigInteger add​(BigInteger x)
BigInteger subtract​(BigInteger x)
BigInteger multiply​(BigInteger x)
BigInteger divide​(BigInteger x)

BigInteger pow​(int exp)
BigInteger sqrt() // since Java 9

BigInteger mod​(BigInteger m)
BigInteger modPow​(BigInteger exp, BigInteger m)
BigInteger modInverse​(BigInteger m)

boolean isProbablePrime​(int certainty) // probability: 1 - (1/2) ^ (certainty)

BigInteger gcd​(BigInteger x)

BigInteger not() // ~this
BigInteger and​(BigInteger x)
BigInteger or​(BigInteger x)
BigInteger xor​(BigInteger x)
BigInteger shiftLeft​(int n)
BigInteger shiftRight​(int n)

int compareTo​(BigInteger x) // -1, 0, 1
BigInteger max​(BigInteger x)
BigInteger min​(BigInteger x)

int intValue()
long longValue()
String toString()

public static BigInteger getsqrt(BigInteger n) {
  if (n.compareTo(BigInteger.ZERO) <= 0) return n;
  BigInteger x, xx, txx;
  xx = x = BigInteger.ZERO;
  for (int t = n.bitLength() / 2; t >= 0; t--) {
    txx = xx.add(x.shiftLeft(t + 1)).add(BigInteger.ONE.shiftLeft(t + t));
    if (txx.compareTo(n) <= 0) {
      x = x.add(BigInteger.ONE.shiftLeft(t));
      xx = txx;
    }
  }
  return x;
}
```

+ DecimalFormat

```java
import java.text.DecimalFormat;

DecimalFormat fmt;

// String s = fmt.format(...)

// round to at most 2 digits, leave of digits if not needed
fmt = new DecimalFormat("#.##");
// 12345.6789 -> "12345.68"
// 12345.0 -> "12345"
// 0.0 -> "0"
// 0.01 -> ".1"

// round to precisely 2 digits
fmt = new DecimalFormat("#.00");
// 12345.6789 -> "12345.68"
// 12345.0 -> "12345.00"
// 0.0 -> ".00"

// round to precisely 2 digits, force leading zero
fmt = new DecimalFormat("0.00");
// 12345.6789 -> "12345.68"
// 12345.0 -> "12345.00"
// 0.0 -> "0.00"

// round to precisely 2 digits, force leading zeros
fmt = new DecimalFormat("000000000.00");
// 12345.6789 -> "000012345.68"
// 12345.0 -> "000012345.00"
// 0.0 -> "000000000.00"
```
