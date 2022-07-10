---
title: im-gochat 
categories: 
- GolangStudy
---
# go-chat

* `connect`架构 <br/>
	`Server`
	``` go
	type Server struct {
		Buckets   []*Bucket
		Options   ServerOptions
		bucketIdx uint32
		operator  Operator
	}
	```

	`Bucket`
	``` go
	type Bucket struct {
		cLock         sync.RWMutex     // protect the channels for chs
		chs           map[int]*Channel // map sub key to a channel
		bucketOptions BucketOptions
		rooms         map[int]*Room // bucket room channels
		routines      []chan *proto.PushRoomMsgRequest
		routinesNum   uint64
		broadcast     chan []byte
	}
	```

	`Room`
	``` go
	type Room struct {
		Id          int
		OnlineCount int // room online user count
		rLock       sync.RWMutex
		drop        bool // make room is live
		next        *Channel
	}
	```

	`Channel`
	``` go
	type Channel struct {
		Room      *Room
		Next      *Channel
		Prev      *Channel
		broadcast chan *proto.Msg
		userId    int
		conn      *websocket.Conn
		connTcp   *net.TCPConn
	}
	```

* 接口文档
	

