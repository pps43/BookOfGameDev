# 大菠萝开发中用到的技巧



# ob需求中用到的技巧
- ###想知道slider里面的handle的拖动事件。

在外面监听不行，因为自带的slider脚本会吃掉begindrag,ondrag,enddrag。在最里面的handle上通过实现接口的方式也不行，这样自带的slider会收不到事件。

解决办法：在handler上实现相关接口的函数最后使用`ExecuteEvents.ExecuteHierarchy`将消息重新抛出。
```cs
void IBeginDragHandler.OnBeginDrag(PointerEventData eventData)
{
    GameObject draggingObj = eventData.pointerCurrentRaycast.gameObject;
    //继续自定义逻辑
        
    ExecuteEvents.ExecuteHierarchy(
            transform.parent.gameObject, 
            eventData, 
            ExecuteEvents.beginDragHandler);
}
```

- ### 不止可拖动，在进度条上点击也能自定义相关逻辑
不要实现click接口，而要用pointer up，挂在Background上。同时还要挂在fill上。

- ### UI动画不受暂停游戏的影响
暂停一般是设置`Time.timescale=0`。若要暂停后菜单动画照常播放，用itween 的ignoretimescale特性。

另外：`FixedUpdate` 当timeScale=0时就不执行了，`Update`每帧还执行的！