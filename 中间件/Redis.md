# 数据类型

key 类型：只能是string

value 类型：

* String
* List

LRANGE  切片时， start与offset都是下标，包含offset，  因此得到个数为offset-start+1
* Set
* Sorted Set
* Hash

map[string]string

不支持类型嵌套

## 跳表

![调表](../asset/skip-list.jpg)

查找区间内的所有元素，比红黑树方便，范围查询时，红黑树需要中序遍历