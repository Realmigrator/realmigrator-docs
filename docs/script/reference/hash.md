# Hash Reference

## gkScript main object

| Script                                            | Function                                                         |
| ------------------------------------------------- | ---------------------------------------------------------------- |
| `string gkScript::hashMD5(string data)`           | Use it to MD5-hash the data string. A binary has string returns. |
| `gkScript::hexEncode` or `gkScript::base64Encode` | Get an appropriate string representation                         |
| `Hash gkScript::createMD5Hasher()`                | Will create a hash object                                        |

## gkScript hash object

```c++
class Hash
{
    reset();
    update(string data);
    string finalize();
};
```

#### Hash sample

```shell
eval> var md5 = gkScript.hexEncode(gkScript.hashMD5("The quick brown fox jumps over the lazy dog"));
9E107D9D372BB6826BD81D3542A419D6

eval> var hasher = gkScript.createMD5Hasher();
eval> hasher.update("The quick brown fox");
eval> hasher.update(" jumps over the lazy dog");
eval> gkScript.hexEncode(hasher.finalize());
9E107D9D372BB6826BD81D3542A419D6
```
