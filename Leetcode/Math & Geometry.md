# [Happy Number](https://leetcode.com/problems/happy-number/)
## First Attempt
- Tried applying the process recursively, passing original & current
	- returning false if they are equal
	- returning true if `current == 1`
- relied on the assumption that 'loops endlessly' in the description meant it would get back to the original number eventually
	- Either not true or it takes too many iterations to get there
## Optimised
- The trick was that the endless loop might not actually include the starting number, it could be from any iteration of summing the squares
	- So you need to store all previous iterations
	- Hash seems intuitively faster to me, but array actually was
	- Set was also slower somehow, and used more memory
	- Either random variance from small sample size/differences or something about constructing the Set/Hash is more expensive
- Could also do it with a linked list, using tortoise and hare pointers to check if there's a cycle