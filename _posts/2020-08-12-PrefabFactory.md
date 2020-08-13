---
layout: post
author: 贺工
---
标题：【贺靖轩】PrefabFactory开发记录   
作者：贺靖轩  
标签：Unity UI 效率  
内容： </br>
Todo:
# 模块初始化  
初始化使用带 [RuntimeInitializeOnLoadMethod] 的脚本，它可以不用继承MonoBehaviour  
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test1 
{
    [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSceneLoad)]
     void Function()
    {
        Debug.LogError("RuntimeInitializeOnLoadMethod");
    }
}
```

# 自动ui绑定
使用Attribute属性标签和属性名称进行快速值绑定，无需拖拽赋值。  
参考：https://gitee.com/yimengfan/BDFramework.Core  

# mydata使用struct替换class，内存排列更紧密  

# 使用package管理版本  
