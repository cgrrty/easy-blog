### 走进C标准库(5)——"stdio.h"中的其他部分函数

函数介绍来自：http://ganquan.info/standard-c/

<br />

函数名: freopen 

功  能: 替换一个流 

用  法: FILE *freopen(char *filename, char *type, FILE *stream); 

    FILE * __cdecl _tfreopen (
            const _TSCHAR *filename,
            const _TSCHAR *mode,
            FILE *str
            )
    {
            REG1 FILE *stream;
            FILE *retval;

            _ASSERTE(filename != NULL);
            _ASSERTE(*filename != _T('\0'));
            _ASSERTE(mode != NULL);
            _ASSERTE(str != NULL);

            /* Init stream pointer */
            stream = str;

            _lock_str(stream);

            /* If the stream is in use, try to close it. Ignore possible
             - error (ANSI 4.9.5.4). */
            if ( inuse(stream) )
                    _fclose_lk(stream);

            stream->_ptr = stream->_base = NULL;
            stream->_cnt = stream->_flag = 0;
            retval = _openfile(filename,mode,_SH_DENYNO,stream);

            _unlock_str(stream);
            return(retval);
    }

调用_fclose_lk断开流stream与文件的连接，然后将stream中的变量恢复到初始状态，并释放其中分配的缓冲区。最后调用_openfile重新建立新的文件描述符以及文件句柄之间的连接。

<br />

函数名: fclose()

功 能: 关闭一个流。

用 法: int fclose(FILE *stream);

    int __cdecl fclose (
            FILE *str
            )
    {
            REG1 FILE *stream;
            REG2 int result = EOF;

            /* Init near stream pointer */
            stream = str;

            if (stream->_flag & _IOSTRG) {
                    stream->_flag = 0;
                    return(EOF);
            }

    #endif  /* _MT */

            _ASSERTE(str != NULL);

            if (inuse(stream)) {

                    /* Stream is in use:
                           (1) flush stream
                           (2) free the buffer
                           (3) close the file
                           (4) delete the file if temporary
                    */

                    result = _flush(stream);
                    _freebuf(stream);

                    if (_close(_fileno(stream)) < 0)
                            result = EOF;

                    else if ( stream->_tmpfname != NULL ) {
                            /*
                             - temporary file (i.e., one created by tmpfile()
                             - call). delete, if necessary (don't have to on
                             - Windows NT because it was done by the system when
                             - the handle was closed). also, free up the heap
                             - block holding the pathname.
                             */
                            _free_crt(stream->_tmpfname);
                    stream->_tmpfname = NULL;
                    }

            }

            stream->_flag = 0;
            return(result);
    }

调用_freebuf，将stream中的变量恢复到初始状态，都是释放其中分配的缓冲区。然后调用_close(_fileno(stream))，在文件描述符管理单元删除该文件描述符，并调用CloseHandle关闭在操作系统中打开的文件句柄。如果有临时文件名被使用，也释放掉。

<br />

函数名: feof 

功  能: 检测流上的文件结束符 

用  法: int feof(FILE *stream); 

    int __cdecl feof (
            FILE *stream
            )
    {
            return( ((stream)->_flag & _IOEOF) );
    }

返回检测stream->_flag是否被标记了_IOEOF，该标记在其它函数中遇到EOF情形时被添加。

函数名: ferror 

<br />

功  能: 检测流上的错误 

用  法: int ferror(FILE *stream); 

    int __cdecl ferror (
            FILE *stream
            )
    {
            return( ((stream)->_flag & _IOERR) );
    }

返回检测stream->_flag是否被标记了_IOERR。

<br />

函数名: fflush 

功 能: 清除文件缓冲区，文件以写方式打开时将缓冲区内容写入文件

用  法: int fflush(FILE *stream); 

    int __cdecl fflush (
            REG1 FILE *str
            )
    {

            /* if stream is NULL, flush all streams */
            if ( str == NULL ) {
                    return(flsall(FFLUSHNULL));
            }

            if (_flush(str) != 0) {
                    /* _flush failed, don't attempt to commit */
                    return(EOF);
            }

            /* lowio commit to ensure data is written to disk */
            if (str->_flag & _IOCOMMIT) {
                    return (_commit(_fileno(str)) ? EOF : 0);
            }
            return (0);
    }

当传入fflush中的stream为NULL时，则_fflush所有写的流。否则，_fflush当前stream，成功的话，调用_commit写入到硬盘中。

flsall中FFLUSHNULL参数意义如下：

/*
* FFLUSHNULL functionality: fflush the write
* stream and kept track of the error, if one
* occurs
*/

在_fflush()函数中会使用：

stream->_ptr = stream->_base;
stream->_cnt = 0;

将缓冲区清空。

如果stream仅具有写标记且buffer已成功分配（if ((stream->_flag & (_IOREAD | _IOWRT)) == _IOWRT && bigbuf(stream) && (nchar = stream->_ptr - stream >_base) > 0)）的话，则调用_write将缓冲区里的内容写到相关的文件句柄中。

注意，操作系统为了减少读写硬盘的次数，也会建立文件缓冲区，并不立刻将数据写入文件，而是先把数据累计到缓冲区，再以块为单位批量输出到文件中。

_write中调用的WriteFile就是将相关内容写到了文件句柄对应的缓冲区中。这一点可以从putc函数产生的现象看到。当执行一句putc语句时，内容并没有马上被写入到流对应的文件中的。

而fflush会强制将I/O控制块缓冲区的内容写入到硬盘中。这就通过在_commit中调用FlushFileBuffers，强行将文件句柄对应的缓冲区的内容写入到了硬盘中。

<br />

函数名: fgetpos 

功  能:取得当前文件的指针所指的位置，并把该指针所指的位置数存放到pos所指的对象中。pos值以内部格式存储,仅由fgetpos和fsetpos使用。其中fsetpos的功能与fgetpos相反，为了详细介绍，将在后节给与说明。

返回值：成功返回0，失败返回非0，并设置errno。

用  法: int fgetpos(FILE *stream);

    int __cdecl fgetpos (
            FILE *stream,
            fpos_t *pos
            )
    {
    #ifdef _MAC
            int posl = ftell(stream);

            *pos = (fpos_t) posl;

            if ( posl != -1L )
                    return(0);
            else
                    return(-1);
    }

通过ftell获得当前文件指针相对文件首的偏移字节，将该位置存储到一个fpos_t对象中。如果ftell是成功的，则返回0表示fgetpos成功，否则，返回-1。

<br />

函数名: fsetpos 

功  能:将文件指针定位在pos指定的位置上。该函数的功能与前面提到的fgetpos相反，是将文件指针fp按照pos指定的位置在文件中定位。pos值以内部格式存储,仅由fgetpos和fsetpos使用。

返回值：成功返回0，否则返回非0。

用  法: int fsetpos(FILE *stream, const fpos_t *pos); 

    int __cdecl fsetpos (
            FILE *stream,
            const fpos_t *pos
            )
    {
            return( fseek(stream, (long) *pos, SEEK_SET) );
    }

直接调用fseek，在参数为SEEK_SET的情况下实现。

<br />

函数名: fread 

功  能: 从一个文件流中读数据，最多读取count个元素，每个元素size字节，如果调用成功返回实际读取到的元素个数，如果不成功返回 0 

用  法: int fread(void *ptr, int size, int nitems, FILE *stream); 

    size_t __cdecl fread (
            void *buffer,
            size_t size,
            size_t num,
            FILE *stream
            )
    {
            char *data;                     /* point to where should be read next */
            unsigned total;                 /* total bytes to read */
            unsigned count;                 /* num bytes left to read */
            unsigned bufsize;               /* size of stream buffer */
            unsigned nbytes;                /* how much to read now */
            unsigned nread;                 /* how much we did read */
            int c;                          /* a temp char */

            /* initialize local vars */
            data = buffer;

            if ( (count = total = size * num) == 0 )
                    return 0;

            if (anybuf(stream))
                    /* already has buffer, use its size */
                    bufsize = stream->_bufsiz;
            else
                    bufsize = BUFSIZ;
            /* here is the main loop -- we go through here until we're done */
            while (count != 0) {
                    /* if the buffer exists and has characters, copy them to user
                       buffer */
                    if (anybuf(stream) && stream->_cnt != 0) {
                            /* how much do we want? */
                            nbytes = (count < (unsigned)stream->_cnt) ? count : stream->_cnt;
                            memcpy(data, stream->_ptr, nbytes);

                            /* update stream and amt of data read */
                            count -= nbytes;
                            stream->_cnt -= nbytes;
                            stream->_ptr += nbytes;
                            data += nbytes;
                    }
                    else if (count >= bufsize) {
                            /* If we have more than bufsize chars to read, get data
                               by calling read with an integral number of bufsiz
                               blocks.  Note that if the stream is text mode, read
                               will return less chars than we ordered. */

                            /* calc chars to read -- (count/bufsize) * bufsize */
                            nbytes = ( bufsize ? (count - count % bufsize) :
                                       count );

                            nread = _read(_fileno(stream), data, nbytes);
                            if (nread == 0) {
                                    /* end of file -- out of here */
                                    stream->_flag |= _IOEOF;
                                    return (total - count) / size;
                            }
                            else if (nread == (unsigned)-1) {
                                    /* error -- out of here */
                                    stream->_flag |= _IOERR;
                                    return (total - count) / size;
                            }

                            /* update count and data to reflect read */
                            count -= nread;
                            data += nread;
                    }
                    else {
                            /* less than bufsize chars to read, so call _filbuf to
                               fill buffer */
                            if ((c = _filbuf(stream)) == EOF) {
                                    /* error or eof, stream flags set by _filbuf */
                                    return (total - count) / size;
                            }

                            /* _filbuf returned a char -- store it */
                            *data++ = (char) c;
                            --count;

                            /* update buffer size */
                            bufsize = stream->_bufsiz;
                    }
            }

            /* we finished successfully, so just return num */
            return num;
    }

在fread中处理3种情况：I/O控制块缓冲区中有剩余数据，则直接从中拿出需要的数据（如果剩余数据不够，则只拿缓冲区里有的）；无剩余数据且仍需要的数据数目大于等于缓冲区中已有的数据数目，通过调用_read直接从stream对应的文件中读取需要的数据（需要注意的是，在这种情况下，get data by calling read with an integral number of bufsiz blocks）；无剩余数据且需要的数据数目小于缓冲区，则调用_filbuf新建缓冲区，同时接受_filbuf返回的一个char保证数据正确。

三种情况在一个循环中，直到需要的数据全取出即可。

简单来说就是，缓冲区有数据，就取出来；没了，就到内存拿整数个缓冲区大小的数据，模拟走缓冲区取数据的过程；最后剩余部分，填入缓冲区后取出。

<br />

函数名: fseek 
功  能: 重定位流上的文件指针 
用  法: int fseek(FILE *stream, long offset, int fromwhere); 

使用setFilePointer移动了文件指针，不是移动缓冲区的当前指针。

<br /> 

函数名: ftell 
功  能: 返回当前文件指针 
用  法: long ftell(FILE *stream); 

<br /> 

调用_lseek，找到当前文件首的地址，再加上当前缓冲区内的指针偏移量，得到在文件中的文件指针。（不必纠结于文件指针本身的概念，其意义就是当前指向的文件数据的地址）

<br /> 

函数名: fwrite 
功  能: 写内容到流中 
用  法: int fwrite(void *ptr, int size, int nitems, FILE *stream); 

<br /> 

fread的相反过程，无需细述。

<br /> 

原型：extern void printf(const char *format,...);  

用法：#include <stdio.h>
功能：格式化字符串输出

主要实现了各种控制符的逻辑，WriteConsole实现了到控制台的输出。

<br /> 

其他的函数就暂时不讨论了。    