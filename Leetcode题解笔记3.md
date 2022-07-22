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



### ==子序列问题==

- 第一种动态规划方法是，定义一个 dp 数组，其中 dp[i] 表示以 i 结尾的子序列的性质。在处理好每个位置后，统计一遍各个位置的结果即可得到题目要求的结果。
- 第二种动态规划方法是，定义一个 dp 数组，其中 dp[i] 表示到位置 i 为止 的子序列的性质，并不必须以 i 结尾。这样 dp 数组的最后一位结果即为题目所求，不需要再对每 个位置进行统计。

- 一般对于子数组（要求完全连续）选第一种，对于子字符串（按顺序即可，不需要连续）选第二种



#### 300、最长递增子序列

- 思路是2重for循环的dp，因为需要考虑位置（可以不是连续的）
- 此处dp[i]的语义是以下标i为终点的子序列的性质，需要统计一遍结果

```c++
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int n = nums.size();
        vector<int> dp(n + 1, 1);
        dp[0] = 0;
        int ans = 0;
        for(int i = 1; i <= n; i++){
            for(int j = 1; j <= i; j++){
                if(nums[i - 1] > nums[j - 1]){
                    dp[i] = max(dp[i], dp[j] + 1);
                }
            }
            ans = max(dp[i], ans);
        }
        return ans;
    }
};
```



#### 1143、最长公共子序列

- 此处dp的语义：到第一个字符串位置 i 为止、到 第二个字符串位置 j 为止、最长的公共子序列长度。
- 直接返回dp（m，n）即为最终结果

```c++
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int m = text1.size(), n = text2.size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        for(int i = 1; i <= m; i++){
            for(int j = 1; j <= n; j++){
                if(text1[i - 1] == text2[j - 1]){
                    dp[i][j] = dp[i - 1][j - 1]  + 1;
                }else{
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[m][n];
    }
};
```



### ==背包问题（很关键）==

- 需要理解背包问题的定义，以便各种场景的转换（选或者不选）

- 有 N 个物品和容量为 W 的背包，每个物品都有 自己的体积 w 和价值 v，
- 求拿哪些物品可以使得背包所装下物品的总价值最大。
- 如果限定每种物 品只能选择 0 个或 1 个，则问题称为 0-1 背包问题；
- 如果不限定每种物品的数量，则问题称为无 界背包问题或完全背包问题。
- 关键在于以下的状态转移方程（==0-1背包问题==）

```c++
vector<vector<int>> dp(N + 1, vector<int>(W + 1, 0));
for (int i = 1; i <= N; ++i) {
	int w = weights[i-1], v = values[i-1];
	for (int j = 1; j <= W; ++j) {
		if (j >= w) {
			dp[i][j] = max(dp[i-1][j], dp[i-1][j-w] + v);
		} else {
			dp[i][j] = dp[i-1][j];
        }
    }
}
```

- 完全背包问题只需要修改一处即可（结合情景分析就很好理解）

```c++
dp[i][j] = max(dp[i-1][j], dp[i][j-w] + v);
```



#### 416、分割等和子集

- 关键在于将其化为背包问题

```c++
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int sum = accumulate(nums.begin(), nums.end(), 0);
        if(sum % 2) return false;
        int target = sum / 2;
        int n = nums.size();
        vector<vector<bool>> dp(n + 1, vector<bool>(target + 1, false));
        for(int i = 0; i <= n; i++){
            dp[i][0] = true;
        }
        for(int i = 1; i <= n; i++){
            for(int j = 1; j <= target; j++){
                if(j >= nums[i - 1]){
                    dp[i][j] = dp[i - 1][j - nums[i - 1]] || dp[i - 1][j];
                }else{
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }
        return dp[n][target];
    }
};
```



#### 474、一和零（==好好理解==）

- 该题的关键在于有两个背包（三维，可通过倒序遍历压缩空间，但我就用三维的学习）
- j和p建议从0开始（有时候是1，看情况），最好不要直接从zero/one开始（无非优化而已，有时候会错）
- 需要有分支
  - if(j >= zero && p >= one) 
  - else

```c++
class Solution {
public:
    int helper(string& str){
        int ret = 0; 
        for(int i = 0; i < str.size(); i++){
            if(str[i] == '0') ret++;
        }
        return ret;     
    }
    int findMaxForm(vector<string>& strs, int m, int n) {
        int k = strs.size();
        vector<vector<vector<int>>> dp(k + 1, vector<vector<int>>(m + 1, vector<int>(n + 1, 0)));
        for(int i = 1; i <= k; i++){
            int zero = helper(strs[i - 1]), one = strs[i - 1].size() - zero;
            for(int j = 0; j <= m; j++){
                for(int p = 0; p <= n; p++){
                    if(j >= zero && p >= one) dp[i][j][p] = max(dp[i - 1][j - zero][p - one] + 1, dp[i - 1][j][p]);
                    else dp[i][j][p] = dp[i - 1][j][p];
                }
            }
        }
        return dp[k][m][n];
    }
};
```



#### 322、零钱兑换

- 关键在于初值给INT_MAX会越界，所以给amount + 2即可（全用1元硬币达到amount也不会超过这个数）
- 最后判断是否更改了，没更改即不符合要求

```c++
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        int n = coins.size();
        vector<vector<int>> dp(n + 1, vector<int>(amount + 1, amount + 2));
        for(int i = 0; i <= n; i++){
            dp[i][0] = 0;
        }
        for(int i = 1; i <= n; i++){
            for(int j = 0; j <= amount; j++){
                if(j >= coins[i - 1]){
                    dp[i][j] = min(dp[i][j - coins[i - 1]] + 1, dp[i - 1][j]);
                }else{
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }
        return dp[n][amount] == amount + 2 ? -1 : dp[n][amount];
    }   
};
```



### ==字符串编辑==

#### 72、编辑距离（==边界条件非常关键==）

- 该题的关键在于需要从0开始比较，即没有字符的位置进行比较
- 将dp声明为size + 1，一般直接从1开始，但本题从0开始
- 因为1个空字符串可以通过增加变成另一个字符串（另一个字符串通过删除也可以）
- 以后都要仔细考虑0处的语义再决定是否需要（也就是边界条件非常关键）
- 一个为空字符串一个不为空时很好理解，需要的操作数就是不为空的字符串的长度

```c++
class Solution {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size(), n = word2.size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        for(int i = 0; i <= m; i++){
            for(int j = 0; j <= n; j++){
                if(i == 0) dp[i][j] = j;
                else if(j == 0) dp[i][j] = i;
                else{
                    dp[i][j] = min({dp[i - 1][j - 1] + (word1[i - 1] == word2[j - 1] ? 0 : 1), dp[i - 1][j] + 1, dp[i][j - 1] + 1});
                }
            }
        }
        return dp[m][n];
    }
};
```



#### 650、只有两个键的键盘

- 思路是简单的dp，但是状态转移是除法不是减法

```c++
class Solution {
public:
    int minSteps(int n) {
        if(n == 1) return 0;
        vector<int> dp(n + 1, n + 2);
        dp[1] = 0;
        for(int i = 2; i <= n; i++){
            for(int j = 1; j <= i; j++){
                if(i % j == 0){
                    dp[i] = min(dp[i], dp[j] + i / j);
                }
            }
        }
        return dp[n];
    }
};
```



### ==股票交易==

- 一般把dp定义为n长度

#### 121、买卖股票的最佳时机

- 思路就是维护一个哨兵（最小值），状态转移分为卖出和不卖出两种，取较大的那种

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        if(n == 0) return 0;
        vector<int> dp(n, 0);
        int pivot = prices[0];
        for(int i = 1; i < n; i++){
            pivot = min(pivot, prices[i]);
            dp[i] = max(dp[i - 1], prices[i] - pivot);
        }
        return dp[n - 1];
    }
};
```



#### 122、买卖股票的最佳时机2

- 思路是一旦prices[i] > prices[i - 1]就更新dp[i]，否则dp[i] = dp[i - 1]

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        vector<int> dp(n, 0);
        for(int i = 1; i < n; i++){
            if(prices[i] > prices[i - 1]){
                dp[i] = dp[i - 1] + prices[i] - prices[i - 1];
            }else{
                dp[i] = dp[i - 1];
            }
        }
        return dp[n - 1];
    }
};
```

- 这样的思路不是很模式化，选择修改，修改状态定义如下：

- dp(i，0)表示第 i 天交易完后手里没有股票的最大利润
- dp(i，1)表示第 i天交易完后手里持有一支股票的最大利润（i 从 0开始）
- 最大利润必然在最后手上不持有股票时取得

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        vector<vector<int>> dp(n, vector<int>(2, 0));
        dp[0][0] = 0, dp[0][1] = -prices[0];
        for(int i = 1; i < n; i++){
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
        }
        return dp[n - 1][0];
    }
};
```





#### 123、买卖股票的最佳时机3

- 关键在于找准状态的定义

- 一天一共就有五个状态，

  0. 没有操作

  1. 第一次买入（持有）
  2. 第一次卖出（未持有）
  3. 第二次买入（持有）
  4. 第二次卖出（未持有）

- 需要注意的是dp(i,1)表示第i天为止只进行了一次买入，并不是一定指当天买入

- 其他状态也类似，表示的是当前所处的状态，不一定是当天更改

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        vector<vector<int>> dp(n, vector<int>(5, 0));
        dp[0][0] = 0, dp[0][1] = -prices[0], dp[0][2] = 0, dp[0][3] = -prices[0], dp[0][4] = 0;
        for(int i = 1; i < n; i++){
            dp[i][0] = dp[i - 1][0];
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
            dp[i][2] = max(dp[i - 1][2], dp[i - 1][1] + prices[i]);
            dp[i][3] = max(dp[i - 1][3], dp[i - 1][2] - prices[i]);
            dp[i][4] = max(dp[i - 1][4], dp[i - 1][3] + prices[i]);
        }
        return dp[n - 1][4];
    }
};
```



#### 188、买卖股票的最佳时机4（==重要==）

- 是上一题的泛化版本，不好2层枚举需要用3层dp
- dp（i，j，k）表示第i天，交易了j次，当前是否持有（0未持有，1持有）
- 交易的次数指的是卖出了多少次
- 若卖出了5次，又再次买入，当前交易次数为5次（这次的交易未结束）
- 当前交易为j时的持有，可能是前天交易j的持有，或前天交易j的未持久在今日买入
  - 不能是前天交易j - 1在今日买入，这样的话买入这次未卖出，状态仍为j - 1次

```c++
class Solution {
public:
    int maxProfit(int k, vector<int>& prices) {
        if(prices.empty()) return 0;
        int n = prices.size();
        vector<vector<vector<int>>> dp(n, vector<vector<int>>(k + 1, vector<int>(2, 0)));
        for(int j = 0; j <= k; j++){
            dp[0][j][0] = 0;
            dp[0][j][1] = -prices[0];
        }
        for(int i = 1; i < n; i++){
            for(int j = 0; j <= k; j++){
                if(j == 0){
                    dp[i][j][0] = dp[i - 1][j][0];
                    dp[i][j][1] = max(dp[i - 1][j][0] - prices[i], dp[i - 1][j][1]);
                }else{
                    dp[i][j][0] = max(dp[i - 1][j][0], dp[i - 1][j - 1][1] + prices[i]);
                    dp[i][j][1] = max(dp[i - 1][j][1], dp[i - 1][j][0] - prices[i]);
                }
            }
        }
        int ans = 0;
        for(int j = 0; j <= k; j++){
            ans = max(ans, dp[n - 1][j][0]);
        }
        return ans;
    }
};
```



#### 309、最佳买卖股票时机含冷冻期

- 关键在于定义状态
- dp(i，0)表示未持有股票，且不处于冷冻期
- dp(i，1)表示持有股票，且不处于冷冻期
- dp(i，2)表示处于冷冻期（前一天卖出了股票）
- 最后的利润必然是不持有股票的两种情况下取到的最大值

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        vector<vector<int>> dp(n, vector<int>(3, 0));
        dp[0][1] = -prices[0];
        for(int i = 1; i < n; i++){
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][2]);
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
            dp[i][2] = dp[i - 1][1] + prices[i];
        }
        return max(dp[n - 1][0], dp[n - 1][2]);
    }
};
```



#### 714、买卖股票的最佳时机含手续费

- 与题122几乎一样，状态转移也类似
- dp(i，0)表示第 i 天交易完后手里没有股票的最大利润（i 从 0开始）
- dp(i，1)表示第 i天交易完后手里持有一支股票的最大利润（最大利润必然在最后手上不持有股票时取得）

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        int n = prices.size();
        vector<vector<int>> dp(n, vector<int>(2, 0));
        dp[0][0] = 0, dp[0][1] = -prices[0];
        for(int i = 1; i < n; i++){
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i] - fee);
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
        }
        return dp[n - 1][0];
    }
};
```



### 练习

#### 213、打家劫舍2

- 思路是分两个区间分别进行dp，最后取两个结果中那个的较大值

```c++
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        if(n == 1) return nums[0];
        if(n == 2) return max(nums[0], nums[1]);
        vector<int> dp(n + 1, 0);
        dp[0] = 0, dp[1] = nums[0], dp[2] = max(nums[0], nums[1]);
        for(int i = 3; i < n; i++){
            dp[i] = max(dp[i - 1], dp[i - 2] + nums[i - 1]);
        }
        int tmp = dp[n - 1];
        dp = vector<int>(n + 1, 0);
        dp[2] = nums[1], dp[3] = max(nums[1], nums[2]);
        for(int i = 4; i <= n; i++){
            dp[i] = max(dp[i - 1], dp[i - 2] + nums[i - 1]);
        }
        int ans = max(tmp, dp[n]);
        return ans;
    }
};
```



#### 53、最大子数组和

- 关键是确定状态定义以及找到状态转移方程
  - 状态dp[i]表示以当前下标元素为结尾的子数组的性质
  - 状态转移方程为dp[i] = max(dp[i - 1] + nums[i], nums[i])

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int n = nums.size();
        if(n == 0) return 0;
        vector<int> dp(n, 0);
        dp[0] = nums[0];
        int ans = nums[0];
        for(int i = 1; i < n; i++){
            dp[i] = max(dp[i - 1] + nums[i], nums[i]);
            ans= max(ans, dp[i]);
        }
        return ans;
    }
};
```



#### 343、整数拆分

- 关键思路是找到其状态转移方程

```c++
class Solution {
public:
    int integerBreak(int n) {
        vector<int> dp(n + 1, 0);
        dp[0] = 0, dp[1] = 0;
        for(int i = 2; i <= n; i++){
            for(int j = 1; j < i; j++){
                dp[i] = max({dp[i], (i - j) * j, j * dp[i - j]});
            }
        }
        return dp[n];
    }
};
```



#### 583、两个字符串的删除操作

- 思路就是简单的二维dp，涉及字符串的大多数情况设置为n + 1比较合适

```c++
class Solution {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size(), n = word2.size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        for(int i = 1; i <= m; i++){
            dp[i][0] = i;
        }
        for(int j = 1; j <= n; j++){
            dp[0][j] = j;
        }
        for(int i = 1; i <= m; i++){
            for(int j = 1; j <= n; j++){
                if(word1[i - 1] == word2[j - 1]){
                    dp[i][j] = dp[i - 1][j - 1];
                }else{
                    dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + 1;
                }
            }
        }
        return dp[m][n];
    }
};
```



#### 646、最长数对链

- 由于其可以任意选择顺序，所以先简单排序在进行dp
- 这种数字序列求最长，常选择dp[i]为以i为结尾的子序列性质，并且通过两层for求解一维dp

```c++
class Solution {
public:
    int findLongestChain(vector<vector<int>>& pairs) {
        int n = pairs.size();
        sort(pairs.begin(), pairs.end(), [](vector<int>& a, vector<int>& b){
            return a[0] < b[0];
        });
        vector<int> dp(n, 1);
        int ans = 0;
        for(int i = 0; i < n; i++){
            for(int j = 0; j <= i; j++){
                if(pairs[i][0] > pairs[j][1]){
                    dp[i] = max(dp[i], dp[j] + 1);
                }
            }
            ans = max(ans, dp[i]);
        }
        return ans;
    }
};
```



#### 376、摆动序列

- 思路是定义两个dp数组（参考的答案）up及down
  - up为上升摆动序列，即结尾大于前一个元素
  - down为下降摆动序列，即结尾小于前一个元素
  - 特殊的，长度为1的序列即是上升摆动序列也是下降摆动序列

```c++
class Solution {
public:
    int wiggleMaxLength(vector<int>& nums) {
        int n = nums.size();
        if(n < 2) return n;
        vector<int> up(n), down(n);
        up[0] = down[0] = 1;
        for(int i = 1; i < n; i++){
            if(nums[i] > nums[i - 1]){
                up[i] = max(up[i - 1], down[i - 1] + 1);
                down[i] = down[i - 1];
            }else if(nums[i] < nums[i - 1]){
                up[i] = up[i - 1];
                down[i] = max(down[i], up[i - 1] + 1);
            }else{
                up[i] = up[i - 1];
                down[i] = down[i - 1];
            }
        }
        return max(up[n - 1], down[n - 1]);
    }
};
```

