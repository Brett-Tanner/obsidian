---
title: Backtracking
description: Leetcode notes on backtracking
---

# Key Points

- You wanna reduce the problem down to 2 decisions you can make recursively

# [Combination Sum](https://leetcode.com/problems/combination-sum/description/)

## First Attempt

- I think we iterate over numbers, using `target - n` to get the number `m` you'd sum `n` with to == target, and binary search for `m`. If it's there you add `m` and `n` to the results array
  - But you can have the number itself if it's in the array
    - Could handle with a conditional
  - You can also have more than two numbers to be summed up
    - Would have to recursively call the function with all remaining elements in the array

## Optimised

- Start with empty arrays of results and operands, a pointer to the first element of the candidates array and a sum initialized to 0
- Return if `sum > target` or `i >= candidates.size`
- Push a copy of `operands` to `results` if `sum == target`
- Your two choices are to add the candidate at the current index to the operands array and the sum, or to increment the index and not add the candidate
  - You need to increment the index to avoid duplicates, as order doesn't matter in this problem
- So first you add the candidate at the current index to the operands and call `dfs(i, operands, sum + candidate`
- Then pop the candidate off the operands call `dfs(i + 1, operands, sum)`

# [Subsets](https://leetcode.com/problems/subsets/description/)

## First Attempt

- Subsets are selections of elements (including none) from an array
- Subsets do not care about order, unlike permutations
- Initial idea was nested loops
  - loop over the array to get the first element of a subset
  - add it to the results array
  - then do an inner loop over the array (sliced from i + 1)
  - add the result of inner loop value + previous subset to the results array
  - But this doesn't get the edges, like no combo of last & first element in array
- Looking at the test output made me think of still iterating over the array, but using purely the previous results to add combinations
  - Start with the empty 2D array as you can have a nil subset
  - Then start iterating, for each iteration push the iteration value + each current result to the results
  - That one did it!

## Optimised

- O(n \* 2^n) regardless of implementation
- But I didn't do it using backtracking, which is the point of the question
- The key thing to realise is that for each value on each iteration, you can choose to include or not include it in the subset
- Initialise a results array and a subset array, subset holds the current subset
- Recursively call a function taking the current index, which
  - If `i >= nums.size`, adds a copy of `subset` to `results` and returns
  - Else adds the value at `nums[i]` to `subset` and calls itself with `i + 1`
  - Then pops the last value from `nums` to simulate choosing not to add the value at `nums[i]`, and again calls itself with `i + 1`
- Make sure to `.dup` the subset you push to results, or you'll keep modifying that one the whole time
