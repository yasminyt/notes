## 二叉树 binary search tree
+ 二叉树的每个节点最多只能有两个分支
+ 对于同一个父节点下的左右分支节点，左节点的数值 <= 父节点的数值 <= 右节点的数值（如下图所示）
  （图）
+ 二叉树在几种常见操作（如查找、插入和删除）的平均情况下提供对数时间，因而二叉树是非常常见且有用的数据结构

## 定义二叉树节点结构
  ```javascript
  function Node(value) {
    this.value = value
    this.left  = null   // 左节点
    this.right = null   // 右节点
  }
  ```
## 定义二叉树
+ 一棵二叉树应该具有以下属性和方法
  - 根节点 `root`
  - 查找最小值：`findMin()`
  - 查找最大值：`findMax()`
  - 添加新节点：`add()`
  - 查找某个元素是否存在：`isPresent()`
+ 定义二叉树结构
  ```javascript
  function BinarySearchTree() {
    this.root = null
  }
  ```
+ 查找最小值：`findMin()`、最大值：`findMax()`
  ```javascript
  this.findMin = function() {
    if (this.root) {
      let currNode = this.root
      while(currNode.left)
        currNode = currNode.left
      return currNode.value
    }
    return null
  }
  this.findMax = function() {
    if (this.root) {
      let currNode = this.root
      while(currNode.right)
        currNode = currNode.right
      return currNode.value
    }
    return null
  }
  ```
+ 添加新节点：`add()`
  ```javascript
  this.add = function (value) {
    const node = new Node(value)
    if (this.root) {
        return addNode(this.root, node);
        // 通过递归的方式遍历添加
        function addNode(currNode, newNode) {
            // 两个节点相同
            if (value == currNode.value)
                return null
            else if (value > currNode.value) {
                if (currNode.right)
                    return addNode(currNode.right, newNode)
                else {
                    currNode.right = newNode
                    return undefined
                }
            }
            else {
                if (currNode.left)
                    return addNode(currNode.left, newNode)
                else {
                    currNode.left = newNode
                    return undefined
                }
            }
        }
    } else {
        this.root = node
        return undefined
    }
  }
  ```
  + 查找某个元素是否存在：`isPresent()`
    ```javascript
    this.isPresent = function (value) {
      if (this.root) {
          return search(this.root)
          function search(currNode) {
              if (value == currNode.value)
                  return true
              else if (value < currNode.value) {
                  if (currNode.left)
                      return search(currNode.left)
                  else
                      return false
              }
              else {
                  if (currNode.right) 
                      return search(currNode.right)
                  else
                      return false
              }
          }
      }
      else
          return false
    }
    ```
  + 查找二叉树的最小、最大高度：`findMinHeight()`、`findMaxHeight()`
    - 二叉树的高度表示：任意一个叶子节点到根节点的距离（叶子节点表示的是没有子节点的节点）
    - 对于一棵**平衡树**，最小高度和最大高度之间的差值最多为1，这意味着，所有的叶子节点要么都在同一个level上，要么它们之间的高度差最多为1
    ```javascript

    ```

