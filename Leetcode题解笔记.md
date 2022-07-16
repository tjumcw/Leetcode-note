# Leetcode题解笔记

### 一、贪心

#### 455、分发饼干

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



#### 135、分发糖果

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



#### 763、划分字母区间

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



#### 406、根据身高重建队列

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

#### 167、两数之和

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





#### 88、合并两个有序数组

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



#### 142、环形链表3

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



#### 76、最小覆盖字串

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



#### 633、平方数之和

- 我的思路是

```c++
class Solution {
public:
    bool judgeSquareSum(int c) {
        int r = sqrt(c), l = 0;
        while(l <= r){
            if(c - l * l == r * r) return true;
            else if(c - l * l < r * r) r--;
            else l++;
        }
        return false;
    }
};
```



#### 680、验证回文字符串2==（关键）==

- 我的思路是双指针，遇到不对判断移动哪根指针（462 / 469）

```c++
class Solution {
public:
    bool validPalindrome(string s) {
        int n = s.size();
        if(n <= 2) return true;
        int l = 0, r = n - 1;
        int cnt = 0;
        while(l <= r){			//=没啥意义，因为偶数不会出现，奇数中间一个不用管
            if(s[l] == s[r]){
                l++;
                r--;
            }else{
                if(s[l + 1] == s[r]){
                    l++;
                    cnt++;
                }
                else if(s[r - 1] == s[l]){
                    r--;
                    cnt++;
                }else{
                    l++;
                    cnt++;
                }
            }
            if(cnt > 1) return false;
        }
        return true;
    }
};
```

- 回文字符串很关键的方法，参考答案的思路为按照不允许删除字符处理

```c++
//关键在于s[l] != s[r]时，对s[l + 1, r]和s[l, r - 1]两个区间判断是否回文即可

class Solution {
public:
    bool check(string s, int l, int r){
        while(l < r){
            if(s[l] != s[r]) return false;
            l++;
            r--;
        }
        return true;
    }
    bool validPalindrome(string s) {
        int l = 0, r = s.size() - 1;
        while(l < r){
            if(s[l] == s[r]){
                l++;
                r--;
                continue;
            }else{
                return check(s, l + 1, r) || check(s, l, r - 1);
            }
        }
        return true;
    }
};
```

- 其中，一旦s[low]!=s[high]，对两个字区间判断的结果直接return，因为只能修改一个位置



#### 524、通过删除字母匹配到字典最长单词

- 我的思路是双指针，分别对每个dict中的单词与字符串s进行比较（看代码即可理解）

```c++
class Solution {
public:
    string findLongestWord(string s, vector<string>& dictionary) {
        string ans = "";
        int len = 0;
        for(int i = 0; i < dictionary.size(); i++){
            int p = 0, q = 0;
            while(p < dictionary[i].size() && q < s.size()){
                if(dictionary[i][p] == s[q]){
                    p++;
                    q++;
                }else{
                    q++;
                }
            }
            if(p == dictionary[i].size()){
                ans = (dictionary[i].size() > ans.size() || (dictionary[i].size() == ans.size() && dictionary[i] <= ans)) ? dictionary[i] : ans;
            }
        } 
        return ans;
    }
};
```



### 三、二分查找

==推荐写法（nums.size() == 1时需要额外判断）==

```c++
int l = 0, r = nums.size() - 1;
while(l <= r){
    //...
    l = mid + 1;
    r = mid - 1;
    //...
}
```

- 具体需要结合情况分析（找几个例子模拟一下题意）

- 其中while的条件能否取等号的关键在于（取到等号时的那一次while循环有没有必要）：

```c++
//实际上哪怕取下面这一套写法，最后l必然也会等于r，但区别在于等于r之后直接while不满足退出了
//若nums[mid]在l == r时的值可能是需要返回的答案，取while(l <= r)比较好，实际可以都这么写，特殊情况再分析
while(l < r){
	l = mid + 1;
    r = mid;
}
```





#### 69、x的平方根

- 我的思路有问题，参考了答案，主要就是向下取整（只在<=时更改ans）

```c++
class Solution {
public:
    int mySqrt(int x) {
        if(x == 1) return 1;
        int l = 0, r = x, ans = 0;
        while(l < r){
            int mid = l + (r - l) / 2;
            if((long)mid * mid <= x){
                l = mid + 1;
                ans = mid;
            }else{
                r = mid;
            }
        }
        return ans;
    }
};
```

- 这个ans在很多二分查找的题中要用到，表示满足条件时先记录的值，因为这题是从小到大去逼近



#### 34、在排序数组中找第一个和最后一个target（==关键==）

- 我的思路是实现louerBound和upperBound两个函数即可（==向上取整==）

```c++
class Solution {
public:
    int myUpperBound(vector<int>& nums, int target){
        int l = 0, r = nums.size() - 1, mid = 0;
        while(l < r){
            mid = l + (r - l + 1) / 2;
            if(nums[mid] <= target){
                l = mid;

            }else{
                r = mid - 1;
            }
        }
        return l;
    }
    int myLowerBound(vector<int>& nums, int target){
        int l = 0, r = nums.size() - 1, mid = 0;
        while(l < r){
            mid = l + (r - l) / 2;
            if(nums[mid] >= target){
                r = mid;
            }else{
                l = mid + 1;
            }
        }
        return l;
    }
    vector<int> searchRange(vector<int>& nums, int target) {
        if(nums.empty()) return {-1, -1};
        int l = myLowerBound(nums, target);
        int r = myUpperBound(nums, target);
        if(nums[l] != target) return {-1, -1};
        return {l ,r};
    }
};
```

- ==二分查找的关键就是需要对照题目找实例看是否能满足==



#### 81、搜索旋转排序数组2（==很关键，仔细揣摩==）

- 我的主要思路是判断nums[l]和nums[mid]大小：
  - 若nums[mid]较小，则必然在右半部分（折断后的后面，值比较小），此时右半区间是有序的
    - 若此时nums[mid] < target，target可能在左半部分也可能在右半部分
      - 判断，若target处于有序区间即右半区间内，l = mid + 1
    - 其余情况直接都需要在左边解决，取r = mid即可
  - 若nums[mid]较大，则必然在左半部分（折断后的前面，值比较大），此时左半区间是有序的
    - 情况和上述同理

- 一个关键点在于其非递减，所以nums[l] == nums[mid]时无法判断区间，做额外处理(l++)

```c++
class Solution {
public:
    bool search(vector<int>& nums, int target) {
        if(nums.size() == 1){
            if(nums[0] == target) return true;
            else return false;
        }
        int l = 0, r = nums.size() - 1, mid = 0;
        while(l <= r){
            mid = l + (r - l) / 2;
            if(nums[mid] == target) return true;
            if(nums[mid] == nums[l]){
                l++;
            }else if(nums[mid] < nums[l]){
                if(nums[mid] < target && target <= nums[r]){
                    l = mid + 1;
                }else{
                    r = mid - 1;
                }
            }else{
                if(nums[mid] > target && target >= nums[l]){
                    r = mid - 1;
                }else{
                    l = mid + 1;
                }
            }
        }
        return false;
    }
};
```



#### 154、寻找旋转排序数组中的最小值2

- 根据上述思路画图并比对具体例子调试（自己写的代码）如下：

```c++
class Solution {
public:
    int findMin(vector<int>& nums) {
        int l = 0, r = nums.size() - 1, mid;
        while(l < r){
            mid = l + (r - l) / 2;
            if(nums[mid] == nums[l]){
                if(nums[l] < nums[r]) return nums[l];
                l++;
            }
            else if(nums[mid] < nums[l]){
                r= mid;
            }else{
                if(nums[mid] > nums[r]){
                    l = mid + 1;
                }else{
                    r = mid;
                }
            }
        }
        return nums[l];		//此处的l和r均可，因为这种while(l < r)最后l == r
    }
};
```

- 这题就是不能取等于号的情形，因为根据二分最后的结果必然是最小的，若取等号则还会再进入一次while循环更改最终的l/r，把对的下标改错了
