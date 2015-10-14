### 走进C标准库(1)——assert.h,ctype.h

  默默觉得原来的阅读笔记的名字太土了，改了个名字，叫做走进C标准库。

  自己就是菜鸟一只，第一次具体看C标准库，文章参杂了对《the standard C library》的阅读和对源码的一些个人浅显理解，自己记录一下，日后有机会来看可能有另一番感悟吧。

#### assert.h

  assert宏定义的两种表达方式：

      #define assert(exp) ((exp) ? (void)0 : _assert(msg))
      #define assert(exp) (void)( (exp) || _assert(msg))

   在《C陷阱与缺陷》一书中有描述关于assert宏实现上的考虑。

   在实现上，我们抛弃了下面这种宏实现的方式：

     #define assert(exp) if(!exp) _assert(msg);

   因为上述这个宏可能会产生某些难于察觉的错误：

     if(x > 0 && y > 0)
         assert(x > y);
     else
         assert(y > x);

   上述代码如果采用该宏就会产生else悬挂的问题，且无法在宏内合理有效消除，很不直观。

   所以，采用一开始的两种assert的宏定义是相对较好的。

#### ctype.h

  ##### 1.发展历程：

  惯用法( if (0 <= c && c <= '9' ) ，使用频繁程序过长且没有利用好重用的代码)  到

  函数区分字符类别( isalpha(c) 调用次数过频，影响了程序的执行时间 ) 到

  宏定义（节约了执行时间，但是会遇到一些问题）

  ##### 2.宏定义可能会产生的问题

  * 程序员编写的代码量虽然小，但是编译后的代码量很大
    
  * 子表达式存在捆绑不紧，可能被分割的问题
    
  * 参数可能会被执行不被期望的多次（如getc，putc，++，--等）

  ##### 3.使用转换表

  字符c编入以_ctype命名的转换表索引中。每个表项的不同位以索引字符为特征。如果任何一个和掩码_XXXMARK相对应的位被设置了，那个字符就在测试的类别中。对所有正确的参数，宏展开成一个紧凑的非零表达式。
    
---
        #define _UPPER          0x1     /* upper case letter */      
        #define _LOWER          0x2     /* lower case letter */
        #define _DIGIT          0x4     /* digit[0-9] */
        #define _SPACE          0x8     /* tab, carriage return, newline, */
                                        /* vertical tab or form feed */
        #define _PUNCT          0x10    /* punctuation character */
        #define _CONTROL        0x20    /* control character */
        #define _BLANK          0x40    /* space char */
        #define _HEX            0x80    /* hexadecimal digit */
        
        #define _LEADBYTE       0x8000                  /* multibyte leadbyte */
        #define _ALPHA          (0x0100|_UPPER|_LOWER)  /* alphabetic character */

        unsigned short *_pctype = _ctype+1;     /* pointer to table for char's      */
        unsigned short *_pwctype = _ctype+1;    /* pointer to table for wchar_t's   */
---
        unsigned short _ctype[257] = {
            0,                      /* -1 EOF   */
            _CONTROL,               /* 00 (NUL) */
            _CONTROL,               /* 01 (SOH) */
            _CONTROL,               /* 02 (STX) */
            _CONTROL,               /* 03 (ETX) */
            _CONTROL,               /* 04 (EOT) */
            _CONTROL,               /* 05 (ENQ) */
            _CONTROL,               /* 06 (ACK) */
            _CONTROL,               /* 07 (BEL) */
            _CONTROL,               /* 08 (BS)  */
            _SPACE+_CONTROL,        /* 09 (HT)  */
            _SPACE+_CONTROL,        /* 0A (LF)  */
            _SPACE+_CONTROL,        /* 0B (VT)  */
            _SPACE+_CONTROL,        /* 0C (FF)  */
            _SPACE+_CONTROL,        /* 0D (CR)  */
            _CONTROL,               /* 0E (SI)  */
            _CONTROL,               /* 0F (SO)  */
            _CONTROL,               /* 10 (DLE) */
            _CONTROL,               /* 11 (DC1) */
            _CONTROL,               /* 12 (DC2) */
            _CONTROL,               /* 13 (DC3) */
            _CONTROL,               /* 14 (DC4) */
            _CONTROL,               /* 15 (NAK) */
            _CONTROL,               /* 16 (SYN) */
            _CONTROL,               /* 17 (ETB) */
            _CONTROL,               /* 18 (CAN) */
            _CONTROL,               /* 19 (EM)  */
            _CONTROL,               /* 1A (SUB) */
            _CONTROL,               /* 1B (ESC) */
            _CONTROL,               /* 1C (FS)  */
            _CONTROL,               /* 1D (GS)  */
            _CONTROL,               /* 1E (RS)  */
            _CONTROL,               /* 1F (US)  */
            _SPACE+_BLANK,          /* 20 SPACE */
            _PUNCT,                 /* 21 !     */
            _PUNCT,                 /* 22 "     */
            _PUNCT,                 /* 23 #     */
            _PUNCT,                 /* 24 $     */
            _PUNCT,                 /* 25 %     */
            _PUNCT,                 /* 26 &     */
            _PUNCT,                 /* 27 '     */
            _PUNCT,                 /* 28 (     */
            _PUNCT,                 /* 29 )     */
            _PUNCT,                 /* 2A *     */
            _PUNCT,                 /* 2B +     */
            _PUNCT,                 /* 2C ,     */
            _PUNCT,                 /* 2D -     */
            _PUNCT,                 /* 2E .     */
            _PUNCT,                 /* 2F /     */
            _DIGIT+_HEX,            /* 30 0     */
            _DIGIT+_HEX,            /* 31 1     */
            _DIGIT+_HEX,            /* 32 2     */
            _DIGIT+_HEX,            /* 33 3     */
            _DIGIT+_HEX,            /* 34 4     */
            _DIGIT+_HEX,            /* 35 5     */
            _DIGIT+_HEX,            /* 36 6     */
            _DIGIT+_HEX,            /* 37 7     */
            _DIGIT+_HEX,            /* 38 8     */
            _DIGIT+_HEX,            /* 39 9     */
            _PUNCT,                 /* 3A :     */
            _PUNCT,                 /* 3B ;     */
            _PUNCT,                 /* 3C <     */
            _PUNCT,                 /* 3D =     */
            _PUNCT,                 /* 3E >     */
            _PUNCT,                 /* 3F ?     */
            _PUNCT,                 /* 40 @     */
            _UPPER+_HEX,            /* 41 A     */
            _UPPER+_HEX,            /* 42 B     */
            _UPPER+_HEX,            /* 43 C     */
            _UPPER+_HEX,            /* 44 D     */
            _UPPER+_HEX,            /* 45 E     */
            _UPPER+_HEX,            /* 46 F     */
            _UPPER,                 /* 47 G     */
            _UPPER,                 /* 48 H     */
            _UPPER,                 /* 49 I     */
            _UPPER,                 /* 4A J     */
            _UPPER,                 /* 4B K     */
            _UPPER,                 /* 4C L     */
            _UPPER,                 /* 4D M     */
            _UPPER,                 /* 4E N     */
            _UPPER,                 /* 4F O     */
            _UPPER,                 /* 50 P     */
            _UPPER,                 /* 51 Q     */
            _UPPER,                 /* 52 R     */
            _UPPER,                 /* 53 S     */
            _UPPER,                 /* 54 T     */
            _UPPER,                 /* 55 U     */
            _UPPER,                 /* 56 V     */
            _UPPER,                 /* 57 W     */
            _UPPER,                 /* 58 X     */
            _UPPER,                 /* 59 Y     */
            _UPPER,                 /* 5A Z     */
            _PUNCT,                 /* 5B [     */
            _PUNCT,                 /* 5C \     */
            _PUNCT,                 /* 5D ]     */
            _PUNCT,                 /* 5E ^     */
            _PUNCT,                 /* 5F _     */
            _PUNCT,                 /* 60 `     */
            _LOWER+_HEX,            /* 61 a     */
            _LOWER+_HEX,            /* 62 b     */
            _LOWER+_HEX,            /* 63 c     */
            _LOWER+_HEX,            /* 64 d     */
            _LOWER+_HEX,            /* 65 e     */
            _LOWER+_HEX,            /* 66 f     */
            _LOWER,                 /* 67 g     */
            _LOWER,                 /* 68 h     */
            _LOWER,                 /* 69 i     */
            _LOWER,                 /* 6A j     */
            _LOWER,                 /* 6B k     */
            _LOWER,                 /* 6C l     */
            _LOWER,                 /* 6D m     */
            _LOWER,                 /* 6E n     */
            _LOWER,                 /* 6F o     */
            _LOWER,                 /* 70 p     */
            _LOWER,                 /* 71 q     */
            _LOWER,                 /* 72 r     */
            _LOWER,                 /* 73 s     */
            _LOWER,                 /* 74 t     */
            _LOWER,                 /* 75 u     */
            _LOWER,                 /* 76 v     */
            _LOWER,                 /* 77 w     */
            _LOWER,                 /* 78 x     */
            _LOWER,                 /* 79 y     */
            _LOWER,                 /* 7A z     */
            _PUNCT,                 /* 7B {     */
            _PUNCT,                 /* 7C |     */
            _PUNCT,                 /* 7D }     */
            _PUNCT,                 /* 7E ~     */
            _CONTROL,               /* 7F (DEL) */
            /* and the rest are 0... */
        };
        
---
        #define _tolower(_c)    ( (_c)-'A'+'a' )
        #define _toupper(_c)    ( (_c)-'a'+'A' )

  ##### 4.转换表可能遇到的问题
  
  当区域设置改变时，我们的字符分类也可能会发生相应的改变。这种方法的弊端是，对于某些错误的参数，宏会产生错误的代码。如果一个宏的参数不在它的定义域内，那么执行这个宏时，它就会访问转换表之外的存储空间。
    
  如当测试某些比较生僻的字符代码时，若符号位被置为，那么参数会是一个负数，在函数的定义域之外。

对于EOF符号，也要慎重处理。

书上貌似没有提对于转换表实现方式的遇到问题的解决方案 -_-|||

  ##### 5.区域设置
  
  当区域设置改变时，我们的字符分类也可能会发生相应的改变。
    
  ##### 6.实践的实现中都是使用宏的吗？静态存储空间
  
  库可以使用指向表的指针的可写的静态存储空间，但我们不能在程序中不同控制线程中共享一个相同的数据对象。
  
  上面的实现没有使用静态存储空间。
  
   ##### 7.实践的实现中都是使用宏的吗？
  
  不小心看到mingw的实现并不是用的宏，代码如下： 

    #define __ISCTYPE(c, mask)  (MB_CUR_MAX == 1 ? (_pctype[c] & mask) : _isctype(c, mask))
    __CRT_INLINE int __cdecl __MINGW_NOTHROW isalnum(int c) {return __ISCTYPE(c, (_ALPHA|_DIGIT));}
    __CRT_INLINE int __cdecl __MINGW_NOTHROW isalpha(int c) {return __ISCTYPE(c, _ALPHA);}
    __CRT_INLINE int __cdecl __MINGW_NOTHROW iscntrl(int c) {return __ISCTYPE(c, _CONTROL);}
    __CRT_INLINE int __cdecl __MINGW_NOTHROW isdigit(int c) {return __ISCTYPE(c, _DIGIT);}
    __CRT_INLINE int __cdecl __MINGW_NOTHROW isgraph(int c) {return __ISCTYPE(c, (_PUNCT|_ALPHA|_DIGIT));}
    __CRT_INLINE int __cdecl __MINGW_NOTHROW islower(int c) {return __ISCTYPE(c, _LOWER);}
    __CRT_INLINE int __cdecl __MINGW_NOTHROW isprint(int c) {return __ISCTYPE(c, (_BLANK|_PUNCT|_ALPHA|_DIGIT));}
    __CRT_INLINE int __cdecl __MINGW_NOTHROW ispunct(int c) {return __ISCTYPE(c, _PUNCT);}
    __CRT_INLINE int __cdecl __MINGW_NOTHROW isspace(int c) {return __ISCTYPE(c, _SPACE);}
    __CRT_INLINE int __cdecl __MINGW_NOTHROW isupper(int c) {return __ISCTYPE(c, _UPPER);}
    __CRT_INLINE int __cdecl __MINGW_NOTHROW isxdigit(int c) {return __ISCTYPE(c, _HEX);}

  使用了内联函数的实现，相对于宏更加安全了，另外把预处理的工作交给了编译器，让编译器在代码量和代码效率间自动进行抉择。