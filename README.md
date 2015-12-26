# axel
axel
逐行分析axel的源码
先以一个http下载连接，并且不涉及到镜像的开始分析

axel_new 
axel_t *axel_new( conf_t *conf, int count, void *url )
根据配置文件，数量，和url初始化一个axel结构体，整个下载过程就是围绕axel这个结构体来进行的

conn_set
int conn_set( conn_t *conn, char *set_url );
根据连接(conn里面只有从axel_new里面获取到的conf结构体)和url 转换成一个conn_t的结构体
Convert an URL to a conn_t structure  

http_encode
void http_encode( char *s )

conn_init
int conn_init( conn_t *conn );
根据conn链接状态去判断下载类型，然后用ftp_connect或者http_connect初始化链接(仅仅建立socket)

                            int http_connect( http_t *conn, int proto, char *proxy, char *host, int port, char *user, char *pass )
                            调用tcp_connect来构造http的链接

                                                        int tcp_connect( char *hostname, int port, char *local_if );
                                                        根据ip和端口建立socket连接

conn_info
int conn_info( conn_t *conn )
根据连接初始化连接的各种数据，包括文件类型，大小等等（获取http头部，然后分析）
                        int conn_setup( conn_t *conn )
                                        根据配置文件构造http请求头和地址
                                        http_get( http_t *conn, char *lurl )
                                                                        构造http头部信息
                                                                         void http_addheader( http_t *conn, char *format, ... )
                                                                           增加http头字段


                    int conn_exec( conn_t *conn )
                     执行http请求，获取头部信息和http状态码                            
                                                                    int http_exec( http_t *conn )
                                                                    访问网站，获取头部信息，包括类型，大小等等

                    void conn_disconnect( conn_t *conn )
                      关闭请求
                                                                    void http_disconnect( http_t *conn )   
                                                                     关闭http文件描述符

char *conn_url( conn_t *conn )
生成一个带端口的完整url

axel_new 分析完成
根据配置文件，数量，和url初始化一个axel结构体，已经构造一个http请求和访问请求，得到返回头，和状态码

text.c
后续是判断是否指定了-o重定向输出到文件，然后判断文件是否存在

axel_open
int axel_open( axel_t *axel )
Open a local file to store the downloaded data 
                       void axel_divide( axel_t *axel )
                       文件大小，除以连接数，分开每个连接的开始结束地址

void axel_start( axel_t *axel )
Start downloading
用pthread_create创造线程

最主要的核心函数
axel_do( axel );
void axel_do( axel_t *axel )


学到的知识：

1，当htt请求头中带Range: bytes=1-后，那么http返回的状态码为HTTP/1.1 206 Partial Content
   参考http://everet.org/http-status-206-partial-content.html
   
2, define用法
   a,定义一个变量替换#define MAX_STRING              1024
   b,定义一个宏#define max( a, b )             ( (a) > (b) ? (a) : (b) )
   参考http://www.cprogramming.com/reference/preprocessor/define.html
strchr and strstr的区别
实现可变函数va_list
分析到axel.c的115行的conn_info了
