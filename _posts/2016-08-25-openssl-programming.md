---
layout: post
title: openssl编程
category: 平台
keywords:
---

### Base64编码——短内容

```cpp
#include <stdio.h>
#include <openssl/evp.h>
int main() {
    unsigned char ibuf[256], obuf[512];
    int ilen = fread(ibuf, 1, sizeof(ibuf), stdin);
    int olen = EVP_EncodeBlock(obuf, ibuf, ilen);
    fwrite(obuf, olen, 1, stdout);
    puts("");
    return 0;
}
```

编译可参考如下命令：

```
$ gcc base64.c -lcrypto
```

### Base64解码——短内容

```cpp
#include <stdio.h>
#include <openssl/evp.h>
int main() {
    unsigned char ibuf[512], obuf[512];
    int ilen = fread(ibuf, 1, sizeof(ibuf), stdin);
    int olen = EVP_DecodeBlock(obuf, ibuf, ilen);
    fwrite(obuf, olen, 1, stdout);
    return 0;
}
```

### Base64编码——长内容

```cpp
#include <stdio.h>
#include <openssl/evp.h>
int main() {
    unsigned char ibuf[256], obuf[512];
    int ilen, olen;
    EVP_ENCODE_CTX ctx;

    EVP_EncodeInit(&ctx);
    while ((ilen = fread(ibuf, 1, sizeof(ibuf), stdin)) > 0) {
        EVP_EncodeUpdate(&ctx, obuf, &olen, ibuf, ilen);
        fwrite(obuf, olen, 1, stdout);
    }
    EVP_EncodeFinal(&ctx, obuf, &olen);
    fwrite(obuf, olen, 1, stdout);
    return 0;
}
```

### Base64解码——长内容

```cpp
#include <stdio.h>
#include <openssl/evp.h>
int main() {
    unsigned char ibuf[512], obuf[512];
    int ilen, olen;
    EVP_ENCODE_CTX ctx;

    EVP_DecodeInit(&ctx);
    while ((ilen = fread(ibuf, 1, sizeof(ibuf), stdin)) > 0) {
        EVP_DecodeUpdate(&ctx, obuf, &olen, ibuf, ilen);
        fwrite(obuf, olen, 1, stdout);
    }
    EVP_DecodeFinal(&ctx, obuf, &olen);
    fwrite(obuf, olen, 1, stdout);
    return 0;
}
```

### 计算哈希值

```cpp
#include <stdio.h>
#include <openssl/md5.h>
int main() {
    unsigned char buf[256], md5[MD5_DIGEST_LENGTH];
    int i, len;

    len = fread(buf, 1, sizeof(buf), stdin);
    MD5(buf, len, md5);

    for (i = 0; i < MD5_DIGEST_LENGTH; i++)
        printf("%02x", md5[i]);
    puts("");
    return 0;
}
```

```cpp
#include <stdio.h>
#include <openssl/md5.h>
int main() {
    unsigned char buf[256], md5[MD5_DIGEST_LENGTH];
    MD5_CTX ctx;
    int i, len;

    MD5_Init(&ctx);
    while ((len = fread(buf, 1, sizeof(buf), stdin)) > 0)
        MD5_Update(&ctx, buf, len);
    MD5_Final(md5, &ctx);

    for (i = 0; i < MD5_DIGEST_LENGTH; i++)
        printf("%02x", md5[i]);
    puts("");
    return 0;
}
```

采用SHA、SHA1算法时类似，相关接口定义在openssl/sha.h中。

### 3DES加解密

```cpp
#include <stdio.h>
#include <string.h>
#include <openssl/evp.h>
int main() {
    unsigned char key[24], iv[8], ibuf[256], obuf[256];
    EVP_CIPHER_CTX ctx;
    int ilen, olen;

    memset(key, 0x61, sizeof(key));
    memset(iv, 0x62, sizeof(iv));

    EVP_CIPHER_CTX_init(&ctx);
    EVP_EncryptInit_ex(&ctx, EVP_des_ede3_cbc(), NULL, key, iv);

    while ((ilen = fread(ibuf, 1, sizeof(ibuf), stdin)) > 0) {
        EVP_EncryptUpdate(&ctx, obuf, &olen, ibuf, ilen);
        fwrite(obuf, olen, 1, stdout);
    }
    EVP_EncryptFinal_ex(&ctx, obuf, &olen);
    fwrite(obuf, olen, 1, stdout);
    return 0;
}
```

```cpp
#include <stdio.h>
#include <string.h>
#include <openssl/evp.h>
int main() {
    unsigned char key[24], iv[8], ibuf[256], obuf[256];
    EVP_CIPHER_CTX ctx;
    int ilen, olen;

    memset(key, 0x61, sizeof(key));
    memset(iv, 0x62, sizeof(iv));

    EVP_CIPHER_CTX_init(&ctx);
    EVP_DecryptInit_ex(&ctx, EVP_des_ede3_cbc(), NULL, key, iv);

    while ((ilen = fread(ibuf, 1, sizeof(ibuf), stdin)) > 0) {
        EVP_DecryptUpdate(&ctx, obuf, &olen, ibuf, ilen);
        fwrite(obuf, olen, 1, stdout);
    }
    EVP_DecryptFinal_ex(&ctx, obuf, &olen);
    fwrite(obuf, olen, 1, stdout);
    return 0;
}
```

### AES加解密

代码几乎与DES相同，只是密钥和初始向量长度有区别，并且使用的算法不一样。

```cpp
#include <stdio.h>
#include <string.h>
#include <openssl/evp.h>
int main() {
    unsigned char key[16], iv[16], ibuf[256], obuf[256];
    EVP_CIPHER_CTX ctx;
    int ilen, olen;

    memset(key, 0x61, sizeof(key));
    memset(iv, 0x62, sizeof(iv));

    EVP_CIPHER_CTX_init(&ctx);
    EVP_EncryptInit_ex(&ctx, EVP_aes_128_cbc(), NULL, key, iv);

    while ((ilen = fread(ibuf, 1, sizeof(ibuf), stdin)) > 0) {
        EVP_EncryptUpdate(&ctx, obuf, &olen, ibuf, ilen);
        fwrite(obuf, olen, 1, stdout);
    }
    EVP_EncryptFinal_ex(&ctx, obuf, &olen);
    fwrite(obuf, olen, 1, stdout);
    return 0;
}
```

```cpp
#include <stdio.h>
#include <string.h>
#include <openssl/evp.h>
int main() {
    unsigned char key[16], iv[16], ibuf[256], obuf[256];
    EVP_CIPHER_CTX ctx;
    int ilen, olen;

    memset(key, 0x61, sizeof(key));
    memset(iv, 0x62, sizeof(iv));

    EVP_CIPHER_CTX_init(&ctx);
    EVP_DecryptInit_ex(&ctx, EVP_aes_128_cbc(), NULL, key, iv);

    while ((ilen = fread(ibuf, 1, sizeof(ibuf), stdin)) > 0) {
        EVP_DecryptUpdate(&ctx, obuf, &olen, ibuf, ilen);
        fwrite(obuf, olen, 1, stdout);
    }
    EVP_DecryptFinal_ex(&ctx, obuf, &olen);
    fwrite(obuf, olen, 1, stdout);
    return 0;
}
```

下面对程序的正确性做验证。

```
$ gcc aes_en.c -o aes_en -lcrypto
$ gcc aes_de.c -o aes_de -lcrypto
$ echo "chenfy" | ./aes_en | ./aes_de
chenfy
$ key=61616161616161616161616161616161
$  iv=62626262626262626262626262626262
$ echo "chenfy" | ./aes_en | openssl enc -d -aes-128-cbc -K $key -iv $iv
chenfy
$ echo "chenfy" | openssl enc -e -aes-128-cbc -K $key -iv $iv | ./aes_de
chenfy
```

