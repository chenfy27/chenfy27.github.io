---
layout: post
title: stl常用容器方法
category: cpp
tag:
---

### vector

- push_back(val)
- pop_back()
- empty()
- size()
- front()
- back()
- insert(pos, val)
- erase(pos)
- clear()

### stack

- push(val)
- top()
- pop()
- empty()
- size()

### queue

- push(val)
- front()
- pop()
- empty()
- size()

### deque

- push_back(val)
- push_front(val)
- front()
- back()
- pop_back()
- pop_front()
- empty()
- size()

### priority_queue

- push(val)
- top()
- pop()
- empty()
- size()

### list

- push_back(val)
- push_front(val)
- front()
- back()
- pop_back()
- pop_front()
- insert(pos, val)
- erase(pos)
- clear()
- sort()
- unique()
- reverse()
- remove()
- remove_if()
- merge()

### map/multimap/unordered_map

- insert(make_pair(key, val))
- mp[key] = val (for map and unordered_map)
- erase(key)
- empty()
- size()
- clear()
- find(key)
- count(key)
- lower_bound(key) (for map and multimap)
- upper_bound(key) (for map and multimap)
- equal_range(key)

### set/multiset/unordered_set

- insert(key)
- erase(key)
- empty()
- size()
- clear()
- find(key)
- count(key)
- lower_bound(key) (for set and multiset)
- upper_bound(key) (for set and multiset)
- equal_range(key)
