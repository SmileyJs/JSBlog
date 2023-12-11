+++
title = 'Leetcode_3_subString'
date = 2022-04-24T18:58:25+08:00
draft = false
+++

[leetCode 3 longest sub string](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

这道题要求出string中没有重复字符的最长sub string. 考虑到复杂度的问题, 肯定不能用多次遍历查找所有sub string中最长的那个. 所以最初的思路大概是, 一次遍历的过程中维护一个没有重复字符的substring, 并记录过程中最大的长度.

### 第一版

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        
        using namespace std;
        
        deque<char> subStr;
        int maxLength = 0;
        
        for (int i = 0, j = 0; j <= i && i < s.length();)
        {
            if (find(begin(subStr), end(subStr), s[i]) == end(subStr))
            {
                subStr.push_back(s[i]);
                ++i;
            }
            else
            {
                maxLength = (subStr.size() > maxLength) ? subStr.size() : maxLength;

                subStr.pop_front();
                ++j;
            }
        }
        
        return (subStr.size() > maxLength) ? subStr.size() : maxLength;
    }
};
```
执行的结果是 Runtime: 58 ms，Memory Usage: 7.8 MB， 比80%的解法都慢，不过内存的使用比其他80%都少

后来参考了别人的解法，其实这里要记录的只是substring里有什么字符，并不需要求这个字符，所以没有必要用这么多的push和pop操作。又要拿一个set记录有没有，没有的添加进来，随着移动走过的丢掉就行。

### 第二版

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        
        using namespace std;
        
        unordered_set<char> visited;
        int maxLength = 0;
        int i, j;
        
        for (i = 0, j = 0; j <= i && i < s.length(); ++i)
        {
            if (visited.find(s[i]) == visited.end())
            {
                visited.insert(s[i]);
                maxLength = max(i - j + 1, maxLength);
            }
            else
            {
                while (j < i && s[j] != s[i])
                {
                    visited.erase(s[j++]);
                }
                j++;
                maxLength = max(i - j + 1, maxLength);
            }
        }
        
        return maxLength;
    }
};
```

这个的执行结果：Runtime: 27 ms，Memory Usage: 9 MB。速度快了很多，超过了47%的用户

其实还不够，因为这个set的操作和find还是比较费时间的，如果用一个array记录每个字符有没有和他的位置，然后拿两个index记录substring的起终点，这样就完全不用维护另一个数据结构来记录substring里有什么，而通过每个字符的位置和index比较就知道substring里到底有没有某个字符。而array的index可以用字符的ASCII码的值直接访问，O(1), 效率比标准容器快很懂。

### 最终版

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        
        using namespace std;
        
        vector<int> dict(256, -1);
        int maxLength = 0;
        int right = 0, left = 0;
        
        for (right = 0; right < s.length(); ++right)
        {
            if (dict[s[right]] != -1)
            {
                left = max(dict[s[right]] + 1, left);
            }
            
            dict[s[right]] = right;
            maxLength = max(maxLength, right - left + 1);
        }
        
        return maxLength;
    }
};
```