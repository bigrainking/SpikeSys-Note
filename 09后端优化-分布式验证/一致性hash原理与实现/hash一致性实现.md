



# 实现一致性hash

## 1. 要实现的功能

- **创建hash环**
  - hash环： map[key]value 节点key：实体服务器
  - sortedHashes ： hash key都排序，方便查找资源缓存在哪台服务器上，查找时通过二分法查找到对应的服务器
  - 虚拟节点个数
  - 读写锁：在添加删除节点时要将hash环锁住
- 【AddKey】向环中插入服务器节点and虚拟节点
  - 【GenerateKey】每个节点生成对应Key
  - 【add】将节点添加到环上

- 【Get】获取内容在哪台服务器上
  - 【hashKey】计算内容对应的hash值
  - 【searchKey】在hash环上查找对应的节点，顺时针查找最近的节点
- 【Remove】删除服务器节点
  - 【remove】删除服务器节点
- 【updateSortedHashes】更新排序后的hash值，用于二分查找(sort.Search)

> 1. 其中hash环 上每个节点无论是虚拟节点or非虚拟节点都指向自己对应的实体节点，因此节点不一样，但节点key对应的value是一样的。
>
> 2. 【generateKey】每个节点生成对应Key副本， 此时是生成的服务器对应的虚拟节点，还不是真正的在hash环上的key，hash环上的key需要 `hashKey`来生成



## 2. 实现

- **创建hash环**

  `type Consistent struct`

- 【AddKey】向环中插入服务器节点and虚拟节点

  - 【generateKey】每个节点生成对应Key副本 (A服务器生成20个副本=虚拟节点)
  - 【add】将节点添加到环上

- 【Get】获取内容在哪台服务器上
  - 【hashKey】计算element对应的hash值：通过hash算法得到真正的Key
  - 【searchKey】在hash环上查找对应的节点，顺时针查找最近的节点
- 【Remove】删除服务器节点
  - 【remove】删除服务器节点
- 【updateSortedHashes】更新排序后的hash值，用于二分查找(sort.Search)

### 1. 创建环

```go
// - 创建hash环
type Consistent struct {
	// 环
	circle map[uint32]string //存储服务器key值and服务器信息
	// 虚拟节点个数
	VirtualNode int
	// 排序后的环的key对应的列表
	sortedHashes units
	// 读写锁:增删环上节点时不能读写
	sync.RWMutex
}
```



### 2.【AddKey】向环中插入服务器节点and虚拟节点

**插入步骤**

1.  将每个服务器生成副本（创建虚拟节点）`generateKey(element string, index int) string`
2.  生成每个节点的hashKey值 `hashKey(element string) uint32`
3.  将hashKey对应的插入到hash环中 `add(element string)`
4.  外部调用Add正式添加服务器时需要对环加锁 

```go
// 【add】将节点添加到环上
func (c *Consistent) add(element string) {
	// 1. 生成虚拟节点的副本 2. 副本添加到环上 3. 每个虚拟节点都指向服务器
	for i := 0; i < c.VirtualNode; i++ {
		c.circle[c.hashKey(c.generateKey(element, i))] = element
	}
	// 更新circle上新的hash值的排序
	c.updateSortedHashes()
}
```



```go
// 【AddKey】向环中插入服务器节点and虚拟节点
func (c *Consistent) Add(element string) {
	// 读写锁
	c.Lock()
	defer c.Unlock()
	c.add(element)
}
```



```go
// 【generateKey】每个节点生成副本Key
func (c *Consistent) generateKey(element string, index int) string {
	return element + strconv.Itoa(index)
}
```



```go
//   - 【hashKey】计算内容对应的hashKey
func (c *Consistent) hashKey(element string) uint32 {
	// 如果element不够容量
	if len(element) < 64 {
		var scratch [64]byte
		copy(scratch[:], element) //扩容
		return crc32.ChecksumIEEE(scratch[:len(element)])
	}
	return crc32.ChecksumIEEE([]byte(element))
}
```



### 3.【Get】获取内容在哪台服务器上

> 查看要查找的内容在哪台服务器上

**步骤**

1. 计算element在环上的key值：`key := c.hashKey(element)`

2. 二分查找对应的服务器：`seachKey(hashKey uint32) int`

   ```go
   f := func(i int) bool {
   		return c.sortedHashes[i] > hashKey
   	}
   
   index := sort.Search(len(c.sortedHashes), f)
   ```

3. 返回服务器对应的element



```go
// - 【Get】获取内容在哪台服务器上
// name:要查找的对象的信息
// element：存储name的服务器信息
func (c *Consistent) Get(name string) (element string, err error) {
	// 查找时，添加读锁
	c.RLock()
	defer c.Unlock()
	if len(c.circle) == 0 {
		return "", errEmpty
	}
	// 计算hash值，返回对应的服务器
	key := c.hashKey(element) //在环上找到自己的位置
	index := c.seachKey(key)
	return c.circle[c.sortedHashes[index]], nil
}
```



通过key值查找对应的服务器： 

```go
//【searchKey】在hash环上查找对应的节点，顺时针查找最近的节点
// 传入对应的hashKey值, 返回在环上满足条件的节点，节点对应的sortedHashes的下标
func (c *Consistent) seachKey(hashKey uint32) int {
	f := func(i int) bool {
		return c.sortedHashes[i] > hashKey //返回hashkey对应的最近的服务器的key
	}
	index := sort.Search(len(c.sortedHashes), f)
	// 如果index没有找到：超出了最大长度
	if index >= len(c.sortedHashes) {
		return 0
	}
	return index
}
```



### 4.【Remove】删除服务器节点



**删除步骤**

- 删除服务器，同时删除对应的所有虚拟节点  `delete`

- 同时更新排序的hashes  : 调用函数 `sort.Sort(hashes)`

  - 调用函数需要hashes实现接口

     `type units []uint32`

    `hashes 是units类型`



**删除**

```go
// - 【Remove】删除服务器节点
func (c *Consistent) Remove(element string) {
	c.Lock()
	defer c.Unlock()
	c.remove(element)
}

//   - 【remove】删除服务器节点
// element：服务器节点信息ip
func (c *Consistent) remove(element string) {
	// 删除节点:删除所有副本
	for i := 0; i < c.VirtualNode; i++ {
		delete(c.circle, c.hashKey(c.generateKey(element, i)))
	}

	// 更新排序hash
	c.updateSortedHashes()
}
```



### 5.【updateSortedHashes】更新排序后的hashKey

```go
// 指定切片:实现接口
// 使用sort.Sort需要实现接口，实现接口对应的三个function
type units []uint32

// 返回切片长度
func (x units) Len() int {
	return len(x)
}

// 对比两个数字的大小
func (x units) Less(i, j int) bool {
	return x[i] < x[j]
}
func (x units) Swap(i, j int) {
	x[i], x[j] = x[j], x[i]
}


// - 【updateSortedHashes】更新排sortedHashes，用于二分查找(sort.Search)
func (c *Consistent) updateSortedHashes() {
	// 拷贝空串
	hashes := c.sortedHashes[:0] //units类型
	// 核对容量是否过大,过大则重置:切片容量 是 hash环上节点数*虚拟节点个数的4倍
	if cap(c.sortedHashes) > len(c.circle)*c.VirtualNode*4 {
		hashes = nil
	}
	// 将环上circle现有的节点添加到sortedHashes中
	for key := range c.circle {
		hashes = append(hashes, key)
	}
	// 将hashes排序
	sort.Sort(hashes)
	// 排序后更新到c.sortedHases
	c.sortedHashes = hashes
}
```

