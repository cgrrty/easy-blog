### 走进C标准库(8)——"string.h"中函数的实现相关字符串操作函数

我的strcat：

    char *strcat(char *dest,char *src)
    {
        char * reval = dest;
        while(*dest)
            dest++;
        while(*src)
            *dest++ = *src++ ;
        *dest = *src;
        return reval;
    }

MSVC：

    char * __cdecl strcat (
            char * dst,
            const char * src
            )
    {
            char * cp = dst;

            while( *cp )
                    cp++;                   /* find end of dst */

            while( *cp++ = *src++ ) ;       /* Copy src to end of dst */

            return( dst );                  /* return dst */

    }

在while( *cp++ = *src++ )中，条件的值为赋值语句的返回值即*cp被赋的值，也就是此时*src的值。则，当*src为0时，将其赋给*cp后完成赋值。非常简洁。

该函数的前提条件：src和dest所指内存区域不可以重叠且dest必须有足够的空间来容纳src的字符串。

<br /> 

我的strncat：

    char *strncat(char *dest,char *src,int n)
    {
        char * reval = dest;
        while(*dest)
            dest++;
        while(n-- && (*dest++ = *src++));
        if(n < 0)
            *dest = 0;
        return reval;
    }

MSVC:

    char * __cdecl strncat (
            char * front,
            const char * back,
            size_t count
            )
    {
            char *start = front;

            while (*front++)
                    ;
            front--;

            while (count--)
                    if (!(*front++ = *back++))
                            return(start);

            *front = '\0';
            return(start);
    }    

<br />

我的strchr:

    char *strchr(const char *s,char c){
        while(*s)
        {
            if(*s == c)
                return s;
            s++;
        }
        return NULL;
    }

MSVC:

    char * __cdecl strchr (
            const char * string,
            int ch
            )
    {
            while (*string && *string != (char)ch)
                    string++;

            if (*string == (char)ch)
                    return((char *)string);
            return(NULL);
    }

<br />

我的strcmp：

    int strcmp(const char *s1,const char * s2){
        while(*s1 == *s2 && *s1)
        {
            ++s1;
            ++s2;
        }
        return *(unsigned char *)s1 - *(unsigned char *)s2;
    }

MSVC:

    int __cdecl strcmp (
            const char * src,
            const char * dst
            )
    {
            int ret = 0 ;

            while( ! (ret = *(unsigned char *)src - *(unsigned char *)dst) && *dst)
                    ++src, ++dst;

            if ( ret < 0 )
                    ret = -1 ;
            else if ( ret > 0 )
                    ret = 1 ;

            return( ret );
    }

strcmp返回值不必为1和-1的。使用unsigned char 因为有符号数可能会导致比较大小错误。

<br />

我的strcpy:

    char *strcpy(char *dest,const char *src){
        char * reval = dest;
        while(*dest++ = *src++);
        return reval;
    }

MSVC:

    char * __cdecl strcpy(char * dst, const char * src)
    {
            char * cp = dst;

            while( *cp++ = *src++ )
                    ;               /* Copy src over dst */

            return( dst );
    }

<br />

我的strncpy:

    char *strncpy(char *dest, const char *src, int n){
        char * reval = dest;
        while(n--){
            if(*src)
                *dest++ = *src++;
            else
                *dest++ = 0;
        }
        return reval;
    }

MSVC:

    char * __cdecl strncpy (
            char * dest,
            const char * source,
            size_t count
            )
    {
            char *start = dest;

            while (count && (*dest++ = *source++))    /* copy string */
                    count--;

            if (count)                              /* pad out with zeroes */
                    while (--count)
                            *dest++ = '\0';

            return(start);
    }

<br />

我的strcspn:

    int strcspn(const char *s1,const char *s2){
        const char *cp;
        int reval = 0;
        for(; *s1; s1++){
            for(cp = s2; *cp; cp++){
                if(*s1 == *cp)
                    return reval;
            }
            ++reval;
        }
        return reval;
    }

MSVC:

    size_t __cdecl strcspn (
            const char * string,
            const char * control
            )
    {
            const unsigned char *str = string;
            const unsigned char *ctrl = control;

            unsigned char map[32];
            int count;

            /* Clear out bit map */
            for (count=0; count<32; count++)
                    map[count] = 0;

            /* Set bits in control map */
            while (*ctrl)
            {
                    map[*ctrl >> 3] |= (1 << (*ctrl & 7));
                    ctrl++;
            }

            /* 1st char in control map stops search */
            count=0;
            map[0] |= 1;    /* null chars not considered */
            while (!(map[*str >> 3] & (1 << (*str & 7))))
            {
                    count++;
                    str++;
            }
            return(count);
    }

函数说明：strcspn()从参数s 字符串的开头计算连续的字符, 而这些字符都完全不在参数reject 所指的字符串中. 简单地说, 若strcspn()返回的数值为n, 则代表字符串s 开头连续有n 个字符都不含字符串reject 内的字符。

返回值：返回字符串s 开头连续不含字符串reject 内的字符数目。

我的实现和《C标准库》书中基本相同，不需要任何的额外存储空间，但是使用两层的循环，花费较多的时间。

该函数的实质其实是判断s1中的每一个字符，是否在s2字符串中，对于这种判断是否在其中的问题，经常会采用映射表的方式来缩减查找时间，典型实现就是布隆过滤器。

此处，MSVC的实现就是采用了映射表的方式。

因为ASCII码有256个，所以需要256bit来作为映射标记。一个字节是8bit，所以需要32个字节。所以在代码中有 unsigned char map[32]的定义。

那么，我们就需要将8bit，分别映射出一个map的下标和bit中的位位置。

map的下表需要使用5bit（32），而bit中的位位置使用剩余的3bit（8）来映射。

通过*ctrl >> 3取到高5位到0-31的映射，通过1 << (*ctrl & 7)取得到1字节中某一位的标记。

完成控制字符的映射表建立，就能用o(1)的时间完成某个字符的查找了。

<br />

strerror

功  能: 返回指向错误信息字符串的指针 

例如：

#include <stdio.h>
#include <errno.h>

    int main(void)
    {
       char *buffer;
       buffer = strerror(errno);
       printf("Error: %s\n", buffer);
       return 0;
    }

此段代码strerror指向的内存中的字符串为No Error

<br />

我的strlen:

    int strlen(const char *s){
        const char *cp = s;
        while(*cp){
            cp++;
        }
        return (cp - s);
    }

MSVC:

    size_t __cdecl strlen (
            const char * str
            )
    {
            const char *eos = str;

            while( *eos++ ) ;

            return( (int)(eos - str - 1) );
    }

返回值应该不需要强制类型转换，因为指针相减返回值是int。当然，加上显式转换则更加明确。

ptrdiff_t
This is the type returned by the subtraction operation between two pointers. This is a signed integral type, and as such can be casted to compatible fundamental data types.

<br /> 

strpbrk和strspn的实现和strcspn相同

<br />

我的strrchr:

    char *strrchr(const char *str, const char c){
        const char *cp = str;
        if(*str == 0)
            return NULL;
        while(*str)
            str++;
        while(*cp++){
            if(*--str == c)
                return str;
        }
        return NULL;
    }

MSVC:

    char * __cdecl strrchr (
            const char * string,
            int ch
            )
    {
            char *start = (char *)string;

            while (*string++)                       /* find end of string */
                    ;
                                                    /* search towards front */
            while (--string != start && *string != (char)ch)
                    ;

            if (*string == (char)ch)                /* char found ? */
                    return( (char *)string );

            return(NULL);
    }

确实只需要初始位置的拷贝，不需要用拷贝来计数。

<br /> 

strtok，没想好如何实现比较合适。

MSVC:

    char * __cdecl strtok (
            char * string,
            const char * control
            )
    {
            unsigned char *str;
            const unsigned char *ctrl = control;

            unsigned char map[32];
            int count;

            static char *nextoken;

            /* Clear control map */
            for (count = 0; count < 32; count++)
                    map[count] = 0;

            /* Set bits in delimiter table */
            do {
                    map[*ctrl >> 3] |= (1 << (*ctrl & 7));
            } while (*ctrl++);

            /* Initialize str. If string is NULL, set str to the saved
             - pointer (i.e., continue breaking tokens out of the string
             - from the last strtok call) */
            if (string)
                    str = string;
            else
                    str = nextoken;

            /* Find beginning of token (skip over leading delimiters). Note that
             - there is no token iff this loop sets str to point to the terminal
             - null (*str == '\0') */
            while ( (map[*str >> 3] & (1 << (*str & 7))) && *str )
                    str++;

            string = str;

            /* Find the end of the token. If it is not the end of the string,
             - put a null there. */
            for ( ; *str ; str++ )
                    if ( map[*str >> 3] & (1 << (*str & 7)) ) {
                            *str++ = '\0';
                            break;
                    }

            /* Update nextoken (or the corresponding field in the per-thread data
             - structure */
            nextoken = str;

            /* Determine if a token has been found. */
            if ( string == str )
                    return NULL;
            else
                    return string;
    }

用一个static变量来记录当前分割到的位置，它是线程不安全的，多次调用也会使它失效。

<br ?> 

strcoll使用当前的区域设置来比较字符串，strxfrm使用当前的区域设置来转换字符串。当前区域设置由LL_COLLATE宏指定。它们均调用带有区域设置参数的内部版本strcoll_l和strxfrm_l来完成实际的工作。

