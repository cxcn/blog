---
title: "输入法词库解析（六）QQ 拼音分类词库.qpyd"
date: 2022-05-25T15:29:14+08:00
categories:
  - 输入法
tags:
  - 输入法
  - 词库
  - 二进制
# draft: true
---

qpyd 格式的难点主要是码表经过了 zlib 压缩，解压后的数据很好解析。

## 原始文件

![](https://tucang.cc/api/image/show/b86501a86fa0ace3fa09f817a4c855cf)

0x38 后跟的 4 字节表示压缩数据开始的字节。

0x44 后跟的 4 字节表示词条数。

## 压缩的数据

使用了 `zlib` 格式。

`golang` 解压 `zlib `:

```go
	// 解压数据
	zrd, err := zlib.NewReader(r)
	if err != nil {
		log.Panic(err)
	}
	defer zrd.Close()
	buf := new(bytes.Buffer)
	buf.Grow(r.Len())
	_, err = io.Copy(buf, zrd)
	if err != nil {
		log.Panic(err)
	}
```

我们看看解压后的数据是什么形式

可以发现它分为两部分，前部分每 10 个一组，总长 10\*词条数。

放到文本编辑器里分析一下，这里取了前后两部分前三条。

![](https://tucang.cc/api/image/show/5f653dd803e89fcca72f59eea9966b52)

可以看到前部分是编码长和词长信息，后半部分 ascii 的编码 + utf-16le 的词条。

## 详解

前半部分保存了所有词条的编码长，词长，索引位置。

![](https://tucang.cc/api/image/show/ef9e706967a70db50a901d2f8ed69e6c)

| 占用字节数 | 描述                    |
| ---------- | ----------------------- |
| 1          | 拼音的长度              |
| 1          | 词字节长                |
| 4          | 未知，全是`00 00 80 3F` |
| 4          | 词条的索引位置          |

后半部分就是词条本身了，拼音和词，词条之间都没有多余字节。

![](https://tucang.cc/api/image/show/78f1112a8bc8ef7ec681162e71ad2e2f)

前面是编码，框里的是词。

## 代码实现

```go

func ParseQqQpyd(rd io.Reader) []Pinyin {
	ret := make([]Pinyin, 0, 1e5)
	data, _ := ioutil.ReadAll(rd)
	r := bytes.NewReader(data)

	// utf-16le 转换器
	decoder := unicode.UTF16(unicode.LittleEndian, unicode.IgnoreBOM).NewDecoder()

	// 0x38 后跟的是压缩数据开始的偏移量
	r.Seek(0x38, 0)
	tmp := make([]byte, 4)
	r.Read(tmp)
	startZip := bytesToInt(tmp)
	// 0x44 后4字节是词条数
	r.Seek(0x44, 0)
	r.Read(tmp)
	dictLen := bytesToInt(tmp)
	// 0x60 到zip数据前的一段是一些描述信息
	r.Seek(0x60, 0)
	head := make([]byte, startZip-0x60)
	r.Read(head)
	b, _ := decoder.Bytes(head)
	fmt.Println(string(b))

	// 解压数据
	zrd, err := zlib.NewReader(r)
	if err != nil {
		log.Panic(err)
	}
	defer zrd.Close()
	buf := new(bytes.Buffer)
	buf.Grow(r.Len())
	_, err = io.Copy(buf, zrd)
	if err != nil {
		log.Panic(err)
	}
	// 解压完了
	r.Reset(buf.Bytes())

	for i := 0; i < dictLen; i++ {
		// 读码长、词长、索引
		addr := make([]byte, 10)
		r.Read(addr)
		idx := bytesToInt(addr[6:]) // 后4字节是索引
		r.Seek(int64(idx), 0)       // 指向索引
		codeSli := make([]byte, addr[0])
		r.Read(codeSli)
		wordSli := make([]byte, addr[1])
		r.Read(wordSli)
		wordSli, _ = decoder.Bytes(wordSli)
		ret = append(ret, Pinyin{string(wordSli), strings.Split(string(codeSli), "'"), 1})
		// 指向下一条
		r.Seek(int64(10*(i+1)), 0)
	}
	return ret
}
```