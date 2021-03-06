# child_process

## spawn

### 选项uid

#### 了解一下文件权限

![](https://img2018.cnblogs.com/blog/978388/201904/978388-20190410111825498-1520948107.png)

权限数组说明：

总共分为四个部分：

【文件或文件夹】【owner权限（所有者）】【group权限（群主）】【others权限（其他人）】

【文件是-，文件夹是d】【r/w/x相加】【r/w/x相加】【r/w/x相加】

Linux档案的基本权限就有九个，分别是owner/group/others三种身份各有自己的read/write/execute权限。

d rwx  rwx  rwx  =777  表示目录的操作权限

-  rwx  rwx  rwx = 777  表示文件的操作权限

![](https://img2018.cnblogs.com/blog/978388/201904/978388-20190410112020377-2029399519.png)

 - rwx rwx rwx =777表示 文件的操作权限

-rw-  r--  r--  = 644  表示文件的操作权限

root root表示文件拥有者

**r 表示文件可以被读（read）**

**w 表示文件可以被写（write）**

**x 表示文件可以被执行（如果它是程序的话）**

**- 表示相应的权限还没有被授予**

#### 修改权限

```shell
# 表示其他人授予写的权限
chmod o w xxx.xxx
# 表示删除xxx.xxx中组群和其他人的读写权限
chmod go-rw xxx.xxx
```

**u 代表所有者（user）**

**g 代表所有者所在的组群（group）**

**o 代表其他人，但不是u和g （other）**

**a 代表全部的人，也就是包括u，g和o**

**r 表示文件可以被读（read）**

**w 表示文件可以被写（write）**

**x 表示文件可以被执行（如果它是程序的话）**

r: 对应数值4  
w: 对应数值2  
x：对应数值1  
－：对应数值0

**-rw------- (600) 只有所有者才有读和写的权限**

**-rw-r--r-- (644) 只有所有者才有读和写的权限，组群和其他人只有读的权限**

**-rwx------ (700) 只有所有者才有读，写，执行的权限**

**-rwxr-xr-x (755) 只有所有者才有读，写，执行的权限，组群和其他人只有读和执行的权限**

**-rwx--x--x (711) 只有所有者才有读，写，执行的权限，组群和其他人只有执行的权限**

**-rw-rw-rw- (666) 每个人都有读写的权限**

**-rwxrwxrwx (777) 每个人都有读写和执行的权限**

后面的数字是怎么来的呢，就是有三种身份，对应身份全新啊数字相加

参数：uid 用来设置进程用户标识

### 
