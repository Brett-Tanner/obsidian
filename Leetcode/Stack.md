# Key Points
- If you're going toward 0 you want truncate, not round/floor/ceil
# [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/)
## First Attempt
- Map over the temperatures with their indexes
	- create a second index pointer, `j`
	- assign result of while `j < temps.size` to `warmer_index`
		- break and set `results[i] = j - i if temps[j] > t`
		- else increment `j`
	- return `warmer_index || 0`
		- As default is 0, and a while which doesn't break returns `nil`
- But this is too slow

## Optimised
- Important to realise the temperatures in the stack will always be in descending order if you pop anything larger than the temp you're adding
- Initialise
	- a results array of the same size as the temperatures array, with 0 as the default value
	- a stack
- Iterate over the temperatures
	- while there's something in the stack and its temperature is less than current temp
		- set `results[i]` to `current_index - i` where `i` is the index of the lower temp value from the stack
		- pop the stack to remove the value you just added to results
	- Then add the current temp and its index to results as an array
		- Or hash if you wanna make it more readable
# [Generate Parentheses](https://leetcode.com/problems/generate-parentheses/description/)

## First Attempt
- Just generate all possible permutations and reject those which aren't valid
- But that would be obscenely slow
## Optimised
- Kinda like a binary tree, you explore to the end of a branch then back off
- Store current iteration in stack, popping the last value every time you finish a recursion so you can try other possibilities from the previous state
- Open must be less than n
- Closed must be less than open
- Finished when open == closed == n
- Recursive function should first check if finished, and add the joined stack to the result array if so
- Check if you can add a leading parentheses, do so and recur with open paren count incremented if so
- Check if you can add a closed paren, do so if possible and call the recursive function with the closed counter updated
- After each recursion resolves you should pop the stack to allow other options for the previous state of the stack to be considered
# [Reverse Polish](https://leetcode.com/problems/evaluate-reverse-polish-notation/)

## First Attempt
- Loop over tokens, pushing them onto the stack if not operators 
- and reducing the stack with the operator if an operator
	- I misunderstood reverse polish, the operator only applies to the last two values on the stack
- If stack is reduced, the new value becomes the sole value in the stack and loop continues
- Return the final value of the stack
## Optimised
- O(n), because you just loop over the operators, adding to the stack once and popping once
	- So 2O(n) but the 2 is irrelevant in Big O
- Got this one on first attempt

# [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)

## First Attempt
- Loop over characters
- If an opening tag, add to the stack and go next
- If a closing tag that closes the opening tag on top of the stack, pop the stack and go next
- Else break
- If the stack is empty & i == string.size, it's a valid string
## Optimised
- Hash, stack, O(n)
- Actually got this one right with the First Attempt attempt