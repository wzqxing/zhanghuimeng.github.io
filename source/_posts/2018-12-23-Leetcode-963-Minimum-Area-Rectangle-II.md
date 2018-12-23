---
title: Leetcode 963. Minimum Area Rectangle II
urlname: leetcode-963-minimum-area-rectangle-ii
toc: true
date: 2018-12-23 19:17:41
updated: 2018-12-24 03:14:00
tags: [Leetcode, Leetcode Contest, alg:Math, alg:Geometry]
---

题目来源：

标记难度：Medium

提交次数：1/1

代码效率：

* hash向量：120ms
* 枚举三个点：32ms

## 题意

给定平面上若干个（<=400个）整点，问能够组成的面积最小的**矩形**的面积是多少。

## 分析

我的做法听起来有点奇怪：用`O(N^2)`的时间把每两个点组成的向量算出来，然后对于相同的向量（也就是说它们对应的4个点可以组成平行四边形），判断对应的4个点能否组成矩形。我感觉时间复杂度大概是`O(N^3)`，加上`map`应该算是`O(log(N))`。考试后我才开始思考，这道题和之前的[Leetcode 939. Minimum Area Rectangle](/post/leetcode-939-minimum-area-rectangle)好像没什么区别，为什么我没有用那种直接`O(N^3)`枚举三个点，再直接算出第四个点的方法呢……

以及，这道题我好像因为不存在矩形时输出0和叉积结果有可能为负数错了好几次……

---

比较简单的一种方法：

* 枚举矩形的三个点（注意顺序），判断组成的两条边是否相互垂直
* 用哈希表查第四个点是否存在，如果存在则计算面积

这种方法是`O(N^3)`的。

另一种时间复杂度更小的方法是，枚举每两个点连线中点和长度，然后检查所有中点和长度相同的点对能否组成矩形。据说每种点对的数量是`O(log(N))`的，所以我感觉这种方法的复杂度应该是`O(N^2*log(N)^2)`？也可能我想错了。

也是好久没正经写过计算几何的题了……（说得就好像你正经写过一样）

## 代码

### 奇怪的方法

```cpp
class Solution {
private:
    struct Point {
        int x, y;
        
        Point(int _x, int _y) {
            x = _x;
            y = _y;
        }
        
        friend bool operator < (const Point& p1, const Point& p2) {
            if (p1.x != p2.x) return p1.x < p2.x;
            return p1.y < p2.y;
        }
        
        friend bool operator == (const Point& p1, const Point& p2) {
            return p1.x == p2.x && p1.y == p2.y;
        }
        
        friend Point operator - (const Point& p1, const Point& p2) {
            return Point(p1.x - p2.x, p1.y - p2.y);
        }
        
        friend int cross(const Point& p1, const Point& p2) {
            return p1.x * p2.y - p2.x * p1.y;
        }
        
        friend int dot(const Point& p1, const Point& p2) {
            return p1.x * p2.x + p1.y * p2.y;
        }
        
        void print() {
            cout << '(' << x << ',' << y << ')';
        }
    };
    
    typedef Point Vector;
    
public:
    double minAreaFreeRect(vector<vector<int>>& points) {
        vector<Point> a;
        int N = points.size();
        for (int i = 0; i < N; i++) {
            a.emplace_back(points[i][0], points[i][1]);
        }
        sort(a.begin(), a.end());
        
        int ans = 0;
        map<Vector, vector<int>> vmap;
        for (int i = 0; i < N; i++) {
            for (int j = i + 1; j < N; j++) {
                Vector v = a[j] - a[i];
                for (int k: vmap[v]) {
                    // a[i].print(); a[j].print(); a[k].print(); cout << endl;
                    Vector v2 = a[k] - a[i];
                    if (dot(v, v2) != 0) continue;
                    int area = cross(v, v2);
                    if (area <= 0) continue;
                    ans = ans == 0 ? area : min(ans, area);
                }
                vmap[v].push_back(i);
            }
        }
        return ans;
    }
};
```

### 枚举三个点的方法

```cpp
class Solution {
private:
    struct Point {
        int x, y;
        
        Point(int _x, int _y) {
            x = _x;
            y = _y;
        }
        
        friend bool operator < (const Point& p1, const Point& p2) {
            if (p1.x != p2.x) return p1.x < p2.x;
            return p1.y < p2.y;
        }
        
        friend bool operator == (const Point& p1, const Point& p2) {
            return p1.x == p2.x && p1.y == p2.y;
        }
        
        friend Point operator + (const Point& p1, const Point& p2) {
            return Point(p1.x + p2.x, p1.y + p2.y);
        }
        
        friend Point operator - (const Point& p1, const Point& p2) {
            return Point(p1.x - p2.x, p1.y - p2.y);
        }
        
        friend int cross(const Point& p1, const Point& p2) {
            return p1.x * p2.y - p2.x * p1.y;
        }
        
        friend int dot(const Point& p1, const Point& p2) {
            return p1.x * p2.x + p1.y * p2.y;
        }
    };
    
public:
    double minAreaFreeRect(vector<vector<int>>& points) {
        set<Point> s;  // 不能用unordered_set……
        vector<Point> p;
        int N = points.size();
        for (int i = 0; i < N; i++) {
            p.emplace_back(points[i][0], points[i][1]);
            s.insert(p.back());
        }
        int ans = -1;
        for (int i = 0; i < N; i++)
            for (int j = 0; j < N; j++) {
                if (i == j) continue;
                for (int k = 0; k < N; k++) {
                    if (i == k || j == k) continue;
                    if (dot(p[i] - p[j], p[k] - p[j]) != 0) continue;
                    Point p4 = p[k] + p[i] - p[j];
                    if (s.find(p4) != s.end()) {
                        int area = abs(cross(p[i] - p[j], p[k] - p[j]));
                        ans = ans == -1 ? area : min(ans, area);
                    }
                }
            }
        return ans == -1 ? 0 : ans;
    }
};
```
