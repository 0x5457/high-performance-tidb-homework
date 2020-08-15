# lesson 01 作业

## 1. clone 代码

```shell
$ git clone https://github.com/pingcap/pd.git
$ git clone https://github.com/tikv/tikv.git
$ git clone https://github.com/pingcap/tidb.git
```

## 2. 修改代码使得 tidb 启动事务时，会打一个“hello transaction”的日志
 
 修改 `tidb/store/tikv/kv.go` 的 `Begin` 和 `BeginWithStartTS` 方法。添加“hello transaction”的日志

 ### 修改前：
 ```go
 func (s *tikvStore) Begin() (kv.Transaction, error) {
	txn, err := newTiKVTxn(s)
	if err != nil {
		return nil, errors.Trace(err)
	}
	return txn, nil
}

// BeginWithStartTS begins a transaction with startTS.
func (s *tikvStore) BeginWithStartTS(startTS uint64) (kv.Transaction, error) {
	txn, err := newTikvTxnWithStartTS(s, startTS, s.nextReplicaReadSeed())
	if err != nil {
		return nil, errors.Trace(err)
	}
	return txn, nil
}
 ```

 ### 修改后：
 ```go
 func (s *tikvStore) Begin() (kv.Transaction, error) {
  logutil.BgLogger().Info("hello transaction")
	txn, err := newTiKVTxn(s)
	if err != nil {
		return nil, errors.Trace(err)
	}
	return txn, nil
}

// BeginWithStartTS begins a transaction with startTS.
func (s *tikvStore) BeginWithStartTS(startTS uint64) (kv.Transaction, error) {
	logutil.BgLogger().Info("hello transaction")
	txn, err := newTikvTxnWithStartTS(s, startTS, s.nextReplicaReadSeed())
	if err != nil {
		return nil, errors.Trace(err)
	}
	return txn, nil
}
 ```



## 2. 构建

### tikv
``` shell
$ cd tikv
$ make build
```

### tidb
``` shell
$ cd tidb
$ make
```

### pd
``` shell
$ cd pd
$ make
```

## 3. 运行

### pd
``` shell
$ cd pd
$ ./bin/pd-server
```

### tikv
``` shell
$ cd tikv
$ 这里为了方便运行 我启动了3个命令行窗口
$ ./target/debug/tikv-server --pd="127.0.0.1:2379"
$ ./target/debug/tikv-server --pd="127.0.0.1:2379"
$ ./target/debug/tikv-server --pd="127.0.0.1:2379"
```

### tidb
``` shell
$ cd pd
$ ./bin/tidb-server --store=tikv --path=127.0.0.1:2379
```

## 4. 测试结果
- 通过 mysql客户端 连接 tidb: `mysql -h127.0.0.1 -P 4000 -uroot -proot。
- 执行 `BEGIN` 发现 "hello transaction" 有正常输出。
> 运行发现每几秒有自动打印打多个 "hello transaction"，猜测是 tidb 内部sql导致。

## 5. 问题记录
- 在运行 tikv 时发生错误: `the maximum number of open file descriptors is too small, got 256, expect greater or equal to 82920`。
  
  **通过 `ulimit -n 82920` 命令修改“打开文件描述符的最大值”到 82920 解决**