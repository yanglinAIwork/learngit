## 一、Blob字段介绍

​	blob类型的字段用于存储二进制数据

### 1、MySql的四种BLOB类型

| 类型       | 大小(单位：字节) |
| ---------- | ---------------- |
| TinyBlob   | 255              |
| Blob       | 65k              |
| MediumBlob | 16M              |
| LongBlob   | 4G               |

---

### 2、Blob字段转String

​	注意：如果Blob是图片、视屏、字体文件等，转完之后会显示为Null

​	格式：SELECT CONVERT(GROUP_CONCAT(字段) USING utf8) FROM 表名 <**此处的bug为只显示前1000个字符**，建议用3的方法转String>

​	例如：

~~~sql
SELECT
	CONVERT (
		GROUP_CONCAT(poster_picture) USING utf8
	) poster_picture
FROM
	t_slup_health_broadcast
~~~

---

### 3、在JAVA中的应用

​	Blob字段可以用String类型的变量接（可以用2的方法转String，也可以用如下的方法）。

~~~java
public static String convertBlobToString(Blob blob) throws SQLException {
		String value = null;
		StringBuffer buffer = new StringBuffer("");
		byte[] bytes = blob.getBytes(1L, (int) blob.length());
		if (bytes != null && bytes.length > 0) {
			for (int i = 0; i < bytes.length; i++) {
				buffer.append(bytes[i]);
			}
			value = buffer.toString();
		}
		return value;
	}
~~~

