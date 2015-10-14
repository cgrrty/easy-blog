### 走进C标准库(4)——"stdio.h"中的putc

花了点时间把园子弄得好看了点，现在继续。

函数名: putc

功  能: 输出一字符到指定流中 

用  法: int putc(int ch, FILE *stream); 

    #define _putc_lk(_c,_stream)    (--(_stream)->_cnt >= 0 ? 0xff & (*(_stream)->_ptr++ = (char)(_c)) :  _flsbuf((_c),(_stream)))

看到这个宏是否觉得很熟悉，很像getc的宏吧。

那么，很容易产生一个问题，同样是改变IO控制块中的信息，同时使用putc和getc能否正常地进行读写操作呢？

使用一个简单的样例测试了一下：

    #include <stdio.h>

    int main(void)
    {
       char a;
       int i = 0;
       FILE *fp = fopen("input.txt","r+");
       a = getc(fp);
       putc('b',fp);
       putc('c',fp);
       putc('d',fp);
       putc('e',fp);
       fclose(fp);
       return 0;
    }

以上这段代码未能正常在input.txt文件中写入内容。

通过单步调试可以看到，getc是成功的，putc也是改变了缓冲区的，但是在写入文件这一步出了问题。

更进一步，比较了在fopen后，先进行getc和先进行putc，fp的各成员的值的异同。可以看到，当先进行getc时，fp->_flag=137，而当先进行putc时，fp->_flag=138。那么，显然在第一次使用函数读写fp时，就会为fp定下不同的读写标记。该标记会阻止其他类型的操作。

于是，在上述代码的第8行下加入代码：fp->_flag = 138后，putc成功。

从缓冲区的意义来说，不会说不允许同时支持读写。原因可能是同时读写文件在是实际意义上会造成读信息或写信息的混乱，亦或者可能是实现上的原因。猜测。

言归正传：

那么putc只有在FILE中为写信息的时候才生效。

当stream->_cnt大于0（缓冲区还有写空位），就继续写，如果没有空位了就调用_flsbuf((_c),(_stream))，通过该函数调用WriteFile，将缓冲区中的内容写入到文件中，然后重置_ptr的位置和_cnt的大小。

简单的逻辑。


