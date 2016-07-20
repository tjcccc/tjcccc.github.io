# 实现 uGUI 的长按

　　uGUI 基础功能只有单击，想要实现长按（Long Press / Hold）功能，需要用到 Event Trigger 组件。

　　设计思路是：按下（OnPointerDown）——计时——根据时间触发事件——放开（OnPointerUP）——如时间不满足则撤销事件执行。

　　网上有很多解决方法，看了一圈，总结思考，写了个相对简单的。代码如下：

```c#
private float _pressedTime;	// 时间点
private bool _longPress;	// 计时开关

// 按下
public void OnPointerDown ()
{
	_pressedTime = Time.time;	// 记录按下时的时间。Time.time 是游戏运行时间。
	_longPress = true;	// 通知 FixedUpdate 开始计时。
}

// 放开
public void OnPointerUp ()
{
	_longPress = false;	// 关闭开关，结束计时。
}

// 觉得比 Update 好，每次执行的时间段固定。
void FixedUpdate ()  
{
	if (_longPress)
	{
		float pressingTime = Time.time - _pressedTime; // 计算时间差（秒）——按着不放的时间。
		if (pressedTime >= 1)	// 这里设定为按1秒。
		{
			YourEvent ();	// 执行按1秒后的事件。
			_longPress = false;	// 执行完关闭开关。
		}
	}
}
```
配合代码，在 Event Trigger 面板上记得给 Pointer Down 和 Pointer UP 绑上对应的方法。

　　我其实不太想用到 Update 的方法，但目前想不到更好的替代品。

　　长按的形式不止一种：

1. 长按到一定时间时执行事件——本文码例。
2. 长按到一定时间，放开，如果长按事件满足条件，则执行事件。例如：角色蓄力发动技能。 
3. 长按时执行事件，放开时停止（执行另一事件）。例如：角色跳跃高度随按键长短变化，按得长跳得高，松开落下。

设计思路大同小异。动作方面，无非就是按下和放开两种。长按——捕获到时间差即可。此外注意什么时候执行判定和取消判定。
