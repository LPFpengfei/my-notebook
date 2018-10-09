## 经典排序算法资料及ruby实现


学习<算法导论>时到网络上找了一批排序算法的资料. 结合参考资料分别用ruby写了一遍. 这里分享给大家. 配上图与参考资料.

### 插入排序

![](https://l.ruby-china.org/photo/2014/9c37b8be54c219e9edc5605376cadeeb.gif)

![](http://upload.wikimedia.org/wikipedia/commons/0/0f/Insertion-sort-example-300px.gif)

```ruby
def insert_sort!
  (0...self.length).to_a.each do |j|
    key = self[j]
    i = j - 1;
    while i >= 0 and self[i] > key
      self[i+1] = self[i]
      i = i-1
    end
    self[i+1] = key
  end
  self
end
```

### 冒泡排序

![](http://upload.wikimedia.org/wikipedia/commons/5/54/Sorting_bubblesort_anim.gif)

![](http://upload.wikimedia.org/wikipedia/commons/0/06/Bubble-sort.gif)

wikipedia 详解: [http://en.wikipedia.org/wiki/Insertion_sort](http://en.wikipedia.org/wiki/Insertion_sort)

[Problem Solving: Searching and sorting](https://en.wikibooks.org/wiki/A-level_Computing/AQA/Paper_1/Fundamentals_of_algorithms/Searching_algorithms)

```ruby
def bubble_sort!
  f = 1
  while f < self.length
    (0...(self.length-f)).to_a.each do |i|
      self[i], self[i+1] = self[i+1], self[i] if self[i] > self[i+1]
    end
    f += 1
  end
  self
end
```

### 鸡尾酒排序

![](http://upload.wikimedia.org/wikipedia/commons/e/ef/Sorting_shaker_sort_anim.gif)

wikipedia 详解: [http://en.wikipedia.org/wiki/Cocktail_sort](http://en.wikipedia.org/wiki/Cocktail_sort)

```ruby
def cocktail_sort!
    f  = 0
    while f < self.length/2
      i = 0
      while i < self.length - 1
        self[i], self[i+1] = self[i+1], self[i] if self[i] > self[i+1]
        i += 1;
      end
      t = self.length - 1
      while t > 0
         self[t], self[t-1] = self[t-1], self[t] if self[t] < self[t-1]
         t -= 1
      end 
      f += 1
    end
    self
  end
```

### 合并排序

![](http://upload.wikimedia.org/wikipedia/commons/c/cc/Merge-sort-example-300px.gif)

![](http://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/Merge_sort_algorithm_diagram.svg/600px-Merge_sort_algorithm_diagram.svg.png)

wikipedia 详解:[http://en.wikipedia.org/wiki/Merge_sort](http://en.wikipedia.org/wiki/Merge_sort)

```ruby
def merge_sort!
  return self if self.size <= 1
  left = self[0, self.size/2]
  right = self[self.size/2, self.size - self.size/2]
  Array.merge(left.merge_sort, right.merge_sort)
end

def self.merge(left, right)
  sorted = []
  until left.empty? or right.empty?
      sorted << (left.first <= right.first ? left.shift : right.shift)
  end
  sorted + left + right
end
```

### 快速排序

![](http://upload.wikimedia.org/wikipedia/commons/6/6a/Sorting_quicksort_anim.gif)

![](http://upload.wikimedia.org/wikipedia/commons/9/9c/Quicksort-example.gif)

wikipedia详解:[http://en.wikipedia.org/wiki/Quicksort](http://en.wikipedia.org/wiki/Quicksort)

三种qicksort的实现方式:[http://c2.com/cgi/wiki?QuickSortInRuby](http://c2.com/cgi/wiki?QuickSortInRuby)

在线图示加语音详解:[http://www.csanimated.com/animation.php?t=Quicksort](http://www.csanimated.com/animation.php?t=Quicksort)

```ruby
def quick_sort!
  return [] if self.empty?
  x, *a = self
  left, right = a.partition{|t| t < x}
  left.quick_sort + [x] + right.quick_sort
end
```

### heapSort

![](http://upload.wikimedia.org/wikipedia/commons/1/1b/Sorting_heapsort_anim.gif)

![](http://upload.wikimedia.org/wikipedia/commons/4/4d/Heapsort-example.gif)

wikipedia 详解: [http://en.wikipedia.org/wiki/Heapsort](http://en.wikipedia.org/wiki/Heapsort)

更深入分析:[http://www.personal.kent.edu/~rmuhamma/Algorithms/MyAlgorithms/Sorting/heapSort.htm](http://www.personal.kent.edu/~rmuhamma/Algorithms/MyAlgorithms/Sorting/heapSort.htm)

```ruby

算法 经典排序算法资料及 ruby 实现 
suffering · 2014年07月18日 · 最后由 arth 回复于 2015年07月17日 · 14918 次阅读

 本帖已被设为精华帖！
经典排序算法资料及ruby实现
学习<算法导论>时到网络上找了一批排序算法的资料. 结合参考资料分别用ruby写了一遍. 这里分享给大家. 配上图与参考资料.

插入排序
insert sort
 insert sort

def insert_sort!
  (0...self.length).to_a.each do |j|
    key = self[j]
    i = j - 1;
    while i >= 0 and self[i] > key
      self[i+1] = self[i]
      i = i-1
    end
    self[i+1] = key
  end
  self
end
冒泡排序
buble sort

buble sort

wikipedia 详解: http://en.wikipedia.org/wiki/Insertion_sort

Problem Solving: Searching and sorting

def bubble_sort!
  f = 1
  while f < self.length
    (0...(self.length-f)).to_a.each do |i|
      self[i], self[i+1] = self[i+1], self[i] if self[i] > self[i+1]
    end
    f += 1
  end
  self
end
鸡尾酒排序
cocktail sort

wikipedia 详解: http://en.wikipedia.org/wiki/Cocktail_sort

def cocktail_sort!
    f  = 0
    while f < self.length/2
      i = 0
      while i < self.length - 1
        self[i], self[i+1] = self[i+1], self[i] if self[i] > self[i+1]
        i += 1;
      end
      t = self.length - 1
      while t > 0
         self[t], self[t-1] = self[t-1], self[t] if self[t] < self[t-1]
         t -= 1
      end 
      f += 1
    end
    self
  end
合并排序
merge sort

merge sort

wikipedia 详解: http://en.wikipedia.org/wiki/Merge_sort

def merge_sort!
  return self if self.size <= 1
  left = self[0, self.size/2]
  right = self[self.size/2, self.size - self.size/2]
  Array.merge(left.merge_sort, right.merge_sort)
end

def self.merge(left, right)
  sorted = []
  until left.empty? or right.empty?
      sorted << (left.first <= right.first ? left.shift : right.shift)
  end
  sorted + left + right
end
快速排序
quick sort

quick sort

wikipedia详解: http://en.wikipedia.org/wiki/Quicksort

三种qicksort的实现方式: http://c2.com/cgi/wiki?QuickSortInRuby

在线图示加语音详解: http://www.csanimated.com/animation.php?t=Quicksort

def quick_sort!
  return [] if self.empty?
  x, *a = self
  left, right = a.partition{|t| t < x}
  left.quick_sort + [x] + right.quick_sort
end
heapSort
heap sort

heap sort

wikipedia 详解: http://en.wikipedia.org/wiki/Heapsort

更深入分析: http://www.personal.kent.edu/~rmuhamma/Algorithms/MyAlgorithms/Sorting/heapSort.htm

def heap_sort!
  # in pseudo-code, heapify only called once, so inline it here
  ((length - 2) / 2).downto(0) {|start| siftdown(start, length - 1)}

  # "end" is a ruby keyword
  (length - 1).downto(1) do |end_|
    self[end_], self[0] = self[0], self[end_]
    siftdown(0, end_ - 1)
  end
  self
end

def siftdown(start, end_)
  root = start
  loop do
    child = root * 2 + 1
    break if child > end_
    if child + 1 <= end_ and self[child] < self[child + 1]
      child += 1
    end
    if self[root] < self[child]
      self[root], self[child] = self[child], self[root]
      root = child
    else
      break
    end
  end
end
```

最后, 附上完整的ruby实现:

```ruby
class Array
  # 插入排序
  def insert_sort!
    (0...self.length).to_a.each do |j|
      key = self[j]
      i = j - 1;
      while i >= 0 and self[i] > key
        self[i+1] = self[i]
        i = i-1
      end
      self[i+1] = key
    end
    self
  end

  # 快速排序
  def quick_sort!
    return [] if self.empty?
    x, *a = self
    left, right = a.partition{|t| t < x}
    left.quick_sort + [x] + right.quick_sort
  end

  # 冒泡排序
  def bubble_sort!
    f = 1
    while f < self.length
      (0...(self.length-f)).to_a.each do |i|
        self[i], self[i+1] = self[i+1], self[i] if self[i] > self[i+1]
      end
      f += 1
    end
    self
  end

  # 鸡尾酒排序(>_<)
  def cocktail_sort!
    f  = 0
    while f < self.length/2
      i = 0
      while i < self.length - 1
        self[i], self[i+1] = self[i+1], self[i] if self[i] > self[i+1]
        i += 1;
      end
      t = self.length - 1
      while t > 0
         self[t], self[t-1] = self[t-1], self[t] if self[t] < self[t-1]
         t -= 1
      end 
      f += 1
    end
    self
  end

  # 合并排序
  def merge_sort!
    return self if self.size <= 1
    left = self[0, self.size/2]
    right = self[self.size/2, self.size - self.size/2]
    Array.merge(left.merge_sort, right.merge_sort)
  end

  def self.merge(left, right)
    sorted = []
    until left.empty? or right.empty?
        sorted << (left.first <= right.first ? left.shift : right.shift)
    end
    sorted + left + right
  end

  # heap排序
  def heap_sort!
    # in pseudo-code, heapify only called once, so inline it here
    ((length - 2) / 2).downto(0) {|start| siftdown(start, length - 1)}

    # "end" is a ruby keyword
    (length - 1).downto(1) do |end_|
      self[end_], self[0] = self[0], self[end_]
      siftdown(0, end_ - 1)
    end
    self
  end

  def siftdown(start, end_)
    root = start
    loop do
      child = root * 2 + 1
      break if child > end_
      if child + 1 <= end_ and self[child] < self[child + 1]
        child += 1
      end
      if self[root] < self[child]
        self[root], self[child] = self[child], self[root]
        root = child
      else
        break
      end
    end
  end

  %w(insert quick bubble cocktail merge heap).each do |metd|
    define_method("#{metd}_sort") do
      self.dup.send("#{metd.to_s}_sort!") 
    end
  end
end

p b = ((1..100).to_a + (20..80).to_a).shuffle.sample(19)
p b.methods.grep /_sort/
p b.insert_sort
p b.bubble_sort
p b.quick_sort
p b.cocktail_sort
p b.merge_sort
p b.heap_sort
```

最后看看这个[http://www.sorting-algorithms.com/](http://www.sorting-algorithms.com/)


