# ldconfig-playground

这里展示一个通过ldconfig升级so的例子

1. user 作为so的使用方，需要使用user_test.h头文件和libuser_test.so文件，至于so的版本不关心
2. libv1 和 libv2在安装时会做以下动作
    * 拷贝user_test.h 到 /usr/local/include/user_test.h
    * 拷贝libuser_test.so.{主版本号}.{次版本号} 到/usr/local/lib/ 并执行ldconfig
    * ldconfig会从二进制文件中的SONAME取出 libuser_test.so.{主版本号} 并创建一个软连接
    * 创建一个软连接 libuser_test.so 指向 libuser_test.so.{主版本号}

## 流程展示

### 编译 & 安装 libv1

```
@freelw ➜ /workspaces/ldconfig-playground/libv1 (main) $ make
gcc -fPIC -Wall -o libuser_test.so.1.0 user_test.c -shared -Wl,-soname,libuser_test.so.1
@freelw ➜ /workspaces/ldconfig-playground/libv1 (main) $ sudo make install
install -D -m 755 libuser_test.so.1.0 /usr/local/lib/libuser_test.so.1.0
ldconfig
ln -sf /usr/local/lib/libuser_test.so.1 /usr/local/lib/libuser_test.so
cp user_test.h /usr/local/include/user_test.h
@freelw ➜ /workspaces/ldconfig-playground/libv1 (main) $ ll /usr/local/lib/libuser_test*
lrwxrwxrwx 1 root root    32 Jul 31 06:06 /usr/local/lib/libuser_test.so -> /usr/local/lib/libuser_test.so.1*
lrwxrwxrwx 1 root root    19 Jul 31 06:06 /usr/local/lib/libuser_test.so.1 -> l
```

### 编译执行user程序

```
@freelw ➜ /workspaces/ldconfig-playground/user (main) $ make
gcc -o user_program main.c -luser_test
@freelw ➜ /workspaces/ldconfig-playground/user (main) $ ./user_program 
Result from user_test_function: 1
```

### 编译 & 安装 libv2(无感升级)

```
@freelw ➜ /workspaces/ldconfig-playground/libv2 (main) $ sudo make install
install -D -m 755 libuser_test.so.1.1 /usr/local/lib/libuser_test.so.1.1
ldconfig
ln -sf /usr/local/lib/libuser_test.so.1 /usr/local/lib/libuser_test.so
cp user_test.h /usr/local/include/user_test.h
@freelw ➜ /workspaces/ldconfig-playground/libv2 (main) $ ll /usr/local/lib/libuser_test*
lrwxrwxrwx 1 root root    32 Jul 31 06:08 /usr/local/lib/libuser_test.so -> /usr/local/lib/libuser_test.so.1*
lrwxrwxrwx 1 root root    19 Jul 31 06:08 /usr/local/lib/libuser_test.so.1 -> libuser_test.so.1.1*
-rwxr-xr-x 1 root root 15128 Jul 31 06:06 /usr/local/lib/libuser_test.so.1.0*
-rwxr-xr-x 1 root root 15128 Jul 31 06:08 /usr/local/lib/libuser_test.so.1.1*
```

可以看到 /usr/local/lib/libuser_test.so.1 的指向已经修改了

### 重新执行user程序

```
@freelw ➜ /workspaces/ldconfig-playground/user (main) $ ./user_program 
Result from user_test_function: 2
```

说明已经调用新的so

这里看到，用户在编译的时候只关心so的名字叫做user_test，但是根本不关心版本号

再看一下ldd效果

```
@freelw ➜ /workspaces/ldconfig-playground/user (main) $ ldd ./user_program 
        linux-vdso.so.1 (0x00007fffccff2000)
        libuser_test.so.1 => /usr/local/lib/libuser_test.so.1 (0x00007c0f47b45000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007c0f47933000)
        /lib64/ld-linux-x86-64.so.2 (0x00007c0f47b58000)
```

说明执行时关心主版本号 libuser_test.so.1 但不关心小版本号

这样如果主版本升级，程序必须重新编译，因为主版本升级意味着接口不兼容，但是小版本升级不影响，一般是功能增加/bug修正

# 参考

* [How Do so (Shared Object) Filenames Work?](https://www.baeldung.com/linux/shared-object-filenames)
