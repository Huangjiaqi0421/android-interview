
# Handler Module
- author : codelang

## Question

### Question 1 
>  说下 handler 机制，Looper 通过 MessageQueue 取消息

### Question 2
>  消息队列是先进先出模式，那我延迟发两个消息，第一个消息延迟2个小时，第二个消息延迟1个小时，那么第二个消息需要等3个小时才能取到吗？

* 首先，消息队列不是先进先出模式，我们可以看源码可知，在Handler调用sendMessageDelayed函数后，函数顺序是：
	* sendMessageAtTime函数
	* enqueueMessage函数
	* 最终调用了MessageQueue对象的enqueueMessage函数
* 我们来看看enqueueMessage函数的具体实现：
```
553            msg.markInUse();
554            msg.when = when;
555            Message p = mMessages;
556            boolean needWake;
557            if (p == null || when == 0 || when < p.when) {
558                // New head, wake up the event queue if blocked.
559                msg.next = p;
560                mMessages = msg;
561                needWake = mBlocked;
562            } else {
563                // Inserted within the middle of the queue.  Usually we don't have to wake
564                // up the event queue unless there is a barrier at the head of the queue
565                // and the message is the earliest asynchronous message in the queue.
566                needWake = mBlocked && p.target == null && msg.isAsynchronous();
567                Message prev;
568                for (;;) {
569                    prev = p;
570                    p = p.next;
571                    if (p == null || when < p.when) {
572                        break;
573                    }
574                    if (needWake && p.isAsynchronous()) {
575                        needWake = false;
576                    }
577                }
578                msg.next = p; // invariant: p == prev.next
579                prev.next = msg;
580            }
``` 
* mMessage表示当前的message，如果当前的message为null，或者when（延时）为0，则传递进来的msg变成当前要被处理的message，如果when（延时）比当前message小，则插到当前message的前面
* 由此，可以知道，后进的延迟一小时的message先被处理

