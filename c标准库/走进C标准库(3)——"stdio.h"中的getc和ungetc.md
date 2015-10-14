### 走进C标准库(3)——"stdio.h"中的getc和ungetc

接前文。

再来看看getc和ungetc的实现。在看这两个函数的实现之前，我们先来想一想这两个函数分别需要做的工作。

<em>int getc(FILE *stream) </em>

说明：函数getc从stream指向的输入流中读取下一个字符（如果有的话），并把它由unsigned char类型转换为int类型，并且流的相关的文件定位符（如果定义的话）向前移动一位。

返回值：函数getc返回stream指向的输入流的下一个字符，如果流处于文件结束处，则设置该流的文件结束指示符，函数getc返回EOF。如果发生了读错误，则设置流的错误指示符。函数getc返回EOF。

<em> int ungetc( int c , FILE *stream ); </em>

说明：

函数ungetc把c指定的字符（转换为unsigned char类型）退回到stream指向的输入流中，退回字符是通过对流的后续读取并按照退回的反顺序来返回的。如果中间成功调用（对同一个流）了一个文件定位函数（fseek、fsetpos或者rewind），那么就会丢弃流的所有退回的字符。流的相应的外部存储保持不变。

退回的一个字符是受到保护的。如果同一个流调用ungetc函数太多次，中间又没有读或者文件定位操作，那么这种操作可能会失败。

如果c的值和宏EOF的值相等，则操作失败且输入流保持不变。

对ungetc的成功调用会清空流的文件结束符。读取或者丢弃所有回退的字符后，流的文件定位符的值和这些字符回退之前的值相同。对文本流来说，在对函数ungetc的一次成功调用之后，它的文件定位符的值是不明确的，直到所有回退字符被读取或者丢弃为止。对二进制流来说，每成功调用一次ungetc之后它的文件定位符都会减1。如果在一次调用之前它的值是零，那么调用之后它的值是不确定的。

返回值:

函数ungetc返回转换后的回退字符，如果操作失败，则返回EOF。

大致来说，getc就是从stream中读入一个字符，并将流的定位符前移；ungetc就是将流的定位符后移，然后将一个字符回退到stream流该定位符的位置上。

这时我们可能会考虑到一些问题：

1.在stdio.h阅读笔记1中，我们看到在fopen函数调用之后，并没有为我们的stream分配buffer，那么这个分配会在真正进行stream读取的时候建立吗？

2.如果在getc之前（即流定位符在_base）就进行ungetc操作，会产生什么样的效果呢？

3.如原文所说：“如果同一个流调用ungetc函数太多次，中间又没有读或者文件定位操作，那么这种操作可能会失败。”，是因为文件定位符到达_base的原因吗？

那我们就来看下getc和ungetc所做的工作：

首先来看ungetc

    int __cdecl ungetc (
            REG2 int ch,
            FILE *str
            )
    {
            REG1 FILE *stream;
            _ASSERTE(str != NULL);
            /* Init stream pointer and file descriptor */
            stream = str;
            /* Stream must be open for read and can NOT be currently in write mode.
               Also, ungetc() character cannot be EOF. */
            if (
                  (ch == EOF) ||
                  !(
                    (stream->_flag & _IOREAD) ||
                    ((stream->_flag & _IORW) && !(stream->_flag & _IOWRT))
                   )
               )
                    return(EOF);
            /* If stream is unbuffered, get one. */
            if (stream->_base == NULL)
                    _getbuf(stream);
            /* now we know _base != NULL; since file must be buffered */
            if (stream->_ptr == stream->_base) {
                    if (stream->_cnt)
                            /* my back is against the wall; i've already done
                             - ungetc, and there's no room for this one
                             */
                            return(EOF);
                    stream->_ptr++;
            }
            if (stream->_flag & _IOSTRG) {
                /* If stream opened by sscanf do not modify buffer */
                    if (*--stream->_ptr != (char)ch) {
                            ++stream->_ptr;
                            return(EOF);
                    }
            } else
                    *--stream->_ptr = (char)ch;
            stream->_cnt++;
            stream->_flag &= ~_IOEOF;
            stream->_flag |= _IOREAD;       /* may already be set */
            return(0xff & ch);
    }

可以看到，当_getbuf分配不成功时，_base指针会指向_charbuf作为一个1 int长度的空间（可能是因为平台的原因，这里将1int理解为2个char），此时的_buffersize就为2。通过_charbuf的使用，也保证了当buffer分配失败时，也能正常从文件中读取数据。

接下来看下getc函数的实现：

    #define getc(_stream)     (--(_stream)->_cnt >= 0 \
                ? 0xff & *(_stream)->_ptr++ : _filbuf(_stream))

表达式 (--(_stream)->_cnt >= 0 ? 0xff & *(_stream)->_ptr++ : _filbuf(_stream)) 的意义为：

当数据存储区域剩余字符的数目大于0的时候，减少一个剩余字符计数，返回0xff & *(_stream)->_ptr++；

否则，调用_filbuf(_stream)。

通过0xff & *(_stream)->_ptr++，getc函数获得了当前文件定位符指向的内容（只取8位？难道char还能不止8位吗？），然后文件定位符前移

剩下函数_filbuf(_stream)，显然其中含有C语言从文件中获取文本的方法（前面都是文本的管理机制）：

    int __cdecl _filbuf (
            FILE *str
            )
    {
            REG1 FILE *stream;
            _ASSERTE(str != NULL);
            /* Init pointer to _iob2 entry. */
            stream = str;
            if (!inuse(stream) || stream->_flag & _IOSTRG)
                    return(_TEOF);
            if (stream->_flag & _IOWRT) {
                    stream->_flag |= _IOERR;
                    return(_TEOF);
            }
            stream->_flag |= _IOREAD;
            /* Get a buffer, if necessary. */
            if (!anybuf(stream))
                    _getbuf(stream);
            else
                    stream->_ptr = stream->_base;
            stream->_cnt = _read(_fileno(stream), stream->_base, stream->_bufsiz);
            if ((stream->_cnt == 0) || (stream->_cnt == -1)) {
                         stream->_flag |= stream->_cnt ? _IOERR : _IOEOF;
                    stream->_cnt = 0;
                    return(_TEOF);
            }

            if (  !(stream->_flag & (_IOWRT|_IORW)) &&
                  ((_osfile_safe(_fileno(stream)) & (FTEXT|FEOFLAG)) ==
                    (FTEXT|FEOFLAG)) )
                    stream->_flag |= _IOCTRLZ;
            /* Check for small _bufsiz (_SMALL_BUFSIZ). If it is small and
               if it is our buffer, then this must be the first _filbuf after
               an fseek on a read-access-only stream. Restore _bufsiz to its
               larger value (_INTERNAL_BUFSIZ) so that the next _filbuf call,
               if one is made, will fill the whole buffer. */
            if ( (stream->_bufsiz == _SMALL_BUFSIZ) && (stream->_flag &
                  _IOMYBUF) && !(stream->_flag & _IOSETVBUF) )
            {
                    stream->_bufsiz = _INTERNAL_BUFSIZ;
            }
            stream->_cnt--;
            return(0xff & *stream->_ptr++);
    }

在语句stream->_cnt = _read(_fileno(stream), stream->_base, stream->_bufsiz);中通过调用_read函数来将原来在硬盘文件中的数据读取到程序数据段的栈空间中，具体的功能看下_read的代码：

    /***
    *int _read(fh, buf, cnt) - read bytes from a file handle
    *
    *Purpose:
    -       Attempts to read cnt bytes from fh into a buffer.
    -       If the file is in text mode, CR-LF's are mapped to LF's, thus
    -       affecting the number of characters read.  This does not
    -       affect the file pointer.
    *
    -       NOTE:  The stdio _IOCTRLZ flag is tied to the use of FEOFLAG.
    -       Cross-reference the two symbols before changing FEOFLAG's use.
    *
    *Entry:
    -       int fh - file handle to read from
    -       char *buf - buffer to read into
    -       int cnt - number of bytes to read
    *
    *Exit:
    -       Returns number of bytes read (may be less than the number requested
    -       if the EOF was reached or the file is in text mode).
    -       returns -1 (and sets errno) if fails.
    *
    *Exceptions:
    *
    *******************************************************************************/

    int __cdecl _read (
            int fh,
            void *buf,
            unsigned cnt
            )
    {
            int bytes_read;                 /* number of bytes read */
            char *buffer;                   /* buffer to read to */
            int os_read;                    /* bytes read on OS call */
            char *p, *q;                    /* pointers into buffer */
            char peekchr;                   /* peek-ahead character */
            ULONG filepos;                  /* file position after seek */
            ULONG dosretval;                /* o.s. return value */

            /* validate fh */
            if ( ((unsigned)fh >= (unsigned)_nhandle) ||
                 !(_osfile(fh) & FOPEN) )
            {
                /* bad file handle */
                errno = EBADF;
                _doserrno = 0;              /* not o.s. error */
                return -1;
            }
            bytes_read = 0;                 /* nothing read yet */
            buffer = buf;
            if (cnt == 0 || (_osfile(fh) & FEOFLAG)) {
                /* nothing to read or at EOF, so return 0 read */
                return 0;
            }
            if ((_osfile(fh) & (FPIPE|FDEV)) && _pipech(fh) != LF) {
                /* a pipe/device and pipe lookahead non-empty: read the lookahead
                 - char */
                *buffer++ = _pipech(fh);
                ++bytes_read;
                --cnt;
                _pipech(fh) = LF;           /* mark as empty */
            }

            /* read the data */

            if ( !ReadFile( (HANDLE)_osfhnd(fh), buffer, cnt, (LPDWORD)&os_read,
                            NULL ) )
            {
                /* ReadFile has reported an error. recognize two special cases.
                 *
                 -      1. map ERROR_ACCESS_DENIED to EBADF
                 *
                 -      2. just return 0 if ERROR_BROKEN_PIPE has occurred. it
                 -         means the handle is a read-handle on a pipe for which
                 -         all write-handles have been closed and all data has been
                 -         read. */

                if ( (dosretval = GetLastError()) == ERROR_ACCESS_DENIED ) {
                    /* wrong read/write mode should return EBADF, not EACCES */
                    errno = EBADF;
                    _doserrno = dosretval;
                    return -1;
                }
                else if ( dosretval == ERROR_BROKEN_PIPE ) {
                    return 0;
                }
                else {
                    _dosmaperr(dosretval);
                    return -1;
                }
            }
            bytes_read += os_read;          /* update bytes read */
            if (_osfile(fh) & FTEXT) {
                /* now must translate CR-LFs to LFs in the buffer */

                /* set CRLF flag to indicate LF at beginning of buffer */
                if ( (os_read != 0) && (*(char *)buf == LF) )
                    _osfile(fh) |= FCRLF;
                else
                    _osfile(fh) &= ~FCRLF;

                /* convert chars in the buffer: p is src, q is dest */
                p = q = buf;
                while (p < (char *)buf + bytes_read) {
                    if (*p == CTRLZ) {
                        /* if fh is not a device, set ctrl-z flag */
                        if ( !(_osfile(fh) & FDEV) )
                            _osfile(fh) |= FEOFLAG;
                        break;              /* stop translating */
                    }
                    else if (*p != CR)
                        *q++ = *p++;
                    else {
                        /* *p is CR, so must check next char for LF */
                        if (p < (char *)buf + bytes_read - 1) {
                            if (*(p+1) == LF) {
                                p += 2;
                                *q++ = LF;  /* convert CR-LF to LF */
                            }
                            else
                                *q++ = *p++;    /* store char normally */
                        }
                        else {
                            /* This is the hard part.  We found a CR at end of
                               buffer.  We must peek ahead to see if next char
                               is an LF. */
                            ++p;

                            dosretval = 0;
                            if ( !ReadFile( (HANDLE)_osfhnd(fh), &peekchr, 1,
                                            (LPDWORD)&os_read, NULL ) )
                                dosretval = GetLastError();
                            if (dosretval != 0 || os_read == 0) {
                                /* couldn't read ahead, store CR */
                                *q++ = CR;
                            }
                            else {
                                /* peekchr now has the extra character -- we now
                                   have several possibilities:
                                   1. disk file and char is not LF; just seek back
                                      and copy CR
                                   2. disk file and char is LF; seek back and
                                      discard CR
                                   3. disk file, char is LF but this is a one-byte
                                      read: store LF, don't seek back
                                   4. pipe/device and char is LF; store LF.
                                   5. pipe/device and char isn't LF, store CR and
                                      put char in pipe lookahead buffer. */
                                if (_osfile(fh) & (FDEV|FPIPE)) {
                                    /* non-seekable device */
                                    if (peekchr == LF)
                                        *q++ = LF;
                                    else {
                                        *q++ = CR;
                                        _pipech(fh) = peekchr;
                                    }
                                }
                                else {
                                    /* disk file */
                                    if (q == buf && peekchr == LF) {
                                        /* nothing read yet; must make some
                                           progress */
                                        *q++ = LF;
                                    }
                                    else {
                                        /* seek back */
                                        filepos = _lseek_lk(fh, -1, FILE_CURRENT);
                                        if (peekchr != LF)
                                            *q++ = CR;
                                    }
                                }
                            }
                        }
                    }
                }

                /* we now change bytes_read to reflect the true number of chars
                   in the buffer */
                bytes_read = q - (char *)buf;
            }
            return bytes_read;              /* and return */
    }

通过调用ReadFile的WINAPI函数，就可以完成从磁盘中把文件的内容读到数据段的栈内了。

    BOOL WINAPI ReadFile(
      _In_         HANDLE hFile,
      _Out_        LPVOID lpBuffer,
      _In_         DWORD nNumberOfBytesToRead,
      _Out_opt_    LPDWORD lpNumberOfBytesRead,
      _Inout_opt_  LPOVERLAPPED lpOverlapped
    );

通过使用已经分配好的文件句柄，将数据读到lpBuffer指向的内存区域，读入的最大数据量通过nNumberOfBytesToRead变量进行设定，通过lpNumberOfBytesRead返回实际读取的字节数。

_read函数在读到数据之后，如果设定的模式为文本流，则需要解决文本流中的换行回车的问题，这里就不深究了。有兴趣可以看文章：

http://www.360doc.com/content/11/0113/20/3508740_86319358.shtml

这样，我们的getc就成功将FILE类的_cnt设定为了剩余字符的数目，将_base指向了通过栈上存储数据的地址。然后，只要将当前文件位置指针_ptr中的数据返回，_ptr指向下一个位置，_cnt减一就好了。

这时候，我们回过头来看FILE结构，就可以知道里面成员变量的意义了。

    typedef struct _iobuf {
        char *_ptr; 
        int _cnt; 
        char *_base; 
        int _flag; 
        int _file; 
        int _charbuf; 
        int _bufsiz; 
        char *_tmpfname;
    }FILE;

_ptr为数据存储区域（可为缓冲区或_charbuf对应的存储区域）当前位置指针。

_cnt为数据存储区域剩余字符数目。

_base为数据存储区域首地址。

_flag为当前I/O控制块所控制的I/O操作类型，有读、写等。

_file为对应的文件描述符（详见博文《走进C标准库(2)》）

_charbuf为缓冲区申请失败时的一个2字节的数据存储区

_bufsiz为缓冲区大小

_tmpfname为所使用存储一些中间操作数据的临时文件。

到这里，我们就可以回答开始时的3个问题了。

1.在getc和ungetc中如果没有分配buffer，都会尝试给I/O控制块分配buffer，但是ungetc分配完buffer后不会尝试从文件中将数据读到buffer中。

2.可以在buffer的首地址处存储一个字符。如果_ptr指在_base，那么数据存储区域为空（为从文件中读取数据）时，ungetc才能在首地址处添加一个字符。

3.多次ungetc使_ptr到达_base之后，就无法再进行ungetc了。