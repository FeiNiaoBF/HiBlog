---
title: "数组(Go语言实现)"
date: 2024-03-10T16:21:29+08:00
draft: false
taps: ["Array", "Golang"]
categories: ["Golang", "CS", "DataStructures"]
author: ["Yeelight"]
showtoc: true
weight:
readingTime: true
---

## 数组介绍

**数组 `array`** 是一种线性数据结构，其将相同类型的元素存储在连续的内存空间中。我们将元素在数组中的位置称为该元素的 **索引 `index`** 。

### 数组的优点与局限性:

数组存储在连续的内存空间内，且元素类型相同。这种做法包含丰富的先验信息，系统可以利用这些信息来优化数据结构的操作效率。

- **空间效率高**：数组为数据分配了连续的内存块，无须额外的结构开销。
- **支持随机访问**：数组允许在 �(1) 时间内访问任何元素。
- **缓存局部性**：当访问数组元素时，计算机不仅会加载它，还会缓存其周围的其他数据，从而借助高速缓存来提升后续操作的执行速度。

连续空间存储是一把双刃剑，其存在以下局限性。

- **插入与删除效率低**：当数组中元素较多时，插入与删除操作需要移动大量的元素。
- **长度不可变**：数组在初始化后长度就固定了，扩容数组需要将所有数据复制到新数组，开销很大。
- **空间浪费**：如果数组分配的大小超过实际所需，那么多余的空间就被浪费了。

## Go 语言实现

```go
/// Array in Golang

// Array Data Structure
type array struct {
    data   []int
    length uint
}

// NewArray create a array
func NewArray(capacity uint) *array {
    if capacity == 0 {
        return nil
    }
    return &array{
        data:   make([]int, capacity),
        length: 0,
    }
}

/// Public function of array

// Len get the array length
func (arr *array) Len() uint {
    return arr.length
}

// Insert insert the value at the index
func (arr *array) Insert(val int, ind uint) (*array, error) {
    if arr.isIndexOutOfRange(ind) {
        return nil, fmt.Errorf("out of range")
    }
    if arr.Len() == uint(cap(arr.data)) {
        return nil, fmt.Errorf("full array")
    }
    for i := arr.Len(); i > ind; i-- {
        arr.data[i] = arr.data[i-1]
    }
    arr.data[ind] = val
    arr.length++
    return arr, nil
}

// Delete delete the value at the index
func (arr *array) Delete(ind uint) (*array, error) {
    if arr.isIndexOutOfRange(ind) {
        return nil, fmt.Errorf("out of range")
    }
    for i := ind; i < arr.Len()-1; i++ {
        arr.data[i] = arr.data[i+1]
    }
    arr.length--
    return arr, nil
}

// Find find the value at the index
func (arr *array) Find(ind uint) (int, error) {
    if arr.isIndexOutOfRange(ind) {
        return 0, fmt.Errorf("out of range")
    }
    return arr.data[ind], nil
}

// Print print the array
func (arr *array) Print(){
    var format string
    for i := uint(0); i < arr.Len(); i++ {
        format += fmt.Sprintf("|%+v", arr.data[i])
    }
    fmt.Println(format)
}


/// Private function of array

// isIndexOutOfRange check if the index is out of range
func (arr *array) isIndexOutOfRange(index uint) bool {
    return index >= uint(cap(arr.data))
}
```


## 相关算法


### Bobble Sort

```go
// BobbleSort bobble sort
func (arr *array) Bobble() {
 if arr.Len() <= 1 {
  return
 }
 for i := uint(0); i < arr.Len(); i++ {
  for j := uint(0); j < arr.Len()-i-1; j++ {
   if arr.data[j] > arr.data[j+1] {
    arr.data[j], arr.data[j+1] = arr.data[j+1], arr.data[j]
   }
  }
 }
}
```

### Binary Search

```go
// BinarySearch
func (arr *array) BinarySearch(tar int) int {
 var (
  low  = 0
  high = int(arr.Len()) - 1
 )
 for low <= high {
  mid := low + (high-low)/2
  if arr.data[mid] > tar {
   high = mid - 1
  }else if arr.data[mid] < tar {
   low = mid + 1
  }else{
   return mid
  }
 }
 return -1
}
```
