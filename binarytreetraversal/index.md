# 二叉树的遍历

> ###  二叉树是一种重要的树形数据结构, 对其遍历的方式有以下几种:  
> [前序](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)  
> [中序](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)  
> [后序](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)  
> 每一种遍历方式都有***递归*** 与 ***非递归***两种, 本文使用java分别进行实现。  
> **本文阅读完成后请回答一下上图红色数字的含义及其与前、中、后序遍历的关系。**  

> ### 二叉树的数据结构定义如下
```java
class TreeNode {
	int val;
    TreeNode left;
    TreeNode right;
    TreeNode() {}
    TreeNode(int val) { 
        this.val = val; 
    }
    TreeNode(int val, TreeNode left, TreeNode right) {
    	this.val = val;
    	this.left = left;
    	this.right = right;
    }
}
```
> ### 二叉树的遍历
* #### **递归**  

  递归版本简单易懂, 注意递归顺序
1. **前序(PreOrder)**  
```java
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> ans = new ArrayList<Integer>();
        preOrder(root, ans);
        return ans;
    }

    private void preOrder(TreeNode root, List<Integer> ans){
        if(root == null)
            return;
        ans.add(root.val);
        preOrder(root.left, ans);
        preOrder(root.right, ans);
    }
}
```
2. **中序(InOrder)**
```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> ans = new ArrayList<Integer>();
        inOrder(root, ans);
        return ans;
    }
    
    private void inOrder(TreeNode root, List<Integer> ans){
        if(root == null)
            return;
        inOrder(root.left, ans);
        ans.add(root.val);
        inOrder(root.right, ans);
    }
}
```
3. **后序(PostOrder)**
```java
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> ans = new ArrayList<Integer>();
        postOrder(root, ans);
        return ans;
    }
    
    private void postOrder(TreeNode root, List<Integer> ans){
        if(root == null)
            return;
        postOrder(root.left, ans);
        postOrder(root.right, ans);
        ans.add(root.val);
    }
}
```

* #### **非递归**  
1. **前序(PreOrder)**  

&emsp;&emsp;非递归即用迭代的方法模拟递归, 首先思考一下递归时的情况:  
* 当遇到到一个节点时, 首先将其值加入列表(访问)
* 接下来对该节点的左子树进行递归访问
* 左子树访问完成后对其右子树进行递归访问  

&emsp;&emsp;注意到在访问右子树时, 我们是从左子树访问结束回到了当前节点, 因此对已经遇到一次的节点，我们后续仍旧需要该节点的信息(访问右子树)。因此我们需要一个栈来保存已经遇到一次的节点(对应于递归时的递归栈):  
```java
Deque<TreeNode> stack = new ArrayDeque<TreeNode>();
```
*何时需要取出栈中的节点呢?*  
**当前子树访问结束时(即迭代走不下去了, 遇到了null)**  
此时我们需要取出栈顶节点(第二次遇到), 对其右子树进行迭代访问  
*何时整颗树的迭代结束呢?*  
**遇到null且栈为空**  
这时所有节点我们都已经遇到过两次了, 已经没有未访问过的节点了  
最终代码如下:

```java
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> ans = new ArrayList<Integer>();
        Deque<TreeNode> stack = new ArrayDeque<TreeNode>();
        TreeNode node = root;
        while(!stack.isEmpty() || node != null){
            if(node != null){
                ans.add(node.val);  // 第一次遇到时访问
                stack.push(node);
                node = node.left;
            }
            else{
                node = stack.pop();
                node = node.right;
            }
        }
        return ans;
    }
}
```
2. **中序(InOrder)**

&emsp;&emsp;中序和前序的区别在于: **前序在第一次遇到节点时就访问, 中序则为第二次遇到节点时才访问**。因而只要对前序遍历的迭代版本代码稍作改动即可得到中序遍历的迭代版本代码。  
代码如下:

```java
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> ans = new ArrayList<Integer>();
        Deque<TreeNode> stack = new ArrayDeque<TreeNode>();
        TreeNode node = root;
        while(!stack.isEmpty() || node != null){
            if(node != null){
                stack.push(node);
                node = node.left;
            }
            else{
                node = stack.pop();
                ans.add(node.val);  // 第二次遇到时访问
                node = node.right;
            }
        }
        return ans;
    }
}
```
3. **后序(PostOrder)**  

&emsp;&emsp;类比前序和中序遍历, 后序遍历则是第三次遇到该节点时对其进行访问。但是这就出现了一个问题: **如何判断是否是第三次遇到?**  
&emsp;&emsp;在前序和中序遍历中, 第一次遇到节点时(入栈)、第二次遇到节点时(出栈), 但是在后序遍历, 第二次遇到时不能出栈, 第三次遇到才可用出栈, 所以第二次和第三次遇到节点的情况变得难以区分。  
&emsp;&emsp;在递归方法中, 第三次遇到节点是在访问了其右子树**根节点**后, 因此在迭代版本中可以记录前一个访问的节点*pre*, 判断是否是第三次遇到节点即可转换为判断pre是否是其右子树根节点。当访问了一个新的节点时, 将*pre*更新为该节点。另外, 即使*pre*不是其右子树根节点, 若其右子树为空, 我们也可以视作第三次遇到该节点。  
代码如下:  

```java
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> ans = new ArrayList<Integer>();
        if(root == null)
            return ans;
        Deque<TreeNode> stack = new ArrayDeque<TreeNode>();
        TreeNode pre = null, node = root;
        while(!stack.isEmpty() || node != null){
            if(node != null){
                stack.push(node);
                node = node.left;
            }
            else{// 不同之处
                node = stack.pop();
                if(node.right == null || node.right == pre){
                    ans.add(node.val);
                    pre = node;
                    node = null;    // 该节点作为根节点的子树已经访问完成
                }
                else{
                    stack.push(node);
                    node = node.right;
                }
            }
        }
        return ans;
    }
}
```

**注意事项：**

1. pre记录的是前一个***访问***的节点（区别***遇到***）
2. 一定要考虑右子树为空的情况（否则会陷入无限循环：pre一直为左子树，node一直为null）

