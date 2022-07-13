# [Trang chủ](https://ppap-1264589.github.io/interesting-solution)

## Ý nghĩa

Tăng tốc độ thực hiện các thao tác update, query trên các đoạn dài của một cây cho trước

Kĩ thuật thực hiện HLD giống với Segment Tree. 

Trên thực tế, HLD chính là "duỗi" cây ra thành một dãy, và thao tác update hay query trên các đoạn con của dãy đó cũng chính là các thao tác trên các chuỗi dài của cây

## Cấu trúc dữ liệu và giải thuật: 

[cp algorithms](https://cp-algorithms.com/graph/hld.html)

## Bài toán cụ thể

Cho một cây n đỉnh, có gốc là R người ta có thể thực hiện một lượng thao tác q trên cây:
  1. tăng tất cả giá trị các nút là con của nút x lên d đơn vị (U x d)
  2. gán tất cả giá trị các nút là tổ tiên của nút x bằng d (F x d)
  3. hỏi tổng giá trị của tất cả các nút từ đỉnh x đến đỉnh y là bao nhiêu ? (V x y)
  
[LIGHTWORLD-problem.pdf](https://github.com/ppap-1264589/HLD/files/9101327/LIGHTWORLD-problem.pdf)

Bạn có thể download bộ test tại [đây](https://drive.google.com/drive/folders/1q5CY566BOgOCaLpkqnRd0R304l-cRc5f?usp=sharing)

## Code: [LIGHTWOLD.cpp](https://ideone.com/TJJdWs)

```c++
#include <bits/stdc++.h>
#define up(i,a,b) for (int i = (int)a; i <= (int)b; i++)
#define ep emplace_back
#define DOMINATE 0
#define ADD 1
using namespace std;

const int maxn = 1e5 + 10;
static int Tsize[maxn], par[maxn];
static int heavy[maxn];
static int W[maxn];
vector<int> a[maxn];
int n,R,Q;

void FileStream(){
    ios_base::sync_with_stdio(false);
    cin.tie(0);
    #define Task "LIGHTWORLD"
    if (fopen(Task".inp", "r")){
        freopen(Task".inp", "r", stdin);
        freopen(Task".out", "w", stdout);
    }
}

void DFS(int& u){
    int id_max = -1;
    Tsize[u] = 1;
    heavy[u] = -1;
    up(i,0,a[u].size()-1){
        int& v = a[u][i];
        if (v == par[u]) continue;
        par[v] = u;
        DFS(v);
        Tsize[u] += Tsize[v];
        if (id_max == -1 || (Tsize[v] > Tsize[a[u][id_max]])){
            id_max = i;
            heavy[u] = v;
        }
    }
    if (id_max != -1) swap(a[u][0], a[u][id_max]);
}

static int HChain[maxn], DChain[maxn];
static int in[maxn], out[maxn], node[maxn];
static int thld;
void HLD(int& u){
    in[u] = ++thld;
    node[thld] = u;
    for (int& v : a[u]){
        if (v == par[u]) continue;
        if (v == heavy[u]){
            HChain[v] = HChain[u];
            DChain[v] = DChain[u];
        }
        else {
            HChain[v] = v;
            DChain[v] = DChain[u] + 1;
        }
        HLD(v);
    }
    out[u] = thld;
}

static struct NODE{
    long long sum, lazyset, lazysum;
    int len;
    void init(){
        sum = lazyset = lazysum = 0ll;
        len = 1;
    }
    void Dominate(long long val){
        sum = 1ll*val*len;
        lazyset = val;
        lazysum = 0ll;
    }
    void Add(long long val){
        sum += 1ll*val*len;
        lazysum += val;
    }
} T[maxn << 2];
static struct Segment_Tree{
private:
    void push_down(int nod){
        if (T[nod].lazyset != 0){
            long long& SETT = T[nod].lazyset;
            T[nod*2].Dominate(SETT);
            T[nod*2+1].Dominate(SETT);
            SETT = 0;
        }
        long long& LAZZ = T[nod].lazysum;
        T[nod*2].Add(LAZZ);
        T[nod*2+1].Add(LAZZ);
        LAZZ = 0;
    }

    void Modify(int nod, int l, int r, int u, int v, long long val, bool type){
        if (l > v || r < u) return;
        if (l >= u && r <= v){
            if (type == DOMINATE) T[nod].Dominate(val);
            if (type == ADD) T[nod].Add(val);
            return;
        }
        push_down(nod);
        int mid = (l+r) >> 1;
        Modify(nod*2, l, mid, u, v, val, type);
        Modify(nod*2+1, mid+1, r, u, v, val, type);
        T[nod].sum = T[nod*2].sum + T[nod*2+1].sum;
    }

    long long Get(int nod, int l, int r, int u, int v){
        if (l > v || r < u) return 0ll;
        if (l >= u && r <= v) return T[nod].sum;
        push_down(nod);
        int mid = (l+r) >> 1;
        long long L = Get(nod*2, l, mid, u, v);
        long long R = Get(nod*2+1, mid+1, r, u, v);
        return L+R;
    }

public:
    void build(int nod, int l, int r){
        if (l == r){
            T[nod].init();
            T[nod].sum = 1ll*W[node[l]];
            return;
        }
        int mid = (l+r) >> 1;
        build(nod*2, l, mid);
        build(nod*2+1, mid+1, r);
        T[nod].sum = T[nod*2].sum + T[nod*2+1].sum;
        T[nod].len = T[nod*2].len + T[nod*2+1].len;
    }
    void Fix(int l, int r, long long val){
        Modify(1, 1, n, l, r, val, DOMINATE);
    }
    void Upgrade(int l, int r, long long val){
        Modify(1, 1, n, l, r, val, ADD);
    }
    long long Get(int l, int r){
        return Get(1, 1, n, l, r);
    }
} Tree;

void real_Fix(int x, long long d){
    Tree.Fix(in[x], out[x], d);
}

void real_Upgrade(int x, long long d){
    while (x != 0){
        Tree.Upgrade(in[HChain[x]], in[x], d);
        x = par[HChain[x]];
    }
}

long long real_Get(int x, int y){
    long long res = 0;
    while (HChain[x] != HChain[y]){
        if (DChain[x] < DChain[y]) swap(x, y);
        res += Tree.Get(in[HChain[x]], in[x]);
        x = par[HChain[x]];
    }
    if (in[x] > in[y]) swap(x, y);
    res += Tree.Get(in[x], in[y]);
    return res;
}

void query(){
    cin >> Q;
    while (Q--){
        int x,y;
        long long d;
        char c;
        cin >> c;
        if (c == 'F') {
            cin >> x >> d;
            real_Fix(x, d);
        }
        else if (c == 'U'){
            cin >> x >> d;
            real_Upgrade(x, d);
        }
        else {
            cin >> x >> y;
            cout << real_Get(x, y) << "\n";
        }
    }
}

void input(){
    cin >> n >> R;
    up(i,1,n) cin >> W[i];
    up(i,1,n-1){
        int u,v;
        cin >> u >> v;
        a[u].ep(v);
        a[v].ep(u);
    }
}

void Process(){
    DFS(R);
    HChain[R] = R;
    DChain[R] = 1;
    HLD(R);
    Tree.build(1, 1, n);
    query();
}

signed main(){
    FileStream();
    input();
    Process();
}
```

## Chú ý

Trong phòng thi được khuyến cáo là không nên code HLD kể cả có nghĩ ra thuật chuẩn (trừ khi bạn code HLD rất khủng) do HLD khá dài và một khi lao vào debug sẽ mất khá nhiều thời gian

Việc sinh test cho các bài HLD cũng phải cẩn thận. Cây được sinh ra phải đảm bảo có nhiều chuỗi có độ dài khoảng 1e4 hoặc 3e4, nếu không cây sẽ mất tính "chuỗi dài" (tức là thay vì cây dài ra thì cây là "rộng" ra thành các chuỗi khác). Một khi mất tính "chuỗi dài" thì nhiều khả năng code trâu cũng AC được :)
