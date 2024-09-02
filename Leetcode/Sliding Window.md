# Key Points
- If you're gonna use `r - l` for the size you also need to add 1

# [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/description/)
## First Attempt
- Initialise
	- `l, r, unused, max = 0, 0, k, 0`
- while `l < s.size < max`
	- increment `r` as long as `s[r] == s[r - 1]`
	- increment `r` and decrement `unused` if `unused.positive?` and you haven't hit the end of the string
		- otherwise you'll use your replacements on `nil`
	- else 
		- `max = r - l if r - l > max`
		- increment `l`
		- set `unused = k`
	- This is close but doesn't account for replacing backwards
		- **Backwards wasn't actually the problem, it's that I was always trying the replace the starting letter rather than adjusting based on which occurred the most frequently**
## Optimised
- Initialise 
	- `l, r, max = 0, 0, 0` 
	- `freq = {}`
- `while r < s.size`
	- initialise `freq[s[r]]` as 1 or increment it if it exists
	- if the length of the substring - the highest frequency is less than k
		- set `max` to the max of substring size & current max
	- Otherwise you've run out of replacements
		- so decrement `freq[s[l]]`
		- and increment `l`
	- increment `r`
# [Longest Substring](https://leetcode.com/problems/longest-substring-without-repeating-characters/description/)
## First Attempt
- Misunderstood the question, once they occur once they can't be used again at all, not just straight after
	- So rather than the index-based condition, store previously used chars in a hash
- Initialise
	- `l`, `r` & `max_length` at 0
	- `used` as an empty hash
- While `l < s.size - max_length`
	- if `used[s[r]] || s[r].nil?`
		- get the length of the substring with `r - l`
		- if it's greater than `max_length`, it's the new `max_length`
		- empty `used`
		- increment `l`, and set `r = l`
	- else add the char at `s[r]` to `used` and increment `r`
## Optimised
- The key is to use a Set
- Initialise
	- `l, r, max_length = 0, 0, 0`
	- `set = Set[]`
- While `l < s.size - max_length`
	- `max_length = set.size if s[r].nil? || set.size > max_length`  
		- `s[r].nil?` in the case where you hit the end of the string without increasing `max_length`. Otherwise you'll never get to the end as the while loop stops at `max_length` from the end of the string 
	- go to next iteration and increment `r` if you can add the value at `r` to `set`
		- `set.add?(char)` is useful for this, `?` variant returns nil if it can't add and adds otherwise
	- if you can't add the value at `r` you have a duplicate, so delete the value at `l` and increment it by one until you can add the value at `r` again
# [Purchase Timing](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)
## First Attempt
- Map over the array, returning the maximum profit obtainable if a purchase is made on each day
- Return the maximum of the resulting array
## Optimised
- Two pointers (but both start at beginning), O(n)
- Move right on each iteration, if profit is negative move left to current right for next iteration since you found a new min
- Store the running max profit