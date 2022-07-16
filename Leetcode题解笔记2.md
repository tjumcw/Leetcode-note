# Leetcode题解笔记

### 四、排序

#### 215、数组中的第K个最大元素（==快速选择==）

- 就是双指针结合快速排序的思想

```c++
class Solution {
public:
    int quickSelect(vector<int>& nums, int l, int r){
        int first = l, last = r;
        int pivot = nums[first];
        while(first < last){
            while(first < last && nums[last] >= pivot) last--;
            nums[first] = nums[last];
            while(first < last && nums[first] <= pivot) first++;
            nums[last] = nums[first];
        }
        nums[first] = pivot;
        return first;
    }
    int findKthLargest(vector<int>& nums, int k) {
       int target = nums.size() - k;
       int l = 0, r = nums.size() - 1, mid = 0;
       while(l < r){
           mid = quickSelect(nums, l , r);
           if(mid == target) return nums[mid];
           else if(mid < target) l = mid + 1;
           else r= mid;
       } 
       return nums[l];
    }
};
```



#### 347、前k个高频元素（==桶排序==）

- 桶排序

```c++
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        map<int, int> dict;
        int cnt = 0;
        for(const auto& num : nums){
            cnt = max(cnt, ++dict[num]);
        }
        vector<vector<int>> buckets(cnt + 1);
        for(const auto& ele : dict){
            buckets[ele.second].push_back(ele.first);
        }
        vector<int> ans;
        for(int i = 0; i <= cnt; i++){
            for(const auto& num : buckets[cnt - i]){
                if(ans.size() == k) return ans;
                ans.push_back(num);
            }
        }
        return ans;
    }
};
```



#### 451、根据字符出现频率排序

- 桶排序

```c++
class Solution {
public:
    string frequencySort(string s) {
        map<char, int> dict;
        int cnt = 0;
        for(const auto& ch : s){
            cnt = max(cnt, ++dict[ch]);
        }
        vector<vector<char>> buckets(cnt + 1);
        for(const auto& ele : dict){
            buckets[ele.second].push_back(ele.first);
        }
        string ans = "";
        for(int i = cnt; i >= 1; i--){
            for(const auto& ch : buckets[i]){
                for(int j = 0; j < i; j++){
                    ans += ch;
                }
            }
        }
        return ans;
    }
};
```



### 五、==搜索==

#### 695、岛屿的最大面积（==经典题==）

- 我的思路是深度优先搜索，搜过的地方直接置为0，先判断再搜索
  - 判断是否进行dfs/bfs条件时，边界条件要放在grid对应节点值==1之前（条件判断从左往右，放左边可能越界）
  - bfs关注修改状态的时机（该题为push进去的时候直接修改）

```c++
//dfs版本
class Solution {
public:
    vector<int> direction{-1, 0, 1, 0, -1};
    int dfs(vector<vector<int>>& grid, int r, int c){
        int cnt = 1;
        grid[r][c] = 0;
        for(int i = 0; i < 4; i++){
            int x = r + direction[i], y = c + direction[i + 1];
            if(x >= 0 && x < grid.size() && y >= 0 && y < grid[0].size() && grid[x][y] == 1){
                cnt += dfs(grid, x, y);
            }
        }
        return cnt;
    }
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        int ans = 0;
        for(int i = 0; i < grid.size(); i++){
            for(int j = 0; j < grid[0].size(); j++){
                if(grid[i][j] == 1){
                    ans = max(ans, dfs(grid, i, j));
                }
            }
        }
        return ans;
    }
};

```



```c++
//bfs版本
class Solution {
public:
    vector<int> direction{-1, 0, 1, 0, -1};
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        queue<pair<int, int>> q;
        int ans = 0;
        for(int i = 0; i < grid.size(); i++){
            for(int j = 0; j < grid[0].size(); j++){
                if(grid[i][j] == 1){
                    q.push({i, j});
                    int cnt = 1;
                    grid[i][j] = 0;
                    while(!q.empty()){
                        auto node = q.front();
                        q.pop();
                        for(int i = 0; i < 4; i++){
                            int x = node.first + direction[i], y = node.second + direction[i + 1];
                            if(x >= 0 && x < grid.size() && y >= 0 && y < grid[0].size() && grid[x][y] == 1){
                                q.push({x, y});
                                grid[x][y] = 0;
                                cnt++;
                            }
                        }
                    }
                    ans = max(ans, cnt);
                }
            }
        }
        return ans;
    }
};
```



#### 547省份数量（==关键题，理解搜索和图论==）

- 我的理解是其本质和上一题的岛屿题相同，不同的是节点及其延伸出的方向

  - 对于题目 695，图的表示方法是，每个位置代表一个节点，每个节点与上下左右四个节点相邻。

  - 而在这一道题里面，每一行（列）表示一个节点，它的每列（行）表示是否存在一个相邻节点。

  - 因此题目 695 拥有 m × n 个节点，每个节点有 4 条边

    - ```c++
      vector<int> direction{-1, 0, 1, 0, -1}		//进行搜索
      ```

    - 且需要对边界进行判断

  - 而本题拥有 n 个节点，每个节点最多 有 n 条边，即与所有城市相连，最少可以有 1 条边，即与自己相连。

    - 上题需对4个方向进行搜索
    - 本题对每一行（每个城市）只需检索一次
    - 每一行需要对其每个元素进行检索，若满足条件则dfs延申向其他城市
    - ==理解成每个城市都可以是n分叉的树节点==
    - 一个节点即一行（一个城市），其每列的元素都是其需要判断是否进行dfs的条件
    - 一旦dfs就跳到另外的城市（n叉树的下一个节点，其也是一个n叉树）
    - 对一个城市（一行）的搜索，可以排除该城市连通的所有城市

- dfs版本

```c++
class Solution {
public:
    void dfs(vector<vector<int>>& isConnected, int idx, vector<bool>& visit){
        visit[idx] = true;
        for(int i = 0; i < isConnected[idx].size(); i++){
            if(isConnected[idx][i] == 1 && !visit[i]){
                dfs(isConnected, i, visit);
            }
        }
    }
    int findCircleNum(vector<vector<int>>& isConnected) {
        int cnt = 0;
        vector<bool> visit(isConnected.size(), false);
        for(int i = 0; i < isConnected.size(); i++){
            if(!visit[i]) {
                dfs(isConnected, i, visit);
                cnt++;
            }
            
        }
        return cnt;
    }
};
```



- bfs版本（更需要好好理解）
- 关键点：
  - 每一行（一个城市）是一个节点，依旧如上分析
  - m × n 个节点，每个节点有 4 条边；本题拥有 n 个节点，每个节点最多 有 n 条边
  - 一旦加入一个节点就visit（记忆化搜索）
  - 对每个节点需要按照其所在行的所有元素进行判断是否需要加入队列（695是4个方向）
  - 将其理解成n叉树的层序遍历（连通的就一直加入队列）
  - queue的元素不需要用pair<int, int>，因为其节点是一维的，只要知道是哪个城市即可

```c++
class Solution {
public:
    int findCircleNum(vector<vector<int>>& isConnected) {
        int n = isConnected.size();
        queue<int> q;
        int ans = 0;
        vector<bool> visit(n, false);
        for(int i = 0; i < n; i++){
            if(!visit[i]){
                q.push(i);
                visit[i] = true;
                while(!q.empty()){
                    int m = q.size();
                    for(int j = 0; j < m; j++){
                        auto node = q.front();
                        q.pop();
                        for(int k = 0; k < n; k++){
                            if(!visit[k] && isConnected[node][k] == 1){
                                q.push(k);
                                visit[k] = true;
                            }
                        }
                    }
                }
                ans++;
            }
        }
        return ans;
    }
};
```



#### 417、太平洋大西洋水流问题

- 我的思路是反向思维（水往高处流）
- 该题的关键点在于并不是全局选择搜索点，而是需要满足一定条件的才作为起点
- 边界点作为起点进行搜索，dfs版本如下：

```c++
class Solution {
public:
    vector<int> d{-1, 0, 1, 0, -1};
    vector<vector<bool>> P, A;
    vector<vector<int>> ans;
    void dfs(vector<vector<int>>& heights, int r, int c, vector<vector<bool>>& visit){
        if(visit[r][c]) return;
        visit[r][c] = true;
        if(P[r][c] && A[r][c]) ans.push_back({r, c});
        for(int i = 0; i < 4; i++){
            int x = r + d[i], y = c + d[i + 1];
            if(x >= 0 && x < heights.size() && y >= 0 && y < heights[0].size() && heights[x][y] >= heights[r][c]) dfs(heights, x, y, visit);
        }
    }
    vector<vector<int>> pacificAtlantic(vector<vector<int>>& heights) {
        int m = heights.size(), n = heights[0].size();
        P.resize(m ,vector<bool>(n, false));
        A.resize(m ,vector<bool>(n, false));
        for(int i = 0; i < m; i++){
            dfs(heights, i, 0, P);
            dfs(heights, i, n - 1, A);
        }
        for(int j = 0; j < n; j++){
            dfs(heights, 0, j, P);
            dfs(heights, m - 1, j, A);
        }
        return ans;
    }
};
```



- bfs版本相对没这么好处理,有几个关键点（整体思想类似）
- 需要两个队列处理bfs，一个针对太平洋，一个针对大西洋
- 在第二个队列中进行result结果的添加，判定条件类上（P和A对于{i，y}点都访问过)
  - 为什么是第二个，很好理解，在第一个bfs的结果上处理
- 添加队列边界元素时需要注意重复添加，就是四个边角点在第二次添加时需要去除（dfs再搜一次也无所谓，因为visit记录过了直接返回）

```c++
class Solution {
public:
    vector<vector<int>> pacificAtlantic(vector<vector<int>>& heights) {
        vector<int> d{-1, 0, 1, 0, -1};
        int m = heights.size(), n = heights[0].size();
        vector<vector<bool>> P(m, vector<bool>(n, false));
        vector<vector<bool>> A(m, vector<bool>(n, false));
        vector<vector<int>> result;
        queue<pair<int, int>> qp;
        queue<pair<int, int>> qa;
        for(int i = 0; i < m; i++){
            qp.push({i, 0});
            P[i][0] = true;
            qa.push({i, n - 1});
            A[i][n - 1] = true;
        }
        for(int j = 0; j < n; j++){
            if(!P[0][j]){
                qp.push({0, j});
                P[0][j] = true;
            }
            if(!A[m - 1][j]){
                qa.push({m - 1, j});
                A[m - 1][j] = true;
            }
        }
        while(!qp.empty()){
            auto node = qp.front();
            qp.pop();
            for(int i = 0; i < 4; i++){
                int x = node.first + d[i], y = node.second + d[i + 1];
                if(x >= 0 && x < m && y >= 0 && y < n && heights[x][y] >= heights[node.first][node.second]){
                    if(!P[x][y]){
                        qp.push({x, y});
                        P[x][y] = true;
                    }
                }
            }
        }
        while(!qa.empty()){
            auto node = qa.front();
            qa.pop();
            if(P[node.first][node.second] && A[node.first][node.second]) result.push_back({node.first, node.second});
            for(int i = 0; i < 4; i++){
                int x = node.first + d[i], y = node.second + d[i + 1];
                if(x >= 0 && x < m && y >= 0 && y < n && heights[x][y] >= heights[node.first][node.second]){
                    if(!A[x][y]){
                        qa.push({x, y});
                        A[x][y] = true;
                    }
                }
            }
        }
        return result;
    }
};
```



#### 回溯专题（dfs）：

#### 46、全排列

- 我的思路是回溯，关键点在于状态修改和恢复的时机，可能在for循环内或者for循环外，本题在循环内

```c++
class Solution {
public:
    vector<vector<int>> ans;
    void backTrack(vector<int>& nums, int level, vector<int>& perms, vector<bool>& visit){
        int n = nums.size();
        if(level == n){
            ans.push_back(perms);
            return;
        }
        for(int i = 0; i < n; i++){
            if(visit[i]) continue;
            perms.push_back(nums[i]);
            visit[i] = true;
            backTrack(nums, level + 1, perms, visit);
            visit[i] = false;
            perms.pop_back();
        }
    }
    vector<vector<int>> permute(vector<int>& nums) {
        int n = nums.size();
        vector<bool> visit(n, false);
        vector<int> perms;
        backTrack(nums, 0, perms, visit);
        return ans;
    }
};
```



#### 77、组合（==理解组合和排列差别==）

- 排列回溯的是交换的位置，而组合回溯的是否把当前的数字加入结果中（即是否选择该数字）
- 组合不需要visit（因为不需要for循环每次都保证全部遍历），只需要分为2种情况（==选或不选==）
  - 考虑选择当前位置：backTrack(n, k, idx + 1, perm);
  - 考虑不选择当前位置：backTrack(n, k, idx + 1, perm);
- 即组合只需要判断两种情况，而排列需要for循环对所有的元素都进行判断（但是需要满足才进入搜索过程）

```c++
class Solution {
public:
    vector<vector<int>> perms;
    void backTrack(int n, int k, int idx, vector<int>& perm){
        // 剪枝：perm 长度加上区间 [idx, n] 的长度小于 k，不可能构造出长度为 k 的 perm
        if (perm.size() + (n - idx + 1) < k) {
            return;
        }
        // 记录合法的答案
        if(perm.size() == k){
            perms.push_back(perm);
            return;
        }
        // 考虑选择当前位置
        perm.push_back(idx);
        backTrack(n, k, idx + 1, perm);
        perm.pop_back();
        // 考虑不选择当前位置
        backTrack(n, k, idx + 1, perm);
    }
    vector<vector<int>> combine(int n, int k) {
        vector<int> perm;
        backTrack(n, k, 1, perm);
        return perms;
    }
};
```



#### 79、单词搜索

- 我的思路是dfs（回溯），关键点在于
  - 状态的更改和恢复在for循环外（大部分回溯题都是）
  - 46全排列那道在for循环内，是因为第一轮也放在for循环内处理了（第一轮选择每个元素都可以）
  - 正常来说回溯的起点已经是第一轮了，所以一般状态更改在for循环外

```c++
class Solution {
public:
    vector<int> d{-1, 0, 1, 0, -1};
    bool flag = false;
    void dfs(vector<vector<char>>& board, string word, int r, int c, int idx, vector<vector<bool>>& visit){
        if(idx == word.size() - 1){
            flag = true;
            return;
        }
        if(flag) return;
        if(visit[r][c]) return;
        visit[r][c] = true;
        for(int i = 0; i < 4; i++){
            int x = r + d[i], y = c + d[i + 1];
            if(x >= 0 && x < board.size() && y >= 0 && y < board[0].size() && board[x][y] == word[idx + 1] && !visit[x][y]){
                dfs(board, word, x, y, idx + 1, visit);
            }
        }
        visit[r][c] = false;
    }
    bool exist(vector<vector<char>>& board, string word) {
        int m = board.size(), n = board[0].size();
        vector<vector<bool>> visit(m, vector<bool>(n, false));
        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                if(!visit[i][j] && board[i][j] == word[0]){
                    cout<<i<<" "<<j<<endl;
                    dfs(board, word, i, j, 0, visit);
                    if(flag) return true;
                }
            }
        }
        return false;
    }   
};
```



#### 934、最短的桥（==最短距离考虑bfs==）

- 思路是两次搜索（自己没做对）
- 关键是搜索起点，选择的是一座岛屿周围的0
- 每次递增len需要遍历完该层附近所有距离1的格子，需要两层for（一层节点数，一层4，类似二叉树层序遍历的思想）

```c++
class Solution {
private:
    queue<pair<int, int>> points;
    vector<int> direction = {-1, 0, 1, 0, -1};
    void dfs(vector<vector<int>>& grid, int i, int j) {
        grid[i][j] = 2;
        for (int k = 0; k < 4; k++) {
            int x = i + direction[k];
            int y = j + direction[k + 1];
            if (x >= 0 && x < grid.size() && y >= 0 && y < grid[0].size()) {
                if (grid[x][y] == 2) continue;
                if (grid[x][y] == 1) dfs(grid, x, y);
                // 收集这个岛屿附近的0
                if (grid[x][y] == 0) points.push({x, y});
            }
        }
    }
public:
    int shortestBridge(vector<vector<int>>& grid) {
        int m = grid.size();
        int n = grid[0].size();
        // 找到第一个岛屿
        bool isFind = false;
        for (int i = 0; i < m; i++) {
            if (isFind) break;
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 1) {
                    // 调用dfs把这个岛屿都标志为2
                    dfs(grid, i, j);
                    isFind = true;
                    break;
                }
            }
        }
        // 找另一个岛屿 BFS
        int res = 0;
        while (!points.empty()) {
            int size = points.size();
            res++;
            while (size--) {
                auto [x, y] = points.front();
                points.pop();
                // 把这一层的0全部填为2，再把外层的0再加入队列，逐层填陆地，直到碰到第二片岛屿
                for (int k = 0; k < 4; k++) {
                    int p = x + direction[k];
                    int q = y + direction[k + 1];
                    if (p >= 0 && p < m && q >= 0 && q < n) {
                        if (grid[p][q] == 1) return res;
                        if (grid[p][q] == 2) continue;
                        points.push({p, q});
                        grid[p][q] = 2;
                    }
                }
            }
        }
        return res;
    }
};
```



#### 130、被围绕的区域

- 我的思路是简单dfs即可

```c++
class Solution {
public:
    vector<int> d{-1, 0, 1, 0, -1};
    void dfs(vector<vector<char>>& board, int r, int c){
        board[r][c] = 'O';
        for(int i = 0; i < 4; i++){
            int x= r + d[i], y = c + d[i + 1];
            if(x >= 0 && x < board.size() && y >= 0 && y < board[0].size()){
                if(board[x][y] == 'P') dfs(board, x, y);
            }
        }
    }
    void solve(vector<vector<char>>& board) {
        int m = board.size(), n = board[0].size();
        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                if(board[i][j] == 'O'){
                    board[i][j] = 'P';
                }
            }
        }
        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                if(i != 0 && i != m - 1 && j != 0 && j != n -1){
                    continue;
                }
                if(board[i][j] == 'P'){
                    dfs(board, i, j);
                }
            }
        }
        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                if(board[i][j] == 'P'){
                    board[i][j] = 'X';
                }
            }
        }
    }
};
```



- bfs版本如下，整体思路完全一致

```c++
class Solution {
public:
    vector<int> d{-1, 0, 1, 0, -1};
    void solve(vector<vector<char>>& board) {
        int m = board.size(), n = board[0].size();
        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                if(board[i][j] == 'O'){
                    board[i][j] = 'P';
                }
            }
        }
        queue<pair<int, int>> q;
        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                if(i != 0 && i != m - 1 && j != 0 && j != n -1){
                    continue;
                }
                if(board[i][j] == 'P'){
                    q.push({i, j});
                    board[i][j] = 'O';
                    while(!q.empty()){
                        auto [r, c] = q.front();
                        q.pop();
                        for(int i = 0; i < 4; i++){
                            int x = r + d[i], y = c + d[i + 1];
                            if(x >= 0 && x < m && y >= 0 && y < n){
                                if(board[x][y] == 'P'){
                                    q.push({x, y});
                                    board[x][y] = 'O';
                                }
                            }
                        }
                    }
                }
            }
        }
        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                if(board[i][j] == 'P'){
                    board[i][j] = 'X';
                }
            }
        }
    }
};
```



#### 257、二叉树的所有路径

- 我的思路是简单的dfs即可

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<string> ans;
    void dfs(TreeNode* root, string str){
        str += to_string(root->val) + "->";
        if(root->left) dfs(root->left, str);
        if(root->right) dfs(root->right, str);
        if(!root->left && !root->right){
            ans.push_back(string(str.begin(), str.end() - 2));
            return;
        }
    }
    vector<string> binaryTreePaths(TreeNode* root) {
        if(!root) return {};
        string str = "";
        dfs(root, str);
        return ans;
    }
};
```



#### 47、全排列2

- 我的思路是按照普通的排列做，最后利用set去重（因为该题有重复元素）

```c++
class Solution {
public:
    vector<vector<int>> tmp;
    vector<int> perm;
    void backTrack(vector<int>& nums, int level, vector<bool>& visit){
        int n = nums.size();
        if(level == n){
            tmp.push_back(perm);
        }
        for(int i = 0; i < n; i++){
            if(visit[i]) continue;
            visit[i] = true;
            perm.push_back(nums[i]);
            backTrack(nums, level + 1, visit);
            perm.pop_back();
            visit[i] = false;
        }
    }
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        vector<bool> visit(nums.size(), false);
        backTrack(nums, 0, visit);
        set<vector<int>> s;
        for(const auto& t : tmp){
            s.insert(t);
        }
        vector<vector<int>> ans;
        for(const auto& vec : s){
            ans.push_back(vec);
        }
        return ans;
    }
};
```

- 答案的思路比较好，不过我个人用的就这种简单的去重方法



#### 40、组合总和（超时）

- 组合题目有点类似==背包问题==，就是选择或是不选择
- 依旧视作普通组合问题加set去重的套路，但是（172/176）

```c++
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> com;
    void backTrack(vector<int>& candidates, int target, int idx){
        if(target == 0){
            ans.push_back(com);
            return;
        }
        if(idx == candidates.size()) return;
        if(target >= candidates[idx]){
            com.push_back(candidates[idx]);
            backTrack(candidates, target - candidates[idx], idx + 1);
            com.pop_back();
        }
        backTrack(candidates, target, idx + 1);
    }
    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        backTrack(candidates, target, 0);
        set<vector<int>> s;
        for(const auto& vec : ans){
            s.insert(vec);
        }
        vector<vector<int>> perms;
        for(const auto& vec : s){
            perms.push_back(vec);
        }
        return perms;
    }
};
```



#### ==组合和排列小结==

- 排列需要考虑每个元素的位置，组合只需要考虑选不选这个元素
  - 在每次调用中，排列都需要for循环针对每个元素进行判断
  - 组合只需要在每次调用中判断是否选择该元素，即只有两种情形（无须for）
- 由于排列每次调用都要考虑所有元素，组合只需要一直往后递增idx即可（对每个idx判断是否选择该元素）
  - 排列需要记忆化，即需要visit矩阵记录当前元素是否访问过
  - 组合不需要visit，只需要针对选或不选写清楚即可
- 具体的对比可以看上面两道题以及回溯里的46和77

