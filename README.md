LeetCode Complete
======

0. 说明
------

* 欢迎 Star/Fork/Pull request，不理解的也可以加我QQ：860220736（仅限学妹）。
* 原则上使用 C 做, 如果需要用到 Hash, Stack, Queue, 或者返回值特别复杂, 或者需要大量拼接字符串时, 使用 C++。
* 这份文档的目的是复习用的，并不是教如何解题的，因此只有简略介绍，适合面试前查漏补缺复习。

Let the hack begin!

^1. 从数组中找出两个数字使得他们的和是给定的数字
------

使用一个散列, 存储数字和他对应的索引。然后遍历数组, 如果另一半在散列当中, 那么返回
这两个数的索引, 程序结束；如果不在, 把当前数字加入到散列中。

``` C++
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int, int> hash;
    vector<int> result(2);
    for (int i = 0; i != nums.size(); ++i) {
        int reminder = target - nums[i];
        if (hash.find(reminder) != hash.end()) {
            result[0] = hash[reminder] + 1;
            result[1] = i + 1;
            return result;
        }
        hash[nums[i]] = i;
    }
    return result;
}
```

如果数组是已经排序的呢？

```C++
sort(nums.begin(), nums.end()) // 假设已经排序，只有一个结果
pair<int> twoSum(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;
    while (left < right) {
        int s = nums[left] + nums[right];
        if (s == target)
            return make_pair(left, right);
        else if (s < sum)
            left++;
        else
            right--;
    }
}
```

2. 给两个列表, 数字在其中按低位到高位存储, 求他们的和
------

直接迭代遍历数组, 考察细节操作。注意 dummy head 的使用。

```C
struct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2) {
    struct ListNode dummy, *p = &dummy;
    int carry = 0;
    while (l1 || l2 || carry) {
        int v1 = l1 ? l1->val: 0;
        int v2 = l2 ? l2->val: 0;
        int v = v1 + v2 + carry;
        p->next = malloc(sizeof(struct ListNode));
        p = p->next;
        p->val = v % 10;
        p->next = NULL;
        carry = v / 10;

        l1 = l1 ? l1->next: NULL;
        l2 = l2 ? l2->next: NULL;
    }
    return dummy.next;
}
```

3. 最长不重复子串
------

使用动态规划, 在一个 Hash 中存储已经出现的字符的上一次出现的索引值, 如果索引值存在, 则把当前最长子串的左边界更新为改索引值。

注意, 当字符有限的时候, 比如限定为 ASCII 字符, 可以使用一个数组代替 Hash。

``` C

int lengthOfLongestSubstring(char* s) {
    int indices[256];
    for (int i = 0; i < 256; i++) //init the array, memset can only be used for char
        indices[i] = -1;
    int left = 0; // 关键点
    int longest = 0; // 关键点

    for (int i = 0; s[i] != '\0'; i++) {
        left = max(left, indices[s[i]] + 1);
        indices[s[i]] = i;
        longest = max(longest, i - left + 1);
    }
    return longest;
}
```

4. 找到两个排序数组的中位数
------

解法在[这里](https://leetcode.com/discuss/15790/share-my-o-log-min-m-n-solution-with-explanation)
把 AB 分成两份，比如 A[0..i], B[0..j] 和 A[i, m], B[j, n]，这样我们只需要下面两个条件就可以了

1. i+j = m-i + n-j
2. B[j-1] <= A[i] && A[i-1] <= B[j]

这时候我们就得到了A[i]就是我们的中位数，或者之一。
i 的初始值在0到 m 指尖，然后我们二分搜索 `i = (imin + imax) / 2, j = mid - i`。

```C
#define max(a,b) ((a)>(b)?(a):(b))
#define min(a,b) ((a)<(b)?(a):(b))
double findMedianSortedArrays(int* A, int m, int* B, int n) {
    if (m > n) return findMedianSortedArrays(B, n, A, m);
    int imin = 0, maxidx = m, i, j, num1, mid = (m + n + 1) >> 1,num2;
    while (imin <= maxidx) {
        i = (imin + maxidx) >> 1;
        j = mid - i;
        if (i < m && j > 0 && B[j-1] > A[i]) // B中的数字偏大
            imin = i + 1;
        else if (i > 0 && j < n && B[j] < A[i-1]) // A中的数字偏大
            maxidx = i - 1;
        else {
            if (i == 0) num1 = B[j-1];
            else if (j == 0) num1 = A[i - 1];
            else num1 = max(A[i-1],B[j-1]); // 普通情况
            break;
        }
    }
    if ((m + n) & 0x1) // odd
        return num1;
    if (i == m)
        num2 = B[j];
    else if (j == n)
        num2 = A[i];
    else 
        num2 = min(A[i], B[j]); // 普通情况
    return (num1 + num2) / 2.0; // 注意整数除法
}
```

5. 最长回文子串
------

1. 以某个元素为中心, 向两边展开, 注意处理奇数和偶数两种情况
2. Manacher 算法, 参见http://taop.marchtea.com/01.05.html

````C
char* longestPalindrome(char* s) {
    if (!s) return NULL;

    int length = 0; // length of the longest palindromic string
    int start = -1; // start of the lonest palidromic string

    int len = strlen(s);
    for (int i = 0; i < len; i++) {

        // length is odd
        for (int j = 0; (i - j >= 0) && (i + j < len); j++) { 
            if (s[i - j] != s[i + j])
                break;
            if (j * 2 + 1 > length) {
                length = j * 2 + 1;
                start = i - j;
            }
        }

        // length is even
        for (int j = 0; (i - j >= 0) && (i + j + 1 < len); j++) {
            if (s[i - j] != s[i + j + 1])
                break;
            if (j * 2 + 2 > length) {
                length = j * 2 + 2;
                start = i - j;
            }
        }
    }

    char* result = malloc(sizeof(char) * length + 1);
    strncpy(result, s + start, length);
    result[length] = 0;

    return result;
}
```

6. ZigZag 字符串, 把字符串掰弯, 然后再按行输出
------

考察数学, 找出规律, 所以实际上并不是 Z 子形，而是由 V 组成的，然后组合按行号重构后的字符串即可。

``` C
char* convert(char* s, int numRows) {
    int len = strlen(s);
    if (!s || numRows <= 1 || len < numRows) return s; // no need to convert

    char* zigzag = malloc(sizeof(char) * (len + 1));
    int cur = 0;

    for (int i = 0; i < numRows; i++) {
        for (int j = i; j < len; j += 2 * (numRows - 1)) { // 每个 v 字型长度
            zigzag[cur++] = s[j];
            if (i != 0 && i != numRows - 1) { // 中间行有斜线
                int t = j + 2 * (numRows - 1) - 2 * i; // V 的第二笔
                if (t < len)
                    zigzag[cur++] = s[t];
            }
        }
    }
    zigzag[cur] = '\0';
    return zigzag;
}
```

7. 翻转数字, 溢出返回0
------

注意溢出

```C
int reverse(int x) {
    if (x == INT_MIN) return 0;
    if (x < 0) return -reverse(-x);

    long result = 0;
    while (x) {
        result = result * 10 + x % 10;
        x /= 10;
    }
    return result > INT_MAX ? 0 : result;

}
```

8. 实现 atoi
------

注意各种特殊情况：

1. 首先过滤空格
2. 判定符号, 符号只能出现一次
3. 是否溢出, 溢出返回 INT_MAX 或者 INT_MIN

```C
int myAtoi(char* str) {
    if (!str) return 0;
    int sign = 1;
    int result = 0;

    // discarding spaces
    while (isspace(*str))
        str++; 

    // determining sign
    if (*str == '-' || *str == '+') {
        if (*str == '-') sign = -1;
        if (*str == '+') sign = 1;
        str++;
    }

    // constructing integer
    while (isdigit(*str)) {
        // handling overflow
        if (result > INT_MAX / 10 || result == INT_MAX / 10 && *str - '0' > INT_MAX % 10)
            return sign > 0 ? INT_MAX : INT_MIN;
        result = *str - '0' + result * 10;
        str++;
    }

    return result * sign;
}
```

9. 是否是回文数字
------

限定不能用额外空间, 所以直接把 x 取余得到的数字作为一个反向作为一个新的数字

```c
bool isPalindrome(int x) {
    // tricky here, for x == k * 10^j
    if (x < 0 || x && (x % 10 == 0)) return false;
    int y = 0;
    while (x > y) {
        y = y * 10 + x % 10;
        x /= 10;
    }

    return x == y || x == y / 10; // 注意 x 可能是奇数长度也可能是偶数
}
```

10. 正则表达式
------

实现正则表达式, 只需要实现`.`代表任意字符, `*`代表任意重复。只需要特殊处理`*`，如果遇到了`*`，贪婪地向后匹配。
和通配符的不同之处在于，正则表达式需要两个字母组成模式，`*`是对前一个字母的修饰。

```c
bool isMatch(char* s, char* p) {
    for (char c = *p; c != 0;s++, c = *p) {
        // if next char in pattern is not *
        if (*(p+1) != '*')
            p++;
        // if we got an *, check if we can skip `.*` or `x*`
        else if (isMatch(s, p + 2))
            return true;

        // s ends or p and s differs
        if (*s == 0 || c != '.' && c != *s)
            return false;
    }
    return *s == 0;
}
```

11. Container with most water
------

从左右向中间逼近，如果有更大的就更新

```C++
int maxArea(vector<int>& height) {
    int left = 0, right = height.size() - 1;
    int result = 0;
    while (left < right) {
        result = max(result, min(height[left], height[right]) * (right - left));
        if (height[left] < height[right])
            left++;
        else
            right--;
    }
    return result;
}
```


12. 十进制转换为罗马数字
------

直接按每位把罗马数字转换出来在拼接就好了, 使用 C 的话, 拼接字符串很麻烦。

```c++
string intToRoman(int num) {
    // note, the leading empty string is the trick here
    string thousands[] = {"", "M", "MM", "MMM"};
    string handreds[] = {"", "C", "CC", "CCC", "CD", "D", "DC", "DCC", "DCCC", "CM"};
    string tens[] = {"", "X", "XX", "XXX", "XL", "L", "LX", "LXX", "LXXX", "XC"};
    string ones[] = {"", "I", "II", "III", "IV", "V", "VI", "VII", "VIII", "IX"};
    return thousands[num / 1000] + handreds[num % 1000 / 100] + tens[num % 100 / 10] + ones[num % 10];

}
```

13. 罗马数字转为十进制
------

主要是当前一个数字小于后一个数字的时候, 需要添加的是后一个谁和钱一个数字的差


```C
// acts like a dict or map
int getVal(char c) {
    switch (c) {
        case 'I': return 1;
        case 'V': return 5;
        case 'X': return 10;
        case 'L': return 50;
        case 'C': return 100;
        case 'D': return 500;
        case 'M': return 1000;
    }
}

int romanToInt(char* s) {
    int result = 0;
    for (int i = 0; s[i] != 0; ) {
        if (getVal(s[i]) < getVal(s[i+1]))
            result += getVal(s[i+1]) - getVal(s[i]), i += 2;
        else
            result += getVal(s[i]), i++;
    }
    return result;
}
```

14. 最长公共前缀
------

横向遍历, 从头到尾, 如果, 不一致, 返回当前子串即可。如果约定不能更改当前字符串的化, 最好用
C++做, 不然操作字符串太复杂了, 没必要出错。

```c
char* longestCommonPrefix(char** strs, int strsSize) {
    if (!strs || !strs[0]) return "";
    if (strsSize == 1) return strs[0];
    int len = strlen(strs[0]);

    for (int i = 0; i < len; i++) {
        for (int j = 1; j < strsSize; j++) {
            if (strs[j][i] != strs[0][i]) {
                strs[0][i] = '\0';
                return strs[0];
            }
        }
    }
    return strs[0];
}
```

15. 从数组中找出三个数使得他们的和是0
------

按照 LeetCode 的要求的话, 使用 C 做, 返回值太复杂了, 所以用 C++ 做了。
首先, 把数组排序, 然后使用类似 two sum 的方法做就好了

```C++
vector<vector<int>> threeSum(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> result;
    for (int i = 0; i < nums.size(); i++) {
        if (i > 0 && nums[i] == nums[i-1])
            continue;
        int k = nums.size() - 1;
        int j = i + 1;
        while (j < k) {
            if (nums[i] + nums[j] + nums[k] > 0)
                k--;
            else if (nums[i] + nums[j] + nums[k] < 0)
                j++;
            else {
                result.push_back({nums[i], nums[j], nums[k]});
                // skipping duplicates
                while (j < k && nums[k] == nums[k - 1])
                    k--;
                while (j < k && nums[j] == nums[j + 1])
                    j++;
                k--; // 别忘了这里，还要继续寻找下一组
                j++;
            }
        }
    }
    return result;
}
```

16. 在数组中找到三个数字使得他们得和尽可能的接近给定数字, 假设结果唯一
------

和上一题解法类似, 在 http://stackoverflow.com/q/2070359 有详尽解释

```C
int cmp(int* a, int* b) {
    return *a - *b;
}

int threeSumClosest(int* nums, int numsSize, int target) {
    if (numsSize <= 3) 
        return nums[0] + nums[1] + nums[2];
    qsort(nums, numsSize, sizeof(int), cmp);

    int result = nums[0] + nums[1] +nums[2];
    for (int i = 0; i < numsSize; i++) {
        int j = i + 1;
        int k = numsSize - 1;
        while (j < k) {
            int sum = nums[i] + nums[j] + nums[k];
            if (sum == target)
                return target;
            if (abs(target - sum) < abs(target - result))
                result = sum;
            if (sum > target)
                k--;
            else
                j++;
        }
    }
    return result;
}
```

E17. 生成电话键盘按键数字对应的所有可能的字符串, 不限制返回结果的顺序
------

![键盘](http://upload.wikimedia.org/wikipedia/commons/thumb/7/73/Telephone-keypad2.svg/200px-Telephone-keypad2.svg.png)

遍历数字, 设当前结果为`{a, b, c}`, 下一个数字是`3`, 找出对应的字母`{d, e, f}`, 则新的结果是

{ a + {def}, b + {def}, c + {def}}

然后把新获得的数组作为下一轮的初始数组。最开始时, 使用空数组开始。

```C++
// iterative
vector<string> letterCombinations(string digits) {
    if (digits.size() == 0) return vector<string> {};
    string mapping[] = {"", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};
    vector<string> combinations(1, ""); // 注意使用空字符串作为种子

    for (int i = 0; i < digits.size(); i++) {
        int digit = digits[i] - '0';
        if (mapping[digit].empty())
            continue;
        vector<string> temp;
        for (auto& c : mapping[digit])
            for (auto& combination : combinations)
                temp.push_back(combination + c);
        swap(combinations, temp);
    }
    return combinations;
}
```

还可以使用深度优先的搜索方法

```C++
// recursive
```

追问：如何通过用户按的数字来查找是否有对应的单词呢

1. 通过把所有的单词计算出来，然后查询哪个是合法的，查询可以使用Trie
2. 通过把已经有的单词字典转换为数字字典，然后通过数字序列查询可能的单词组合。

18. 4Sum
------

```C++
vector<vector<int>> fourSum(vector<int>& nums, int target) {
    vector<vector<int>> result;
    int n = nums.size();
    
    if (n < 4) return result;
    
    sort(nums.begin(), nums.end());
    unordered_map<int, vector<pair<int, int>>> hash;
    for(int i = 0; i < n; i++){
        for(int j = i + 1; j < n; j++){
            hash[nums[i]+nums[j]].push_back(make_pair(i,j));
        }
    }
    
    for (int i = 0; i < n; i++) {
        if (i > 0 && nums[i] == nums[i-1])
            continue;
        for (int j = i+1; j < n; j++) {
            if (j > i + 1 && nums[j] == nums[j-1])
                continue;
            int re = target - nums[i] - nums[j];
            if (hash.find(re) != hash.end()) {
                for (auto match : hash[re]) {
                    int k = match.first, l = match.second;
                    if (k > j) {
                        if (!result.empty()
                            && result.back()[0] == nums[i] && result.back()[1] == nums[j]
                            && result.back()[2] == nums[k] && result.back()[3] == nums[l])
                        continue;
                        result.push_back({nums[i], nums[j], nums[k], nums[l]});
                    }
                }
            }
        }
    }
    return result;
}
```

E19. 删除链表中倒数第 k 的节点
------

双指针经典题目, 一个快指针先走 k 步, 另一个慢指针再出发, 注意链表长度小于 k 时。
注意：LeetCode 给定的 n 都是有效地, 但要求返回头指针, 如果头指针被删除需要额外注意, 因此采用 dummy head

```C
struct ListNode* removeNthFromEnd(struct ListNode* head, int n) {
    struct ListNode dummy, *fast, *slow;
    dummy.next = fast = head;
    slow = &dummy;

    while (n--)
        fast = fast->next;
    while (fast) {
        fast = fast->next;
        slow = slow->next;
    }
    struct ListNode* next = slow->next;
    slow->next = next->next;
    free(next); // remeber to free memory
    return dummy.next;
}
```


E20. 判定给定的字符串是否是合法的括号序列, 可能包括大中小三类
------

使用栈的基础题，注意逻辑简化

```C
char opposite(char c) {
    switch (c) {
        case ')' : return '(';
        case ']' : return '[';
        case '}' : return '{';
    }
}

bool isValid(string s) {
    stack<char> stk;
    for (auto& c : s) {
        if (c == '(' || c == '[' || c == '{')
            stk.push(c);
        else if (!stk.empty() && stk.top() == opposite(c))
            stk.pop();
        else
            return false;
    }

    return stk.empty(); // 注意为空的条件
}
```

E21. 合并两个已经排序的链表
------

考察链表的基本操作, 很简单

```C
struct ListNode* mergeTwoLists(struct ListNode* l1, struct ListNode* l2) {
    if (l1 == NULL) return l2;
    if (l2 == NULL) return l1;
    struct ListNode dummy;
    dummy.next == NULL;
    struct ListNode* p = &dummy;
    while (l1 && l2) {
        if (l1->val < l2->val) {
            p->next = l1;
            l1 = l1->next;
        } else {
            p->next = l2;
            l2 = l2->next;
        }

        p = p->next;
    }

    if (l1)
        p->next = l1;

    if (l2)
        p->next = l2;

    return dummy.next;
}
```

22. 给定数字n, 生成所有合法的 n 个括号组成的序列
------

Cracking 上还提供了另一种复杂的思路

```C++
vector<string> generateParenthesis(int n) {
    vector<string> result;
    gen(result, "", n, n);
    return result;
}

// left 剩下的左括号，right 剩下的右括号
void gen(vector<string>& result, string s, int left, int right) {
    if (left == 0 && right == 0) {
        result.push_back(s);
        return;
    }
    if (left != 0)
        gen(result, s + '(', left - 1, right);
    if (left < right)
        gen(result, s + ')', left, right - 1);
}
```

M23. 合并 k 个已经排序的列表
------

把列表看做一个队列, 每次拿出两个列表, 合并他们后放回到列表中, 每次遍历列表的一半, 这样每次遍历完一遍, 
    列表的长度都会减半, 直到列表的长度为1,  合并函数使用21题中的合并两个列表的函数

```C
struct ListNode* mergeTwoLists(struct ListNode* l1, struct ListNode* l2) {
    // see above
}

struct ListNode* mergeKLists(struct ListNode** lists, int listsSize) {
    if (!lists || listsSize < 1)
        return NULL;

    while (listsSize > 1) {
        // listsize is halfed
        for (int i = 0; i < listsSize / 2; i++)
            // merge i and last i list
            lists[i] = mergeTwoLists(lists[i], lists[listsSize-1-i]);
        listsSize = (listsSize + 1) / 2; // 注意这里！

    }
    return lists[0];
}
```

24. 给定一个链表, 交换两个相邻节点的值
------

最简单的做法显然是直接把前后两个节点的值交换, 但是LeetCode规定不能改变节点的值。
主要考察链表的指针操作, 注意各种细节, 一定要在纸上先把链表画出来。

```C
struct ListNode* swapPairs(struct ListNode* head) {
    struct ListNode dummy, *temp, *pnext, *p = &dummy;
    dummy.next = head;
    while (p->next && p->next->next) {
        temp = p->next;
        p->next = temp->next;
        temp->next = p->next->next;
        p->next->next = temp;
        p = temp;
    }
    return dummy.next;
}
```

25. 给定一个链表, 把相邻的 k 个节点反转
------

和上题一样, 同样禁止改变节点的值。比较简单地解法是浪费一点空间, 使用
Stack, 实现逆转 k 个节点, 注意如果 k 较大的话, 这种方法是不合适的。

```C++
ListNode* reverseKGroup(ListNode* head, int k) {
    stack<ListNode*> stk;
    ListNode dummy(-1), *p = &dummy, *pp;
    dummy.next = head;
    while (1) {
        pp = p;
        for (int i = 0; i < k; i++) {
            if (pp->next) {
                stk.push(pp->next);
                pp = pp->next;
            } else {
                break;
            }
        }

        if (stk.size() < k) // 剩下的节点不够 k 个了
            return dummy.next;

        pp = stk.top()->next; // 下一组中的第一个
        while (!stk.empty()) {
            p->next = stk.top();
            stk.pop();
            p = p->next;
        }

        p->next = pp;
    }
}
```

26. 从已排序数组中删除重复元素, 并返回新数组的长度
------

in-place的删除重复元素, 使用两个指针, 一个遍历, 一个指向当前的结尾。

PS：这个基础题竟然做了半个小时才做对, ⊙﹏⊙b汗, 要加强基础啊！

```C
int removeDuplicates(int* nums, int numsSize) {
    if (numsSize <= 1) return numsSize;
    int len = 0;
    for (int i = 0; i < numsSize; i++)
        if (i == 0 || nums[i] != nums[len - 1])
            nums[len++] = nums[i];
    return len;
}
```

27. 删除元素
------

和上一题类似, 注意细节

```C
int removeElement(int* nums, int numsSize, int val) {
    if (!nums || numsSize == 0) return 0;
    int len = 0;
    for (int i = 0; i < numsSize; i++) {
        if (nums[i] != val)
            nums[len++] = nums[i];
    }
    return len;
}
```

28. 实现 strstr 函数, 即查找子串
------

使用暴力算法, 时间复杂度O(n)。也可以用 kmp 算法。

```C
/*
 * Brute Force
 */
int strStr(char* haystack, char* needle) {
    int h = strlen(haystack);
    int n = strlen(needle);
    if (n == 0) return 0;
    // note h - n + 1
    for (int i = 0; i < h - n + 1; i++) {
        for (int j = 0; j < n; j++) {
            if (needle[j] != haystack[i+j])
                break;
            if (j == n - 1)
                return i;
        }
    }
    return -1;
}
```

```C
/*
 * KMP
 */

int strStr(char* haystack, char* needle) {
    if (strlen(needle) == 0) return 0;
    return kmp(needle, haystack);
}

void construct(char* pattern, int* lps) {

    int n = strlen(pattern);
    lps[0] = 0;
    int i = 1, len = 0;
    while (i < n) {
        if (pattern[i] == pattern[len]) {
            lps[i++] = ++len;
        } else {
            if (len != 0)
                len = lps[len - 1];
            else
                lps[i++] = 0;
        }
    }
}

int kmp(char* needle, char* haystack) {

    int n = strlen(needle);
    int m = strlen(haystack);

    int* lps = malloc(sizeof(int) * n);
    construct(needle, lps);

    int i = 0, j = 0;
    while (i < m) {
        if (haystack[i] == needle[j])
            i++, j++;
        if (j == n) {
            return i - n;
            j = lps[j - 1];
        } else if (i < m && needle[j] != haystack[i]) {
            if (j != 0)
                j = lps[j - 1];
            else
                i++;
        }
    }

    free(lps);
    return -1;
}
```

29. 给定连个整数, 不使用乘法和除法计算除法。
------

[这里](https://leetcode.com/discuss/38997/detailed-explained-8ms-c-solution)有一个非常好的算法

计算可以从被除数中减去除数的次数

```c
int divide(int dividend, int divisor) {
    // abs(INT_MIN) == INT_MAX + 1
    if (divisor == 0 || (dividend == INT_MIN && divisor == -1)) 
        return INT_MAX;
    int sign = (dividend > 0) == (divisor > 0) ? 1 : -1;
    long long n = labs(dividend);
    long long d = labs(divisor);

    int result = 0;
    while (n >= d) {
        long long temp = d;
        long long multi = 1;
        while (n >= (temp << 1)) {
            temp <<= 1;
            multi <<= 1;
        }
        n -= temp;
        result += multi;
    }

    return sign * result;
}
```

30. 包含所有单词的子字符串，找出所有。单词的长度都是一样的
------

```C++
vector<int> findSubstring(string s, vector<string>& words) {
    unordered_map<string, int> counts;
    for (string word : words)
        counts[word]++;
    int n = s.length(), num = words.size(), len = words[0].size();
    vector<int> indexes;
    for (int i = 0; i < n - num * len + 1; i++) {
        unordered_map<string, int> seen;
        int j = 0;
        for (; j < num; j++) {
            string word = s.substr(i + j * len, len);
            if (counts.find(word) != counts.end()) {
                seen[word]++;
                if (seen[word] > counts[word])
                    break;
            } else {
               break;
            }
        }
        if (j == num) 
            indexes.push_back(i);
    }
    return indexes;
}
```

31. 全排列，下一个
------

首先, 对于所有的组合, 最小的一个一定是按照升序排序的, 最大的一定是倒过来, 因此

1. 如果我们发现是倒序的, 直接翻转就好了；
2. 如果是一般情况, 从后向前遍历, 找到逆序的数字的边界,  假设是 k。那么我们翻转

```C++
void nextPermutation(vector<int>& nums) {
    int k = -1;
    for (int i = nums.size() - 2; i >= 0; i--) {
        if (nums[i] < nums[i + 1]) {
            k = i;
            break;
        }
    }
    // 完全是逆序的，直接返回第一个，也就是升序排列
    if (k == -1) {
        reverse(nums.begin(), nums.end());
        return;
    }
    int l = -1;
    for (int i = nums.size() - 1; i > k; i--) {
        if (nums[i] > nums[k]) {
            l = i;
            break;
        } 
    } 
    swap(nums[k], nums[l]);
    reverse(nums.begin() + k + 1, nums.end()); 
}
```

32. 从一个括号构成的字符串中找出最长的合法括号序列
------

显然判定合法括号顺序的题都可以用栈来做, 但是不妨使用动态规划来尝试一下 😄

动态规划: 见注释

```C
int longestValidParentheses(char* s) {
    int len = strlen(s);
    // 遍历到当前位置时的最长序列
    int dp[len];
    for (int i = 0; i < len; i++)
        dp[i] = 0;
    int longest = 0;
    // 从倒数第二个位置开始遍历
    for (int i = len - 2; i >= 0; i--) {
        // 尝试把上一个序列包围住
        int match = i + dp[i+1] + 1;
        if (s[i] == '(' && match < len && s[match] == ')') {
            dp[i] = dp[i+1] + 2;
            // 拼接合法序列，注意 match + 1
            if (match + 1 < len)
                dp[i] += dp[match + 1];
        }
        longest = longest > dp[i] ? longest : dp[i];
    }
    return longest;
}
```

33. 在排序后又被反转的数组中搜索
------

既然是部分有序的,自然还是使用二分搜索了,注意终止条件.
不同于普通二分搜索的两种情况, 我们有了四种情况:

1. 前半部分有序, 并且在前半部分当中, 
2. 前半部分有序, 但是不在前半部分
3. 后半部分有序, 并且在后半部分
4. 后半部分有序, 但是不在后半部分

```C
int search(int* nums, int numsSize, int target) {
    int left = 0, right = numsSize - 1;

    // plain old binary search
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target)
            return mid;

        // left half is sorted
        if (nums[left] <= nums[mid]) {
            if (nums[left] <= target && target < nums[mid])
                right = mid - 1;
            else
                left = mid + 1;
            // right half is sorted
        } else {
            if (nums[mid] < target && target <= nums[right])
                left = mid + 1;
            else
                right = mid - 1;
        }
    }

    return -1;
}
```

34. 查找数组中一个重复出现数字的下界和上界, 数组已排序
------

在 C++的标准库中包含了这两个函数, 分别是`std::lower_bound`和`std::upper_bound`. 


```C++
vector<int> searchRange(vector<int>& nums, int target) {
    return vector<int> {lower(nums, target), upper(nums, target)}; 
}

int lower(vector<int>& nums, int target) {
    int first = 0, last = nums.size();
    while (first < last) {
        int middle = first + (last - first) / 2;
        if (target > nums[middle]) // 寻找下界
            first = middle + 1;
        else
            last = middle;
    }
    return nums[first] == target ? first : -1;
}

int upper(vector<int>& nums, int target) {
    int first = 0, last = nums.size();
    while (first < last) {
        int middle = first + (last - first) / 2;
        if (target >= nums[middle]) // 寻找上界only difference with lower
            first = middle + 1;
        else
            last = middle;
    }

    // note: std::upper_bound return offset by 1
    return nums[first - 1] == target ? first - 1 : -1;
}
```

35. 二分查找数字, 如果没有找到, 返回应该插入的位置
------

就是最基础的二分查找


```C
int searchInsert(int* nums, int numsSize, int target) {
    int left = 0, right = numsSize - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target)
            left = mid + 1;
        else
            right = mid - 1;
    }
    return left;
}
```


36. 合法数独, 给定一个数独表,判定当前是否合法
------

```C
// 有点浪费空间
bool isValidSudoku(char** board, int row, int col) {
    bool used_row[9][9] = {false}, used_col[9][9] = {false}, used_box[9][9] = {false};
    for (int i = 0; i < row; i++)
        for (int j = 0; j < col; j++)
            if (board[i][j] != '.') {
                int num = board[i][j] - '0' - 1;
                int k = i / 3 * 3 + j / 3;
                if (used_row[i][num] || used_col[j][num] || used_box[k][num])
                    return false;
                used_row[i][num] = used_col[j][num] = used_box[k][num] = true;
            }
    return true;
}
```

37. 求解数独
------

```C++
void solveSudoku(vector<vector<char>>& board) {
        solve(board, 0);
    }
    bool solve(vector<vector<char>>& board, int ind){
        if(ind==81) return true; 
        int i=ind/9, j=ind%9;
        if(board[i][j]!='.') 
            return solve(board, ind+1);
        else{
            for(char f = '1'; f <= '9'; f++) {
                if(isValidFill(board, i, j, f)) {
                    board[i][j]= f;
                    if(solve(board, ind+1)) return true;                
                    board[i][j]='.';
                }
            }
            return false;
        }
    }
    bool isValidFill(vector<vector<char>>& board, int i, int j, char fill) {
        for(int k=0; k<9; k++) {
            if(board[i][k]==fill) return false; //check the row
            if(board[k][j]==fill) return false; //check the column
            int r= i/3*3+j/3;   //select the block
            if(board[r/3*3+k/3][r%3*3+k%3]==fill) return false; //check the block
        }            
        return true;
    }
```

38. 数数并说出来
------

不太理解这道题有什么意义,直接暴力做出来了

```C++
string countAndSay(int n) {
    string result = "1";
    for (int i = 0; i < n -1; i++) {
        string temp;
        for (int j = 0; j < result.size(); j++) {
            int count = 1;
            while (j + 1 < result.size() && result[j+1] == result[j]) {
                j++; count++;
            }
            temp += count + '0';
            temp += result[j];
        }
        result = temp;
    }
    return result;
}
```

39. 给定一个集合，在集合中找出和为target的数字，数字可以使用多次，集合中没有重复数字
------

典型的深度优先搜索

```C++
vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
    vector<vector<int>> result;
    dfs(result, candidates, {}, target);
    return result;
}

void dfs(vector<vector<int>>& result, vector<int>& candidates, vector<int> comb, int target) {
    if (target == 0) {
        result.push_back(comb);
        return;
    }
    
    for (auto c : candidates) {
        if (target - c < 0) continue; // 数字太大了
        if (!comb.empty() && c < comb.back()) continue; // 保证不重复
        comb.push_back(c);
        dfs(result, candidates, comb, target - c);
        comb.pop_back();
    }
}
```

40. 同上提一样，但是集合中的数字只能使用一次，但是集合中有重复数字
------

```C++
vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
    vector<vector<int>> result;
    sort(candidates.begin(), candidates.end());
    dfs(result, candidates, {}, target, 0);
    return result;
}

void dfs(vector<vector<int>>& result, vector<int>& candidates, vector<int> comb, int target, int start) {
    if (target == 0) {
        result.push_back(comb);
        return;
    }
    
    for (int i = start; i < candidates.size(); i++) {
        if (candidates[i] > target)
            break;
        if (i != start && candidates[i] == candidates[i-1])
            continue;
        comb.push_back(candidates[i]);
        dfs(result, candidates, comb, target - candidates[i], i + 1);
        comb.pop_back();
    }
}
```

41. 给定一个数组,找到第一个缺失的正数
------

显然,结果的范围是[1..n+1]. 而数组的长度为 n 我们把每个位置都放上 i+1,
这样如果有位置不是 i+1, 则找到了结果, 如果都相等则是 n+1.

```c
void swap(int* a, int* b) {
    int t = *a; *a = *b; *b = t;
}

int firstMissingPositive(int* nums, int numsSize) {
    for (int i = 0; i < numsSize; i++)
        // 注意此处的 while
        while (nums[i] > 0 && nums[i] <= numsSize && nums[i] != nums[nums[i] - 1])
            swap(&nums[i], &nums[nums[i] - 1]);
    
    for (int i = 0; i < numsSize; i++)
        if (nums[i] != i + 1)
            return i + 1;
    
    return numsSize + 1;
}
```

42. 给定一个数组表示柱子的高度，求能存贮的雨水的总量
------

从两边向中间收拢

```C
int trap(int* height, int heightSize) {
    int left = 0, right = heightSize - 1;
    int water = 0;
    int max_left = 0, max_right = 0;
    
    // 从两侧向中间缩小, 可以算作是两个指针吧
    while (left <= right) {
        if (height[left] <= height[right]) {
            if (height[left] >= max_left)
                max_left = height[left];
            else
                water += max_left - height[left];
            left++;
        } else {
            if (height[right] >= max_right)
                max_right = height[right];
            else
                water += max_right - height[right];
            right--;
        }
    }
    return water;
}
```

43. 给定两个任意长的字符串，返回一个字符串，代表他们相乘的结果
------

按整数除法运算即可，重点是下标的表示

```C
#define tonum(c) (c - '0')
#define tochar(i) (i + '0')

char* multiply(char* num1, char* num2) {
    // 结果的长度不会超过 m+n,
    // 假设某个数是 n 位的9, 则结果比另一个数结尾加上 n 个 0还小
    int n = strlen(num1), m = strlen(num2);
    int len = m+n;
    char* result = malloc(sizeof(char) * (len + 1));
    for (int i = 0; i < len; i++)
        result[i] = '0';
    result[len] = '\0';

    for (int i = n - 1; i >= 0; i--) {
        int carry = 0;
        for (int j = m - 1; j >= 0; j--) {
            int v = tonum(result[i+j+1]) +  tonum(num1[i]) * tonum(num2[j]) + carry;
            result[i+j+1] = tochar(v % 10);
            carry = v / 10;
        }
        result[i] += carry;
    }
    
    for (int i = 0; i < len; i++)
        if (result[i] != '0')
            return result+i;
    
    return "0";
}
```

44. 通配符匹配，`?` 代表任意一个字符，`*`代表任意一个或多个字符
------

注意和正则表达式的区别，要求完全匹配。这道题的关键在于对星号的处理, 如果出现星号的时候, 我们记录当时的p 和 s 的值, 如果发生了不匹配的话, 我们尝试回到该位置的下一个位置开始匹配

```C

bool isMatch(char* s, char* p) {
    char* star = NULL;
    char* revert = s;
    while (*s) {
        if (*s == *p || *p == '?')
            s++, p++;
        else if (*p == '*')
            star = p++, revert = s;
        else if (star) 
            p = star + 1, s = ++revert;
        else
            return false;
    }
    
    // 如果剩下了 p, 那应该全都是*才对
    while (*p) {
        if (*p++ != '*')
            return false;
    }
    
    return true;
}
```

45. 跳跃游戏，给定一个数组，每个数字是在该位置可以向前移动的距离，返回最少需要多少次才能到达终点
------

比较简单，看注释吧

```C
int jump(int* nums, int numsSize) {
    int steps = 0;
    int last = 0; // last range
    int cur = 0; // current range
    
    for (int i = 0; i < numsSize; i++) {
        // beyond range, make another jump
        if (i > last)
            last = cur, steps++;
        // if we could reach longer?
        if (nums[i] + i > cur)
            cur = nums[i] + i;
    }
    return steps;
}
```

46. 生成全排列
------

Cracking上给出了一种解法，通过不断的添加下一个元素到上一组元素的不同位置来生成全排列，这样固然可以，但是大规模的拼接数组或者字符串是很耗费资源的。

在已经有了字符串（或者数组）的初始排列以后，可以通过不断交换的方法生成每一组全排列。
比如对于xyz，我们有全排列为

    x + per(yx)
    y + per(xz)
    z + per(xy)

那么我们通过把每个元素交换到第一个位置，就把问题规模缩小了，知道把问题规模缩小为1.

```C++
vector<vector<int>> permute(vector<int>& nums) {
    vector<vector<int>> result;
    per(result, nums, 0);
    return result;
}

void per(vector<vector<int>>& result, vector<int>& nums, int begin) {
    if (begin >= nums.size()) {
        result.push_back(nums);
        return;
    }
    
    for (int i = begin; i < nums.size(); i++) { // 注意是从begin开始，这样未改变的才能加入进来
        swap(nums[begin], nums[i]);
        per(result, nums, begin + 1);
        swap(nums[begin], nums[i]); // 注意因为参数中是传引用，这里需要复原
    }
}
```

47. 全排列，数组中有重复元素
------

和上一题基本是一样的，注意跳过重复元素就好了

```C++
vector<vector<int>> permuteUnique(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> result;
    per(result, nums, 0);
    return result;
}

void per(vector<vector<int>>& result, vector<int> nums, int start) {
    if (start >= nums.size()) {
        result.push_back(nums);
        return;
    }
    
    for (int i = start; i < nums.size(); i++) {
        if (start != i && nums[start] == nums[i])
            continue;
        swap(nums[start], nums[i]);
        per(result, nums, start + 1); // 事实证明，传引用反倒会超时
    }
}
```

48. 给定一个`n*n`的图像旋转图像，顺时针旋转90度
------

做法显然是从里到外，一层一层的旋转，这道题主要考察下标的操作

```C
void rotate(int** matrix, int m, int n) {
    for (int layer = 0; layer < n / 2; layer++) {
        int first  = layer;
        int last = n - 1 - layer;
        for (int i = first; i < last; i++) {
            int offset = i - first;
            int top = matrix[first][i];
            // up <- left
            matrix[first][i] = matrix[last-offset][first];
            // left <- down
            matrix[last-offset][first] = matrix[last][last-offset];
            // down <- right
            matrix[last][last-offset] = matrix[i][last];
            // right <- up
            matrix[i][last] = top;
        }
    }
}
```

49. 给定字符数组，把他们按照Anagram分组
------

```C++
// Anagram 分组
// 这道题没什么可做的，只需要统计
vector<vector<string>> groupAnagrams(vector<string>& strs) {
    vector<vector<string>> result;
    string temp;
    unordered_map<string, vector<string>> records;
    for (int i = 0; i < strs.size(); ++i) {
        temp = strs[i];
        sort(temp.begin(), temp.end());
        records[temp].push_back(strs[i]);
    }
    for (auto& record : records) {
        sort(record.second.begin(), record.second.end());
        result.push_back(record.second);
    }
    return result;
}
```

回头再写

50. 实现pow(x, n)
------

显然不能直接阶乘过去，分治法

```C
double myPow(double x, int n) {
    if (n == INT_MIN) return myPow(x, n - 1) * x;
    if (n < 0) return 1 / myPow(x, -n);
    if (n == 0) return 1;
    if (n == 1) return x;
    double y = myPow(x, n / 2);
    if (n & 0x1) 
        return y * y * x;
    else
        return y * y;
}
```

51. N 皇后问题
------

需要大幅度修改

```C++
// N皇后问题，皇后不能再一条直线，一条竖线，一条斜线上
    
// 使用深度优先求解，对于 dfs 问题，我们首先把算法的框架写下来，然后确定这个问题的限制条件
// 对于这个问题，限制条件当前行的元素不能在以前的列中出现过，也不能在对角线中出现过
vector<vector<string>> result;

vector<vector<string>> solveNQueens(int n) {
    if (n < 1) return result;
    vector<int> x(n);
    dfs(0, x, n);
    return result;
    
}

void dfs(int t, vector<int>& x, int n) {
    // 当新添加一个 Q 到当前解的时候
    if (t >= n) {
        // result.push_back(make_solution(x));
        // return;
        vector<string> solution;
        for (int i = 0; i < n; i++) {
            string line(n, '.');
            line[x[i]] = 'Q';
            solution.push_back(line);
        }
        result.push_back(solution);
    } else {
        for (int i = 0; i < n; i++) {
            bool skip = false;
            for (int j = 0; j < t; j++) {
                if (x[j] == i || abs(i - x[j]) == abs(t - j)) {
                    skip = true;
                    break;
                }
            }
            if (skip) continue;
            x[t] = i;
            dfs(t+1, x, n);
        }
    }
}
```

52. N 皇后一共有多少个解
------

不要直接把皇后放好，而是把占用的都记录下来，然后继续深度优先搜索

```C++
class Solution {
public:
    unordered_set<int> cols, digs1, digs2;
    int totalNQueens(int n) {
        return total(0, 0, n);
    }
    
    int total(int row, int count, int n) {
        for (int col = 0; col < n; col++) {
            if (cols.find(col) != cols.end()
                || digs1.find(row - col) != digs1.end()
                || digs2.find(row + col) != digs2.end())
            continue;
            
            if (row == n-1)
                count++;
            else {
                cols.insert(col);
                digs1.insert(row-col);
                digs2.insert(row+col);
                count = total(row+1, count, n);
                cols.erase(col);
                digs1.erase(row-col);
                digs2.erase(row+col);
            }
        }
        return count;
    }
};
```

^53. 最大子序列和
------

动态规划经典题目，遍历数组，如果已经当前子序列已经小于0了，抛弃并置 sum = 0
如果比当前和更大，更新。对于一个子序列，要么使得序列和增大，要么减小。

`dp[n+1] = max(dp[n], dp[n] + A[n+1])`

```C
int maxSubArray(int* nums, int numsSize) {
    int sum = 0;
    int m = INT_MIN;

    for (int i =0; i< numsSize; i++) {
        sum += nums[i];
        if (sum > m)
            m = sum;
        if (sum < 0)
            sum = 0;
    }
    return m;
}
```

54. 顺时针螺旋打印矩阵
------

一圈一圈地打印就好了

```C
int* spiralOrder(int** matrix, int row, int col) {
    if (row == 0 || col == 0) return NULL;
    int top = 0, right = col - 1, down = row - 1, left = 0;
    int index = 0;
    int* result = malloc(sizeof(int) * row * col);
    while (top <= down && left <= right) {
        for (int i = left; i <= right; i++)
            result[index++] = matrix[top][i];
        top++; //
        for (int i = top; i <= down; i++)
            result[index++] = matrix[i][right];
        right--; // 
        // 注意这个 if 语句
        if (top <= down)
            for (int i = right; i >= left; i--)
                result[index++] = matrix[down][i];
        down--; // 
        // 注意这个 if 语句
        if (left <= right)
            for (int i = down; i >= top; i--)
                result[index++] = matrix[i][left];
        left++; // 
    }
    return result;
}
```

55. 给定一个数组，每个数字表示在当前步可以移动的距离，返回是不是能够到达终点
------

使用动态规划求解，如果当前距离大于最远距离，更新最远距离，如果已经超过了最远距离，跳出

```C
bool canJump(int* nums, int numsSize) {
    int i;
    int reach = 0;
    for (i = 0; i < numsSize && i <= reach; i++)
            reach = max(reach, nums[i] + i);
    return i == numsSize;
}
```

56. 合并序列，给定一组序列，把其中重叠的序列合并
------

这道题用 Python 做竟然比用 C++ 还要快

```Python
"""
class Interval(object):
    def __init__(self, start=0, end=0):
        self.start = start
        self.end= end
"""

def merge(intervals):
    intervals.sort(key=lambda x: x.start)
    combined = []
    for interval in intervals:
        if combined and interval.start <= combined[-1].end:
            combined[-1].end = max(combined[-1].end, interval.end)
        else:
            combined.append(interval)
    return combined
```

57. 添加序列，给定一组已经排序的序列，向其中插入一个序列，需要合并的合并
------

这道题inplace 的做法感觉没有什么意义，因为如果在中间插入的话，后半部分还是要被拷贝

```Python
def insert(self, intervals, newInterval):
    result = []
    start = newInterval.start
    end = newInterval.end
    remainder = 0
    for interval in intervals:
        if start <= interval.end:
            if end < interval.start:
                break
            start = min(start, interval.start)
            end = max(end, interval.end)
        else:
            result.append(interval)
        remainder += 1
    result.append(Interval(start, end))
    result += intervals[remainder:]
    return result
```

58. 给定一个字符串，求其中最后一个单词的长度
------

显然这道题可以用 strlen 求出长度然后从后往前数，但是，这样相当于多遍历了一次
直接从后往前可以保证只遍历一次

```C
int lengthOfLastWord(char* s) {
    int len = 0;
    bool inWord = false;
    while (*s) {
        if (isspace(*s)) {
            inWord = false;
        } else {
            if (!inWord) {
                len = 1;
                inWord = true;
            } else {
                len++;
            }
        }
        s++;
    }
    return len;
}
```

59. 给定 n，把1, 2, 3 ...螺旋打印到矩阵中
------

和上一个完全一样的思路，只是这次是打印罢了

```C
/**
 * Return an array of arrays.
 * Note: The returned array must be malloced, assume caller calls free().
 */
int** generateMatrix(int n) {
    int** matrix = malloc(sizeof(int*) * n);
    for (int i = 0; i < n; i++)
        matrix[i] = malloc(sizeof(int) * n);
        
    int top = 0, left = 0, down = n - 1, right = n - 1;
    int a = 1;
    while (top <= down && left <= right) {
        for (int i = left; i <=right; i++)
            matrix[top][i] = a++;
        top++;
        for (int i = top; i <= down; i++) {
            matrix[i][right] = a++;
        }
        right--;
        if (top <= down)
            for (int i = right; i >= left; i--)
                matrix[down][i] = a++;
        down--;
        if (left <= right)
            for (int i = down; i >= top; i--)
                matrix[i][left] = a++;
        left++;
    }
    return matrix;
}
```

60. 给定n个数字，找出第k个Permutation
------

```C++
class Solution {
public:
    /*The logic is as follows: 
    for n numbers the permutations can be divided to (n-1)! groups, 
    thus k/(n-1)! indicates the index of current number, 
    and k%(n-1)! denotes remaining sequence (to the right). 
    We keep doing this until n reaches 0, then we get n numbers permutations that is kth. 
    */
    string getPermutation(int n, int k) {
        int f = 1;
        string s(n, '0');
        for (int i = 1; i <= n; i++) {
            f *= i;
            s[i-1] = i + '0';
        }
        // 给定n,一共有n!个序列，f == n!
        
        k--;
        for (int i = 0; i < n; i++) {
            f /= n - i; // f /= n, f /= n - 1 ...
            int j = i + k / f;
            char c= s[j];
            for (;j > i; j--) // shift space to put `c`, actually we could use swap
                s[j] = s[j-1];
            s[i] = c;
            k %= f;
        }
        
        return s;
    }
};
```

61. 把列表旋转到倒数第 k 位
------

需要注意的是 k 大于列表长度的情况，这时候需要取余

```C
struct ListNode* rotateRight(struct ListNode* head, int k) {
    if (!head || k <= 0) return head;
    
    int l = 1;
    struct ListNode* n = head;
    while (n->next) {
        n = n->next;
        l++;
    }
    // n is now the tail!
    
    if (k >= l) k %= l;
    if (k == 0) return head;
    
    struct ListNode dummy, *p = &dummy;
    dummy.next = head;
    int i = l - k;
    while (i--)
        p = p->next;
    
    dummy.next = p->next;
    p->next = NULL;
    n->next = head;
    
    return dummy.next;
}
```

62. 给定一个`m*n`的矩阵，有多少种方法从左上角移动到右下角
------

显然可以使用组合数学直接求出来解，但是容易溢出。而且这是一道经典的动态规划题目，对于
每个格子，可以从他的上部或者左面移动过来。

```C++
int uniquePaths(int m, int n) {
    vector<vector<int>> grid(m, vector<int> (n, 1));
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            grid[i][j] = grid[i - 1][j] + grid[i][j - 1];
    return grid[m - 1][n - 1];
}
```

63. 同上题，区别是在一些位置是有障碍物的
------

经过分析可知，递推关系是一样的，只需要把有障碍格子的到达方法设定为0。这个主要是实现上的一些技巧，
见注释。

```C++
int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
    int m = obstacleGrid.size(), n = obstacleGrid[0].size();
    // 注意设定长宽均 +1，但是初始化为0，边界就成了障碍
    vector<vector<int>> pathes(m + 1, vector<int> (n + 1, 0));
    pathes[0][1] = 1; // 给定一个入口
    for (int i = 1; i < m + 1; i++)
        for (int j = 1; j < n + 1; j++)
            // 注意此处的偏移
            if (obstacleGrid[i-1][j-1] == 1)
                pathes[i][j] = 0;
            else
                pathes[i][j] = pathes[i-1][j] + pathes[i][j-1];
    return pathes[m][n]; 
}
```

64. 给定一个`m*n`矩阵，每个数字代表经过该处的耗费，找出一条耗费最小的路径
------

依然是动态规划

```C++
int minPathSum(vector<vector<int>>& grid) {
    // if modifying the grid is disallowed, copy it 
    int m = grid.size(), n = grid[0].size();
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            if (i == 0 && j == 0)
                continue;
            else if (i == 0 && j != 0)
                grid[i][j] += grid[i][j-1];
            else if (i != 0 && j == 0)
                grid[i][j] += grid[i-1][j];
            else
                grid[i][j] += min(grid[i-1][j], grid[i][j-1]);
    return grid[m-1][n-1];
}
```

65. 判定一个字符串是否是合法的数字，包括了正负号，小数点，`e`等
------

一些例子：

    "0" => true
    " 0.1 " => true
    "abc" => false
    "1 a" => false
    "2e10" => true

这道题就是细节题，用 C 处理字符串太蛋疼了，直接上 Python 了

```Python
def isNumber(self, s):
    BEFORE = 0 # before dot
    AFTER = 1 # after dot
    EXP = 2 # after e
    phase = BEFORE
    allow_sign = True

    s = s.strip()

    if not any([c.isdigit() for c in s]):
        return False

    if s == '' or s[0] == 'e' or s[-1] == 'e' or s == '.':
        return False
    if s[0] == '.' and s[1] == 'e':
        return False
    if s[0] == '-' and s[1] == 'e':
        return False

    for c in s:
        if '0' <= c <= '9':
            allow_sign = False
        elif c == '.':
            allow_sign = False
            if phase == EXP or phase == AFTER:
                return False
            else:
                phase = AFTER
        elif c == 'e':
            if phase == EXP:
                return False
            allow_sign = True
            phase = EXP

        elif c == '-' or c == '+':
            if not allow_sign:
                return False
            allow_sign = False
        else:
            return False

    if phase == EXP:
        return s[-1].isdigit()

    return True
```

66. 给定一个字符串代表的数字，返回加1后的数字
------

我们知道只有当数字是999....999的时候，才会使得数字的长度+1变为1000...000。

```C++
vector<int> plusOne(vector<int>& digits) {
    int n = digits.size();
    for (int i = n - 1; i >= 0; i--) {
        if (digits[i] == 9) {
            digits[i] = 0;
        } else {
            digits[i]++;
            return digits;
        }
    }
    // real trick here, we know that the number is 999...999
    // 改成insert
    digits[0] = 1;
    digits.push_back(0);
    return digits;
}
```

67. 给定两个字符串代表的二进制数字，返回他们相加的和
------

和上一题一样，按照加法定义做就好了

```C
#define tonum(c) (c - '0')
#define tochar(i) (i + '0')

char* addBinary(char* a, char* b) {
    int m = strlen(a), n = strlen(b);
    int len = (m > n ? m : n) + 1; // strlen(c)
    char* c = malloc(sizeof(char) * len + 1); // with ending null
    memset(c, '0', len+1);
    c[len] = 0;
    int carry = 0;
    for (int i = 1; i <= len; i++) {
         c[len-i] = tochar((i <= m ? tonum(a[m-i]) : 0) ^ (i <= n ? tonum(b[n-i]) : 0) ^ carry);
         carry = ((i <= m ? tonum(a[m-i]) : 0) + (i <= n ? tonum(b[n-i]) : 0) + carry) >> 1;
    }
    return c[0] == '0' ? c+1 : c;
}
```

68. 文字对齐
------

待研究

```C++
vector<string> fullJustify(vector<string>& words, int L) {
    vector<string> res;
    for(int i = 0, k, l; i < words.size(); i += k) {
        for(k = l = 0; i + k < words.size() and l + words[i+k].size() <= L - k; k++) {
            l += words[i+k].size();
        }
        string tmp = words[i];
        for(int j = 0; j < k - 1; j++) {
            if(i + k >= words.size()) tmp += " ";
            else tmp += string((L - l) / (k - 1) + (j < (L - l) % (k - 1)), ' ');
            tmp += words[i+j+1];
        }
        tmp += string(L - tmp.size(), ' ');
        res.push_back(tmp);
    }
    return res;
}
```

69. 给定整数x，求 sqrt(x)
------

比较坑的是 LeetCode 要求的是 `y*y < x` 的最大整数

```C
int mySqrt(int x) {
    if (x <= 1) return x;
    const double EPS = x * 0.0001;
    double y = x / 2; // initial guess
    while (fabs(y * y - x) > EPS) {
        y = (y + x / y) / 2;
    }
     
    long z = (long) y;
    while (z * z > x) z--;
    return z;
}
```

70. 爬梯子，一次可以爬一步或者两步，有几种方法爬完梯子
------

斐波那契数列，也可以理解为动态规划

```C
int climbStairs(int n) {
    int a = 1, b = 1, t;
    for (int i = 1; i < n; i++)
        t = b, b += a, a = t;
    return b;
}
```

71. 简化 Unix 路径，需要处理`.`, `..` 和多个斜杠等情况
------

没有什么需要注意的，主要是使用 stringstream 用作string.split

```C++
string simplifyPath(string& path) {
    vector<string> dirs;
    stringstream ss(path);
    string dir;
    while (getline(ss, dir, '/')) {
        if (dir == "." || dir == "")
            continue;
        else if (dir == "..") {
            if (!dirs.empty())
            dirs.pop_back();
        } else
            dirs.push_back(dir);
    }
    string result;
    for (auto& dir : dirs)
        if (!dir.empty())
            result += "/" + dir;
    return result.size() ? result : "/";
}
```

72. 编辑距离，允许替换，删除，插入三种操作
------

对于两个字符串比较，往往要使用二维的动态规划。
使用f[i][j]表示word1[1..i]和word2[1..j]之间的距离。
see [here](https://leetcode.com/discuss/43398/20ms-detailed-explained-c-solutions-o-n-space)

那么：

1. 相等 f[i][j] = f[i-1][j-1];
2. 不相等

    1. 替换: f[i][j] = f[i-1][j-1] + 1;  都向前一步
    2. 添加: f[i][j] = f[i][j-1] + 1; word2向前一步
    3. 删除: f[i][j] = f[i-1][j] + 1; word1向前一步

另外使用一维数组表示二维数组还需要了解
```C++

// unoptimized code
int minDistance(string word1, string word2) { 
    int m = word1.length(), n = word2.length();
    vector<vector<int> > dp(m + 1, vector<int> (n + 1, 0));
    for (int i = 1; i <= m; i++)
        dp[i][0] = i;
    for (int j = 1; j <= n; j++)
        dp[0][j] = j;  
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1[i - 1] == word2[j - 1]) 
                dp[i][j] = dp[i - 1][j - 1];
            else dp[i][j] = min(dp[i - 1][j - 1] + 1, min(dp[i][j - 1] + 1, dp[i - 1][j] + 1));
        }
    }
    return dp[m][n];
}
```

```C++
// optimized
int minDistance(string word1, string word2) {
    int m = word1.length(), n = word2.length();
    vector<int> cur(m + 1, 0);
    // 把剩余的字符删掉的距离
    for (int i = 1; i <= m; i++)
        cur[i] = i;
    for (int j = 1; j <= n; j++) {
        int pre = cur[0];
        cur[0] = j;
        for (int i = 1; i <= m; i++) {
            int temp = cur[i];
            if (word1[i - 1] == word2[j - 1])
                cur[i] = pre;
            else cur[i] = min(pre + 1, min(cur[i] + 1, cur[i - 1] + 1));
            pre = temp;
        }
    }
    return cur[m]; 
}
```

```C++
// recursive code from beauty of programming
// TLE on LeetCode
int minDistance(string word1, string word2) {
    return minDistance(&word1.front(), &word1.back(), &word2.front(), &word2.back())
}

int minDistance(char* start1, char* end1, char* start2, char* end2) {
    if (start1 > end1) 
        return start2 > end2 ? 0 : end2 - start2 + 1;

    if (start2 > end2)
        return start1 > end1 ? 0 : end1 - start1 + 1;

    if (*start1 == *start2)
        return minDistance(start1 + 1, end1, start2 + 1, end2);
    else {
        int t1 = minDistance(start1 + 1, end1, start2 + 1, end2);
        int t2 = minDistance(start1 + 1, end1, start2, end2);
        int t3 = minDistance(start1, end1, start2 + 1, end2);
        return min(t1, min(t2, t3)) + 1;
    }
}
```

73. 给定一个矩阵，如果某个元素为零，把所在的行和所在的列都设为零
------

一种可以接受的方法是使用O(m+n)的空间，记录哪行哪列需要设为零

```C++
void setZeroes(vector<vector<int>>& matrix) {
    int m = matrix.size();
    if (m == 0) return;
    int n = matrix[0].size();
    if (n == 0) return;
    
    vector<bool> row(m), column(n);
    
    for (int i = 0; i < m; ++i)
        for (int j = 0; j < n; ++j)
            if (matrix[i][j] == 0)
                row[i] = true, column[j] = true;
    
    for (int i = 0; i < m; ++i)
        for (int j = 0; j < n; ++j)
            if (row[i] || column[j])
                matrix[i][j] = 0;
}
```

74. 搜索矩阵，矩阵每行从左到右依次增大，每行都比上一行大
------

当做数组直接二分搜索就可以了

```C++
bool searchMatrix(int** matrix, int row, int col, int target) {
    int left = 0, right = row * col - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (matrix[mid/col][mid%col] < target)
            left = mid + 1;
        else if (matrix[mid/col][mid%col] == target)
            return true;
        else
            right = mid - 1;
    }
    return false;
}
```

75. 颜色排序，每个物体有颜色属性，把他们按照RGB的顺序排序(🇳🇱国旗问题)
------

一种方法是简单地2 pass解法，遍历一遍计数再输出。另一种方法是把红色往前交换，蓝色往后交换

```

void swap(int* a, int* b) {
    int t = *a; *a = *b; *b = t;
}

void sortColors(int* nums, int numsSize) {
    const int RED = 0, GREEN = 1, BLUE = 2;
    int reds = 0,  blues = numsSize - 1;
    for (int i = 0; i <= blues; i++) {
        while (nums[i] == BLUE && i < blues) swap(&nums[i], &nums[blues--]);
        while (nums[i] == RED && i > reds) swap(&nums[i], &nums[reds++]);
    }
}
```

76. 跳过
------

77. 给定数字 n 和 k，生成从 n 中取出 k 个数字的所有情况
------

数学上的组合，使用回溯来做，对状态空间进行深度搜索。

回溯方法通常适合对状态空间树的深度优先搜索相结合的，当一个解已经不满足条件时，剪枝；
如果满足条件，直到找到完全解未知。

```C++
// 组合是不要求顺序的
vector<vector<int>> combine(int n, int k) {
    vector<vector<int>> result;
    if (n < k)
        return result;
    vector<int> temp(0, k);
    combine(result, temp, 0, 0, n, k);
    return result;
}

void combine(vector<vector<int>>& result, vector<int>& temp, int start, int count, int n, int k) {
    // 1. 回溯条件，找到了一个解
    if (count == k) {
        result.push_back(temp);
        return;
    }
    // 2. 深度优先搜索
    for (int i = start; i < n; i++) {
        temp.push_back(i + 1);
        // 只搜索比 i 大的即可
        combine(result, temp, i+ 1, count+1, n, k);
        temp.pop_back();
    }
}
```

78. 给定一个集合，找到它的所有子集
------

这道题至少有3种解法：

1. DFS，我们知道对于 n 个元素的集合，有2^n个子集，通过每个元素在不在子集中构造一个状态空间树
2. 类似于电话键盘生成字母，迭代
3. 巧妙的利用1..2^n对应

```C++
// use backtracking and do a dfs search
    
vector<vector<int>> subsets(vector<int>& nums) {
    vector<vector<int>> result;
    if (nums.empty()) return result;
    sort(nums.begin(), nums.end());
    vector<int> temp;
    subsets(nums, result, temp, 0);
    return result;
}

// for each solution, the can be divided into two sub solutions: in or out
void subsets(vector<int>& nums, vector<vector<int>>& result, vector<int> temp, int i) {
    if (i == nums.size()) {
        result.push_back(temp);
        return;
    }
    
    vector<int> t = temp;
    subsets(nums, result, temp, i + 1);
    temp.push_back(nums[i]);
    subsets(nums, result, temp, i + 1);
}
```

```C++
// iterative
vector<vector<int>> subsets(vector<int>& nums) {
    vector<vector<int>> result;
    result.push_back({});
    if (nums.empty())
        return result;
    result.push_back(vector<int>(1, nums[0]));
    for (int i = 1; i < nums.size(); i++) {
        int size = result.size(); // notice the cached size
        for (int j = 0; j < size; j++) {
            auto new_subset = result[j];
            new_subset.push_back(nums[i]);
            sort(new_subset.begin(), new_subset.end());
            result.push_back(new_subset);
        }
    }
    return result;
}
```

```C++
// tricky
vector<vector<int>> subsets(vector<int>& nums) {
    vector<vector<int>> result;
    int size = (1 << nums.size());
    for (int i = 0; i < size; i++) {
        vector<int> subset;
        int k = i;
        for (int j = 0; j < nums.size(); j++) {
            if (k & 0x1)
                subset.push_back(nums[j]);
            k >>= 1;
        }
        sort(subset.begin(), subset.end());
        result.push_back(subset);
    }
    return result;
}
```

79. 给定一个二维字符数组，查找一个单词是否能够有连续的字母构成，不能交叉
------

也是深度优先的做法，首先找到开始的字母，然后依次向上下左右查找，注意还需要统计有没有访问过

```C++
bool exist(vector<vector<char>>& board, string word) {
    int row = board.size();
    int col = board[0].size();
    vector<vector<bool>> visited(row, vector<bool> (col, false));
    
    bool found = false;
    for (int i = 0; i < row; i++) {
        for (int j = 0; j < col; j++) {
            if (board[i][j] == word[0]) {
                if (findNext(board, word, visited, i, j, 0))
                    found = true;
            }
        }
    }
    return found;
}

bool findNext(vector<vector<char>>& board, string& word, vector<vector<bool>>& visited, int m, int n, int i) {
    
    if (i == word.size())
        return true;
    if (m >= board.size() || n >= board[0].size() || m < 0 || n < 0|| visited[m][n] || board[m][n] != word[i])
        return false;
    char temp = board[m][n];
    board[m][n] = -1;
     
    bool exist = findNext(board, word, visited, m + 1, n, i+1) ||
        findNext(board, word, visited, m - 1, n, i+1) ||
        findNext(board, word, visited, m, n+1, i+1) ||
        findNext(board, word, visited, m, n-1, i+1);
    board[m][n] = temp;
    return exist;
}
```

80. 从排序数组中删除重复元素，但是允许一个元素重复出现两次
------

巧妙地解法，和`i-2`的元素对比

```C
int removeDuplicates(int* nums, int numsSize) {
    if (!nums || numsSize < 1) return 0;
    int len = 0, counter = 0;
    for (int i = 0; i < numsSize; i++) {
        if (len < 2 || nums[i] != nums[len-2])
            nums[len++] = nums[i];
    }
    return len;
}
```

81. 在被翻转的数组中查找元素，可能包含重复元素
------

经典题目，还是一个二分查找问题，只是要分很多种情况

```C
bool search(int A[], int n, int key) {
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = left + (right - left)/2;
        if (A[mid] == key) 
            return true; //return m in Search in Rotated Array I
        if (A[left] < A[mid]) { //left half is sorted
            if (A[left] <= key && key < A[mid])
                right = mid - 1;
            else
                left = mid + 1;
        } else if (A[left] > A[mid]) { //right half is sorted
            if (A[mid] < key && key <= A[right])
                left = mid + 1;
            else
                right = mid - 1;
        } else { // A[left] == A[mid]
            left++;
        }
    }
    return false;
}
```

82. 从已经排序的链表中删除所有重复过的元素，只留下只出现一次的元素
------

考察链表操作

```C
struct ListNode* deleteDuplicates(struct ListNode* head) {
    struct ListNode dummy, *p = &dummy;
    dummy.next = head;
    while (p && p->next && p->next->next) {
        if (p->next->val == p->next->next->val) {
            struct ListNode* distinct = p->next;
            int dup = p->next->val;
            while (distinct && distinct->val == dup) {
                distinct = distinct->next; // TODO: fix mem leak
            }
            p->next = distinct;
        } else {
            p=p->next;
        }
    }
    return dummy.next;
}
```

83. 从已经排序的链表中删除所有重复过的元素，但是重复过的也留下一个，即，使新链表不重复
------

同样是考察链表基本操作

```C
struct ListNode* deleteDuplicates(struct ListNode* head) {
    struct ListNode dummy, *p = &dummy; dummy.next = head; dummy.val = head->val + 1;
    while (p && p->next) {
        if (p->val == p->next->val) {
            int dup = p->val;
            while (p->next && p->next->val == dup)
                p->next = p->next->next; // TODO: fix mem leak
        } else
            p = p->next;
    }
    return dummy.next;
}
```

84. 在柱状图中查找最大的矩形
------

见注释

```C++
int largestRectangleArea(vector<int>& height) {
    stack<int> stk;
    height.push_back(0); // dummy end
    int result  =0;
    // 总结，对于需要查找上一次最大元素的问题，可以考虑使用栈存储
    for (int i = 0; i < height.size(); ) {
        // 当遇到更高的柱子时候，先存入堆栈
        if (stk.empty() || height[i] > height[stk.top()]) // meet higher
            stk.push(i++);
        // 当遇到低一些的柱子时候，计算这些柱子到上一个更矮的柱子之间的最大举行，如果已经清空，说明之前所有柱子都更低
        else { // lower
            int h = stk.top();
            stk.pop();
            result = max(result, height[h] * (stk.empty() ? i : i - stk.top() -1));
        }
    }
    return result;
}
```

85. 最大的长方形
------

```C
int max(int a, int b) {
    return a > b ? a : b;
}

int min(int a, int b) {
    return a < b ? a : b;
}

int maximalRectangle(char** matrix, int row, int col) {
    if (!matrix) return 0;
    int left[col], right[col], height[col];
    for (int i = 0; i < col; i++)
        left[i] = 0, right[i] = col, height[i] = 0;
    int area = 0;
    for (int i = 0; i < row; i++) {
        int cur_left = 0, cur_right = col;
        for (int j = 0; j < col; j++)
            if (matrix[i][j] == '1')  // 在第 j 列的高度
                height[j]++;
            else
                height[j] = 0;
        for (int j = 0; j < col; j++)
            if (matrix[i][j] == '1')
                left[j] = max(left[j], cur_left);
            else
                left[j] = 0, cur_left = j + 1;
        for (int j = col - 1; j >= 0; j--)
            if (matrix[i][j] == '1')
                right[j] = min(right[j], cur_right);
            else
                right[j] = col, cur_right = j;
        for (int j = 0; j < col; j++)
            area = max(area, (right[j] - left[j]) * height[j]);
    }
    
    return area;
}
```

86. 链表分区，要求把小于某个值得元素全都放到前面
------

对于链表这道题很简单，分两个列表在合并就好了，问题是当我们处理类似的数组问题时，也有一种巧妙地O(n)的解法

```C
struct ListNode* partition(struct ListNode* head, int x) {
    struct ListNode small, *psmall = &small; // double dummy head
    struct ListNode big, *pbig = &big;
    psmall->next = pbig->next = NULL;
    
    while (head != NULL) {
        if (head->val < x) {
            psmall->next = head;
            psmall = psmall->next;
        } else {
            pbig->next = head;
            pbig = pbig->next;
        }
        head = head->next;
    }
    psmall->next = big.next;
    pbig->next = NULL;
    return small.next;
}
```

87. 把字符串分区后，交换得到的字符串
------

```C++
bool isScramble(string s1, string s2) {
    if(s1==s2)
        return true;

    // 先判断字符是否一致
    int len = s1.size();
    int count[26] = {0};
    for(int i=0; i<len; i++) {
        count[s1[i]-'a']++;
        count[s2[i]-'a']--;
    }

    for(int i = 0; i < 26; i++)
        if(count[i]!=0)
            return false;

    for(int i = 1; i < len; i++) {
        if( isScramble(s1.substr(0,i), s2.substr(0,i)) && isScramble(s1.substr(i), s2.substr(i)))
            return true;
        if( isScramble(s1.substr(0,i), s2.substr(len-i)) && isScramble(s1.substr(i), s2.substr(0,len-i)))
            return true;
    }
    return false;
}
```

88. 合并已排序数组，要求合并到其中一个空间较大的数组中
------

对于这种要求 in-place的算法，从后往前往往可以解决

```C
void merge(int* nums1, int m, int* nums2, int n) {
    int len = m + n - 1;
    m--, n--;
    while (m >= 0 && n >= 0) {
        if (nums1[m] > nums2[n]) {
            nums1[len--] = nums1[m--];
        } else {
            nums1[len--] = nums2[n--];
        }
    }
    while (n >= 0) {
        nums1[n] = nums2[n];
        n--;
    }
    
}
```

89. 生成格雷码(Gray Code)
------

记住格雷码的生成规则

```C++
vector<int> grayCode(int n) {
    vector<int> v;
    for (int i = 0; i < (1 << n); i++) {
        v.push_back((i >> 1) ^ i);
    }
    return v;
}
```

90. 由给定元素生成子集，可能包含重复元素
------

使用了和手机键盘生成字符串号码类似的迭代算法，注意其中对重复元素的处理，当然也可以用DFS来做

```C++
vector<vector<int>> subsetsWithDup(vector<int>& nums) {
    vector<vector<int>> sets;
    sets.push_back({});
    sort(nums.begin(), nums.end()); // 处理包含重复元素的一半需要预排序
    for (int i = 0; i < nums.size(); ) {
        int count = 0; // dup count
        while (count + i < nums.size() && nums[count+i] == nums[i])
            count++;
        int prev_n = sets.size();
        for (int j = 0; j < prev_n; j++) {
            vector<int> instance = sets[j];
            // put dup element `count` times
            for (int k = 0; k < count; k++) {
                instance.push_back(nums[i]);
                sets.push_back(instance);
            }
        }
        i += count;
        
    }
    return sets;
}
```

91. 给定一个数组只包含1-9，可以用1-26代表字母，求出从其中能都得到多少字符串
------

使用动态规划，但是注意其中0的处理，很玄妙

```C
int numDecodings(char* s) {
    if (!s || strlen(s) == 0 || s[0] == '0') return 0;
    int r1 = 1, r2 = 1; // r1: 前一个字符， r2：前两个字符
    char* p = s++; // 上一个字符

    while (*s) {
        if (*s == '0')
             r1 = 0; // 0 不能单独构成字母
        if (*p == '1' || *p == '2' && *s < '7') { // 形成两种可能
            int t = r1; 
            r1 = r2 + r1; 
            r2 = t;
        } else {
            r2 = r1; // 新加入的数字只能单独构成字母
        }
        
        p = s++;
    }
    return r1;
}
```

92. 在给定区间上翻转数组
------

同样是数组草错细节题

```C
struct ListNode* reverseBetween(struct ListNode* head, int m, int n) {
    if (m == n) return head;
    struct ListNode dummy, *p = &dummy, * small_node, * big_node; // actually the prev ones
    dummy.next = head;
    n -= m;
    
    while (--m) // m starts from 1, so not m--
        p = p->next;
    struct ListNode* start = p->next;
    while (n--) {
        struct ListNode* next = start->next;
        start->next = next->next;
        next->next = p->next;
        p->next = next;
    }
    
    return dummy.next;
}
```

93. 恢复 IP 地址，给定一个字符串，适当插入点，一共有多少种方式构成 IP 地址
------

又是一道 DFS 的题，注意对于字符串问题如何处理

```C++
vector<string> restoreIpAddresses(string s) {
    vector<string> result;
    restore(result, s, "", 0, 0);
    return result;
}

void restore(vector<string>& result, string& s, string restored, int start, int dots) {
    if (dots > 4) return;
    if (dots == 4 && start == s.size())
        result.push_back(restored);
    
    for (int i = 1; i < 4; i++) {
        if (start + i > s.size())
            break;
        string part = s.substr(start, i);
        if (part[0] == '0' && part.size() > 1 || i == 3 && stoi(part) > 255)
            continue;
        restore(result, s, restored + part + (dots==3 ? "" : "."), start + i, dots + 1);
        
    }

}
```

94. 中序遍历二叉树
------

当然是使用栈了

```C++
vector<int> inorderTraversal(TreeNode* root) {
    vector<int> result;
    stack<TreeNode*> stk;
    TreeNode* current = root;
    
    while (!stk.empty() || current) {
        if (current) {
            stk.push(current);
            current = current->left;
        } else {
            current = stk.top();
            stk.pop();
            result.push_back(current->val);
            current = current->right;
        }
    }
    return result;
}
```

95. 生成二叉树，同下体一样
------

```C＋＋
vector<TreeNode*> generateTrees(int n) {
    return gen(1, n);
}

vector<TreeNode*> gen(int start, int end) {
    vector<TreeNode*> result;
    if (start > end) {
        result.push_back(NULL);
        return result;
    }
    
    for (int i = start; i <= end; i++) {
        auto leftTrees = gen(start, i - 1);
        auto rightTrees = gen(i + 1, end);
        for (auto& l : leftTrees) {
            for (auto& r : rightTrees) {
                auto root = new TreeNode(i);
                root->left = l;
                root->right = r;
                result.push_back(root);
            }
        }
    }
    return result;
    
}
```


96. 给定数字n，从1到 n 作为节点有多少种方式生成二叉树
------

这道题看似是树，实际上是一个动态规划问题。

```C
int numTrees(int n) {
    if (n == 0) return 0;
    
    int* dp = malloc(sizeof(int) * (n+1));
    dp[0] = 1;
    
    for (int i = 1; i <= n; i++) {
        int num = 0;
        for (int j = 0; j <= i; j++) // 依次选取第 k 个点作为根
            num += dp[j - 1] * dp[i - j];
        dp[i] = num;
    }
    return dp[n];
}
```

97. 给定两个字符串交叉是否能够构成第三个字符串
------

这道题是一道二维的 DP 问题，因为需要对于每个字符串的每个位置用另一个字符串尝试匹配

```C
bool isInterleave(char* s1, char* s2, char* s3) {
    int l1 = strlen(s1), l2 = strlen(s2), l3 = strlen(s3);
    if (l1 + l2 != l3) return false;
    // 在 i+j 位置 s1[i] s2[j] 是否能够构成s[i+j]
    bool** dp = malloc(sizeof(bool*) * (l1 + 1));
    for (int i = 0; i <= l1; i++)
        dp[i] = malloc(sizeof(bool) * (l2 + 1));
    
    for (int i = 0; i <= l1; i++)
        for (int j = 0; j <= l2; j++)
            if (i == 0 && j == 0)
                dp[i][j] = true;
            else if (i == 0)
                dp[i][j] = (dp[i][j-1] && s2[j-1] == s3[i+j-1]); // 注意：赋值的优先级更高
            else if (j == 0)
                dp[i][j] = (dp[i-1][j] && s1[i-1] == s3[i+j-1]);
            else
                dp[i][j] = (dp[i-1][j] && s1[i-1] == s3[i+j-1] || dp[i][j-1] && s2[j-1] == s3[i+j-1]);
    return dp[l1][l2];
}
```

98. 验证二叉搜索树是否合法
------

先序遍历即可

```C
bool valid(struct TreeNode* root, long left, long right) {
    return root == NULL || root->val > left && root->val < right &&
        valid(root->left, left, root->val) &&
        valid(root->right, root->val, right);
}
 
bool isValidBST(struct TreeNode* root) {
    return valid(root, INT_MIN - 1l, INT_MAX + 1l);
}
```

99. 在二叉搜索树中有两个节点被调换了，找出这两个节点，并恢复该二叉树
------

```C
struct TreeNode* prev = NULL;
struct TreeNode* first = NULL;
struct TreeNode* second = NULL;

void traverse(struct TreeNode* root) {
    if (!root) return;
    traverse(root->left);
    if (prev && prev->val > root->val) {
         if (!first) first = prev;
         second = root;
    }
    prev = root;
    traverse(root->right);
}

void recoverTree(struct TreeNode* root) {
    prev = first = second = NULL;
    traverse(root);
    if (!first) return;
    int temp = first->val;
    first->val = second->val;
    second->val = temp;
}
```

100. 判断是否是相同的树
------

```C
bool isSameTree(struct TreeNode *p, struct TreeNode *q) {
    if (p == NULL || q == NULL) {
        return p == q;
    } else {
        return p->val == q->val
            && isSameTree(p->left, q->left)
            && isSameTree(p->right, q->right);
    }
}
```

101. 判断是不是左右对称的树
------

```C
bool sym(struct TreeNode* left, struct TreeNode* right) {
    if (left && !right || !left && right)
        return false;
    return !left && !right ||
        left->val == right->val && 
        sym(left->left, right->right) &&
        sym(right->left, left->right);
}

bool isSymmetric(struct TreeNode* root) {
    if (!root) return true;
    return sym(root->left, root->right);
}
```

102. 二叉树层序遍历
------

```C++
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> result;
    if (!root) return result;
    vector<TreeNode*> current, next;
    current.push_back(root);
    while (!current.empty()) {
        next.resize(0);
        vector<int> vals;
        for (int i = 0; i < current.size(); i++) {
            if (current[i]->left)
                next.push_back(current[i]->left);
            if (current[i]->right)
                next.push_back(current[i]->right);
            vals.push_back(current[i]->val);
        }
        result.push_back(vals);
        current = next;
    }
    return result;
}
```

103. 二叉树 ZigZag 层序遍历
------

这道题更好的做法是使用一个栈，从而使得每行的顺序都是上一行的翻转

```C++
vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
    vector<vector<int>> result;
    if (!root) return result;
    vector<TreeNode*> current, next;
    current.push_back(root);
    bool odd = true;
    while (!current.empty()) {
        next.resize(0);
        vector<int> vals;
        for (int i = 0; i < current.size(); i++) {
            if (current[i]->left)
                next.push_back(current[i]->left);
            if (current[i]->right)
                next.push_back(current[i]->right);
            vals.push_back(current[i]->val);
        }
        if (!odd) reverse(vals.begin(), vals.end());
        odd = !odd;
        result.push_back(vals);
        current = next;
    }
    return result;
}
```

104. 树的最大深度
------

```C
int maxDepth(struct TreeNode* root) {
    if (!root) return 0;
    int left = maxDepth(root->left), right = maxDepth(root->right);
    return (left > right ?left : right) + 1;
}
```

105. 从前序遍历和中序遍历生成生二叉树
------

```C
struct TreeNode* build(int* prestart, int* preend, int* instart, int* inend) {
    struct TreeNode* root = malloc(sizeof(struct TreeNode));
    root->val = *prestart;
    root->left = root->right = NULL;
    
    if (prestart == preend)
        return root;
    
    int* root_inorder = instart;
    while (root_inorder <= inend && *root_inorder != *prestart)
        root_inorder++;
    int left_len = root_inorder - instart;
    int right_len = inend - root_inorder;
    if (left_len > 0)
        root->left = build(prestart + 1, prestart + left_len, instart, root_inorder - 1);
    if (right_len > 0)
        root->right = build(prestart + left_len + 1, preend, root_inorder + 1, inend);
    return root;
}
// m always equals n, otherwise it's bad input
struct TreeNode* buildTree(int* preorder, int m, int* inorder, int n) {
    if (n==0) return NULL;
    return build(preorder, preorder + n - 1, inorder, inorder + n - 1);
}
```

106. 从中序遍历和后序遍历生成二叉树
------

```C
struct TreeNode* build(int* instart, int* inend, int* poststart, int* postend) {
    struct TreeNode* root = malloc(sizeof(struct TreeNode));
    root->val = *postend;
    root->left = root->right = NULL;
    
    if (poststart == postend)
        return root;
    
    int* root_inorder = instart;
    while (root_inorder <= inend && *root_inorder != *postend)
        root_inorder++;
    int left_len = root_inorder - instart;
    int right_len = inend - root_inorder;
    if (left_len > 0)
        root->left = build(instart, root_inorder - 1, poststart, poststart + left_len - 1);
    if (right_len > 0)
        root->right = build(root_inorder + 1, inend, poststart + left_len, postend - 1);
    return root;
}
struct TreeNode* buildTree(int* inorder, int m, int* postorder, int n) {
    if (n == 0) return NULL;
    return build(inorder, inorder + n - 1, postorder, postorder +n - 1);
}
```

107. 二叉树层序遍历，但要生成翻转的遍历序列
------

```C++
vector<vector<int>> levelOrderBottom(TreeNode* root) {
    vector<vector<int>> result;
    if (!root) return result;
    vector<TreeNode*> current, next;
    current.push_back(root);
    while (!current.empty()) {
        next.resize(0);
        vector<int> vals;
        for (int i = 0; i < current.size(); i++) {
            if (current[i]->left)
                next.push_back(current[i]->left);
            if (current[i]->right)
                next.push_back(current[i]->right);
            vals.push_back(current[i]->val);
        }
        result.push_back(vals);
        current = next;
    }
    reverse(result.begin(), result.end());
    return result;
}
```

108. 把排序数组转化为二叉树
------

```C
 struct TreeNode* bst(int* left, int* right) {
    int* mid = left + (right - left) / 2;
    struct TreeNode* root = malloc(sizeof(struct TreeNode));
    root->val = *mid;
    root->left = root->right = NULL;
    if (left < mid)
        root->left = bst(left, mid-1);
    if (mid < right)
        root->right = bst(mid+1, right);
    return root;
}
struct TreeNode* sortedArrayToBST(int* nums, int n) {
    if (n == 0) return NULL;
    return bst(nums, nums + n -1);
}
```

109. 把排序列表转化为二叉树
------

```C
struct ListNode* list;
int len(struct ListNode* head) {
    int l = 0;
    while (head)
        head = head->next, l++;
    return l;
}

struct TreeNode* bst(int n) {
    if (n == 0) return NULL;
    struct TreeNode* root = malloc(sizeof(struct TreeNode));
    root->left = bst(n/2);
    root->val = list->val;
    list = list->next;
    root->right = bst(n - n / 2 - 1);
    return root;
}
struct TreeNode* sortedListToBST(struct ListNode* head) {
    if (!head) return 0;
    list = head;
    return bst(len(head));
}
```

110. 平衡二叉树
------

```C
int height(struct TreeNode* root) {
    if (!root) return 0;
    int left = height(root->left);
    int right = height(root->right);
    if (left > right + 1 || right > left + 1 || left == -1 || right == -1)
        return -1;
    return (left > right ? left : right) + 1;
}
bool isBalanced(struct TreeNode* root) {
    return height(root) != -1;
}
```

111. 二叉树最小高度
------

```C
int minDepth(struct TreeNode* root) {
    if (!root) return 0;
    int left = minDepth(root->left);
    int right = minDepth(root->right);
    if (!right) return left + 1;
    if (!left) return right + 1; // tricky here,当有空节点时，不能返回0，而是返回另一个值
    
    return (left < right ? left : right) + 1;
}
```

112. 二叉树中是否存在和为某个数的路径
------

```C
bool hasPathSum(struct TreeNode* root, int sum) {
    if (!root) return false;
    if (!root->left && !root->right) return sum == root->val;
    return hasPathSum(root->left, sum - root->val) ||
        hasPathSum(root->right, sum - root->val);
}
```

113. 接上题，把这个路径找出来
------

```C++
vector<vector<int>> pathSum(TreeNode* root, int sum) {
    vector<vector<int>> result;
    vector<int> path;
    getPaths(result, path, root, sum);
    return result;
}

void getPaths(vector<vector<int>>& result, vector<int> path, TreeNode* root, int sum) {
    if (!root)
        return;
    path.push_back(root->val);
    if (!root->left && !root->right && sum == root->val) {
        result.push_back(path);
        return;
    }
    
    getPaths(result, path, root->left, sum - root->val);
    getPaths(result, path, root->right, sum - root->val);
}
```

114. 把二叉树扁平成列表
------

```C++
TreeNode* prev;
void flatten(TreeNode* root) {
    if (!root) return;
    flatten(root->right);
    flatten(root->left);
    root->right = prev;
    root->left = NULL;
    prev = root; // last flattened element
}
```

115. 通过删掉一些字母得到子序列，问有多少种方法能够得到子序列呢
------

使用DP，

```C++
/**
 * Solution (DP):
 * 我们扫描字符串s
 * Path[i][j] 代表T.substr(1...i) 在 S(1...j)不同的子序列的数量
 * 
 * Path[i][j] = Path[i][j-1]            (discard S[j])
 *              +     Path[i-1][j-1]    (S[j] == T[i] and we are going to use S[j])
 *                 or 0                 (S[j] != T[i] so we could not use S[j])
 * while Path[0][j] = 1 and Path[i][0] = 0.
 */


class Solution {
public:
    int numDistinct(string s, string t) {
        int m = t.size();
        int n = s.size();
        
        if (m > n)
            return 0;
        vector<vector<int>> path(m+1, vector<int>(n+1, 0));
        
        for (int i = 0; i <= n; i++)
            path[0][i] = 1;
        
        for (int j = 1; j <= n; j++) // S
            for (int i = 1; i <= m; i++) // T
                path[i][j] = path[i][j-1] + (t[i-1] == s[j-1] ? path[i-1][j-1] : 0);
        
        return path[m][n];
    }
};
```


116. 完全二叉树中把每个节点指向他这一层的右面的节点
------

显然左节点的下一个节点是父节点的右节点，右节点的下一个节点是父节点下一个节点的左节点。

```C
void connect(struct TreeLinkNode *root) {
    if (!root)
        return;
    if (root->left)
        root->left->next = root->right;
    if (root->right)
        root->right->next = root->next ? root->next->left : NULL;
    connect(root->left);
    connect(root->right);
}
```

117. 同上题，但是是任意的🌲
------

通过上一层已经被连接的 next 指针，顺序层序访问，从而连接下一层。

```
void connect(struct TreeLinkNode *root) {
    struct TreeLinkNode* head = root, * prev = NULL, *p = NULL;
    while (head) { // head 是每层的开始
        p = head;
        prev = head = NULL;
        
        while (p) {
            if (p->left) {
                if (prev)
                    prev->next = p->left;
                else 
                    head = p->left;
                prev = p->left;
            } 
            
            if (p->right) {
                if (prev)
                    prev->next = p->right;
                else
                    head = p->right;
                prev = p->right;
            }
            p = p->next;
        }
    }
}
```

118. 杨辉三角
------

注意坐标关系，不要被骗了

```C++
vector<vector<int>> generate(int n) {
    vector<vector<int>> result(n);
    
    
    for (int i = 0; i < n; i++) {
        result[i].resize(i+1);
        result[i][0] = result[i][i] = 1;
        for (int j = 1; j < i; j++)
            result[i][j] = result[i-1][j-1] + result[i-1][j];
    }
    return result;
}
```

119. 返回杨辉三角的第 k 行
------

要求只能使用O(k)的额外空间，比较蛋疼的是这里的 k 是从0计数的。

```C++
vector<int> getRow(int rowIndex) {
    rowIndex++;
    vector<int> row;
    for (int i = 0; i < rowIndex; i++) {
        vector<int> newRow(i+1);
        newRow[0] = newRow[i] = 1;
        for (int j = 1; j < i; j++)
            newRow[j] = row[j-1] + row[j];
        swap(row, newRow);
    }
    return row;
}
```

120. 给定一个类似杨辉三角形状的数组，求从顶部到底部的最短路径
------

显然是使用 DP，但是有一个问题，如果是 top down 的话，最后还需要遍历一下，而如果是 bottom up 就只需要返回dp[0]就好了。

```C++
int minimumTotal(vector<vector<int>>& triangle) {
    vector<int> dp(triangle.back()); // 复制最后一行
    for (int layer = triangle.size() - 2; layer >= 0; layer--)
        for (int i = 0; i <= layer; i++)
            dp[i] = triangle[layer][i] + min(dp[i], dp[i+1]);
    return dp[0];
}
```

121. 买卖股票最佳时机，只能交易一次
------

```C
int maxProfit(int* prices, int pricesSize) {
    if (pricesSize < 2) return 0;
    int profit = 0;
    int min = prices[0];
    // 从前到后依次遍历，如果有更好的收益更新，或者更新 min，限制条件是先出现最小值
    for (int i = 0; i < pricesSize; i++) {
        if (prices[i] > min) {
                profit = max(profit, prices[i] - min);
        } else {
            min = prices[i];
        }
    }
    return profit;
}
```

122. 买卖股票的最佳时机，可以做任意多比交易
------

有两种解法，一种是不断做交易，完全不考虑交易次数，这种做法不符合实际情况。
另一种做法是模拟交易，这样会生成最少的交易次数，结果也是对的。

```C
// 1
int maxProfit(int* prices, int pricesSize)
    int total = 0;
    for (int i=0; i< pricesSize-1; i++)
        if (prices[i+1]>prices[i]) 
            total += prices[i+1]-prices[i];

    return total;
}
```

```C
// 2
int maxProfit(int* prices, int pricesSize) {
    if (!prices) return 0;
    int profit = 0;bool buy = true;
    int min = prices[0], max = prices[0];
    for (int i = 0; i < pricesSize; i++) {
        if (prices[i] < min && buy) {
            min = prices[i];
            max = prices[i];
        }
        if (prices[i] > min && buy)
            buy = false;
        if (prices[i] > max && !buy)
            max = prices[i];
        if ((prices[i] < max || i == pricesSize - 1) && !buy){
            profit += max - min;
            min = prices[i];
            max = prices[i];
            buy = true;
        }
            
    }
    return profit;
    
}
```

123. 股票交易，限制只能交易两股
------

每次求解的是：卖出两股以后的最大值，刚刚买入第二股的最大值，卖出第一股时候的最大值，买入第一股时候的最大值。

```C++
int maxProfit(vector<int>& prices) {
    int hold1 = INT_MIN, hold2 = INT_MIN;
    int release1 = 0, release2 = 0;
    
    for (auto i : prices) {
        release2 = max(release2, hold2 + i);
        hold2 = max(hold2, release1 - i);
        release1 = max(release1, hold1 + i);
        hold1 = max(hold1, -i);
    }
    
    return release2;
}
```

124. 二叉树路径最大和，路径可以从任意一个节点开始到任意一个节点结束
------

```C
int max(int a, int b) {
    return a > b ? a : b;
}

int doSum(struct TreeNode* root, int* sum) {
    if (!root)
        return 0;
    int left = max(0, doSum(root->left, sum));
    int right = max(0, doSum(root->right, sum));
    *sum = max(*sum, left+right+root->val);
    return max(left, right) + root->val;
} 


int maxPathSum(struct TreeNode* root) {
    int sum = INT_MIN;
    doSum(root, &sum);
    return sum;
}
```


125. 给定一个字符串，只考虑字母和数字，忽略大小写，判断是否是回文字符串
------

太简单了，没啥可说的

```C
bool isPalindrome(char* s) {
    int len = strlen(s);
    if (len == 0) return true;
    int left = 0, right = len - 1;
    while (left < right) {
        char l = s[left], r = s[right];
        if (isalnum(l) && isalnum(r)) {
            if (tolower(l) != tolower(r))
                return false;
            left++, right--;
        } else {
            if (!isalnum(l))
                left++;
            if (!isalnum(r))
                right--;
        } 
    }
    return true;
}
```

127. 单词梯子
------

给定梯子，和开始单词和结束单词，最少需要多少个中间单词，才能变化过去，每次只能变化一个字母

```C++
int ladderLength(string beginWord, string endWord, unordered_set<string>& wordList) {
    unordered_set<string> beginSet, endSet, *set1, * set2;
    beginSet.insert(beginWord);
    endSet.insert(endWord);
    
    int dist = 2;
    while (!beginSet.empty() && !endSet.empty()) {
        if (beginSet.size() < endSet.size()) {
            set1 = &beginSet;
            set2 = &endSet;
        } else {
            set1 = &endSet;
            set2 = &beginSet;
        }
        
        unordered_set<string> temp;
        for (auto word : *set1) { // notice word in not ref
            wordList.erase(word);
            for (auto& letter : word) {
                for (int i = 0; i < 26; i++) {
                    char oldLetter = letter;
                    letter = 'a' + i;
                    if (set2->find(word) != set2->end())
                        return dist;
                    if (wordList.find(word) != wordList.end()) {
                        temp.insert(word);
                        wordList.erase(word);
                    }
                    letter = oldLetter;
                }
            }
        }
        dist++;
        swap(*set1, temp);
        
    }
    return 0;
}
```

128. 最长递增子序列
------

使用动态规划

```C++
int longestConsecutive(vector<int>& nums) {
    int result = 0;
    unordered_map<int, int> hash; // 每个元素和它们所在序列的长度
    
    for (auto n : nums) {
        if (hash.find(n) == hash.end()) {
            // 查找两边的元素，如果找到，把新元素合并进去
            int left = hash.find(n-1) != hash.end() ? hash[n-1] : 0;
            int right = hash.find(n+1) != hash.end() ? hash[n+1] : 0;
            int sum = left + right + 1;
            hash[n] = hash[n-left] = hash[n+right] = sum; // 注意此处的更新，并不需要更新区间内的每个值，只需要更新边界即可
            result = max(result, sum);
        }
    }
    
    return result;
}
```

129. 二叉树中只有0-9找出所有根节点到子节点的和
------

```C
int sum(struct TreeNode* root, int x) {
    if (!root->left && !root->right)
        return x * 10 + root->val;
    int val = 0;
    if (root->left)
        val += sum(root->left, x * 10 + root->val);
    if (root->right)
        val += sum(root->right, x * 10 + root->val);
    return val;
}

int sumNumbers(struct TreeNode* root) {
    if (!root) return 0;
    return sum(root, 0);
}
```

130. 把所有被包围的O置为X
------

使用并查集

```C++
class UnionFind {
private:
    vector<int> m_father, m_rank;
public:
    UnionFind(int n): m_father(n), m_rank(n, 0) {
        for (int i = 0; i < m_father.size(); i++)
            m_father[i] = i;
    }
    
    int find(int x) {
        if (x != m_father[x])
            m_father[x] = find(m_father[x]);
        return m_father[x];
    }
    
    void unionify(int x, int y) {
        x = find(x);
        y = find(y);
        
        if (x == y) return;
        
        if (m_rank[x] > m_rank[y]) {
            m_father[y] = x;
        } else {
            if (m_rank[x] == m_rank[y])
                m_rank[y]++;
            m_father[x] = y;
        }
    }
};

class Solution {
public:
    void solve(vector<vector<char>>& board) {
        int n = board.size();
        if (n == 0) return;
        int m = board[0].size();
        UnionFind uf(n*m+1);
        
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if ((i == 0 || j == 0 || i == n-1 || j == m-1) && board[i][j] == 'O')
                    uf.unionify(i * m + j, n * m);
                else if (board[i][j] == 'O') {
                    if (board[i-1][j] == 'O')
                        uf.unionify(i * m + j, (i - 1) * m + j);
                    if (board[i+1][j] == 'O')
                        uf.unionify(i*m+j, (i+1)*m+j);
                    if (board[i][j-1] == 'O')
                        uf.unionify(i*m+j, i*m+j-1);
                    if (board[i][j+1] == 'O')
                        uf.unionify(i*m+j, i*m+j+1);
                }
            }
        }
        
        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++)
                if (uf.find(i*m+j) != uf.find(n*m))
                    board[i][j] = 'X';
    }
};
```

131. 对字符串分组，是的每个字串都是回文，返回所有可能的分组
------

```C++
vector<vector<string>> partition(string s) {
    vector<vector<string>> result;
    vector<string> group;
    dfs(result, s, group, 0);
    return result;
}

void dfs(vector<vector<string>>& result, const string& s, vector<string>& group, int start) {
    if (start == s.size()) {
        result.push_back(group);
        return;
    }
    
    for (int i = start; i < s.size(); i++) {
        if (isPalindrome(s, start, i)) {
            group.push_back(s.substr(start, i - start + 1));
            dfs(result, s, group, i + 1);
            group.pop_back();
        }
    }
}

bool isPalindrome(const string& s, int left, int right) {
    while (left < right) {
        if (s[left++] != s[right--])
            return false;
    }
    return true;
}
```

132. 如上题，找出最少需要分组几次
------

```C++
int minCut(string s) {
    vector<int> cut(s.size() + 1, 0);
    for (int i = 0; i < s.size() + 1; i++)
        cut[i] = i - 1;

    for (int i = 0; i < s.size(); i++) {
        for (int j = 0; i - j >= 0 && i + j < s.size() && s[i+j] == s[i-j]; j++)
            cut[i+j+1] = min(cut[i+j+1], cut[i-j] + 1); // i-j -> i+j是 palindrome，所以只需要cut[i-j] 在加上这一段就好了 
        for (int j = 1; i - j + 1 >= 0 && i + j < s.size() && s[i+j] == s[i-j+1]; j++)
            cut[i+j+1] = min(cut[i+j+1], cut[i - j + 1] + 1);
    }
    
    return cut[s.size()];
}
```

133. 复制有向图
------

```C++
unordered_map<UndirectedGraphNode*, UndirectedGraphNode*> hash; // old -> new pair
UndirectedGraphNode *cloneGraph(UndirectedGraphNode *node) {
    if (!node)
        return NULL;
    if (hash.find(node) == hash.end()) {
        hash[node] = new UndirectedGraphNode(node->label);
        for (auto n : node->neighbors)
            hash[node]->neighbors.push_back(cloneGraph(n));
    }
    
    return hash[node];
}
```

134. 加油站
------

```C
int canCompleteCircuit(int* gas, int gasSize, int* cost, int costSize) {
    int total = 0;
    int j = -1;
    
    for (int i = 0, sum = 0; i < gasSize; ++i) {
        sum += gas[i] - cost[i]; // 从此处经过能够净增多少汽油
        total += gas[i] - cost[i]; // 记录总的汽油量是否是正的
        if (sum < 0) { // 如果当前汽油量已经小于 0，说明之前的节点都是不行的，到下一个节点
            j = i;
            sum = 0; // 同时重新开始计数
        }
    }
    
    return total >= 0 ? j + 1 : -1;
}
```

135. 糖块，成绩高的需要比他身边成绩低的获得更多的糖
------

```C++
int candy(vector<int>& ratings) {
    int n = ratings.size();
    if (n <= 1)
        return n;
    vector<int> candies(n, 1);
    
    for (int i =1; i < n; i++)
        if (ratings[i] > ratings[i-1])
            candies[i] = candies[i-1] + 1;
    
    for (int i = n - 1; i > 0; i--)
        if (ratings[i-1] > ratings[i])
            candies[i-1] = max(candies[i] + 1, candies[i-1]);
    
    int result = 0;
    for (auto i : candies)
        result += i;
    
    return result;
}
```

136. 找出数组中只出现一次的数字
------

```C
int singleNumber(int* nums, int numsSize) {
    int result = nums[0];
    for (int i = 1; i < numsSize; i++)
        result ^= nums[i];
    return result;
}
```

137. 一个数组中，所有数字都出现三次，除了一个数字以外，找出这个数字
------

```C
// 使用二进制计算
// 00->10->01->00(0->1->2->3/0)
// ones = ones ^ A[i]; if (twos == 1) then ones = 0
// twos = twos ^ A[i]; if (ones* == 1) then twos = 0

int singleNumber(int* nums, int numsSize) {
    int ones = 0, twos = 0;
    for (int i = 0; i < numsSize; i++) {
        ones = (ones ^ nums[i]) & ~twos;
        twos = (twos ^ nums[i]) & ~ones;
    }
    return ones;
}
```

138. 复制复杂结构链表
------

```C
/**
 * Definition for singly-linked list with a random pointer.
 * struct RandomListNode {
 *     int label;
 *     struct RandomListNode *next;
 *     struct RandomListNode *random;
 * };
 */
struct RandomListNode *copyRandomList(struct RandomListNode *head) {
    struct RandomListNode* p;
    p = head;
    while (p) {
        struct RandomListNode* node = malloc(sizeof(struct RandomListNode));
        node->next = node->random = NULL; // spicial notice to struct initialization in c
        node->label = p->label;
        node->next = p->next;
        p->next = node;
        p = node->next;
    }
    
    p = head;
    while (p) {
        if (p->random)
            p->next->random = p->random->next;
        p = p->next->next;
    }
    
    struct RandomListNode dummy, *q = &dummy;
    dummy.next = dummy.random = NULL;
    p = head;
    while (p) {
        q->next = p->next;
        q = q->next;
        p->next = p->next->next;
        p = p->next;
    }
    return dummy.next;
}
```

139. 查找单词是否能组成句子
------

```C++
bool wordBreak(string s, unordered_set<string>& wordDict) {
    if (wordDict.empty()) return false;
    vector<bool> dp(s.size() + 1, false);
    dp[0] = true;
    // 动态规划，假设前 i 个字符已经匹配到了，尝试匹配 i 到 i+j，如果找到了，就匹配到了 i+j
    for (int i = 1; i <= s.size(); i++) {
        for (int j = i-1; j >= 0; j--) {
            if (dp[j]) {
                string word = s.substr(j, i-j);
                if (wordDict.find(word) != wordDict.end()) {
                    dp[i] = true;
                    break;
                }
            }
        }
    }
    return dp[s.size()];
}
```

141. 列表是否有环
------

slow每次走一步，而fast每次走两步，因此在进入环之后，两者一定会相遇

```C
bool hasCycle(struct ListNode *head) {
    struct ListNode* slow = head, * fast = head;
    while (fast && fast->next && fast->next->next) {
        fast = fast->next->next;
        slow = slow->next;
        if (slow == fast)
            return true;
    }
    return false;
}
```

142. 列表是否有环？如果有找到环的开始
------

从两者出发，到两者相遇，slow指针走了p步，而fast指针走了2p步，显然fast多走了一圈（或者多圈）。
设 p = k + x, 2p = k + x + loop -> 2k + 2x = k + x + loop -> k + x = loop -> k = loop - x，剩下的长度正好也是k。
假设入口处距离起点的距离是k，那么发生碰撞的点距离环的入口处距离也是k，所以两个指针分别从开始和碰撞点出发匀速一定会在环的入口相遇。

```C
struct ListNode *detectCycle(struct ListNode *head) {
    struct ListNode* slow = head, * fast = head, *entry = NULL;
    bool found = false;
    while (!found && fast && fast->next && fast->next->next) {
        fast = fast->next->next;
        slow = slow->next;
        if (slow == fast)
            found = true;
    }

    if (!found) return NULL;

    slow = head;
    while (slow != fast) {
        slow = slow->next;
        fast = fast->next;
    }

    return slow;
}
```

144. 前序遍历
------

```C++
vector<int> preorderTraversal(TreeNode* root) {
    vector<int> result;
    if (!root) return result;
    
    stack<TreeNode*> stk;
    stk.push(root);
    
    while (!stk.empty()) {
        TreeNode* node  = stk.top();
        stk.pop();
        result.push_back(node->val);
        if (node->right)
            stk.push(node->right);
        if (node->left)
            stk.push(node->left);
        
    }
    
    return result;
        
}
```

145. 后序遍历
------

```C++
vector<int> postorderTraversal(TreeNode* root) {
    vector<int> result;
    if (!root) return result;
    stack<TreeNode*> stk, output;
    stk.push(root);
    while (!stk.empty()) {
        auto node = stk.top();
        stk.pop();
        output.push(node);
        
        if (node->left)
            stk.push(node->left);
        if (node->right)
            stk.push(node->right);
    }
    while (!output.empty()) {
        result.push_back(output.top()->val);
        output.pop();
    }
    return result;
}
```

146. LRU缓存
------

```C++
class LRUCache{
public:
typedef unordered_map<int, pair<int, list<int>::iterator>> cache_t; // k: v, iter

    LRUCache(int capacity) : m_capacity(capacity) {
        
    }
    
    int get(int key) {
        auto it = m_cache.find(key);
        if (it == m_cache.end())
             return -1;
        touch(it);
        return it->second.first;
    }
    
    void set(int key, int value) {
        auto it = m_cache.find(key);
        if (it != m_cache.end()) {
            touch(it);
        } else {
            if (m_cache.size() == m_capacity) {
                m_cache.erase(m_used.back());
                m_used.pop_back();
            }
            m_used.push_front(key);
        }
        m_cache[key] = {value, m_used.begin()};
    }
private:
    void touch(cache_t::iterator it) {
        int key = it->first;
        m_used.erase(it->second.second);
        m_used.push_front(key);
        it->second.second = m_used.begin();
    }

    cache_t m_cache;
    list<int> m_used;
    int m_capacity;
};
```

147. 链表插入排序
------

```C
struct ListNode* insertionSortList(struct ListNode* head) {
    if (!head) return NULL;
    struct ListNode dummy, *p = head;
    dummy.val = INT_MIN;
    dummy.next = NULL;
    while (p) {
        struct ListNode* iter = &dummy;
        while (iter->next && iter->next->val < p->val)
            iter = iter->next;
        struct ListNode* pnext = p->next;
        p->next = iter->next;
        iter->next = p;
        p = pnext;
    }
    return dummy.next;
}
```

148. 排序链表，要求达到O(nlogn)时间复杂度
------

```C
void split(struct ListNode* source, struct ListNode** frontptr, struct ListNode** backptr) {
    struct ListNode* fast, * slow;
    if (!source || !source->next)
        *backptr = source;
    else {
        slow = source;
        fast = source->next;
        while (fast && fast->next) {
            fast = fast->next->next;
            slow = slow->next;
            
        }
        
        *backptr = slow->next;
        slow->next = NULL;
    }
    *frontptr = source;
}

struct ListNode* merge(struct ListNode* l1, struct ListNode* l2) {
    if (l1 == NULL) return l2;
    if (l2 == NULL) return l1;
    struct ListNode dummy;
    dummy.next == NULL;
    struct ListNode* p = &dummy;
    while (l1 && l2) {
        if (l1->val < l2->val) {
            p->next = l1;
            l1 = l1->next;
        } else {
            p->next = l2;
            l2 = l2->next;
        }

        p = p->next;
    }

    if (l1)
        p->next = l1;

    if (l2)
        p->next = l2;

    return dummy.next;
}

// merge sort
struct ListNode* sortList(struct ListNode* head) {
    struct ListNode* front, * back;
    if (!head || !head->next) return head;
    split(head, &front, &back);
    front = sortList(front);
    back = sortList(back);
    head = merge(front, back);
    return head;
}
```

149. 在同一条线上的点最多的线
------

```C++
int maxPoints(vector<Point>& points) {
    if (points.size() < 2) return points.size();
    int result = 0;

    // 对于每一个点
    for (int i = 0; i < points.size(); i++) {
        // 经过该点的直线，使用分数作为斜率，避免使用浮点数
        map<pair<int, int>, int> lines;
        int localMax = 0, overlap = 0;
        for (int j = i + 1; j < points.size(); j++) { // 避免重复计算
            if (points[j].x == points[i].x && points[j].y == points[i].y) {
                overlap++; // 同一个点
                continue;
            } else {
                int x = points[j].x - points[i].x;
                int y = points[j].y - points[i].y;
                int g = gcd(x, y);
                x /= g, y /= g; // verticle case: x == 0 -> (0, 1)
                lines[make_pair(x, y)]++;
                localMax = max(localMax, lines[make_pair(x, y)]);
            }
        }
        // overlap算在任意条线上
        result = max(result, localMax + overlap + 1);
    }
    return result;
}

int gcd(int x, int y) {
    if (y == 0) return x;
    else return gcd(y, x % y);
}
```

150. 后缀表达式求值
------

```C++
bool is_operator(char t) {
    return t == '+' || t == '-' || t == '*' || t == '/';
}
int calc(int left, char op, int right) {
    switch(op) {
        case '+': return left + right;
        case '-': return left - right;
        case '*': return left * right;
        case '/': return left / right;
    }
}
int evalRPN(vector<string>& tokens) {
    stack<int> nums;
    for (auto& token : tokens) {
        if (is_operator(token[0]) && token.size() == 1) {
            char op = token[0];
            int right_num = nums.top();
            nums.pop();
            int left_num = nums.top();
            nums.pop();
            nums.push(calc(left_num, op, right_num));
        } else {
            nums.push(stoi(token));
        }
    }
    return nums.top();
}
```

151. 反转句子中的单词顺序
------

一般面试的时候会假定没有多余字符的，解法是

```C
```

LeetCode需要处理多余空格：

```C
void swap(char *a, char *b) {
    char tmp = *a; *a = *b; *b = tmp;
}

void reverse(char* start, char* end) {
    while(start < end)  
        swap(start++, end--);
}

void trim(char* s) {

    char* fast, *slow;
    for (fast = s; *fast !='\0'; fast++) {
        if (isspace(*fast)) {
            while(isspace(*(fast + 1)) && *(fast + 1) != 0) 
                fast++;
            if(*(fast+1) == 0) 
                break;
            if(slow == s) 
                continue;
        }
        swap(fast, slow++);
    }
    *slow = 0;
}

void reverseWords(char *s) {
    int len = strlen(s);
    if (len == 0)
        return;
    trim(s);
    len = strlen(s);
    if (len == 0)
        return;
    reverse(s, s + len - 1);
    char* head = s, * tail =s ;
    while (*(tail + 1) != '\0') {
        tail = head;
        while (!isspace(*(tail + 1)) && *(tail + 1) != '\0')
            tail++;
        reverse(head, tail);
    }
}
```

152. 最大子序列成绩
------

```C
int maxProduct(vector<int>& A) {
    int n = A.size();
    int r = A[0];
    for (int i = 1, imax = r, imin = r; i < n; i++) {
        if (A[i] < 0)
            swap(imax, imin);

        imax = max(A[i], imax * A[i]);
        imin = min(A[i], imin * A[i]);

        r = max(r, imax);
    }
    return r;
}
```

*153. 在旋转数组中查找最小值
------

```C
int findMin(int* A, int n) {
    int left = 0; int right = n - 1;
    while (left < right - 1) {
        int mid = left + (right - left) / 2;
        if (A[left] > A[mid])
            right = mid;
        else if (A[right] < A[mid])
            left = mid;
        else
            right = mid;
    }
    return A[left] < A[right] ? A[left] : A[right];
}
```

*154. 在旋转数组中查找最小值，可能有重复
------

```C
int findMin(int* A, int n) {
    int left = 0, right = n - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (A[mid] > A[right]) { // 当需要找的是 left，也就是较小的数字，使用 right 比较不需要等于号
            left = mid + 1;
        } else if (A[right] < A[mid]) {
            right = mid;
        } else {
            right--;
        }
    }
    return A[left];
}
```

155. 设计一个栈，在普通栈的基础上支持 getmin 操作
------

解法1: 使用额外的栈，每个值都记录一个当前最小值，浪费空间

解法2: 也是使用额外的栈，但是惰性记录，只有当需要更新的时候才去记录

```C++
class MinStack {
private:
    stack<int> m_stk;
    stack<int> m_min;
public:
    void push(int x) {
        if (x <= getMin())
            m_min.push(x);
        m_stk.push(x);
    }

    void pop() {
        if (m_stk.top() == getMin())
            m_min.pop();
        m_stk.pop();
    }

    int top() {
        return m_stk.top();
    }

    int getMin() {
        return m_min.empty() ? INT_MAX : m_min.top();
    }
};
```

156-159 Locked
------

160. 求两个链表的交叉点
------

分析题目可知，如果有一个交叉点，那么在这之后的所有点都是交叉的。这里有一个非常巧妙
的做法。使用两个指针，如果到达结尾就指向另一个链表，会产生一下三种情况：

1. 如果交叉点前面的节点数目相同，显然会返回正确节点。
2. 如果不同假设 A 的节点为 a + c，B 的节点为 b + c，则在下一次遍历时：
    a + c + b == b + c + a，恰好到达相同部分的第一个顶点 C1
3. 如果两个列表不相交，那么经过 a + b, b + a距离后，恰好都等于 NULL

```C
struct ListNode *getIntersectionNode(struct ListNode *headA, struct ListNode *headB) {
    if (!headA || !headB) return NULL;
    struct ListNode *p1 = headA, *p2=headB;
    while (p1 != p2) {
        // 两个列表手尾相接，如果有一个点相同，一定会返回
        // a + c + b == b + c + a   --> C1
        // a + b == b + a    --> NULL
        p1 = p1 ? p1->next : headB;
        p2 = p2 ? p2->next : headA;
    }
    
    return p1;
}
```

161. Locked
------

162.  找到极大值，给定一个数组，可能有多个极大值，找到任意一个即可，给定数组中A[i] != A[i+1]
------

题目要求在对数时间内做出来，二分搜索，如果中间的数在左半部分，就向右找。

```C
int findPeakElement(int* nums, int numsSize) {
    int left = 0, right = numsSize - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < nums[mid + 1]) // mid in left part of summit
            left = mid + 1;
        else                           // mid in right part of summit
            right= mid;
    }
    return left;
}
```

163. Locked
------

164. 未排序数组中相差最大的两个数之间的差
------

根据抽屉原理，最大差不可能小于(max - min) / (n - 1)。证明：如果小于，那么整个数组的大小就会小于max - min。
因此我们把


165. 比较版本号大小
------

```C++
vector<int> ver(const string& version) {
    vector<int> result;
    int num = 0;
    for (auto c : version) {
        if (c != '.') {
            num = num * 10 + c - '0';
        } else {
            result.push_back(num);
            num = 0;
        }
    }
    // 对于所有这种分割符中读取数字的都需要注意最后一个
    result.push_back(num); // notice here
    return result;
}

int compareVersion(string version1, string version2) {
    auto v1 = ver(version1);
    auto v2 = ver(version2);
    
    for (int i = 0; i < v1.size() || i < v2.size(); i++) {
        int a = i < v1.size() ? v1[i] : 0;
        int b = i < v2.size() ? v2[i] : 0;
        if (a != b)
            return a > b ? 1 : -1;
    }
    
    return 0;
}
```

＊166. 分数生成小数
------

```C++
string fractionToDecimal(long numerator, long denominator) {
    if (numerator == 0) return "0";
    string result;
    
    // 符号
    if (numerator < 0 ^ denominator < 0)
        result += "-";
    long n = abs(numerator), d = abs(denominator);
    
    // 整数部分
    result += to_string(n / d);
    if (n % d == 0) return result;
    
    // 小数部分
    result+= ".";
    unordered_map<int, int> map;
    for (long r = n % d; r != 0; r %= d) { // 模拟手工除法
        if (map.count(r) > 0) {
            result.insert(map[r], 1, '(');
            result += ")";
            break;
        }
        
        map[r] = result.size(); // 记录对应的位置，以便插入括号
        r *= 10; // 从上借位
        result += to_string(r / d);
    }
    return result;
}
```

167. Locked
------

＊168. 生成 Excel 表格标题
------

注意 A 对应的是1而不是0，而且数字也是从1开始的

```C++
string convertToTitle(int n) {
    string title;
    while (n) {
        char c = (n-1) % 26 + 'A';
        n = (n-1) / 26;
        title = c + title;
    }
    return title;
}
```

169. 给定一个数组，有一个数字的出现频率超过了一半，找出这个数字
------

非常经典的一道题，首先我们假设拿到的数字就是目标，并记录他出现的次数，如果下一个
数字和他不一样，那么我们减一，当次数为0的时候，我们知道这个数字在已经便利过的数字
中出现小于一半了，这时候我们换下一个数字，最后剩下的一定是超过一半的数字。

```C++
int majorityElement(vector<int>& nums) {
    int candidate, count = 0;
    for (auto i : nums) {
        if (count == 0 || candidate == i) {
            count++;
            candidate = i;
        } else {
            count--;
        }
    }
    return candidate;
}
}
```

170. Locked
------

171. Excel 标题转换为数字
------

同样，我们需要注意 A 对应的是1，而不是0

```C
int titleToNumber(char* s) {
    int result = 0;
    while (*s)
        result = result * 26 + *s++ - 'A' + 1;
    return result;
}
```

172. 阶乘中能有几个0
------

显然先算出阶乘数字是会溢出的，而有0的话，就是需要10，也就是就需要2和5，
显然2是比5多的。那么我们只要考虑5的个数就行了， 这时候需要注意，5/15等是算一个5，
而25/75包含了两个5，所以我们计算的时候，数一遍包含5的（这时25等也被计算了），
然后再数一遍包含25的就像当于数了两次了。

```C
int trailingZeroes(int n) {
    if (n < 0)
        return -1;
    int fives = 0;
    for (int i = 5; n / i > 0; i *= 5)
        fives += n / i;
    return fives;
}
```

173. 二叉树中序遍历迭代器
------

```C
class BSTIterator {
public:
    BSTIterator(TreeNode *root) {
        pushAll(root);
    }

    /** @return whether we have a next smallest number */
    bool hasNext() {
        return !m_stack.empty();
        
    }

    /** @return the next smallest number */
    int next() {
        TreeNode* temp = m_stack.top();
        m_stack.pop();
        pushAll(temp->right);
        return temp->val;
    }
    
private:
    stack<TreeNode*> m_stack;
    void pushAll(TreeNode* root) {
        while (root) {
            m_stack.push(root);
            root = root->left;
        }
    }
};

/**
 * Your BSTIterator will be called like this:
 * BSTIterator i = BSTIterator(root);
 * while (i.hasNext()) cout << i.next();
 */
```

174. 地下城游戏
------

王子在格子的左上角，需要到右下角去救公主，在过程中王子不能死掉，和机器人走路一样，使用动态规划

```C++
int calculateMinimumHP(vector<vector<int>>& dungeon) {
    int row = dungeon.size();
    int col = dungeon[0].size();
    vector<vector<int>> bloods(row + 1, vector<int> (col + 1, INT_MAX));
    bloods[row][col-1] = bloods[row-1][col] = 1; // 公主的两边
    // 从公主那里逆向推
    for (int i = row-1; i >= 0; i--) {
        for (int j = col-1; j >= 0; j--) {
             int need = min(bloods[i+1][j], bloods[i][j+1]) - dungeon[i][j]; // 缺乏的血量 = 到达下一步最少的血量 - 这一步消耗的血量
             bloods[i][j] = need > 0 ? need : 1; // 王子的血量至少为1
        }
    }
    return bloods[0][0];
}
```

175-178. Missing
------

179. 最大的数字
------

神奇的排序方法

```C++
string largestNumber(vector<int>& nums) {
    vector<string> num_strings(nums.size());
    for (int i = 0; i < nums.size(); i++)
        num_strings[i] = to_string(nums[i]);
    auto comparator = [] (string& s1, string& s2) {
        return s1 + s2 > s2 + s1;
    };
    sort(num_strings.begin(), num_strings.end(), comparator);
    string result;
    for (auto& num_string: num_strings)
        result += num_string;
    int start = result.find_first_not_of("0");
    if (start == string::npos) return "0";
    return result.substr(start);
}
```

180-185 Missing
------

186 Locked
------

187. 找到所有10个字母唱的重复DNA序列
------

```C++
// naive 的做法从前往后，记录字符串
// 观察 ATCG 四个字符的特征，并把他们编码为一个int
// 十个字符正好编码在 32bit 的 int 中
vector<string> findRepeatedDnaSequences(string s) {
    unordered_map<int, int> hash;
    vector<string> result;
    for (int t = 0, i = 0; i < s.size(); i++)
        // 左移弹出老元素，求交为了只使用30bit，求或添加新元素。
        if (hash[t = t << 3 & 0x3FFFFFFF | s[i] & 0b111]++ == 1) // 等于1为了避免重复
            result.push_back(s.substr(i - 9, 10));
    return result;
}
```

189. 翻转树组
------

```C
void reverse(int* nums, int left, int right) {
    while (left < right) {
        int temp = nums[left];
        nums[left] = nums[right];
        nums[right] = temp;
        left++;
        right--;
    }

}

void rotate(int* nums, int numsSize, int k) {
    if (k >= numsSize) k %= numsSize;
    if (k <= 0) return;
    reverse(nums, 0, numsSize - k - 1);
    reverse(nums, numsSize - k, numsSize - 1);
    reverse(nums, 0, numsSize - 1);
}
```

190. 翻转二进制表示
------


```C
uint32_t reverseBits(uint32_t n) {
    uint32_t r = 0;
    int len = sizeof(n) * 8 - 1;
    while (len--) { // 31 times shift
        r |= n & 0x1;
        n >>= 1;
        r <<= 1; // only shift 31 times
    }
    
    r |= n & 0x1;
    return r;
}
```

191. 数字二进制表示中1的个数
------

我们知道n&(n-1)会把 n 中的最后一个1去掉，所以循环直到 n 为0即可

```C
int hammingWeight(uint32_t n) {
    int count = 0;
    while (n) {
        n &= n - 1;
        count++;
    }
    return count;
}
```

还可以采用查表法，对于表我们可以预先构造，活着利用上一个方法生成，对于长度过大的，我们可以分块查表。

```
#include <stdio.h>
#include <stdlib.h>

int counts[16];

int _get_count(n) {
    int count = 0;
    while (n) {
        n &= n-1;
        count++;
    }
    return count;
}

int init_counts() {
    for (int i = 0; i < 16; i++)
        counts[i] = _get_count(i);
};

int get_count(n) {
    int count = 0;
    while (n) {
        int index = n & 0xF;
        count += counts[index];
        n >>= 4;
    }
    return count;
}

int main() {
    init_counts();
    for (int i = 0; i < 100; i++)
        printf("%d: %d\n", _get_count(i), get_count(i));
    return 0;
}
```

192-197. Missing
------

198.  有一排房子，每个房子中都有一定财产，但是不能偷相邻的两个房子，求能偷到的最大值
------

使用 DP，对于每个房子，可以选择不偷或者前 i-1个房子加上偷当前房子，即`dp[i+1]
= max(dp[i], dp[i-1] + A[i])`

```C
int rob(int* nums, int numsSize) {
    if (!nums) return 0;
    // 因为不能相邻，所以可以从相隔一个的取值
    // dp[n] = max(dp[n-1], dp[n-2] + A[n])
    int temp, m = 0, n = nums[0];
    for (int i = 1; i < numsSize; i++) {
        temp = n;
        if (m + nums[i] > n)
            n = m + nums[i];
        m = temp;
    }
    return n;
}
```

199. 从右边看二叉树的效果
------

```C++
// level order 遍历
vector<int> rightSideView(TreeNode* root) {
    vector<int> result;
    if (!root)
        return result;
    queue<TreeNode*> q;
    q.push(root);
    
    while (!q.empty()) {
        TreeNode* node;
        int len = q.size(); // 保存为了获得最后一个元素
        for (int i = 0; i < len; i++) { // 当前数组的最后一个元素就是最右边的元素
            node = q.front();
            q.pop();
            if (node->left)
                q.push(node->left);
            if (node->right)
                q.push(node->right);
        }
        result.push_back(node->val);
    }
    return result;
}
```

200. 找出小岛的数量
------

```C++
class UnionFind {
private:
    vector<int> m_father, m_rank;
    int m_count; // sets count
public:
    UnionFind(int n): m_father(n), m_rank(n, 0), m_count(n) {
        for (int i = 0; i < m_father.size(); i++)
            m_father[i] = i;
    }
    
    int find(int x) {
        if (x != m_father[x])
            m_father[x] = find(m_father[x]);
        return m_father[x];
    }
    
    void unionify(int x, int y) {
        x = find(x);
        y = find(y);
        
        if (x == y) return;
        
        if (m_rank[x] > m_rank[y]) {
            m_father[y] = x;
        } else {
            if (m_rank[x] == m_rank[y])
                m_rank[y]++;
            m_father[x] = y;
        }
        m_count--;
    }
    
    int getCount() {
        return m_count;
    }
};

class Solution {
const static char LAND = '1';
const static char WATER = '0';

public:
    int numIslands(vector<vector<char>>& grid) {
        if (grid.empty() || grid[0].empty())
            return 0;
        int r = grid.size(), c = grid[0].size();
        UnionFind uf(r * c + 1); // extra element is for water
        for (int i = 0; i < r; i++) {
            for (int j = 0; j < c; j++) {
                if (grid[i][j] == LAND) {
                    if (i != r - 1 && grid[i+1][j] == LAND)
                         uf.unionify(i*c+j, (i+1)*c+j);
                    if (j != c - 1 && grid[i][j+1] == LAND)
                         uf .unionify(i*c+j, i*c+j+1);
                } else {
                    uf.unionify(i*c+j, c*r);
                }
            }
        }
        return uf.getCount() - 1; // islands + water - 1;
    }
};
```

201. 给定区间内，所有数字AND的结果
------

显然直接过一遍是会超时的，那么分析可知

```C
// 如果两个数不相等，一定是有不同的位，那么这一位一定为0
int rangeBitwiseAnd(int m, int n) {
    int t = 0;
    while (m != n) {
        t++;
        m >>= 1;
        n >>= 1;
    }
    return m << t;
}
```

202. 快乐数字，各位数字平方相加得到下一个数字，如果最后等于1
------

没啥，一直算就可以了。。

```C++
bool isHappy(int n) {
    while (n > 6) {
        int next = 0;
        while (n) {
            next += (n%10) * (n%10);
            n /= 10;
        }
        n = next;
    }
    return n == 1;
}
```

203. 删除链表中给定的值
------

```C
struct ListNode* removeElements(struct ListNode* head, int val) {
    struct ListNode dummy, *p = &dummy;
    dummy.next = head;
    while (p) {
        if (p->next && p->next->val == val) { // not forward here
            struct ListNode* next = p->next;
            p->next = next->next;
            free(next);
        } else {
            p = p->next;
        }
        
    }
    return dummy.next;
}
```

204. 找出素数
------

什么筛子，忘了

```C++
int countPrimes(int n) {
    vector<bool> primes(n, true);
    primes[0] = primes[1] = false;
    
    for (int i = 2; i * i < n; i++) // 注意，只到sqrt(n)
        if (primes[i])
            for (int j = i * i; j < n; j += i) // 从 i * i 开始，因为i* i--已经被杀过了
                primes[j] = false;
                
    int count = 0;
    for (int i = 2; i < n; i++)
        if (primes[i])
            count++;
    return count;
}
```

205. 同构字符串，可以看作word pattern的简化
------

```C
bool isIsomorphic(char* s, char* t) {
    int ss[256] = { 0 };
    int ts[256] = { 0 };
    if (strlen(s) != strlen(t))
        return false;
    int i = 0;
    while (s[i]) {
        if (ss[s[i]] != ts[t[i]])
            return false;
        ss[s[i]] = ts[t[i]] = i + 1;
        i++;
    }
    return true;
}
```

206. 反转链表
------

```C
struct ListNode* reverseList(struct ListNode* head) {
    if (!head || !head->next)
        return head;
    struct ListNode *p = NULL, *cur = head, *next;
    
    while (cur) {
        next = cur->next; // cache
        cur->next = p; // reverse pointing
        p = cur; // moves forwards
        cur = next;
    }
    return p;
}
```

```C
// recursive
```

*207. 标准的拓扑排序
------

给定边这种方法表示图也是醉了

```C++
bool canFinish(int numCourses, vector<pair<int, int>>& prerequisites) { // next -> before
    vector<unordered_set<int>> graph(numCourses); // 每条边和他的下一步，临接表
    for (auto& p : prerequisites)
        graph[p.second].insert(p.first); 
        
    vector<int> d(numCourses, 0); // in degree
    
    for (auto& nexts : graph)
        for (auto next : nexts)
            d[next]++;
    
    for (int i = 0; i < numCourses; i++) {
        int nondep; // in degree == 0
        for (nondep = 0; nondep < numCourses && d[nondep] != 0; nondep++)
            ;
        if (nondep == numCourses)
            return false;
        d[nondep] = -1; // remove
        for (auto next : graph[nondep]) // 所有下一步都 －1 
            d[next]--;
    }
    
    return true;
}
```



208. 实现前缀树
------

```C++
class TrieNode {
public:
    static const int branchCount = 26;
    bool isWord;
    TrieNode* next[branchCount];
    // Initialize your data structure here.
    TrieNode() : isWord(false) {
        for (int i = 0; i < branchCount; i++)
            next[i] = NULL;
    }
};

class Trie {
public:
    Trie() {
        root = new TrieNode();
    }

    // Inserts a word into the trie.
    void insert(string word) {
        TrieNode* location = root;
        for (auto& c : word) {
            if (!location->next[c - 'a'])
                location->next[c - 'a'] = new TrieNode;
            location = location->next[c - 'a'];
        }
        location->isWord = true;
    }

    // Returns if the word is in the trie.
    bool search(string word) {
        TrieNode* location = root;
        for (auto& c : word) {
            location = location->next[c - 'a'];
            if (!location)
                return false;
        }
        return location->isWord;
    }

    // Returns if there is any word in the trie
    // that starts with the given prefix.
    bool startsWith(string prefix) {
        TrieNode* location = root;
        for (auto& c : prefix) {
            location = location->next[c - 'a'];
            if (!location)
                return false;
        }
        return true;
    }

private:
    TrieNode* root;
};

// Your Trie object will be instantiated and called as such:
// Trie trie;
// trie.insert("somestring");
// trie.search("key");
```

209. 最短子数组使得和大于某个数
------

双指针，超过和之后再尝试从开始处减去元素

```C++
int minSubArrayLen(int s, vector<int>& nums) {
    int start = 0, sum = 0, len = INT_MAX;
    for (int i = 0; i < nums.size(); i++) {
        sum += nums[i];
        while (sum >= s) {
            len = min(len, i - start + 1);
            sum -= nums[start++];
        }
    }
    
    return len == INT_MAX? 0 : len;
}
```

210. Course Schedule II
------

BFS

```C++
class Solution {
public:
    vector<int> findOrder(int numCourses, vector<pair<int, int>>& prerequisites) {
        vector<unordered_set<int>> graph = make_graph(numCourses, prerequisites);
        vector<int> degrees = compute_indegree(graph);
        queue<int> zeros;
        for (int i = 0; i < numCourses; i++)
            if (!degrees[i]) zeros.push(i);
        vector<int> toposort;
        for (int i = 0; i < numCourses; i++) {
            if (zeros.empty()) return {};
            int zero = zeros.front();
            zeros.pop();
            toposort.push_back(zero);
            for (int neigh : graph[zero]) {
                if (!--degrees[neigh])
                    zeros.push(neigh);
            }
        }
        return toposort;
    }
private:
    vector<unordered_set<int>> make_graph(int numCourses, vector<pair<int, int>>& prerequisites) {
        vector<unordered_set<int>> graph(numCourses);
        for (auto pre : prerequisites)
            graph[pre.second].insert(pre.first);
        return graph; 
    }
    vector<int> compute_indegree(vector<unordered_set<int>>& graph) {
        vector<int> degrees(graph.size(), 0);
        for (auto neighbors : graph)
            for (int neigh : neighbors)
                degrees[neigh]++;
        return degrees;
    }
};
```

211. 添加和搜索字符串
------

```C++
class TrieNode {
public:
    static const int branchCount = 26;
    bool isWord;
    TrieNode* next[branchCount];
    // Initialize your data structure here.
    TrieNode() : isWord(false) {
        for (int i = 0; i < branchCount; i++)
            next[i] = NULL;
    }
};

class Trie {
public:
    Trie() {
        root = new TrieNode();
    }

    // Inserts a word into the trie.
    void insert(string word) {
        TrieNode* location = root;
        for (auto& c : word) {
            if (!location->next[c - 'a'])
                location->next[c - 'a'] = new TrieNode;
            location = location->next[c - 'a'];
        }
        location->isWord = true;
    }

    // Returns if the word is in the trie.
    virtual bool search(string word) {
        TrieNode* location = root;
        for (auto& c : word) {
            location = location->next[c - 'a'];
            if (!location)
                return false;
        }
        return location->isWord;
    }

    // Returns if there is any word in the trie
    // that starts with the given prefix.
    bool startsWith(string prefix) {
        TrieNode* location = root;
        for (auto& c : prefix) {
            location = location->next[c - 'a'];
            if (!location)
                return false;
        }
        return true;
    }
    
    TrieNode* getRoot() {
        return root;
    }

private:
    TrieNode* root;
};


class WordDictionary : public Trie{

public:
    WordDictionary() : Trie(){}

    // Adds a word into the data structure.
    void addWord(string word) {
        insert(word);
    }

    // Returns if the word is in the data structure. A word could
    // contain the dot character '.' to represent any one letter.
    bool search(string word) override {
        return search(word.c_str(), getRoot());
    }
    
    bool search(const char* word, TrieNode* root) {
        TrieNode* run = root;
        for (int i = 0; word[i]; i++) {
            if (run && word[i] != '.')
                run = run->next[word[i] - 'a'];
            else if (run && word[i] == '.') { 

                // skip checking this char
                TrieNode* tmp = run;
                for (int j = 0; j < 26; j++) {
                    run = tmp->next[j];
                    if (search(word + i + 1, run))
                        return true;
                }
            }
            else break;
        }
        return run && run->isWord; 
    }
};

// Your WordDictionary object will be instantiated and called as such:
// WordDictionary wordDictionary;
// wordDictionary.addWord("word");
// wordDictionary.search("pattern");
```

212. 单词搜索
------

Trie结构见前面，注意要记录visited，还有边界的问题，另外集合的使用

```C++
class Solution {
private:
    Trie m_trie;
public:
    vector<string> findWords(vector<vector<char>>& board, vector<string>& words) {
        for (auto& word : words)
            m_trie.insert(word);
        int row = board.size();
        int col = board[0].size();
        
        unordered_set<string> result_set;
        vector<vector<bool>> visited(row, vector<bool>(col, false));
        for (int i = 0; i < row; i++)
            for(int j = 0; j < col; j++)
                find(result_set, board, visited, "", i, j);
        vector<string> result;
        for (auto& r : result_set)
            result.push_back(r);
        return result;
    }
    
    void find(unordered_set<string>& r, vector<vector<char>>& board, vector<vector<bool>>& visited, string word, int i, int j)
    {
        if (i < 0 || i >= board.size() || j < 0 || j >= board[0].size() || visited[i][j])
            return;
        word += board[i][j];
        if (!m_trie.startsWith(word))
            return;
        if (m_trie.search(word))
            r.insert(word);
        
        visited[i][j] = true;
        find(r, board, visited, word, i-1, j);
        find(r, board, visited, word, i+1, j);
        find(r, board, visited, word, i, j-1);
        find(r, board, visited, word, i, j+1);
        visited[i][j] = false;

    }
    
};
```

213. 小偷偷环状数组
------

```C
int max(int a, int b) {
    return a > b ? a : b;
}

int robNonCyclic(int* nums, int numsSize) {
    if (!nums) return 0;
    // 因为不能相邻，所以可以从相隔一个的取值
    // dp[n] = max(dp[n-1], dp[n-2] + A[n])
    int temp, m = 0, n = nums[0];
    for (int i = 1; i < numsSize; i++) {
        temp = n;
        if (m + nums[i] > n)
            n = m + nums[i];
        m = temp;
    }
    return n;
}

int rob(int* nums, int numsSize) {
    return max(robNonCyclic(nums, numsSize - 1), robNonCyclic(nums + 1, numsSize - 1));
}
```

*214. 最短回文字符串，给指定的字符串添加字母获得回文
------

```C++
// based on kmp next array
string shortestPalindrome(string s) {
    string rev_s = s;
    reverse(rev_s.begin(), rev_s.end());
    string l = s + "#" + rev_s;

    vector<int> p(l.size(), 0);
    for (int i = 1; i < l.size(); i++) {
        int j = p[i - 1];
        while (j > 0 && l[i] != l[j])
            j = p[j - 1];
        p[i] = (j += l[i] == l[j]);
    }

    return rev_s.substr(0, s.size() - p[l.size() - 1]) + s;
}
```


*215. 数组中第k大的数字
------

实际上这道题更可能的题目是找到前k大的所有数字。
首先，设计到数组排序的问题一定向面试官要问清楚数据量的大小，这影响到接下来的实现，
同时和面试官探讨数据量大小对实现的影响，有助于更好的把握局面。

我们先假设数据量是比较小的，也就是能够放到内存中。

0. 使用排序就实在是naive了，不过面试官非要问的话，当然是使用选择排序更好了。
1. 使用快排中的partition算法，时间复杂度O(n*logk)。
2. 使用size为k的堆，时间复杂度也是O(n*logk)，不管数字多大，都只需要遍历一遍。
3. 使用类似插入排序的方法，保持数组大小不变，这样的时间复杂度是O(n*k)。
4. 数据的范围有限时候，使用计数排序。

当数据过大的时候，我们可以想通过哈希取模之后把文件分组，找出每个文件中最大的k个数字

如果数字中有重复呢？使用计数排序，计数强制按一算
如果既有重复又是浮点数呢？

```C
int swap(int* a, int* b) {
    int t = *a;
    *a = *b;
    *b = t;
}

int partition(int* nums, int start, int end) {
    int small = start - 1;
    int pivot = nums[end];
    for (int i = start; i < end; i++)
        if (nums[i] < pivot)
            swap(&nums[++small], &nums[i]);
    swap(&nums[++small], &nums[end]);
    return small;
}

int findKthLargest(int* nums, int numsSize, int k) {
    int left = 0, right = numsSize - 1;
    while (1) {
        int index = partition(nums, left, right);
        if (index == numsSize - k)
            return nums[index];
        if (index > numsSize - k)
            right = index - 1;
        else
            left = index + 1;
    }
}
```

216. 找到k个数字[1...9]，使得他们的和是n
------

```C++
vector<vector<int>> combinationSum3(int k, int n) {
    vector<vector<int>> result;
    dfs(result, {}, n, k);
    return result;
}

void dfs(vector<vector<int>>& result, vector<int> combination, int n, int k) {
    if (combination.size() == k) {
        if (n == 0)
            result.push_back(combination);
        return;
    }
    int i = combination.empty() ? 1 : combination.back() + 1; // 保证不重复切实递增序列
    while (i <= n && i < 10) {
        combination.push_back(i);
        dfs(result, combination, n-i, k);
        combination.pop_back();
        i++;
    }
}
```

217. 包含重复数字
------

这道题太简单了，也没有什么精妙的解法，可以使用排序，Hash 等多种解法

```C++
bool containsDuplicate(vector<int>& nums) {
    unordered_set<int> s;
    for (auto& n : nums)
        if (s.find(n) != s.end())
            return true;
        else
            s.insert(n);
    return false;
}
```

*218. 获得矩形重合部分的拐点
------

```C++
vector<pair<int, int>> getSkyline(vector<vector<int>>& buildings) {
    vector<pair<int, int>> res;
    int cur=0, cur_X, cur_H =-1,  len = buildings.size();
    priority_queue< pair<int, int>> liveBlg; // first: height, second, end time
    while(cur<len || !liveBlg.empty()) { // if either some new building is not processed or live building queue is not empty
        cur_X = liveBlg.empty()? buildings[cur][0]:liveBlg.top().second; // next timing point to process

        if(cur>=len || buildings[cur][0] > cur_X) { //first check if the current tallest building will end before the next timing point
              // pop up the processed buildings, i.e. those  have height no larger than cur_H and end before the top one
            while(!liveBlg.empty() && ( liveBlg.top().second <= cur_X) ) liveBlg.pop();
        } else { // if the next new building starts before the top one ends, process the new building in the vector
            cur_X = buildings[cur][0];
            while(cur<len && buildings[cur][0]== cur_X)  // go through all the new buildings that starts at the same point
            {  // just push them in the queue
                liveBlg.push(make_pair(buildings[cur][2], buildings[cur][1]));
                cur++;
            }
        }
        cur_H = liveBlg.empty()?0:liveBlg.top().first; // outut the top one
        if(res.empty() || (res.back().second != cur_H) ) res.push_back(make_pair(cur_X, cur_H));
    }
    return res;
}
```


219. 包含重复数字，并且两个的坐标不超过k
------

```C++
// 滑动窗口保存前k个值，如果有重复的就返回
// num[i-k] num[i-1]，如果滑过了，就删除该元素
bool containsNearbyDuplicate(vector<int>& nums, int k) {
    unordered_set<int> s;
    if (k <= 0)
        return false;
    if (k >= nums.size()) // notice here
        k = nums.size() - 1;
        
    for (int i = 0; i < nums.size(); i++) {
        if (i > k)
            s.erase(nums[i - k - 1]); // delete first note
        if (s.find(nums[i]) != s.end())
            return true;
        s.insert(nums[i]); // insert
    }
    
    return false;
}
```

*220. 同上一题，同时保证两个数字之间小于t
------

保证两个数字之差小于t

```C++
bool containsNearbyAlmostDuplicate(vector<int>& nums, int k, int t) {
    set<int> window; // 注意不能使用unordered
    if (k <= 0)
        return false;
    if (k >= nums.size()) // notice here
        k = nums.size() - 1;
    for (int i = 0; i < nums.size(); i++) {
        if (i > k)
            window.erase(nums[i - k - 1]);
        auto pos = window.lower_bound(nums[i] - t); // notice set.lower_bound
        if (pos != window.end() && *pos - nums[i] <= t)
            return true;
        window.insert(nums[i]);
    }
    return false;
}
```

＊221. 找到最大的正方形
------

使用动态规划 https://leetcode.com/discuss/38489/easy-solution-with-detailed-explanations-8ms-time-and-space

```C++
int maximalSquare(vector<vector<char>>& matrix) {
    if (matrix.empty()) return 0;
    int m = matrix.size(), n = matrix[0].size();
    vector<int> dp(m + 1, 0);
    int maxsize = 0, pre = 0;
    for (int j = 0; j < n; j++) { // 每一列
        for (int i = 1; i <= m; i++) { // notice i range
            int temp = dp[i];
            if (matrix[i - 1][j] == '1') {
                dp[i] = min(dp[i], min(dp[i - 1], pre)) + 1;
                maxsize = max(maxsize, dp[i]);
            }
            else dp[i] = 0; 
            pre = temp;
        }
    }
    return maxsize * maxsize;
}
```




222. 给定一个完全树，计算节点的数量。
------

```C++
int countNodes(struct TreeNode* root) {
    if (!root)
             return 0;
    int left_height = 0, right_height = 0;
    struct TreeNode* left = root, *right = root;
    while (left) {
        left = left->left;
        left_height++;
    }
    
    while (right) {
        right = right->right;
        right_height++;
    }
    
    if (left_height == right_height) // 满树 2^h - 1
        return (1 << left_height) - 1;
    
    return countNodes(root->left) + countNodes(root->right) + 1;
}
```

223. 找出两个长方形覆盖的面积
------

```C
int computeArea(int left1, int down1, int right1, int up1, int left2, int down2, int right2, int up2) {
    int left = max(left1, left2); //靠右的
    int right = max(min(right1, right2), left);// 靠左的，但是比左边大
    
    int down = max(down1, down2);
    int up = max(min(up1, up2), down);
    
    // 不小心写反了。。
    return -((left1 - right1) * (up1 - down1) + (left2 - right2) * (up2 - down2) - (left - right) * (up - down));
}
```

224. 给定一个字符串，包含加减和括号，计算值
------

难点是对括号的处理，注意每次都要和signs.top()相乘

```C++
int calculate(string s) {
    stack<int> signs; // signs before braces
    int sign = 1;
    int num = 0;
    int result = 0;
    
    signs.push(1);
    for (auto c : s) {
        if (isdigit(c)) {
            num = 10 * num + (c - '0');
        } else if (c == '+' || c == '-') {
            result += signs.top() * sign * num;
            num = 0;
            sign = c == '+' ? 1 : -1;
        } else if (c == '(') { 
            signs.push(sign * signs.top()); // tricky
            sign = 1;
        } else if (c == ')') {
            result += signs.top() * sign * num;
            num = 0;
            signs.pop();
            sign = 1;
        }
    }

    result += signs.top() * sign * num; // tricky
    
    return result;
}
```

225. 使用队列模拟栈
------

其实有两种做法，一种是在push的时候，把队列清空，把x放到最底下。
另一种是在pop的时候，把队列清空到1，然后弹出。应当询问面试官究竟是push居多还是pop居多

```C++
class Stack {
public:
    // Push element x onto stack.
    void push(int x) {
        while (!nums.empty()) {
            temp.push(nums.front());
            nums.pop();
        }
        nums.push(x);
        while (!temp.empty()) {
            nums.push(temp.front());
            temp.pop();
        }
    }

    // Removes the element on top of the stack.
    void pop() {
        nums.pop();
    }

    // Get the top element.
    int top() {
        return nums.front();
    }

    // Return whether the stack is empty.
    bool empty() {
        return nums.empty();
    }
private:
    queue<int> nums;
    queue<int> temp;
};
```

226. 反转二叉树
------

```C
struct TreeNode* invertTree(struct TreeNode* root) {
    if (!root) return NULL;
    struct TreeNode* temp = root->left;
    root->left = invertTree(root->right);
    root->right = invertTree(temp);
    return root;
}
```

227. 给定一个字符串包含+-*/计算他的值
------

```C++
int calculate(string s) {
    vector<int> stk; // 使用vector便于统计最后的值
    char token = '+';
    int num = 0;
    for (int i = 0; i < s.size(); i++) {
        if (isdigit(s[i]))
            num = num * 10 + s[i] - '0';
        // 这里不是else if
        if (s[i] == '+' || s[i] == '-' || s[i] == '*' || s[i] == '/' || i == s.size() - 1) { // 注意最后一步还需要把最后的值计算
            int a;
            switch (token) {
            case '+':
                stk.push_back(num);
                break;
            case '-':
                stk.push_back(-num);
                break;
            case '*':
                a = stk.back();
                stk.pop_back();
                stk.push_back(a * num);
                break;
            case '/':
                a = stk.back();
                stk.pop_back();
                stk.push_back(a / num);
                break;
            };
            token = s[i];
            num = 0;
        }
    }
    
    int result = 0;
    for (auto i : stk)
        result += i;
    return result;
}
```

聚合区间，给定一排序数组，把相邻的数字用区间表示
------

```C++
vector<string> summaryRanges(vector<int>& nums) {
    int n = nums.size();
    vector<string> result;
    if (n == 0) return result;
    
    for (int i = 0; i < n; ) {
        int start = i, end = i;
        while (end + 1 < n && nums[end + 1] == nums[end] + 1)
            end++;
        if (end > start)
            result.push_back(to_string(nums[start]) + "->" + to_string(nums[end]));
        else
            result.push_back(to_string(nums[start]));
        i = end + 1;
    }
    
    return result;
}
```

229. 找出超过三分之一的元素
------

```C++
vector<int> majorityElement(vector<int>& nums) {
    
    int count1 = 0, count2 = 0;
    int a, b;
    
    for (auto n : nums) {
        if (count1 == 0 || n == a) {
            count1++;
            a = n;
        } else if (count2 == 0 || n == b) {
            count2++;
            b = n;
        } else {
            count1--;
            count2--;
        }
    }
    
    count1 = count2 = 0;
    for (int n : nums) {
        if (n == a) count1++;
        if (n == b) count2++;
    }
    
    vector<int> result;
    
    if (count1 > nums.size() / 3) // verify a
        result.push_back(a);
    if (count2 > nums.size() / 3 && a != b) // verify b
        result.push_back(b);
    return result;
}
```

＊230. 二叉树中第k小的数字
------

```C
// 传递指针
void inorder(struct TreeNode* root, int* k, int* number) {
    if (!root)
        return;
    inorder(root->left, k, number);
    (*k)--;
    if (*k == 0) {
        *number = root->val;
        return;
    }
    inorder(root->right, k, number);
}
int kthSmallest(struct TreeNode* root, int k) {
    int number;
    inorder(root, &k, &number); 
    return number;
}
```

231. 2的次方
------

```
bool isPowerOfTwo(int n) {
    if (n <= 0) return false;
    return (n & (n - 1)) == 0;
}
```



232. 使用栈模拟队列
------

```C++
class Queue {
public:
    // Push element x to the back of queue.
    void push(int x) {
        in.push(x);
    }

    // Removes the element from in front of queue.
    void pop(void) {
        if (empty())
            return;
        if (out.empty())
            transfer();
        out.pop();
    }

    // Get the front element.
    int peek(void) {
        if (empty())
            return INT_MIN;
        if (out.empty())
            transfer();
        return out.top();
    }

    // Return whether the queue is empty.
    bool empty(void) {
        return in.empty() && out.empty();
    }
private:
    void transfer() {
        while (!in.empty()) {
            out.push(in.top());
            in.pop();
        }
    };
    stack<int> in;
    stack<int> out;
};
```

*233. 小于n的数字中1的个数
------

对于每一位，有三种情况：

1. 当是数字0的时候，可能出先1的情况完全由高位出现决定，因为这一位不能贡献1
2. 当是数字1的时候，同上，但是这一位和低位一起可以贡献一个1
3. 当时数字2-9的时候，相当于这一位的1可以任意出现，因此高位＋1

```C
int countDigitOne(int n) {
    int ones = 0;
    for (int m = 1; m <= n; m *= 10) { // m is the factor
        int a = n/m, b = n%m;  // a is left half, b is right half
        if (a % 10 >= 2)
            ones += (a / 10 + 1) * m;    
        if (a % 10 == 1)
            ones += (a / 10) * m + b + 1;
        if (a % 10 == 0)
            ones += (a / 10) * m;
    }
    return ones;
}
```

二进制呢

```C
int countDigitOneBinary(int n) {
    int ones = 0;
    for (int m = 1; m <= n; m <<= 1) {
        int a = n / m, b = n % m;
        if (a & 0x01)
            ones += (a >> 1) * m + b + 1;
        else
            ones += (a >> 1) * m;
    }
}
```

求最大的countDigitOne(n) == n

    9    1
    99   20
    999  300
    ...
    99999999  10000000

即｀f(10^n-1) ＝ n * 10^(n-1)｀，找到之后递减找最大的即可

234. 判断一个链表是否是回文
------

解法1: 如果链表是可以改变的，不妨反转它的前半部分，然后再与后半部分比较

解法2: 如果是只读的，复制一份也可以，但是不如使用堆栈

注意对奇数偶数的处理

```C++
bool isPalindrome(ListNode* head) {
    if (!head || !head->next)
        return true;
    int len = 0;
    ListNode* temp = head;
    while (temp) {
        len++;
        temp = temp->next;
    }
    
    stack<int> stk;
    temp = head;
    int mid = len / 2;
    while (mid--) {
        stk.push(temp->val);
        temp = temp->next;
    }
    
    if (len & 0x01)
        temp = temp->next;
    
    while (temp != NULL && !stk.empty()) {
        int a = stk.top();
        stk.pop();
        int b = temp->val;
        temp = temp->next;
        if (a != b) {
            return false;
        }
    }
    
    return true;
}
```

235. 二叉搜索树公共祖先
------

```C
struct TreeNode* lowestCommonAncestor(struct TreeNode* root, struct TreeNode* p, struct TreeNode* q) {
    while (root) {
        if (root->val > p->val && root->val > q->val)
            root = root->left;
        else if (root->val < p->val && root->val < q->val)
            root = root->right;
        else
            return root;
    }
}
```

*236. 二叉树公共祖先
------

如果二叉树的根就是其中一个节点，那显然是这个。
在两颗子树中分别查找，如果找到了，返回一个非 NULL 值，如果都找到了，则这个节点就是 LCA

```C
struct TreeNode* lowestCommonAncestor(struct TreeNode* root, struct TreeNode* p, struct TreeNode* q) {
    if (!root || root == p || root == q)
        return root;
    struct TreeNode* left = lowestCommonAncestor(root->left, p, q);
    struct TreeNode* right = lowestCommonAncestor(root->right, p, q);
    
    if (!left) // not in left subtree
        return right;
    if (!right)
        return left;
    return root; // both left and right are found!
}
```

237. 删除链表中的元素
------

直接将后继节点的值复制到当前节点

```C
void deleteNode(struct ListNode* node) {
    if (!node || !node->next)
        return;
    struct ListNode* next = node->next;
    node->val = next->val;
    node->next = next->next;
    free(next);
}
```

238. 数组除了自己以外的乘积，规定不能用除法
------

首先从前往后乘，错开一位元素，这样每个元素都乘到了他之前的所有元素，最后一个元素已经是结果了。
然后从后往前乘，同样错开一位，这样每个元素又把他之后的元素都得到了。

```C++
    
```


239. 滑动窗口最大值，给定一个滑动窗口，返回它移动过程中的最大值
------

这道题和 min stack 的思路完全一样，只不过换成了 deque

```C++
// 题目给定 k 一定是有效地
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    vector<int> result;
    if (nums.empty() || k <= 0)
        return result;
    deque<int> dq; // 存储的是索引，front 存储最大值，保证递减
    for (int i = 0; i < nums.size(); i++) {
        while (!dq.empty() && dq.front() < i - k + 1) // 弹出滑过的窗口
            dq.pop_front();
        while (!dq.empty() && nums[dq.back()] < nums[i]) // 弹出小的
            dq.pop_back();
        dq.push_back(i);
        if (i >= k - 1)
            result.push_back(nums[dq.front()]);
    }
    return result;
}
```

240. 给定一个矩阵，每行从左到右都是增大的，每一列从上到下都是增大的，找出给定数字是否存在
------

我们考虑右上角的元素

1.如果这个元素比 taget 大，那么整列都比 target 大，我们可以 c--
2.如果这个元素比 target 小，那么正行都比 target 小，我们可以 r++

```C
bool searchMatrix(int** matrix, int row, int col, int target) {
    int r = 0, c = col - 1; // 右上角
    while (r < row && c > -1) // 向左下角
        if (matrix[r][c] == target)
            return true;
        else if (matrix[r][c] > target)
            c--;
        else
            r++;
    return false;
}
```

241. 添加括号得到不同的结果
------

对每一个符号，在他的两边添加括号的好的不同结果再计算。

```C++
vector<int> diffWaysToCompute(string input) {
    vector<int> output;
    for (int i = 0; i < input.size(); i++) {
        char token = input[i];
        if (!isdigit(token)) // not digit
            for (int a : diffWaysToCompute(input.substr(0, i))) // 左半部分
                for (int b : diffWaysToCompute(input.substr(i+1))) // 右半部分
                    output.push_back(token == '+' ? a + b : token == '-'? a - b: a *b); // 两半部分之和
    }
    
    if (output.empty())
        output.push_back(stoi(input));
    return output;
}
```

242. 一个单词是否能由另一个变幻而来
------

还是，对于 ASCII 字符，直接用数组代替字典

```C
bool isAnagram(char* s, char* t) {
    char ss[26] = {0};
    char ts[26] = {0};
    while (*s) {
        ss[*s - 'a']++;
        s++;
        ts[*t - 'a']++;
        t++;
    }
    if (*t) return false;
    return memcmp(ss, ts, sizeof(ss)) == 0;
}
```

243-256 Locked
------

257. 二叉树左右路径
------

典型的 DFS，发挥所有从根节点到叶节点的路径

```C++
vector<string> binaryTreePaths(TreeNode* root) {
    vector<string> result;
    if (!root) return result;
    paths(result, "", root);
    return result;
}

void paths(vector<string>& result, string path, TreeNode* root) {
    if (path.empty()) 
        path += to_string(root->val);
    else 
        path += "->" + to_string(root->val);
    if (root->left)
        paths(result, path, root->left);
    if (root->right)
        paths(result, path, root->right);
    if (!root->left && !root->right)
        result.push_back(path);
}
```  

258. 把数字的每一位加起来，直到变成一个一位的数字
------

这完全是一道数学题，对于每个进制的数字都有规律 `(n - 1) % (x - 1) + 1`。实际上是把10进制的转化为9进制数字

```
int addDigits(int num) {
    return (num - 1) % 9 + 1;
}
```

259 Locked
------

260. 给定一个数组，每个数字都是重复的，只有两个数字不是，找出这两个数字
------

这道题很奇妙，依然可以使用 XOR 来解，首先遍历一遍，这时候由于有两个数字是不同的，那么一定结果不为0，那么其中一个 bit 位一定是一个数字有，另一个数字没有。
在遍历一遍，同时把数字分两组，一组是有这个 bit 位，一组没有。就得出了结果。

```C++
vector<int> singleNumber(vector<int>& nums) {
    int r = 0;
    for (auto& n : nums)
        r ^= n; 
    int bit = r & -r; // last sig bit

    vector<int> result = {0, 0};
    for (auto& n : nums)
        if (n & bit)
            result[0] ^= n;
        else
            result[1] ^= n;
    return result;
}
```

261. 262 Locked
------

263. 丑陋的数字，质数因子只含有2,3,5的数字
------

按定义做就好了

```C
bool isUgly(int n) {
    if (n <= 0)
        return false;
    if (n == 1)
        return true;
    while (n > 1)
        if (n % 2 == 0)
            n /= 2;
        else if (n % 3 == 0)
            n /= 3;
        else if (n % 5 == 0)
            n /= 5;
        else
            return false;
    return true;
}
```

264. 找出第 n 个丑陋数字
------

使用数列记录 n 个丑陋数字，每一个丑陋数字肯定是之前数字乘以235得到的，然后用三个指针分别指向上一个做乘法的数字，每次找出最小的一个

```C
#define MIN(a,b) ((a)<(b)?(a):(b))

int nthUglyNumber(int n) {
    if (n <= 0)
        return -1;
    if (n < 6) // 1..6 恰好都是
        return n;
    int s2 = 0, s3 = 0, s5 = 0;
    int* uglies[n];
    uglies[0] = 1;
    for (int i = 1; i < n; i++) {
        int c2 = uglies[s2] * 2, c3 = uglies[s3] * 3, c5 = uglies[s5] * 5;
        uglies[i] = MIN(c2, MIN(c3, c5));
        if (uglies[i] == c2) s2++;
        if (uglies[i] == c3) s3++;
        if (uglies[i] == c5) s5++;
    }
    return uglies[n-1];
}
```

268. 丢失的数字，给定0...n，丢失了一个，然后放在长度为 n 的数组之中，找出这个数字
------

显然还是使用异或，注意 0 ^ x == x，所以直接把 0 忽略就行了。把每个数字都和 i 异或，丢失的数字就出来了

```C
int missingNumber(int* nums, int n) {
    int result = 0;
    for (int i = 0; i < n; i++)
        result = result ^ (i + 1) ^ nums[i];
    return result;
}
```

269-272 Locked
------

273. 数字转换为英语单词
------

```C++
class Solution {
public:
    vector<string> digits = {"", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine", "Ten", "Eleven", "Twelve", "Thirteen", "Fourteen", "Fifteen", "Sixteen", "Seventeen", "Eighteen", "Nineteen"};
    vector<string> tens = {"", "", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"};
    vector<string> seps = {"", " Thousand ", " Million ", " Billion "}; // notice the trailing spaces
    
    string numberToWords(int num) {
        if (num == 0)
            return "Zero";
        if (num < 0)
            return "Negative " + numberToWords(-num);
        int count = 0;
        string result;
        
        while (num) {
            if (num % 1000 != 0)
                result = s2word(num % 1000) + seps[count] + result;
            num /= 1000;
            count++;
        }
        
        // removw unnecessary tailing space
        if (isspace(result.back()))
            result.resize(result.size() - 1);
        
        return result;
        
    }
    
    string s2word(int num) {
        string result;
        if (num >= 100) {
            result += digits[num/100] + " Hundred ";
            num %= 100;
        }
        
        if (num >= 20) {
            result += tens[num / 10] + " ";
            num %= 10;
        }
        
        if (num >= 1 && num <= 19)
            result += digits[num];
        
        // remove tailing spaces
        if (isspace(result.back()))
            result.resize(result.size() - 1);
            
        return result;
        
    }  
};
```

274. H-Index
------

H-Index的定义：一个科学家的N篇论文h个至少有h个引用，而且剩下的N-h篇论文都没有超过h个引用。

```
int hIndex(int* cites, int n) {
    int hs[n+1]; // Hindex不可能大于N

    for (int i = 0; i <= n; i++)
        hs[i] = 0;

    for (int i = 0; i < n; i++) {
        if (cites[i] > n)
            hs[n]++;
        else
            hs[cites[i]]++;
    }
    
    for (int i = n, papers = 0; i >= 0; i--) { // 从后往前，如果有符合条件的，那么就是Hindex
        papers += hs[i];
        if (papers >= i)
            return i;
    }
    
    return 0;
}
```

275. H-index II，论文已经按照引用数量排序
------

```C
int hIndex(int* citations, int n) {
    int left = 0, right = n - 1;
    while (left <= right) { // 二分查找是小于等于
        int mid = left + (right - left) / 2;
        if (citations[mid] == n - mid)
            return citations[mid];
        else if (citations[mid] < n - mid)
            left = mid + 1;
        else
            right = mid - 1;
    }
    return n - right - 1;
}
```

276-277 Locked
------

278. 第一个坏版本
------

```C
// 实际上是lower_bound函数
int firstBadVersion(int n) {
    int left = 0, right = n; // 记住lower_bound的right是n
    while (left < right) {   // 使用小于号
        int mid = left + (right - left) / 2;
        if (!isBadVersion(mid))
            left = mid + 1;
        else
            right = mid;
    }
    return left;
}
```

279. 分解为平方数的和
------

最多4个即可，尝试在三个以内是否可以。

```C
int numSquares(int n) {
    int ub = sqrt(n);
    for (int a=0; a<=ub; ++a) {
        for (int b=a; b<=ub; ++b) {
            int c = sqrt(n - a*a - b*b);
            if (a*a + b*b + c*c == n)
                return !!a + !!b + !!c;
        }
    }
    return 4;
}
```

282. 添加运算符使得算式成立
------

```C++
vector<string> addOperators(string num, int target) {
    vector<string> result;
    if (num.size() == 0) 
        return result;
    dfs(num, target, result, num[0] - '0', num.substr(0, 1), 1, 1);
    return result;
}

void dfs(string num, int target, vector<string> & v, long long last, string s, int idx, int left) {
    if (idx == num.length()){
        if (target == last*left)
            v.push_back(s);
        return;
    } else {
        if(last!=0) 
            dfs(num, target,         v, last * 10 + num[idx] - '0', s + num.substr(idx, 1), idx + 1, left); // 尝试拼成10
        dfs(num, target,             v, num[idx] - '0', s + '*' + num.substr(idx, 1), idx + 1, last*left);
        dfs(num, target - left*last, v, num[idx] - '0', s + '+' + num.substr(idx, 1), idx + 1, 1);
        dfs(num, target - left*last, v, num[idx] - '0', s + '-' + num.substr(idx, 1), idx + 1, -1);
    }
}
```

283. 移动0
------

注意swap的使用

```C++
void moveZeroes(vector<int>& nums) {
    int n = 0;
    for (int i = 0; i < nums.size(); i++) {
        if (nums[i] != 0)
            swap(nums[n++], nums[i]);
    }
}
```

284. Peek Iterator
------

```C++
// Below is the interface for Iterator, which is already defined for you.
// **DO NOT** modify the interface for Iterator.
class Iterator {
    struct Data;
    Data* data;
public:
    Iterator(const vector<int>& nums);
    Iterator(const Iterator& iter);
    virtual ~Iterator();
    // Returns the next element in the iteration.
    int next();
    // Returns true if the iteration has more elements.
    bool hasNext() const;
};


class PeekingIterator : public Iterator {
public:
    PeekingIterator(const vector<int>& nums) : Iterator(nums) {
        // Initialize any member here.
        // **DO NOT** save a copy of nums and manipulate it directly.
        // You should only use the Iterator interface methods.
        
    }

    // Returns the next element in the iteration without advancing the iterator.
    int peek() {
        return Iterator(*this).next();
    }

    // hasNext() and next() should behave the same as in the Iterator interface.
    // Override them if needed.
    int next() {
        return Iterator::next();
    }

    bool hasNext() const {
        return Iterator::hasNext();
    }
};
```

285. 286 Locked
------

287. 一个n的数组包含了1...n-1中的这些数字，证明一定存在重复，并找出这个重复
------

使用 Pigeon Hole Priciple 可以证明一定存在重复。据说高纳德解这个问题花了四个小时。

我们把这个数组看做一个变幻方程 f(i) = A[i]，把一些数字变幻到另一些，那么存在一个 i != j s.t. f(i) == f(j).
那么这个问题变成了链表求环的问题。对于链表，我们有 n = n->next 遍历列表，对于这个序列，则是 n = f(n) 

```C
int findDuplicate(int* nums, int n) {
    // 从 n-1 开始
    int fast = n - 1, slow = n - 1;
    do {
        slow = nums[slow] - 1; // 减一是为了转化为坐标
        fast = nums[nums[fast] - 1] - 1;
    } while (slow != fast);
    
    fast = n - 1;
    do {
        slow = nums[slow] - 1;
        fast = nums[fast] - 1;
    } while (slow != fast);
    
    return slow + 1; // 从坐标到数字
}
```

288 Locked
------

289. Conway's Game of Life
------

哈哈，机智，使用没有使用的第二个位存储下一代

```C
int max(int a, int b) {return a > b ? a :b;}
int min(int a, int b) {return a < b ? a :b;}
void gameOfLife(int** board, int row, int col) {
    for (int i = 0; i < row; i++) {
        for (int j = 0; j < col; j++) {
            int count = 0;
            for (int m=max(i-1, 0); m<min(i+2, row); m++) // 这里的min，max使用的太屌了
                for (int n=max(j-1, 0); n<min(j+2, col); n++)
                    count += (board[m][n] & 1);
            if (count == 3 || count - board[i][j] == 3) // 当前为0，周围为3；or 当前为1，周围为2/3 here
                board[i][j] |= 2;
        }
    }
    for (int i = 0; i < row; i++)
        for (int j = 0; j < col; j++)
            board[i][j] >>= 1;
}
```

290. 单词模式，给定一个模式abba等，判断单词是否是这个模式的。
------

```C++
bool wordPattern(string pattern, string str) {
    map<char, int> chars;  // 使用两个map纪录
    map<string, int> words;
    istringstream in(str);
    int i = 0, n = pattern.size(); // `i` is word count      
    for (string word; in >> word; i++) {
        if (i == n || chars[pattern[i]] != words[word]) // 检查是否相等
            return false;
        chars[pattern[i]] = words[word] = i + 1; // distinct non zero 
    }
    
    return i == n; // 检查长度是否相等
}
```

291. Locked
------

292. Nim游戏，每个人可以选择丢掉1，2，3，最后一个操作者获胜
------

显然，当我们遇到4的时候会输，其他情况都可以赢。

```C
bool canWinNim(int n) {
    return n % 4 != 0;
}
```

