# Leetcode题解笔记

## 六、动态规划（==重要==）

### ==一维dp==

#### 70、爬楼梯

- 较为简单的动态规划问题，关键在于写出状态转移方程dp[i] = dp[i - 1] + dp[i - 2]

```c++
class Solution {
public:
    int climbStairs(int n) {
        if(n <= 2) return n;
        vector<int> dp(n + 1, 0);
        dp[1] = 1;
        dp[2] = 2;
        for(int i = 3; i <= n; i++){
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        return dp[n];
    }
};
```



- 还可以压缩空间实现如下：

```c++
class Solution {
public:
    int climbStairs(int n) {
        int p = 0, q = 0, ans = 1;
        for(int i = 1; i <= n; i++){
            p = q;
            q = ans;
            ans = p + q;
        }
        return ans;
    }
};
```



#### 198、打家劫舍

- 状态转移方程为：dp[i] = max(dp[i - 1], dp[i - 2] + nums[i - 1])

```c++
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        vector<int> dp(n + 1);
        dp[0] = 0;
        dp[1] = nums[0];
        for(int i = 2; i <= n; i++){
            dp[i] = max(dp[i - 1], dp[i - 2] + nums[i - 1]);
        }
        return dp[n];
    }
};
```



#### 413、等差数列划分

- 该题主要要想清楚一个点，首先明确dp[i]定义以该元素为结尾的子数组的等差数列个数
- 其次，需要明确若dp[i - 1]不为0，即以i - 1为结尾子数组有等差数列
- 则dp[i ] = do[i - 1] + 1
  - 其中1表示原来数列再加1位这个新数列，为什么还有dp[i - 1]个呢
  - 理解成所有原来的子数列的第一个元素都没了，但是新加了最后一个新的元素（总个数不变）
- 则就这两个子数组而言就有dp[i - 1] + dp[i]个等差数列了，对所有dp[i]需要求和

```c++
class Solution {
public:
    int numberOfArithmeticSlices(vector<int>& nums) {
        int n = nums.size();
        if(n < 3) return 0;
        vector<int> dp(n, 0);
        int ans = 0;
        for(int i = 2; i < n; i++){
            if(nums[i] + nums[i - 2] == 2 * nums[i - 1]){
                dp[i] = dp[i - 1] + 1;
            }
        }
        return accumulate(dp.begin(), dp.end(), 0);
    }
};
```



### ==二维dp==

#### 64、最小路径和

- 本题的难度不大，但是处理起来需要判断各种边界条件（实际上我做了优化）
- 就是选择vector<vector<int>> dp(m + 1, vector<int>(n + 1, INT_MAX))这种声明方式
- 并且对于特殊值：dp（0， 1）和dp（1，0）做了初始化，其余全部最大，保证不影响状态转移方程
- 因为只有一个入口（起始点）即i = 1, j = 1（对应于grid左上角的元素）
- 只要保证该点的状态dp是正确的，并且其他点的边界条件不会影响dp的判定即可

```c++
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, INT_MAX));
        dp[0][1] = dp[1][0] = 0;
        for(int i = 1; i <= m; i++){
            for(int j = 1; j <= n; j++){
                dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + grid[i - 1][j - 1];
            }
        }
        return dp[m][n];
    }
};
```



#### 542、01矩阵

- 我的思路是对其进行两次扫描，一次右下，一次左上（四个方向不好直接一次dp解决）
- i，j分开判断，若是都满足会取较小的那个
- 若是为左上角0，0或者右下角m-1，n-1，则会交给另外一个方向的dp赋值

```c++
class Solution {
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
        int m = mat.size(), n = mat[0].size();
        vector<vector<int>> dp(m, vector<int>(n, INT_MAX / 2));
        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                if(mat[i][j] == 0){
                    dp[i][j] = 0;
                }else{
                    if(i > 0){
                        dp[i][j] = min(dp[i][j], dp[i - 1][j] + 1);
                    }
                    if(j > 0){
                        dp[i][j] = min(dp[i][j], dp[i][j - 1] + 1);
                    }
                }
            }
        }
        for(int i = m - 1; i >= 0; i--){
            for(int j = n - 1; j >= 0; j--){
                if(mat[i][j] != 0){
                    if(i < m - 1){
                        dp[i][j] = min(dp[i][j], dp[i + 1][j] + 1);
                    }
                    if(j < n - 1){
                        dp[i][j] = min(dp[i][j], dp[i][j + 1] + 1);
                    }
                }
            }
        }
        return dp;
    }
};
```



#### 221、最大正方形

- 我没啥好的思路（看的参考答案），关键思想在于：
- 假设 dp(i, j)= k，其充分条件为 dp(i - 1, j - 1)、dp(i, j - 1) 和 dp(i - 1, j)的值必须 都不小于k - 1
- 其中，k表示边长，若仅有1个1则k = 1，画个图就比较好理解（参考LeetCode 101）

```c++
class Solution {
public:
    int maximalSquare(vector<vector<char>>& matrix) {
        int m = matrix.size(), n = matrix[0].size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1 ,0));
        int num = 0;
        for(int i = 1; i <= m; i++){
            for(int j = 1; j <= n; j++){
                if(matrix[i - 1][j - 1] == '1'){
                    dp[i][j] = min({dp[i - 1][j - 1], dp[i - 1][j], dp[i][j - 1]}) + 1;
                }
                num = max(num, dp[i][j]);
            }
        }
        return num * num;
    }
};
```



### ==分割类型==

- 对于分割类型题，动态规划的状态转移方程通常并不依赖相邻的位置，而是依赖于满足分割 条件的位置。
- 可以参看279和139理解

#### 279、完全平方数

- 思路是dp，主要关键就在于边界条件的给定，dp[0] = 0作为dp的起点

```c++
class Solution {
public:
    int numSquares(int n) {
        vector<int> dp(n + 1, INT_MAX);
        dp[0] = 0;
        for(int i = 1; i <= n; i++){
            for(int j = 1; j * j <= i; j++){
                dp[i] = min(dp[i], dp[i - j * j] + 1);
            }
        }
        return dp[n];
    }
};
```



#### 91、解码方法（==细节较多==）

- 我的思路不太全，参考了答案

```c++
class Solution {
public:
    int numDecodings(string s) {
        int n = s.size();
        vector<int> dp(n + 1, 1);
        if(s[0] - '0' == 0) return false;
        for(int i = 2; i <= n; i++){
            int num = s[i - 1] - '0';
            int prev = s[i - 2] - '0';
            if((prev == 0 || prev > 2) && num == 0) return 0;
            if((prev > 0 && prev < 2) || (prev == 2 && num < 7)){
                if(num == 0){
                    dp[i] = dp[i - 2];
                }else{
                    dp[i] = dp[i - 1] + dp[i - 2];
                }
            }else{
                dp[i] = dp[i - 1];
            }
        }
        return dp[n];
    }
};
```



#### 139、单词拆分（类似完全平方数分割问题）

- 思路类似完全平方数

```c++
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        int n = s.size();
        vector<int> dp(n + 1, false);
        dp[0] = true;
        for(int i = 1; i <= n; i++){
            for(const auto& word : wordDict){
                int n = word.size();
                if(i >= n && s.substr(i - n, n) == word){
                    dp[i] = d[i] || dp[i - n];
                }
            }
        }
        return dp[n];
    }
};
```

