我肤浅的认为想把算法题又快又对的做出来，靠的是满满的套路感。

我决定以后把收集到的套路感都在这记下来。

做一条油腻的算法狗。

[TOC]



### 套路1：回溯 dfs做排列组合

```
private void dfsCore2(List<List<Integer>> res, int curIdx, List<Integer> tmp, int[] nums) {
		// 注意退出条件的不同
		if (curIdx == nums.length) {
			res.add(new ArrayList<Integer>(tmp));
			return;
		}
		// 不加入nums[curIdx]
		dfsCore2(res, curIdx + 1, tmp, nums);
		// 加入nums[curIdx]
		tmp.add(nums[curIdx]);
		dfsCore2(res, curIdx + 1, tmp, nums);
		// 回溯
		tmp.remove(tmp.size() - 1);

	}
```

```
void backTrackDFS(int[] nums, int index, List<Integer> list) {

		if (index == nums.length) {
			// return
			if (isDistinct(list) == true) {
				res.add(new ArrayList<Integer>(list));
				// System.out.println("++"+list);
			}
			return;

		}

		backTrackDFS(nums, index + 1, list);
		list.add(nums[index]);
		backTrackDFS(nums, index + 1, list);
		list.remove(list.size() - 1);

	}
```



### 套路2：大问题递归拆成子问题

求解树的题中非常常见

```
private boolean helper(TreeNode p, TreeNode q) {
		// 判断有没有节点为null的情况，一句话搞定
		if (p == null || q == null)
			return p == q;

		return (p.val == q.val) && helper(p.left, q.right) && helper(p.right, q.left);
	}
```



### 套路3：BFS用先进先出的queue

```
public static List<List<Integer>> levelOrder(TreeNode root) {
		Queue<TreeNode> queue = new LinkedList<TreeNode>();// 队列里面存放结点
		List<List<Integer>> result = new ArrayList<List<Integer>>();

		// 如果为空树就直接返回
		if (root == null) {
			return result;
		}

		queue.offer(root);// 根节点先入队
		// 只要队列非空就一直循环;
		while (!queue.isEmpty()) {
			int levelNum = queue.size();// 获取当前层的节点数.
			List<Integer> subList = new LinkedList<>();
			// 遍历当前层结点
			for (int i = 0; i < levelNum; i++) {
				// 队首出队并将value加入子list
				TreeNode node = queue.poll();
				subList.add(node.val);

				// 将非空左右子树加入queue
				if (node.left != null) {// 如果队首的左结点不为空就把左结点入队
					queue.offer(node.left);
				}
				if (node.right != null) {// 如果队首的右结点不为空就把右结点入队
					queue.offer(node.right);
				}
			}
			result.add(subList);// 添加一层
		}

		return result;
	}
```



### 套路4：变来变去都一样的二分

https://www.cnblogs.com/luoxn28/p/5767571.html

二分有各种变种，我们留意到：

- while (left <= right)都一样，
- right = mid - 1;left = mid + 1;也照写
- if (array[mid] >= key) {按照查找条件来
- return条件取反的那一边就行

```
// 查找第一个等于或者大于key的元素
static int findFirstEqualLarger(int[] array, int key) {
    int left = 0;
    int right = array.length - 1;

    // 这里必须是 <=
    while (left <= right) {
        int mid = (left + right) / 2;
        if (array[mid] >= key) {
            right = mid - 1;
        }
        else {
            left = mid + 1;
        }
    }
    return left;
}
```



### 套路5：快排式分治

```
/* 算法导论标准实现版本 */
	public static int qsort_partition(int[] nums, int start, int end)
	{
		int base = nums[end];
		int i = start - 1;  //i下标指向的元素以及之前的元素都小于base
		for(int j = start; j < end; j++)
		{
			if(nums[j] <= base)//小于base，i移动一格
			{
				i++;
				swap(nums,i,j);			
			}
		}
		swap(nums,i+1,end);
		return i+1;
	}
	
```



### 套路6：佛系随缘的DP

贴了代码也画不出瓢。。



### 套路7：冥冥之中靠贪心蒙一把最优

https://leetcode-cn.com/problems/jump-game-ii/description/

给定一个非负整数数组，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

你的目标是使用最少的跳跃次数到达数组的最后一个位置。

输入: [2,3,1,1,4]
输出: 2
解释: 跳到最后一个位置的最小跳跃数是 2。
从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。

```
public int jump_greedy(int[] nums) {

		int stepcnt = 0;
		// for(int i = 0; i < nums.length; i++)
		int cur = 0;
		while (cur < nums.length - 1) {
			stepcnt++;
			int thisfar = nums[cur];
			int maxfar = 0;// 贪心算法 每次选择下一步可达最远的那一格
			int nextstep = -1;
			// int bound = Math.min(nums.length-1, arg1)
			for (int j = 1; j <= thisfar; j++) {

				if (cur + j >= nums.length - 1) {
					return stepcnt;
				}

				if (cur + j + nums[cur + j] > maxfar) {
					maxfar = cur + j + nums[cur + j];
					nextstep = cur + j;
				}

			}
			cur = nextstep;// 贪心算法 每次选择下一步可达最远的那一格
		}

		return stepcnt;
	}
```



### 套路8：HashMap乾坤大挪移

HashMap往往能够四两拨千斤，时间复杂度杀手。

贴了代码也画不出瓢。。



### 套路9：堆--深藏功与名

```
// create a min heap
        PriorityQueue<Pair> queue = new PriorityQueue<Pair>(new Comparator<Pair>(){
            public int compare(Pair a, Pair b){
                return a.count-b.count;
            }
        });
  
        //maintain a heap of size k.
        for(Map.Entry<Integer, Integer> entry: map.entrySet()){
            Pair p = new Pair(entry.getKey(), entry.getValue());
            queue.offer(p);
            if(queue.size()>k){
                queue.poll();
            }
        }
```







