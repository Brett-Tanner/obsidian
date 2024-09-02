# Key Points
- Use hash to reduce time needed to find k/v pairs

# [Group Anagrams](https://leetcode.com/problems/group-anagrams/description/)
## First Attempt
- Use the array of words to populate a hash with letters as keys and the indexes of words containing those letters as values
- Then create an `untried` array with all indexes of `strs` and while it has elements
	- retrieve the indexes for each letter from the hash and compare them to the previous iteration, carrying forward the union
	- 
- If the union is empty, go next, if it isn't then return the word and the words at the union indexes
- Rather than adding the index to the array, why not the word? That's what I need to return anyway, and easier to compare
	- This is definitely a step in the right direction, lets you check the size is the same
	- Stuck on how to keep duplicates when reducing by intersection though
		- Solved by getting them back by selecting from untried before deleting them from there
- Maybe it's a nested hash? First key is the letter, then the count of that letter in the word, then the words that it applies to
	- once you have that hash, maybe you can just iterate over it to solve? 
	- or, key the hash by the strings, with a nested hash of letter/count k/v pairs (maybe length too)
	- yeah this one with the length, because then if all the keys are equal they're an anagram
	- maybe need a count too, to handle duplicate values
	- then iterate over and they're equal if same length and all counts equal
- So in summary, a hash with structure
```ruby
{
	length: {
		string: {
			count: n,
			letter1_count: n,
			...
			letterN_count: n,
		}
	}
}
```
- Then you iterate within each length
	- add the string to anagrams array (`count` times) because it's an anagram of itself
	- if any of the letter counts are different you go next
	- if they're all the same 
		- it's an anagram 
		- you add it to the anagrams array for this string
		- you remove the word from the length hash
		- and push anagrams array to results
- This was sooooo close to working, I could make it correct if I excluded the occurence count from the comparison of letter counts
	- Necessary because otherwise `["tea", "eat", "tea"]` will not recognise 'eat' and 'tea' as anagrams due to them having a different number of occurences
- But the exclusion made it too slow to finish in the allotted time
- So I added some complexity to the initial construction of the hash by separating each word's `counts` hash into `count` (would have been better to call it `occurences`) and `char_counts`, then just comparing the `char_counts` to check if it's an anagram
- This implementation just snuck in under the time limit
- Which coupled with the huge amount of code I wrote, means there's definitely a better way
	- But hey, at least I solved it myself
## Optimised
- Can group anagrams by sorting them, but O(m * n log n) where n is the average string length and m is the number of strings
- Ahhhh, I was on the right track but didn't realise you can use the hash of counted values as the key
	- That lets you make the value an array of words which match the count, which you can just return
	- Time complexity is O(m * n)
	- He made the key an array of letter counts, but doesn't that take extra unnecessary space/longer to compare?
		- Was because Python has more restrictions on hash keys than Ruby, but in Ruby a hash as a key is fine (with `=>` syntax)
# [Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/description/)
## First Attempt
- Calculate the sum of all values, then map over the input array returning that sum divided by the current element
	- But you can't use division
	- You really can't because they added a 0 in haha
- If you don't care about time complexity, could just map over (with index)
	- Return the results of reducing the array with `:*`, skipping the current index
	- Yeah gets the correct solution but takes too long
- Follow up says to do it in constant extra space, so I assume the initial solution uses another array or hash in some way
	- So how do I do this in O(n) using an array/hash/both?
- Splitting it into start and end arrays before and after the current number also works
	- But is also too slow
## Optimised
- Last idea of splitting into 2 arrays was exactly right, but I was doing a lot of extra multuplication
- Initialise
	- `prefix` array of size `nums.size`, each index will be the product of all values up to and including that index in `nums`
		- You can do this incrementally by multiplying `n` by `prefix[i - 1]`
	- `postfix` array of size `nums.size`, each index will be the product of all the values after and including that index in `nums`
		- You can do this incrementally by multiplying `n` by `postfix[i + 1]`
- Need to iterate over `nums` once, populating `prefix` and `postfix` incrementally from the start and finish respectively
- Once you've filled those two, loop `nums.size` times
	- prefix is `before[i - 1]`
		- Or 1 if `i.zero?`
	- postfix is `postfix[i + 1]`
		- Or 1 if `i == nums.size - 1`
	- `results[i] = prefix * postfix`


# [Top K frequent elements](https://leetcode.com/problems/top-k-frequent-elements/description/)
## First Attempt
- Should just be able to make a hash of the element counts, key is the count & value is value
	- But sorting it is kinda a problem maybe?
	- Also you're not really looking for a specific value, but a specific place in order
- Or make a 2D array of counts & values, then sort by counts and take the first `k`
	- Can speed it up by skipping processed values at cost of memory, store them in a hash?
		- Was actually still pretty slow
	- Or to save memory just count each of the elements returned by a `uniq` call
		- This was also slow
- Maybe put the counts/values in an array already in order using binary search, to avoid the sort? 
	- Then just return the last `k` values
## Optimised
- Use bucket sort to do it in O(n) time
- Create a second array `freqs` with length `nums + 1`
	- use counts of each value as indexes to store an array of values which occur `count` times
- Get the `counts` hash by iterating over nums and incrementing the count value for the current `n` each time
	- This is faster than calling `nums.count(n)`, even with duplicate prevention
- Map `counts` hash to `freqs` array, using count as the index and pushing the value to the values array at that index
- Take the last `k` values from freqs

# [Two Sum](https://leetcode.com/problems/two-sum/description/)
## First Attempt
- Just try all the combinations
- Start by subtracting current value from target, and searching for the remainder
## Optimised
- Hash, O(n)
- Create a hash of value/index pairs by iterating over the array
- For each value, subtract it from the target and check if there's a key for the remainder
	- Check as you create the hash so you don't have to loop again
- If yes, return the value for the remainder key and the index of the current iteration value
- If no, add the current iteration value and its index as a k/v pair

# [Valid Sudoku](https://leetcode.com/problems/valid-sudoku/description/)
## First Attempt
- Initialise
	- `cols`, an array of columns (which are Sets)
	- `row`, an set representing the current row
	- `squares`, an array of squares (which are Sets)
- Iterate over the existing array (of rows) and for each cell
	- next if it's a dot
	- `cols[j].add?(n)`
	- `row.add?(n)`
	- `squares[(i / 3 * 3) + (j / 3)].add?(n)`
		- `/3*3` works because it's integer division
		- so for example `i = 4`, `4 / 3 = 1, 1 * 3 = 3`
	- If any of those return false, return false as the puzzle is invalid
- Nailed it :) Other than taking ages an a lot of googling to figure out the algorithm for finding the current square