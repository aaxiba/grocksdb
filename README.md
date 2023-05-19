# grocksdb, RocksDB wrapper for Go

## Usage

See also: [doc](https://godoc.org/github.com/linxGnu/grocksdb)

## API Support

Almost C API, excepts:
- [ ] putv/mergev/deletev/delete_rangev
- [ ] compaction_filter/compaction_filter_factory/compaction_filter_context
- [ ] transactiondb_property_value/transactiondb_property_int
- [ ] optimistictransactiondb_property_value/optimistictransactiondb_property_int


# 20230519 更新：

> 说明：这是一个临时的fork仓库，用来在Mac 下方便测试 rocksdb 的 8.0+ 版本的

## 使用步骤

- 使用 brew 安装
```
brew install rocksdb
brew link --overwrite rocksdb

```

- 引用本仓库：
```
go get github.com/aaxiba/grocksdb
```

- 示例代码：
```
package main

import (
	"fmt"
	"github.com/aaxiba/grocksdb"
)

func main() {
	fmt.Println("go rocksdb")
	cache := grocksdb.NewLRUCache(3 << 10)
	bbto := grocksdb.NewDefaultBlockBasedTableOptions()
	bbto.SetBlockCache(cache)

	opts := grocksdb.NewDefaultOptions()
	opts.SetBlockBasedTableFactory(bbto)
	opts.SetCreateIfMissing(true)

	db, err := grocksdb.OpenDb(opts, "/tmp/rocksdb-test")

	wo := grocksdb.NewDefaultWriteOptions()
	err = db.Put(wo, []byte("k1"), []byte("v1"))
	defer db.Close()
	fmt.Println(db, err)
	value, err := db.Get(grocksdb.NewDefaultReadOptions(), []byte("k1"))
	defer value.Free()
	fmt.Printf("value.Data: %+v, err: %v\n", string(value.Data()), err)
}
```

- build 示例代码：
```
CGO_LDFLAGS="-L/usr/local/Cellar/rocksdb/8.1.1/ -lrocksdb -lstdc++ -lm -lz -lsnappy -llz4 -lzstd -lbz2"   /data/go1.19/bin/go build main.go
```

