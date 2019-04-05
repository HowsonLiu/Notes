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