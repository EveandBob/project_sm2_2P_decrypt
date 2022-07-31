#写在之前
项目中用到的函数简介：https://github.com/EveandBob/Introduction-to-some-functions-in-elliptic-curves-not-projects-

# 项目名称
实现sm2的二方解密

# 项目实现
该协议的描述为：
![Screenshot 2022-07-31 134655](https://user-images.githubusercontent.com/104854836/182012135-1a5d06c7-224b-4612-b090-477f91d09918.jpg)

由于该项目是解密算法，所以我就提前给A分配d1，B分配d2，并且计算得到协商私钥d，和协商公钥P，然后加密msg
"message digest"得到密文

C1, C2, C3 = [(9208123485055232548406805933208065892301341694609167043436851590198577329182,
                   22497156797805874980417938328502193624670699415281173926964553771202322035334),
                  428345813838056492794735501498020,
                  12159561144976371370304274003186071825317752572795393426538011957777595730391]
                  
# 部分代码
1.A
```python
def A():
    msg = "message digest"
    C1, C2, C3 = [(9208123485055232548406805933208065892301341694609167043436851590198577329182,
                   22497156797805874980417938328502193624670699415281173926964553771202322035334),
                  428345813838056492794735501498020,
                  12159561144976371370304274003186071825317752572795393426538011957777595730391]

    def A_1():
        d1 = 56978832327631670959038636096318190185559761644917797097344730557150433420452
        return d1

    def A_2(d1):
        x1,y1=C1
        T1=calculate_np(x1,y1,get_inverse(d1,n),a,b,p)
        return T1

    def A_4(T2):
        C1_T=calculate_Tp(C1[0],C1[1],a,b,p)
        (x2,y2)=calculate_p_q(T2[0],T2[1],C1_T[0],C1_T[1],a,b,p)
        num = int_to_bytes(x2) + int_to_bytes(y2)
        klen = len(msg.encode()) * 8
        t = KDF(num, klen)
        M=C2^t
        temp = int_to_bytes(x2) + int_to_bytes(M) + int_to_bytes(y2)
        u = int(sm3.sm3_hash(list(temp)), 16)
        if u == C3:
            return int_to_bytes(M).decode()


    s = socket.socket()  # 创建 socket 对象
    host = socket.gethostname()  # 获取本地主机名
    port = 12345  # 设置端口号

    s.connect((host, port))
    d1 = A_1()
    T1=str(A_2(d1))
    s.send(T1.encode())
    T2_str = s.recv(1024)
    T2 = json.loads(T2_str.decode())
    print("解密为")
    print(A_4(T2))
```
2.B
```python
def B():
    Q1 = 0, 0
    e = 0
    Bd2 = 0
    r, s2, s3 = 0, 0, 0

    def B_1():
        d2 = 37924224460188212696442974290998605142121773550747161941364380535096233845372
        return d2

    def B_3(d2,T1):
        return calculate_np(T1[0],T1[1],get_inverse(d2,n),a,b,p)

    s = socket.socket()  # 创建 socket 对象
    host = socket.gethostname()  # 获取本地主机名
    port = 12345  # 设置端口
    s.bind((host, port))  # 绑定端口
    s.listen(5)  # 等待客户端连接
    c, addr = s.accept()  # 建立客户端连接
    T1_str = c.recv(1024)
    T1 = json.loads(T1_str.decode())
    d2=B_1()
    T2 = str(B_3(d2,T1))
    c.send(str(T2).encode())
```

# 实验结果
![Screenshot 2022-07-31 135317](https://user-images.githubusercontent.com/104854836/182012284-ddf810f5-f5d9-4d76-9a1b-91a0de953c47.jpg)
