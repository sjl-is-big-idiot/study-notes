《拉勾教育-3天掌握项目管理核心》



![image-20210608202656749](C:\Users\sjl\AppData\Roaming\Typora\typora-user-images\image-20210608202656749.png)

管理的入门就是把人分类。



沟通秘诀一：DISC性格分类



![image-20210608203620787](C:\Users\sjl\AppData\Roaming\Typora\typora-user-images\image-20210608203620787.png)





![image-20210608204001844](C:\Users\sjl\AppData\Roaming\Typora\typora-user-images\image-20210608204001844.png)





![image-20210608210316191](C:\Users\sjl\AppData\Roaming\Typora\typora-user-images\image-20210608210316191.png)

### 题目 1

给定 n，生成由 n 对括号（仅小括号）组成的所有合法的括号组合。例如：N = 2，输出

```
()()
(())
func generate(N int) []string {
    // please enter your code here
}
```

### 题目 2

判定给定的括号序列是否合法。例如：`()[]{}` 合法，`({[]}(` 不合法



```python
def validate(string):
    stack = []
    tmp = {"(":")", "[":"]", "{": "}"}
    for s in string:
        if s in tmp.keys():
            stack.append(s)
        else:
            if stack and stack.pop() == temp.get(s):
                continue
            return False
            
    if not stack:
        return True
    return False
```

