---
title: Golang interface best practice
date: 2025-09-02 22:00:53
tags: Golang
---
# 1. 在使用接口的地方定义接口
带来的优点：
+ 各个包之间松耦合，增强了灵活性，简化了测试
+ 增加了代码灵活性
+ 遵循依赖倒置原则

# 2. 让接口简单专用
好的接口设计应该遵循：
+ 只包含所需功能依赖的必要方法
+ 容易实现及理解
+ 可重用、可替换
  
# 3. 使用组合来实现更复杂的接口
+ 提升可重用性，更加清晰
+ 简化了维护和扩展
+ 与go的设计哲学保持一致

# 4. 理解golang中接口的0值
接口nil和纯nil不一样
```golang
var err1 error // Uninitialized interface, nil

type MyError struct{}
func (m *MyError) Error() string{
    return "my error"
}

var myError *MyError
err1 = myError
```