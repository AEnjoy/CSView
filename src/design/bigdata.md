

### 海量日志数据，提取出某日访问百度次数最多的那个IP？

首先是找到这一天的日志，并且是访问百度的IP从日志中提取出来逐个写入大文件中，如果IP是32位的，最多可能有2^32次方种结果。

采用映射的方法，比如%1000，把大文件依次映射成1000个小文件，找出每个小文件中出现频率最多的IP，可以使用hashMap统计频率，依次找出每个文件中出现频率最大IP，一共1000个。然后在1000个IP中找出频率最大即为所求，可以使用归并或者小顶堆。



### 寻找热门查询中，1000万个查询字符串中最热门的10个查询？要求内存使用不超过1G，每个查询串长度都是1~255字节?

1000万个字符串肯定会有重复，假设不重复字段大约300万，3M * 1K/4 = 0.75G，可以在内存中全部装下，因此采用hashTable，key位字符串，value位为出现的次数，每次读取一个串，如果串在hashTable中就将统计频率加1，否则将value置为1。然后维护一个堆，保存10个元素，找出TopK，遍历300个串的value分别与根元素进行比较，最终时间复杂度为O(N)+M*O(logK)，N为1000万，M为300万，K为堆大小10。



### 有一个1G大小的一个文件，里面每一行是一个词，词的大小不超过16字节，内存限制大小是1M。返回频数最高的100个词？

顺序读取文件，对于每个词采用hash%5000，映射到(x0,x1,……,x4999)，每个文件大约200KB，如果有文件大小超过1K就继续分治，直到所有文件大小都小于1M。对于每个小文件采用hashMap或者前缀树统计相应的词和频率。取出现频率最大的100个词，存入文件可以得到5000个文件，最后把5000个文件进行归并排序。



### 海量数据分布在100台电脑中，想个办法高效统计出这批数据的TOP10？

如果每个元素只出现一次并且出现在某一台机器上，可以在每台电脑上求TOP10，采用最大堆，然后求出每个电脑的TOP10之后将100个电脑的TOP组合起来得到1000条数据，再用大顶堆划分出TOP10。

如果同一元素重复出现在不同电脑中，可以遍历一遍所数据后hash取模，使同一元素只出现在一台电脑上。统计每个元素的TOP10，找出最后的TOP10。

或者直接暴力统计每个电脑各个元素出现的次数，把同一元素在不同电脑上次数相加，找出最终数据的TOP10。



### 有10个文件，每个文件1G，每个文件的每一行存放的都是用户的查询，每个文件的查询都可能重复。要求你按照查询的频度排序？

顺序读取10个文件，按照hash(query)%10的结果写入10个文件中，生成的新文件大小也是1G。找一个内存是2G的机器，依次对用hashMap(query, query_count)来统计每个query出现的次数。然后将排序好的query和count存储到文件中，得到10个已经排序的文件，最后对10个文件进行归并排序。



### 给定a、b两个文件，各存放50亿个url，每个url各占64字节，内存限制是4G，让你找出a、b文件共同的url？

50亿个URL每个URL是64字节，一共占用320亿字节，内存4G装不下，所以采用分治的思想。先遍历a文件，对于每个hash(URL)%1000，分别将1000个URL存储到1000个小文件里面，记为a1、a2、……、a1000，每个文件大约就是300M，遍历文件b采取同样的方法。因为使用的是相同的哈希算法，所以相同的URL在对应文件对里面，这样就会有1000对小文件。不对应的小文件不可能有相同的URL。

然后对每对URL，先把一个文件的URL存储到hashSet中，遍历另一个小文件的每个URL，看是否在hashSet，如果有相同的URL存储到文件就行。

如果允许一定错误率，可以使用布隆过滤器，4亿内存可以保存340亿bit。用一个布隆过滤器映射340亿bit，然后读取另一个文件的URL，检查是否在布隆过滤器里面，快速判断URL是否存在。



### 在2.5亿个整数中找出不重复的整数，注，内存不足以容纳这2.5亿个整数？

**方案1**：采用2比特位的位图，每个数分配2位，00表示这个数不存在，01表示出现一次，10表示出现多次，11无意义，共需要内存2^32*2 = 1G内存，扫描2.5亿个整数判断位图中是否存在，并调换对应的位，查看所有的01位并进行输出即可。

**方案2**：先把2.5亿个小文件进行划分，在小文件中找到不重复整数，并且排序，然后再归并。



### 给40亿个不重复的unsigned int的整数，没排过序的，然后再给一个数，如何快速判断这个数是否在那40亿个数当中？

采用位图，申请512M内存，一位代表一个unsigned int，然后读入40亿数字，设置相应的bit位。读出要查询的数并且判段bit是否为1。



### 100w个数中找出最大的100个数？

**局部淘汰法**：选取前100个元素，并排。依次扫描剩余元素，与排好序的最小的元素比，如果比这个最小的大，那么把这个最小的元素删除，并利用插入排序的思想，把新元素插入到序列中。依次循环，直到扫描了所有的元素。复杂度为O(100w\*100)。
**快排**：采用快速排序的思想，每次分割之后只考虑比轴大的一部分，知道比轴大的一部分在比100多的时候，采用传统排序算法排序，取前100个。复杂度为O(100w\*100)。
**最小堆**：用一个含100个元素的最小堆完成。复杂度为O(100w*lg100)。

**双层桶划分**：整数个数一共2^32个，划分为2^8个区域，一个单一的文件可以代表一个区域。将数据离散到不同区域，在不同的区域利用bitmap进行统计，只要磁盘空间足够即可。



### 5亿个int找到它的中位数？

**bitmap**：可以把int划分为2^16个区域，然后读取数据落在各个区域的个数，在统计结构中判断中位数落在哪个位置，同时对这个区域操作，知道第几大的是中位数。数据量过大可以进行两次划分，先将int64放入2^24个区域内，确定是第几区域大的数，将该区域划分为2^20个子区域，确定是子区域第几大的数字，直接进行统计。

**快排**：内存足够的情况可以使⽤用类似quick sort的思想进行，均摊复杂度为O(n)：随机选取一个元素，将比它小的元素放在它左边，比它大的元素放在右边；如果它恰好在中位数的位置，那么它就是中位数，可以直接返回；如果小于它的数超过一半，那么中位数一定在左半边，递归到左边处理；否则中位数一定在右半边，根据左半边的元素个数计算出中位数是右半边的第几大，然后递归到右半边处理。 
