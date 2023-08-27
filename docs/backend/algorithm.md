---
comments: true
---

## leetcode

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


## 剑指offer

### [二叉树的下一个节点](https://www.acwing.com/problem/content/description/31/)

- 中序遍历**左根右**
- 如果不是叶节点，要求的点相当于先往右一步，再一直往左走到头,是一个 **>** 的形状
- 如果是叶节点**A**，可以先考虑它的下一个节点**B**怎么到的它本身，
    - 就是**B**先往左一步，再往右走到头到达**A**，形成一个 **<** 的形状
    - 然后发现拐角处容易求出来 `p->father->left==p`
    - 如果说没有父节点，说明没有要求的答案，返回 `nullptr`

??? slove
    ```cpp
    class Solution {
    public:
        TreeNode *inorderSuccessor(TreeNode *p) {
            if (p->right != nullptr) {
                p = p->right;
                while (p->left != nullptr)
                    p = p->left;
                return p;
            }
            else {
                while (p->father != nullptr && p->father->left != p)
                    p = p->father;
                if (p->father == nullptr)
                    return nullptr;
                else
                    return p->father;
            }
        }
    };
    ```


### [旋转数组的最小数字](https://www.acwing.com/problem/content/description/20/)

- 从后往前把和 `nums[0]` 相同的删除
- 特判一下：如果是完全升序就返回 ` nums[0]`
- 进行二分查找 小于 `nums[0]` 的左端点

[辅助题解](https://www.acwing.com/solution/content/727/)

??? slove
    ```cpp
    class Solution {
    public:
        int findMin(vector<int>& nums) {
            if(nums.size()==0) return -1;
            int n=nums.size()-1;
            while (n>0&&nums[0]==nums[n]) n--;
            if(nums[n]>=nums[0]) return nums[0];
            
            int l=0,r=n;
            while (l<r){
                int mid=l+r>>1;
                if(nums[mid]<nums[0]) r=mid;
                else l=mid+1;
            }
            return nums[l];
        }
    };
    ```

**举一反三**😋

这个题做完了，可以尝试[搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/description/)

- 先特判一下是不是完全升序，决定是不是直接二分
- 和上一题一样，先找到最小值
- 如果目标值在最小值和结尾中间就二分，反之在另一半二分

??? solve
    ```cpp
    class Solution {
    public:
        int search(vector<int> &nums, int target) {

            if (nums[nums.size() - 1] > nums[0])
                return bins(nums, 0, nums.size() - 1, target);

            int l = 0, r = nums.size() - 1;
            while (l < r) {
                int mid = l + r >> 1;
                if (nums[mid] < nums[0]) r = mid;
                else l = mid + 1;
            }
            cout << nums[l] << endl;

            if (target >= nums[l] && target < nums[0])
                return bins(nums, l, nums.size() - 1, target);
            else
                return bins(nums, 0, l - 1, target);

        }

        int bins(vector<int> &nums, int l, int r, int target) {
            while (l < r) {
                int mid = l + r >> 1;
                if (nums[mid] >= target)
                    r = mid;
                else l = mid + 1;
            }
            if (nums[l] != target) return -1;
            else return l;
        }
    };
    ```

### [矩阵中的路径](https://www.acwing.com/problem/content/21/)

为什么这个不能调过来

```cpp
//correct
if (matrix[x][y] != str[po]) return false;
if (po == str.size() - 1) return true;
// wrong
if (po == str.size()) return true;
if (matrix[x][y] != str[po]) return false;  
```

这个例子 `str=a martix=[a]`

第一步成功了，但是在继续dfs的时候是不能继续dfs的，因为 `if (!(cx >= 0 && cx < n && cy >= 0 && cy < m)) continue;` 下标全都越界，进入 `continue` ,没法进入预计的 `dfs(1)`

??? slove
    ```cpp
    class Solution {
    public:
        bool hasPath(vector<vector<char>> &matrix, string &str) {
            if (matrix.size() == 0) return false;
            bool ff = false;

            int n = matrix.size();
            int m = matrix[0].size();

            for (int i = 0; i < n; ++i) {
                for (int j = 0; j < m; ++j) {
                    if (dfs(i, j, 0, str, matrix)) {
                        ff = true;
                        break;
                    }
                }
            }
            if (ff) return true;
            else return false;

        }

        bool dfs(int x, int y, int po, string &str, vector<vector<char>> &matrix) {
            if (matrix[x][y] != str[po]) return false;
            if (po == str.size() - 1) return true;

            int dx[4] = {-1, 1, 0, 0};
            int dy[4] = {0, 0, -1, 1};
            int n = matrix.size();
            int m = matrix[0].size();

            char tem = matrix[x][y];
            matrix[x][y] = '*';
            for (int i = 0; i < 4; ++i) {
                int cx = x + dx[i];
                int cy = y + dy[i];
                if (!(cx >= 0 && cx < n && cy >= 0 && cy < m)) continue;
                if (dfs(cx, cy, po + 1, str, matrix))
                    return true;
            }
            matrix[x][y] = tem;
            return false;
        }
    };
    ```

### [二进制中1的个数](https://www.acwing.com/problem/content/description/25/)

1. 循环32次，每次判断最低位是不是1，再右移一位
2. lowbit操作，求一次 [lowbit](https://oi-wiki.org/ds/fenwick/) ,anw++； 数字-lowbit

=== "循环32次"

    ```cpp
    class Solution {
    public:
    int NumberOf1(int n) {

        int anw=0;

        for(int i=0;i<32;i++){
            if(n&1) anw++;
            n>>=1;
        }
        return anw;
    }
    };
    ```

=== "lowbit"

    ```cpp
    class Solution {
    public:
    int NumberOf1(int n) {
        int anw=0;
        while(n!=0){
            n=n-(n&-n); anw++;
        }
        return anw;
    }
    };
    ```


### [删除链表中重复的节点](https://www.acwing.com/problem/content/description/27/)

- 我理解此题关键在于**重复数字区间起点的前一个节点**，如果找区间的起点，不方便修改区间起点前一个的指向
- 为了方便删除开始就有相同的情况，在头节点前开一个假的节点 `vir`
- p指向vir，当p的下一个值是重复的一段 `[p->next,lat]` ，lat前进
    - 如果重复， `p->next=lat->next` ;但是不能保证p直接进入 `p->next` 后不重复。
        - 反例： `1, 2, 3, 3, 4, 4, 5`
    - 不重复，这个时候已经保证p，p的下一个，p的下第二个都不是重复，放心大胆的往下连接🥰

??? slove
    ```cpp
    class Solution {
    public:
        ListNode *deleteDuplication(ListNode *head) {
            ListNode *vir = new ListNode(-1);
            vir->next = head;
            
            ListNode *p = vir;

            while (p->next){
                ListNode* lat=p->next;
                while (lat->next&&lat->next->val==p->next->val)
                    lat=lat->next;
                if(p->next==lat){
                    p->next=lat;
                    p=p->next;
                }
                else{
                    p->next=lat->next;
                }
            }
            return vir->next;
        }
    };
    ```


### [调整数组顺序使奇数位于偶数前面](https://www.acwing.com/problem/content/description/30/)

- 双指针，一个从前往后找偶数，一个从后往前找奇数，不重复就交换，重复就退出
- 还尝试用归并去求，但是TLE了，原理应该是对的,还能保证相对顺序
    - 把左区间结尾的偶数和右区间的奇数进行互换


??? "双指针，归并"
    ```cpp
    class Solution {
    public:
        void reOrderArray(vector<int> &array) {
            if(array.size()==0) return;
            int l = 0, r = array.size() - 1;
            while(l<r){
            while (l < r && array[l] % 2 !=0) l++;
            while (l<r&&array[r]%2==0) r--;
            if(l==r) return;
            else swap(array[l],array[r]);}
        }

        vector<int> mergesort(vector<int> &array, int l, int r) {
            if (l >= r) return array;
            int mid = l + r >> 1;
            mergesort(array, 0, mid);
            mergesort(array, mid + 1, r);

            int i = mid, j = mid + 1;
            vector<int> tem;

            while (i >= 0 && j <= r && array[i] % 2 == 0 && array[j] % 2 != 0)
                swap(array[i--], array[j--]);
            return array;

        }
    };
    ```

### [反转链表](https://www.acwing.com/problem/content/33/)

老生常谈了，刚学链表的时候受不了直接投降😰。力扣的递归感觉不是一下就理解了，我这个应该好一些

=== "迭代"

    ```cpp
    class Solution {
        public:
        ListNode* reverseList(ListNode* head) {

            if(head== nullptr) return head;
            ListNode* pre= nullptr;
            while (head!= nullptr){
                ListNode*lat=head->next;
                head->next=pre;
                pre=head;
                head=lat;
            }
            return pre;
        }
    };
    ```

=== "递归"

    ```cpp
    class Solution {
    public:
        ListNode *reverseList(ListNode *head) {

            if (head == nullptr) return head;
            return rev(head, nullptr);
        }

        ListNode *rev(ListNode *head, ListNode *pre) {
            if (head == nullptr) return pre;

            auto tem = head->next;
            head->next = pre;
            //先修改当前的next，再递归下一个
            auto anw = rev(tem, head);
            return anw;

        }
    };
    ```

### [树的子结构](https://www.acwing.com/problem/content/35/)

想出来一部分😰

??? slove
    ```cpp
    class Solution {
    public:
        bool hasSubtree(TreeNode *pRoot1, TreeNode *pRoot2) {
            if (pRoot2 == nullptr || pRoot1 == nullptr) return false;
            if (check(pRoot1, pRoot2))
                return true;
            return hasSubtree(pRoot1->left, pRoot2) || hasSubtree(pRoot1->right, pRoot2);
            //1--p1,p2为根就相同
            //2--p1的左子树里和p2相同
            //3--p1的右子树里和p2相同
        }

        bool check(TreeNode *p1, TreeNode *p2) {
            if (p2 == nullptr) return true;//条件成立
            if (p1 == nullptr) return false;//2空1不空
            if (p1->val != p2->val) return false;//12都不空且值不同
            return check(p1->left, p2->left) && check(p1->right, p2->right);
        }
    };
    ```

### [对称的二叉树](https://www.acwing.com/problem/content/description/38/)

直接看代码更好理解

1. 递归检查两个节点a,b的值，a的左子树和b的右子树，a的右子树和b的左子树是否相同 👍👍👍
2. bfs a,b的值，a的左子树和b的右子树，a的右子树和b的左子树是否相同 👍👍
3. bfs 把下一层的值全存起来看是不是对称 👍

=== "递归"

    ```cpp
    class Solution {
    public:
        bool isSymmetric(TreeNode *root) {

            if (root == nullptr) return true;
            return dfs(root->left, root->right);
        }

        bool dfs(TreeNode *l1, TreeNode *l2) {
            if (l1 == nullptr || l2 == nullptr)
                return l1 == nullptr && l2 == nullptr;
            //非常的精妙啊
            return l1->val == l2->val && dfs(l1->left, l2->right)
                && dfs(l1->right, l2->left);
        }
    };
    ```

=== "bfs左右"

    ```cpp
    class Solution {
    public:
        bool isSymmetric(TreeNode *root) {
            if (root == nullptr) return true;
            return bfs(root);
        }

        bool bfs(TreeNode *root) {
            queue<TreeNode *> qu;
            qu.push(root->left);
            qu.push(root->right);
            while (qu.size()) {
                TreeNode *left = qu.front();
                qu.pop();
                TreeNode *right = qu.front();
                qu.pop();
                if (left == nullptr && right == nullptr)
                    continue;
                if (left == nullptr && right != nullptr ||
                    left != nullptr && right == nullptr ||
                    left->val != right->val)
                    return false;

                qu.push(left->left);
                qu.push(right->right);
                qu.push(left->right);
                qu.push(right->left);
            }
            return true;
        }
    };
    ```

=== "bfs全存"

    ```cpp
    class Solution {
    public:
        bool isSymmetric(TreeNode *root) {
            if (root == nullptr) return true;
            return bfs(root);
        }

        bool bfs(TreeNode *root) {
            queue<TreeNode *> qu;
            qu.push(root);
            while (qu.size()) {
                vector<TreeNode *> newqu;
                while (qu.size()) {
                    auto tem = qu.front();
                    qu.pop();
                    if (tem == nullptr) continue;
                    newqu.push_back(tem->left);
                    newqu.push_back(tem->right);
                }

                for (int i = 0, j = newqu.size() - 1; i < j;) {

                    if (newqu[i] == nullptr && newqu[j] != nullptr
                        || newqu[i] != nullptr && newqu[j] == nullptr)
                        return false;
                    if(newqu[i]== nullptr&&newqu[j]== nullptr) 
                    {i++,j--;continue;}
                    if (newqu[i]->val == newqu[j]->val) i++, j--;
                    else return false;
                }

                for (auto x: newqu) {
                    qu.push(x);
                }

            }


            return true;
        }
    };
    ```

### [栈的压入、弹出序列](https://www.acwing.com/problem/content/40/)

[题解](https://www.acwing.com/solution/content/44046/)

??? "slove"
    ```cpp
    class Solution {
    public:
        bool isPopOrder(vector<int> pushV,vector<int> popV) {
            if(pushV.size()!=popV.size()) return false; //两个序列长度不等，直接返回false
            int k=0;
            stack<int>st;
            for (int i = 0; i < popV.size(); ++i) {
                while (st.empty()||st.top()!=popV[i]&&k<pushV.size()) 
                    st.push(pushV[k++]);
                if(st.top()==popV[i]) st.pop();
                else return false;
            }
            return st.empty();
        }
    };
    ```

### [分行从上往下打印二叉树](https://www.acwing.com/problem/content/42/)

用一个变量记录一下本层处理几个就好了

??? "slove"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> printFromTopToBottom(TreeNode *root) {

            vector<vector<int>> anw;
            if (root == nullptr) return anw;
            queue<TreeNode *> qu;
            qu.push(root);
            while (qu.size()) {
                vector<int> res;
                int k = qu.size();
                while (k--) {
                    auto tem = qu.front();
                    qu.pop();
                    res.push_back(tem->val);
                    if (tem->left != nullptr) qu.push(tem->left);
                    if (tem->right != nullptr) qu.push(tem->right);
                }
                anw.push_back(res);
            }
        }
    };
    ```

### [二叉树中和为某一值的路径](https://www.acwing.com/problem/content/description/45/)

dfs思路，处理本层，判断下一层

一些小坑：

- 有可能 **没到结尾（路径端点）时结果就相等** ，但是这不是我们要的
- 如果传引用注意回复现场

=== "传值"

    ```cpp
    class Solution {
    public:

        vector<vector<int>> anw;

        vector<vector<int>> findPath(TreeNode *root, int sum) {
            vector<int> tem;
            dfs(root, 0, sum, tem);
            return anw;
        }

        void dfs(TreeNode *root, int now, int sum, vector<int> tem) {

            if (root == nullptr) return;

            now += root->val;
            tem.push_back(root->val);
            if (now == sum&&root->left== nullptr&&root->right== nullptr) {
                anw.push_back(tem);
                return;
            }
            if (root->left != nullptr) {
                dfs(root->left, now, sum, tem);
            }
            if (root->right != nullptr) {
                dfs(root->right, now, sum, tem);
            }
        }
    };
    ```

=== "传引用"

    ```cpp
    class Solution {
    public:

        vector<vector<int>> anw;

        vector<vector<int>> findPath(TreeNode *root, int sum) {
            vector<int> tem;
            dfs(root, 0, sum, tem);
            return anw;
        }

        void dfs(TreeNode *root, int now, int sum, vector<int>& tem) {

            if (root == nullptr) return;

            now += root->val;
            tem.push_back(root->val);
            if (now == sum&&root->left== nullptr&&root->right== nullptr) {
                anw.push_back(tem);
                //取消掉注释也可以，但是popback不能丢
                //不然既没进入最后的poopback也没回复现场
                // tem.pop_back();
                // return;
            }
            
            dfs(root->left, now, sum, tem);
            dfs(root->right, now, sum, tem);
            tem.pop_back();
        }
    };
    ```


### [复杂链表的复刻](https://www.acwing.com/problem/content/description/89/)

[直接看视频题解](https://www.acwing.com/video/172/)

这是人能想出来的？😰 多复习复习吧

??? "solve"
    ```cpp
    class Solution {
    public:
        ListNode *copyRandomList(ListNode *head) {

            if (head == nullptr) return head;

            auto p = head;
            while (p != nullptr) {
                ListNode *np = new ListNode(p->val);
                auto next = p->next;
                np->next = next;
                p->next = np;
                p = p->next->next;
            }

            p = head;
            while (p != nullptr) {
                if (p->random != nullptr)
                    p->next->random = p->random->next;
                p = p->next->next;
            }

            ListNode *vir = new ListNode(-1);
            p = head;
            ListNode *cur = vir;
            while (p != nullptr) {
                cur->next = p->next;
                p->next = p->next->next;
                p = p->next;
                cur = cur->next;
            }
            return vir->next;
        }
    };
    ```

### [二叉搜索树与双向链表](https://www.acwing.com/problem/content/description/87/)

还真是人能想出来的，我想不出来，所以我不是人。 多复习复习吧

??? "slove"
    ```cpp
    class Solution {
    public:
    
        TreeNode *pre = nullptr;

        TreeNode *convert(TreeNode *root) {

            if (root == nullptr) return nullptr;
            midread(root);
            while (root->left != nullptr)
                root = root->left;
            return root;

        }

        void midread(TreeNode *root) {
            if (root == nullptr) return;
            midread(root->left);

            root->left = pre;
            if (pre) pre->right = root;
            pre = root;

            midread(root->right);
        }
    };
    ```

### [数据流中的中位数](https://www.acwing.com/problem/content/description/88/)

用两个堆动态维护序列，最大堆放比中位数小的值，最小堆放比中位数大的值，想法很精妙，实现方法也比较多

- 可以先往最小堆里放，也可以先往最大堆里放，我这个先放到最小堆
- 如果两个堆顶逆序，调整，如果最小堆数量==最大堆数量，把最小堆的堆顶放到最大堆里
- 取的时候，如果元素个数为奇数，取最大堆堆顶，偶数取两个堆顶平均值

![示例图](../image/swardofferMidnum.png){ loading=lazy }

??? "solve"
    ```cpp
    class Solution {
    public:
        priority_queue<int, vector<int>, greater<>> minheap;
        priority_queue<int, vector<int>, less<>> maxheap;

        void insert(int num) {
            minheap.push(num);
            if (maxheap.size() && minheap.top() < maxheap.top()) {
                int minv = minheap.top(), maxv = maxheap.top();
                minheap.pop(), maxheap.pop();
                minheap.push(maxv), maxheap.push(minv);
            }
            if (minheap.size() > maxheap.size()) {
                maxheap.push(minheap.top());
                minheap.pop();
            }

        }

        double getMedian() {
            if ((maxheap.size() + minheap.size()) % 2 == 0) {
                return (maxheap.top() + minheap.top()) / 2.0;
            }
            else
                return maxheap.top();
        }
    };
    ```

### [连续子数组的最大和](https://www.acwing.com/problem/content/description/50/)

比较简单的一道题 

=== "空间On"

    ```cpp
    class Solution {
    public:
        int maxSubArray(vector<int>& nums) {

            vector<int>anw(nums.size());
            anw[0]=nums[0];
            for (int i = 1; i <nums.size(); ++i) {
                if(nums[i]+anw[i-1]>nums[i]) anw[i]=nums[i]+anw[i-1];
                else anw[i]=nums[i];
            }
            return *max_element(anw.begin(), anw.end());
        }
    };
    ```

=== "空间O1"

    ```cpp
    class Solution {
    public:
        int maxSubArray(vector<int>& nums) {

            vector<int>anw(nums.size());
            int pre=nums[0];
            int fin=nums[0];
            for (int i = 1; i <nums.size(); ++i) {
                if(pre>0) pre+=nums[i];
                else pre=nums[i];
                fin= max(fin,pre);
            }
            return fin;
        }
    };
    ```

### [求1+2+…+n](https://www.acwing.com/problem/content/80/)

短路或者开空间，开空间真是 大受震撼.jpg

=== "短路"

    ```cpp
    class Solution {
    public:
        int getSum(int n) {
            (n>0)&&(n+= getSum(n-1));
            return n;
        }
    };
    ```

=== "开空间"

    ```cpp
    class Solution {
    public:
        int getSum(int n) {
            char x[n][n+1];
            return sizeof(x)>>1;
        }
    };
    ```

### [把数组排成最小的数](https://www.acwing.com/problem/content/54/)

不好解释

??? "slove"

    ```cpp
    class Solution {
    public:
        string printMinNumber(vector<int> &nums) {
            string anw;
            if (nums.size() == 0) return anw;
            sort(nums.begin(), nums.end(), [](int a, int b) {
                if (to_string(a) + to_string(b) < to_string(b) + to_string(a))
                    return true;
                else return false;
            });
            for(int  x:nums){
                anw+= to_string(x);
            }
            return anw;
        }
    };
    ```

### [把数字翻译成字符串](https://www.acwing.com/problem/content/55/)

这个还好，最开始想的是dfs处理，看题解后发现还可以用跳台阶做。
都差不多😋

=== "dfs"

    ```cpp
    class Solution {
    public:

        int getTranslationCount(string s) {
            int anw = dfs(s, 0);
            return anw;
        }

        int dfs(string s, int po) {
            if (po >= s.size()) return 1;
            int cnt = 0;
            cnt += dfs(s, po + 1);

            if (po + 1 < s.size() && ((s[po] - '0') * 10 + (s[po + 1] - '0') <= 25)
                && (s[po] - '0') * 10 + (s[po + 1] - '0') != (s[po + 1] - '0')
                    )
                cnt += dfs(s, po + 2);
            return cnt;
        }
    };
    ```

=== "dp"

    ```cpp
    class Solution {
    public:
        int getTranslationCount(string s) {

            vector<int> dp(s.size() + 1, 0);
            dp[0] = 1;
            if (s.size() == 1) return dp[0];
            dp[1] = 1;
            for (int i = 2; i <= s.size(); ++i) {
                dp[i] = dp[i - 1];
                int a = s[i - 2] - '0', b = s[i - 1] - '0';
                if (a != 0 && a * 10 + b <= 25)
                    dp[i] += dp[i - 2];
            }

            return dp[s.size()];
        }
    };
    ```

### [礼物的最大价值](https://www.acwing.com/problem/content/56/)

- dfs时间超了，用dp
-  `grid[i][j]` 用过之后就没用了，可以直接存结果，节省一点空间

??? "不额外空间"

    ```cpp
    class Solution {
    public:
        int getMaxValue(vector<vector<int>>& grid) {

            int m=grid.size(),n=grid[0].size();
            for (int i = 0; i < m; ++i) {
                for (int j = 0; j < n; ++j) {
                    if(i&&j) grid[i][j]+= max(grid[i-1][j],grid[i][j-1]);
                    else if(i) grid[i][j]+=grid[i-1][j];
                    else if(j) grid[i][j]+=grid[i][j-1];
                }
            }
            return grid[m-1][n-1];
        }
    };
    ```

??? "额外空间"

    ```cpp
    class Solution {
    public:
        int getMaxValue(vector<vector<int>> &grid) {
            int m = grid.size(), n = grid[0].size();

            vector<vector<int>> dp(grid.size() + 1,
                                vector<int>(grid[0].size() + 1,0));
            for (int i = 0; i <= m; ++i)
                dp[m][0] = 0;
            for (int i = 0; i <= n; ++i) {
                dp[0][n] = 0;
            }

            for (int i = 1; i <= m; ++i) {
                for (int j = 1; j <= n; ++j) {
                    dp[i][j] = max(dp[i][j - 1], dp[i - 1][j]) + grid[i - 1][j - 1];
                }
            }
            return dp[m][n];
        }
    };
    ```


## codeforces

### [lakes](https://codeforces.com/contest/1829/problem/E)

??? lakes
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

??? solve
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


## template

mergesort

??? "&"
    ```C++
    #include "iostream"
    #include "vector"

    using namespace std;

    void merge(vector<int> &nums, vector<int> &copy, int l, int r) {
        if (l == r) {
            copy[l] = nums[l];
            return;
        }
        int mid = l + r >> 1;
        merge(copy, nums, l, mid);
        merge(copy, nums, mid + 1, r);

        int i = l, j = mid + 1, idx = l;
        while (i <= mid && j <= r) {
            if (nums[i] < nums[j]) copy[idx++] = nums[i++];
            else copy[idx++] = nums[j++];
        }
        while (i <= mid)
            copy[idx++] = nums[i++];
        while (j<=r)
            copy[idx++] = nums[j++];

    }

    void mergesort(vector<int> &nums) {
        if (nums.size() <= 1) return;
        vector<int> copy = nums;
        merge(nums, copy, 0, nums.size() - 1);
        nums=copy;
    }

    int main() {
        vector<int> nums;
        int n;
        cin >> n;
        nums.resize(n);
        for (int i = 0; i < n; ++i) {
            cin >> nums[i];
        }
        mergesort(nums);
        for (auto x: nums)
            cout << x << ' ';
        return 0;
    }
    ```

heap
??? "stl"
    ```cpp
    #include "bits/stdc++.h"

    using namespace std;

    vector<int> heap;

    int main() {
        int n;
        cin >> n;
        while (n--) {
            int op;
            cin >> op;
            if (op == 1) {
                int x;
                cin >> x;
                heap.push_back(x);
                push_heap(heap.begin(), heap.end(), greater<int>());
    //            make_heap(heap.begin(), heap.end(), greater<int>());
            }
            else if (op == 2)
                cout << heap.front() << endl;
            else {
                pop_heap(heap.begin(), heap.end(), greater<int>());
                heap.pop_back();
            }

        }
        return 0;
    }
    ```

## thought

### Q: 为什么区间dp先枚举长度再枚举左端点

A: 防止用到还没算好的小区间 

eg:[最长回文字串"aaaaa"](https://leetcode.cn/problems/longest-palindromic-substring/description/)

用 `dp[0][4]` 的时候应该先算 `dp[1][3]` ,但是先枚举左端点的话就没做到先算 `dp[1][3]`

??? "wrong answer"
    ```C++
    class Solution {
    public:
        string longestPalindrome(string s) {

            vector<vector<bool>> dp(s.size(), vector(s.size(), false));
            pair<int, int> anw = {0, 1};
            for (int i = 0; i < s.size() - 1; ++i) {
                dp[i][i] = true;
                if (s[i] == s[i + 1]) {
                    dp[i][i + 1] = true;
                    anw = {i, 2};
                }
            }
            dp[s.size() - 1][s.size() - 1] = true;


            for (int l = 0; l < s.size(); ++l) {
                for (int len = 2; len <= s.size(); ++len) {
                    if(len+l-1>=s.size()) continue;
                    int r = l + len - 1;

                    if (dp[l + 1][r - 1] == true && s[l] == s[r]) {
                        dp[l][r] = true;
                        cout<<l<<' '<<r<<endl;
                        if (len > anw.second)
                            anw = {l, len};
                    }
                }
            }

            return s.substr(anw.first, anw.second);

        }
    };
    ```

??? "correct answer"
    ```C++
    class Solution {
    public:
        string longestPalindrome(string s) {
            if (s.size() == 0 || s.size() == 1) return s;
            vector<vector<bool>> dp(s.size(), vector<bool>(s.size()));
            int maxlen = 1, begin = 0;
            for (int i = 0; i < s.size(); i++) {
                dp[i][i] = true;
                if (i < s.size() - 1 && s[i] == s[i + 1]) {
                    dp[i][i + 1] = true;
                    begin = i;
                    maxlen = 2;
                }
            }

            for (int len = 2; len <= s.size(); len++)
                for (int i = 0; i + len - 1 < s.size(); i++) {
                    int r = i + len - 1;
                    if (s[i] == s[r] && dp[i + 1][r - 1] == true) {
                        dp[i][r] = true;
                        maxlen = len;
                        begin = i;
                    }
                }

            return s.substr(begin, maxlen);
        }
    };
    ```


### 记忆化搜索

[oi介绍](https://oi-wiki.org/dp/memo/#%E5%BC%95%E5%85%A5)

[最长上升子序列](https://www.acwing.com/problem/content/description/897/)

在原始dfs中，会出现很多重复没有用的计算。

求数字8的位置时，把之前的3，1，2等就重复算了。
```
7
3 1 2 1 8 5 6
```

??? "原始dfs"
    ```cpp
    #include "bits/stdc++.h"

    using namespace std;

    const int N = 1010;
    int res = 0;
    int a[N];
    int dp[N];
    int n;

    int solve(int po) {
        dp[po] = 1;
        for (int i = 0; i < po; ++i) {
            if (a[i] < a[po]) dp[po] = max(dp[po], solve(i) + 1);
        }
        return dp[po];
    }

    int main() {
        cin >> n;
        for (int i = 0; i < n; ++i) {
            cin >> a[i];
        }
        for (int i = 0; i < n; ++i) {
            res = max(res, solve(i));
        }
        cout << res;
        return 0;
    }
    ```

那么我们就存一个记忆化数组，如果之前算过了直接返回算好的值，否则，继续算

??? "记忆化"
    ```cpp
    #include "bits/stdc++.h"

    using namespace std;

    const int N = 1010;
    int res = 0;
    int a[N];
    int dp[N];
    int n;

    int solve(int po) {
        // 算好了就返回
        if (dp[po] != -1) return dp[po];

        dp[po] = 1;
        for (int i = 0; i < po; ++i) {
            if (a[i] < a[po]) dp[po] = max(dp[po], solve(i) + 1);
        }
        return dp[po];
    }

    int main() {

        ::memset(dp, -1, sizeof dp);


        cin >> n;
        for (int i = 0; i < n; ++i) {
            cin >> a[i];
        }
        for (int i = 0; i < n; ++i) {
            res = max(res, solve(i));
        }
        cout << res;
        return 0;
    }
    ```


