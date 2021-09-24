---
title: Android百分比透明度与十六进制值的转换
date: 2021-09-24 17:26:33
categories: Android
tags: View
---

很多时候，UI设计师给我们的设计稿上面，对于有透明度变化的UI一般都以百分比的形式告诉我们（比如下面的图片中遮罩背景不透明度为70%），但是这个百分比对应的十六进制alpha值到底是多少呢？下面我们通过代码来实现百分比到十六进制值的转换。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191010114411854.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)
```java
private void printAlpha() {
    for (double i = 1; i >= 0; i -= 0.01) {
        //透明度转成alpha值
        int alpha = (int) Math.round(i * 255);

        //alpha值转成16进制
        String hex = Integer.toHexString(alpha).toUpperCase();
        if (hex.length() == 1) {
            hex = "0" + hex;
        }
        int percent = (int) (i * 100);
        System.out.println(String.format("%d%%  | %s", percent, hex));
    }
}
```
写个单元测试验证一下打印结果
```java
public class ExampleUnitTest {

    @Test
    public void test() {
         printAlpha();
    }
}
```
打印结果如下：

| 百分比  | 十六进制alpha值 |
| ---- | ---------- |
| 100% | FF         |
| 99%  | FC         |
| 98%  | FA         |
| 97%  | F7         |
| 96%  | F5         |
| 95%  | F2         |
| 94%  | F0         |
| 93%  | ED         |
| 92%  | EB         |
| 91%  | E8         |
| 90%  | E6         |
| 89%  | E3         |
| 88%  | E0         |
| 87%  | DE         |
| 86%  | DB         |
| 85%  | D9         |
| 84%  | D6         |
| 83%  | D4         |
| 82%  | D1         |
| 81%  | CF         |
| 80%  | CC         |
| 79%  | C9         |
| 78%  | C7         |
| 77%  | C4         |
| 76%  | C2         |
| 75%  | BF         |
| 74%  | BD         |
| 73%  | BA         |
| 72%  | B8         |
| 71%  | B5         |
| 70%  | B3         |
| 69%  | B0         |
| 68%  | AD         |
| 67%  | AB         |
| 66%  | A8         |
| 65%  | A6         |
| 64%  | A3         |
| 63%  | A1         |
| 62%  | 9E         |
| 61%  | 9C         |
| 60%  | 99         |
| 59%  | 96         |
| 58%  | 94         |
| 57%  | 91         |
| 56%  | 8F         |
| 55%  | 8C         |
| 54%  | 8A         |
| 53%  | 87         |
| 52%  | 85         |
| 51%  | 82         |
| 50%  | 80         |
| 49%  | 7D         |
| 48%  | 7A         |
| 47%  | 78         |
| 46%  | 75         |
| 45%  | 73         |
| 44%  | 70         |
| 43%  | 6E         |
| 42%  | 6B         |
| 41%  | 69         |
| 40%  | 66         |
| 39%  | 63         |
| 38%  | 61         |
| 37%  | 5E         |
| 36%  | 5C         |
| 35%  | 59         |
| 34%  | 57         |
| 33%  | 54         |
| 32%  | 52         |
| 31%  | 4F         |
| 30%  | 4D         |
| 29%  | 4A         |
| 28%  | 47         |
| 27%  | 45         |
| 26%  | 42         |
| 25%  | 40         |
| 24%  | 3D         |
| 23%  | 3B         |
| 22%  | 38         |
| 21%  | 36         |
| 20%  | 33         |
| 19%  | 30         |
| 18%  | 2E         |
| 17%  | 2B         |
| 16%  | 29         |
| 15%  | 26         |
| 14%  | 24         |
| 13%  | 21         |
| 12%  | 1F         |
| 11%  | 1C         |
| 10%  | 1A         |
| 9%   | 17         |
| 8%   | 14         |
| 7%   | 12         |
| 6%   | 0F         |
| 5%   | 0D         |
| 4%   | 0A         |
| 3%   | 08         |
| 2%   | 05         |
| 1%   | 03         |
| 0%   | 00         |

所以我们可以知道黑色的70%透明度在Android中可以用`#B3000000`表示（#AARRGGBB，A表示透明度、R表示红色通道、G表示绿色通道、B表示蓝色通道）