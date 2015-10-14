### 走进C标准库(7)——"string.h"中函数的实现memcmp,memcpy,memmove,memset

我的memcmp：

    int memcmp(void *buf1, void *buf2, unsigned int count){
        int reval;
        while(count && !(reval = (*(unsigned char *)buf1) - (*(unsigned char *)buf2)))
        {
            buf1 = (unsigned char *)buf1 + 1;
            buf2 = (unsigned char *)buf2 + 1;
            --count;
        }
        return reval;
    }

MS VC：

    int __cdecl memcmp (
            const void * buf1,
            const void * buf2,
            size_t count
            )
    {
            if (!count)
                    return(0);

            while ( --count && *(char *)buf1 == *(char *)buf2 ) {
                    buf1 = (char *)buf1 + 1;
                    buf2 = (char *)buf2 + 1;
            }

            return( *((unsigned char *)buf1) - *((unsigned char *)buf2) );
    }

应该使用const void *buf为宜，不改变该块内存的内容，最终使用unsigned char *进行运算，保证运算结果的符号正确。

<br />

我的memcpy：

    void *memcpy(void *dest, const void *src, unsigned int count){
        void *reval = dest;
        while(count--){
             (*(unsigned char *)dest++) = (*(unsigned char *)src++);
        }
        return reval;
    }

MSVC：

    void * __cdecl memcpy (
            void * dst,
            const void * src,
            size_t count
            )
    {
            void * ret = dst;

    #if defined (_M_MRX000) || defined (_M_ALPHA) || defined (_M_PPC)
            {
            extern void RtlMoveMemory( void *, const void *, size_t count );

            RtlMoveMemory( dst, src, count );
            }
    #else  /* defined (_M_MRX000) || defined (_M_ALPHA) || defined (_M_PPC) */
            /*
             - copy from lower addresses to higher addresses
             */
            while (count--) {
                    *(char *)dst = *(char *)src;
                    dst = (char *)dst + 1;
                    src = (char *)src + 1;
            }
    #endif  /* defined (_M_MRX000) || defined (_M_ALPHA) || defined (_M_PPC) */

            return(ret);
    }

<br />

我的memmove:

    void *memmove(void *dest, const void *src, unsigned int count){
        void *reval = dest;
        int overlap = ((unsigned char *)src < (unsigned char *)dest && ((unsigned char *)src + count) > dest);
        while(count--){
            if(overlap)//src is in front of dest and overlap. copy direction is from endIndex to beginIndex
                  (*((unsigned char *)dest + count)) = (*((unsigned char *)src + count));
            else
                (*(unsigned char *)dest++) = (*(unsigned char *)src++);
        }
        return reval;
    }

MSVC:

    void * __cdecl memmove (
            void * dst,
            const void * src,
            size_t count
            )
    {
            void * ret = dst;

    #if defined (_M_MRX000) || defined (_M_ALPHA) || defined (_M_PPC)
            {
            extern void RtlMoveMemory( void *, const void *, size_t count );

            RtlMoveMemory( dst, src, count );
            }
    #else  /* defined (_M_MRX000) || defined (_M_ALPHA) || defined (_M_PPC) */
            if (dst <= src || (char *)dst >= ((char *)src + count)) {
                    /*
                     - Non-Overlapping Buffers
                     - copy from lower addresses to higher addresses
                     */
                    while (count--) {
                            *(char *)dst = *(char *)src;
                            dst = (char *)dst + 1;
                            src = (char *)src + 1;
                    }
            }
            else {
                    /*
                     - Overlapping Buffers
                     - copy from higher addresses to lower addresses
                     */
                    dst = (char *)dst + count - 1;
                    src = (char *)src + count - 1;

                    while (count--) {
                            *(char *)dst = *(char *)src;
                            dst = (char *)dst - 1;
                            src = (char *)src - 1;
                    }
            }
    #endif  /* defined (_M_MRX000) || defined (_M_ALPHA) || defined (_M_PPC) */

            return(ret);
    }

关于memcpy和memmove的区别，memcpy不考虑内存区域重叠的情况而memmove保证内存区域重叠也能正常复制成功。

有时候我们的memcpy也可能在内存重叠的情况下正常使用，这取决于它的实现，不具有普遍性，C语言标准中未对其有这种要求。

参考资料：

[《关于memcpy和memmove两函数的区别》](http://blog.csdn.net/caowei840701/article/details/8491836)

[《memcpy() vs memmove()》](http://stackoverflow.com/questions/4415910/memcpy-vs-memmove)

<br /> 

我的memset:

    void *memset(void *buffer, int c, int count){
        void *reval = buffer;
        while(count--){
            (*(unsigned char *)buffer++) = (unsigned char)c;
        }
        return reval;
    }

MSVC:

    void * __cdecl memset (
            void *dst,
            int val,
            size_t count
            )
    {
            void *start = dst;

    #if defined (_M_MRX000) || defined (_M_ALPHA) || defined (_M_PPC)
            {
            extern void RtlFillMemory( void *, size_t count, char );

            RtlFillMemory( dst, count, (char)val );
            }
    #else  /* defined (_M_MRX000) || defined (_M_ALPHA) || defined (_M_PPC) */
            while (count--) {
                    *(char *)dst = (char)val;
                    dst = (char *)dst + 1;
            }
    #endif  /* defined (_M_MRX000) || defined (_M_ALPHA) || defined (_M_PPC) */

            return(start);
    }

