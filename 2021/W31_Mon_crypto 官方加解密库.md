# crypto 官方加解密库

## 1. 为什么用官方crypto，不用第三方库

因为这个加解密库实际很简单，并不需要太多的配置以及学习；且特别是比较敏感的数据，如果代码圈发现有一种算法的加密方式是有漏洞的，第三方库真不一定会及时解决。

## 2. 是什么原因导致我去用它

最近对接华立电表与立方停车，哎，看着java代码发呆，还望这两家公司提供多语言的SDK，特别是这几年日渐火热的golang。

## 3. 怎么使用

我先上**java**代码吧，这样比较有对比性：

AESUtils.java

```java
package src.com.first;

import java.security.Key;
import java.security.NoSuchAlgorithmException;
import java.util.Base64;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;

/**
 * AES加密类
 *
 */
public class AESUtils {
    /**
     * 密钥算法
     */
    private static final String KEY_ALGORITHM = "AES";

    private static final String DEFAULT_CIPHER_ALGORITHM = "AES/ECB/PKCS5Padding";

    /**
     * 初始化密钥
     *
     * @return byte[] 密钥
     * @throws Exception
     */
    public static byte[] initSecretKey() {
        // 返回生成指定算法的秘密密钥的 KeyGenerator 对象
        KeyGenerator kg = null;
        try {
            kg = KeyGenerator.getInstance(KEY_ALGORITHM);
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
            return new byte[0];
        }
        // 初始化此密钥生成器，使其具有确定的密钥大小
        // AES 要求密钥长度为 128
        kg.init(128);
        // 生成一个密钥
        SecretKey secretKey = kg.generateKey();
        return secretKey.getEncoded();
    }

    /**
     * 转换密钥
     *
     * @param key
     *            二进制密钥
     * @return 密钥
     */
    public static Key toKey(byte[] key) {
        // 生成密钥
        return new SecretKeySpec(key, KEY_ALGORITHM);
    }

    /**
     * 加密
     *
     * @param data
     *            待加密数据
     * @param key
     *            密钥
     * @return byte[] 加密数据
     * @throws Exception
     */
    public static byte[] encrypt(byte[] data, Key key) throws Exception {
        return encrypt(data, key, DEFAULT_CIPHER_ALGORITHM);
    }

    /**
     * 加密
     *
     * @param data
     *            待加密数据
     * @param key
     *            密钥
     * @param cipherAlgorithm
     *            加密算法/工作模式/填充方式
     * @return byte[] 加密数据
     * @throws Exception
     */
    public static byte[] encrypt(byte[] data, Key key, String cipherAlgorithm) throws Exception {
        // 实例化
        Cipher cipher = Cipher.getInstance(cipherAlgorithm);
        // 使用密钥初始化，设置为加密模式
        cipher.init(Cipher.ENCRYPT_MODE, key);
        // 执行操作
        return cipher.doFinal(data);
    }

    /**
     * 解密
     *
     * @param data
     *            待解密数据
     * @param key
     *            密钥
     * @return byte[] 解密数据
     * @throws Exception
     */
    public static byte[] decrypt(byte[] data, Key key) throws Exception {
        return decrypt(data, key, DEFAULT_CIPHER_ALGORITHM);
    }

    /**
     * 解密
     *
     * @param data
     *            待解密数据
     * @param key
     *            密钥
     * @param cipherAlgorithm
     *            加密算法/工作模式/填充方式
     * @return byte[] 解密数据
     * @throws Exception
     */
    public static byte[] decrypt(byte[] data, Key key, String cipherAlgorithm) throws Exception {
        // 实例化
        Cipher cipher = Cipher.getInstance(cipherAlgorithm);
        // 使用密钥初始化，设置为解密模式
        cipher.init(Cipher.DECRYPT_MODE, key);
        // 执行操作
        return cipher.doFinal(data);
    }

    public static String showByteArray(byte[] data) {
        if (null == data) {
            return null;
        }
        StringBuilder sb = new StringBuilder("{");
        for (byte b : data) {
            sb.append(b).append(",");
        }
        sb.deleteCharAt(sb.length() - 1);
        sb.append("}");
        return sb.toString();
    }

    /**
     * 将16进制转换为二进制
     *
     * @param hexStr
     * @return
     */
    public static byte[] parseHexStr2Byte(String hexStr) {

        if (hexStr.length() < 1)
            return null;
        byte[] result = new byte[hexStr.length() / 2];
        for (int i = 0; i < hexStr.length() / 2; i++) {
            int high = Integer.parseInt(hexStr.substring(i * 2, i * 2 + 1), 16);
            int low = Integer.parseInt(hexStr.substring(i * 2 + 1, i * 2 + 2), 16);
            result[i] = (byte) (high * 16 + low);
        }
        return result;
    }

    /**
     * 将二进制转换成16进制
     *
     * @param buf
     * @return
     */
    public static String parseByte2HexStr(byte buf[]) {
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < buf.length; i++) {
            String hex = Integer.toHexString(buf[i] & 0xFF);
            if (hex.length() == 1) {
                hex = '0' + hex;
            }
            sb.append(hex.toUpperCase());
        }
        return sb.toString();
    }

    public static void main(String[] args) throws Exception {
        /*byte[] key = initSecretKey();
        System.out.println("key：" + Base64.getEncoder().encode(key));
        System.out.println("key：" + showByteArray(key));*/

        // 指定key
        String kekkk = "cmVmb3JtZXJyZWZvcm1lcg==";
        System.out.println("kekkk:" + showByteArray(Base64.getDecoder().decode(kekkk)));
        Key k = toKey(Base64.getDecoder().decode(kekkk));


        String data = "{\"carCode\":\"川A07E0M\",\"inTime\":\"2021-07-27 13:35:10\",\"passTime\":\"2021-07-27 16:50:34\",\"parkID\":\"88\",\"inOrOut\":\"1\",\"GUID\":\"f025e064c1864af68406c797b0999c70\",\"channelID\":\"2\",\"channelName\":\"1号门停车场(出场通道1)\",\"imagePath\":\"http://192.168.0.101:9988\\\\Capture_Images\\\\20210727\\\\川A07E0M\\\\川A07E0M_20210727165034406.jpg\"}";
        System.out.println("加密前数据: string:" + data);
        System.out.println("加密前数据: byte[]:" + showByteArray(data.getBytes("utf-8")));
        System.out.println();

        byte[] encryptData = encrypt(data.getBytes("utf-8"), k);
        String encryptStr=parseByte2HexStr(encryptData);

        System.out.println("加密后数据: byte[]:" + showByteArray(encryptData));
        System.out.println("加密后数据: Byte2HexStr:" + encryptStr);
        System.out.println();

        byte[] decryptData = decrypt(parseHexStr2Byte(encryptStr), k);
        System.out.println("解密后数据: byte[]:" + showByteArray(decryptData));
        System.out.println("解密后数据: string:" + new String(decryptData,"utf-8"));

    }
}
```



开始上对比的**golang**代码，请注意看我的注释：

```go
package lib

import (
	"bytes"
	"crypto/aes"
	"encoding/base64"
	"encoding/hex"
	"errors"
	"strings"
)

type liFangEncryptionInterface interface {
	LiFangEncrypt() (string, error) // 立方停车加密
	LiFangDecrypt() ([]byte, error) // 立方停车解密
}

type LiFangEncryptionStruct struct {
	Key               string // 立方密钥
	NeedEncryptString string // 需要加密的字符串
	NeedDecryptString string // 需要解密的字符串
}

// NewLiFangEncryption 创建立方加解密对象
func NewLiFangEncryption(lfs *LiFangEncryptionStruct) liFangEncryptionInterface {
	return &LiFangEncryptionStruct{
		Key:               lfs.Key,
		NeedEncryptString: lfs.NeedEncryptString,
		NeedDecryptString: lfs.NeedDecryptString,
	}
}

func (lfs *LiFangEncryptionStruct) LiFangEncrypt() (encodeStr string, err error) {
	decodeKey, err := base64.StdEncoding.DecodeString(lfs.Key) //这一行是说，将Key密钥进行base64编码，这一行与加密 AES/ECB/PKCS5Padding 没有关系
	aseByte, err := aesEncrypt([]byte(lfs.NeedEncryptString), decodeKey)//这一行开始就是 AES/ECB/PKCS5Padding 的标准加密了
	encodeStr = strings.ToUpper(hex.EncodeToString(aseByte)) //把加密后的字符串变为大写
	return
}

func (lfs *LiFangEncryptionStruct) LiFangDecrypt() (lastByte []byte, err error) {
	hexStrByte, err := hex.DecodeString(lfs.NeedDecryptString) //这一行的意思，把需要解密的字符串从16进制字符转为2进制byte数组
	decodeKey, err := base64.StdEncoding.DecodeString(lfs.Key) //这行还是将Key密钥进行base64编码
	lastByte, err = aesDecrypt(hexStrByte, decodeKey) // 这里开始就是 AES/ECB/PKCS5Padding 的标准解密了
	return
}

func aesEncrypt(src, key []byte) ([]byte, error) {
	block, err := aes.NewCipher(key) // 生成加密用的block对象
	if err != nil {
		return nil, err
	}
	bs := block.BlockSize() // 根据传入的密钥，返回block的大小，也就是俗称的数据块位数，如128位，192位，256位
	src = pKCS5Padding(src, bs)// 这里是PKCS5Padding填充方式，继续向下看
	if len(src)%bs != 0 { // 如果加密字符串的byte长度不能整除数据块位数，则表示当前加密的块大小不适用
		return nil, errors.New("Need a multiple of the blocksize")
	}
	out := make([]byte, len(src))
	dst := out
	for len(src) > 0 {
		block.Encrypt(dst, src[:bs]) // 开始用已经产生的key来加密
		src = src[bs:]
		dst = dst[bs:]
	}
	return out, nil
}

func aesDecrypt(src, key []byte) ([]byte, error) {
	block, err := aes.NewCipher(key)
	if err != nil {
		return nil, err
	}

	out := make([]byte, len(src))
	dst := out
	bs := block.BlockSize()
	if len(src)%bs != 0 {
		return nil, errors.New("crypto/cipher: input not full blocks")
	}
	for len(src) > 0 {
		block.Decrypt(dst, src[:bs])
		src = src[bs:]
		dst = dst[bs:]
	}
	out = pKCS5UnPadding(out)
	return out, nil
}

func pKCS5Padding(ciphertext []byte, blockSize int) []byte {
	padding := blockSize - len(ciphertext)%blockSize
	padtext := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(ciphertext, padtext...)
}

func pKCS5UnPadding(origData []byte) []byte {
	length := len(origData)
	unpadding := int(origData[length-1])
	return origData[:(length - unpadding)]
}
```



之后附赠普通带偏移量的AES加解密   AES/CBC/PKCS5Padding 的代码：

```go
// aesEncrypt 加密
func (hs *HuaLiServiceStruct) aesEncrypt() (string, error) {
	key := []byte(hs.DataSecret)
	encodeBytes := []byte(hs.EncodeStr)
	//根据key密钥 生成密文
	block, err := aes.NewCipher(key)
	if err != nil {
		return "", err
	}
	blockSize := block.BlockSize()
	encodeBytes = pKCS5Padding(encodeBytes, blockSize)
	//添加偏移量
	blockMode := cipher.NewCBCEncrypter(block, []byte(hs.DataSecretIV)) // 其实这里就更简单了，只需要创建一个CBC的加密对象，传入偏移量即可
	crypted := make([]byte, len(encodeBytes))
	blockMode.CryptBlocks(crypted, encodeBytes)
	return base64.StdEncoding.EncodeToString(crypted), nil
}

// aesDecrypt 解密
func (hs *HuaLiServiceStruct) aesDecrypt() ([]byte, error) {
	key := []byte(hs.DataSecret)
	//先解密base64
	decodeBytes, err := base64.StdEncoding.DecodeString(hs.DecodeStr)
	if err != nil {
		return nil, err
	}
	block, err := aes.NewCipher(key)
	if err != nil {
		return nil, err
	}
	blockMode := cipher.NewCBCDecrypter(block, []byte(hs.DataSecretIV)) // 其实这里就更简单了，只需要创建一个CBC的解密对象，传入偏移量即可
	origData := make([]byte, len(decodeBytes))

	blockMode.CryptBlocks(origData, decodeBytes)
	origData = pKCS5UnPadding(origData)
	return origData, nil
}

func pKCS5Padding(ciphertext []byte, blockSize int) []byte {
	padding := blockSize - len(ciphertext)%blockSize
	padtext := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(ciphertext, padtext...)
}

func pKCS5UnPadding(origData []byte) []byte {
	length := len(origData)
	unpadding := int(origData[length-1])
	return origData[:(length - unpadding)]
}
```

 

再赠送两种MD5加密写法，也是利用了crypto库：

```go
func TestMd5Create(t *testing.T) {
	CreateMd5Demo1("anyanfei")
	s := CreateMd5Demo2("anyanfei") // ca43e4338149bad344b75378ce5447ea
	fmt.Printf("%x\n", s)
}

func CreateMd5Demo1(s string){
	h := md5.New()
	io.WriteString(h, s)
	fmt.Printf("%x\n", h.Sum(nil)) // ca43e4338149bad344b75378ce5447ea
}

func CreateMd5Demo2(s string) [16]byte {
	data := []byte(s)
	return md5.Sum(data)
}
```



## 总结

其实大部分加密方法都是前人栽树后人乘凉，我们只需要知道最简单的使用即可，至于里面的逻辑，真想了解可以自行看源码解决，本次只写了两个案例，也是工作中用到最多的AES对称加密

AES/ECB/PKCS5Padding  ECB不需要带偏移量的对称加密写法

AES/CBC/PKCS5Padding  带偏移量的对称加密写法

MD5加密写法

## 参考资料

- https://godoc.org/golang.org/x/crypto