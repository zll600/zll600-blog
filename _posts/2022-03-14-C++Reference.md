记录一下 C++ 中关于引用的一些知识

开发环境

````
$ uname -a
Linux ... 5.13.0-35-generic #40~20.04.1-Ubuntu SMP Mon Mar 7 09:18:32 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux

$ gcc --version
gcc (Ubuntu 9.4.0-1ubuntu1~20.04) 9.4.0
Copyright (C) 2019 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

````

现看一小段代码，然后总结一下

````c++
#include <iostream>

#include <stdio.h>

using namespace std;

int main(int argc, char *argv[]) {
    int i = 0;
    int& ref1 = i;
    cout << "i's val is " << i << endl;
    cout << "ref1's val is " << ref1 << endl;
    printf("i's addr is %p\n", &i);
    printf("ref1's addr is %p\n", &ref1);

    double d = 0.001;
    double& ref2 = d;
    cout << "d's val is " << d << endl;
    cout << "ref2's val is " << ref2 << endl;
    printf("d's addr is %p\n", &d);
    printf("ref2's addr is %p\n", &ref2);

    return 0;
}

````

运行结果

````
i's val is 0
ref1's val is 0
i's addr is 0x7ffd4e31692c
ref1's addr is 0x7ffd4e31692c
d's val is 0.001
ref2's val is 0.001
d's addr is 0x7ffd4e316930
ref2's addr is 0x7ffd4e316930

````

关于引用：

- 引用的值即为引用所指变量的值
- 引用的地址即为引用所指变量的地址

然后我们改一下代码

````c++
....
    cout << endl;

    ref2 = i;
    ref2 = 0.002;
    cout << "i's val is " << i << endl;
    cout << "ref2's val is " << ref2 << endl;
    cout << "d's val is " << d << endl;
    printf("i's addr is %p\n", &i);
    printf("ref2's addr is %p\n", &ref2);
    printf("d's addr is %p\n", &d);

....
````

运行结果如下：

````
.....
i's val is 0
ref2's val is 0.002
d's val is 0.002
i's addr is 0x7fffce3291ac
ref2's addr is 0x7fffce3291b0
d's addr is 0x7fffce3291b0

````

我们还可以得出

- 引用的类型必须和引用指向的变量的类型一致，
- 引用的改变也会导致引用所指向变量的改变

再改一下代码

````c++
....
    cout << "d's size is " << sizeof(d) << endl;
    cout << "ref2's size is " << sizeof(ref2) << endl;

.....
````

运行结果：

````
d's size is 8
ref2's size is 8

````

可以得出：

- 引用的大小，和引用所指变量的大小相同


待补充。。。。。。
