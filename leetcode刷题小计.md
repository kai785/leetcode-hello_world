# LeetCode刷题小计

==函数括号记得另起一行==

## 滑动窗口

> 万能模板请：

```C
int findSubArray(nums)
{
    int len = strlen(s); // 数组，字符串长度
    int left = 0;        // 左右指针
    int right = 0;
    int sum = 0;         // 用于统计 子数组/子区间是否有效，根据题目要求可能会改成求和，								计数
    int ret;             // 保存满足题意的子数组/子字符串的长度
    
    while (right < len) {
        sum += nums[right] // 增加当前右边指针的数字/字符的求和/计数
        while 区间[left, right] { // 此时需要一直移动左指针，直到找到一个符合题意得区间
            sum -= nums[left]; // 移动左指针前需要从counter中减少left位置字符的求和/计								  数
            left++;
    }
    ret = fmax(ret, right - left + 1);
    right++;
}
```



## 二分查找（注意边界）

```C
// 左闭右开
int binarySearch(int *nums, int target, int numsSize)
{
    int low = 0;
    int high = numsSize;
    while (low < high) {
        int middle = (high - low)/2 + low;
        if (nums[middle] < target) {
            low = middle + 1;
        } else if(nums[middle] > target) {
            high = middel;
        } else {
            return middle;
        }
    }
    return -1;
}

// 左闭右闭
int binarySearch(int *nums, int target, int numsSize)
{
    int left = 0;
    int right = numsSize - 1;
    while (left <= right) {  // notice
        int middle = (right - left) / 2 + left;
        if (nums[middle] < target) {
            left = middle + 1;
        } else if (nums[middle] > target) {
            right = middle - 1;
        } else {
            return middle;
        }
    }
    return -1;
}

// 使用左闭右闭实现对相同target的左边界和右边界的查找
// 左边界
int leftBound(int *nums, int target, int numsSize)
{
    int left = 0;
    int right = numsSize - 1;
    while (left <= right) {
        int middle = (right - left) / 2 + left;
        if (nums[middle] < target) {
            left = middle + 1;
        } else if (nums[middle] > target) {
            right = middle - 1;
        } else {
            right = middle - 1;  // 缩小左区间，向左边界靠拢
        }
    }
    // 检查left越界情况
    if (left >= numsSize || nums[left] != target) {
        return -1;
    }
    return left;
}

// 右边界
int rightBound(int *nums, int target, int numsSize)
{
    int left = 0;
    int right = numsSize - 1;
    while (left <= right) {
        int middle = (right - left) / 2 + left;
        if (nums[middle] < target) {
            left = middle + 1;
        } else if (nums[middle] > target) {
            right = middle - 1;
        } else {
            left = middle + 1;   // 缩小右区间，向右边界靠拢
        }
    }
    // 判断是否越界
    if (right < 0 || nums[right] != target) {
        return -1;
    }
    return right;
}
```



## hashTable ([347. 前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/))

```C
typedef struct hashTable{
    int key;
    int val;
    UT_hash_handle hh;
} hashTable;

hashTable *cmpVal(hashTable *a, hashTable *b)
{
    // 降序排序
    return b->val - a->val;
    // 升序 ab换位
}

hashTable *cmpKey(hashTable *a, hashTable *b)
{
    return b->key - a->key;
}

int *topKFrequent(int *nums, int numsSize, int *returnSize, int k) 
{
    hashTable *dict = NULL;
    for (int i = 0; i < numsSize; i++) {
        hashTable *tmp = NULL;
        HASH_FIND_INT(dict, &nums[i], tmp);
        if (tmp == NULL) {
            tmp = malloc(sizeof(hashTable));
            tmp->key = nums[i];
            tmp->val = 1;
            HASH_ADD_INT(dict, key, tmp);
        } else {
            tmp->val++;
        }
    }
    *returnSize = k;
    int *ret = malloc(sizeof(int) * k);
    HASH_SORT(dict, cmp);
    int j = 0;
    hashTable *cur = NULL;
    hashTable *next = NULL;
    HASH_ITER(hh, dict, cur, next) {
        if (j < k) {
            ret[j++] = cur->key;
        }
    }
    return ret;
}
```

## 二叉树(中序遍历)

> 递归

```C
struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
};

void inorder(struct TreeNode* root, int* res, int* resSize) {
    if (!root) {
        return;
    }
    inorder(root->left, res, resSize);
    res[(*resSize)++] = root->val;
    inorder(root->right, res, resSize);
}

int* inorderTraversal(struct TreeNode* root, int* returnSize) {
    int* res = malloc(sizeof(int) * 501);
    *returnSize = 0;
    inorder(root, res, returnSize);
    return res;
}
```

我们有一棵二叉树：

```undefined
               中
              /  \
             左   右
```

前序遍历：中，左，右
中序遍历：左，中，右
后序遍历：左，右，中

### 颜色标记法（记不住）

```C
int* inorderTraversal(struct TreeNode* root, int* returnSize)
{
  *returnSize = 0;
   int* res = malloc(sizeof(int)*501);
   //建立堆栈
   struct TreeNode** node = malloc(sizeof(struct TreeNode*)*501);
   int top = -1;
   int* colors=malloc(sizeof(int)*510);
   //根节点入栈
   node[++top]=root;
   colors[top]=1;
   while(top>=0){   
       int color=colors[top];
       struct TreeNode *t=node[top--];
       if(t==NULL) continue;
       if(color==1){
           node[++top]=t->right;
           colors[top]=1;
           node[++top]=t;
           colors[top]=0;
           node[++top]=t->left;
           colors[top]=1;
       } else {
           res[(*returnSize)++]=t->val;
       }
   }
   return res;
}
```

## 层序遍历

```C
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */

/**
 * Return an array of arrays of size *returnSize.
 * The sizes of the arrays are returned as *returnColumnSizes array.
 * Note: Both returned array and *columnSizes array must be malloced, assume caller calls free().
 */
int** levelOrder(struct TreeNode* root,int* returnSize,int** returnColumnSizes){
    *returnSize = 0;
    if(root == NULL)
        return NULL;
    int** res = (int**)malloc(sizeof(int*) * 2000);
    *returnColumnSizes = (int*)malloc(sizeof(int) * 2000);
    struct TreeNode* queue[2000];
    int front = 0,rear = 0;
    struct TreeNode* cur;

    queue[rear++] = root;
    while(front != rear){
        int colSize = 0,last = rear;
        res[*returnSize] = (int*)malloc(sizeof(int) * (last - front));
        while(front < last){
            cur = queue[front++];
            res[*returnSize][colSize++] = cur ->val;
            if(cur -> left != NULL)
                queue[rear++] = cur -> left;
            if(cur -> right != NULL)
                queue[rear++] = cur -> right;
        }
        (*returnColumnSizes)[*returnSize] = colSize;
        (*returnSize)++;
    }
    return res;
}

// 自底向上
#define LEN 2001
int** levelOrderBottom(struct TreeNode* root, int* returnSize, int** returnColumnSizes){
    int **ans = (int **)malloc(sizeof(int *) * LEN);
    *returnColumnSizes = (int *)malloc(sizeof(int) * LEN);
    *returnSize = 0;
    if (!root)
        return ans;
    
    struct TreeNode *q[LEN];
    int l = 0, r = 0;
    q[r++] = root;

    while (l < r) {
        int len = r - l;
        int *inner = (int *)malloc(sizeof(int) * len);
        for (int i = 0; i < len; i++) {
            root = q[l++];
            inner[i] = root->val;
            if (root->left)
                q[r++] = root->left;
            if (root->right)
                q[r++] = root->right;
        }
        ans[*returnSize] = inner;
        (*returnColumnSizes)[(*returnSize)++] = len;
    }
    struct TreeNode *a;
    int b;
    for (int i = 0, j = *returnSize - 1; i < j; i++, j--) {
        a = ans[i];
        ans[i] = ans[j];
        ans[j] = a;
        b = (*returnColumnSizes)[i];
        (*returnColumnSizes)[i] = (*returnColumnSizes)[j];
        (*returnColumnSizes)[j] = b;
    }
    return ans;
}


```

## 对称二叉树

```C
bool compare(struct TreeNode *left,struct TreeNode *right)
{
    if(left==NULL && right==NULL)
        return true;
    if(left==NULL || right ==NULL)
        return false;
    if(left->val != right->val)
        return false;
    return compare(left->left,right->right) && compare(left->right,right->left);
}

bool isSymmetric(struct TreeNode* root)
{
    if(root==NULL)
        return true;
    return compare(root->left,root->right);
}
```



## 递归写法

> 1. 找到停止条件
> 2. 找返回值
> 3. 本级递归应该做什么

## 回溯

```C
void backtracking(参数) {
    if (终止条件) {
        存放结果;
        return;
    }

    for (选择：本层集合中元素（树中节点孩子的数量就是集合的大小）) {
        处理节点;
        backtracking(路径，选择列表); // 递归
        回溯，撤销处理结果
    }
}
```



## 链表

### 链表操作的两种方式

* 直接在原链表上操作
* 添加一个虚拟头节点进行操作

### 删除链表（[203. 移除链表元素](https://leetcode-cn.com/problems/remove-linked-list-elements/)）

```C
struct ListNode {
    int val;
    struct ListNode *next;
};

// 删除节点
// method 1:
struct ListNode *deleteNode(struct ListNode *head, int val)
{
    if (cur->val == val) {
        return cur->next;
    }
    while(cur) {
        if (cur->next->val == val) {
            cur->next = cur->next->next;
            break;
        }
        cur = cur->next; // 类似于i++
    }
    return head;
}

// method 2: dummyNode
struct ListNode *deleteNode(struct ListNode *head, int val)
{
    typedef struct ListNode ListNode;
    ListNode *dummyNode = head;
    ListNode *cur = dummyNode;
    while(cur->next) {
        if (cur->next->val == val) {
            cur->next = cur->next->next;
        } else {
            cur = cur->next;
        }
    }
    return dummyNode->next;
}
```

### 设计链表 （[707. 设计链表](https://leetcode-cn.com/problems/design-linked-list/)）==存疑==

```c
void myLinkedListDeleteAtIndex(MyLinkedList *obj, int index)
{
    typedef struct MyLinkedList{
    int val;
    struct MyLinkedList *next;
} MyLinkedList;

MyLinkedList* myLinkedListCreate() {
    MyLinkedList *dummyNode = malloc(sizeof(MyLinkedList));
    dummyNode->next = NULL;
    return dummyNode;
}

int myLinkedListGet(MyLinkedList* obj, int index) {
    MyLinkedList *cur = obj->next;
    for (int i = 0; cur != NULL; i++) {
        if (i == index) {
            return cur->val;
        } else {
            cur = cur->next;
        }
    }
    return -1;
}

void myLinkedListAddAtHead(MyLinkedList* obj, int val) {
    MyLinkedList *head = malloc(sizeof(MyLinkedList));
    head->val = val;
    head->next = obj->next;
    obj->next = head;
}

void myLinkedListAddAtTail(MyLinkedList* obj, int val) {
    MyLinkedList *tail = malloc(sizeof(MyLinkedList));
    tail->val = val;
    MyLinkedList *cur = obj;
    while (cur->next != NULL) {
        cur = cur->next;
    }
    cur->next = tail;
    tail->next = NULL;
}

void myLinkedListAddAtIndex(MyLinkedList* obj, int index, int val) {
    MyLinkedList *cur = obj->next;
    MyLinkedList *insert = malloc(sizeof(struct MyLinkedList));
    if (index == 0) {
        myLinkedListAddAtHead(obj, val);
        return;
    }
    for (int i = 1; cur != NULL; i++) {
        if (i == index) {
            insert->next = cur->next;          
            insert->val = val;
            cur->next = insert;   
            return;
        } else {
            cur = cur->next;
        }
    }
}

void myLinkedListDeleteAtIndex(MyLinkedList* obj, int index) {
    MyLinkedList *cur = obj->next;
    MyLinkedList *pre = obj;
    for (int i = 0; cur != NULL; i++) {
        if (i == index) {
            pre->next = cur->next;
            free(cur);
            return;
        } else {
            pre = cur;
            cur = cur->next;
        }
    }
}

void myLinkedListFree(MyLinkedList* obj) {
    while (obj != NULL) {
        MyLinkedList *tmp = obj;
        obj = obj->next;
        free(tmp);
    }
}

/**
 * Your MyLinkedList struct will be instantiated and called as such:
 * MyLinkedList* obj = myLinkedListCreate();
 * int param_1 = myLinkedListGet(obj, index);
 
 * myLinkedListAddAtHead(obj, val);
 
 * myLinkedListAddAtTail(obj, val);
 
 * myLinkedListAddAtIndex(obj, index, val);
 
 * myLinkedListDeleteAtIndex(obj, index);
 
 * myLinkedListFree(obj);
*/
}
```

## 正整数每一位平方相加（[快乐数](https://leetcode-cn.com/problems/happy-number/solution/shuang-100-jian-dan-yi-du-bu-tou-ji-qu-qiao-by-pgy/)

```C
int sumDigit(int n)
{
    int num = 0;
    while (n != 0) {
        num = num + (n % 10) * (n % 10);  // n % 10 获得最高位数字
        n = n/10;
    }
    return num;
}
```

## returnColSize (nmsl)

returnColsize不应该认为是一个二维数组，虽然从int** 的角度看是的，但它的作用是，返回作为答案的二维数组中每一行的元素个数，因此，首先对 *returnColsize赋值，returnColsize看作一个一维数组，用malloc提供存储空间，数组大小为答案数组的行数，接着，该一维数组中的元素就是答案数组中每一行对应的元素个数。既然前面提到returnColsize看作一个一维数组，那么returnColsize便自然视为这个一维数组的地址，或者说是指向这个一维数组的指针，而不应看作一个二维数组。

# 将两位数的每一位数字平方并求和

```C
例如：
    19 = 1*1 + 9*9；
代码：
    int sumDigit(int num)
	{
    	int ret = 0;
    	while (num != 0) {
            ret = ret + (num % 10) * (num % 10);
            num = num / 10;
        }
    	return ret;
	}
```

## 设计题

#### [348. 设计井字棋](https://leetcode.cn/problems/design-tic-tac-toe/)

```C
typedef struct {
	int* rows;	//每行的和
	int* cols;	//每列的和
	int dig[2]; //两条对角线的和
	int size;
} TicTacToe;

/** Initialize your data structure here. */

TicTacToe* ticTacToeCreate(int n) {
	TicTacToe* obj = (TicTacToe*)calloc(1, sizeof(TicTacToe));
	obj->size = n;
	obj->rows = (int*)calloc(n, sizeof(int));
	obj->cols = (int*)calloc(n, sizeof(int));
	return obj;
}

/** Player {player} makes a move at ({row}, {col}).
        @param row The row of the board.
        @param col The column of the board.
        @param player The player, can be either 1 or 2.
        @return The current winning condition, can be either:
                0: No one wins.
                1: Player 1 wins.
                2: Player 2 wins. */
int ticTacToeMove(TicTacToe* obj, int row, int col, int player) {
	int n = obj->size;
	int add = (player == 1)? 1: -1;
	obj->rows[row] += add;
	obj->cols[col] += add;

    // 主对角线
	if(row == col)
		obj->dig[0] += add;
    // 副对角线
	if(row + col == n-1)
		obj->dig[1] += add;
	if(player == 1)
	{
		if(obj->rows[row] == n || obj->cols[col] == n || obj->dig[0] == n | obj->dig[1] == n)
			return player;
	}else{
		if(obj->rows[row] == -n || obj->cols[col] == -n || obj->dig[0] == -n | obj->dig[1] == -n)
			return player;
	}

	return 0;
}

void ticTacToeFree(TicTacToe* obj) {
	free(obj->rows);
	free(obj->cols);
	free(obj);
}
```

[#1912 设计电影租借系统](https://leetcode.cn/problems/design-movie-rental-system/)

```C
#define MAX_ANS 5

typedef struct {
    int shop;
    int movie;
} MovieKey; //准备hash解的，先数组试试

typedef struct {
    MovieKey key;
    int price;
    bool isRent;
} Movies;

typedef struct {
    int movieNum;
    Movies *mvs; // 所有输入的列表，按mv和shop升序，参考cmpMvs
    Movies **srchList; // 用于Searh接口的排序表，参考cmpSrch
    Movies **rptList;  // 用于Report接口的排序表，参考cmpRpt
} MovieRentingSystem;

MovieRentingSystem g_sys = { 0 };

int cmpSrch(const void *a, const void *b)
{
    Movies **ma = (Movies **)a;
    Movies **mb = (Movies **)b;

    if (ma[0]->key.movie != mb[0]->key.movie)
        return ma[0]->key.movie - mb[0]->key.movie;
    if (ma[0]->price != mb[0]->price)
        return ma[0]->price - mb[0]->price;
    return ma[0]->key.shop - mb[0]->key.shop;
}

int cmpRpt(const void *a, const void *b)
{
    Movies **ma = (Movies **)a;
    Movies **mb = (Movies **)b;

    if (ma[0]->price != mb[0]->price)
        return ma[0]->price - mb[0]->price;           
    if (ma[0]->key.shop != mb[0]->key.shop)
        return ma[0]->key.shop - mb[0]->key.shop;
    return ma[0]->key.movie - mb[0]->key.movie;
}

int cmpMvs(const void *a, const void *b)
{
    Movies *ma = (Movies *)a;
    Movies *mb = (Movies *)b;

    if (ma->key.movie != mb->key.movie) {
        return ma->key.movie - mb->key.movie;
    }

    return ma->key.shop - mb->key.shop;
}

MovieRentingSystem *movieRentingSystemCreate(int n, int **entries, int entriesSize, int *entriesColSize)
{
    g_sys.movieNum = entriesSize;

    g_sys.mvs = (Movies *)malloc(entriesSize * sizeof(Movies));
    g_sys.srchList = (Movies **)malloc(entriesSize * sizeof(Movies *));
    g_sys.rptList = (Movies **)malloc(entriesSize * sizeof(Movies *));

    for (int i = 0; i < entriesSize; i++) {
        g_sys.mvs[i].isRent = false;
        g_sys.mvs[i].key.shop = entries[i][0];
        g_sys.mvs[i].key.movie = entries[i][1];
        g_sys.mvs[i].price = entries[i][2];

        g_sys.srchList[i] = &g_sys.mvs[i];
        g_sys.rptList[i] = &g_sys.mvs[i];        
    }

    qsort(g_sys.mvs, g_sys.movieNum, sizeof(Movies), cmpMvs);
    qsort(g_sys.srchList, g_sys.movieNum, sizeof(Movies *), cmpSrch);
    qsort(g_sys.rptList, g_sys.movieNum, sizeof(Movies *), cmpRpt);    

    return &g_sys;
}

// 是找到最左的边界
int BinSearch(MovieRentingSystem *obj, int movie)
{
    int mid;
    int min = 0;
    int max = obj->movieNum - 1;

    while(min <= max){
        mid = min + ( max - min ) / 2;
        if (obj->srchList[mid]->key.movie < movie) {
            min = mid + 1;
        } else if (obj->srchList[mid]->key.movie > movie){
            max = mid - 1;
        } else if(obj->srchList[mid]->key.movie == movie){
            max = mid - 1; // 继续找左边界
        }
    }

    if (min >= obj->movieNum || obj->srchList[min]->key.movie != movie) {
        return -1;
    }

    return min;
}

int *movieRentingSystemSearch(MovieRentingSystem *obj, int movie, int *retSize)
{
    int count = 0;
    int *ans = (int *)malloc(MAX_ANS * sizeof(int));

    int i = BinSearch(obj, movie);
    if (i == -1) {
        *retSize = 0;
        return NULL;
    }
    // 已经按要求排序
    for (; i < obj->movieNum; i++) {
        if (obj->srchList[i]->key.movie == movie && obj->srchList[i]->isRent == false) {
            ans[count++] = obj->srchList[i]->key.shop;
        }
        if (count >= MAX_ANS || obj->srchList[i]->key.movie > movie) {
            break;
        }
    }

    *retSize = count;
    return ans;
}


void setMovies(MovieRentingSystem *obj, int shop, int movie, bool isRent) {
    Movies want = {0};
    want.key.shop = shop;
    want.key.movie = movie;

    Movies *find = bsearch(&want, obj->mvs, obj->movieNum, sizeof(Movies), cmpMvs);

    find->isRent = isRent;
}

void movieRentingSystemRent(MovieRentingSystem *obj, int shop, int movie)
{
    setMovies(obj, shop, movie, true);
}

void movieRentingSystemDrop(MovieRentingSystem *obj, int shop, int movie)
{
    setMovies(obj, shop, movie, false);
}

int **movieRentingSystemReport(MovieRentingSystem *obj, int *retSize, int **retColSize)
{
    int count = 0;
    int **ans = (int **)malloc(MAX_ANS * sizeof(int *));
    // 已经按要求排序
    for (int i = 0; i < obj->movieNum; i++) {
        if (obj->rptList[i]->isRent == true) {
            ans[count] = (int *)malloc(2 * sizeof(int));
            ans[count][0] = obj->rptList[i]->key.shop;
            ans[count][1] = obj->rptList[i]->key.movie;
            count++;
        }
        if (count >= MAX_ANS) {
            break;
        }
    }

    *retSize = count;
    *retColSize = (int *)malloc(sizeof(int) * count);
    for (int i = 0; i < count; i++) {
        (*retColSize)[i] = 2;
    }

    return ans;
}

void movieRentingSystemFree(MovieRentingSystem *obj) {
    free(obj->mvs);
    free(obj->rptList);
    free(obj->srchList);
}
```

