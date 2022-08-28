---
title: 数据结构与算法 排序算法
tags: algorithm
---



# 常用的排序算法

今天被米哈游面试的算法题难住了，利用快速排序解决从无序队列中找出的第k小的值，然而调了一个多小时都没有调出来。所以本篇对常用的排序算法进行归纳总结，熟悉一下经典排序的思想与代码。

<!--more-->


算法概况如下图所示:

![](/My_Assets/sort.svg)

以下算法皆是将无序数组排序为升序数组，实现语言为C++。


### 1 冒泡排序

+ 算法原理：

    1. 比较相邻的元素。如果第一个比第二个大，就交换他们两个。

    2. 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。

    3. 针对所有的元素重复以上的步骤，除了最后一个。

    4. 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

    动图如下：

    ![](/My_Assets/bubbleSort.gif)

+ 代码实现

    ```cpp
    void bubbleSort(vector<int> & nums)
    {
        int N = nums.size();
        for(int i = 0 ; i < N - 1 ; i++ )
        {
            // 外层循环控制次数，总共比较N - 1 趟
            // 每趟可以确定一个数的位置
            for(int j = 0; j < N - 1 - i ; j++ )
            {
                // 内层循环控制每趟比较的元素
                // 第i趟循环可以确定后i个元素
                // 故j每趟循环范围为 [0, N - 1 - i]
                if( nums[j] > nums[j+1] )
                    swap(nums[j], nums[j+1]);
            }
        }
    }
    ```

### 2 选择排序

+ 算法原理：

    1. 首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置。

    2. 再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。

    3. 重复第二步，直到所有元素均排序完毕。

    动图如下：

    ![]( /My_Assets/selectionSort.gif )

+ 代码实现：

    ```cpp
    void selectionSort(vector<int> &nums)
    {
        int min_i; // 保存未排序序列的最小值的下标
        for (int i = 0; i < nums.size(); i++)
        {
            // 外层循环控制获取第 i 小的值的下标
            min_i = i; // 将min_i重置为当前未排序序列的第一个下标
            for (int j = i; j < nums.size(); j++)
            {
                // 获取未排序序列的最小值下标
                if (nums[j] < nums[min_i])
                    min_i = j;
            }
            swap(nums[i], nums[min_i]);
        } 
    }
    ```

### 3 插入排序

+ 算法原理：

    插入排序是一种最简单直观的排序算法，它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。

    具体步骤：

    1. 将第一待排序序列第一个元素看做一个有序序列，把第二个元素到最后一个元素当成是未排序序列。

    2. 从头到尾依次扫描未排序序列，将扫描到的每个元素插入有序序列的适当位置。（如果待插入的元素与有序序列中的某个元素相等，则将待插入元素插入到相等元素的后面。）

    3. 重复上述步骤直至完成排序。

    动图如下：

    ![](/My_Assets/insertionSort.gif)

+ 代码实现

    ```cpp
    void insertionSort(vector<int> & nums)
    {
        int j = 0, tmp;
        for( int i = 0 ; i < nums.size() ; i++ )
        {
            // 外层循坏控制目前是确定第i个元素的位置
            j = i;  // 保存第 i 个 元素的值
            tmp = nums[i];
            while( j >= 1 && tmp < nums[j - 1] )
            {
                // 当前一个位置大于待排序元素时，后移
                nums[j ] = nums[j - 1];
                j--;
            }
            // 找到正确的位置
            nums[j] = tmp;    
        }
    }
    ```

### 4 希尔排序

+ 算法原理：

    希尔排序，也称递减增量排序算法，是插入排序的一种更高效的改进版本。但希尔排序是非稳定排序算法。

    希尔排序是基于插入排序的以下两点性质而提出改进方法的：

    插入排序在对几乎已经排好序的数据操作时，效率高，即可以达到线性排序的效率。但插入排序一般来说是低效的，因为插入排序每次只能将数据移动一位；

    希尔排序的基本思想是：先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，待整个序列中的记录"基本有序"时，再对全体记录进行依次直接插入排序。

    步骤如下：

    1. 选择一个增量序列 t1，t2，……，tk，其中 ti > tj, tk = 1；

    2. 按增量序列个数 k，对序列进行 k 趟排序；

    3. 每趟排序，根据对应的增量 ti，将待排序列分割成若干长度为 m 的子序列，分别对各子表进行直接插入排序。仅增量因子为 1 时，整个序列作为一个表来处理，表长度即为整个序列的长度。

    动图如下：

    ![](/My_Assets/shellSort.gif#pic_center)

+ 代码实现：

    ```cpp
    void shellSort(vector<int> &nums)
    {
        int gap = 1;
        //构建增量数列，保证gap可以为1
        while (gap < nums.size() / 3)
            gap = gap * 3 + 1;

        while (gap >= 1)
        {
            for (int i = 0; i < nums.size(); i += gap)
            {
                // 满足条件 j >= gap ，等号是为了保证 0 号元素得到处理
                for (int j = i; j >= gap && nums[j] < nums[j - gap]; j -= gap)
                    swap(nums[j], nums[j - gap]);
            }
            gap = gap / 3;
        }
    }
    ```

### 5 归并排序

+ 算法原理：
    
    归并排序（Merge sort）是建立在归并操作上的一种有效的排序算法。该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。

    作为一种典型的分而治之思想的算法应用，归并排序的实现由两种方法：

    + 自上而下的递归（所有递归的方法都可以用迭代重写，所以就有了第 2 种方法）；

    + 自下而上的迭代；

    算法步骤如下：

    1. 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列；

    2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置；

    3. 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置；

    4. 重复步骤 3 直到某一指针达到序列尾；

    5. 将另一序列剩下的所有元素直接复制到合并序列尾。

    动图如下：

    ![](/My_Assets/mergeSort.gif)

+ 代码实现：

    ```cpp
    void merge( vector<int> & nums, int left, int right )
    {
        int mid = (left  + right) / 2; 
        //这里计算mid的方式一定与mergeSort保持一致
        //或者从mergeSort直接传过来也可以
        int i = left, j = mid + 1, index = left ;
        vector<int> tmp(nums.begin(), nums.end());
        while (i <= mid && j <= right)
        {
            if( tmp[i] < tmp[j] ) 
                nums[index++] = tmp[i++];
            else
                nums[index++] = tmp[j++]; 
        }

        while( i <= mid ) nums[index++] = tmp[i++];
        while( j <= right ) nums[index++] = tmp[j++]; 
    }

    void mergeSort(vector<int> & nums, int left, int right)
    {
        if( left >= right ) return;
        int mid = (left  + right) / 2;
        
        mergeSort(nums, left, mid );
        mergeSort(nums, mid + 1, right );
        merge(nums, left, right);
    }
    ```

### 6 快速排序

+ 算法原理：

    快速排序是由东尼·霍尔所发展的一种排序算法。在平均状况下，排序 n 个项目要 Ο(nlogn) 次比较。在最坏状况下则需要 Ο(n2) 次比较，但这种状况并不常见。事实上，快速排序通常明显比其他 Ο(nlogn) 算法更快，因为它的内部循环（inner loop）可以在大部分的架构上很有效率地被实现出来。

    快速排序使用分治法（Divide and conquer）策略来把一个串行（list）分为两个子串行（sub-lists）。

    快速排序又是一种分而治之思想在排序算法上的典型应用。本质上来看，快速排序应该算是在冒泡排序基础上的递归分治法。

    算法步骤如下：

    1. 从数列中挑出一个元素，称为 "基准"（pivot）;

    2. 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partition）操作；

    3. 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序；

    动图如下：

    ![](/My_Assets/quickSort.gif)

+ 代码实现：

    ```cpp
    int partation_1(vector<int> & nums, int left, int right)
    {
        int pivot = left ; // 以最左序号作为基准值
        int index = pivot + 1;  // 待处理的位置序号
        for(int i = index; i <= right ; i++ )
        {
            // 遍历整个序列
            if( nums[i] < nums[pivot] )
            {
                // 当当前值小于基准值，与index位置值交换
                // index 代表当前要找的小于基准值的值
                // 所处的位置下标
                swap(nums[i], nums[index]);
                index++;
            }
        }
        // 循环结束，index代表第一个大于等于基准的值的下标
        // 与基准进行交换，index - 1即基准值在排序后的序列的下标
        swap(nums[pivot], nums[index - 1]);
        return index - 1; //返回基准下标
    }


    int partation_2(vector<int> & nums, int left, int right)
    {
        int pivot = nums[left] ; // 保存基准值
        int i = left, j = right;
        while( i < j)
        {
            // 这里 一定要先循环 右面
            // 因为是将 最左的基准位置作为 可覆盖位置
            // 也就是 初始 i 的位置
            // i ,  j 永远代表可以覆盖的位置
            // 而初始化的 j 一定不是 可以覆盖的位置
            // 但 i 一定是， 故先循环右侧，找到j用来覆盖 i
            while (i < j && nums[j] > pivot) j--;
            nums[i] = nums[j];

            while( i < j && nums[i] < pivot) i++;
            nums[j] = nums[i];
            
        }
        // 循环退出， i 与 j 一定相等，也就是 基准的下标
        nums[i] = pivot;

        return i; //返回基准下标
    }

    void quickSort(vector<int> & nums, int left, int right)
    {
        if( left >= right )
            return ;
        int pivot = partation_1(nums, left, right);
        // int pivot = partation_2(nums, left, right);
        quickSort(nums, left, pivot - 1);
        quickSort(nums, pivot + 1 , right);   
    }

    ```

### 7 堆排序

+ 算法原理：

    堆排序（Heapsort）是指利用堆这种数据结构所设计的一种排序算法。堆积是一个近似完全二叉树的结构，并同时满足堆积的性质：即子结点的键值或索引总是小于（或者大于）它的父节点。堆排序可以说是一种利用堆的概念来排序的选择排序。分为两种方法：

    + 大顶堆：每个节点的值都大于或等于其子节点的值，在堆排序算法中用于升序排列；

    + 小顶堆：每个节点的值都小于或等于其子节点的值，在堆排序算法中用于降序排列；

    算法步骤：

    1. 创建一个堆 H[0……n-1]；

    2. 把堆首（最大值）和堆尾互换；

    3. 重新调整堆；

    4. 重复步骤 2，直到堆的尺寸为 1。

    动图如下：

    ![](/My_Assets/heapSort.gif)

+ 代码实现：

    ```cpp
    void adjustHeap(vector<int> & nums, int start, int end)
    {
        // 相当于在start位置新增了一个节点
        // 现在要调整原始堆，使其成为大根堆
        int dad = start;
        int son  = dad * 2 + 1;
        while (son <= end)
        {   
            if( son + 1 <= end && nums[son + 1] >  nums[son] )
                son += 1;
            
            if( nums[dad] >= nums[son] ) 
                break;
            else
            {
                // 此时，父亲节点小于孩子节点，交换
                // 并继续向下调整
                swap(nums[dad], nums[son]);
                dad  = son;
                son  = dad * 2 + 1;
            }
        }
    }

    void heapSort(vector<int> & nums)
    {
        for(int i = nums.size() / 2  - 1 ; i >= 0 ; i--)
        {
            // 从下往上，调整父节点，构建大根堆
            adjustHeap(nums, i, nums.size() - 1);
        }

        for(int i = nums.size() - 1 ; i > 0 ; i--)
        {
            // 大根堆堆顶一定是当前未排序序列的最大值
            // 通过交换到未排序序列最后并入已排序序列中
            swap(nums[i], nums[0]);
            // 交换之后重新构建大根堆
            // 继续找下一个最大元素
            adjustHeap(nums, 0, i - 1);
        }
    }
    ```

### 8 计数排序

+ 算法原理：

    计数排序（counting sort）就是一种牺牲内存空间来换取低时间复杂度的排序算法，同时它也是一种不基于比较的算法。这里的不基于比较指的是数组元素之间不存在比较大小的排序算法，我们知道，用分治法来解决排序问题最快也只能使算法的时间复杂度接近 O(nlogn) ，即**基于比较的时间复杂度存在下界 O(nlogn)** ，而不基于比较的排序算法可以突破这一下界。当输入的元素是 n 个 0 到 k 之间的整数时，它的运行时间是 O(n + k)。

    算法步骤：

    1. 找到序列的最大值 max_v 最小值 min_v

    2. 创建 k = ( max_v - min_v + 1 ) 个桶，保存在最大值最小值之间各个整数的个数。

    3. 遍历各个桶，重构有序序列

    动图如下：

    ![](/My_Assets/countingSort.gif)

+ 代码实现：

    ```cpp
    void countingSort(vector<int> & nums)
    {
        int max_v = nums[0] , min_v = nums[0];
        for(int & num :nums)
        {
            // 获取最大值最小值
            max_v = max(max_v, num);
            min_v = min(min_v, num);
        }
        
        // 创建 max_v - min_v + 1 计数器 
        vector<int> counter ( max_v - min_v + 1, 0 );
        
        for(int & num :nums)
        {
            // 计数
            counter[ num - min_v ]++;
        }

        int index = 0;
        for(int i = 0 ; i < counter.size() ; i++ )
        {
            // 构建排序序列
            for( int j = 0 ; j < counter[i] ; j++ )
                nums[index++] = i + min_v;
        }
    }
    ```

### 9 桶排序

+ 算法原理：

    桶排序（Bucket sort）或所谓的箱排序，是一个排序算法，工作的原理是将数组分到有限数量的桶里。每个桶再个别排序（有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排序），最后依次把各个桶中的记录列出来记得到有序序列。桶排序并不是比较排序，他不受到O(nlogn)下限的影响。

    算法步骤：

    1. 确定桶的个数

    2. 分桶

    3. 每个桶单独排序

    4. 合并排序结果

    5. 得到有序序列

    动图如下：

    ![](/My_Assets/bucketSort.gif)

+ 算法实现：

    ```cpp
    void buckerSort(vector<int> & nums)
    {
        int max_v = nums[0] , min_v = nums[0];
        for(int & num :nums)
        {
            // 获取最大值最小值
            max_v = max(max_v, num);
            min_v = min(min_v, num);
        }
        // 设定桶的容量
        int bucket_size = 3;
        // 创建桶
        int bucket_cnt = ( max_v - min_v ) / bucket_size + 1;
        vector<vector<int>> buckets(bucket_cnt, vector<int>());
        
        // 分桶
        int bucket_id = 0;
        for(int i = 0; i < nums.size() ; i++)
        {
            buckets[ ( nums[i] - min_v ) / bucket_size ].push_back( nums[i] );
        }

        // 每个桶单独排序
        for(int i = 0 ; i < bucket_cnt ; i++)
        {
            bubbleSort( buckets[i] ) ; // 排序算法可以自己选择
        }

        // 合并排序结果
        int index = 0;
        for(int i = 0 ; i < bucket_cnt ; i++)
        {
            // 构建有序序列
            for( int j = 0 ; j < buckets[i].size() ;j++ )
                nums[index++] = buckets[i][j];
        }
    }
    ```

### 10 基数排序

+ 算法原理

    基数排序(radix sort)是一种非比较型整数排序算法，其原理是将整数按位数切割成不同的数字，然后按每个位数分别比较。由于整数也可以表达字符串（比如名字或日期）和特定格式的浮点数，所以基数排序也不是只能使用于整数。

    基于两种不同的排序顺序，我们将基数排序分为

    + LSD（Least significant digital）：排序方式由数值的最右边（低位）开始

    + MSD（Most significant digital）：由数值的最左边（高位）开始。

    注意一点：

    LSD的基数排序适用于位数少的数列，如果位数多的话，使用MSD的效率会比较好。

    MSD的方式由高位数为基底开始进行分配，但在分配之后并不马上合并回一个数组中，而是在每个“桶子”中建立“子桶”，将每个桶子中的数值按照下一数位的值分配到“子桶”中。在进行完最低位数的分配后再合并回单一的数组中。

    下面以LSD为例，算法步骤如下：

    1. 确定数组中的最大元素有几位（MAX）（确定执行的轮数）

    2. 创建0~9个桶（桶的底层是队列），因为所有的数字元素都是由0~9的十个数字组成

    3. 依次判断每个元素的个位，十位至MAX位，存入对应的桶中，出队，存入原数组；直至MAX轮结束输出数组。

    动图如下：

    ![](/My_Assets/radixSort.gif)

+ 代码实现：

    ```cpp
    void radixSort(vector<int> & nums)
    {
        int base = 10, cnt = 1;
        // 获取数据最大位数
        for(int & num : nums)
        {
            if(num / base > 0)
            {
                base *= 10;
                cnt++;
            }
        }
        // 创建桶0-9
        vector<queue<int>> buckets(10, queue<int>());
        base = 1; // 表示要处理哪一位的数字
        while (cnt >= 1)
        {
            for(int & num : nums)
            {
                // 下式 根据base 获取对应位置的数字
                // ( num % ( base * 10 ) / base
                buckets[ ( num % ( base * 10 ) ) / base ].push(num);
            }

            for(int index = 0, i = 0; i < 10 ; i++)
            {
                
                while( !buckets[i].empty() )
                {
                    nums[index++] = buckets[i].front();
                    buckets[i].pop();
                }
            }
            printVec(nums);cout<<endl;
            cnt--;
            base *= 10; // 处理下一位  
        }
    }
    ```

