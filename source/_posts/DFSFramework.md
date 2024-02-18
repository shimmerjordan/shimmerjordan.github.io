---
title: The Framework of Backtracking Algorithm Problems (DFS)
date: 2021-08-07 00:00:12
tags: 
  - Algorithm
  - DFS
  - Backtracking
categories: Algorithm
id: DFSFramework
---

This paper addresses several questions.

What is the backtracking algorithm? What are the techniques for solving problems related to backtracking algorithms? How do I learn backtracking algorithms? Is there a pattern to the backtracking algorithm code?

The backtracking algorithm is actually what we often call the **DFS algorithm**, which is essentially a violent exhaustive **enumeration algorithm**.

<!-- more -->

**Solving a backtracking problem is really a process of traversing a decision tree.** You only need to think about 3 issues.

1. The path: that is, the choices that have already been made.

2. The list of choices: that is, the choices you can currently make.

3. End condition: that is, the condition that you have reached the bottom of the decision tree and can no longer make a choice.

If you don't understand the explanation of these three terms, that's fine, we'll use the classic backtracking algorithm problems of 'full permutation' and 'N queen problem' later to help you understand what they mean, so for now you'll stay with the impression.

# Framework Code

In terms of code, the framework of the backtracking algorithm:

```python
result = []
def backtrack(paths, choice_list):
    if Meeting the ending conditions:
        result.add(path)
        return
    
    for choice in choice_list:
        make choices
        backtrack(paths, choice_list)
        undo choices
```

**The core of this is the recursion inside the for loop, where you 'make a choice' before the recursive call and 'undo the choice' after the recursive call.**

What does it mean to select and deselect, and what is the underlying principle of this framework? Let's go through the problem of 'full permutation' to clear up the confusion and explore the intricacies in detail!

# Ⅰ. Full Permutation

We have done maths problems with permutations in high school and we know that there are $n$ non-repeating numbers and that there are $n!$

PS: **For the sake of simplicity and clarity, we are discussing permutations without repeating numbers.**

So how did we exhaust the full permutations then? If you are given three numbers `[1,2,3]`, you would not exhaust them in a random way.

First fix the first place as `1`, then the second place can be `2`, then the third place can only be `3`; then you can turn the second place into `3`, and the third place can only be `2`; then you can only change the first place into `2`, and then exhaust the last two places ......

In fact, this is the backtracking algorithm, which we use in high school without a teacher, or some students directly draw this backtracking tree as follows.

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/DFSFramework/pic1.png" width='70%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">backtracking tree</div>
</center>

Simply traverse the tree from the root and **record the numbers on the path**, which are in fact all the full permutations. We might **call this tree the 'decision tree' of the backtracking algorithm.**

**Why do you call this a decision tree? Because you are actually making a decision at each node.** Let's say you are standing on the red node in the diagram below:

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/DFSFramework/pic2.png" width='70%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">backtracking tree</div>
</center>

You are now making a decision, either to choose the `1` branch or to choose the `3` branch. Why can you only choose between `1` and `3`? Because the branch `2` is behind you, a choice you have made before, and full permutation does not allow for repeated use of numbers.

**Now you can answer the first few terms-: `[2]` is the 'path', which records the choices you have already made; `[1,3]` is the 'choice list', which indicates the choices you can currently make; and the 'end condition' is the traversal to the bottom of the tree, in this case when the choice list is empty.**

If you understand these terms, you can **use the 'path' and 'choice' lists as attributes of each node in the decision tree**, the following diagram lists the attributes of several nodes:

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/DFSFramework/pic3.png" width='70%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">attributes of several nodes</div>
</center>

**The `backtrack` function we define is actually like a pointer that wanders around the tree, while maintaining the correct properties of each node, and whenever it reaches the bottom of the tree, its 'path' is a full alignment.**

Taking this a step further, how do you traverse a tree? This shouldn't be too difficult. **In fact, all kinds of search problems are in fact tree traversal problems**, and the framework for traversing a multinomial tree is as follows.

```java
void traverse(TreeNode root) {
    for (TreeNode child : root.childern)
        // Operations required for pre-order traversal
        traverse(child);
        // Operations required for post-order traversal
}
```

And the so-called pre-order traversal and post-order traversal, they're just two very useful points in time, as you'll see if I draw you a diagram:

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/DFSFramework/pic4.png" width='70%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">different traversal ways</div>
</center>

**Pre-order traversal code is executed at the point in time before entering a node, and post-order traversal code is executed at the point in time after leaving a node.**

To recall what we said earlier, 'path' and 'selection' are properties of each node. And a function needs to properly maintain the properties of a node as it travels through the tree, it has to do something at these two particular times.

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/DFSFramework/pic5.png" width='70%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">maintain the properties</div>
</center>

Now, do you understand this core framework of the backtracking algorithm?

```python
for choice in choice_list:
    # make a choice
    # remove this choice from choice_list
    path.add(choice)
    backtrack(path, choice_list)
    # Undo a choice
    path.remove(choice)
    # add this choice into choice_list
```

We can get the correct list of choices and paths for each node by simply **making the choices before the recursion and undoing the choices just made after the recursion.**

Below, look directly at the full alignment code:

```java
List<List<Integer>> res = new LinkedList<>();

/* main function: Enter a set of non-repeating numbers and return their full alignment */
List<List<Integer>> permute(int[] nums) {
    // record「path」
    LinkedList<Integer> track = new LinkedList<>();
    backtrack(nums, track);
    return res;
}

// path：recorded in track
// Select list: those elements of nums that do not exist in track
// End condition: all elements in nums are present in track
void backtrack(int[] nums, LinkedList<Integer> track) {
    // Trigger end conditions
    if (track.size() == nums.length) {
        res.add(new LinkedList(track));
        return;
    }
    
    for (int i = 0; i < nums.length; i++) {
        // Excluding unlawful options
        if (track.contains(nums[i]))
            continue;
        // Make a choice
        track.add(nums[i]);
        // Go to the next level of the decision tree
        backtrack(nums, track);
        // Undo choice
        track.removeLast();
    }
}
```

Instead of explicitly recording the 「selection list」, we have derived the current selection list from `nums` and `track`.

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/DFSFramework/pic6.png" width='70%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">derived the current selection list</div>
</center>

At this point, we have explained the underlying principle of the backtracking algorithm in detail through the full permutation problem. Of course, this algorithm is not very efficient in solving full permutations, as it requires an $O(N)$ time complexity to use the `contains` method on a linked table. There are better ways to achieve this by swapping elements, but they are harder to understand, so I won't write about them here, so you can search for them yourself if you are interested.

However, it is important to note that no matter how it is optimised, it fits into the backtracking framework and the time complexity cannot be lower than $O(N!)$ because exhausting the whole decision tree is unavoidable. **This is a feature of backtracking algorithms; unlike dynamic programming where there are overlapping subproblems that can be optimised, backtracking algorithms are purely violent exhaustive enumeration and the complexity is generally very high.**

Once the full permutation problem is understood, it is straightforward to apply the backtracking algorithm framework, and the following is a brief look at the N queen problem.

# Ⅱ. N queen problem

This is a classic problem, to explain it simply: you are given an $N \times N$ board, and you are asked to place $N$ queens so that they cannot attack each other.

PS: Queens can attack any unit in the same row, column, top-left, bottom-left, top-right and bottom-four directions.

This problem is essentially like the full permutation problem, where each level of the decision tree represents each row on the board; each node can make the choice of placing a queen in any column of that row.

Apply code framework directly :

```java
vector<vector<string>> res;

/* Enter the board edge length 'n' and return all legal placements */
vector<vector<string>> solveNQueens(int n) {
    // '.' means empty, 'Q' means queen, initializing empty board
    vector<string> board(n, string(n, '.'));
    backtrack(board, 0);
    return res;
}

// Path: all rows in the 'board' less than 'row' have successfully placed queens
// selection list: all columns in row 'row' are choices for placing the queen
// End condition: 'row' exceeds the last row of 'board'
void backtrack(vector<string>& board, int row) {
    // Trigger end conditions
    if (row == board.size()) {
        res.push_back(board);
        return;
    }
    
    int n = board[row].size();
    for (int col = 0; col < n; col++) {
        // Exclusion of unlawful options
        if (!isValid(board, row, col)) 
            continue;
        // Make a choice
        board[row][col] = 'Q';
        // Go to the next line of decision making
        backtrack(board, row + 1);
        // Undo choice
        board[row][col] = '.';
    }
}
```

The main code in this section is actually similar to the full alignment problem, and the implementation of the `sValid` function is very simple:

```java
/* Is it possible to place queens in 'board[row][col]'  */
bool isValid(vector<string>& board, int row, int col) {
    int n = board.size();
    // Check if the columns have conflicting queens
    for (int i = 0; i < n; i++) {
        if (board[i][col] == 'Q')
            return false;
    }
    // Check the top right for conflicting queens
    for (int i = row - 1, j = col + 1; 
            i >= 0 && j < n; i--, j++) {
        if (board[i][j] == 'Q')
            return false;
    }
    // Check the top left for conflicting queens
    for (int i = row - 1, j = col - 1;
            i >= 0 && j >= 0; i--, j--) {
        if (board[i][j] == 'Q')
            return false;
    }
    return true;
}
```

The `backtrack` function is still like a pointer that wanders through the decision tree, with `row` and `col` representing the positions traversed by the function, and the `isValid` function pruning out the unqualified cases:

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/DFSFramework/pic7.png" width='70%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">pruning out the unqualified cases</div>
</center>

If you were given such a large piece of solution code straight away, it might be confusing. But now that you understand the framework of the backtracking algorithm, what's so hard to understand? It's just a matter of changing the way you make choices and eliminating illegitimate choices, and once you have the framework in mind, you're left with a small problem.

When `N = 8`, it's the eight queens problem. The mathematician Gauss has never been able to count the number of possible ways to place the eight queens problem in his lifetime, but our algorithm can work out all the possible outcomes in just one second.

But it's not really Gauss's fault. The complexity of the problem is very high indeed, and looking at our decision tree, despite the `isValid` function pruning, the worst time complexity is still $O(N^{N+1})$ and cannot be optimised. If `N = 10`, the computation is already time-consuming.

**There are times when we do not want to get all the legal answers, but only one answer.** For example, in the algorithm for solving Sudoku, the complexity of finding all solutions is too high, and finding just one solution is fine.

In fact it is particularly simple, with a slight modification to the code of the backtracking algorithm.

```java
// The function returns `true` when it finds an answer
bool backtrack(vector<string>& board, int row) {
    // Trigger end conditions
    if (row == board.size()) {
        res.push_back(board);
        return true;
    }
    ...
    for (int col = 0; col < n; col++) {
        ...
        board[row][col] = 'Q';

        if (backtrack(board, row + 1))
            return true;
        
        board[row][col] = '.';
    }

    return false;
}
```

With this modification, any subsequent recursive exhaustion of the `for` loop will be blocked as soon as an answer is found. Perhaps you could write an algorithm for solving Sudoku with a slight modification to the code framework of the N Queen problem?

# Ⅲ. Conclusion

The backtracking algorithm is a multinomial tree traversal problem, the key is to do some operations at the location of the preorder traversal and postorder traversal, the algorithm framework is as follows:

```python
def backtrack(...):
    for choice in choice_list:
        # make choices
        backtrack(...)
        # Undo choices
```

**When writing the `backtrack` function, you need to maintain the 'path' travelled and the 'list of choices' that you can currently make, and when the 'end condition' is triggered, the 'path' is recorded in the result set.**

If you think about it, isn't the backtracking algorithm a bit like dynamic programming? As we have emphasized many times in the dynamic programming series, the three points that need to be clarified in dynamic programming are "state", "choice" and "base case", does this not correspond to the "path" travelled, the current "choice list" and the "end condition"?

In a way, the violent solution phase of dynamic programming is a backtracking algorithm. It's just that some problems have the nature of overlapping subproblems, and can be optimized with dp tables or memoization to prune the recursive tree significantly, which becomes dynamic programming. In today's two problems, however, there are no overlapping subproblems, which means that the problem is a backtracking algorithm, and a very high complexity is inevitable.
