---
title: udp编程
date: 2016-03-16 17:31:36
tags:
---

创建udp socket 监听2222端口接收数据
发送数据端口为3333

### 代码部分
```cpp
#include<sys/types.h>  
#include<sys/socket.h>  
#include<unistd.h>  
#include<netinet/in.h>  
#include<arpa/inet.h>  
#include<stdio.h>  
#include<stdlib.h>  
#include<errno.h>  
#include<netdb.h>  
#include<stdarg.h>  
#include<string.h>
#include<pthread.h>

#define SERV_PORT_UDP 2222
#define SERV_SEND_PORT_UDP 3333
#define RECV_BUF_LMT 1024
#define HEART_BEAT 10
#define QUEUE_MAX_EVENTS 5
#define SZ_FR_LEN_MIN 9 //sizeof(SHUNZHOU_PRO_HEAD_S)+sizeof(SHUNZHOU_PRO_SUM_S)

#define FALSE 0
#define TRUE  1

#define FR_HEART_BEAT 0
#define FR_SINGLE_ADD_DEV 1
#define FR_BATCH_ADD_DEV 2
#define FR_BATCH_ADD_CLOSE 3
#define FR_ADD_SUCC_TIP 4
#define FR_DEL_DEV 5
#define FR_DEV_GROUP 6
#define FR_DEV_STA_QUE 7
#define FR_GET_DEV_STA 8
#define FR_SINGLE_COMM_SEND 9
#define FR_DATA_STA_UP 0x0a
#define FR_GROUP_COMM_SEND 0x0b

#define VOID void
#define STATIC static
#define SIZEOF sizeof
typedef unsigned char BYTE;
typedef BYTE *PBYTE;
typedef unsigned short WORD;
typedef WORD *PWORD;
typedef unsigned int DWORD;
typedef DWORD *PDWORD;
typedef unsigned int UINT;
typedef int INT;
typedef unsigned int *PUINT;

#define WORD_SWAP(X) ((X <<8) | (X >> 8))   //short
#define DWORD_SWAP(X)(((X)&0xff)<<24) + \
                     (((X)&0xff00)<<8) + \
                     (((X)&0xff0000)>>8) + \
                     (((X)&0xff000000)>>24)  //int

#pragma pack(1)
typedef struct
{
    WORD fr_head; // 0x55aa
    BYTE fr_seq;
    BYTE fr_type;
    BYTE fr_ret;
    WORD fr_len;
    BYTE fr_data[0];
}SZ_PRO_HEAD_S;

typedef struct {
    WORD fr_sum;
}SZ_PRO_SUM_S;

typedef struct
{
    WORD len;       
    BYTE is_ret;   
    BYTE data[0];  
}SZ_SEND_DATA_S;
#pragma pack()

typedef struct {
    INT udp_fd;
    INT udp_send_fd;
    INT time_out_cnt;
    UINT recv_cout;
    UINT seq;
    struct sockaddr_in udp_addr;
    struct sockaddr_in server_send_addr;
    BYTE recv_buf[RECV_BUF_LMT];
}SZ_PRO_CNTL_S;

typedef enum {
    SZ_UDP_RECV = 0,
    SZ_UDP_PROC,
}SZ_UDP_RECV_S;

/***********************************************************
*************************variable define********************
***********************************************************/
STATIC SZ_PRO_CNTL_S sz_pro_cntl;
STATIC int sz_udp_send_data(BYTE fr_type,\
                            BYTE fr_ret,BYTE *fr_data,\
                            UINT fr_len,BYTE is_ret);
/***********************************************************
*************************function define********************
***********************************************************/
STATIC WORD getCheckSum(BYTE *pack, INT pack_len)
{
    WORD check_sum = 0;
    while(--pack_len >= 0)
    {
        check_sum += *pack++;
    }
    return check_sum;
}

int recv_socket(VOID)
{
    int32_t iRet;
    sz_pro_cntl.udp_fd = -1;
    
    _Bool optval = TRUE;
    bzero(&sz_pro_cntl.udp_addr, sizeof(sz_pro_cntl.udp_addr));
    
    sz_pro_cntl.udp_addr.sin_family = AF_INET;
    sz_pro_cntl.udp_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    sz_pro_cntl.udp_addr.sin_port = htons(SERV_PORT_UDP);
    
    sz_pro_cntl.udp_fd = socket(AF_INET, SOCK_DGRAM, 0);
    if(sz_pro_cntl.udp_fd < 0) {
        printf("sz_pro_cntl.udp_fd < 0\n");
        return -1;
    }
    
    setsockopt(sz_pro_cntl.udp_fd,SOL_SOCKET,SO_REUSEADDR,&optval,sizeof(optval));
    iRet = bind(sz_pro_cntl.udp_fd, (struct sockaddr *)&sz_pro_cntl.udp_addr, sizeof(sz_pro_cntl.udp_addr));
    if(iRet < 0) {
        printf("bind err\n");
        return -1;
    }
    return 0;
}

int send_socket(VOID)
{
    int32_t iRet;
    sz_pro_cntl.udp_send_fd = -1;
    
    _Bool optval = TRUE;
    bzero(&sz_pro_cntl.server_send_addr, sizeof(sz_pro_cntl.server_send_addr));
    
    sz_pro_cntl.server_send_addr.sin_family = AF_INET;
    sz_pro_cntl.server_send_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    sz_pro_cntl.server_send_addr.sin_port = htons(SERV_SEND_PORT_UDP);
    
    sz_pro_cntl.udp_send_fd = socket(AF_INET, SOCK_DGRAM, 0);
    if(sz_pro_cntl.udp_send_fd < 0) {
        printf("sz_pro_cntl.udp_send_fd < 0\n");
        return -1;
    }
    
    setsockopt(sz_pro_cntl.udp_send_fd,SOL_SOCKET,SO_REUSEADDR,&optval,sizeof(optval));
    iRet = bind(sz_pro_cntl.udp_send_fd, (struct sockaddr *)&sz_pro_cntl.server_send_addr, sizeof(sz_pro_cntl.server_send_addr));
    if(iRet < 0) {
        printf("bind err hh\n");
        return -1;
    }
    return 0;
}

VOID ty_thread(VOID)
{
    <!-- sz_udp_send_data(i,0,NULL,0,0); -->
}

VOID sz_udp_recv_pro(VOID)
{
    struct sockaddr_in client_addr;
    socklen_t client_addr_len = sizeof(client_addr);
    INT count = 0;
    count = recvfrom(sz_pro_cntl.udp_fd, (sz_pro_cntl.recv_buf + \
        sz_pro_cntl.recv_cout), (1024 - sz_pro_cntl.recv_cout),0,\
        (struct sockaddr*)&client_addr, &client_addr_len);
    sz_pro_cntl.recv_cout += count;
}

STATIC VOID sz_udp_msg_proc(VOID)
{
    printf("enter sz_udp_msg_proc\n");
    INT fr_len = 0, offset = 0;

    while((sz_pro_cntl.recv_cout-offset) >= SZ_FR_LEN_MIN) {
        SZ_PRO_HEAD_S *head=(SZ_PRO_HEAD_S *)(sz_pro_cntl.recv_buf+offset);
        head->fr_head = WORD_SWAP(head->fr_head);
        if(0x55aa != head->fr_head) {
            offset += 2;
            continue;
        }
        // if((head->fr_seq != (sz_pro_cntl.seq-1)) && (head->fr_seq != 0)) {
        //     offset += 3;
        //     continue;
        // }
        if((head->fr_type < 0x00) || (head->fr_type > 0x0b)) {
            offset += 4;
            continue;
        }
        if(head->fr_ret != 0x00) {
            offset += 5;
            continue;  
        }
        head->fr_len = WORD_SWAP(head->fr_len);
        fr_len = SZ_FR_LEN_MIN + head->fr_len;
        if((sz_pro_cntl.recv_cout-offset) < fr_len) {
            break;
        }

        UINT check_num = getCheckSum(head,fr_len-2);
        SZ_PRO_SUM_S *sum = (SZ_PRO_SUM_S *)(sz_pro_cntl.recv_buf+offset+fr_len-2);
        sum->fr_sum = WORD_SWAP(sum->fr_sum);
        if(check_num != sum->fr_sum) {
            offset += fr_len;
            continue;
        } 
        printf("recv succ\n");
        offset += fr_len;
    }

    if(offset > 0) {
        sz_pro_cntl.recv_cout -= offset;
        memmove(sz_pro_cntl.recv_buf,sz_pro_cntl.recv_buf+ \
            offset,sz_pro_cntl.recv_cout);
    }
}

int recv_thread(VOID)
{
    printf("enter ty_thread\n");
    SZ_UDP_RECV_S status = SZ_UDP_RECV;
    while(1) {
        printf("status:%d\n",status);
        switch(status) {
            case SZ_UDP_RECV: {
                sz_udp_recv_pro();
                if(sz_pro_cntl.recv_cout >= SZ_FR_LEN_MIN) {
                    status = SZ_UDP_PROC;
                }
            }
            break;
            case SZ_UDP_PROC: {
                sz_udp_msg_proc();
                status = SZ_UDP_RECV;
            }
            break;
        }
    }
}

VOID main(VOID)
{	
	sz_pro_cntl.seq = 1;
    sz_pro_cntl.recv_cout = 0;
    send_socket();
    recv_socket();
    
    int err;
    pthread_t thread_id;
    err = pthread_create(&thread_id, NULL, &ty_thread, NULL);
    if(err != 0) {
        printf("pthread_create err\n");
    }

    pthread_t thread_id_recv;
    err = pthread_create(&thread_id_recv, NULL, &recv_thread, NULL);
    if(err != 0) {
        printf("pthread_create thread_id_recv err\n");
    }

    pthread_join(thread_id, NULL);
    pthread_join(thread_id_recv, NULL);
}

STATIC int sz_udp_send_data(BYTE fr_type,\
                            BYTE fr_ret,BYTE *fr_data,\
                            UINT fr_len,BYTE is_ret)
{
    UINT send_da_len = sizeof(SZ_PRO_HEAD_S) + fr_len + sizeof(SZ_PRO_SUM_S);
    BYTE *send_da = malloc(send_da_len);
    if(send_da == NULL) {
        return -1;
    }

    SZ_PRO_HEAD_S *sz_head = (SZ_PRO_HEAD_S *)send_da;
    sz_head->fr_head = WORD_SWAP(0x55aa);
    sz_head->fr_seq = sz_pro_cntl.seq;
    sz_head->fr_type = fr_type;
    sz_head->fr_len = WORD_SWAP(fr_len);
    sz_head->fr_ret = fr_ret;
    memcpy(sz_head->fr_data,fr_data,fr_len);
    SZ_PRO_SUM_S *sz_sum = (SZ_PRO_SUM_S *)(send_da + sizeof(SZ_PRO_HEAD_S) + fr_len);
    WORD fr_sum = getCheckSum(send_da, (sizeof(SZ_PRO_HEAD_S) + fr_len));
    sz_sum->fr_sum = WORD_SWAP(fr_sum);
    UINT max = 0;
    if(sz_pro_cntl.seq >= (~max)) {
        sz_pro_cntl.seq = 1;
    }else {
        sz_pro_cntl.seq += 1;
    }
    int ret = sendto(sz_pro_cntl.udp_send_fd,send_da,send_da_len,0,\
                     (struct sockaddr*)&sz_pro_cntl.server_send_addr, SIZEOF(sz_pro_cntl.server_send_addr));
    free(send_da);  
    if(ret <= 0 || \
       ret != send_da_len) {
       printf("sendto Failed,ret:%d,errno:%d",ret,errno);
       close(sz_pro_cntl.udp_send_fd),sz_pro_cntl.udp_send_fd = -1;
       return -1;
    }
    return 0;
}
```