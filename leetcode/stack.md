# 341 扁平化嵌套列表迭代器
[https://leetcode-cn.com/problems/flatten-nested-list-iterator/](https://leetcode-cn.com/problems/flatten-nested-list-iterator/)
## 解
使用dfs能直接将多层列表降维成单行列表
```c++
void dfs(vector<NestedInteger> &nestedList) {
    for(auto& l : nestedList) {
        if(l.isInteger())
            mem.push_back(l.getInteger());
        else 
            dfs(l.getList());
    }
}
```

但是只能在初始化的时候使用，有点不符合迭代器的定义，正确方式应该使用栈

实际上从递归转成栈不难，每一层调用实际上栈帧帮我们保存了vector迭代器的值，因此我们只需要一个类似这样的栈就可以了
```c++
stack<pair<vector<NestedInteger>::iterator, vector<NestedInteger>::iterator>> stk;
```
初始化保存第一层栈帧
```c++
NestedIterator(vector<NestedInteger> &nestedList) {
    stk.emplace(nestedList.begin(), nestedList.end());
}
```

调用`next`函数，他会直接返回`int`类型的值，并且题目可以保证调用`next`前必定调用`hasNext`函数，因此`hasNext`可以帮我们做到移动当前迭代器至下一个非`List`类型的`NestedInteger`上，这使得`next`函数十分简单，只需要调用并移动迭代器就可以了
```c++
int next() {
    return stk.top().first++->getInteger();
}
```

`hasNext`函数职责大点，他需要移动迭代器至非`List`类型的`NestedInteger`上
```c++
bool hasNext() {
    while(!stk.empty()) {
        auto& curLayer = stk.top();
        if(curLayer.first == curLayer.second) {
            stk.pop();                                          // “递归”退出
            continue;
        }
        if(curLayer.first->isInteger()) {
            return true;
        }
        else {
            auto& nextLayer = curLayer.first++->getList();      // 记得移动迭代器
            // expand
            stk.emplace(nextLayer.begin(), nextLayer.end());    // “递归”调用
        }
    }
    return false;
}
```

## 注意事项
1. 注意引用
2. `emplace`函数替代`push`
    `emplace`函数与`push`的区别是：`push`参数必须为`pair`，而`emplace`参数为`pair`构造函数的参数，写起来方便，还省了一个临时变量