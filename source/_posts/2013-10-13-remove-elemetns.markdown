---
layout: post
title: "删除Java集合中元素"
date: 2013-10-13 21:42
comment: true
categories: java
---
有时候我们需要删除list中的某个元素，最开始我会这样写：

<!-- more -->

``` java
public class ListTest {
	public static void main(String[] args) {
		List<Integer> numbers = new ArrayList();
		for(int i = 1;i <= 10; ++i) {
			numbers.add(i);
		}

		for(int i = 0;i < 10; ++i) {
			if(numbers.get(i) == 5) {
				numbers.remove(i);
			}
		}

		System.out.println(numbers);
	}
}
```
这样写之后，会报错。因为list在循环中的时候是不可以删除它的元素的。
对上面的代码进行一点下改进就可以删除list中的元素了，就是在`remove`之后加个`break`就可以了。
``` java
public class ListTest {
	public static void main(String[] args) {
		List<Integer> numbers = new ArrayList();
		for(int i = 1;i <= 10; ++i) {
			numbers.add(i);
		}

		for(int i = 0;i < 10; ++i) {
			if(numbers.get(i) == 5) {
				numbers.remove(i);
				break;
			}
		}

		System.out.println(numbers);
	}
}
```
上面的方法适用于只删除一个元素，可有的时候我们需要删除多个元素，上面的方法虽可行，但需要为每个需要删除的元素写个for循环，这样效率太低了。
使用`Iterator`可以完成上面的操作。
``` java
public class ListTest {
	public static void main(String[] args) {
		List<Integer> numbers = new ArrayList();
		for(int i = 1;i <= 10; ++i) {
			numbers.add(i);
		}

		for(Iterator it=numbers.iterator() ;it.hasNext(); ) {
			Integer tmp = (Integer) it.next();
			if(tmp == 5) {
				it.remove();
			}
			if(tmp == 7) {
				it.remove();
			}
		}

		System.out.println(numbers);
	}
}
```
上面的方法虽都可以删除元素，但都需要for循环，效率不高，但没有找到更好的方法。感觉java的Iterator没有c++的那么只能，用起来没有c++顺手。

> 对Java的基础知识了解越多，代码就会写得越简洁
