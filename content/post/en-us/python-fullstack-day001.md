---
title: "Python Fullstack Day 001"
date: 2019-05-01T09:04:16+08:00
draft: true
tags: ["python", "fullstack"]
categories: ["programming"]
auther: "steve dong"
reward: true

---


# Python FullStack Day 001

Today I will learn python fullstack developer from strach.This article will be a notebook for learning python and This will push me to keep to learn.




## Computer basic
| Type | Function |
|---|---|
|cpu| The brain of a computer |
|memory | More expensive than HD, Data on memory will lost when power lost.|
| OS    |   |


## Python History



## Python2 vs python3

  - python2 encoding with ascii,python3 encoding with utf-8, so python2 need to add `#-*- encoding:utf-8 -*-` to specify encoding type.

## Variable

### How to define a variable

#### 3 Rules:

  - Variable must be contant with numbers,alphebet and under scope, and the first letter can not be with numbers.
  - Not define with key words, like these:
 
```python
 ['and', 'as', 'assert', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'exec', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'not', 'or', 'pass', 'print', 'raise', 'return', 'try', 'while, 'with, 'yield']
```
 - varible must be a specified name . It is clearly to let reader can read it.

## Constant

## Comment

  1. `#` to comment single line
  2. `'''` or `"""` to comment multilines.

## Data type

  - string
  - number
  - bool

## interacting

`input()`
## Operator

 ``` python
 +,-, *, /, **
 %
 type()
 int(str)
 str(int)
 str + str # Join two strings as one.
 
 ```
## flow
### if
``` 
 ### `if`
 if (...):
 	...;
 elif (...):
 	...; 
 else:
 	...; 
```
 	
```python
score = int(input("Enter your score:"))

if score > 100;
	print("Just 100 huh? :-)")
elif score >=90:
	print("A")
elif score >=80:
	print("B")
elif score >=60:
	print("C")
elif score >=40:
	print("D")
else:
	print("Dummy! E")
``` 	

### while

```python
82
#!/usr/bin/env python
# -*- coding:utf-8 -*-

count = 1
flag = True

while  flag:
     print(count)
     count += 1
     if count > 100:
         flag = False


```
 	
 	
 	
