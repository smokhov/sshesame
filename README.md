我是光年实验室高级招聘经理。
我在github上访问了你的开源项目，你的代码超赞。你最近有没有在看工作机会，我们在招软件开发工程师，拉钩和BOSS等招聘网站也发布了相关岗位，有公司和职位的详细信息。
我们公司在杭州，业务主要做流量增长，是很多大型互联网公司的流量顾问。公司弹性工作制，福利齐全，发展潜力大，良好的办公环境和学习氛围。
公司官网是http://www.gnlab.com,公司地址是杭州市西湖区古墩路紫金广场B座，若你感兴趣，欢迎与我联系，
电话是0571-88839161，手机号：18668131388，微信号：echo 'bGhsaGxoMTEyNAo='|base64 -D ,静待佳音。如有打扰，还请见谅，祝生活愉快工作顺利。

# sshesame
A fake SSH server that lets everyone in and logs their activity

## Warning
This software, just like any other, might contain bugs. Given the popular nature of SSH, you probably shouldn't run it unsupervised as root on a production server on port 22. Use common sense.

## Motivation
I was just curious what all these guys were up to:
```
sshd[8128]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=<client>  user=root
sshd[8128]: Failed password for root from <client> port 37510 ssh2
sshd[8128]: Received disconnect from <client> port 37510:11:  [preauth]
sshd[8128]: Disconnected from <client> port 37510 [preauth]
sshd[8141]: Received disconnect from <client> port 59353:11:  [preauth]
sshd[8141]: Disconnected from <client> port 59353 [preauth]
sshd[8151]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=<client>  user=root
sshd[8151]: Failed password for root from <client> port 63785 ssh2
sshd[8159]: Received disconnect from <client> port 24889:11:  [preauth]
sshd[8159]: Disconnected from <client> port 24889 [preauth]
```

## Details
`sshesame` accepts and logs
* every password authentication request,
* every SSH channel open request and
* every SSH request

**without actually executing anything on the host**.

For more details, read the [relevant RFC](https://tools.ietf.org/html/rfc4254).

## Installing
* [Install go](https://golang.org/doc/install) (version 1.4 or newer required)
* `go get -u github.com/jaksi/sshesame`

## Usage
```
$ sshesame -h
Usage of sshesame:
  -host_key string
    	a file containing a private key to use (default "host_key")
  -listen_address string
    	the local address to listen on (default "localhost")
  -port uint
    	the port number to listen on (default 2022)
```
Consider creating a private key to use with sshesame, for example using `ssh-keygen`.

## Example output
```
2016/11/01 18:20:50 Listen: <host>:22
2016/11/01 18:53:05 Login: client=<client>:9224, user="iwan-admin", password="kss123!@#"
2016/11/01 18:53:05 NewChannel: clinet=<client>:9224, type=session, payload=[]
2016/11/01 18:53:05 Request: client=<client>:9224, channel=session, type=shell, payload=[]
2016/11/01 18:53:10 NewChannel: clinet=<client>:9224, type=session, payload=[]
2016/11/01 18:53:10 Request: client=<client>:9224, channel=session, type=exec, payload=[0 0 0 5 117 110 97 109 101]
2016/11/01 18:53:15 EOF
2016/11/01 18:53:15 EOF
2016/11/01 18:53:15 Disconnect: client=<client>:9224
```
So what happened here?
* A client logged in with the user "iwan-admin" and the password "kss123!@#"
* They opened a session channel and requested a shell on it (this is what happens on a "regular" SSH connection)
* They opened another session channel and tried to execute "uname" (the first 4 bytes of the payload store the length, the rest contains the command in ASCII)
* They waited 5 seconds for a response (some input on the channel and an exit-status request, probably)
* They closed both channels and disconnected

Again, if you're interested in the technical details of SSH, read the [RFC](https://tools.ietf.org/html/rfc4254).

## Known issues
* No exit-status request is sent in response to exec requests
* Request payloads are not parsed
* A terminal is created on session channels even if no shell request is received
