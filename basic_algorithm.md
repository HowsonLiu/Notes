# 基本算法
## 快速排序
### 数组
```c++
/**
 *@param arr：数组
 *@param start：起始下标
 *@param end：终止下标
 *@return：排序后基准值的下标
 */
int Partition(int* arr, int start, int end){
    int target = arr[start];    // 默认选取第一个为基准值，某些算法可以通过随机取值优化
    int i = start;              // 注意这里i没有选start+1
    int j = end;
    while(i < j){
        // 必须先从后找，因为这语句的终止情况要么是j指向比target小的；要么j == i。
        // 前者没问题，后者只有i在初始值的情况下才不会出错，所以必须先从后找
        while(arr[j] >= target && i < j) --j;
        while(arr[i] <= target && i < j) ++i;   // 注意 <=
        swap(arr[i], arr[j]);   
    }
    swap(arr[start], arr[j]);   // j只有两种情况：要么指向比target小的；要么是target的下标
    return j;
}

/**
 *@param arr：数组
 *@param start：起始地址
 *@param end：结束地址
 */
void quicksort(int* arr, int start, int end){
    if(start >= end) return;
    int index = Partition(arr, start, end);
    quicksort(arr, start, index-1);
    quicksort(arr, index+1, end);
}
```

## 归并排序
### 数组
```c++
/**
 *@param arr: 数组首地址
 *@param left: 起始下标
 *@param right: 结束下标，不是大小
 *@param tmp: 辅助空间
 */
void sort(int* arr, int left, int right, int* tmp){
    if(left < right){
        int mid = (left + right) / 2;
        sort(arr, left, mid, tmp);          // 左边排序
        sort(arr, mid+1, right, tmp);       // 右边排序
        merge(arr, left, mid, right, tmp);  // 合并两个有序数组
    }
}

/**
 *@param arr：数组首地址
 *@param left：左数组的起始下标
 *@param mid：右数组的起始下标
 *@param right：右数组的终止下标
 *@param tmp：辅助空间
 */
void merge(int* arr, int left, int mid, int right, int* tmp){
    int i = left;   // 左边index
    int j = mid+1;  // 右边index
    int t = 0;      // tmp index
    while(i <= mid && j <= right){
        if(arr[i] <= arr[j])    // <=保证排序稳定
            tmp[t++] = arr[i++];
        else
            tmp[t++] = arr[j++];
    }
    while(i <= mid)             // 将剩余左边放进tmp
        tmp[t++] = arr[i++];
    while(j <= right)           // 将剩余右边放进tmp
        tmp[t++] = arr[j++];
    t = 0;
    while(left <= right)        // 将tmp的放进原数组
        arr[left++] = tmp[t++]; 
}

void merge2(int* arr, int* tmp, int lo, int mid, int hi) {
    for(int k = lo; i <= hi; ++k)
        tmp[k] = arr[k];
    int i = lo, j = mid+1;
    for(int k = lo; k <= hi; ++k) {
        if(i > mid) arr[k] = tmp[j++];                  // 剩余右边
        if(j > hi) arr[k] = tmp[i++];                   // 剩余左边
        else if(tmp[i] < tmp[j]) arr[k] = tmp[i++];     // 比较，稳定排序
        else arr[k] = tmp[j++];                 
    }
}
```
### 链表
```c++
/*
 *@param head 链表首结点
 *@return 排序后的链表首结点
 */
ListNode* sort(ListNode* head){
    if(!head || !head->next) return head;
    ListNode *pPre = head, *pMid = head, *pFast = head;
    while(pFast && pFast->next){    // 快慢指针找出链表中点
        pFast = pFast->next->next;
        pPre = pMid;
        pMid = pMid->next;
    }
    pPre->next = nullptr;   // 断开链表，否则遍历左边时会跑到右边
    return merge(sort(head), sort(pMid));   // 对左右链表排序并合并
}

/*
 *@param left 左链表（已排序）
 *@param right 右链表（已排序）
 *@return 合并后的链表（已排序）
 */
ListNode* sort(ListNode* left, ListNode* right){
    if(!left) return right;
    if(!right) return left;
    if(left->val <= right->val){    // <=确保稳定性
        left->next = sort(left->next, right);
        return left;
    }
    else{
        right->next = sort(left, right->next);
        return right;
    }
}
```
# 树的非递归遍历
## 中序遍历（最简单）
```c++
vector<int> inorderTraversal(TreeNode* root) {
    vector<int> res;
    stack<TreeNode*> stk;
    TreeNode* cur = root;
    while(cur || !stk.empty()) {
        while(cur) {                // 如果有cur，则需要向左遍历至最左节点
            stk.push(cur);          // 并将经过的节点压栈
            cur = cur->left;
        }
        cur = stk.top();            // 此时cur必为空，栈顶为最左节点
        res.push_back(cur->val);    // 访问
        stk.pop();                  // 出栈
        cur = cur->right;           // 即使没有右节点也不用担心，下一轮循环的while(cur)会跳过
    }
    return res;
}
```
访问的时机在出栈前
## 前序遍历
```c++
vector<int> preorderTraversal(TreeNode* root) {
    vector<int> res;
    stack<TreeNode*> stk;
    auto cur = root;
    while(cur || !stk.empty()) {
        while(cur) {
            res.push_back(cur->val);    // 入栈前访问
            stk.push(cur);
            cur = cur->left;
        }
        cur = stk.top();
        stk.pop();
        cur = cur->right;
    }
    return res;
}
```
前序遍历跟中序遍历一样，区别在于访问的时机在入栈前
## 后序遍历
```c++
vector<int> postorderTraversal(TreeNode* root) {
    vector<int> res;
    stack<TreeNode*> stk;
    TreeNode *cur = root, *prev = nullptr;      // 需要一个额外的变量记录右子树是否被遍历过了
    while(cur || !stk.empty()) {                // 老三样
        while(cur) {                            // 老三样，先找到最左节点
            stk.push(cur);
            cur = cur->left;
        }
        cur = stk.top();                        // 此时cur是（未访问的）最左节点，此时他必定没有左子树（或左子树已经访问过了），但是他有可能有右子树，不能直接访问，所以需要下面的判断
        stk.pop();
        if(!cur->right || cur->right == prev) { // 太棒了，这个最左节点没有右节点/右节点已经被访问过了
            res.push_back(cur->val);            // 终于能访问中间这个节点了（以父节点的身份访问）
            prev = cur;                         // 我这节点有可能是别人的右子树，需要记录一下
            cur = nullptr;                      // 下一个要访问节点从栈里面取吧
        }
        else {                                  // 可恶，居然有未访问过的右子树
            stk.push(cur);                      // 白取出来了，压回去
            cur = cur->right;                   // 从右节点开始走一遍流程吧
        }
    }
    return res;
}
```
其实大概框架也跟中序遍历一样，只不过由于节点的访问时机跟右子树是否被访问过有关，所以需要增加一个变量记录前一个访问的节点，以及增加一个if语句判断右子树是否存在或北方问过
# 常见题目
## 约瑟夫环
### 题目
> n个人围成一环，编号从1~n。数到m的人出列，问最后生还者编号

### 解法
```c++
int lastRemaining(int n, int m) {
    int f = 0;
    for(int i = 2; i <= n; ++i)
        f = (m+f)%i;
    return f+1;
}
```
### 理解
我们先明确一下条件以及变量：
- `n`：当前围成一环的人数
- `m`：出列偏移
- `f(n)`：最终生还者在**第`n`轮的编号**（第n轮指的是人数为n的那一轮）
- 为了方便用数组表示，下标统一从0开始
- `f(1)=0`：显然，在只有一个人的情况下，无论`m`是多少，都是下标为0的人出列

解法是一个自底向上的解法，它所表达的意思是：**最终生还者，在出列偏移为`m`的情况下，在第`n`轮的编号是多少**。也就是说，它求的是一个**编号映射关系**  

编号的映射关系自底向上不好思考，我们不妨切换到自顶向下进行思考：
1. 在第n轮中，出列的人的编号为：`(0+m-1) % n`  
    `-1`是因为下标偏移到从0开始，`%n`是因为成环，`0+`是因为在第n轮中从下标为0的人开始
2. 那么在第n-1轮中，**出列的人后面第一个人，就成为了第一个数0的人，也就是第n-1轮的下标0**。也就是说，第n-1轮的下标0对应在第n轮的下标为`(0+m-1+1) % n => m % n`
3. 依次类推，在第n-1轮下标i对应在第n轮的下标为`(m+i) % n`，也就是**`i = (m+i) % n`**

### 总结
- 求约瑟夫环最终生还者的自底向上解本质上是求编号映射关系
- 出列者编号对于我们毫无作用，反而是**出列者后一位对应是下一轮的第一位**这个点，是帮助我们确定编号映射关系的关键点
