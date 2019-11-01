---
title: Golang -- Redis Lock
---

> Redis分布式锁的使用。

## Sample Code

以下是示例实现, 利用redislock库实现分布式锁：

```
package redis

import (
	"github.com/bsm/redislock"
	"github.com/go-redis/redis"
	"gslb/pkg/gslb/logger"
	"time"
)

const (
	LOCKTTL    = 10 * time.Second
	LOCKRETRY  = 50
	RETRYDELAY = 20 * time.Millisecond
)

type RedisLock struct {
	LockKey     string
	lock        *redislock.Lock
	ttl         time.Duration
	retryTime   int
	retryDelay  time.Duration
	redisClient *redis.Client
}

func NewChannelLock(key string) *RedisLock {
	return &RedisLock{
		LockKey:     key,
		lock:        nil,
		ttl:         LOCKTTL,
		retryTime:   LOCKRETRY,
		retryDelay:  RETRYDELAY,
		redisClient: r,
	}
}

func (l *RedisLock) RetryObtainLock() bool {
	for i := 1; i <= l.retryTime; i++ {
		if l.TryObtainLock() == true {
			return true
		}
		time.Sleep(l.retryDelay)
		logger.Infof("Retry Obtain Lock...(%v times)", i)
		// TODO : Record to Retry Obtain Lock as metrics log
	}
	return false
}

func (l *RedisLock) TryObtainLock() bool {
	locker := redislock.New(l.redisClient)
	lock, err := locker.Obtain(l.LockKey, l.ttl, nil)
	if err == redislock.ErrNotObtained {
		return false
	} else if err != nil {
		return false
	}
	l.lock = lock
	return true
}

func (l *RedisLock) ReleaseLock() {
	if l.lock != nil {
		l.lock.Release()
		l.lock = nil
	}
}

```

注：

Redis Lock本身可以支持定义Strategy，但是由于想记录retry的Metric和次数等log信息。

以下Option可以定义RetryStrategy:
```
// Options describe the options for the lock
type Options struct {
	// RetryStrategy allows to customise the lock retry strategy.
	// Default: do not retry
	RetryStrategy RetryStrategy

	// Metadata string is appended to the lock token.
	Metadata string

	// Optional context for Obtain timeout and cancellation control.
	Context context.Context
}
```
