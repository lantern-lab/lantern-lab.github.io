---
title: 2025第三届数信杯
slug: 2025-the-3rd-shuxin-cup-qxcuc
url: /post/2025-the-3rd-shuxin-cup-qxcuc.html
date: '2026-01-01 18:16:49+08:00'
lastmod: '2026-01-01 19:41:21+08:00'
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

### 流量分析

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
