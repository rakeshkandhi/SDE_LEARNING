# Sliding Window Pattern 🪟

## Overview
The Sliding Window technique uses two pointers to create a "window" that slides through the data structure. It's particularly effective for problems involving contiguous subarrays or substrings.

## When to Use
- Finding substrings/subarrays with specific properties
- Maximum/minimum in fixed-size windows
- Problems with contiguous elements
- Optimization problems on sequences

## Pattern Types

### 1. Fixed Window Size
The window size remains constant throughout the algorithm.

```python
def max_sum_subarray(arr, k):
    """Find maximum sum of subarray of size k"""
    if len(arr) < k:
        return -1
    
    # Calculate sum of first window
    window_sum = sum(arr[:k])
    max_sum = window_sum
    
    # Slide the window
    for i in range(k, len(arr)):
        window_sum = window_sum - arr[i-k] + arr[i]
        max_sum = max(max_sum, window_sum)
    
    return max_sum
```

### 2. Variable Window Size
The window expands and contracts based on certain conditions.

```python
def longest_substring_k_distinct(s, k):
    """Find longest substring with at most k distinct characters"""
    if k == 0:
        return 0
    
    window_start = 0
    max_length = 0
    char_frequency = {}
    
    for window_end in range(len(s)):
        # Expand window
        right_char = s[window_end]
        char_frequency[right_char] = char_frequency.get(right_char, 0) + 1
        
        # Contract window if needed
        while len(char_frequency) > k:
            left_char = s[window_start]
            char_frequency[left_char] -= 1
            if char_frequency[left_char] == 0:
                del char_frequency[left_char]
            window_start += 1
        
        max_length = max(max_length, window_end - window_start + 1)
    
    return max_length
```

## Common Problems

### 1. Longest Substring Without Repeating Characters
```python
def length_of_longest_substring(s):
    """Find length of longest substring without repeating characters"""
    window_start = 0
    max_length = 0
    char_index_map = {}
    
    for window_end in range(len(s)):
        right_char = s[window_end]
        
        # If character is already in window, move start
        if right_char in char_index_map:
            window_start = max(window_start, char_index_map[right_char] + 1)
        
        char_index_map[right_char] = window_end
        max_length = max(max_length, window_end - window_start + 1)
    
    return max_length
```

### 2. Minimum Window Substring
```python
def min_window(s, t):
    """Find minimum window substring containing all characters of t"""
    if not s or not t:
        return ""
    
    # Count characters in t
    dict_t = {}
    for char in t:
        dict_t[char] = dict_t.get(char, 0) + 1
    
    required = len(dict_t)
    formed = 0
    window_counts = {}
    
    left = right = 0
    ans = float("inf"), None, None
    
    while right < len(s):
        # Expand window
        char = s[right]
        window_counts[char] = window_counts.get(char, 0) + 1
        
        if char in dict_t and window_counts[char] == dict_t[char]:
            formed += 1
        
        # Contract window
        while left <= right and formed == required:
            char = s[left]
            
            # Update answer if this window is smaller
            if right - left + 1 < ans[0]:
                ans = (right - left + 1, left, right)
            
            window_counts[char] -= 1
            if char in dict_t and window_counts[char] < dict_t[char]:
                formed -= 1
            
            left += 1
        
        right += 1
    
    return "" if ans[0] == float("inf") else s[ans[1]:ans[2] + 1]
```

### 3. Fruits into Baskets (Longest Subarray with 2 Distinct Elements)
```python
def total_fruit(fruits):
    """Maximum fruits you can collect with 2 baskets"""
    window_start = 0
    max_fruits = 0
    fruit_frequency = {}
    
    for window_end in range(len(fruits)):
        # Expand window
        right_fruit = fruits[window_end]
        fruit_frequency[right_fruit] = fruit_frequency.get(right_fruit, 0) + 1
        
        # Contract window if more than 2 types
        while len(fruit_frequency) > 2:
            left_fruit = fruits[window_start]
            fruit_frequency[left_fruit] -= 1
            if fruit_frequency[left_fruit] == 0:
                del fruit_frequency[left_fruit]
            window_start += 1
        
        max_fruits = max(max_fruits, window_end - window_start + 1)
    
    return max_fruits
```

### 4. Permutation in String
```python
def check_inclusion(s1, s2):
    """Check if s2 contains permutation of s1"""
    if len(s1) > len(s2):
        return False
    
    # Count characters in s1
    s1_count = {}
    for char in s1:
        s1_count[char] = s1_count.get(char, 0) + 1
    
    window_size = len(s1)
    window_count = {}
    
    for i in range(len(s2)):
        # Expand window
        right_char = s2[i]
        window_count[right_char] = window_count.get(right_char, 0) + 1
        
        # Contract window if size exceeds
        if i >= window_size:
            left_char = s2[i - window_size]
            window_count[left_char] -= 1
            if window_count[left_char] == 0:
                del window_count[left_char]
        
        # Check if permutation found
        if window_count == s1_count:
            return True
    
    return False
```

## Advanced Techniques

### 1. Sliding Window Maximum
```python
from collections import deque

def max_sliding_window(nums, k):
    """Find maximum in each sliding window of size k"""
    if not nums:
        return []
    
    deq = deque()  # Store indices
    result = []
    
    for i in range(len(nums)):
        # Remove indices outside current window
        while deq and deq[0] <= i - k:
            deq.popleft()
        
        # Remove smaller elements from back
        while deq and nums[deq[-1]] < nums[i]:
            deq.pop()
        
        deq.append(i)
        
        # Add to result if window is complete
        if i >= k - 1:
            result.append(nums[deq[0]])
    
    return result
```

## Time & Space Complexity
- **Time**: O(n) - each element visited at most twice
- **Space**: O(k) where k is the size of the window or distinct elements

## Tips & Tricks
1. **Use HashMap** to track character/element frequencies
2. **Two pointers**: start and end of window
3. **Expand first**, then contract when condition violated
4. **Update result** during expansion or contraction
5. **Handle edge cases**: empty strings, k > length

## Common Patterns
1. **Fixed size window**: Calculate for first window, then slide
2. **Variable size with condition**: Expand until violation, then contract
3. **Find minimum/maximum**: Track best result while sliding
4. **Count problems**: Use frequency maps

## Practice Problems
1. Maximum Average Subarray I
2. Longest Repeating Character Replacement
3. Max Consecutive Ones III
4. Subarrays with K Different Integers
5. Minimum Size Subarray Sum
6. Find All Anagrams in a String
7. Longest Substring with At Most K Distinct Characters
8. Sliding Window Median
9. Substring with Concatenation of All Words
10. Minimum Number of K Consecutive Bit Flips

## Next Pattern
Continue with [Binary Search Pattern](./binary_search.md)
