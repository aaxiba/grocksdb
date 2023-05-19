# grocksdb, RocksDB wrapper for Go

[![](https://github.com/linxGnu/grocksdb/workflows/CI/badge.svg)]()
[![Go Report Card](https://goreportcard.com/badge/github.com/linxGnu/grocksdb)](https://goreportcard.com/report/github.com/linxGnu/grocksdb)
[![Coverage Status](https://coveralls.io/repos/github/linxGnu/grocksdb/badge.svg?branch=master)](https://coveralls.io/github/linxGnu/grocksdb?branch=master)
[![godoc](https://img.shields.io/badge/docs-GoDoc-green.svg)](https://godoc.org/github.com/linxGnu/grocksdb)

This is a `Fork` from [tecbot/gorocksdb](https://github.com/tecbot/gorocksdb). I respect the author work and community contribution.
The `LICENSE` still remains as upstream.

Why I made a patched clone instead of PR:
- Supports almost C API (unlike upstream). Catching up with latest version of Rocksdb as promise.
- This fork contains `no defer` in codebase (my side project requires as less overhead as possible). This introduces loose
convention of how/when to free c-mem, thus break the rule of [tecbot/gorocksdb](https://github.com/tecbot/gorocksdb).

## Install

### Prerequisite 

- librocksdb
- libsnappy
- libz
- liblz4
- libzstd
- libbz2 (optional)

Please follow this guide: https://github.com/facebook/rocksdb/blob/master/INSTALL.md to build above libs.

### Build 

After installing both `rocksdb` and `grocksdb`, you can build your app using the following commands:

```bash
CGO_CFLAGS="-I/path/to/rocksdb/include" \
CGO_LDFLAGS="-L/path/to/rocksdb -lrocksdb -lstdc++ -lm -lz -lsnappy -llz4 -lzstd" \
  go build
```

Or just:
```bash
go build // if prerequisites are in linker paths
```

If your rocksdb was linked with bz2:
```bash
CGO_LDFLAGS="-L/path/to/rocksdb -lrocksdb -lstdc++ -lm -lz -lsnappy -llz4 -lzstd -lbz2" \
  go build
```

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

