# C语言函数声明中的空参数


### 函数声明：int getP()与int getP(void)

> 总结：函数声明时参数为空是一种陋习，这表明参数类型和数目都不明确，可能会导致UB

```c
#include <stdio.h>

int getP();	 
float outF();

int main(void) {
    printf("%d\n", getP(10));
    outF(10);
    return 0;
}

int getP(int a) {
    int ret = 1;
    for (int i = 0; i < a; i++)
        ret = ret << 1;

    return ret;
}

float outF(float a) {
    printf("%f\n", a);
}
```

其中一个可以过编译，而另一个不行

`an argument type that has a default promotion can’t match an empty parameter name list declaration`

> ***《C语言程序设计第二版》，第4.2节：***
>
> 如果函数声明中不包含参数，例如： `double atof();`， 那么编译程序也不会对函数 atof 的参数作任何假设，并会关闭所有的参数检查。对空参数表的这种特殊处理是为了使新的编译器能编译比较老的 C 语言程序。不过，在新编写的程序中 这么做是不提倡的。如果函数带有参数，则要声明它们；如果没有参数，则使用 void 进行声 明。

更多详情见[Is it better to use C void arguments "void foo(void)" or not "void foo()"? - Stack Overflow](https://stackoverflow.com/questions/693788/is-it-better-to-use-c-void-arguments-void-foovoid-or-not-void-foo/36292431#36292431)


