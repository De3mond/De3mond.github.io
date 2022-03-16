# RC4 解密模板

```
import hashlib
def crypt(data,key) :
    s = [0] * 256
    for i in range(256) :
        s[i] = i
    print(s)
    j = 0
    for i in range(256) :
        j = (j + s[i] + key[i % len(key)]) % 256
        print(j)
        s[i], s[j] = s[j], s[i]
    i = 0
    j = 0
    res = ""
    for c in data :
        i = (i + 1) % 256
        j = (j + s[i]) % 256
        s[i], s[j] = s[j], s[i]
        res = res + chr(c ^ s[(s[i] + s[j]) % 256])
    return res


key2 = "ACTF"
key = []
for i in key2:
    key.append(ord(i))
print key
data = [196, 197, 137, 138, 204, 156, 24, 68, 8, 10, 106, 41, 210, 228, 46]
print crypt(data,key)
```