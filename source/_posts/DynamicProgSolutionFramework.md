---
title: The Framework of Solving Dynamic Programming Problems
date: 2021-07-31 18:38:12
tags: 
  - Dynamic Programming
  - Algorithm
id: DPFramwork
---

Dynamic Programming is a problem for many people, but it is also one of the most skillful and interesting strategies. This article gives a partial overview of these problems, plus a framework for solving dynamic programming.

**This paper addresses several questions.**

What is dynamic programming? 

What are the techniques for solving dynamic programming problems?

How can I learn dynamic programming?

<!-- more-->

# What is dynamic programming?

First of all, the general form of a dynamic programming problem is **to find the optimal value**. Dynamic programming is actually an **optimisation method** in operations research, but it is more commonly used in computer problems, such as asking you to find the longest increasing subsequence, the minimum edit distance, and so on.

Since it can be concluded into solving optimal problems, the core of dynamic programming is **exhaustive enumeration**, in which case we are required to  exhaust all possible answers and then find the best value among them.

To begin with, the exhaustion of dynamic programming is a bit special because these problems have '**overlapping sub-problems'**, which could be extremely inefficient if they were  violently exhausted, so **'memos' or 'DP tables'** are needed to optimise the exhaustion process and avoid unnecessary computations.

Moreover, dynamic programming problems **must have an 'optimal sub-structure'** in order to get to the optimal value of the original problem through the optimal value of the sub-problem.

In addition, although the core idea of dynamic programming is to exhaustively find the best value, the problem can be ever-changing. It is not an easy task to exhaust all feasible solutions. Only by listing the **correct 'state transition equations'** can the exhaustion be correctly exhausted. .

The above-mentioned overlapping sub-problems, optimal sub-structures, and state transition equations are the three elements of dynamic programming. The specific meaning will be explained in detail with examples, but in the actual algorithm problem, **it is the most difficult part to write the state transition equation**. This is why many friends find the problem of dynamic programming difficult. Let me provide a thinking framework that I have researched to help you think about the state transition equation:

**Clear base case -> clear 'state' -> clear 'choice' -> define the meaning of dp array/function.**

Follow the above routine, the final result can be set in this framework:

```python
# initialize base case
dp[0][0][...] = base
# State transition
for state1 in all_values_in_state1:
    for state2 in all_values_in_state2:
        for ...
            dp[state1][state2][...] = optimise(choice1，choice2...)
```

The following is a detailed explanation of the fundamentals of dynamic programming through the Fibonacci series problem and the rounding up of change problem. The former is mainly to give you an idea of what an overlapping subproblem is (the Fibonacci sequence does not have an optimum, so it is not strictly speaking a dynamic programming problem), and the latter is mainly cited to focus on how to list the state transfer equations.

# Ⅰ. Fibonacci sequence

Please don't mind the simplicity of this example. **Only a simple example will allow you to concentrate fully on the general ideas and techniques behind the algorithm without being baffled by the obscure details.** 

## 1. Violent recursion

The mathematical form of the Fibonacci sequence is recursive, and is written in code like this：

```java
int fib(int N) {
    if (N == 1 || N == 2) return 1;
    return fib(N - 1) + fib(N - 2);
}
```

We also know that this is simple and easy to understand, but it is very inefficient. Assuming n = 20, draw the recursive tree:

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/DynamicProgSolutionFramework/pic1.png" width='80%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">Recursive tree</div>
</center>

PS: Whenever you encounter a problem that requires recursion, it is a good idea to draw a recursive tree, as this will help you enormously in analysing the complexity of the algorithm and finding the reasons for its inefficiency.

How do I understand this recursive tree? It means that if I want to compute the original problem `f(20)`, I have to first compute the subproblems `f(19)` and `f(18)`, then to compute `f(19)`, I have to first compute the subproblems `f(18)` and `f(17)`, and so on. Finally, when I encounter `f(1)` or `f(2)`, whose result is known, so I can return the result directly, and the recursive tree will not grow further down

**How is the time complexity of a recursive algorithm calculated? It is the number of sub-problems multiplied by the time it takes to solve a sub-problem.**

First, the number of subproblems is calculated, i.e. the total number of nodes in the recursive tree. Obviously the total number of nodes in a binary tree is exponential, so the number of subproblems is $O(2^n)$.

Then the time to solve a subproblem is calculated. In this algorithm, there is no loop and there is only one addition operation, `f(n - 1) + f(n - 2)`, in $O(1)$.

Therefore, the time complexity of this algorithm is the multiplication of the two, i.e. $O(2^n)$, which is at exponential explosion level.

Looking at the recursive tree, it is clear why the algorithm is inefficient: there are a lot of repeated calculations, for example `f(18)` is calculated twice, and you can see that this recursive tree, rooted at `f(18)`, is huge, and calculating it more than once can take a huge amount of time. What is more, more than one node, `f(18)`, is being repeated, so the algorithm is very inefficient.

This is the first property of the dynamic programming problem: **the overlapping subproblem**. Let's find a way to solve this problem.

## 2. Recursive solution with memo

Once the problem is clear, half of the problem is solved. If the reason for such time-consuming is repeated computation, we can create a 'memo', so that each time we have worked out the answer to a sub-problem, we don't rush to return, but first write it down in the 'memo' and then return.

You can also use a hash table (dictionary), and the idea is the same.

```java
int fib(int N) {
    if (N < 1) return 0;
    // Memo fully initialised to 0
    vector<int> memo(N + 1, 0);
    // Perform recursion with memo
    return helper(memo, N);
}
 
int helper(vector<int>& memo, int n) {
    // base case
    if (n == 1 || n == 2) return 1;
    // Already calculated
    if (memo[n] != 0) return memo[n];
    memo[n] = helper(memo, n - 1) + helper(memo, n - 2);
    return memo[n];
}
```

Now, draw the recursive tree so you know exactly what the 'memo' does:

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/DynamicProgSolutionFramework/pic2.png" width='80%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">Recursive tree</div>
</center>

In fact, the recursive algorithm with 'memoization' transforms a recursive tree with a huge amount of redundancy into a recursive graph without redundancy by 'pruning' it, greatly reducing the number of sub-problems (i.e. nodes in the recursive graph).

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/DynamicProgSolutionFramework/pic3.png" width='70%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">Top-down</div>
</center>

**How is the time complexity of a recursive algorithm calculated? It is actually the number of sub-problems multiplied by the time it takes to solve a sub-problem.**

The number of subproblems, i.e. the total number of nodes in the graph, is `f(1)`, `f(2)`, `f(3)` ... `f(20)`, proportional to the input size `n = 20`, since there is no redundant computation in this algorithm, so the number of subproblems is $O(n)$.

The time to solve a subproblem, ditto above, without any loops, is $O(1)$.

Therefore, the time complexity of this algorithm is $O(n)$. It is a downscaling blow compared to the brute force algorithm.

Up to this point, the recursive solution with memoization has been as efficient as the iterative dynamic programming solution. In fact, this solution is already similar to iterative dynamic programming, except that this method is called 'top-down' and dynamic programming is called 'bottom-up'.

What does **'top-down'** mean? Note that the recursive tree (or graph) we just drew extends from the top down, from a larger original problem, such as `f(20)`, down to the base case of `f(1)` and `f(2)`, and then returns the answer layer by layer.

What do you mean by **'bottom-up'**? On the contrary, we start from the bottom, the simplest and smallest problem, `f(1)` and `f(2)`, and work our way up until we get to the answer we want, `f(20)`. **This is the idea behind dynamic planning, which is why it generally moves away from recursion and instead completes the calculation by loop iteration.**

## 3. Iterative solution with dp arrays

Inspired by the previous step, we can make this 'memo' a separate table, called a **DP table**, on which we can do the 'bottom-up' projection.

```java
int fib(int N) {
    if (N < 1) return 0;
    if (N == 1 || N == 2) return 1;
    vector<int> dp(N + 1, 0);
    // base case
    dp[1] = dp[2] = 1;
    for (int i = 3; i <= N; i++)
        dp[i] = dp[i - 1] + dp[i - 2];
    return dp[N];
}
```

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/DynamicProgSolutionFramework/pic4.png" width='70%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">Process demonstrationn</div>
</center>

The DP table is particularly like the result of the previous 'pruning', only in reverse. In fact, the 'memo' in the recursive solution with memoization ends up being this DP table, so the two solutions are in fact similar and, in most cases, essentially the same in terms of efficiency.

Here, the term **'state transfer equation'** is introduced, which is actually the mathematical form that describes the structure of the problem.
$$
f(n)=
\begin{cases}
1,\quad n=1,2 \\\ 
f(n-1)+f(n-2),\quad n>2
\end{cases}
$$
Why is it called the **'state transfer equation'**? It's just to sound high end. You think of `f(n)` as a state `n`, which is transferred by adding up states `n - 1` and `n - 2`, and that's called a state transfer, and that's it.

You will see that all the operations in the above solutions, such as` return f(n - 1) + f(n - 2)`, `dp[i] = dp[i - 1] + dp[i - 2]`, and the initialisation of memos or DP tables, are all different manifestations of the equation around which the equation revolves. This shows the importance of listing the 'state transfer equation', which is at the core of the problem. And it is easy to see that the state transfer equation actually represents a straightforward violent solution.

**Don't look down on violent solutions. The most difficult part of a dynamic programming problem is to write the violent solution, i.e. the state transfer equation.** As soon as the violent solution is written, the optimisation method is just a memoization or a DP table, and there is no more subtlety.

This example concludes with a detailed optimization. The careful reader will notice that, according to the Fibonacci sequence of state transfer equations, the current state is only related to the previous two states, so it is not necessary to have a DP table that long to store all the states. Therefore, the space complexity can be reduced to $O(1)$ by further optimisation.

```java
int fib(int n) {
    if (n < 1) return 0;
    if (n == 2 || n == 1) 
        return 1;
    int prev = 1, curr = 1;
    for (int i = 3; i <= n; i++) {
        int sum = prev + curr;
        prev = curr;
        curr = sum;
    }
    return curr;
}
```

This technique is known as **'state compression'**. If we find that only a portion of the DP table is needed for each state transfer, then we can try to reduce the size of the DP table by using state compression to record only the necessary data, as in the above example, which is equivalent to reducing the size of the DP table from `n` to `2`. We will see examples of this in subsequent chapters on dynamic programming, which generally compress a two-dimensional DP table into one dimension, i.e., reducing the space complexity from $O(n^2)$ to $O(n)$.

One may ask how another important feature of dynamic programming, 'optimal substructures', is not covered. It will be covered below. The Fibonacci sequence example is not strictly speaking dynamic programming, since it does not involve finding the optimal value. The above is intended to illustrate the elimination of overlapping subproblems, and to demonstrate the step-by-step process of obtaining the optimal solution. Next, look at the second example, the problem of getting change.

# Ⅱ. The problem of getting change

Let's start with the following question: you are given `k` coins of denominations `c1, c2 ... ck`, each with an infinite number of coins. Given an infinite number of each coin, and a total `amount`, you are asked how many coins **at least** you need to make up this amount, and if this is not possible, the algorithm returns -1. The function signature of the algorithm is as follows

```java
// coins are the optional coin denominations, and amount is the target amount
int coinChange(int[] coins, int amount);
```

For instance,  `k = 3` and the denominations are `1`, `2` and `5` and the total `amount = 11`. Then a minimum of `3` coins is needed to come up with `11 = 5 + 5 + 1`.

How do you think the computer should solve this problem? Obviously, by exhausting all the possible ways to make up the coins, and then finding the minimum number of coins needed.

## 1. Violent recursion

Firstly, the problem is a dynamic programming problem because it has an 'optimal sub-structure'. **To be 'optimally sub-structured', the sub-problems must be independent of each other.** What do you mean by mutually independent? You won't want to read a mathematical proof, so I'll use a visual example to explain.

For example, suppose you take an exam and your results in each subject are independent of each other. Your original problem is to get the highest overall score, so your sub-problem is to get the highest score in language and the highest score in maths ...... In order to get the highest score in each subject, you have to get the highest score in the corresponding multiple-choice questions and the highest score in the fill-in-the-blank questions in each subject ...... Of course, the end result is that you get a perfect score in each subject, which is the highest overall score.

The correct result is obtained: the highest total score is the total score. Because this process fits into the optimal sub-structure, the sub-problems "highest in each subject" are independent of each other and do not interfere with each other.

However, if you add the condition that your language and maths scores will constrain each other, a high maths score will lower your language score and vice versa. In that case, obviously the highest total score you can get will not reach the total, and you will get the wrong result along the lines of the one you just did. Because the sub-problems are not independent, the language and mathematics scores cannot be optimal at the same time, so the optimal substructure is broken.

Returning to the problem of rounding up the change, why is it consistent with the optimal sub-structure? For example, if you want to find the minimum number of coins for `amount = 11` (the original problem), and if you know the minimum number of coins to come up with for `amount = 10` (the subproblem), you simply add one to the answer to the subproblem (and choose a coin of denomination `1`) to give you the answer to the original problem. Since there is no limit to the number of coins, **the sub-problems are not interlocked and are independent of each other**.

So, since we know that this is a dynamic programming problem, we have to think about **how to list the correct state transfer equation**?

1. **Determine base case**, which is not difficult. The algorithm returns `0` when the target `amount` is `0`, since it does not need any coins to come up with the target amount.

2. **Determine the 'state', i.e. the variables that will change in the original problem and the sub-problem.** Since the number of coins is infinite and the denomination of the coins is given by the question, only the target amount will keep moving closer to the base case, so the only 'state' is the target `amount`.

3. **Determine the 'choice', that is, the behaviour that causes the 'state' to change.** The reason why the target amount changes is because you are choosing coins, and every time you choose a coin, you are decreasing the target amount. So the face value of all coins is your 'choice'.

4. **Clarify the definition of the `dp` function/array.** We are talking about a top-down solution here, so there will be a recursive `dp` function. Generally speaking, the argument of the function is the amount that will change in the state transfer, that is, the 'state' mentioned above; the return value of the function is the amount that the question asks us to calculate. In this case, there is only one state, the 'target amount', and we are asked to calculate the minimum number of coins needed to come up with the target amount. So we can define the `dp` function like this.

Definition of `dp(n)`: Enter a target amount `n` and return the minimum number of coins needed to reach the target amount `n`.

With these key points clear, the pseudocode for the solution can be written as follows.

```python
# Pseudocode framework
def coinChange(coins: List[int], amount: int):

    # Definition: To come up with the amount n, at least dp(n) coins are needed
    def dp(n):
        # Make choice: Choose the result that requires the fewest coins
        for coin in coins:
            res = min(res, 1 + dp(n - coin))
        return res

    # The final result required by the title is dp(amount)
    return dp(amount)
```

Based on the pseudocode, we add the base case to get the final answer. Obviously when the target amount is `0`, the number of coins required is `0`; when the target amount is less than `0`, there is no solution and the answer is `-1`.

```python
def coinChange(coins: List[int], amount: int):

    def dp(n):
        # base case
        if n == 0: return 0
        if n < 0: return -1
        # to solve the minimal, so initialize to positive infinity
        res = float('INF')
        for coin in coins:
            subproblem = dp(n - coin)
            # Sub-questions no solution, skipped
            if subproblem == -1: continue
            res = min(res, 1 + subproblem)

        return res if res != float('INF') else -1
    
    return dp(amount)
```

At this point, the state transfer equation is in fact complete, and the above algorithm is already a violent solution; the mathematical form of the above code is the state transfer equation.
$$
dp(n)=
\begin{cases}
0, \quad n=0 \\\
-1, \quad n<0 \\\
min\{dp(n-coin)+1|coin \in coins \}, \quad n>0
\end{cases}
$$
At this point, the problem is in fact solved, except for the elimination of the overlapping subproblem, e.g. when `amount = 11`, `coins = {1,2,5}.` We draw a recursive tree to see this situation:

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/DynamicProgSolutionFramework/pic5.png" width='80%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">recursive tree</div>
</center>

**Time complexity analysis of recursive algorithm: total number of subproblems * time per subproblem costs.**

The total number of subproblems is the number of nodes in the recursive tree, which is harder to see and is $O(n^k)$, in short exponential. Each subproblem contains a `for` loop, which has a complexity of $O(k)$. So the total time complexity is $O(k \cdot n^k)$, which is also exponential level.

## 2. Recursion with memo

Similar to the previous example of the Fibonacci series, the subproblem can be eliminated by memoing, with only minor modifications.

```python
def coinChange(coins: List[int], amount: int):
    # Memo
    memo = dict()
    def dp(n):
        # Check memo to avoid extra counting
        if n in memo: return memo[n]
        # base case
        if n == 0: return 0
        if n < 0: return -1
        res = float('INF')
        for coin in coins:
            subproblem = dp(n - coin)
            if subproblem == -1: continue
            res = min(res, 1 + subproblem)
        
        # Write into memo
        memo[n] = res if res != float('INF') else -1
        return memo[n]
    
    return dp(amount)
```

Obviously 'memo' reduces the number of subproblems significantly and eliminates the redundancy of subproblems completely, so the total number of subproblems does not exceed the number of amounts `n`, i.e. the number of subproblems is $O(n)$. The time to process a subproblem remains the same, $O(k)$, so the total time complexity is $O(kn)$.

## 3. Iterative solution of dp arrays

Of course, we can also use the bottom-up **dp table**  to eliminate the overlapping sub-problem, and there is no difference between the "state", "selection" and base case as before. However, the `dp` function is reflected in the function parameters, while the `dp` array is reflected in the array index.

**The definition of the `dp` array: when the target amount is `i`, at least `dp[i]` coins are needed to round up.**

According to the dynamic programming code framework given at the beginning of our article, the solution can be written as follows.

```java
int coinChange(vector<int>& coins, int amount) {
    // The array size is amount + 1 and the initial value is also amount + 1
    vector<int> dp(amount + 1, amount + 1);
    // base case
    dp[0] = 0;
    // The outer for loop is iterating over all values of all states
    for (int i = 0; i < dp.size(); i++) {
        // The inner for loop is finding the minimum value of all choices
        for (int coin : coins) {
            // Sub-questions not solved, skipped
            if (i - coin < 0) continue;
            dp[i] = min(dp[i], 1 + dp[i - coin]);
        }
    }
    return (dp[amount] == amount + 1) ? -1 : dp[amount];
}
```

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/DynamicProgSolutionFramework/pic6.png" width='80%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">Process demonstration</div>
</center>

PS: The reason why the `dp` array is initialized to `amount + 1` is because the number of coins that make up the amount can only be equal to the amount at most (all `1` coins), so initializing to `amount + 1` is equivalent to initializing to positive infinity, making it easier to take the minimum value later.

# Ⅲ. Conclusion

The first Fibonacci sequence problem explains how to optimise the recursive tree by means of a 'memo' or 'dp table' approach, and makes it clear that these two approaches are essentially the same, just top-down and bottom-up.

The second problem of scraping together change shows how to process a 'state transfer equation', and once the violent recursive solution is written through the state transfer equation, all that remains is to optimise the recursive tree and eliminate overlapping sub-problems.

If you don't know much about dynamic programming, I applaud you for reading this and believe you have mastered the design of this algorithm.

**There is no special technique for a computer to solve a problem. Its only solution is to exhaust all possibilities.** Algorithm design is simply a matter of thinking **'how to exhaust'** and then pursuing **'how to exhaust intelligently'**.

Listing the dynamic transfer equations is a solution to the problem of "how to exhaust". It is difficult because many of them require recursive implementation, and because some of the problems themselves have complex solution spaces that are not so easy to exhaust completely.
