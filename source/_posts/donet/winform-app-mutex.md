---
title: C# Winform 解决程序多开
date: 2022-04-17 15:04:02
tags: C#
---



# 说明
在开发winform 程序是，有时要求程序只能打开一个，这时可以使用Mutex 类来控制  
具体详细文档请看微软 [Mutex](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.mutex.-ctor?view=net-6.0#system-threading-mutex-ctor(system-boolean-system-string-system-boolean@))

## 代码示例


```C#

[STAThread]
static void Main(string[] args) 
{
    var createNew = false;
    var mutex = new Mutex(true, "program.exe", out createNew);
    if(!createNew) 
    {
        MessageBox.Show("该程序已在运行！");
        return;
    }
    Application.Run(new Form1());
}

```


