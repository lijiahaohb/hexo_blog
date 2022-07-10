---
title: 音视频服务器 
categories: 
- av
---
## nginx-rtmp部署

* 下载nginx 和 nginx-rtmp-module
	```
	https://nginx.org/en/download.html
	https://github.com/arut/nginx-rtmp-module.git
	```

* 解压压缩包
	``` bash
	tar xvf nginx-1.20.2.tar.gz
	unzip nginx-rtmp-module-master.zip
	```

* 创建build目录
	``` bash
	cd nginx-1.20.2
	mkdir build
	```
* config & make & make install
	``` bash
	./configure --prefix=/home/lijiahao/code/av/nginx-1.20.2/build --add-module=/home/lijiahao/code/av/nginx-rtmp-module-master
	make
	make install
	```

* config过程中错误及错误处理
	``` bash
	error : ./configure: error: the HTTP rewrite module requires the PCRE library.
	error :./configure: error: SSL modules require the OpenSSL library.
	error :./configure: error: the HTTP gzip module requires the zlib library.

	sudo apt-get update
	sudo apt-get install libpcre3 libpcre3-dev
	sudo apt-get install openssl libssl-dev
	sudo apt-get install zlib1g-dev
	```

* 添加rtmp配置：`/home/lijiahao/code/av/nginx-1.20.2/build/conf/nginx.conf`
	``` 
	rtmp {
		server {
			listen 1935;
			chunk_size 4096;

			# live on (用于直播)
			application rtmp_live {
				live on;
				# hls on; #这个参数把直播服务器改造成实时回放服务器。
				# wait_key on; #对视频切片进行保护，这样就不会产生马赛克了。
				# hls_path ./sbin/html; #切片视频文件存放位置。
				# hls_fragment 10s;     #每个视频切片的时长。
				# hls_playlist_length 60s;  #总共可以回看的时间，这里设置的是1分钟。
				# hls_continuous on; #连续模式。
				# hls_cleanup on;    #对多余的切片进行删除。
				# hls_nested on;     #嵌套模式。
			}

			# play videos (用于点播)
			application rtmp_play{
				play ./videos;  #build directory
			}
		}
	}
	```

* 启动 nginx
	``` bash
	~/code/av/nginx-1.20.2/build$ sudo ./sbin/nginx
	```

* ffmpeg推流
	``` bash
	ffmpeg -i demo.flv -vcodec libx264 -acodec aac -f flv rtmp://localhost:1935/rtmp_live/mystream
	```


* VLC拉流（直播、点播）
	``` 
	直播：rtmp://localhost:1935/rtmp_live/mystream
	点播：rtmp://localhost:1935/rtmp_play/demo.flv
	```

## srs部署

* build srs from srouce
	``` bash
	git clone -b develop https://gitee.com/ossrs/srs.git 
	cd srs/trunk 
	./configure 
	make 
	./objs/srs -c conf/srs.conf
	```

* 添加配置文件`conf/my_hls.conf`
	``` 
	listen              1935;
	max_connections     1000;
	daemon              on;
	srs_log_tank        file;
	srs_log_level        error;
	srs_log_file        ./objs/srs.log;

	http_server {
		enabled         on;
		listen          8081;
		dir             ./objs/nginx/html;
	}

	vhost __defaultVhost__ {
		hls {
			enabled         on;
			hls_fragment    10;
			hls_window      60;
			hls_path        ./objs/nginx/html;
			hls_m3u8_file   [app]/[stream].m3u8;
			hls_ts_file     [app]/[stream]-[seq].ts;
			hls_cleanup     on;
			hls_dispose     30;
			hls_on_error    continue;
			hls_storage     disk;
			hls_wait_keyframe       on;
			hls_acodec      aac;
			hls_vcodec      h264;
		}
	}
	```

* 启动srs
	``` bash
	./objs/srs -c conf/my_hls.conf
	```

* ffmpeg推流
	``` bash
	ffmpeg -re -i demo.flv -c copy -f flv -y rtmp://localhost/live/livestream
	```

* 生成的 m3u8 和 ts 文件路径
	``` 
	/home/lijiahao/code/av/srs/trunk/objs/nginx/html/live
	```

* VLC拉流（rtmp、http）
	```
	rtmp://localhost:1935/live/livestream
	http://localhost:8081/live/livestream.m3u8
	```

## live555部署

* 下载live555源码
	``` 
	http://www.live555.com/liveMedia/public/
	```


* 解压
	``` bash
	tar vxf live.2021.08.24.tar.gz
	cd live
	```

* 编译
	``` bash
	./genMakefiles linux-64bit
	make
	```

* 启动live555
	``` bash
	cd mediaServer
	sudo ./live555MediaServer
	```


* 上传视频
	``` 
	将test.mkv上传至 live/mediaServer目录下
	```

* 播放
	``` 
	拷贝 live555 生成的 url 地址
	vlc拉流：rtsp://192.168.80.128/<filename>
	```

