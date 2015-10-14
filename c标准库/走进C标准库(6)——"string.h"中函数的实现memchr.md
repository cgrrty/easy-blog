### 走进C标准库(6)——"string.h"中函数的实现memchr

我写的memchr：

    void *memchr(const void *buf, char ch, unsigned count){
        unsigned int cnt = 0;
        while(*(buf++) != ch && cnt <= count){cnt++;}
        if(cnt > count)
           return NULL;
        else
           return buf;
    }

while循环中的*(buf++)部分报错。

该错误为为ANSIC中认定的错误，是因为它坚持：进行算法操作的指针必须是确定知道其指向数据类型大小的。

但是GNU则不这么认定，它指定void * 的算法操作与char * 一致。

在实际的程序设计中，为迎合ANSI标准，并提高程序的可移植性，我们可以对void指针先进行强制类型转换为特定类型的指针。

则上述代码修改为：

    void *memchr(const void *buf, char ch, unsigned count){
         unsigned int cnt = 0;
         while(*((char *)buf++) != ch && cnt <= count){cnt++;}
         if(cnt > count)
            return NULL;
         else
            return buf;
     }

---

    int main(void)
    {
        char *s = "hello world";
        char *ptr = memchr(s,'w',10);
        printf("%x\n%x\n%c",s,ptr,*ptr);
        return 0;
    }

输出结果为：

403000

403007

o

有错，理论上*ptr应该为w

循环条件上多加了1

最终的程序：

    void *memchr(const void *buf, char ch, unsigned count){
        unsigned int cnt = 0;
        while(*((char *)buf++) != ch && cnt <= count){cnt++;}
        if(cnt > count)
           return NULL;
        else
           return (char *)buf - 1;
    }

microsoft visual C 中的memchr的实现为：

    void * __cdecl memchr (
            const void * buf,
            int chr,
            size_t cnt
            )
    {
            while ( cnt && (*(unsigned char *)buf != (unsigned char)chr) ) {
                    buf = (unsigned char *)buf + 1;
                    cnt--;
            }

            return(cnt ? (void *)buf : NULL);
    }

参考其代码仍有两点可以改进：

1.可将最后的if语句改为三元表达式的形式。

2.使用unsigned char保证能接受非常规的字符（如字符¥的ASCII码值在850 (Latin 1)为190），同时保证字符运算时结果的正确性（有符号数的运算结果会受符号位的影响，如0-(-128)=-128）。

