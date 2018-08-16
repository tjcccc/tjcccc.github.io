---
layout: post
title: Play Timeline with UniRx
key: 20180722
tags: UniRx Unity
---
# 用 UniRx 实现 Timeline 式的异步操作

　　没接触 UniRx 之前，我在 Unity 中通常用 Coroutine 或 Callback 来实现异步操作。根据我的任务，一般都是去实现游戏组件的演出，比如：敌方角色图形显示后，我方角色 UI 出现，再跳出信息窗口什么的。

　　举个抽象例子：一开始执行 A ——第 3 秒执行 B ——第 5 秒执行 C

<!--more-->

　　用 Coroutine 方式：

```csharp
IEnumerator Play()
{
    DoA();
    yield return WaitForSeconds(3);
    DoB();
    yield return WaitForSeconds(2);
    DoC();
}
StartCoroutine(Play());
```

用 DOTween CallBack 方式：

```csharp
DoA().OnComplete(() =>
{
    DoB().SetDelay(3).OnComplete(() =>
    {
        DoC().SetDelay(2);
    });
});
```

这些做法并无不妥，但我在学习 UniRX 之后发现了更加逻辑清晰的方式，那就是实现一个 Timeline 时间轴：

```csharp
void PlayTimeline()
{
    // 设置计时器
    var timer = new IntReactiveProperty(0);

    // 将 1 秒所用的 frames 作为间隔参数，进行每秒执行
    timer.SampleFrame((int)(1 / Time.fixedDeltaTime)).BatchFrame().Subscribe(_ => timer.Value += 1);

    // 根据时间表安排执行任务
    timer.Where(t => t == 1).Subscribe(_ => DoA());
    timer.Where(t => t == 3).Subscribe(_ => DoB());
    timer.Where(t => t == 5).Subscribe(_ => DoC());

    // 执行完取消订阅
    timer.Where(t => t == 6).Subscribe(_ => timer.Dispose());
}
```

这种思路和做动画类似，就是时间点到了各自去执行自己的演出任务。

　　当然你 `DoB()` 也可以等待 `DoA()` 给个信号再执行，只要把 `DoA()` 变成一个 Observable 对象就可以了。但那也就不是 Timeline 了，更适合其他场景。
