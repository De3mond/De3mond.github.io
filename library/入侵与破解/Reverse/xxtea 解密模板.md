# xxtea 解密模板

```
#include <stdio.h>  
#include <stdint.h>  
#define DELTA 0x9e3779b9  
#define MX (((z>>5^y<<2) + (y>>3^z<<4)) ^ ((sum^y) + (key[(p&3)^e] ^ z)))  
  
void btea(uint32_t *v, int n, uint32_t const key[4])  
{  
    uint32_t y, z, sum;  
    unsigned p, rounds, e;  
    if (n > 1)            /* Coding Part */  
    {  
        rounds = 6 + 52/n;  
        sum = 0;  
        z = v[n-1];  
        do  
        {  
            sum += DELTA;  
            e = (sum >> 2) & 3;  
            for (p=0; p<n-1; p++)  
            {  
                y = v[p+1];  
                z = v[p] += MX;  
            }  
            y = v[0];  
            z = v[n-1] += MX;  
        }  
        while (--rounds);  
    }  
    else if (n < -1)      /* Decoding Part */  
    {  
        n = -n;  
        rounds = 6 + 52/n;  
        sum = rounds*DELTA;  
        y = v[0];  
        do  
        {  
            e = (sum >> 2) & 3;  
            for (p=n-1; p>0; p--)  
            {  
                z = v[p-1];  
                y = v[p] -= MX;  
            }  
            z = v[n-1];  
            y = v[0] -= MX;  
            sum -= DELTA;  
        }  
        while (--rounds);  
    }  
}  
  
  
int main()  
{  
    unsigned char v[40]= {0x18, 0x4E, 0x8E, 0x7F, 0x2E, 0x69, 0xB7, 0x02, 0xEE, 0xAA, 0x50, 0x39, 0x90, 0xDE, 0xE5, 0x9F, 
    0xAE, 0x4C, 0x4D, 0x06, 0x93, 0x71, 0x64, 0x20, 0x8B, 0x02, 0x34, 0xB8, 0x3C, 0xA1, 0x88, 0x4A, 
    0x21, 0x67, 0x1A, 0x37, 0x83, 0xD1, 0xF2, 0xB1};  
    uint32_t const k[4]= {1,2,3,4};  
    int n=-10; //n的绝对值表示v的长度，取正表示加密，取负表示解密  
    // v为要加密的数据是两个32位无符号整数  
    // k为加密解密密钥，为4个32位无符号整数，即密钥长度为128位  
    //printf("%u %u\n",v[0],v[1]);  
    btea(v, n, k);  
    puts(v);
    /*for (int i=0;i<40;i++){
        printf("%c",v[i]);   
    }*/

    return 0;  
}  
```