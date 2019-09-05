---
layout: post
title: "java AES对称加密工具 适用于windows与linux"
date: 2017-07-18
tags: java
---

之前在网上搜索的AESUtil都使用了SecureRandom获取随机数，部署到linux上就会出现解密错误的问题

这里提供一个windows和linux都能使用的AESUtil

简书格式不好看。CSDN地址 https://blog.csdn.net/YZX2018/article/details/86552683

引入Base64的依赖包

<dependency>
   <groupId>commons-codec</groupId>
   <artifactId>commons-codec</artifactId>
   <version>1.11</version>
</dependency>

复制下面的代码就能使用了

import org.apache.commons.codec.binary.Base64;
import javax.crypto.*;
import javax.crypto.spec.SecretKeySpec;
import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;

/**
 * @version V1.0
 * @desc AES 加密工具类
 */
public class AESUtil {

    private static final String KEY_ALGORITHM = "AES";
    private static final String DEFAULT_CIPHER_ALGORITHM = "AES/ECB/PKCS5Padding";// 默认的加密算法

    /**
     * AES 加密操作
     *
     * @param content 待加密内容
     * @param key     密钥
     * @return 返回Base64转码后的加密数据
     * @throws NoSuchPaddingException 
     * @throws NoSuchAlgorithmException 
     * @throws UnsupportedEncodingException 
     * @throws InvalidKeyException 
     * @throws BadPaddingException 
     * @throws IllegalBlockSizeException 
     */
    public static String encrypt(String content, String key, String charset) throws NoSuchAlgorithmException, NoSuchPaddingException, UnsupportedEncodingException, InvalidKeyException, IllegalBlockSizeException, BadPaddingException {
      
            Cipher cipher = Cipher.getInstance(DEFAULT_CIPHER_ALGORITHM);// 创建密码器
    
            byte[] byteContent = content.getBytes(charset);
    
            cipher.init(Cipher.ENCRYPT_MODE, getSecretKey(key));// 初始化为加密模式的密码器
    
            byte[] result = cipher.doFinal(byteContent);// 加密
    
            return Base64.encodeBase64String(result);// 通过Base64转码返回
        
   
}
   
    /**
     * AES 解密操作
       *
     * @param content 待解密内容
     * @param key 密钥
     * @param charset
     * @return
     * @throws NoSuchPaddingException 
     * @throws NoSuchAlgorithmException 
     * @throws InvalidKeyException 
     * @throws BadPaddingException 
     * @throws IllegalBlockSizeException 
     * @throws UnsupportedEncodingException 
       */
    public static String decrypt(String content, String key, String charset) throws NoSuchAlgorithmException, NoSuchPaddingException, InvalidKeyException, IllegalBlockSizeException, BadPaddingException, UnsupportedEncodingException \{
            // 实例化
            Cipher cipher = Cipher.getInstance(DEFAULT_CIPHER_ALGORITHM);
            // 使用密钥初始化，设置为解密模式
            cipher.init(Cipher.DECRYPT_MODE, getSecretKey(key));
            // 执行操作
            byte[] result = cipher.doFinal(Base64.decodeBase64(content));
            return new String(result, charset);
   
 }
   
    /**
     * 生成加密秘钥
       *
     * @return
     * @throws NoSuchAlgorithmException 
       */
    private static SecretKeySpec getSecretKey(final String key) throws NoSuchAlgorithmException\{
        // 返回生成指定算法密钥生成器的 KeyGenerator 对象
     KeyGenerator kg  = KeyGenerator.getInstance(KEY_ALGORITHM);
   
            // AES 要求密钥长度为 128
            SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");
         secureRandom.setSeed(key.getBytes());
       
         kg.init(128, secureRandom);
       
            // 生成一个密钥
         SecretKey secretKey = kg.generateKey();
       
            return new SecretKeySpec(secretKey.getEncoded(), KEY_ALGORITHM);// 转换为AES专用密钥
   
 }
 }