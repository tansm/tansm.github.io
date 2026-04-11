# Kotlin 实现类似 C# 的 Event 事件代码

> 原文：https://www.cnblogs.com/tansm/p/KotlinEvents.html

在c#中，内置了对事件的设计模式，你可以简单的 += 来订阅一个事件。

Kotlin 目前我没有发现内置的支持（如果你发现了，请留言告诉我 tansm），但Kotlin 非常方便的运算符重载，自己实现也非常简单。

```text
internal typealias EventHandle<T> = (Any,T) -> Unit  //sender, args

internal class EventHandleList<T>{
    private val _observers = mutableListOf<EventHandle<T>>()

    operator fun plusAssign(observer : EventHandle<T>){
        _observers.add(observer)
    }

    operator fun minusAssign(observer: EventHandle<T>){
        _observers.remove(observer)
    }

    operator fun invoke(sender : Any,args : T){
        for(observer in _observers){
            observer(sender,args)
        }
    }
}
```

 

要使用这个定义，也非常容易。下面的代码我们假装 RedoReceiver 是事件的发布者，ReplicationManager 是事件的订阅者。

```text
internal class RedoReceiver{
    val signalArrivalEvents = EventHandleList<SignalArrivalEventArgs>()

    fun daoDa(){
        val signal = SignalType.CONTROL_CONNECT

        signalArrivalEvents(this, SignalArrivalEventArgs(signal))
    }
}

internal class SignalArrivalEventArgs(
        val SignalType : SignalType
)

internal class ReplicationManager{
    private val _r = RedoReceiver()

    init {
        _r.signalArrivalEvents += this::onSignalArraival
    }

    private fun onSignalArraival(sender : Any, e : SignalArrivalEventArgs){

    }
}
```
