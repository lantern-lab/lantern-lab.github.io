---
title: 2025第三届数信杯
slug: 2025-the-3rd-shuxin-cup-qxcuc
url: /post/2025-the-3rd-shuxin-cup-qxcuc.html
date: '2026-01-01 18:16:49+08:00'
lastmod: '2026-01-02 03:11:25+08:00'
toc: true
isCJKLanguage: true
---



# 2025第三届数信杯

## 数据处理

背景：随着数据安全行业的大力发展，你作为一名刚毕业的大学生，也打算进入数据安全的行业之 中。在不断的简历投递下，你终于如愿进入到一家物联网公司进行数据安全方向的工作。刚入职的第一 天，你就接到了一系列任务。

### 管理员账号密码

#### 题目

第一天上班的你去查看公司的流量监控记录，发现了一串非常奇怪的流量信息，你一眼就判断出这是黑客攻击所产生的流量，你向上级报告后，上级让你查清黑客获取了什么数据。

[答案标准] 你需要提交泄露的管理员的账号密码

例：管理员账号密码为123/123，则最终提交的答案为：123/123

#### 解答

在报文里找登录成功的

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101182231.png)

### 家庭住址

#### 题目

与你交接的同事由于工作上的疏忽将原先的居民信息文件误删除了，但是你发现公司的系统上依旧存在居民的信息，下载后发现进行了脱敏处理，你需要利用技术手段将居民信息快速整理出来。

[答案标准] 你需要提交手机号为18896239239的家庭住址

例：手机号为18896239239的家庭住址为青海省沈阳市合川徐街9号，则最终提交的答案为： 青海省沈阳市合川徐街9号

#### 解答

发现表格里都是base64编码的

把手机号也base64编码后在表格里搜索

把对应的其他列用base解码

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101182430.png)

### 最大次数

#### 题目

在得到居民信息之后，领导让你统计一下居民信息中重名的数量，方便后续的工作开展。

[答案标准] 你需要统计出现重名次数出现最多的人的姓名以及出现的次数

例：重名最多的人叫张三，出现了10次，则最终提交的答案为：张三10

#### 解答

用excel自带功能处理，筛选后按计数排序

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101182705.png)

将结果base64解码，答案是刘红梅

## **数据泄露**

直接base64解码

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101182843.png)

## 数据恢复溯源

背景：一家科技公司的运维人员在日常巡查的时候，发现一台服务器上出现了异常，这台服务器记录了许多重要的资料，初步排查下来后发现是黑客入侵了服务器并通过技术手段窃取了重要的文件资料。现在公司找到你进行合作，请你根据系统自动保存下来的残存内容溯源整个攻击行为并找到泄露了哪些文件资料。

### 数据恢复

#### 题目

黑客在攻击时，为了对公司造成更大的破坏，直接删除了磁盘中的文件。但好在系统有自动的磁盘备份计划，保留了一个备份磁盘。请你通过技术手段，恢复出黑客删除的文件。

[答案标准] 请找出删除的文件中的一个合同文件，提交合同编号。

例：如果合同编号为flag{xx-xx-xx}，则最终提交答案为：xx-xx-xx

#### 解答

查看disk.img文件类型，用testdisk提取文件

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101183404.png)

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101183440.png)

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101183500.png)

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101183516.png)

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101183541.png)

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101183610.png)

可以看到有三个文件，根据提示，把这三个文件都提取出来。

我的kali显示不了中文，PDF文件名其实是“商业合作合同.pdf”，打开后获得答案

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101183757.png)

### 数据存储安全

#### 题目

你通过技术手段恢复出黑客删除的文件后，发现存在加密文件，但是管理员忘记了密钥，但经验丰富的你知道可以绕开验证直接读取加密文件中的内容。请你通过技术手段，读取加密内容。

[答案标准] 请读取加密文件中的密码本，提交密码本中的内容

例：如果密码本中的内容为flag{xxxxxx}，则最终提交答案为：xxxxxx

#### 解答

上题中获得的还有一个raw文件和一个data文件。data文件大小刚好512MB，根据题目提示，猜测data文件就是加密文件。

先用内存取证工具读取raw文件。

**坑点！！！**

这里有个坑点，我用windows.filescan扫描后发现里面有一个“密码本.txt”，但就是提取不出来。其实答案的“密码本.txt”在data里，不在raw里。

查看进程信息，发现有TrueCrypt，猜测data文件就是用它加密的。

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101184923.png)

这里使用volatility2的truecryptmaster脚本从raw文件中提取出masterkey，注意volatility2需要用python2。

```plaintext
python2 "D:\SecSpace\mem\Lovelymem\Tools\volatility2_python\vol.py" -f "C:\Users\33113\Desktop\数信杯\disk\recovered\WIN-SERVER-PC-20251202-122722.raw" --profile=Win7SP1x64 truecryptmaster -D  "C:\Users\33113\Desktop\数信杯\disk\recovered"
```

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101185225.png)

接下来用获取的masterkey解密data，注意truecrypt不支持用masterkey解密

Windows平台可用以下脚本获取一个解密后的img文件，然后用testdisk提取里面的内容

```python
#!/usr/bin/env python3
"""
TrueCrypt 解密脚本 - 使用 cryptography 库的原生 XTS 实现
这是最快的 Python 实现方式
"""

import os
import sys
import struct
import time
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes

def decrypt_truecrypt_fast(encrypted_file, key_file, output_file, offset=256):
    """使用 cryptography 库的原生 XTS 解密 - 极速版本"""
    print(f"\n=== TrueCrypt 解密工具 (极速版) ===\n")
    
    start_time = time.time()
    
    # 读取 master key
    with open(key_file, 'rb') as f:
        master_key = f.read()
    
    print(f"Master key: {len(master_key)} 字节")
    
    if len(master_key) < 64:
        master_key = master_key + b'\x00' * (64 - len(master_key))
    
    # cryptography 的 XTS 需要完整的 64 字节密钥
    # 格式: key1 (32 bytes) + key2 (32 bytes)
    # 但顺序与 TrueCrypt 相反：tweak key 在前，data key 在后
    # TrueCrypt: data_key (32) + tweak_key (32)
    # cryptography XTS: key = data_key + tweak_key (相同顺序)
    
    file_size = os.path.getsize(encrypted_file)
    print(f"文件大小: {file_size / (1024*1024):.2f} MB")
    
    sector_size = 512
    data_offset = offset * sector_size
    data_size = file_size - data_offset
    
    # 使用 64 MB 块以最大化性能
    chunk_size = 64 * 1024 * 1024
    total_chunks = (data_size + chunk_size - 1) // chunk_size
    
    print(f"块大小: {chunk_size // (1024*1024)} MB")
    print(f"\n开始解密...")
    
    with open(encrypted_file, 'rb') as f_in:
        with open(output_file, 'wb') as f_out:
            f_in.seek(data_offset)
            
            current_sector = offset
            processed = 0
            bytes_processed = 0
            
            while True:
                chunk = f_in.read(chunk_size)
                if not chunk:
                    break
                
                # 逐扇区解密（XTS 的 tweak 基于扇区号）
                decrypted_chunk = bytearray()
                
                for i in range(0, len(chunk), sector_size):
                    sector_data = chunk[i:i + sector_size]
                    if len(sector_data) < sector_size:
                        sector_data = sector_data + b'\x00' * (sector_size - len(sector_data))
                    
                    # 创建 tweak (128 位小端序扇区号)
                    tweak = struct.pack('<QQ', current_sector, 0)
                    
                    # 使用 cryptography 的 XTS 模式
                    cipher = Cipher(algorithms.AES(master_key), modes.XTS(tweak))
                    decryptor = cipher.decryptor()
                    decrypted_sector = decryptor.update(sector_data) + decryptor.finalize()
                    
                    decrypted_chunk.extend(decrypted_sector)
                    current_sector += 1
                
                f_out.write(decrypted_chunk)
                
                bytes_processed += len(chunk)
                processed += 1
                
                progress = (bytes_processed / data_size) * 100
                elapsed = time.time() - start_time
                speed = (bytes_processed / (1024*1024)) / elapsed if elapsed > 0 else 0
                eta = (data_size - bytes_processed) / (bytes_processed / elapsed) if bytes_processed > 0 else 0
                print(f"进度: {progress:.1f}% | 速度: {speed:.1f} MB/s | 剩余: {eta:.0f}s", end='\r')
    
    elapsed = time.time() - start_time
    avg_speed = (data_size / (1024*1024)) / elapsed
    print(f"\n\n✓ 解密完成!")
    print(f"用时: {elapsed:.1f} 秒")
    print(f"平均速度: {avg_speed:.1f} MB/s")
    print(f"输出���件: {output_file}")
    
    # 检查文件系统
    print(f"\n=== 检查文件系统 ===")
    with open(output_file, 'rb') as f:
        header = f.read(4096)
        if header[3:8] == b'NTFS ':
            print("检测到: NTFS")
        elif header[3:8] == b'MSDOS':
            print("检测到: FAT32/FAT16")
        elif header[3:8] == b'EXFAT':
            print("检测到: exFAT")
        elif len(header) > 1082 and header[1080:1082] == b'\x53\xef':
            print("检测到: EXT2/3/4")
        else:
            print(f"文件头 (hex): {header[:64].hex()}")
            if header[:512] == b'\x00' * 512:
                print("警告: 文件头全为零，可能解密失败")

def main():
    script_dir = os.path.dirname(os.path.abspath(__file__))
    
    encrypted_file = os.path.join(script_dir, 'data')
    key_file = os.path.join(script_dir, '0xfffffa8018ea11a8_master.key')
    output_file = os.path.join(script_dir, 'decrypted_data.img')
    
    if not os.path.exists(encrypted_file):
        print(f"错误: 找不到 {encrypted_file}")
        sys.exit(1)
    
    if not os.path.exists(key_file):
        print(f"错误: 找不到 {key_file}")
        sys.exit(1)
    
    decrypt_truecrypt_fast(encrypted_file, key_file, output_file)

if __name__ == '__main__':
    main()

```

Linux平台可以在github下载[MKDecrypt.py](https://github.com/AmNe5iA/MKDecrypt)，把data挂载到本地查看

```python
python MKDecrypt.py -X -m /mnt data 0xfffffa8018ea11a8_master.key
```

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101185954.png)

### 流量分析（没做出来）

#### 题目

你在破解开加密文件后，发现加密文件中存放着一个流量包，通过分析你发现流量包中记录着黑客攻击时产生的流量信息。你需要分析流量包，找出泄露的内容。

[答案标准] 请读取泄露的内容，提交手机号为：18316978925的家庭住址信息

例：手机号18316978925的家庭住址为：四川省成都市锦江区学府路159号和谐苑23栋5单元22室，则最终提交答案为：四川省成都市锦江区学府路159号和谐苑23栋5单元22室

#### 解答

流量包就是上题获得的challenge.zip，但这题我未解出。

## **数据隐写**

### 提取信息

#### 题目

隐写规则提示：

1. 图片的红色（R）通道中隐藏了模型输入特征；
2. 取图片左上角前20个像素的R值，计算 R值 mod 10 得到20维特征；
3. 20维特征输入multimodal_model.pth模型后，输出的数值取整即为flag的ASCII码；
4. ASCII码转换为字符即可得到完整flag。

模型提示：

- 模型为轻量全连接神经网络（MLP），仅含3层线性层+ReLU激活。

#### 解答

先查看模型结构

```python
import torch

# Load the model state dictionary
model_path = "multimodal_model.pth"
try:
    # weights_only=False is needed for older pytorch versions or specific save formats, 
    # though strictly for weights it might not be needed, keeping it safe as per user's solve.py
    state_dict = torch.load(model_path, map_location='cpu') 
    
    print(f"Content of {model_path}:")
    for key, value in state_dict.items():
        if isinstance(value, torch.Tensor):
            print(f"{key}: {value.shape}")
        else:
            print(f"{key}: {type(value)}")

except Exception as e:
    print(f"Error loading file: {e}")

```

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101192846.png)

编写py脚本取数

```python
import torch
import torch.nn as nn
from PIL import Image
import numpy as np

# 定义完全相同的MLP模型结构（根据实际权重：20->64->32->27）
class MultimodalModel(nn.Module):
    def __init__(self):
        super(MultimodalModel, self).__init__()
        self.fc1 = nn.Linear(20, 64)
        self.relu1 = nn.ReLU()
        self.fc2 = nn.Linear(64, 32)
        self.relu2 = nn.ReLU()
        self.fc3 = nn.Linear(32, 27)
    
    def forward(self, x):
        x = self.fc1(x)
        x = self.relu1(x)
        x = self.fc2(x)
        x = self.relu2(x)
        x = self.fc3(x)
        return x

# 1. 加载图片并提取R通道
image_path = r"c:\Users\33113\Desktop\数信杯\数据隐写\secret_image.png"
img = Image.open(image_path)
img_array = np.array(img)

# 获取R通道
r_channel = img_array[:, :, 0]  # R通道

# 隐写信息在第一列！取前20行的第一列像素
r_values = r_channel[:20, 0]
print("左上角前20行第一列的R值:", r_values)

# 2. 计算 R值 mod 10 得到20维特征
features = r_values % 10
print("特征 (R mod 10):", features)

# 3. 加载模型
model_path = r"c:\Users\33113\Desktop\数信杯\数据隐写\multimodal_model.pth"
model = MultimodalModel()
state_dict = torch.load(model_path, map_location='cpu', weights_only=False)
model.load_state_dict(state_dict)
model.eval()

# 4. 将特征输入模型
input_tensor = torch.tensor(features, dtype=torch.float32).unsqueeze(0)
with torch.no_grad():
    output = model(input_tensor)

# 5. 输出取整得到ASCII码
ascii_codes = output.squeeze().numpy().round().astype(int)
print("ASCII码:", ascii_codes)

# 6. 转换为字符
flag = ''.join([chr(code) for code in ascii_codes if 32 <= code <= 126])
print("Flag:", flag)

```

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101192929.png)

## 流量数据分析

某公司运维团队在日常安全巡检过程中发现系统存在异常访问和潜在入侵迹象。经初步技术研判，问题可能与公司用于关键服务器相关。该服务器被确认存在一定安全风险，但目前尚无法确定攻击者是否已成功获取其中的证书信息。为进一步排查与分析潜在的数据泄漏情况，运维团队已导出该服务器的所有网络流量文件，请根据流量包文件分析并回答以下的问题。

### 证书合成

#### 题目

请根据题目提供的证书关键参数，合成私钥解密证书。请选手找到id为285的参数合成的证书(参考附件params.csv)，可以解密哪个流量包(参考附件:pcap.zip)。并将其流量包名称作为答案提交。  
【答案标准】  
若id为285的参数合成证书，可以解密"UT5NHVWo2Z.pcap"，则答案提交为UT5NHVWo2Z.pcap

#### 解答

查看一个流量包内容，发现是先做了TLS密钥交换，然后用加密形式传数据

查看params.csv文件，发现是rsa的e、p、q值

已知在TLS密钥交换的时候，会传递rsa模数n的值

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101203927.png)

因此可以通过n，找到id为285的参数可以解密哪个流量包。

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import os
import glob
from scapy.all import rdpcap, Raw
import warnings
from Crypto.Util.number import *
warnings.filterwarnings("ignore")

# 路径配置
pcap_dir = r"c:\Users\33113\Desktop\数信杯\数据溯源\pcap"

# RSA参数 (id=285)
e = 65537
p = 177264302295959185550899884811457697789837321132319354039496340545988969470422347313577084568610012957139649359576035974322283705879187577664768699213211347033624840318251940972496063336844685896882713624561971974788692556498019960846465311267474369690812099681875735569564330504277517754796899917257323134723
q = 143990163909936129648804807321551478733567016733642335522156625973321506509458427490929508866189002322826874210961910641865602374675333206288577734876005828016379170951078934469472423840724164310343028554942665656763806178050620857070820746559094989518930589089970082522430002916286799917078785172333647028571

# 计算n
n = p * q
print(f"n = {n}\n")

# 获取所有pcap文件
pcap_files = glob.glob(os.path.join(pcap_dir, "*.pcap"))
print(f"找到 {len(pcap_files)} 个pcap文件")

matches = []

for i, pcap_file in enumerate(pcap_files):
    pcap_name = os.path.basename(pcap_file)
    try:
        # 读取pcap文件
        packets = rdpcap(pcap_file)
        # 遍历所有数据包
        for pkt in packets:
            # 检查是否有Raw层,即载荷数据
            if pkt.haslayer(Raw):
                # 获取载荷数据
                raw_data = bytes(pkt[Raw])
                # 检查模数n是否在载荷数据中
                n_bytes = long_to_bytes(n)
                if n_bytes in raw_data:
                    print(f"[匹配!] {pcap_name}: 找到匹配的模数")
                    matches.append(pcap_name)
                    break
    except Exception as e:
        pass
    
    if (i + 1) % 100 == 0:
        print(f"已处理 {i+1}/{len(pcap_files)} 个文件...")

print(f"\n=== 匹配的文件 ===")
for m in matches:
    print(m)

```

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101204055.png)

### 证书解密

#### 题目

请根据上面题目对每个流量包进行解密，并得到明文。请选手找到"王梅"的手机号，并进行答案提交。

【答案标准】若找到"王梅"的手机号为15306364252，则答案提交为15306364252

#### 解答

```python
"""
HTTPS 流量解密脚本（基于已知 RSA 私钥参数 e/p/q）

【原理说明】
TLS/HTTPS 握手中，客户端生成48字节预主密钥(Pre-Master Secret)，用服务器RSA公钥加密后发送。
已知RSA参数(e,p,q)可以：
  1. 计算私钥 d = e^(-1) mod φ(n)
  2. 解密预主密钥
  3. 派生会话密钥并解密应用数据

【TLS 1.2 密钥派生】
  Pre-Master Secret → PRF("master secret") → Master Secret → PRF("key expansion") → Key Block
"""

import os
import re
import csv
import glob
import hmac
import struct
import hashlib
import warnings
from Crypto.Util.number import inverse, long_to_bytes, bytes_to_long
from Crypto.Cipher import AES
from scapy.all import rdpcap, Raw

warnings.filterwarnings("ignore")

# ========================= 配置 =========================
PCAP_DIR = r"C:\Users\33113\Desktop\数信杯\数据溯源\pcap"
PARAMS_FILE = r"C:\Users\33113\Desktop\数信杯\数据溯源\params.csv"


# ========================= TLS PRF =========================
def prf_sha256(secret, label, seed, length):
    """TLS 1.2 PRF (P_SHA256): 迭代HMAC生成指定长度的密钥材料"""
    result, a = b'', hmac.new(secret, label + seed, hashlib.sha256).digest()
    while len(result) < length:
        result += hmac.new(secret, a + label + seed, hashlib.sha256).digest()
        a = hmac.new(secret, a, hashlib.sha256).digest()
    return result[:length]


# ========================= RSA参数加载 =========================
def load_rsa_params(params_file):
    """加载RSA参数，预计算私钥d，返回 (params字典, n_bytes映射)"""
    params, n_bytes_map = {}, {}
    with open(params_file, 'r') as f:
        for row in csv.DictReader(f):
            id_, e, p, q = int(row['id']), int(row['e']), int(row['p']), int(row['q'])
            n = p * q
            d = inverse(e, (p - 1) * (q - 1))
            params[id_] = {'n': n, 'd': d}
            n_bytes_map[long_to_bytes(n)] = id_
    return params, n_bytes_map


# ========================= TLS解析 =========================
def parse_tls_records(data):
    """
    解析TLS记录层
    格式: ContentType(1) + Version(2) + Length(2) + Payload
    ContentType: 22=握手, 23=应用数据
    """
    records, offset = [], 0
    while offset < len(data) - 5:
        ctype, length = data[offset], struct.unpack('>H', data[offset+3:offset+5])[0]
        if offset + 5 + length > len(data):
            break
        records.append({'type': ctype, 'payload': data[offset+5:offset+5+length]})
        offset += 5 + length
    return records


def parse_handshakes(payload):
    """
    解析握手消息
    格式: Type(1) + Length(3) + Data
    Type: 1=ClientHello, 2=ServerHello, 16=ClientKeyExchange
    """
    messages, offset = [], 0
    while offset < len(payload) - 4:
        hs_type = payload[offset]
        length = struct.unpack('>I', b'\x00' + payload[offset+1:offset+4])[0]
        messages.append({'type': hs_type, 'data': payload[offset+4:offset+4+length]})
        offset += 4 + length
    return messages


def extract_tls_data(packets):
    """从数据包中提取TLS握手数据和应用数据"""
    client_random = server_random = encrypted_pms = cipher_suite = None
    app_data_list = []
    
    for pkt in packets:
        if not pkt.haslayer(Raw):
            continue
        for record in parse_tls_records(bytes(pkt[Raw])):
            if record['type'] == 22:  # 握手
                for hs in parse_handshakes(record['payload']):
                    data = hs['data']
                    if hs['type'] == 1 and len(data) >= 34:  # ClientHello
                        client_random = data[2:34]
                    elif hs['type'] == 2 and len(data) >= 34:  # ServerHello
                        server_random = data[2:34]
                        if len(data) >= 37:
                            sid_len = data[34]
                            if len(data) >= 37 + sid_len:
                                cipher_suite = struct.unpack('>H', data[35+sid_len:37+sid_len])[0]
                    elif hs['type'] == 16 and len(data) >= 2:  # ClientKeyExchange
                        pms_len = struct.unpack('>H', data[0:2])[0]
                        if len(data) >= 2 + pms_len:
                            encrypted_pms = data[2:2+pms_len]
            elif record['type'] == 23:  # 应用数据
                app_data_list.append(record['payload'])
    
    return client_random, server_random, encrypted_pms, cipher_suite, app_data_list


# ========================= RSA解密 =========================
def rsa_decrypt_pms(encrypted_pms, n, d):
    """RSA解密预主密钥，去除PKCS#1 v1.5填充 (00 02 [padding] 00 [data])"""
    m = pow(bytes_to_long(encrypted_pms), d, n)
    decrypted = long_to_bytes(m)
    
    # 去除PKCS#1 v1.5填充
    if len(decrypted) > 11 and decrypted[:2] == b'\x00\x02':
        sep = decrypted.find(b'\x00', 2)
        pms = decrypted[sep+1:] if sep >= 10 else decrypted[-48:]
    else:
        pms = decrypted[-48:] if len(decrypted) >= 48 else decrypted
    
    return pms if len(pms) == 48 else None


# ========================= 会话密钥派生 =========================
def derive_keys(pms, client_random, server_random, is_gcm):
    """派生主密钥和会话密钥"""
    master = prf_sha256(pms, b"master secret", client_random + server_random, 48)
    
    if is_gcm:
        # AES-GCM: key(16) + key(16) + iv(4) + iv(4) = 40 bytes
        kb = prf_sha256(master, b"key expansion", server_random + client_random, 40)
        return kb[0:16], kb[16:32], kb[32:36], kb[36:40]
    else:
        # AES-CBC: mac(20) + mac(20) + key(16) + key(16) = 72 bytes (TLS 1.1+ 使用显式IV)
        kb = prf_sha256(master, b"key expansion", server_random + client_random, 72)
        return kb[40:56], kb[56:72], None, None


# ========================= 应用数据解密 =========================
def decrypt_app_data(app_data_list, client_key, server_key, client_iv, server_iv, is_gcm):
    """解密应用层数据"""
    results = []
    
    for data in app_data_list:
        if len(data) < 16:
            continue
        
        if is_gcm:
            from cryptography.hazmat.primitives.ciphers.aead import AESGCM
            explicit_nonce, ciphertext = data[:8], data[8:]
            
            for key, iv in [(client_key, client_iv), (server_key, server_iv)]:
                nonce = iv + explicit_nonce
                for seq in range(20):  # 尝试不同序列号
                    aad = struct.pack('>Q', seq) + b'\x17\x03\x03' + struct.pack('>H', len(ciphertext) - 16)
                    try:
                        results.append(AESGCM(key).decrypt(nonce, ciphertext, aad).decode('utf-8', errors='ignore'))
                        break
                    except:
                        pass
                else:
                    continue
                break
        else:
            # AES-CBC (TLS 1.1+ 显式IV)
            iv, ciphertext = data[:16], data[16:]
            for key in [client_key, server_key]:
                try:
                    decrypted = AES.new(key, AES.MODE_CBC, iv).decrypt(ciphertext)
                    pad = decrypted[-1]
                    if 0 < pad <= 16 and all(b == pad for b in decrypted[-pad:]):
                        plaintext = decrypted[:-pad-20]  # 去除填充和MAC(SHA1=20)
                        results.append(plaintext.decode('utf-8', errors='ignore'))
                        break
                except:
                    pass
    
    return results


# ========================= 核心解密函数 =========================
def decrypt_pcap(pcap_file, params, n_bytes_map):
    """解密单个pcap文件"""
    packets = rdpcap(pcap_file)
    
    # 收集原始数据并匹配RSA参数
    all_raw = b''.join(bytes(pkt[Raw]) for pkt in packets if pkt.haslayer(Raw))
    if not all_raw:
        return None
    
    matched_id = next((id_ for n_bytes, id_ in n_bytes_map.items() if n_bytes in all_raw), None)
    if not matched_id:
        return None
    
    n, d = params[matched_id]['n'], params[matched_id]['d']
    
    # 提取TLS数据
    client_random, server_random, encrypted_pms, cipher_suite, app_data_list = extract_tls_data(packets)
    if not all([client_random, server_random, encrypted_pms, app_data_list]):
        return None
    
    # 解密预主密钥
    pms = rsa_decrypt_pms(encrypted_pms, n, d)
    if not pms:
        return None
    
    # 派生密钥并解密
    is_gcm = cipher_suite in [0x009C, 0x009D]
    client_key, server_key, client_iv, server_iv = derive_keys(pms, client_random, server_random, is_gcm)
    
    return decrypt_app_data(app_data_list, client_key, server_key, client_iv, server_iv, is_gcm)


# ========================= 主程序 =========================
if __name__ == "__main__":
    print("📖 读取RSA参数...")
    params, n_bytes_map = load_rsa_params(PARAMS_FILE)
    print(f"✅ 共读取 {len(params)} 组RSA参数")
    
    pcap_files = glob.glob(os.path.join(PCAP_DIR, "*.pcap"))
    print(f"📁 找到 {len(pcap_files)} 个pcap文件")
    
    all_results, success = [], 0
    
    for i, pcap_file in enumerate(pcap_files):
        name = os.path.basename(pcap_file)
        try:
            texts = decrypt_pcap(pcap_file, params, n_bytes_map)
            if texts:
                success += 1
                for text in texts:
                    all_results.append((name, text))
                    if '王梅' in text or '\\u738b\\u6885' in text:
                        print(f"\n🎯 [找到目标!] {name}\n{text[:500]}")
                        match = re.search(r'"phone":\s*"(\d+)"', text)
                        if match:
                            print(f"📱 手机号: {match.group(1)}")
        except:
            pass
        
        if (i + 1) % 50 == 0:
            print(f"⏳ 已处理 {i+1}/{len(pcap_files)}，成功 {success} 个...")
    
    print(f"\n{'='*60}")
    print(f"✅ 完成！处理 {len(pcap_files)} 文件，成功 {success} 个，共 {len(all_results)} 条记录")
    
    output = os.path.join(os.path.dirname(PCAP_DIR), "all_decrypted_new.txt")
    with open(output, 'w', encoding='utf-8') as f:
        for name, text in all_results:
            f.write(f"=== {name} ===\n{text}\n\n")
    print(f"📄 已保存到 {output}")

```

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101214352.png)

### 数据核验

#### 题目

为了安全可靠，该系统在请求包设置了Authorization头部字段。该字段是使用JWT进行加密的，每个人的密钥为了安全考虑设置成自己姓名的拼音。请选手解密JWT身份认证核对信息是否与请求体中的内容是否一致。若非一致，该用户可能在进行越权访问。请选手统计出有多少个账号进行越权。  
【答案标准】  
若有101个账号进行越权，则答案提交为101

#### 解答

```python
"""
JWT 越权访问检测脚本

【功能说明】
检测HTTPS请求中是否存在越权访问：
  - 解析解密后的HTTP请求
  - 比较JWT令牌中的用户信息与请求体中的用户信息
  - 如果不一致，说明用户A使用自己的JWT访问用户B的数据（越权）

【JWT结构】
JWT由三部分组成，用.分隔：Header.Payload.Signature
  - Header: {"alg": "HS256", "typ": "JWT"}
  - Payload: {"username": "xxx", "phone": "xxx"} (用户信息)
  - Signature: HMAC-SHA256签名

【越权判定】
若 JWT.payload.username ≠ Body.username 或 JWT.payload.phone ≠ Body.phone
则判定为越权访问（用户试图修改他人数据）
"""

import re
import json
import base64
import hashlib


def base64url_decode(data):
    """Base64URL解码（自动补齐padding）"""
    padding = 4 - len(data) % 4
    if padding != 4:
        data += '=' * padding
    return base64.urlsafe_b64decode(data)


def decode_jwt_payload(token):
    """
    解码JWT的Payload部分（不验证签名）
    
    JWT格式: header.payload.signature
    Payload是Base64URL编码的JSON
    """
    try:
        parts = token.split('.')
        if len(parts) != 3:
            return None
        return json.loads(base64url_decode(parts[1]).decode('utf-8'))
    except:
        return None


def parse_decrypted_file(content):
    """
    解析解密文件，提取每个pcap对应的请求内容
    
    文件格式:
        === xxx.pcap ===
        HTTP请求内容...
        
        === yyy.pcap ===
        HTTP请求内容...
    
    返回: [(pcap_name, request_content), ...]
    """
    pattern = r'=== ([^=]+\.pcap) ==='
    parts = re.split(pattern, content)
    
    # parts结构: ['', pcap1, content1, pcap2, content2, ...]
    return [(parts[i].strip(), parts[i + 1]) 
            for i in range(1, len(parts), 2) 
            if i + 1 < len(parts)]


def extract_jwt_and_body(request):
    """
    从HTTP请求中提取JWT令牌和JSON请求体
    
    JWT位置: Authorization: Bearer <token>
    请求体: {"username": "xxx", "phone": "xxx"}
    """
    # 提取JWT
    jwt_match = re.search(r'Authorization: Bearer ([^\s\r\n]+)', request)
    jwt_token = jwt_match.group(1) if jwt_match else None
    
    # 提取JSON请求体
    body_match = re.search(r'\{[^{}]+\}', request)
    try:
        body = json.loads(body_match.group(0)) if body_match else None
    except:
        body = None
    
    return jwt_token, body


def check_authorization(jwt_token, body):
    """
    检查是否存在越权访问
    
    比较JWT中的用户信息与请求体中的用户信息
    返回: (is_unauthorized, jwt_info, body_info)
    """
    payload = decode_jwt_payload(jwt_token)
    if not payload:
        return None, None, None
    
    jwt_info = (payload.get('username', ''), payload.get('phone', ''))
    body_info = (body.get('username', ''), body.get('phone', ''))
    
    # 越权判定：JWT中的用户信息与请求体不一致
    is_unauthorized = jwt_info != body_info
    
    return is_unauthorized, jwt_info, body_info


def main():
    # 读取解密后的请求数据
    with open('all_decrypted_new.txt', 'r', encoding='utf-8') as f:
        content = f.read()
    
    requests = parse_decrypted_file(content)
    print(f"📂 共解析 {len(requests)} 个请求")
    
    unauthorized_list = []
    
    for pcap_name, request_content in requests:
        jwt_token, body = extract_jwt_and_body(request_content)
        
        # 跳过无法解析的请求
        if not jwt_token or not body:
            continue
        
        is_unauthorized, jwt_info, body_info = check_authorization(jwt_token, body)
        
        if is_unauthorized is None:
            print(f"[ERROR] {pcap_name}: JWT解码失败")
            continue
        
        if is_unauthorized:
            unauthorized_list.append({
                'pcap': pcap_name,
                'jwt': {'username': jwt_info[0], 'phone': jwt_info[1]},
                'body': {'username': body_info[0], 'phone': body_info[1]}
            })
            print(f"🚨 [越权] {pcap_name}")
            print(f"   JWT:  username={jwt_info[0]}, phone={jwt_info[1]}")
            print(f"   Body: username={body_info[0]}, phone={body_info[1]}")
    
    # 输出统计结果
    count = len(unauthorized_list)
    print(f"\n{'='*50}")
    print(f"✅ 检测完成！发现 {count} 个越权访问")
    print(f"🔑 答案MD5: {hashlib.md5(str(count).encode()).hexdigest()}")


if __name__ == '__main__':
    main()

```

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101214433.png)

## 文档安全事件分析

随着维真科技(VerityTech)业务规模的不断扩张，公司积累了大量内部 PDF 文档，包括重要的研发资料、业务报告、内部制度说明等。在长期的运作过程中，由于文档管理规范不完善、缺乏统一的安全策略，公司文档的安全风险逐渐显现。  
近期，公司文档服务器遭遇未知勒索程序攻击。部分 PDF 文件被加密无法打开，系统中还出现可疑链接的文档，疑似为攻击入口。同时，安全团队在受感染终端中提取到了一份可疑的可执行程序(exe 文件)，推测为勒索程序主体。  
为了定位安全漏洞、恢复文档数据、查明攻击来源，公司启动了“文档安全事件分析”:专项工作。本题给出攻击样本、加密文档及疑似钓鱼 PDF 文档，请你协助安全团队进行取证分析、密钥恢复、数据解密以及钓鱼链接识别。

公司合法域名为:veritytech.com以及其子域:  
*.veritytech.com其余域名均视为可疑链接，需重点审查。

### 密钥取证

#### 题目

安全团队提取到疑似勒索程序(exe 文件)，该程序采用"混合加密机制"对 PDF 文档进行加密使用

- RSA 公钥加密 AES 密钥(用于安全存储/传输 AES 密钥);
- 使用原始的 AES 密钥(明文)直接加密 PDF 文档。

若要恢复文档，必须从勒索程序中获取攻击者遗留的 RSA 私钥。

请你分析勒索程序，找到私钥，并将私钥中"----BEGIN PRIVATE KEY--……"之后的32为字符当做答案提交。

#### 解答

用记事本打开exe文件，发现里面有python关键字，判断为python打包的exe文件

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101225207.png)

使用pyinstxtractor.py解包，找到key

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101225315.png)

```python
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCt4TrETrlzU4NV
...
```

### 数据解密

#### 题目

在成功获得攻击者的 RSA 私钥后，可解密 AES 密钥，并进一步解密被勒索的 PDF 文件。请使用正确的私钥恢复文件内容。  
请计算"0999_内部文档0999.pdf"解密后,文件的 MD5 值，并提交其 32 位小写 MD5 作为答案。

#### 解答

使用pycdc.exe反编译en.exe_extracted\encrypt_files.pyc，查看使用的是AES哪种模式

```python
# pycdc.exe encrypt_files.pyc > encrypt.py
def encrypt_file_aes_ecb(file_path, aes_key, output_path = (None,)):
    '''
    使用AES-256-ECB算法加密文件
    
    Args:
        file_path: 要加密的文件路径
        aes_key: AES密钥（32字节）
        output_path: 输出文件路径，如果为None则添加.enc后缀
        
    Returns:
        加密后的文件路径
    '''
    with open(file_path, 'rb') as f:
        plaintext = f.read()
        None(None, None, None)
```

编写解密代码

```python
"""
批量文件解密程序
使用RSA私钥解密AES密钥，然后使用AES-256-ECB算法解密文件
"""
import os
from pathlib import Path
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.primitives import padding, hashes
from cryptography.hazmat.primitives.asymmetric import padding as asym_padding
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.backends import default_backend


def load_rsa_private_key(private_key_file):
    """
    从文件加载RSA私钥
    """
    with open(private_key_file, 'rb') as f:
        private_key = serialization.load_pem_private_key(
            f.read(),
            password=None,
            backend=default_backend()
        )
    return private_key


def decrypt_aes_key_with_rsa(encrypted_key_file, private_key):
    """
    使用RSA私钥解密AES密钥
    """
    with open(encrypted_key_file, 'rb') as f:
        encrypted_key = f.read()
    
    # 使用OAEP padding解密 (与加密时使用的相同)
    aes_key = private_key.decrypt(
        encrypted_key,
        asym_padding.OAEP(
            mgf=asym_padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )
    return aes_key


def decrypt_file_aes_ecb(file_path, aes_key, output_path=None):
    """
    使用AES-256-ECB算法解密文件
    """
    with open(file_path, 'rb') as f:
        ciphertext = f.read()
    
    # 创建AES-ECB解密器
    cipher = Cipher(algorithms.AES(aes_key), modes.ECB(), backend=default_backend())
    decryptor = cipher.decryptor()
    
    # 解密数据
    padded_plaintext = decryptor.update(ciphertext) + decryptor.finalize()
    
    # 移除PKCS7填充
    unpadder = padding.PKCS7(128).unpadder()
    plaintext = unpadder.update(padded_plaintext) + unpadder.finalize()
    
    # 确定输出路径
    if output_path is None:
        # 移除.enc后缀
        if str(file_path).endswith('.enc'):
            output_path = str(file_path)[:-4]
        else:
            output_path = str(file_path) + '.dec'
    
    with open(output_path, 'wb') as f:
        f.write(plaintext)
    
    return output_path


def batch_decrypt_files(directory, private_key_file, encrypted_key_file):
    """
    批量解密指定目录下的.enc文件
    """
    print(f"正在加载RSA私钥: {private_key_file}")
    private_key = load_rsa_private_key(private_key_file)
    
    print(f"正在解密AES密钥: {encrypted_key_file}")
    aes_key = decrypt_aes_key_with_rsa(encrypted_key_file, private_key)
    print(f"AES密钥已解密，长度: {len(aes_key)} 字节")
    
    # 保存解密后的AES密钥
    with open("decrypted_aes.key", 'wb') as f:
        f.write(aes_key)
    print("AES密钥已保存到 decrypted_aes.key")
    
    # 查找所有.enc文件
    directory_path = Path(directory)
    enc_files = list(directory_path.glob("**/*.enc"))
    
    if not enc_files:
        print("未找到需要解密的.enc文件")
        return
    
    print(f"\n找到 {len(enc_files)} 个文件需要解密")
    print("-" * 50)
    
    decrypted_count = 0
    for enc_file in enc_files:
        try:
            output_path = decrypt_file_aes_ecb(enc_file, aes_key)
            print(f"已解密: {enc_file.name} -> {Path(output_path).name}")
            decrypted_count += 1
        except Exception as e:
            print(f"解密失败: {enc_file.name} - {e}")
    
    print("-" * 50)
    print(f"解密完成! 成功解密 {decrypted_count}/{len(enc_files)} 个文件")


if __name__ == "__main__":
    # 设置路径
    base_dir = r"c:\Users\33113\Desktop\数信杯\enpdf"
    private_key_file = os.path.join(base_dir, "key", "pr.pem")
    encrypted_key_file = os.path.join(base_dir, "store.key")
    pdfs_directory = os.path.join(base_dir, "pdfs")
    
    batch_decrypt_files(pdfs_directory, private_key_file, encrypted_key_file)

```

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260101234131.png)

提交该文档md5值即可

### 钓鱼链接💩💩💩

#### 题目

在事件调查中，安全团队统计出共计 999 份 PDF 文档，其中部分文档包含内部外链。为避免安全风险，需对所有文档进行链接审查。  
PDF 中的 URL 可能以以下形式出现:  
	1. 文本中直接显示的 URL  
	2. 隐藏在 PDF Annotation 超链接内的 URL(显示内容与真实跳转地址可能不同)  
	3. URL 参数中的跳转地址:某些 URL 可能包含 redirect、next、ur1 等参数，参数值指向实际跳转的目标地址(即使 URL 本身的主域名为合法域名，参数中的跳转地址也需要单独检查)

公司合法域名范围为:  
veritytech.com 及其所有子域名(*.veritytech.com)

请分析 999份PDF，找出所有指向非 veritvtech,com 域名 的 URL(包括文本显示、Annotation 跳转 URI以及 URL 参数中的跳转地址)，提取其完整URL，按字典序升序排序后，用英文半角逗号拼接，最终对拼接字符串计算 32 位小写 MD5 作为答案。  
对于包含跳转参数的 URL(如 https://mail.veritytech.com/auth?next=https://evi1.com)，如果 URL本身的主域名为合法域名，则不应提取该完整 URL;但需要提取参数中的跳转地址(如 https://evi1.com)进行判断。

若识别出的钓鱼链接为如下3条(字典序升序)  
https://company.com/login  
https://veritytech.com.io/auth  
https://verify-veritytech.xyz/update  
则拼接为:  
https://company.com/login,https://veritytech.com.io/auth,https://verify-veritytech.xyz/update  
对该字符串计算 MD5 即为最终提交值，例如:  
7c57e0774ddf8f4b8539b255a8f6f506

示例说明:  
如果 PDF 中存在 URL https://mail.veritytech.com/auth?next=https://evi1.com/login，由于完整URL的主域名为 mail.veritytech.com(合法)不应提取该完整 URL;但需要提取参数中的 https://evi1.com/login 进行判断，若其主域名为非veritytech.com 域名，则应提取该 URL。  
如果 PDF 文本中直接显示 https://evil.com/login，则直接提取该 URL。  
如果 PDF Annotation 中隐藏的跳转地址为 https://evi1.com/login，则提取该 URL。

#### 解答

比赛时间内没做出来，不知道下面脚本对不对

```python
"""
最终验证脚本 - 全面检查所有URL
"""

import os
import re
import hashlib
from urllib.parse import urlparse, parse_qs, unquote
import fitz  # PyMuPDF


def clean_url(url):
    """清理URL，移除末尾的非URL字符"""
    url = re.sub(r'[\u4e00-\u9fff，。、；：！？【】（）《》""'']+.*$', '', url)
    while url and url[-1] in '.,;:!?)]\'"':
        url = url[:-1]
    return url


def is_valid_url(url):
    """检查URL是否有效（有完整的域名）"""
    try:
        parsed = urlparse(url)
        netloc = parsed.netloc.lower()
        if ':' in netloc:
            host = netloc.split(':')[0]
        else:
            host = netloc
        
        if not host or '.' not in host:
            return False
        
        parts = host.split('.')
        if len(parts[-1]) < 2:
            return False
        
        incomplete_endings = [
            'veri', 'verit', 'verity', 'verityt', 'verityte', 'veritytec', 'veritytech',
            've', 'ver'
        ]
        if parts[-1] in incomplete_endings:
            return False
        
        if host.endswith('.veritytech') or host.endswith('.veritytec') or \
           host.endswith('.verityte') or host.endswith('.verityt') or \
           host.endswith('.verity') or host.endswith('.verit') or \
           host.endswith('.veri') or host.endswith('.ver') or host.endswith('.ve'):
            return False
        
        if host.endswith('.veritytech.co') and (not parsed.path or parsed.path == '/'):
            return False
        
        return True
    except:
        return False


def extract_domain(url):
    """提取URL的主域名"""
    try:
        parsed = urlparse(url)
        return parsed.netloc.lower()
    except:
        return ""


def get_host(domain):
    """从域名中提取主机名（去掉端口）"""
    if ':' in domain:
        return domain.split(':')[0]
    return domain


def is_legitimate_domain(domain):
    """检查域名是否为合法域名（veritytech.com及其子域名）"""
    if not domain:
        return False
    host = get_host(domain.lower())
    return host == "veritytech.com" or host.endswith(".veritytech.com")


def extract_redirect_urls(url):
    """从URL参数中提取可能的跳转地址"""
    redirect_params = ['redirect', 'next', 'url', 'uri', 'goto', 'target', 'to', 'link', 'return', 'returnurl', 'return_url', 'redirect_uri', 'redirect_url', 'callback', 'continue', 'dest', 'destination', 'redir', 'ref', 'referer', 'referrer']
    
    redirect_urls = []
    try:
        parsed = urlparse(url)
        query_params = parse_qs(parsed.query)
        
        for param_name, values in query_params.items():
            param_lower = param_name.lower()
            if any(rp in param_lower for rp in redirect_params):
                for value in values:
                    decoded_value = unquote(value)
                    if decoded_value.startswith('http://') or decoded_value.startswith('https://'):
                        redirect_urls.append(decoded_value)
    except Exception as e:
        pass
    
    return redirect_urls


def extract_urls_from_text(text):
    """从文本中提取URL"""
    url_pattern = r'https?://[a-zA-Z0-9][-a-zA-Z0-9]*(?:\.[a-zA-Z0-9][-a-zA-Z0-9]*)+(?::\d+)?(?:/[^\s<>\"\'）\]}\u4e00-\u9fff]*)?'
    urls = re.findall(url_pattern, text)
    
    cleaned_urls = []
    for url in urls:
        url = clean_url(url)
        if url and is_valid_url(url):
            cleaned_urls.append(url)
    
    return cleaned_urls


def analyze_pdf(pdf_path):
    """分析单个PDF文件，提取所有URL"""
    urls_found = set()
    
    try:
        doc = fitz.open(pdf_path)
        
        for page_num in range(len(doc)):
            page = doc[page_num]
            
            # 1. 提取文本中的URL
            text = page.get_text()
            text_urls = extract_urls_from_text(text)
            urls_found.update(text_urls)
            
            # 2. 提取Annotation中的URL
            annotations = page.annots()
            if annotations:
                for annot in annotations:
                    try:
                        info = annot.info
                        if 'uri' in info:
                            url = clean_url(info['uri'])
                            if url and is_valid_url(url):
                                urls_found.add(url)
                    except:
                        pass
            
            # 3. 提取链接（Link annotations）
            links = page.get_links()
            for link in links:
                if 'uri' in link:
                    url = clean_url(link['uri'])
                    if url and is_valid_url(url):
                        urls_found.add(url)
        
        doc.close()
    except Exception as e:
        print(f"Error processing {pdf_path}: {e}")
    
    return urls_found


def main():
    pdf_dir = r"c:\Users\33113\Desktop\数信杯\文档安全事件分析\decrypted_pdfs"
    
    all_urls = set()
    url_sources = {}  # 记录每个URL的来源
    
    pdf_files = [f for f in os.listdir(pdf_dir) if f.endswith('.pdf')]
    print(f"共发现 {len(pdf_files)} 个PDF文件")
    
    for i, pdf_file in enumerate(pdf_files):
        if (i + 1) % 100 == 0:
            print(f"正在处理: {i + 1}/{len(pdf_files)}")
        
        pdf_path = os.path.join(pdf_dir, pdf_file)
        urls = analyze_pdf(pdf_path)
        
        for url in urls:
            all_urls.add(url)
            if url not in url_sources:
                url_sources[url] = []
            url_sources[url].append(pdf_file)
    
    print(f"\n共发现 {len(all_urls)} 个URL")
    
    # 分析URL，找出非veritytech.com域名的URL
    non_legitimate_urls = set()
    
    for url in all_urls:
        domain = extract_domain(url)
        
        if is_legitimate_domain(domain):
            # 合法域名，但需要检查参数中的跳转地址
            redirect_urls = extract_redirect_urls(url)
            for redirect_url in redirect_urls:
                redirect_url_clean = clean_url(redirect_url)
                if redirect_url_clean and is_valid_url(redirect_url_clean):
                    redirect_domain = extract_domain(redirect_url_clean)
                    if redirect_domain and not is_legitimate_domain(redirect_domain):
                        non_legitimate_urls.add(redirect_url_clean)
        else:
            # 非合法域名
            if domain:
                non_legitimate_urls.add(url)
    
    print(f"共发现 {len(non_legitimate_urls)} 个非法URL")
    
    # 按字典序排序
    sorted_urls = sorted(non_legitimate_urls)
    
    # 打印所有非法URL及其来源
    print("\n非法URL列表及来源:")
    for url in sorted_urls:
        sources = url_sources.get(url, [])
        # 检查是否来自跳转参数
        is_redirect = False
        for full_url in all_urls:
            if is_legitimate_domain(extract_domain(full_url)):
                redirect_urls = extract_redirect_urls(full_url)
                if url in [clean_url(r) for r in redirect_urls]:
                    is_redirect = True
                    sources = url_sources.get(full_url, [])
                    break
        
        if is_redirect:
            print(f"  {url} (来自跳转参数, 源文件: {sources[:2]}...)")
        else:
            print(f"  {url} (源文件: {sources[:2]}...)")
    
    # 用逗号拼接
    joined_string = ",".join(sorted_urls)
    print(f"\n拼接后的字符串:")
    print(joined_string)
    
    # 计算MD5
    md5_hash = hashlib.md5(joined_string.encode('utf-8')).hexdigest()
    print(f"\nMD5 (32位小写): {md5_hash}")
    
    return sorted_urls, md5_hash


if __name__ == "__main__":
    main()

```

MD5 (32位小写): e5d7a8f4928cf5bb60df13dad400adbb

## 数据存储

### 数据存储1

#### 题目

工程师小王为了保证数据的安全存储，开发了对数据处理的程序，但这样的处理方式安全吗?分析程序功能，解密文件获取原始数据，提交第6行第2列数据。

给了re87a57766文件和info_19ff9a2.ori.en文件，解密info_19ff9a2.ori.en，提交第六行第二列

#### 解答

本题是为第二题做铺垫的

直接ida

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260102003941.png)

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260102004017.png)

中转函数，看sub_401249

```c
char *__fastcall sub_401249(unsigned __int8 *a1, int a2, unsigned int *a3)
{
  unsigned __int8 *v4; // rax
  unsigned int v5; // eax
  int v6; // eax
  unsigned int v7; // eax
  unsigned int v8; // eax
  unsigned __int8 *v11; // [rsp+10h] [rbp-18h]
  unsigned int v12; // [rsp+1Ch] [rbp-Ch]
  unsigned int v13; // [rsp+1Ch] [rbp-Ch]
  int v14; // [rsp+20h] [rbp-8h]
  int v15; // [rsp+24h] [rbp-4h]
  int v16; // [rsp+24h] [rbp-4h]
  int v17; // [rsp+24h] [rbp-4h]

  v11 = a1;
  v15 = 0;
  v14 = 0;
  v12 = 0;
  if ( !a1 )
    return 0;
  if ( !dword_4042A0 )
    sub_4011D6();
  while ( 1 )
  {
    v6 = a2--;
    if ( !v6 || v12 > 0xFFFFFFFB )
      break;
    v4 = v11++;
    v16 = *v4 + v15;
    if ( ++v14 == 3 )
    {
      byte_4042A4[v12] = byte_402020[v16 >> 18];
      byte_4042A4[v12 + 1] = byte_402020[(v16 >> 12) & 0x3F];
      byte_4042A4[v12 + 2] = byte_402020[(v16 >> 6) & 0x3F];
      v5 = v12 + 3;
      v12 += 4;
      byte_4042A4[v5] = byte_402020[v16 & 0x3F];
      v15 = 0;
      v14 = 0;
    }
    else
    {
      v15 = v16 << 8;
    }
  }
  if ( v14 )
  {
    v17 = v15 << (8 * (2 - v14));
    byte_4042A4[v12] = byte_402020[v17 >> 18];
    byte_4042A4[v12 + 1] = byte_402020[(v17 >> 12) & 0x3F];
    v7 = v12 + 2;
    v13 = v12 + 3;
    if ( v14 == 1 )
      byte_4042A4[v7] = 61;
    else
      byte_4042A4[v7] = byte_402020[(v17 >> 6) & 0x3F];
    v8 = v13;
    v12 = v13 + 1;
    byte_4042A4[v8] = 61;
  }
  byte_4042A4[v12] = 0;
  *a3 = v12;
  return byte_4042A4;
}
```

这是一个base64编码函数

总结：re87a57766是个ELF可执行文件，读取info_19ff9a2.ori文件，base64编码其内容后，输出为info_19ff9a2.ori.enc

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20260102004333.png)

### 数据存储2（没做出来）

#### 题目

工程师小王认识到前面开发的程序并不能保证对数据的安全存储，现在对处理程序进行了改进，这次能行吗?分析程序功能，解密文件获取原始数据，提交第8行第2列数据。

类似上题，加密方式变了

#### 解答

‍

## 数据解密

### 题目

```python
import os
from Crypto.Util.number import *
from Crypto.Cipher import AES
from secret import flag, key
from Crypto.Util.Padding import pad

assert(len(flag) == 38)
assert flag[:5] == b'flag{' and flag[-1:] == b'}'
assert(len(key) == 16)

flag='flag{IADMIN-TOP-18880101-7634567_2025}'
def padding(msg):
    tmp = 16 - len(msg) % 16
    pad = format(tmp, '02x')
    return bytes.fromhex(pad * tmp) + msg
message = padding(flag)
hint = bytes_to_long(key) ^ bytes_to_long(message[:16])
message = pad(message, 16, 'pkcs7')
print(message)
IV = os.urandom(16)
encryption = AES.new(key, AES.MODE_CBC, iv=IV)
enc = encryption.encrypt(message)

print('enc =', enc.hex())
print('hint =', hex(hint)[2:])

# enc = 1ce1df3812668ce0bccd86c146cc56989681e128edd0676f5d26e01abdee90c860e22a5a491f94ac5ca3ab02242740fb8c35a3b60ea737ca0d2662fba2b0e299
# hint = 32393f4e3c3c4f3e323a512a5356437d
```

*吐槽：这题是长亭出的吧，跟我一个月前做的sm4版本一模一样*

### 解答

```python
def padding(msg):
    tmp = 16 - len(msg) % 16
    pad = format(tmp, '02x')
    return bytes.fromhex(pad * tmp) + msg

msg1 = "flag{0123456789abcdef0123456789abcdef}"
msg2 = padding(msg1.encode())
print(msg2)
msg3 = pad(msg2 , 16, 'pkcs7')
print(padding(msg3))
```

自己编一点数据看看填充效果

```python
b'\n\n\n\n\n\n\n\n\n\nflag{0123456789abcdef0123456789abcdef}'
b'\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\n\n\n\n\n\n\n\n\n\nflag{0123456789abcdef0123456789abcdef}\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10'
```

可知hint=‘\n\n\n\n\n\n\n\n\n\nflag{?' ^ key

可通过爆破最后一位的值，尝试哪个key是对的

```python
c1 = enc(k,p1^iv)
c2 = enc(k,p2^c1)
c3 = enc(k,p3^c2)

p3 = dec(k,c3)^c2
p2 = dec(k,c2)^c1
p1 = dec(k,c1)^iv
```

根据cbc的加密方式特征，iv不知道的情况下，不影响解密p2，p3等后续密文

根据flag的特征可知，明文的结尾是特征是  }\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10

因此在爆破的时候，看哪个key可以满足该特征

```python
import hashlib
from Crypto.Cipher import AES
import os
import string
from Crypto.Util.number import *
from Crypto.Util.Padding import pad

enc = '1ce1df3812668ce0bccd86c146cc56989681e128edd0676f5d26e01abdee90c860e22a5a491f94ac5ca3ab02242740fb8c35a3b60ea737ca0d2662fba2b0e299'
hint = '32393f4e3c3c4f3e323a512a5356437d'

"""
c1 = enc(k,p1^iv)
c2 = enc(k,p2^c1)
c3 = enc(k,p3^c2)

p3 = dec(k,c3)^c2
p2 = dec(k,c2)^c1
p1 = dec(k,c1)^iv

"""

'''
flag = 6+16+16
flag{? 5位，还差一位
} 结尾
'''

def padding(msg):
    tmp = 16 - len(msg) % 16
    pad = format(tmp, '02x')
    return bytes.fromhex(pad * tmp) + msg

msg1 = "flag{0123456789abcdef0123456789abcdef}"
msg2 = padding(msg1.encode())
print(msg2)
msg3 = pad(msg2 , 16, 'pkcs7')
print(padding(msg3))


enc_b = bytes.fromhex(enc) # 64
hint_b = bytes.fromhex(hint)

IV_fake = os.urandom(16)
for char in string.ascii_letters + string.digits:
    # print("flag{"+char)
    test_msg = '\n\n\n\n\n\n\n\n\n\nflag{'+char
    # print(test.encode())
    test_key = long_to_bytes(bytes_to_long(hint_b) ^ bytes_to_long(test_msg.encode()))

    encryption = AES.new(test_key, AES.MODE_CBC, iv=IV_fake)

    p = encryption.decrypt(enc_b)
    if '}\\x10' in str(p):
        print(p)
        print(test_msg)


# flag{IADMIN-TOP-18880101-7634567_2025}
```

## 数据隐藏

### 题目

某汽车供应链物流中台正在进行季度数据归档，由于归档任务占用了主索引资源，运维团队启用了一套“底层应急索引机制”。该机制并不依赖 SOLite 原生的索引，而是设计了一套自定义的跨页链表协议，将关键筛选逻辑碎片化地存储在数据库文件的物理空闲块 (Freeblocks) 数据区中。 请检査 sys_config 表，获取底层链表的入口指针以及自定义链表节点的结构定义。根据结构定义，从底层物理空间中提取并重组出“特定批次货物筛选脚本”(SQL)。隐写数据位于 SQLite Freeblock 的有效载荷区(跳过 Freeblock 自身的4字节头部)。数据经过了异或处理，密钥与所在物理页号有关。运行提取出的脚本，定位出该批次雷达模组所在的 集装箱编号(containerid)和车牌号(license_plate)，最终需要将 container_id 和license_plate 的后五位数字使用下划线连接提交，例如:CN2877541671_72345。

### 解答

‍
