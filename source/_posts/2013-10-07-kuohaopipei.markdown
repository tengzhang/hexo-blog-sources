---
layout: post
title: "括号匹配"
date: 2013-10-07 16:36
comment: true
categories: 面试题
---
## 原题
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;给定字符串，输出括号是否匹配，例如:  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1."()" yes；  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.")(" no；  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3."(abcd(e)" no；  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4."(a)(b)" yes。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;要求必须用递归写，整个实现不可以出现一个循环语句.

<!-- more -->

## 分析
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个题目很多同学都见过了，如果没有后面的条件，会张口就说就来用栈来实现，时间复杂度O(n)，空间复杂度O(n)。这个是很好的一个解答，没有 问题的。但是我们在做面试题，准备面试的过程中，每一个题目都不应该仅仅局限于某一个方法。应该尝试更多的思路，尽管有些思路的时间、空间复杂度并不是很 好，但是可以带来变化，举一反三，这才是真正的收获。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个题要求了，只目能使用递归并且不能出现循环语句。这个时候，我们应该如何处理呢？其实告诉了大家递归，就比较好想了：怎么定义好问题和子问题。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果字符串中的括号是匹配的，则'('的数量和')'的数量是相等的，反之是不相等的。这样，在递归的过程中，可以保存一个变量，用来记录'('的 数量和')'的数量是否匹配。这样定义递归问题f(p,count)，表示当前字符p之前的字符串中'('的数量和')'的数量的匹配情况，p表示指向当 前字符的指针。初始的时候，f(p, 0)，递归的过程如下：  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果p为空，则考察count是否为0，如果为0，则匹配；如果不为0，则不匹配；  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果不为空，则考察当前字符p，如果p='('，则递归调用f(p++, count++);如果p=')'，则递归调用f(p++, count--)。如果p是其他的字符，并不是'('和')'，则递归调用f(p++, count)，count不变，继续考虑下一次字符。其中需要检查和保证count>=0.  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其实，递归的问题有的时候不是那么好像的，需要大家不断的练习。如果不采用count来记录括号匹配的情况，这个题目的递归也不好想。  
代码如下：

``` cpp
#include <stdio.h>
#include <string.h>
#include <iostream>
#include <string>
#include <algorithm>
#include <map>
using namespace std;
char s[10000];
bool match(char *s,int count) {
    if(!*s && count==0)
        return true;
    else if(!*s && count!=0)
        return false;
    if(count < 0)
        return false;
    if(*s == '(')
        match(s+1,count+1);
    else if(*s == ')')
        match(s+1,count-1);
    else
        match(s+1,count);
}
int main() {
    while(~scanf("%s",s)) {
        if(match(s,0)) {
            printf("yes!\n");
        }
        else {
            printf("no!\n");
        }
    }
    return 0;
}
```
