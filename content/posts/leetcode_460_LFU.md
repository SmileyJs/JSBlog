+++
title = 'Leetcode_460_LFU'
date = 2022-05-02T19:02:45+08:00
draft = false
+++

[LeetCode LFU](https://leetcode.com/problems/lfu-cache/)

这道题要求实现一个LFU cache, 数据的类型是int.

LFU, 即least frequently used, 当容量满时删除掉使用频率最低的一项. 不同于LRU, least recently used, 当容量满时删除掉最近没有使用的一项.

### 第一版
因为LFU是<key, value>类型的, 所以实际存储的数据结构肯定是个map; 其次, 因为要记录各项的使用个数, 所以还要每一项添加一个connt值. 代码如下所示:

```cpp
class LFUCache {
public:
    LFUCache(int capacity) {
        capacity_ = capacity;
    }
    
    int get(int key) {
        auto it = table_.find(key);
        if (it != table_.end())
        {
            active(key);
            it->second.cnt++;
            return it->second.value;
        }
        else
        {
            return -1;
        }
    }
    
    void put(int key, int value) {
        auto it = table_.find(key);
        
        if (it != table_.end())
        {
            active(key);
            it->second.cnt++;
            it->second.value = value;
            
            active(key);
        }
        else
        {
            if (table_.size() >= capacity_)
            {
                evict();
            }
            
            if (table_.size() < capacity_)
            {
                table_.insert({key,{value, 1}});
                active(key);
            }
        }
    }
private:
    void active(int key)
    {
        auto it = find(nodeList_.begin(), nodeList_.end(), key);
        
        if (it != nodeList_.end())
        {
            nodeList_.remove(*it);
        }
        
        nodeList_.push_front(key);
    }
    
    void evict()
    {
        uint32_t minCnt = UINT_MAX;
        std::vector<int> minKeys;
        for (auto& item : table_)
        {
            if (minCnt > item.second.cnt)
            {
                minCnt = item.second.cnt;
                minKeys.clear();
                minKeys.push_back(item.first);
            }
            else if (minCnt == item.second.cnt)
            {
                minKeys.push_back(item.first);
            }
        }
        
        auto it = find_if(nodeList_.crbegin(), nodeList_.crend(),  [&minKeys] (auto item) {
            return minKeys.end() != find(minKeys.begin(), minKeys.end(), item);
        });
        
        table_.erase(*it);
        nodeList_.remove(*it);
    }
    
    struct ValueWithCnt
    {
        int value;
        int cnt;
    };
    
    map<int, ValueWithCnt> table_;
    list<int> nodeList_;
    int capacity_;
};
```

很遗憾的是,这一版因为效率比较低, 运行时限没有达到leetCode的要求, 没有被接受. 其实是因为 evict时的查找太费时了, 基本上把整个map和list都遍历了一遍, 最应该优化的就是这里, minKey不应通过查找出来, 应该再查询和插入的过程中进行缓存.

### 第二版

用一个map来缓存每项的使用次数, key是使用次数, value是符合当前使用次数的所有项目的list, 每次push_back, 删除的时候优先用第一个. 代码如下:

```cpp
class LFUCache {
public:
    LFUCache(int capacity) {
        capacity_ = capacity;
        minFreq_ = 0;
    }
    
    int get(int key) {
        if (capacity_ == 0)
        {
            return -1;
        }
        
        auto it = table_.find(key);
        if (it != table_.end())
        {
            active(key, it->second.cnt);
                        
            it->second.cnt++;
            return it->second.value;
        }
        else
        {
            return -1;
        }
    }
    
    void put(int key, int value) {
        if (capacity_ == 0)
        {
            return;
        }
        
        auto it = table_.find(key);
        
        if (it != table_.end())
        {
            active(key, it->second.cnt);
            
            it->second.cnt++;
            it->second.value = value;
        }
        else
        {
            if (table_.size() >= capacity_)
            {
                evict();
            }

            active(key, 0);
            table_.insert({key,{value, 1}});
        }
        
        
    }
private:
    void active(int key, int cnt)
    {
        if (cnt != 0)
        {
            freqMap_[cnt].remove(key);
        }
        freqMap_[cnt+1].push_back(key);
        
        updateMinFreq();
    }
    
    void evict()
    {
        int minKey = freqMap_[minFreq_].front();
        freqMap_[minFreq_].pop_front();
        
        table_.erase(minKey);
        
        updateMinFreq();
    }
    
    void updateMinFreq()
    {
        for (int i = 0; i < freqMap_.size(); i++)
        {
            if (freqMap_[i].size() != 0)
            {
                minFreq_ = i;
                break;
            }
        }
    }
    
    struct ValueWithCnt
    {
        int value;
        int cnt;
    };
    
    map<int, ValueWithCnt> table_;
    map<int, list<int>> freqMap_;
    int capacity_;
    int minFreq_;
};
```

这一版的提交结果: Runtime: 937 ms, Memory Usage: 181.7 MB.
效率超过了28%的提交, 内存超过了78%的.

### 第三版
和其他人提交的结果比较后发现，与别人的差异主要在 list的remove操作上，list的remove操作是遍历整个list，删除所有等于该item的项目，其实是个O(n)的操作，而其他人提交的都是缓存了一个当前key对应的frequent list的iterator，这样就不用重新去查找，代码如下：

```cpp
class LFUCache {
public:
    LFUCache(int capacity) {
        capacity_ = capacity;
        minFreq_ = 0;
    }
    
    int get(int key) {
        if (capacity_ == 0)
        {
            return -1;
        }
        
        auto it = table_.find(key);
        if (it != table_.end())
        {                     
            active(key, it->second.cnt);
            
            updateMinFreq();
            it->second.cnt++;
            return it->second.value;
        }
        else
        {
            return -1;
        }
    }
    
    void put(int key, int value) {
        if (capacity_ == 0)
        {
            return;
        }
        
        auto it = table_.find(key);
        
        if (it != table_.end())
        {
            active(key, it->second.cnt);
            
            it->second.cnt++;
            it->second.value = value;
            
            updateMinFreq();
        }
        else
        {
            if (table_.size() >= capacity_)
            {
                evict();
            }

            freqMap_[1].push_back(key);
            table_.insert({key,{value, 1}});
            iterMap_[key] = --freqMap_[1].end();
            minFreq_ = 1;
        }
    }
private:
    void active(int key, int cnt)
    {
        freqMap_[cnt].erase(iterMap_[key]);
        freqMap_[cnt+1].push_back(key);
        iterMap_[key] = --freqMap_[cnt+1].end();
    }
    
    void evict()
    {
        int minKey = freqMap_[minFreq_].front();
        freqMap_[minFreq_].pop_front();
        
        table_.erase(minKey);
        iterMap_.erase(minKey);
    }
    
    void updateMinFreq()
    {
        if (freqMap_[minFreq_].size() == 0)
        {
            minFreq_++;
        }
    }
    
    struct ValueWithCnt
    {
        int value;
        int cnt;
    };
    
    unordered_map<int, ValueWithCnt> table_;
    unordered_map<int, list<int>> freqMap_;
    unordered_map<int, list<int>::iterator> iterMap_;
    int capacity_;
    int minFreq_;
};
```

这一版提交的结果
Runtime: 617 ms, faster than 69.09%， Memory Usage: 186.8 MB, less than 43.59%

做这个题花了比较长时间，其实是因为没有意识到lsit remove操作的耗时， 虽然只是一行代码， 但其背后的实现方式还是需要仔细思考的.