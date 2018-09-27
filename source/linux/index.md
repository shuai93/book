# linux 基本命令

## curl

cURL是一个利用URL语法在命令行下工作的文件传输工具，1997年首次发行。它支持文件上传和下载，所以是综合传输工具，但按传统，习惯称cURL为下载工具。cURL还包含了用于程序开发的libcurl。

cURL支持的通信协议有FTP、FTPS、HTTP、HTTPS、TFTP、SFTP、Gopher、SCP、Telnet、DICT、FILE、LDAP、LDAPS、IMAP、POP3、SMTP和RTSP。

curl还支持SSL认证、HTTP POST、HTTP PUT、FTP上传, HTTP form based upload、proxies、HTTP/2、cookies、用户名+密码认证(Basic, Plain, Digest, CRAM-MD5, NTLM, Negotiate and Kerberos)、file transfer resume、proxy tunneling。
```bash
# 选项-O 将下载的数据写入到文件，必须使用文件的绝对地址
curl http://apache.fayea.com/kafka/2.0.0/kafka-2.0.0-src.tgz -O


# 选项-o将下载数据写入到指定名称的文件中，并使用--progress显示进度条
curl http://apache.fayea.com/kafka/2.0.0/kafka-2.0.0-src.tgz -o kafka --progress


# curl能够从特定的文件偏移处继续下载，它可以通过指定一个便宜量来下载部分文件
curl URL/File -C 偏移量

#偏移量是以字节为单位的整数，如果让curl自动推断出正确的续传位置使用-C
curl -C -URL

# 重定向
curl --referer http://www.google.com http://www.baidu.com

# cookie 信息
curl  http://www.baidu.com --cookie "user=root;pass=123456"

# cookie 以文件形式存储
curl http://www.baidu.com --cookie-jar cookie_file

# 用curl设置用户代理字符串
curl URL --user-agent "Mozilla/5.0"
curl URL -A "Mozilla/5.0"

# 使用-H"头部信息" 传递多个头部信息
curl -H "Host:man.linuxde.net" -H "accept-language:zh-cn" URL

# curl的带宽控制和下载配额
curl URL --limit-rate 50k

# 只打印响应头部信息
curl -I http://man.linuxde.net
```





## linux 命令终端显示-bash-4.2# 解决方法

近期在折腾docker centos 使用别人的镜像 发现终端显示 `bash-4.2#` 很是忧伤（之前遇到，好记性不如烂笔头）

``` bash
# 这里的root 可以替换成你的用户
cp /etc/skel/.bashrc /root/  
cp /etc/skel/.bash_profile  /root/  

```
然后 exit 重新登录就好了


## linux 文件编辑
在线上操作文件 一般都是 vi 但是 vi 适用于各种花式操作 这里介绍实用的简单的操作

 ```bash
 # echo > file 覆盖文件
echo "" > file  # 这个操作可以快速清空某一文件
echo "Hello World" > file
 # echo >> file 追加文件
echo "Hello World" >> file
# cat 配合 EOF > 覆盖  >> 追加
cat << EOF > file
Hello
Hello world

EOF
```

EOF是END Of File的缩写,表示自定义终止符.既然自定义,那么EOF就不是固定的,可以随意设置别名,在linux按ctrl-d就代表EOF, **EOF一般会配合cat能够多行文本输出.**
