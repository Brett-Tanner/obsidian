# Key Points
- If you're in a rotated sorted array, figure out if you're in the left or right sorted portion
	- If middle < right pointer, you're in the right portion
	- If middle > left pointer, you're in the left portion
# [Binary Search](https://leetcode.com/problems/binary-search/description/)
## First Attempt
- Start at middle
	- obv return if value at middle == target
	- if too high move right pointer down to middle
	- if too low move left pointer up to middle
- New middle is the average of L and R, rounded down
- Finished if middle == L, check if value at middle == target & return -1 if not
## Optimised
- Should be O(log n)
- Should have moved L/R pointers to middle +/- 1 instead, while L <= R; so when they haven't crossed

# [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/description/)

## First Attempt
- Could brute force by trying all k values from 1, counting num of hours required for each pile and breaking when > h
- **Watched first bit of vid to get here**
- Min k is 1, max would be max number of bananas in pile
- So binary search that array of possible k values for the min that allows her to finish in h
	- If she can finish in h, try lower half
	- If not, try upper half
- **Big ol memory efficiency problem with this way, probably from creating the array from a range up to big maximums**
- After watching a bit more of the video figured out there's a bsearch method for ranges, so didn't need to make an array from the range. That got it to pass
- After thinking a bit more myself, realised I don't need the range and can just check the middle directly, as indexes are values in a range
## Optimised
- O(log(piles.max ) * piles)
- I got it right myself after some hints about how to get to something binary-searchable

# [Min of Rotated Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/)

## First Attempt
- Maybe b_search each half to find the minimum?
## Optimised
- Yep, that did it, but probably not quite in O(log n) (I think O(2 log n))
- Splitting it into halves was not necessary btw lol, the important thing was that I knew if the middle value was less than the current min I needed to go left.
	- Splitting it into halves didn't actually do anything
- Could also have started with the min as the first element, done <= to handle 1 element arrays
- When you get to a position where L value is < R value you have a sorted array, and can just return L value if less than min

# [Search in Rotated Array](https://leetcode.com/problems/search-in-rotated-sorted-array/description/)

## First Attempt
- Like above, you just need to figure out if you're in the upper or lower half of the sorted array, then once L < R & L < min return the min of L's value and min
- Nope, you're searching for a value not finding the min lol. Read the question.
- Could try finding the min to know the pivot point, then b_search each sorted half for the target
- Could try recursion, search one half then the other
	- But if the pivot isn't the middle does that even do anything?
## Optimised
- O(logn)
- First need to know if you're in the L or R sorted portion of the array
	- If middle value > L value, you're in L sorted portion
	- If middle value < R value, you're in R sorted portion
- Once you know which side you're in
	- If L & target < L value || target > middle value, search right. 
	- If L & target > L value && target < middle value, search left.
	- If R & target > R value || target < middle value, search left.
	- If R & target < R value && target > middle value, search right.

# [Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/description/)

## First Attempt
- Binary search matrix first
	- If last element of middle is < target, move L to middle + 1
	- If first element of middle is > target, move R to middle - 1
	- If target is between first & last elements, binary search the row
	- Continue while L <= R, otherwise return false
- If a possible row is found, binary search it (find_in_row) and return result
	- regular binary search as above, but return true if found/false if not
## Optimised
- O(log n * m)
- Got it first try
- It doesn't need to be a nested loop for the row search, you can just break if target isn't < L or > R
	- Return false if !L <= R because you didn't finish searching but it can't be found
	- Else binary search the row you were on