---
comment: true
---

## leetcode

### 232 两个栈实现队列
push 的时候好说，在 pop 的时候没有必要全倒腾，只有在输出栈为空的时候再倒腾就行了

??? "solve"

    ```cpp

    class MyQueue {
    private:
        stack<int> inSt;
        stack<int> outSt;
    public:
        MyQueue() {
        }

        void push(int x) {
            inSt.push(x);
        }

        int pop() {
            if (outSt.empty()) {
                while (!inSt.empty()) {
                    outSt.push(inSt.top());
                    inSt.pop();
                }
            }
            int x = outSt.top();
            outSt.pop();
            return x;
        }

        int peek() {
            if (outSt.empty()) {
                while (!inSt.empty()) {
                    outSt.push(inSt.top());
                    inSt.pop();
                }
            }
            return outSt.top();
        }

        bool empty() {
            return inSt.empty() && outSt.empty();
        }
    };
    ```

### 第K大的数

[链接](https://leetcode.cn/problems/kth-largest-element-in-an-array/description/)

- 想要达到 $O(n)$ 时间，就得从快排变形。
- 第K大的数正好是下标为size-k
- 一次快排相当于把一个数放到对应位置，那就找哪一次放好了的下标正好是要求的
> 没做出来时痛苦万分，~~~抄完了~~~ 学会了之后觉得就应该这么写😥

??? slove
    ```C++
    class Solution {
    public:
    int findKthLargest(vector<int> &nums, int k) {
        return quicksort(nums,0,nums.size()-1,nums.size()-k);
    }

    int quicksort(vector<int> &nums, int l, int r, int k) {
        if (l == r) return nums[k];
        int i = l - 1, j = r + 1, mid = nums[l + r >> 1];
        while (i < j) {
            do i++; while (nums[i] < mid);
            do j--; while (nums[j] > mid);
            if (i < j) swap(nums[i], nums[j]);
        }
        if (k <= j) return quicksort(nums, l, j, k);
        else return quicksort(nums, j + 1, r, k);
    }
    };
    ```

从这个题谈开，假如要找[第K小的数](https://www.acwing.com/problem/content/description/788/)可以说每次快排结束能获得左边≤a[j]的结果，右边大于等于a[j+1]的结果，问题在于是怎么判断下一次的区间（我个人觉得看成下标好理解）

1. k看成长度，比较到lo的距离
```cpp
if (j - lo + 1 >= k)
    return wqsort(a, lo, j, k);
else
    return wqsort(a, j + 1, hi, k - (j + 1 - lo));
```
2. k看成下标
```cpp
if (k<=j)
    return wqsort(a, lo, j, k);
else
    return wqsort(a, j + 1, hi, k);
```

=== "下标"

    ```cpp
    #include <iostream>

    using namespace std;
    const int N = 100010;

    int wqsort(int a[], int lo, int hi, int k) {
        if (lo >= hi)
            return a[k];//a[lo] also OK

        int i = lo - 1, j = hi + 1, mid = a[lo + hi >> 1];
        while (i < j) {
            do
                i++;
            while (a[i] < mid);
            do
                j--;
            while (a[j] > mid);
            if (i < j)
                swap(a[i], a[j]);
        }
        if (k <= j)
            return wqsort(a, lo, j, k);
        else
            return wqsort(a, j + 1, hi, k);
    }

    int main() {
        int n;
        cin >> n;
        int k;
        cin >> k;
        int a[N];
        for (int i = 0; i < n; i++)
            cin >> a[i];
        cout << wqsort(a, 0, n - 1, k - 1);//转成下标
        return 0;
    }
    ```

=== "长度"

    ```cpp
    #include <iostream>

    using namespace std;
    const int N = 100010;

    int wqsort(int a[], int lo, int hi, int k) {
        if (lo >= hi)
            return a[lo];

        int i = lo - 1, j = hi + 1, mid = a[lo + hi >> 1];
        while (i < j) {
            do
                i++;
            while (a[i] < mid);
            do
                j--;
            while (a[j] > mid);
            if (i < j)
                swap(a[i], a[j]);
        }
        if (j - lo + 1 >= k)
            return wqsort(a, lo, j, k);
        else
            return wqsort(a, j + 1, hi, k - (j + 1 - lo));
    }

    int main() {
        int n;
        cin >> n;
        int k;
        cin >> k;
        int a[N];
        for (int i = 0; i < n; i++)
            cin >> a[i];
        cout << wqsort(a, 0, n - 1, k);
        return 0;
    }
    ```

### 两个有序数组第K大元素
这里指的是排列好后，下标为`k-1`的元素，不是去重后的第K大

1. 最基础的：两个数组合并到一个大数组，排序，返回即可
2. 改进一点：不用开太多的空间，用两个指针分别指向起始位置，移动就加一，一直加到等于K

??? "双指针"

    ```cpp
    int kthElement(vector<int> arr1, vector<int> arr2, int array1len, int array2len, int k) {
        int l1 = 0, l2 = 0;
        int cnt = 0;
        int tem = 0;
        while (l1 < array1len && l2 < array2len) {
            if (arr1[l1] < arr2[l2]) {
                tem = arr1[l1];
                l1++;
            }
            else {
                tem = arr2[l2];
                l2++;
            }
            cnt++;
            if (cnt == k) return tem;
        }
        while (cnt < k && l1 < array1len) {
            tem = arr1[l1++];
            cnt++;
        }
        while (cnt < k && l2 < array2len) {
            tem = arr2[l2++];
            cnt++;
        }
        if (cnt == k)return tem;
        return -1;
    }

    ```

**最优版**：可以发现这么一个规律 `l1 <= r2, l2 <= r1`

![cut](https://lh3.googleusercontent.com/3kUSML0zVJzEHWT5LVrLWWuzEJPyedVbFX0ykHI08Q9IzP8lon0Sl5m-qwj2b9Y8X9yCp6fbc8aSxb-3tmM7FmYPt8Q35FWlC3RK8adqo8ArIiH3oR19BqCSTvCDkC1Ujr-ap6cC)

只要找到符合上边条件并且元素个数正好为 K 的情况， `max(l1,l2)` 就是答案

- 假定 `arr1len <= arr2len` ，用二分确定从 arr1 取几个元素(arr2 取 `k- cut1` 个)

假如 `arr1len = 5, arr2len = 7`, `lo` 是 arr1 最少取的个数， `hi` 是最多取的个数

```txt
k = 3, lo = 0, hi = 3
k = 6, lo = 0, hi = 5
k = 11, lo = 4, hi = 5
```

arr1 取几个元素和 arr2len 有关，arr1最少取 `max(0, k-arr2len)`， arr1最多取 `min(k, arr1len)` 个

- 假如 arr1 取了0个元素，那么为了方便判断， `l1=INT_MIN`
- 假如 arr1 取了 arr1len 个元素，那么为了方便判断， `l1=INT_MAX`

```cpp
int kthElement(vector<int> &arr1, vector<int> &arr2, int arr1len, int arr2len, int k) {
    if (arr1len > arr2len) {
        return kthElement(arr2, arr1, arr2len, arr1len, k);
    }
    int lo = max(0, k - arr2len), hi = min(k, arr1len);
    while (lo <= hi) {
        int cut1 = lo + hi >> 1;
        int cut2 = k - cut1;
        int l1 = cut1 == 0 ? INT_MIN : arr1[cut1 - 1];
        int l2 = cut2 == 0 ? INT_MIN : arr2[cut2 - 1];
        int r1 = cut1 == arr1len ? INT_MAX : arr1[cut1];
        int r2 = cut2 == arr2len ? INT_MAX : arr2[cut2];

        if (l1 <= r2 && l2 <= r1) {
            return max(l1, l2);
        }
        else if (l1 > r2)
            hi = cut1 - 1;
        else lo = cut1 + 1;
    }
    return 1;
}
```

如果没有理解，可以参考以下资料，视频讲的比较清楚

- [印度老哥的视频讲解](https://takeuforward.org/data-structure/k-th-element-of-two-sorted-arrays/#google_vignette)
- [有关cut的讲解](https://blog.csdn.net/hk2291976/article/details/51107778?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-51107778-blog-119826156.235%5Ev40%5Epc_relevant_3m_sort_dl_base4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-51107778-blog-119826156.235%5Ev40%5Epc_relevant_3m_sort_dl_base4&utm_relevant_index=2)

### 环形链表2

[题目链接](https://leetcode.cn/problems/linked-list-cycle-ii/description/)

[完整版题解](https://leetcode.cn/problems/linked-list-cycle-ii/solutions/12616/linked-list-cycle-ii-kuai-man-zhi-zhen-shuang-zhi-/)

- 设有a个节点（不含环的起点），环内有b个节点
- 当第一次相遇时，

$$
\begin{aligned}
fast &=2*low \\
fast &=low+n*b \\
fast &=2nb \\
low  &=nb 
\end{aligned}
$$


- 所有从头开始走到环的起点都是 $a+Nb步$
- 所以low再走a步就到起点，那么让快指针重新指向头，一次一步走a步，两者就会重合

??? solve
    ```C++
    class Solution {
    public:
    ListNode *detectCycle(ListNode *head) {
        ListNode *low = head;
        ListNode *fast = head;
        bool ff = false;
        while (fast != nullptr && fast->next != nullptr) {
            fast = fast->next->next;
            low = low->next;
            if (fast == low) {
                ff = true;
                break;
            }
        }
        if (ff) {
           fast=head;
            while (fast!=low){
                fast=fast->next;
                low=low->next;
            }
            return low;
        }
        return nullptr;

    }
    };
    ```



### 寻找重复数 要求O(1)空间

[transport](https://leetcode.cn/problems/find-the-duplicate-number/description/)

以[1,3,4,2,2]为例，如果有相同数字，相当于会存在一个环

**核心**：
下标和内容一起做指向

| 下标 | 0 | 1 | 3 | 2 | 4       |
|------|---|---|---|---|---------|
| 内容 | 1 | 3 | 2 | 4 | 2(成环) |
| 节点 | 1 | 3 | 2 | 4 | 2       |

然后就和[环形链表2](https://leetcode.cn/problems/linked-list-cycle-ii/description/)一个做法，判环找入口

??? slove
    ```C++
    class Solution {
    public:
        int findDuplicate(vector<int>& nums) {
            int low=0,fast=0;
            //go 1step 2step
            low=nums[low];
            fast=nums[nums[fast]];
            while (low!=fast){
                low=nums[low];
                fast=nums[nums[fast]];
            }
            //fast goto begin node
            fast=0;
            while (low!=fast){
                low=nums[low];
                fast=nums[fast];
            }
            return fast;
            
        }
    };
    ```


### [最短无序连续子数组](https://leetcode.cn/problems/shortest-unsorted-continuous-subarray/)

**双指针**

- 找出升序，降序的区间，中间就是无序。
- 希望中间的值 `x>Lmax&&x<Rmin` ，反过来说，当 `x<Lmax||x>Rmin` 就应该调整左右端点
- 细节部分
- 为了方便调整到数组开始和结尾，用1e5+10和-1e5-10进行设置
- 为什么无序区间的数字开始从**L**找？如果从**L+1**开始，反例是 `1, 3, 2, 2, 2`

??? "双指针"
    ```cpp

    class Solution {
    public:
        int findUnsortedSubarray(vector<int> &nums) {
            if (nums.size() == 1) return 0;

            int l = 0, r = nums.size() - 1;
            while (l < r && nums[l] <= nums[l + 1]) l++;
            while (l < r && nums[r] >= nums[r - 1]) r--;

            int lmaxval = nums[l], rminval = nums[r];
            if (l == r) return 0;
    
            int i = l + 1;
            for (int k = l ; k < r; ++k) {
                if (nums[k] < lmaxval) {
                    while (l >= 0 && nums[k] < lmaxval) {
                        l--;
                        if (l < 0) lmaxval = -1e5 - 10;
                        else lmaxval = nums[l];
                    }
                
                }
                if (nums[k] > rminval) {
                    while (r < nums.size() && nums[k] > rminval) {
                        r++;
                        if (r >= nums.size())
                            rminval = 1e5 + 10;
                        else rminval = nums[r];
                    }
                
                }
            }

            return r - l - 1;
        }
    };
    ```

**一次遍历** [传送门](https://leetcode.cn/problems/shortest-unsorted-continuous-subarray/solutions/422614/si-lu-qing-xi-ming-liao-kan-bu-dong-bu-cun-zai-de-/comments/1194164)

先只考虑中段数组，设其左边界为L，右边界为R：

`nums[R]` 不可能是 `[L，R]` 中的最大值（否则应该将 `nums[R]` 并入右端数组）

`nums[L]` 不可能是`[L,R]`中的最小值（否则应该将 `nums[L]` 并入左端数组）

很明显:

 `[L,R]` 中的最大值 等于 `[0，R]` 中的最大值，设其为 max

 `[L,R]` 中的最小值 等于 `[L， nums.length-1]`中的最小值，设其为 min

那么有：

`nums[R] < max < nums[R+1] < nums[R+2] < ...`  所以说，从左往右遍历，最后一个小于max的为右边界

`nums[L] > min > nums[L-1] > nums[L-2] > ... ` 所以说，从右往左遍历，最后一个大于min的为左边界

??? "一次遍历"
    ```cpp
    class Solution {
    public:
        int findUnsortedSubarray(vector<int> &nums) {

    
            int min = nums[nums.size() - 1], max = nums[0];
            int end = -1, begin = 0;
            //end和begin的初值不重要，让end-bigin+1=0即可
            for (int i = 0; i < nums.size(); ++i) {
                if (nums[i] < max)
                    end = i;
                else max = nums[i];

                if (nums[nums.size() - 1 - i] > min)
                    begin = nums.size() - 1 - i;
                else min = nums[nums.size() - 1 - i];
            }
            return end - begin + 1;
        }
    };
    ```


### [最长上升子序列](https://www.acwing.com/problem/content/description/897/)

1. dp $O(n^2)$ , `dp[i]=max(dp[i],dp[j]+1) when a[i]>a[j],`
2. dp+贪心，每次找 `x<=anw[i]`的左端点更新
3. 记忆化搜索

=== "dp On2"

    ```cpp
    #include "bits/stdc++.h"

    using namespace std;

    int main() {
        int n;
        cin >> n;
        vector<int> a(n);
        for (int i = 0; i < n; i++)
            cin >> a[i];
        vector<int> dp(n, 1);
        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (a[i] > a[j])
                    dp[i] = max(dp[i], dp[j] + 1);
            }
        }
        cout << *max_element(dp.begin(), dp.end());
        return 0;
    }
    ```

=== "dp+贪心"

    ```cpp
    #include "bits/stdc++.h"

    using namespace std;

    void check(int x, vector<int> &anw) {
        int l = 0, r = anw.size() - 1;
        while (l < r) {
            int mid = l+r>>1;
            if (anw[mid] <x)l = mid+1;
            else r = mid;
        }

        anw[r] = x;
    }

    int main() {
        int n;
        cin >> n;
        vector<int> a(n);
        for (int i = 0; i < n; i++)
            cin >> a[i];
        vector<int> anw;
        anw.push_back(a[0]);
        for (int i = 1; i < n; i++) {
            if (anw[anw.size() - 1] < a[i])
                anw.push_back(a[i]);
            else check(a[i], anw);
        }
        cout << anw.size();
        return 0;
    }
    ```

### [数组中的逆序对](https://www.acwing.com/problem/content/description/61/)

- 归并排序，注意循环的边界是 `l r`不是 **0**
- 套模板会空间多一点但是直观，优化一下相当于不停在求子问题

=== "开空间较多"

```cpp
class Solution {
public:
    int inversePairs(vector<int> &nums) {
        if(nums.size()==0) return 0;
        int anw = mergesort(nums, 0, nums.size() - 1);
        return anw;
    }

    int mergesort(vector<int> &nums, int l, int r) {
        if (l == r) return 0;
        int mid = l + r >> 1;
        int cnt = 0;
        cnt += mergesort(nums, l, mid);
        cnt += mergesort(nums, mid + 1, r);

        int m = mid, n = r, id = r;
        
        vector<int> anw(nums.size());
        
        while (m >= l && n > mid) {
            if (nums[n] >= nums[m]) anw[id--] = nums[n--];
            else cnt+=n-mid, anw[id--] = nums[m--];
        }
        while (m >= l) anw[id--] = nums[m--];
        while (n > mid)anw[id--] = nums[n--];
        
        id = l;
        while (id <= r)
            nums[id] = anw[id], id++;
            
        return cnt;
    }
};
```

=== "On空间"

```cpp
class Solution {
public:
    int inversePairs(vector<int> &nums) {
        if (nums.size() == 0) return 0;
        vector<int> tem = nums;
        int anw = mergesort(nums, 0, nums.size() - 1, tem);
        return anw;
    }

    int mergesort(vector<int> &nums, int l, int r, vector<int> &tem) {
        if (l == r) {
            tem[l] = nums[l];
            return 0;
        }
        int mid = l + r >> 1;
        int cnt = 0;
        cnt += mergesort(tem, l, mid, nums);
        cnt += mergesort(tem, mid + 1, r, nums);

        int m = mid, n = r, id = r;
        while (m >= l && n >= mid+1) {
            if(nums[m]>nums[n]) {
                tem[id--]=nums[m--];
                cnt+=n-mid;
            }
            else tem[id--]=nums[n--];
        }
        while (m >= l) tem[id--] = nums[m--];
        while (n > mid)tem[id--] = nums[n--];

        return cnt;
    }
};
```

### [编辑距离](https://leetcode.cn/problems/edit-distance/)

[题解](https://leetcode.cn/problems/edit-distance/solutions/2468072/dai-ma-sui-xiang-lu-72-bian-ji-ju-chi-by-or3j/)

可以优化成O(M)

??? "slove"

    ```cpp
    class Solution {
    public:
        void printmarix(vector<int> source) {
            for (auto x : source)
                cout << x << ' ';
            cout << endl;
        }

        void printVect(vector<vector<int>> dp) {
            for (auto x : dp) {
                for (auto y : x)
                    cout << y << ' ';
                cout << endl;
            }
        }

        int minDistance(string source, string dest) {
            vector<int> dp(dest.size() + 1, 0);


            for (int i = 0; i <= dest.size(); i++)
                dp[i] = i;

            for (int i = 1; i <= source.size(); i++) {
                int tem = dp[0];
                dp[0] = i;
                for (int j = 1; j <= dest.size(); j++) {
                    int pre_dp_j = dp[j];
                    if (source[i - 1] == dest[j - 1])
                        dp[j] = tem;
                    else
                        dp[j] = min(tem, min(dp[j - 1], dp[j])) + 1;

                    tem = pre_dp_j;
                }
                //  printmarix(dp);
            }
            return dp[dest.size()];
        }

        int minDistance_corr(string source, string dest) {
            vector<vector<int>> dp(source.size() + 1, vector<int>(dest.size() + 1, 0));

            for (int i = 0; i <= source.size(); i++)
                dp[i][0] = i;
            for (int i = 0; i <= dest.size(); i++)
                dp[0][i] = i;

            for (int i = 1; i <= source.size(); i++)
                for (int j = 1; j <= dest.size(); j++) {
                    if (source[i - 1] == dest[j - 1])
                        dp[i][j] = dp[i - 1][j - 1];
                    else
                        dp[i][j] = min(min(dp[i - 1][j - 1], dp[i - 1][j]), dp[i][j - 1]) + 1;
                }
            //  printVect(dp);
            return dp[source.size()][dest.size()];
        }
    };

    ```

## review
two sum with link node

??? "solve"

    ```cpp
    class Solution {
    public:
        ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
            ListNode* head = new ListNode(0);
            ListNode* tem = head;
            int adding = 0;
            while (l1 || l2) {
                int x = 0, y = 0;
                if (l1) x = l1->val;
                if (l2) y = l2->val;
                int sum = (x + y + adding) % 10;
                adding = (x + y + adding) / 10;
                ListNode* ne = new ListNode(sum);
                tem->next = ne;
                tem = tem->next;
                if (l1)
                    l1 = l1->next;
                if (l2)
                    l2 = l2->next;
            }
            if (adding) {
                ListNode* ne = new ListNode(adding);
                tem->next = ne;
            }
            tem = head->next;
            delete head;
            return tem;
        }
    };
    ```

[**The Dutch national flag**](https://en.wikipedia.org/wiki/Dutch_national_flag_problem): sort an array of some 0,1,2 in O(n) 

- [0, i-1] < midElement
- [i, j-1] = midElement
- [j, k] unsorted
- [k+1, end] >midElement


??? "solve"

    ```cpp
    void dutchFlag(vector<int>&todo){
        int N=todo.size();
        int low=0,mid=0,high=N-1;
        while (mid<=high){
            if(todo[mid]==0){
                swap(todo[low],todo[mid]);
                low++;
                mid++;
            }
            else if(todo[mid]==2){
                swap(todo[mid],todo[high]);
                high--;
            }
            else mid++;
        }
    }
    ```


**Merging with smaller auxiliary array**: given an array[2n], which is sorted from a[0] to a[n], and sorted from a[n+1] to a[2n]. you need to sort the entire array with O(n) space

solve:
1. copy the first part to auxiliary array
2. merge auxiliary array and the second part of original array 

??? "solve"

    ```cpp
    #include <vector>
    #include <algorithm>
    #include <iostream>

    using namespace std;

    void merge(vector<int> &a) {
        vector<int> aux = vector<int>(a.size() / 2);
        int half = a.size() / 2;
        for (int i = 0; i < half; i++)
            aux[i] = a[i];

        int lo = 0, hi = half;
        int k = 0;
        
        while (lo < half && hi < a.size()) {
            if (aux[lo] < a[hi])
                a[k++] = aux[lo++];
            else a[k++] = a[hi++];
        }
        while (lo < half) a[k++] = aux[lo++];
        while (hi < a.size()) a[k++] = a[hi++];

    }

    void test() {
        vector<int> a = {6, 7, 8, 9, 10, 1, 2, 3, 4, 5};
        vector<int> expect = a;
        std::sort(expect.begin(), expect.end());

        merge(a);
        for (auto x: a)
            cout << x << ' ';

        if (a == expect)
            cout << "yes";
        else
            cout << "NO";
    }

    int main() {
        test();
        return 0;
    }
    ```

## codeforces

### [lakes](https://codeforces.com/contest/1829/problem/E)

??? "lakes"

    这个题在于剪枝，有的不用再dfs了，不然超时。假如（1，1）和（1，2）联通，dfs（1，1）和dfs（1，2）是一个结果。
    ```C++
    #include <cstring>
    #include "iostream"

    using namespace std;
    const int N = 1010;
    int gra[N][N];
    int n, m;
    int total;
    int dx[4] = {0, 0, -1, 1};
    int dy[4] = {1, -1, 0, 0};
    bool visited[N][N];


    int dfs(int a, int b) {
        visited[a][b] = true;
        if (gra[a][b] == 0) return 0;
        int anw = gra[a][b];

        for (int i = 0; i < 4; i++) {
            int nx = a + dx[i];
            int ny = b + dy[i];
            if (nx >= 1 && nx <= n && ny >= 1 && ny <= m
                && gra[nx][ny] > 0 && visited[nx][ny] == false) { anw += dfs(nx, ny); }
        }
        return anw;
    }

    void solve() {

        int fin = 0;
        cin >> n >> m;

        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= m; j++)
                cin >> gra[i][j];


        memset(visited, 0, sizeof visited);
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= m; j++) {
                if (gra[i][j] != 0 && visited[i][j] == false) {

                    fin = max(dfs(i, j), fin);
                }
            }
        cout << fin << endl;

    }

    int main() {

        cin >> total;
        while (total--)
            solve();
        return 0;
    }

    ```

### [Hits Different](https://codeforces.com/contest/1829/problem/G)

[前缀和动画讲解](https://usaco.guide/silver/more-prefix-sums?lang=cpp#2d-prefix-sums)

??? "solve"

    非常巧妙啊，转成前缀和,详情可以见相应英文题解
    ```C++
    #include "iostream"

    using namespace std;

    typedef long long  llint;
    llint anw[2050000];
    llint gra[2029][2029];
    llint cur = 1;

    void solve() {
        llint x;
        cin >> x;
        cout << anw[x] << endl;
    }

    int main() {
        ios::sync_with_stdio(false);
        cin.tie(nullptr);
        int n;
        for (int i = 1; i <= 2023; i++)
            for (int j = i; j >= 1; j--) {
                gra[j][i - j + 1] = gra[j - 1][i - j + 1] + gra[j][i - j + 1 - 1]
                                    - gra[j - 1][i - j + 1 - 1]
                                    + cur * cur;
                anw[cur] = gra[j][i - j + 1];
                cur++;
            }
        cin >> n;
        while (n--)
            solve();
        return 0;
    }
    ```

### [Distinct Split](https://codeforces.com/contest/1791/problem/D)

??? slove
    1. 一次遍历统计出所有字母的出现次数
    2. 从前往后开始算，给pre分一个字母，就在该字母出现总数-1
    3. 统计所有字母，进行加和；
    巧妙在相当于并行处理2个字符串，想不出来😥 

