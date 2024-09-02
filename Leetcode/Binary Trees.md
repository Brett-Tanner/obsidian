# Key Points
- You need to check every node, not from the root, because the tree could be unbalanced
- Height of a null node is -1, height of a single node is 0
	- 0 because there are 0 edges from a single node
	- -1 because you went there despite there being no edge, so you've gone 'under' the tree
- Height of other nodes is 1 + the max of their left and right nodes
- Diameter from a node is `2 + l_height + r_height`
	- Because there is an edge linking to each node, and `height` edges after those nodes
	
# [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/)
## First Attempt
- First thought is to find the height of the left subtree and add it to the height of the right subtree
	- Nope
## Optimised
- Store a max diameter, initialised at 0
	- Because I couldn't use global variables, I passed the max up as well and de-structured it
- For each node, starting from the bottom, calculate the height and diameter
	- If the diameter is > max_diameter, update max_diameter
	- Return the height to be used in calculating the parent node's diameter


# [Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/description/)

## First Attempt
 - Can you just recursively swap the L & R nodes?
	 - Yes, you can
	 - Just return if root is nil
	 - Otherwise save L in temp var, set L to R and R to temp
	 - Make recursive calls with L & R
	 - Return root at the end
- ~~Push all nodes onto the stack so you can start from the leaves, then switch their L & R values~~
	- Turned out starting from leaves wasn't necessary

# [Max Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/description/)

## First Attempt
- Just recursively call the initial function
	- returning 0 if you hit an edge
	- otherwise returning 1 + the max depth of the subtrees

## Optimisation
- The way I did it was recursive DFS
- Can also use iterative DFS or BFS
	- BFS
		- Add root to the queue
		- While there's a node in the queue
			- Store the length of the queue (number of nodes in row)
			- Pop that many nodes from the start of the queue, adding their children to the end
			- Increment level counter by 1 and store the length of the queue again
	- DFS iterative
		- Initialize max_depth as 0
		- Initialize stack with root node and depth of 1 (stack is 2D array)
		- While stack is not empty
			- Pop a node and 
				- If it's depth is > max_depth, max_depth = depth
				- add its children to the stack with the node's depth plus one