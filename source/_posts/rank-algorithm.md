---
title: 10种常用排序算法总结
excerpt: 整理总结常用的十种排序算法
mathjax: true
categories: 算法笔记
tags: 数据结构
keywords: 排序,算法,排序算法,数据结构
date: 2020-12-27 16:08:34
---



# 概述



本文整理了常见的十种排序算法，介绍了其原理。开头先介绍几个相关的概念，然后是几种算法的复杂度以及稳定性的比较。

## 相关概念

* **稳定排序**：如果排序前a在b前面,且a和b相等，排序后a也在b的前面，称为稳定排序。
* **不稳定排序**：和稳定排序相反，如果排序后a在b的后面，则为不稳定排序。
* **原地排序**：排序过程中不申请多余的存储空间，只利用原来存储待排数据的存储空间进行比较和交换的数据排序。
* **非原地排序**：需要利用额外的数组来辅助排序。
* **比较类排序**：通过比较来决定元素间的相对次序，时间复杂度不能突破$O(n\log n)$，因此也被称为非线性时间比较类排序。
* **非比较排序**：不通过比较来决定元素间的相对次序，可以突破基于比较排序的时间下界，以线性时间运行，因此也称为线性时间非比较排序。

> 本文中的**计数排序**、**桶排序**、**基数排序**属于非比较排序，其余的七种排序方法都是比较类排序。



## 算法概述

| 排序算法  | 平均时间复杂度 |    最好情况    |    最坏情况    | 空间复杂度  | 排序方式  | 稳定性 |
| :-------: | :------------: | :------------: | :------------: | :---------: | :-------: | :----: |
| 冒泡排序  |    $O(n^2)$    |     $O(n)$     |    $O(n^2)$    |   $O(1)$    | In-place  |  稳定  |
| 选择排序  |    $O(n^2)$    |    $O(n^2)$    |    $O(n^2)$    |   $O(1)$    | In-place  | 不稳定 |
| 插入排序  |    $O(n^2)$    |     $O(n)$     |    $O(n^2)$    |   $O(1)$    | In-place  |  稳定  |
| 希尔排序* |  $O(n^{1.3})$  |     $O(n)$     |    $O(n^2)$    |   $O(1)$    | In-place  | 不稳定 |
| 归并排序  |  $O(n\log n)$  |  $O(n\log n)$  |  $O(n\log n)$  |   $O(n)$    | Out-place |  稳定  |
| 快速排序  |  $O(n\log n)$  |  $O(n\log n)$  |    $O(n^2)$    | $O(\log n)$ | In-place  | 不稳定 |
|  堆排序   |  $O(n\log n)$  |  $O(n\log n)$  |  $O(n\log n)$  |   $O(1)$    | In-place  | 不稳定 |
| 计数排序  |    $O(n+k)$    |    $O(n+k)$    |    $O(n+k)$    |   $O(k)$    | Out-place |  稳定  |
|  桶排序   |    $O(n+k)$    |     $O(n)$     |    $O(n^2)$    |  $O(n+k)$   | Out-place |  稳定  |
| 基数排序  | $O(n\times k)$ | $O(n\times k)$ | $O(n\times k)$ |  $O(n+k)$   | Out-place |  稳定  |

> 希尔排序算法的时间复杂度需要根据所选的增量决定。



下面从小到大排序为例，介绍各种排序算法的原理及其代码实现，如无特殊说明，算法演示图片来自文末的参考文章。



# 选择排序(Select Sort)

**算法原理：**

* 从整个序列中找到最小的元素，和第一个元素交换位置。
* 从剩下的未排序的元素中找到最小元素，未排序的第一个元素交换位置。
* 重复第二步，不断地将未排序的最小值放到前面，直到所有元素均排列完毕。

**动画演示：**

![选择排序算法图解](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rank1.gif)

**代码实现：**

```java
public class SelectSort{
    public static int[] selectSort(int[] nums){
        int len = nums.length;
        for (int i=0;i<len;i++){
            int min = i; //记录最小值的位置
            for(int j=i+1;j<len;j++){  //从i之后的范围遍历查找
                if (nums[j]<nums[min]) min=j;  //如果找到更小的值，记录索引
            }
            int temp = nums[i];  //交换两元素的位置
            nums[i] = nums[min];
            nums[min] = temp;
        }
        return nums;
    }
}
```

**分析：**

* 时间复杂度：$O(n^2)$

* 空间复杂度：$O(1)$

* 不稳定排序

* 原地排序



# 插入排序(Insert Sort)

**算法原理：**

* 将待排序列第一个元素看作有序序列，从第2个元素开始依次抽取元素。
* 将抽取的元素与其左边第一个元素比较，如果左边第一个元素比它大，则继续与左边第二个元素比较，直到找到比它小的元素(或者已经到达序列头部)，插入这个元素的右边。
* 继续选取第3,4，...n个元素，重复步骤二，并将其插入适当的位置。

**动画演示：**

![插入排序算法图解](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rank2.gif)



**代码实现：**

```java
public class InsertSort{
	public static int[] insertSort(int[] nums){
        if(nums==null||nums.length<2) return nums;//如果数组为空或只有一个元素，直接返回
        int len = nums.length;
        for(int i=1;i<len;i++){  //从下标为1的元素开始遍历，第0个元素默认是有序的
            int cur = nums[i];  //记录当前值
            int index = i-1;
            while(index>=0 && cur<nums[index]){//往左遍历，直到找到一个比cur小的元素
                index -= 1;
            }//要插入的位置是index+1
            for(int j=i;j>index+1;j--){ //将index+1和i之间的元素后移一位
                nums[j] = nums[j-1];
            }
            nums[index+1] = cur;  //将cur插入到index+1的位置
        }
        return nums;
    }
}
```

**分析：**

* 时间复杂度：$O(n^2)$

* 空间复杂度：$O(1)$

* 稳定排序

* 原地排序



# 冒泡排序(Bubble Sort)

**算法原理：**

* 比较相邻的两个元素，如果第一个比第二个大，就交换他们两个。
* 对每一对相邻元素作同样的工作，从开始的一对到最后一对，执行完一遍后，末尾的元素是最大的值。
* 对于最后一个元素外的其他元素，同样执行步骤二的操作。
* 重复以上步骤，直到排序完成。

**动画演示：**

![冒泡排序算法图解](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rank3.gif)



**代码实现：**

```java
public class BubbleSort{
    public static int[] bubbleSort(int[] nums){
        int len = nums.length;
        //每次外层循环一次之后，最后的元素是最大的
        for(int i=0;i<len;i++){//i控制已经排序的个数，长度为len的序列需要循环len次
            for (int j=0;j<len-i-1;j++){ //对未排序的序列进行比较
                if(nums[j]>nums[j+1]){
                    int temp = nums[j];
                    nums[j] = nums[j+1];
                    nums[j+1] = temp;
                }
            }
        }
        return nums;
    }
}
```



上面代码的缺陷是无论序列有没有序，循环次数总是固定的，如果一个序列本来就是从小到大排好序的，按照上述代码两层循环的次数不变。因此可以做进一步的优化，在内层循环中，如果从第一对元素到最后一对元素，都没有发生交换操作，说明序列是有序的，无需对剩余的元素重复比较下去了。

代码如下：

```java
public class BubbleSort{
    public static int[] bubbleSort(int[] nums){
        int len = nums.length;
        //每次外层循环一次之后，最后的元素是最大的
        for(int i=0;i<len;i++){//i控制已经排序的个数，长度为len的序列需要循环len次
            boolean flag = true;  //使用flag记录是否发生交换
            for (int j=0;j<len-i-1;j++){ //对未排序的序列进行比较
                if(nums[j]>nums[j+1]){
                    flag = false;  //如果发生交换，则flag变为false
                    int temp = nums[j];
                    nums[j] = nums[j+1];
                    nums[j+1] = temp;
                }
            }
            //如果一轮循环下来，没有发生元素交换，说明元素有序，之间跳出循环
            if(flag) break;
        }
        return nums;
    }
}
```



**分析：**

* 时间复杂度：$O(n^2)$
* 空间复杂度：$O(1)$
* 稳定排序
* 原地排序



# 希尔排序(Shell Sort)



**希尔排序(Shell Sort)**是简单插入排序的改进版，先比较距离较远的元素，进行宏观调控。希尔排序又叫**缩小增量排序**

**算法原理：**

* 首先将序列分为间隔为gap的几组元素，每组使用插入排序的方法排序。
* 缩小h的值，比如第一次可以是gap=n/2，第二次gap=n/4，...，直到gap=1，重复步骤一的操作。
* gap=1的时候，排序完以后说明序列中任意间隔为1的元素有序，此时的序列就是有序的了。

**动画演示：**（图源水印）

![希尔排序算法图解](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rank4.gif)

**代码实现：**

```java
public class ShellSort{
    public static int[] shellSort(int[] nums){
        int len = nums.length;
        for (int gap=len/2; gap>0; gap=gap/2){//这里的间隔值每次取上一次的二分之一
            for(int i=gap; i<len; i++){ //这里交替对每个分组执行插入排序
                int cur = nums[i]; //记录当前值
                int j = i;
                //对于当前分组的当前值cur，找到其在当前分组中合适的位置，进行插入排序
                while(j>=gap && nums[j-gap]>cur){
                    nums[j] = nums[j-gap]; //将大于cur的值后移
                    j=j-gap;
                }
                nums[j] = cur; //最后将cur插入到合适的位置
            }
        }
        return nums;
    }
}
```

>代码中，是轮流对每个分组进行排序，而不是单独对一个分组插入排序完再排序另一个。

**分析：**

* 时间复杂度：一般为$O(n\log n)$，具体根据gap设置的大小决定。
* 空间复杂度：$O(1)$
* 不稳定排序
* 原地排序



# 归并排序(Merge Sort)



**算法原理：**

* 利用分治的思想。如果数组只有一个元素，直接返回。
* 将数组均分为两部分(分)。
* 对于两部分分别进行归并排序(治)。
* 将两部分排好序的数组合并在一起，类似于两个有序链表合并。

**动画演示：**



![归并算法图解](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rank5.gif)

**代码实现：**

递归版本：

```java
public class MergeSort{
    public static int[] mergeSort(int[] nums,int left,int right){
        if(left==right) return nums; //如果数组只有一个值，直接返回
        int mid = left+(right-left)/2;//这里取中值的方法没有直接求和然后除以2，防止数据溢出
        mergeSort(nums,left,mid);  //递归调用，对左半部分进行排序
        mergeSort(nums,mid+1,right); //递归调用，对右半部分进行排序
        merge(nums,left,mid,right);//对两部分进行合并
        return nums;
    }
    //merge函数，将两个有序数组合并，类似于合并两个有序链表
    public static void merge(int[] nums,int left,int mid,int right){
        //合并两个有序数组，即nums[left,mid]和nums[mid+1,right]
        int[] temp = new int[right-left+1];//临时存放合并后的数组
        int m = left; //第一个数组的指针
        int n = mid+1; //第二个数组的指针
        int index = 0;  //指向临时数组temp
        while(m<=mid && n<=right){
            if(nums[m]<nums[n]) temp[index++] = nums[m++]; //将较小的值加到temp数组中
            else temp[index++] = nums[n++];
        }
        //将没遍历完的数组直接添加到temp数组中
        while(m<=mid) temp[index++] = nums[m++]; 
        while (n<=right) temp[index++] = nums[n++];
        for(int i:temp) nums[left++] = i; //将临时数组的值复制到nums数组中，表示合并结束
    }
}
```

非递归版本：

```java
public class MergeSort{
    public static int[] mergeSort(int[] nums){
        int len = nums.length;
        for(int i=1;i<len;i=i+i){//子数组的大小分别为1,2,4,8,...
            int left = 0;//对数组进行划分
            int mid = left+i-1;
            int right = mid+i;
            while (right<len){//对大小为i的数组两两合并
                merge(nums,left,mid,right); //合并函数和之前一样
                left = right+1;
                mid = left+i-1;
                right = mid+i;
            }
            if(left<len && mid<len){//有一些大小不足i的范围，也要合并起来
                merge(nums,left,mid,len-1);
            }
        }
        return nums;
    }
    //merge函数，将两个有序数组合并，类似于合并两个有序链表
    public static void merge(int[] nums,int left,int mid,int right){
        //合并两个有序数组，即nums[left,mid]和nums[mid+1,right]
        int[] temp = new int[right-left+1];//临时存放合并后的数组
        int m = left; //第一个数组的指针
        int n = mid+1; //第二个数组的指针
        int index = 0;  //指向临时数组temp
        while(m<=mid && n<=right){
            if(nums[m]<nums[n]) temp[index++] = nums[m++]; //将较小的值加到temp数组中
            else temp[index++] = nums[n++];
        }
        //将没遍历完的数组直接添加到temp数组中
        while(m<=mid) temp[index++] = nums[m++]; 
        while (n<=right) temp[index++] = nums[n++];
        for(int i:temp) nums[left++] = i; //将临时数组的值复制到nums数组中，表示合并结束
    }
}
```



**分析：**

* 时间复杂度：$O(n\log n)$
* 空间复杂度：$O(n)$
* 稳定排序
* 非原地排序



# 快速排序(Quick Sort)

快排也是一个分治的算法，每次选择一个元素作为**基准(pivot)**，然后将序列根据pivot分为两部分，pivot此时是有序的，不需要再次移动。然后递归对两部分序列操作。

与归并排序不同的是，快排**不需要额外的辅助空间**和将排序好的**临时序列的值复制到原序列**的时间，因此一般比归并排序要快。

**算法原理：**

* 从序列中挑出一个元素，作为基准(pivot)。选择基准值的方法有多种，可以一直挑选第一个或最后一个元素，也可以随机选择或者取中间值等。
* 重新排列序列，将所有大于pivot的元素放到其后面，所有小于pivot的元素放到其前面。然后pivot就位于中间位置，其已经是有序的，不需要再移动。这个称为**分区(partition**)操作。
* 通过递归，将pivot左右两边的序列同样根据步骤二排列。直到序列的大小为1，递归终止。

> 实际操作的时候可以使用两个指针从两边开始遍历，只有两个指针指向的值同时需要移动的时候，交换两个值，这样可以减少交换次数，节省时间。

**动画演示：**

![快速排序算法图解](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rank6.gif)

**代码实现：**

```java
public class QuickSort{
    public static int[] quickSort(int[] nums,int left,int right){
        if (left>=right) return nums; //当前范围非法或者只有一个元素的时候直接返回
        int mid = partition(nums,left,right); //获取pivot的所在位置
        quickSort(nums,left,mid-1); //以pivot为中心，对左右两边递归
        quickSort(nums,mid+1,right);
        return nums;
    }
    //根据pivot元素将序列分为左右两部分
    public static int partition(int[] nums, int left, int right){
        int pivot = nums[right]; //这里一直选用最右边的元素作为基准元素
        int i = left; //定义i，j两个指针，分别从左右两边往中间遍历
        int j = right-1; //因为最右边的元素已经是基准元素了，所以需要往左一位
        while(true){
            //这里判断条件包括等号，不然会出现两个相同的值重复交换陷入死循环的情况。
            while(i<=j && nums[i]<=pivot) i += 1;//左指针向右遍历，直到找到一个大于pivot的值
            while(i<=j && nums[j]>=pivot) j -= 1;//右指针向左遍历，直到找到一个小于pivot的值
            if(i>=j) break;
            swap(nums,i,j);//交换两个指针指向的元素的位置
        }
        //将序列根据pivot划分为两部分后，将pivot的值放到分界点的位置
        nums[right] = nums[i];//这里和左右指针哪一个指向的值交换都可以
        nums[i] = pivot;
        return i;
    }
    private static void swap(int[] nums, int i, int j) { //用于交换两个值
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```



上述的快排函数是从两端开始遍历，也可以使用两个指针从一端开始遍历：

```java
//使用两个指针，从一端开始遍历
public static int partition(int[] nums, int left, int right) {
        int pivot = nums[right]; //选用最右边的元素作为基准值
        int i = left - 1; //每交换一次，i都+1，表示小于基准值的元素+1
        for (int j = left; j <= right - 1; ++j) {
            if (nums[j] <= pivot) { //从最左边开始遍历，不断地把小于基准值的元素放到左边
                i = i + 1;
                swap(nums, i, j);
            }
        }
        //最后，下标为i和小于i的元素都是小于基准值的元素，将基准值和i+1交换位置
        swap(nums, i + 1, right);
        return i + 1;//交换位置后，i+1表示排序后基准值的位置
    }
```



> 如果序列近乎有序，每次选取第一个数或者最后一个数作为基准的方法时间复杂度会接近$O(n^2)$，因此需要随机选取基准点。
>
> 如果是随机选取基准点，需要在`int pivot = nums[right];`语句前，将随机选取的值放到最右边或最左边，其余代码不变：
>
> ```java
>  int random_index = new Random().nextInt(right-left+1)+left; //随机选取下标
>  swap(nums,right,random_index); //然后将选取的基准值放到最右边
>  int pivot = nums[right]; //将此时最右边的元素作为基准值，也就是随机选取的基准值
> ```
>
> 



**分析：**

* 时间复杂度：$O(n\log n)$
* 空间复杂度：$O(\log n)$
* 非稳定排序
* 原地排序



# 堆排序(Heap Sort)



**堆排序(Heapsort)**是利用**二叉堆**的一种排序方法。二叉堆除了具有完全二叉树的特征外，所有父节点的值都大于或小于它的孩子节点。其中所有父节点的值都大于其孩子节点的堆叫做**最大堆(大顶堆)**，反之称为**最小堆(小顶堆)**。所以说，最大堆的堆顶元素即整个堆的最大值，最小堆的堆顶元素是堆的最小值，堆排序正是利用了二叉堆的这一特征。



> 二叉堆实现是采用**数组**的形式来存储的，假设一个节点的下标为n，则其左孩子和右孩子节点的下标分别为2n+1、2n+2。
>
> 二叉堆每次添加元素都是添加到最后一个位置，即数组最后，然后使用**上浮** 的方法再调整。
>
> 二叉堆删除元素是删除堆顶元素，然后把最后一个位置的元素放到堆顶，采用**下沉** 的方法调整。
>
> 关于二叉堆的**上浮**和**下沉**操作，可以参考[二叉堆](https://mp.weixin.qq.com/s?__biz=Mzg2NzA4MTkxNQ==&mid=2247485231&idx=1&sn=8dfdc04bd209fba3077269faabe7c36f&source=41#wechat_redirect)



**算法原理：**

* 创建一个最大堆，包含n个节点。
* 把堆顶(最大值)和堆最后一个位置元素互换(相当于删除堆顶元素)，这样最大值位于最后面。
* 将堆的大小减1，调整堆，使剩下的n-1个节点保持最大堆的性质。
* 重复步骤2、3，直到堆的大小为1，此时数组中的元素从小到大排列。



**动画演示：**

图片来自 [here](https://commons.wikimedia.org/wiki/File:Heap_sort_example.gif)

![堆排序算法图解](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rank7.GIF)

**代码实现：**

```java
public class HeapSort{
    public static int[] headSort(int[] nums){
        int len = nums.length;
        //构建最大堆
        for(int i = (len-2)/2;i>=0;i--){//i初始化为最后一个元素的父节点
            heapify(nums,i,len);//从最后一个元素的父节点开始调整堆结构，构建最大堆
        }
        //进行堆排序，每次交换堆顶和堆尾两个节点，交换完一次就调整一次结构。
        for(int i = len-1;i>=1;i--){
            swap(nums,0,i);
            heapify(nums,0,i);//这里每次传入i，用来表示堆的个数减一
        }
        return nums;
    }
    public static void heapify(int[] nums,int parent,int len){//调整堆结构
        int largest = parent;
        int left = 2*parent+1;//左孩子下标
        int right = 2*parent+2;//右孩子下标
        if(left<len && nums[left]>nums[largest]) largest=left;//找到较大的那个值
        if(right<len && nums[right]>nums[largest]) largest=right;
        if(largest != parent){//然后交换最大值和父节点，实现父节点的下沉
            swap(nums,parent,largest);
            heapify(nums,largest,len); //一直往下调整，直到父节点是最大值
        }
    }
    public static void swap(int[] nums, int i, int j){//swap函数用于交换数组的两个值
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

上述用于调整堆结构的heapify函数使用了递归的方法，如果不使用递归，可以写成以下形式：

```java
//非递归形式实现调整堆结构
public static void heapify(int[] nums,int parent,int len){
        int temp = nums[parent];//临时保存父节点元素
        int child = 2*parent+1;//用于指向孩子下标
        while(child<len){
            if(child+1<len && nums[child+1]>nums[child]){//寻找左右孩子中较大的值
                child += 1;
            }
            if(nums[child]>temp){//如果孩子节点比父节点要大，则父节点下沉
                swap(nums,parent,child);//父节点下沉，即交换两个节点的值
                parent = child; //然后把孩子节点作为父节点，继续向下调整
                child = 2*parent+1;
            }else break; //如果父节点已经是最大的了，不必下沉，跳出循环。
        }
    }
```



**分析：**

* 时间复杂度：$O(n\log n)$
* 空间复杂度：$O(1)$
* 非稳定排序
* 原地排序



# 计数排序(Counting Sort)



**计数排序(counting sort)**是一种适合最大值和最小值的**差值k**不是很大的排序，其核心在于**将输入的数据值转化为键存储在额外开辟的数组空间**中。计数排序要求输入的数据必须是有确定范围的整数。

**算法原理：**

* 扫描待排序列，找出最大值max和最小值min。
* 开辟新的数组Res，长度为max-min+1。
* 以序列中的值为下标，对应数组B中的值为其在整个序列中出现的次数。
* 遍历数组B，得到每个值的出现次数，并输出。

**动画演示：**

![计数排序算法图解](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rank8.gif)

**代码实现：**

```java
public class CountSort{
    public static int[] countSort(int[] nums){
        if(nums==null || nums.length<2) return nums;
        int len = nums.length;
        int max = nums[0];
        int min = nums[0];
        for (int i:nums){//遍历数组，得到最大值和最小值
            if(i<min) min=i;
            if(i>max) max=i;
        }
        int n = max-min+1; //辅助数组的长度
        int[] temp = new int[n];//建立辅助数组
        for(int i:nums){//遍历原数组，获取每个值的出现次数,在辅助数组中记录
            //下标=真实值-最小值
            temp[i-min] = temp[i-min]+1; //当前值对应于辅助数组中的值+1，表示出现次数+1
        }
        //将辅助数组中的值，根据出现次数恢复到原数组中，完成排序
        int index = 0;//用于指向原数组
        for(int i=0;i<n;i++){ //遍历辅助数组
            for(int j=0;j<temp[i];j++){//辅助数组中的值作为出现次数，将对应的值恢复到原数组
                nums[index++] = i+min;//下标+最小值=真实值
            }
        }
        return nums;
    }
}
```



**分析：**

* 时间复杂度：$O(n+k)$
* 空间复杂度：如果是在原数组的基础上修改，只需要$O(k)$的额外空间
* 稳定排序
* 非原地排序



# 桶排序(Bucket Sort)

**桶排序**是**计数排序**的升级版，它利用一定的函数映射关系，划分出多个桶，将序列中的值放入对应的桶中，然后各个桶内元素再排序(递归或其他排序方法)。桶排序要求数据分布均匀。

> 需要根据待排序列的数据特征，选择合适的分桶方法。

**算法原理：**

* 根据一定方法，划分出n个桶。
* 将数据放入对应的桶中。
* 对每个桶中的数据进行排序，可以递归使用桶排序，或者其他排序方法。
* 将各个桶中的数据拼接，得出结果。



**动画演示：**

图片来自[here](https://blog.csdn.net/donghuabianc/article/details/105252373)

![桶排序算法图解](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rank9.gif)

**代码实现：**

>这里的分桶方法根据序列中数值的范围确定，划分为(max-min)/5 + 1个桶，第i个桶中的元素存放的数值范围是5\*i~5\*i+5-1，即每个桶中存放5个数。

```java
public class BucketSort{
    public static int[] bucketSort(int[] nums){
        if(nums==null || nums.length<2) return nums;
        int len = nums.length;
        int max = nums[0];
        int min = nums[0];
        for (int i:nums){//遍历数组，得到最大值和最小值
            if(i<min) min=i;
            if(i>max) max=i;
        }
        int d = max-min;
        //创建d/5+1个桶，第 i 桶存放5*i ~ 5*i+5-1范围的数
        int bucketNum = d/5 + 1; //桶的数量
        //建立一个链表集合，每个桶都是一个LinkedList类型的链表
        ArrayList<LinkedList<Integer>> bucketList = new ArrayList<>(bucketNum);
        //初始化bucketNum个桶
        for(int i=0;i<bucketNum;i++) bucketList.add(new LinkedList<Integer>());
        //遍历数组，将每个值放入对应的桶中
        for(int i:nums) bucketList.get((i-min)/d).add(i);
        //对每个桶内的元素排序，这里使用系统排序函数，对链表进行排序
        for(int i=0;i<bucketNum;i++) Collections.sort(bucketList.get(i));
        //将各个桶的数据合并，放回原数组
        int index = 0; //用于指向原数组
        for(int i=0;i<bucketNum;i++){//将每个桶中的值依次放回原数组
            for(Integer t: bucketList.get(i)) nums[index++] = t;
        }
        return nums;
    }
}
```



**分析：**

* 时间复杂度：$O(n+k)$
* 空间复杂度：$O(n+k)$
* 稳定排序
* 非原地排序

> k表示桶的个数。桶排序的时间复杂度取决于各个桶内数据排序的时间复杂度，其他部分时间复杂度都是$O(n)$。桶划分的个数越多，桶内的个数越少，单个桶排序用的时间也越少，但空间复杂度也随之增加。



# 基数排序(Radix Sort)

**基数排序(Radix Sort)**是按照低位先排序，然后收集；再按照高位排序，然后再收集；以此类推，直到最高位。如果有的属性有优先级顺序，则先按低优先级排序，再按高优先级排序。最后的结果是高优先级高的在前，如果高优先级相同，则低优先级高的在前。

下面以数字为例：

**算法原理：**

* 取序列中最大的数，并取得位数，将序列中所有数填充至和最大值位数相等，前面填充0。
* 从最低位开始，使用计数排序依次进行一次排序。
* 从最低位排序，一直到最高位排序完成后，序列变为有序序列。

**动画演示：**

![基数排序算法图解](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rank10.gif)

**代码实现：**

```java
public class RaxixSort{
    public static int[] radixSort(int[] nums){
        if(nums==null || nums.length<2) return nums;
        int max = nums[0];
        for (int i:nums) if(i>max) max=i; //求出序列最大值
        for(int exp=1; max/exp>0; exp = exp*10){//从最低位到最高位依次进行计数排序
            countSort(nums,exp);
        }
        return nums;
    }
    public static void countSort(int[] nums,int exp){
        int len = nums.length;
        int[] output = new int[len]; //临时存放根据当前位排序的序列
        int[] temp = new int[10]; //定义辅助数组，用来存放每个基数下的值
        for(int i:nums) temp[(i/exp)%10] += 1;//将每个值在对应数组中的出现次数+1
        //和计数排序不同的是，这里的键是数值的某位的值，所以需要根据下标还原到原来的值
        //将temp中的数值改为累和，表示小于等于当前下标的数值的总数
        for(int i=1;i<10;i++) temp[i] += temp[i-1];
        for(int i=len-1;i>=0;i--){ //一定要从后往前添加，保证temp数组计数正确
            output[temp[(nums[i]/exp)%10]-1] = nums[i];
            temp[(nums[i]/exp)%10]--; //每添加完一个数值，temp对应的下标的次数-1
        }
        for(int i=0;i<len;i++) nums[i] = output[i]; //将每次排序好的序列返回原数组
    }
}
```



**分析：**

* 时间复杂度：$O(n\times k)$
* 空间复杂度：$O(n+k)$
* 稳定排序
* 非原地排序

> 这里的k指的是基的个数，比如0-9一共10个基，k就等于10。



# 总结

实际应用中，需要根据序列的特点选择合适的排序算法，假设一个待排序列总元素个数为n。

1. 当n较大时，应采用时间复杂度为$O(n\log n)$的排序方法：**快速排序、堆排序、归并排序**。

> 快速排序时目前基于比较的内部排序中被认为最好的方法，当待排序的关键字时随机分布时，快排的平均时间最短。
>
> 堆排序适用于内存空间允许且要求稳定性。
>
> 归并排序有一定数量的数据移动。

2. 当n较大，内存空间允许，且要求稳定性，使用**归并排序**。
3. 当n较小，可采用**直接插入排序**或**直接选择排序**。

> 当元素分布有序，直接插入排序将大大减少比较次数和移动纪录的次数。
>
> 当元素分布有序，如果不要求稳定性，使用直接选择排序。

4. 一般不使用或者不直接使用**冒泡排序**。

5. **基数排序**是一种稳定的排序算法，但有一定的局限性：

   a.要求关键字必须可分解。

   b.记录的关键字位数较少，如果密集更好。

   c.如果是数字时，最好是无符号数字，否则将增加相应的映射复杂度，可将正负数分开排序。

# Reference

1. [必学十大经典排序算法](https://mp.weixin.qq.com/s/mq2NSG3xMqIs28nU354TjQ)
2. [十大经典排序算法(动图演示)](https://zhuanlan.zhihu.com/p/73714165)
3. [五分钟学算法](https://www.zhihu.com/question/51337272/answer/572455307)
4. [排序算法的比较和选择依据](https://blog.csdn.net/FISHBALL1/article/details/52425521)

