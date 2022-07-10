---
title: Golang 配置文件解析 
categories: 
- GolangStudy
---

## viper
* 安装
	``` bash
	go get -u github.com/spf13/viper
	```

* quickStart(常用配置) 
	`config.go`
	``` go
	type Config struct {
		MySQL MySQLConfig
		Redis RedisConfig
		Log LogConfig
		MsgChannelType MsgChannelType
	}

	// MySQL相关配置
	type MySQLConfig struct {
		Host string
		Port int
		User string
		Password string
		DbName string
		TablePrefix string
	}

	type RedisConfig struct {
		Addr string
		Password string
		DB int
		PoolSize int
		MinIdleConns int
	}


	// 日志保存地址
	type LogConfig struct {
		Path string
		Level string
	}

	type MsgChannelType struct {
		ChannelType string
		KafkaHosts string
		KafkaTopic string
	}

	var c Config 

	func init() {
		// 设置文件名
		viper.SetConfigName("config")
		// 设置文件类型
		viper.SetConfigType("toml")
		// 设置文件路径，可以多个viper会根据设置的顺序依次查找
		viper.AddConfigPath(".")
		viper.AutomaticEnv()
		err := viper.ReadInConfig()
		if err != nil {
			panic(fmt.Errorf("fatal error config file: %s", err))
		}

		viper.Unmarshal(&c)
	}

	func GetConfig() Config {
		return c
	}
	```

	`config.toml`
	``` toml
	[mysql]
	Host="127.0.0.1"
	Port=3306
	User="root"
	Password="cv11010216"
	DbName="chat"
	TablePrefix=""

	[redis]
	Addr="localhost:6379"
	Password=""
	DB=0
	PoolSize=30
	MinIdleConns=30
	```

	`config_test.go`
	``` go
	func TestGetConfig(t *testing.T) {
		mySQLConfig := GetConfig().MySQL
		if mySQLConfig.Host != "127.0.0.1" {
			t.Errorf("get mySQLConfig error, expect %s, get %s", "127.0.0.1", mySQLConfig.Host)
		}
		if mySQLConfig.DbName != "chat" {
			t.Errorf("get mySQLConfig error, expect %s, get %s", "chat", mySQLConfig.DbName)
		}
		if mySQLConfig.TablePrefix != "" {
			t.Errorf("get mySQLConfig error, expect %s, get %s", "chat", mySQLConfig.TablePrefix)
		}
	}
	```

	

## ini

