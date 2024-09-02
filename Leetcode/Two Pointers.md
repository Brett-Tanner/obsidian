# [Container with Most Water](https://leetcode.com/problems/container-with-most-water/)
# First Attempt
- Iterate over, and for each height calculate the area of the rectangles formed by it and all following heights
	- Update the max when a new one is found
- But too slow
## Optimised
- The trick was to start at either end
- Move the smaller pointer inward each time
	- If they're equal you can do the comparison with their next values
	- But ultimately could loop forever

# [Three Sum](https://leetcode.com/problems/3sum/description/)
## First Attempt
- If you lock a number you end up with 2 sum, so just 2 sum with an extra loop?
	- Array is unsorted, given the number of loops probably makes sense to sort it
	- Extract binary searching for a complement into a function you can call recursively, returning up the 
- Create a hash of indexes keyed by their values
- And an empty array `results`
- For each `n` in `nums`
	- create a local variable `operands = []`
	- add `n`'s index to operands
	- subtract `n` from target then check if the result is a key in the hash
		- If it is, add the key's value to operands (unless present)
		- remember there can be multiple values, if one is present try the next
	- Repeat until operands is 3 elements long or you don't find the key
		- Then empty operands and go next
	- How do I deal with duplicates? Just uniq the results array?
## Optimised
- Sort the array
- Iterate through the sorted array
	- Because the array is sorted;
		- if a `nums[i] == nums[i - 1]` you can skip the second one, you already tried that in the starting position
			- But only if `i > 0`, otherwise you're comparing to `nums[-1]`
		- If the value is positive you can return, because all following numbers are 0 and you can't get to 0 by adding positive numbers
	- Use the two_sum(sorted) solution on the remainder of the array
		- A pointer at each end
		- Add `n` and the values at each pointer
		- If larger than 0 you move right pointer down
		- If smaller than 0 you move left pointer up
		- If they add up to 0 you found a solution
			- Remember you still need to look for others
			- So after adding to the array you keep going
			- So increment L pointer by 1
				- then keep incrementing it while `nums[l] == nums[l -1] && l < r` to avoid duplicates
# [#Two Sum (sorted)](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/description/)
## First Attempt
- Just try all the combinations of numbers.
- Since sorted, you can stop once your sum is > target
- You can also slice out previous values since you're tried them
## Optimised
- Two pointers, O(n)
- Start at either end, if too small move left pointer up, too big move right pointer down
- I think a while loop is better than recursion because you're not (maybe) making copies of the array on each call