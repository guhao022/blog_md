---
date: 2015-10-02
title: golang写一个http代理服务器
categories: 
- 技术
tags: 
- golang
- 服务器
---

一个简单的http代理服务器


代码，首先创建一个名为Anget的结构体：

```go
type Anget struct {
	Server string           //需要代理的host
	Uri    string           //访问的uri
	Param  string           //传递的参数
	Method string           //访问方式（restful）
	resp   *http.Response
}
```

给Anget添加一个result方法，接收返回的信息

```go
func (anget *Anget) result(w http.ResponseWriter, r *http.Request) {
	// 解析参数, 默认是不会解析的
	r.ParseForm()

	anget.Uri = r.URL.Path  //获取访问的uri
	anget.Method = r.Method //获取访问方法method
	anget.Param = r.Form.Encode()   //获取需要的参数

	u, _ := url.Parse("http://" + anget.Server + anget.Uri)

    // 输出想要的信息
	log.Println("地址：", u)
	log.Println("参数：", anget.Param)
	log.Println("方法：", anget.Method)
	total++

    // 判断访问方法，访问原链接
	switch anget.Method {
	case "GET":
		anget.resp, _ = http.Get(u.String())
	case "POST":
		anget.resp, _ = http.Post(u.String(), "application/x-www-form-urlencoded", strings.NewReader(anget.Param))
	default:
		http.Error(w, http.StatusText(500), 500)
	}
	defer anget.resp.Body.Close()
	body, _ := ioutil.ReadAll(anget.resp.Body)

	// 输出到客户端
	fmt.Fprintf(w, string(body))
	log.Println("返回：", string(body))
}
```

然后写一个http服务器，用作客户端访问

```go
func (anget *Anget) Run(port string) {
    //检测需要代理的host是否有效
	conn, err := net.Dial("tcp", anget.Server)
	if err != nil {
		fmt.Println("连接服务端失败:", err.Error())
		os.Exit(0)
	}
	fmt.Println("已连接测试服务器～～～")
	conn.Close()

	http.HandleFunc("/"+anget.Uri, anget.result)
	
	fmt.Println("代理服务器开启，端口为：" + port)
	// 设置监听的端口
	err = http.ListenAndServe(":" + port, nil)
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}
```

```go
func main() {
    var ang mod.Anget

	commands := os.Args
	if len(commands) < 3 {
		help("help")
		os.Exit(0)
	}

	switch commands[1] {
	case "help":
		fmt.Println(Helptext)
	case "run":
		switch commands[2] {
		case "--help", "-h":
			help("help")
			os.Exit(0)
		default:
			ang.Server = commands[2]
		}
		ang.Run("9900")
	default:
		help("help")
	}
}
```

ok，代理服务器完成，help的内容自己完成吧，或者删除也可以，只要把原来的host地址改为代理服务器的host就可以实现代理了

```bash
go run main.go run 127.0.0.1:80
```




