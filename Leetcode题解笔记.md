# Leetcode题解笔记

### 一、贪心

- #### 455、分发饼干

- 我的做法是排序+双指针遍历，利用贪心思想，先拿最小尺寸的饼干满足要求最小的孩子，依次遍历

```c++
class Solution {
public:
    int findContentChildren(vector<int>& g, vector<int>& s) {
        int p = 0;
        int m = g.size(), n = s.size();
        sort(g.begin(), g.end());
        sort(s.begin(), s.end());
        int ans = 0;
        for(int i = 0; i < m && p < n; i++){
            while(p < n && g[i] > s[p]){
                p++;
            }
            if(p == n){
                break;
            }
            ans++;
            p++;
        }
        return ans;
    }
};
```

- 可以进行优化，因为for和while可以合并在一起（两个数组只要有一个到头就结束遍历）

```c++
class Solution {
public:
    int findContentChildren(vector<int>& g, vector<int>& s) {
        int m = 0, n = 0;
        sort(g.begin(), g.end());
        sort(s.begin(), s.end());
        while(m < g.size() && n < s.size()){
            if(g[m] <= s[n]) ++m;
            ++n;
        }
        return m;
    }
};
```



- #### 135、分发糖果

- 我的做法是正反遍历，每次都找局部最优（反向遍历时需要做额外的判断，因为正向遍历有初步结果）

```c++
class Solution {
public:
    int candy(vector<int>& ratings) {
        int n = ratings.size();
        vector<int> nums(n, 1);
        for(int i = 1; i < n; ++i){
            if(ratings[i] > ratings[i - 1]){
                nums[i] = nums[i - 1] + 1;
            }
        }
        for(int i = n - 2; i >= 0; --i){
            if(ratings[i] > ratings[i + 1]){
                nums[i] = max(nums[i], nums[i + 1] + 1);
            }
        }
        for(auto a : nums){
            cout<<a<<" ";
        }
        int ans =  accumulate(nums.begin(), nums.end(), 0);
        return ans;
    }
};
```

- 我的做法很类似答案的做法，就不把答案做法贴上来了



#### 435、无重叠区间

- 我的思路，对区间排序后，依次比较两区间是否有交集（特意尝试了lambda表达式写在外面）

```c++
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        auto cmp = [](vector<int>& a, vector<int>& b) -> bool{
            if(a[0] == b[0]){
                return a[1] < b[1];
            }
            return a[0] < b[0];
        };
        sort(intervals.begin(), intervals.end(), cmp);
        for(auto a : intervals){
            cout<<a[0]<<":"<<a[1]<<" ";
        }
        cout<<endl;
        int pivot = 0, ans = 0;
        for(int i = 1; i < intervals.size(); i++){
            if(intervals[i][0] < intervals[pivot][1]){
                ++ans;
                pivot = intervals[i][1] < intervals[pivot][1] ? i : pivot;
            }else{
                pivot = i;
            }
        }
        return ans;
    }
};
```

- 答案方法差不多，但是是按照区间尾端进行排序的（时间和空间复杂度都更低）

```c++
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        if(intervals.size() <= 1){
            return 0;
        }
        int n = intervals.size();
        sort(intervals.begin(), intervals.end(), [](vector<int>& a, vector<int>& b){
            return a[1] < b[1];
        });
        int prev = intervals[0][1], ans = 0;
        for(int i = 1; i < n; i++){
            if(intervals[i][0] < prev) ans++;
            else prev = intervals[i][1];
        }
        return ans;
    }
};
```





#### 605、种花问题

- 我的思路就是简单的一个一个位置判断，满足则插入

```c++
class Solution {
public:
    bool canPlaceFlowers(vector<int>& flowerbed, int n) {
        if(n == 0) return true;
        int cnt = 0;
        int len = flowerbed.size();
        if(n == 1 && len == 1) return flowerbed[0] == 0;
        for(int i = 0; i < len; i++){
            if(flowerbed[i] == 0){
                if(i == 0){
                    if(flowerbed[i + 1] == 1) continue;
                    cnt++;
                    flowerbed[i] = 1;
                    continue;
                }
                if(i == len - 1){
                    if(flowerbed[i - 1] == 1) continue;
                    cnt++;
                    flowerbed[i] = 1;
                    continue;
                }
                if(flowerbed[i - 1] == 0 && flowerbed[i + 1] == 0){
                    cnt++;
                    flowerbed[i] = 1;
                }
            }
        }
        return cnt >= n;
    }
};
```



#### 452、用最少数量的箭射爆气球

- 我的思路是理解成435问题的差集（所有重叠子区间都可以通过一箭解决，无需关注谁重叠谁）
  - 只要求出去除最少区间能使无重叠的数量即可（因为无重叠的每箭只能射破一个气球）
  - 画图，分别画出下面两种情况分析即可理解
    - [[10,16],[2,8],[1,6],[7,12]]
    - [[14,16],[2,8],[1,6],[7,12]]

```c++
class Solution {
public:
    int findMinArrowShots(vector<vector<int>>& points) {
        int n = points.size();
        sort(points.begin(), points.end(), [](vector<int>& a, vector<int>& b){
            return a[1] < b[1];
        });
        int prev = points[0][1], cnt = 0;
        for(int i = 1; i < points.size(); i++){
            if(points[i][0] <= prev) cnt++;
            else prev = points[i][1];
        }
        return n - cnt;
    }
};
```



- #### 763、划分字母区间

- 我的思路是从前往后一块一块满足（利用贪心找当前最优，一旦满足就加入结果），一直迭代往后找

```c++
class Solution {
public:
    vector<int> partitionLabels(string s) {
        map<char, int> dict;
        for(int i = 0; i < s.size(); i++){
            dict[s[i]] = i;
        }
        int i = 0;
        char ch = s[0];
        vector<int> ans;
        while(i < s.size()){        
            for(; i <= dict[ch]; i++){
                if(dict[s[i]] > dict[ch]){
                    ch = s[i];
                    break;
                }
            }    
            if(i - 1 == dict[ch]){
                ans.push_back(i);
                ch = s[i];
            }     
        }
        for(int i = ans.size() - 1; i > 0; i--){
            ans[i] -= ans[i - 1];
        }
        return ans;
    }
};
```

- 答案的方法在代码结构上比我优化了，实际上思想差不多

```c++
class Solution {
public:
    vector<int> partitionLabels(string s) {
        int last[26];
        int length = s.size();
        for (int i = 0; i < length; i++) {
            last[s[i] - 'a'] = i;
        }
        vector<int> partition;
        int start = 0, end = 0;
        for (int i = 0; i < length; i++) {
            end = max(end, last[s[i] - 'a']);
            if (i == end) {
                partition.push_back(end - start + 1);
                start = end + 1;
            }
        }
        return partition;
    }
};
```



#### 122、买卖股票的最佳时机2

- 我的思路是（直接看代码就能理解我的思路了）

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        for(int i = n - 1; i > 0; i--){
            prices[i] -= prices[i - 1];
        }
        int ans = 0;
        for(int i = 1; i < n; i++){
            if(prices[i] > 0) ans += prices[i];
        }
        return ans;
    }
};
```



- #### 406、根据身高重建队列

- 没想出好方法，但是参考答案后发现该题关键在于排序，以身高降序，保证前面的都比自己高，那么只要插在第二个元素对应的索引上即是其正确的位置。身高相同的，位次低的在前面

```c++
class Solution {
public:
    static bool cmp(vector<int> a, vector<int> b) {
        if(a[0] == b[0]) return a[1] < b[1];
        return a[0] > b[0];
    }
    vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
        sort(people.begin(),people.end(),cmp);
        vector<vector<int>> result;

        for(int i = 0; i < people.size(); i++) {
            int index = people[i][1];
            result.insert(result.begin()+index, people[i]);
        }
        return result;
    }
};

```



#### 665、非递减数列

- 我的思路是各试一次，利用is_sorted（参考答案了，很暴力法）

```c++
class Solution {
public:
    bool checkPossibility(vector<int> &nums) {
        int n = nums.size();
        for (int i = 0; i < n - 1; ++i) {
            int x = nums[i], y = nums[i + 1];
            if (x > y) {
                nums[i] = y;
                if (is_sorted(nums.begin(), nums.end())) {
                    return true;
                }
                nums[i] = x; // 复原
                nums[i + 1] = x;
                return is_sorted(nums.begin(), nums.end());
            }
        }
        return true;
    }
};
```



### 二、双指针

- #### 167、两数之和

- 我的想法是：

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        int l = 0, r = numbers.size() - 1;
        while(l < r){
            if(numbers[l] + numbers[r] == target) return {l + 1, r + 1};
            if(numbers[l] + numbers[r] < target) l++;
            else{
                r--;
            }
        }
        return {-1, -1};
    }
};
```





- #### 88、合并两个有序数组

- 我的思路是双指针：

```c++
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        int l = m, r = n;
        while(l != 0 && r != 0){
            if(nums1[l - 1] > nums2[r - 1]){
                cout<<"nums1:"<<nums1[l-1]<<endl;
                nums1[l + r - 1] = nums1[l - 1];
                l--;
            }
            else{
                cout<<"nums2:"<<nums2[r-1]<<endl;
                nums1[l + r - 1] = nums2[r - 1];
                r--;
            }
        }
        if(l == 0){
            for(int i = 0; i < r; i++){
                nums1[i] = nums2[i];
            }
        }else if(r == 0){
            for(int i = 0; i < l; i++){
                nums1[i] = nums1[i];
            }
        }
    }
};
```



- #### 142、环形链表3

- 我的思路是快慢指针（一个一次2步一个一次1步，遇见时让快指针回到头节点并让其也一次走1步，相遇即环路开始点）：

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if(!head) return NULL;
        ListNode* fast = head;
        ListNode* slow = head;
        if(fast->next){
            fast = fast->next->next;
        }
        slow = slow->next;
        while(fast != slow){
            if(!fast || !fast->next) return NULL;
            fast = fast->next->next;
            slow = slow->next;
        }
        fast = head;
        while(fast != slow){
            fast = fast->next;
            slow = slow->next;
        }
        return fast;
    }
};
```



- #### 76、最小覆盖字串

- 我的思路是滑动窗口（超时了，265/266）

```c++
class Solution {
public:
    map<char, int> tmp;
    map<char, int> dict;
    bool helper(){
        for(const auto& a : tmp){
            if(dict[a.first] < a.second) return false;
        }
        return true;
    }
    string minWindow(string s, string t) {
        for(auto a : t){
            tmp[a]++;
        }
        int l = 0, r = 0;
        vector<string> str;
        while(l <= r && r < s.size()){
            if(!helper()){
                dict[s[r++]]++;
            }
            else{
                str.push_back(string(s.begin() + l, s.begin() + r));
                dict[s[l]]--;
                l++;
            }
        }
        while(helper()){
            dict[s[l]]--;
            l++;
        }
        if(l != 0){
                cout<<string(s.begin() + l - 1, s.begin() + r)<<endl;
                str.push_back(string(s.begin() + l - 1, s.begin() + r));
        }
        if(str.size() != 0){
            sort(str.begin(), str.end(), [](string& a, string& b){
                return a.size() < b.size();
            });
            return str[0];
        }
        return "";
    }
};
```

