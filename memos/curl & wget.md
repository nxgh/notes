- wget 专业的下载器
- curl 模拟提交 web 数据


## wget

### 下载单个文件

```bash
wget [url]
```

### -O 下载并重命名
```bash
wget [url] -O [name]
```
### -c 执行上次未下载完的任务
```bash
wget -c [url]
```

### -b 后台下载
```bash
wget -b [url]
```

### 带宽控制
```bash
wget [url] --limit-rate=300k
```

### 多文件下载
```bash
wget -i 
```

### 递归下载

```sh
wget -r url
```
-   `-r`：递归在下整个站点（www.example.com）资源
-   `-nd`：递归下载时不创建一层一层的目录，把所有的文件下载到当前目录；不指定该选项默认按照资源在站点位置创建相应目录
-   `-np`：递归下载时不搜索上层目录，只在当前路径path2下进行下载；不指定该选项默认搜素整个站点
-   `-A 后缀名`：指定要下载文件的后缀名，多个后缀名之间使用逗号进行分隔
-   `-R 后缀名`：排除要下载文件的后缀名，多个后缀名之间使用逗号进行分隔
-   `-L`：递归时不进入其它主机。不指定该选项的话，如果站点包含了外部站点的链接，这样可能会导致下载内容无限大

## curl

### 单个文件下载
```sh

# 输出到指定文件
curl -o [filename] [url]
# 以 url 最后一个 / 之后的部分作为文件名
curl -O url
```

### 断点下载

```sh
curl -O -C [offset] [url]
# 自动推断正确的续传位置
curl -O -C - [url]
```

### 带宽控制

```sh
# 限定不超过指定的下载速度
curl -O --limit-rate 400k [url]
# 指定最大可下载文件大小
curl -O --max-filesize 300k [url]
```

### web 请求
1. 自动跳转

```sh
curl -L [url]
```
2. 显示响应头信息
```sh
# 仅包含响应头
curl -i [url]
# 
curl -I [url]
```
3. 显示通信过程
```sh
# 显示一次http通信的整个过程，包括端口连接和http request头信息
curl -v [url]
```
4. 指定 http 请求方式
```sh
curl -X [GET|POST|DELETE|PUT|...] [url]
```
5. 添加http 请求头
```sh
curl -H 'key:value' [url]
```
6. 传递请求参数
```sh
curl -X POST -d '参数' [url]
```
-   `-d '参数'`：指定POST请求体。参数形式可以是 "k1=v1&k2=v2", 也可以是json串
-   `--data-urlencode '参数'`：与 `-d` 相同，区别在于会自动将发送的数据进行 URL 编码
7. 文件上传
```sh
curl -F 'file=@FILE' [url]
```
8. 设置来源网址
```sh
curl -e '源网址' [url]
```
9. 设置客户端用户代理
```sh
curl -A 'proxy-info' [url]
```
10. 设置 cookie
```sh
curl -b 'value' [url]
```
11. 设置服务器认证的用户名和密码
```sh
curl -u 'user[:password]' [url]
```

-----------------------------------------------------------
