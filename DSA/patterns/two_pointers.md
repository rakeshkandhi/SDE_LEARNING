# Two Pointers Pattern 🎯

## Overview
The Two Pointers technique is a powerful approach that reduces time complexity from O(n²) to O(n) for many array and string problems. It uses two pointers to traverse the data structure efficiently.

## When to Use
- Finding pairs in sorted arrays
- Detecting cycles in linked lists
- Palindrome checking
- Merging sorted arrays
- Removing duplicates
- Partitioning arrays

## Pattern Types

### 1. Opposite Direction Pointers
Used when you need to find pairs or check conditions from both ends.

```python
def two_sum_sorted(nums, target):
    """Find two numbers that add up to target in sorted array"""
    left, right = 0, len(nums) - 1
    
    while left < right:
        current_sum = nums[left] + nums[right]
        if current_sum == target:
            return [left, right]
        elif current_sum < target:
            left += 1
        else:
            right -= 1
    
    return [-1, -1]
```

### 2. Same Direction Pointers (Fast & Slow)
Used for cycle detection, finding middle elements, or removing elements.

```python
def find_middle_node(head):
    """Find middle node of linked list"""
    slow = fast = head
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    
    return slow

def has_cycle(head):
    """Detect cycle in linked list"""
    slow = fast = head
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    
    return False
```

## Common Problems

### 1. Valid Palindrome
```python
def is_palindrome(s):
    """Check if string is palindrome (ignoring non-alphanumeric)"""
    left, right = 0, len(s) - 1
    
    while left < right:
        # Skip non-alphanumeric characters
        while left < right and not s[left].isalnum():
            left += 1
        while left < right and not s[right].isalnum():
            right -= 1
        
        # Compare characters
        if s[left].lower() != s[right].lower():
            return False
        
        left += 1
        right -= 1
    
    return True
```

### 2. Container With Most Water
```python
def max_area(height):
    """Find container that holds most water"""
    left, right = 0, len(height) - 1
    max_water = 0
    
    while left < right:
        # Calculate current area
        width = right - left
        current_area = min(height[left], height[right]) * width
        max_water = max(max_water, current_area)
        
        # Move pointer with smaller height
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1
    
    return max_water
```

### 3. Remove Duplicates from Sorted Array
```python
def remove_duplicates(nums):
    """Remove duplicates in-place, return new length"""
    if not nums:
        return 0
    
    slow = 0  # Position for next unique element
    
    for fast in range(1, len(nums)):
        if nums[fast] != nums[slow]:
            slow += 1
            nums[slow] = nums[fast]
    
    return slow + 1
```

### 4. 3Sum Problem
```python
def three_sum(nums):
    """Find all unique triplets that sum to zero"""
    nums.sort()
    result = []
    
    for i in range(len(nums) - 2):
        # Skip duplicates for first number
        if i > 0 and nums[i] == nums[i-1]:
            continue
        
        left, right = i + 1, len(nums) - 1
        
        while left < right:
            current_sum = nums[i] + nums[left] + nums[right]
            
            if current_sum == 0:
                result.append([nums[i], nums[left], nums[right]])
                
                # Skip duplicates
                while left < right and nums[left] == nums[left + 1]:
                    left += 1
                while left < right and nums[right] == nums[right - 1]:
                    right -= 1
                
                left += 1
                right -= 1
            elif current_sum < 0:
                left += 1
            else:
                right -= 1
    
    return result
```

## Time & Space Complexity
- **Time**: O(n) for most problems, O(n²) for nested loops (like 3Sum)
- **Space**: O(1) additional space (in-place operations)

## Tips & Tricks
1. **Sort first** when dealing with sum problems
2. **Skip duplicates** to avoid duplicate results
3. **Use while loops** to handle multiple duplicates
4. **Consider edge cases**: empty arrays, single elements
5. **Fast & slow pointers** are great for linked lists

## Practice Problems
1. Two Sum II - Input array is sorted
2. 3Sum Closest
3. 4Sum
4. Remove Element
5. Move Zeroes
6. Sort Colors (Dutch National Flag)
7. Trapping Rain Water
8. Linked List Cycle II
9. Remove Nth Node From End of List
10. Intersection of Two Linked Lists

## Next Pattern
Continue with [Sliding Window Pattern](./sliding_window.md)
