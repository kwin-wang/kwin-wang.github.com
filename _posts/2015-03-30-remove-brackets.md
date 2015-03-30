---
layout: post
title: "面试题目: 括号匹配算法"
description: ""
category: "编程"
tags: [面试题]
---
{% include JB/setup %}

##前言

python写了半个多月，结果今天面试写C++的时候老是忘了写分号，错误百出，导致本来简单的算法题目却没有做！

##题目

题目大意是：“给定一个字符删除对应的括号，比如字符串s='hello<*<abc>>'，输出应该为'hello'"

思想就是：用Stack来解决!

不多说，直接上代码!

##代码

{% highlight cpp %}

#include <cstdio>
#include <string.h>
#include <assert.h>
#include <iostream>

#define STACK_SIZE  1024

template<typename T>
class Stack{
public:
    Stack():pos(-1){
        memset(elem, 0, STACK_SIZE);
    }
    ~Stack(){}

    void push(T& c) {
        assert( pos+1 < STACK_SIZE);

        elem[++pos] = c;
    }

    T pop() {
        return elem[pos--];

    }

    size_t size() {
        return pos + 1;
    }

    bool empty() {
        return (pos == -1) ? true : false;
    }

    T top() {
        assert(pos >= 0);

        return elem[pos];
    }
private:
    T elem[STACK_SIZE];
    int pos; // 当前栈顶元素位置
};

int remove_brackets(char *s)
{
    assert(s != NULL);

    int len = strlen(s);
    assert(len > 0);

    Stack<char> stack;

    for (int i=0; i < len; ++i)
    {
        stack.push(s[i]);

        if (s[i] == '>') {
            while(!stack.empty() && stack.top() != '<') {
                stack.pop();
            }

            if (!stack.empty() && stack.top() == '<') {
                stack.pop();
            }
        }
    }

    return stack.size();
}


int main()
{
    char s[] = "hello<*<*<hellsdfkj>*>*>";

    int pos = remove_brackets(s);
    char temp[100];
    memset(temp, 0, 100);

    strncpy(temp, s, pos);

    printf("%s\n", temp);
    return 0;
}

{% endhighlight cpp %}
