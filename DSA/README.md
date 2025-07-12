# Data Structures & Algorithms (DSA) Learning Guide 📊

A comprehensive guide to mastering essential DSA patterns and techniques for coding interviews and competitive programming.

## 📚 Table of Contents

### Pattern Categories
1. [Linear Data Structure Patterns](#linear-patterns)
   - [Two Pointers](./patterns/two_pointers.md)
   - [Sliding Window](./patterns/sliding_window.md)
   - [Binary Search](./patterns/binary_search.md)

2. [Non-Linear Data Structure Patterns](#non-linear-patterns)
   - [Breadth-First Search (BFS)](./patterns/bfs.md)
   - [Depth-First Search (DFS)](./patterns/dfs.md)
   - [Backtracking](./patterns/backtracking.md)

3. [Advanced Patterns](#advanced-patterns)
   - [Heaps & Priority Queues](./patterns/heaps.md)
   - [Dynamic Programming](./patterns/dynamic_programming.md)

### Practice Resources
4. [Code Examples](./examples/)
5. [Practice Problems](./practice/)
6. [Interview Preparation](./practice/interview_prep.md)

## 🎯 Learning Paths

### Beginner Path (4-6 weeks)
**Goal**: Master fundamental patterns and solve 50+ problems

**Week 1-2: Linear Patterns**
- Day 1-3: [Two Pointers](./patterns/two_pointers.md) (5 problems/day)
- Day 4-7: [Sliding Window](./patterns/sliding_window.md) (4 problems/day)
- Day 8-10: [Binary Search](./patterns/binary_search.md) (3 problems/day)
- Day 11-14: Mixed practice and review

**Week 3-4: Tree/Graph Patterns**
- Day 1-5: [BFS](./patterns/bfs.md) (3 problems/day)
- Day 6-10: [DFS](./patterns/dfs.md) (3 problems/day)
- Day 11-14: [Backtracking](./patterns/backtracking.md) (2 problems/day)

**Week 5-6: Advanced Topics**
- Day 1-7: [Heaps](./patterns/heaps.md) (2 problems/day)
- Day 8-14: [Dynamic Programming](./patterns/dynamic_programming.md) (2 problems/day)

### Intermediate Path (3-4 weeks)
**Goal**: Optimize solutions and handle complex variations

**Week 1**: Advanced Two Pointers and Sliding Window
**Week 2**: Complex Tree/Graph problems
**Week 3**: Advanced DP patterns
**Week 4**: Mock interviews and optimization

### Advanced Path (2-3 weeks)
**Goal**: Master all patterns and solve hard problems efficiently

**Week 1**: Hard problems across all patterns
**Week 2**: System design + algorithms
**Week 3**: Competition-level problems

## 📊 Pattern Overview

### Linear Patterns

#### Two Pointers 🎯
**Time**: O(n) | **Space**: O(1)
- **Use Cases**: Pair finding, palindromes, cycle detection
- **Key Insight**: Reduce O(n²) to O(n) with smart pointer movement
- **Start Here**: [Two Pointers Guide](./patterns/two_pointers.md)

#### Sliding Window 🪟
**Time**: O(n) | **Space**: O(k)
- **Use Cases**: Subarray/substring problems, optimization
- **Key Insight**: Maintain window state while sliding
- **Start Here**: [Sliding Window Guide](./patterns/sliding_window.md)

#### Binary Search 🔍
**Time**: O(log n) | **Space**: O(1)
- **Use Cases**: Sorted arrays, optimization problems
- **Key Insight**: Works on any monotonic function
- **Start Here**: [Binary Search Guide](./patterns/binary_search.md)

### Non-Linear Patterns

#### Breadth-First Search (BFS) 🌊
**Time**: O(V + E) | **Space**: O(V)
- **Use Cases**: Level-order traversal, shortest path
- **Key Insight**: Explore neighbors before going deeper
- **Start Here**: [BFS Guide](./patterns/bfs.md)

#### Depth-First Search (DFS) 🏔️
**Time**: O(V + E) | **Space**: O(V)
- **Use Cases**: Path finding, tree traversal
- **Key Insight**: Go deep before exploring alternatives
- **Start Here**: [DFS Guide](./patterns/dfs.md)

#### Backtracking 🔄
**Time**: O(N!) worst case | **Space**: O(N)
- **Use Cases**: Combinatorial problems, constraint satisfaction
- **Key Insight**: Build solutions incrementally with undo
- **Start Here**: [Backtracking Guide](./patterns/backtracking.md)

### Advanced Patterns

#### Heaps & Priority Queues ⛰️
**Time**: O(log n) operations | **Space**: O(n)
- **Use Cases**: Top K problems, scheduling
- **Key Insight**: Efficient min/max operations
- **Start Here**: [Heaps Guide](./patterns/heaps.md)

#### Dynamic Programming 🧩
**Time**: O(n²) typical | **Space**: O(n) or O(n²)
- **Use Cases**: Optimization, counting problems
- **Key Insight**: Optimal substructure + overlapping subproblems
- **Start Here**: [DP Guide](./patterns/dynamic_programming.md)

## 🔗 Pattern Relationships

```
Two Pointers ←→ Sliding Window (both use two pointers)
     ↓                ↓
Binary Search ←→ Divide & Conquer
     ↓                ↓
    BFS ←→ DFS (both graph traversal)
     ↓     ↓
Backtracking ←→ Dynamic Programming (both solve subproblems)
     ↓                ↓
   Heaps ←→ Priority-based algorithms
```

## 📈 Difficulty Progression

### Easy (Foundation Building)
- Two Pointers: Two Sum, Valid Palindrome
- Sliding Window: Maximum Sum Subarray
- Binary Search: Search Insert Position
- BFS: Binary Tree Level Order
- DFS: Maximum Depth of Binary Tree

### Medium (Pattern Mastery)
- Two Pointers: 3Sum, Container With Most Water
- Sliding Window: Longest Substring Without Repeating
- Binary Search: Search in Rotated Sorted Array
- BFS: Word Ladder
- DFS: Number of Islands
- Backtracking: Permutations
- Heaps: Top K Frequent Elements
- DP: Coin Change

### Hard (Advanced Applications)
- Two Pointers: Trapping Rain Water
- Sliding Window: Minimum Window Substring
- Binary Search: Median of Two Sorted Arrays
- BFS: Word Ladder II
- DFS: Serialize and Deserialize Binary Tree
- Backtracking: N-Queens
- Heaps: Merge K Sorted Lists
- DP: Edit Distance

## 🎯 Problem-Solving Framework

### 1. Pattern Recognition (30 seconds)
- Identify data structure type
- Look for pattern keywords
- Consider constraints and requirements

### 2. Approach Selection (2 minutes)
- Choose appropriate pattern
- Consider time/space complexity
- Think about edge cases

### 3. Implementation (15-20 minutes)
- Write clean, readable code
- Handle edge cases
- Test with examples

### 4. Optimization (5 minutes)
- Review time/space complexity
- Consider alternative approaches
- Optimize if necessary

## 📚 Study Resources

### Online Platforms
- **LeetCode**: Pattern-based problem sets
- **HackerRank**: Structured learning paths
- **CodeSignal**: Interview practice
- **Pramp**: Mock interviews

### Books
- "Cracking the Coding Interview" by Gayle McDowell
- "Elements of Programming Interviews" by Aziz, Lee, Prakash
- "Algorithm Design Manual" by Steven Skiena
- "Introduction to Algorithms" by CLRS

### Video Courses
- "Grokking the Coding Interview" (Educative)
- "Master the Coding Interview" (Udemy)
- "Algorithms Specialization" (Coursera)

## 🏆 Practice Schedule

### Daily Practice (Recommended)
- **Morning (30 min)**: Learn new pattern/concept
- **Afternoon (45 min)**: Solve 2-3 problems
- **Evening (15 min)**: Review and optimize solutions

### Weekly Goals
- **Week 1**: Master one linear pattern
- **Week 2**: Master one non-linear pattern
- **Week 3**: Combine patterns and solve mixed problems
- **Week 4**: Focus on weak areas and optimization

### Monthly Milestones
- **Month 1**: Complete all basic patterns
- **Month 2**: Solve 100+ problems across patterns
- **Month 3**: Handle medium/hard problems confidently
- **Month 4**: Ready for technical interviews

## 🔧 Tools and Setup

### Development Environment
- **IDE**: VS Code, IntelliJ, or preferred editor
- **Languages**: Python, Java, C++, or JavaScript
- **Debugging**: Learn to use debugger effectively
- **Testing**: Write test cases for solutions

### Time Tracking
- **Pomodoro Technique**: 25-minute focused sessions
- **Problem Timer**: Track solving time
- **Progress Journal**: Note learnings and improvements

## 🚀 Getting Started

1. **Choose Your Path**: Select beginner, intermediate, or advanced
2. **Set Up Environment**: Install necessary tools and create workspace
3. **Start with Basics**: Begin with [Two Pointers](./patterns/two_pointers.md)
4. **Practice Daily**: Consistency is key to mastery
5. **Track Progress**: Use the checklists and milestones
6. **Join Community**: Engage with other learners

---

**Ready to begin?** Start with [Two Pointers Pattern](./patterns/two_pointers.md) or explore the [main DSA guide](../DSA.md) for detailed explanations!
