## 相关的类：
* WatchDog
* HandlerChecker


## 两类Checker
* Monitor Checker（不是我们关注的重点）：预警我们不能长时间持有核心系统服务的对象锁，否则会阻塞很多函数的运行; 
* Looper Checker：预警我们不能长时间的霸占消息队列，否则其他消息将得不到处理。这两类都会导致系统卡住(System Not Responding)。

## Looper Checker中比较重要的函数：
* WatchDog的addThread函数：
    * 构建一个HandlerChecker实例
    * 加入mHandlerCheckers这个list中
    * 注意这个可以自定义超时时间
```
    public void addThread(Handler thread) {
        addThread(thread, DEFAULT_TIMEOUT);
    }

    public void addThread(Handler thread, long timeoutMillis) {
        synchronized (this) {
            if (isAlive()) {
                throw new RuntimeException("Threads can't be added once the Watchdog is running");
            }
            final String name = thread.getLooper().getThread().getName();
            mHandlerCheckers.add(new HandlerChecker(thread, name, timeoutMillis));
        }
    }
```
* WatchDog的run函数：
    * 调度所有的HandlerChecker
    * 开始定期检查
    * 检查HandlerChecker的完成状态
```
@Override
public void run() {
    boolean waitedHalf = false;
    while (true) {
        ...
        synchronized (this) {
            ...
            // 1. 调度所有的HandlerChecker
            for (int i=0; i<mHandlerCheckers.size(); i++) {
                HandlerChecker hc = mHandlerCheckers.get(i);
                hc.scheduleCheckLocked();
            }
            ...
            // 2. 开始定期检查
            long start = SystemClock.uptimeMillis();
            while (timeout > 0) {
                ...
                try {
                    wait(timeout);
                } catch (InterruptedException e) {
                    Log.wtf(TAG, e);
                }
                ...
                timeout = CHECK_INTERVAL - (SystemClock.uptimeMillis() - start);
            }
 
            // 3. 检查HandlerChecker的完成状态
            final int waitState = evaluateCheckerCompletionLocked();
            if (waitState == COMPLETED) {
                ...
                continue;
            } else if (waitState == WAITING) {
                ...
                continue;
            } else if (waitState == WAITED_HALF) {
                ...
                continue;
            }
 
            // 4. 存在超时的HandlerChecker
            blockedCheckers = getBlockedCheckersLocked();
            subject = describeCheckersLocked(blockedCheckers);
            allowRestart = mAllowRestart;
        }
        ...
        // 5. 保存日志，判断是否需要杀掉系统进程
        Slog.w(TAG, "*** GOODBYE!");
        Process.killProcess(Process.myPid());
        System.exit(10);
    } // end of while (true)
 
}
```    
* HandlerChecker的scheduleCheckLocked函数
    * 判断该Handler对应的Looper消息队列是否是空的
```
 public void scheduleCheckLocked() {
            if (mMonitors.size() == 0 && mHandler.getLooper().getQueue().isPolling()) {
                // If the target looper has recently been polling, then
                // there is no reason to enqueue our checker on it since that
                // is as good as it not being deadlocked.  This avoid having
                // to do a context switch to check the thread.  Note that we
                // only do this if mCheckReboot is false and we have no
                // monitors, since those would need to be executed at this point.
                mCompleted = true;
                return;
            }

            if (!mCompleted) {
                // we already have a check in flight, so no need
                return;
            }

            mCompleted = false;
            mCurrentMonitor = null;
            mStartTime = SystemClock.uptimeMillis();
            mHandler.postAtFrontOfQueue(this);
        }
```
* HandlerChecker的getCompletionStateLocked函数：
    * 判断出该Handler的处理状态
```
        public int getCompletionStateLocked() {
            if (mCompleted) {
                return COMPLETED;
            } else {
                long latency = SystemClock.uptimeMillis() - mStartTime;
                if (latency < mWaitMax/2) {
                    return WAITING;
                } else if (latency < mWaitMax) {
                    return WAITED_HALF;
                }
            }
            return OVERDUE;
        }
```