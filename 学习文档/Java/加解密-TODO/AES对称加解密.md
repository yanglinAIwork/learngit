~~~java
package com.slup.customer.util;

import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.KeyGenerator;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;

public class Demo {
    /**
     *
     *
     * @param content
     *
     * @param password
     *
     * @return
     */

    public static byte[] encrypt(String content, String password) {
        try {
            KeyGenerator kgen = KeyGenerator.getInstance("AES");
            kgen.init(128, new SecureRandom(password.getBytes()));
            SecretKey secretKey = kgen.generateKey();
            byte[] enCodeFormat = secretKey.getEncoded();
            SecretKeySpec key = new SecretKeySpec(enCodeFormat, "AES");
            Cipher cipher = Cipher.getInstance("AES");
            byte[] byteContent = content.getBytes("utf-8");
            cipher.init(Cipher.ENCRYPT_MODE, key);
            byte[] result = cipher.doFinal(byteContent);
            return result;
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
        } catch (InvalidKeyException e) {
            e.printStackTrace();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        } catch (IllegalBlockSizeException e) {
            e.printStackTrace();
        } catch (BadPaddingException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     *
     *
     * @param content
     *
     * @param password
     *
     * @return
     */
    public static byte[] decrypt(byte[] content, String password) {
        try {
            KeyGenerator kgen = KeyGenerator.getInstance("AES");
            kgen.init(128, new SecureRandom(password.getBytes()));
            SecretKey secretKey = kgen.generateKey();
            byte[] enCodeFormat = secretKey.getEncoded();
            SecretKeySpec key = new SecretKeySpec(enCodeFormat, "AES");
            Cipher cipher = Cipher.getInstance("AES");
            cipher.init(Cipher.DECRYPT_MODE, key);
            byte[] result = cipher.doFinal(content);
            return result;
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
        } catch (InvalidKeyException e) {
            e.printStackTrace();
        } catch (IllegalBlockSizeException e) {
            e.printStackTrace();
        } catch (BadPaddingException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     *
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

    /**
     *
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
     *
     *
     * @param content
     *
     * @param password
     *
     * @return
     */
    public static byte[] encrypt2(String content, String password) {
        try {
            SecretKeySpec key = new SecretKeySpec(password.getBytes(), "AES");
            Cipher cipher = Cipher.getInstance("AES/ECB/NoPadding");
            byte[] byteContent = content.getBytes("utf-8");
            cipher.init(Cipher.ENCRYPT_MODE, key);
            byte[] result = cipher.doFinal(byteContent);
            return result;
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
        } catch (InvalidKeyException e) {
            e.printStackTrace();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        } catch (IllegalBlockSizeException e) {
            e.printStackTrace();
        } catch (BadPaddingException e) {
            e.printStackTrace();
        }
        return null;
    }

    public static void main(String[] args) throws Exception {
        String content = "{  \"head\": {   \"logid\" : \"5c7a017a-feca-44f9-b437-02c83c0c7389\",   \"wsid\":\"SLIS_NB_CN_001\",   \"sendtime\": \"20190507124950932\",    \"resultcode\": \"1\", \"resultcontent\": \"这个是啥啊\"  }}";
        String password = "SLIS_NB_CN_001";

        System.out.println("aaa" + content);
        byte[] encode = encrypt(content, password);


        String code = parseByte2HexStr(encode);
        System.out.println(code);

        String  dd  ="5EE499B777C9932AACD5D0EF0CB36B48DB78C24E63D282D797E7FBEB236C374105671D3AC09AA6578DB3CEA12ECDCBFB422272BD6655F54B6DC6CDE4AF4D76BD777AA010D22EF6993C079EEC1C3C220F429C19F06D0FC36F3AF2A9AAD209A022930738DB61921133959A02EB4EC192CDCAAEF4498D1FF17D4272987C9CAC6C19F82312D5A4E751318601CD64B3B40DD7F5A7E1FC8C819427D2D91D4A5A33B30F2B585E291091C8296BC6C988594D420AC4DC4E2B8A4B833E9BDE44E3CFD1C894";
        byte[] decode = parseHexStr2Byte(dd);
        byte[] decryptResult = decrypt(decode, password);
        System.out.println("cc" + new String(decryptResult, "UTF-8"));

    }
}
~~~



~~~java
/**
 * AESHelper.java
 * cn.com.songjy.test
 *
 * Function： TODO 
 *
 *   ver     date      		author
 * ──────────────────────────────────
 *   		 2012-6-29 		songjianyong
 *
 * Copyright (c) 2012, TNT All Rights Reserved.
*/

package cn.com.songjy.test;

import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.KeyGenerator;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;

/**
 * ClassName:AESHelper
 *
 * @author   songjianyong
 * @version  1.0
 * @since    v1.0
 * @Date	 2012-6-29 下午2:06:07 
 */
public class AESHelper {

	/**
	 * @method main
	 * @param args
	 * @throws 
	 * @since v1.0
	 */

	public static void main(String[] args) {

		String content = "宋建勇";
		String password = "12345678";
		byte[] encryptResult = encrypt(content, password);//加密
		byte[] decryptResult = decrypt(encryptResult,password);//解密
		System.out.println("解密后：" + new String(decryptResult));
		
		/*容易出错的地方，请看下面代码*/
		System.out.println("***********************************************");
		try {
			String encryptResultStr = new String(encryptResult,"utf-8");
			decryptResult = decrypt(encryptResultStr.getBytes("utf-8"),password);
			System.out.println("解密后：" + new String(decryptResult));
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		} catch (Exception e) {
			// TODO: handle exception
		}
		/*则，系统会报出如下异常：javax.crypto.IllegalBlockSizeException: Input length must be multiple of 16 when decrypting with padded cipher
	at com.sun.crypto.provider.SunJCE_f.b(DashoA13*..)
	at com.sun.crypto.provider.SunJCE_f.b(DashoA13*..)
	at com.sun.crypto.provider.AESCipher.engineDoFinal(DashoA13*..)
	at javax.crypto.Cipher.doFinal(DashoA13*..)
	at cn.com.songjy.test.ASCHelper.decrypt(ASCHelper.java:131)
	at cn.com.songjy.test.ASCHelper.main(ASCHelper.java:58)
		 * */
		/*这主要是因为加密后的byte数组是不能强制转换成字符串的, 换言之,字符串和byte数组在这种情况下不是互逆的,
		 * 要避免这种情况，我们需要做一些修订，可以考虑将二进制数据转换成十六进制表示,
		 * 主要有两个方法:将二进制转换成16进制(见方法parseByte2HexStr)或是将16进制转换为二进制(见方法parseHexStr2Byte)*/
		
		/*然后，我们再修订以上测试代码*/
		System.out.println("***********************************************");
		String encryptResultStr = parseByte2HexStr(encryptResult);
		System.out.println("加密后：" + encryptResultStr);
		byte[] decryptFrom = parseHexStr2Byte(encryptResultStr);
		decryptResult = decrypt(decryptFrom,password);//解码
		System.out.println("解密后：" + new String(decryptResult));
	}

	/**
	 * 加密
	 * @method encrypt
	 * @param content	需要加密的内容
	 * @param password	加密密码
	 * @return
	 * @throws 
	 * @since v1.0
	 */
	public static byte[] encrypt(String content, String password){
		try {
			KeyGenerator kgen = KeyGenerator.getInstance("AES");
			kgen.init(128, new SecureRandom(password.getBytes()));
			SecretKey secretKey = kgen.generateKey();
			byte[] enCodeFormat = secretKey.getEncoded();
			SecretKeySpec key = new SecretKeySpec(enCodeFormat, "AES");
			Cipher cipher = Cipher.getInstance("AES");// 创建密码器
			byte[] byteContent = content.getBytes("utf-8");
			cipher.init(Cipher.ENCRYPT_MODE, key);// 初始化
			byte[] result = cipher.doFinal(byteContent);
			return result; // 加密
		} catch (NoSuchAlgorithmException e) {
			e.printStackTrace();
		}catch (NoSuchPaddingException e) {
			e.printStackTrace();
		}catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		}catch (InvalidKeyException e) {
			e.printStackTrace();
		}catch (IllegalBlockSizeException e) {
			e.printStackTrace();
		}catch (BadPaddingException e) {
			e.printStackTrace();
		}
		return null;
	}
	
	/**
	 * 解密
	 * @method decrypt
	 * @param content	待解密内容
	 * @param password	解密密钥
	 * @return
	 * @throws 
	 * @since v1.0
	 */
	public static byte[] decrypt(byte[] content, String password){
		try {
			KeyGenerator kgen = KeyGenerator.getInstance("AES");
			kgen.init(128, new SecureRandom(password.getBytes()));
			SecretKey secretKey = kgen.generateKey();
			byte[] enCodeFormat = secretKey.getEncoded();
			SecretKeySpec key = new SecretKeySpec(enCodeFormat, "AES");
			Cipher cipher = Cipher.getInstance("AES");// 创建密码器
			cipher.init(Cipher.DECRYPT_MODE, key);// 初始化
			byte[] result = cipher.doFinal(content);
			return result; // 解密
		} catch (NoSuchAlgorithmException e) {
			e.printStackTrace();
		}catch (NoSuchPaddingException e) {
			e.printStackTrace();
		}catch (InvalidKeyException e) {
			e.printStackTrace();
		}catch (IllegalBlockSizeException e) {
			e.printStackTrace();
		}catch (BadPaddingException e) {
			e.printStackTrace();
		}
		
		return null;
	}
	
	/**
	 * 将二进制转换成16进制
	 * @method parseByte2HexStr
	 * @param buf
	 * @return
	 * @throws 
	 * @since v1.0
	 */
	public static String parseByte2HexStr(byte buf[]){
		StringBuffer sb = new StringBuffer();
		for(int i = 0; i < buf.length; i++){
			String hex = Integer.toHexString(buf[i] & 0xFF);
			if (hex.length() == 1) {
				hex = '0' + hex;
			}
			sb.append(hex.toUpperCase());
		}
		return sb.toString();
	}
	
	/**
	 * 将16进制转换为二进制
	 * @method parseHexStr2Byte
	 * @param hexStr
	 * @return
	 * @throws 
	 * @since v1.0
	 */
	public static byte[] parseHexStr2Byte(String hexStr){
		if(hexStr.length() < 1)
			return null;
		byte[] result = new byte[hexStr.length()/2];
		for (int i = 0;i< hexStr.length()/2; i++) {
			int high = Integer.parseInt(hexStr.substring(i*2, i*2+1), 16);
			int low = Integer.parseInt(hexStr.substring(i*2+1, i*2+2), 16);
			result[i] = (byte) (high * 16 + low);
		}
		return result;
	}
	
	/**
	 * 另外一种加密方式--这种加密方式有两种限制
	 * 1、密钥必须是16位的
	 * 2、待加密内容的长度必须是16的倍数，如果不是16的倍数，就会出如下异常
	 * javax.crypto.IllegalBlockSizeException: Input length not multiple of 16 bytes
        at com.sun.crypto.provider.SunJCE_f.a(DashoA13*..)
        at com.sun.crypto.provider.SunJCE_f.b(DashoA13*..)
        at com.sun.crypto.provider.SunJCE_f.b(DashoA13*..)
        at com.sun.crypto.provider.AESCipher.engineDoFinal(DashoA13*..)
        at javax.crypto.Cipher.doFinal(DashoA13*..)
		要解决如上异常，可以通过补全传入加密内容等方式进行避免。
	 * @method encrypt2
	 * @param content	需要加密的内容
	 * @param password	加密密码
	 * @return
	 * @throws 
	 * @since v1.0
	 */
	public static byte[] encrypt2(String content, String password){
		try {
			SecretKeySpec key = new SecretKeySpec(password.getBytes(), "AES");
			Cipher cipher = Cipher.getInstance("AES/ECB/NoPadding");
			byte[] byteContent = content.getBytes("utf-8");
			cipher.init(Cipher.ENCRYPT_MODE, key);// 初始化
			byte[] result = cipher.doFinal(byteContent);
			return result; // 加密
		} catch (NoSuchAlgorithmException e) {
			e.printStackTrace();
		} catch (NoSuchPaddingException e) {
			e.printStackTrace();
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		} catch (InvalidKeyException e) {
			e.printStackTrace();
		} catch (IllegalBlockSizeException e) {
			e.printStackTrace();
		} catch (BadPaddingException e) {
			e.printStackTrace();
		}
		return null;
	}
}
~~~

