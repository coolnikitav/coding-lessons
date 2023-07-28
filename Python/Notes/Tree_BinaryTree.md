# Tree Data Structure

Non-linear data structure. Nodes have to parts: data and a link.

Example uses: hierarchical data, files and folders, XML/HTML data

Types of trees: Binary search trees, AVL, Red Black Tree, Trie

### Tree Terminology

Root: top node without a parent

Edge: a link between parent and child

Sibling: children of the same parent

Ancestor: parent, grandparent, great grandparent of a node

Depth of node: a length of the path from root to node

Height of node: a length of the path form the node to the deepest node

Depth of tree: depth of root node (always 0)

Height of tree: height of root node

### Creating a tree:

```Python
class TreeNode:
    def __init__(self, data, children = []):
        self.data = data
        self.children = children

    def __str__(self, level = 0):
        ret = " " * level + str(self.data) + "\n"
        for child in self.children:
            ret += child.__str__(level + 1)
        return ret
    def addChild(self, TreeNode):
        self.children.append(TreeNode)

#example
tree = TreeNode('Drinks', [])
cold = TreeNode('Cold', [])
hot = TreeNode('Hot', [])
tree.addChild(cold)
tree.addChild(hot)
tea = TreeNode('tea', [])
coffee = TreeNode('coffee', [])
cola = TreeNode('cola', [])
fanta = TreeNode('fanta', [])
cold.addChild(cola)
cold.addChild(fanta)
hot.addChild(tea)
hot.addChild(coffee)
print(tree)
```

# Binary Tree

Definition: data structures in which each node has at most tow children, often referred to as the left and right children.

Tree examples: BST, Heap tree, AVL, red black tress, Syntax trees

### Types of binary tree:

Full binary tree: each node has 2 children or no children

Perfect binary tree: each node has 2 children, all leaf nodes are at the same level

Complete binary tree: all levels are filled, except last level, which has all notes to the left as much as possible

Balanced binary tree: binary tree in which the height of the left and right subtree of any node differ by not more than 1.