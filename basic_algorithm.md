# 基本算法
## 归并排序
### 数组
```c++
/**
 *@arr: 数组首地址
 *@left: 起始下标
 *@right: 结束下标，不是大小
 *@tmp: 辅助空间
 */
void sort(int* arr, int left, int right, int* tmp){
    if(left < right){
        int mid = (left + right) / 2;
        sort(arr, left, mid, tmp);          // 左边排序
        sort(arr, mid+1, right, tmp);       // 右边排序
        merge(arr, left, mid, right, tmp);  // 合并两个有序数组
    }
}

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
 *@head 链表首结点
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
 *@left 左链表（已排序）
 *@right 右链表（已排序）
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