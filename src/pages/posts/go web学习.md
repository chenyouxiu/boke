---
layout: '../../layouts/MarkdownPost.astro'
title: 'go web的学习'
pubDate: 2022-04-17
description: '从Go源码分析切片的扩容机制。'
author: 'Austin'
cover:
    url: 'https://pic.lookcos.cn/i/usr/uploads/2023/02/1277661091.png'
    square: 'https://pic.lookcos.cn/i/usr/uploads/2023/02/1277661091.png'
    alt: 'cover'
tags: ["源码研究", "标准库", "golang", "slice"]
theme: 'dark'
featured: false
---

## 请求requests

```go
package main

import (
	"fmt"
	"net/http"
)



func main()  {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/post", func(writer http.ResponseWriter, request *http.Request) {
		length := request.ContentLength
		body := make([]byte,length)
		request.Body.Read(body)
		fmt.Fprintln(writer,string(body))
	})

	server.ListenAndServe()

}
```

## 查询参数

```go
package main

import (
	"log"
	"net/http"
)


func main()  {
	http.HandleFunc("/home", func(writer http.ResponseWriter, request *http.Request) {
		url := request.URL
		query := url.Query()

		id := query["id"]   //返回列表
		log.Println(id)
		
		name := query.Get("name")  //返回的是一个字符串，只返回第一个
		log.Println(name)


	})
	http.ListenAndServe("localhost:8080",nil)

}
```



## FORM





## ResponesWriter



返回网页

```go
package main

import (
	//"log"
	"net/http"
)

func writeExample(w http.ResponseWriter,r *http.Request)  {
	str := `<H1>hello</h1>`
	w.Write([]byte(str))

}

func main()  {
	server := http.Server{
		Addr: "localhost:8080",
	}
	http.HandleFunc("/write",writeExample)
	server.ListenAndServe()

}
```



![image-20221124074610224](C:\Users\Mo\AppData\Roaming\Typora\typora-user-images\image-20221124074610224.png)







## writeHeader方法

![image-20221124074655523](C:\Users\Mo\AppData\Roaming\Typora\typora-user-images\image-20221124074655523.png)





修改状态码 501 返回

```go
package main

import (
	//"log"
	"fmt"
	"net/http"
)

func writeExample(w http.ResponseWriter,r *http.Request)  {
	str := `<H1>hello</h1>`
	w.Write([]byte(str))

}

func writeHeaderExample(w http.ResponseWriter,r *http.Request)  {
	w.WriteHeader(501)
	fmt.Fprintln(w,"no such service")

}

func main()  {
	server := http.Server{
		Addr: "localhost:8080",
	}
	http.HandleFunc("/write",writeExample)
	http.HandleFunc("/writeheader",writeHeaderExample)

	server.ListenAndServe()
}
```



![image-20221124075147121](image/image-20221124075147121.png)



协议头设置  必须先修改   否则写入后就不可以修改了

```go
package main

import (
	//"log"
	//"fmt"
	"net/http"
)

func redirect(w http.ResponseWriter,r *http.Request)  {
	w.Header().Set("test","123")
	w.WriteHeader(302)

}

func main()  {
	server := http.Server{
		Addr: "localhost:8080",
	}
	http.HandleFunc("/redirect",redirect)

	server.ListenAndServe()


}
```



```go
package main

import (
	json2 "encoding/json"
	"net/http"
)

type Post struct {
	User string
	Threads []string
}
func jsonExample(w http.ResponseWriter,r *http.Request)  {
	w.Header().Set("Content-Type","application/json")
	post := &Post{
		User: "sau sheong",
		Threads: []string{"first","second","third"},
	}
	json,_ := json2.Marshal(post)
	w.Write(json)
}

func main()  {
	server := http.Server{
		Addr: "localhost:8080",
	}
	http.HandleFunc("/json",jsonExample)
	server.ListenAndServe()
}
```



## 内置响应

![image-20221124080800502](image/image-20221124080800502.png)





## 模板

![image-20221124080931124](image/image-20221124080931124.png)



![image-20221124081005124](image/image-20221124081005124.png)



go模板引擎的工作原理



![image-20221124081119786](image/image-20221124081119786.png)



关于模板

![image-20221124081216145](image/image-20221124081216145.png)

![image-20221124081234017](image/image-20221124081234017.png)

![image-20221124081310963](image/image-20221124081310963.png)



```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    {{ . }}
</body>
</html>
```

```go 
package main

import (
	"html/template"
	"net/http"
)


func main()  {
	server := http.Server{
		Addr: "localhost:8080",
	}
	http.HandleFunc("/process",process)
	server.ListenAndServe()
}

func process(w http.ResponseWriter,r *http.Request)  {
	t,_ := template.ParseFiles("templ.html")
	t.Execute(w,"hello world!")

}
```



## 模板执行与解析

![image-20221124082611601](image/image-20221124082611601.png)

![image-20221124082624441](image/image-20221124082624441.png)

```go
import "text/template"

func main(){
    t,_ := template.ParseGlob("*.html")
    
}
```



![image-20221124082332027](image/image-20221124082332027.png)

```go
package main

import (
	"html/template"
	"net/http"
)


func main()  {
	server := http.Server{
		Addr: "localhost:8080",
	}
	http.HandleFunc("/process",process)
	server.ListenAndServe()
}

func process(w http.ResponseWriter,r *http.Request)  {
	t,_ := template.ParseFiles("t1.html")
	t.Execute(w,"hello world!")

	ts,_ := template.ParseFiles("t1.html","t2.html")
	ts.ExecuteTemplate(w,"t2.html","hello world!")

}
```



## demo 模板与解析

```go
package main

import (
	"html/template"
	"log"
	"net/http"
)


func main()  {
	templates := loadTemplates()
	http.HandleFunc("/", func(writer http.ResponseWriter, request *http.Request) {
		fileName := request.URL.Path[1:]
		t := templates.Lookup(fileName)
		if t != nil {
			err := t.Execute(writer,nil)
			if err != nil{
				log.Fatalln(err.Error())
		}else{
			writer.WriteHeader(http.StatusNotFound)
		}
	})	}
	http.HandleFunc("/css/",http.FileServer(http.Dir("wwwroot")))
	http.HandleFunc("/img/",http.FileServer(http.Dir("wwwroot")))
	http.ListenAndServe("localhost:8080",nil)
}

func loadTemplates() *template.Template  {
	result := template.New("templates")
	template.Must(result.ParseGlob("templates/*.html"))
	return result
}
```



## 模板  Action

![image-20221124084020492](image/image-20221124084020492.png)