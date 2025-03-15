---
title: 'picoCTF 2022 | Fresh Java WriteUp'
author: 'appdone'
categories: [picoCTF 2022 Challenges]
tags: [reverse engineering]
render_with_liquid: false
---

Bu yazıda, picoCTF platformunda yer alan "Fresh Java" isimli meydan okumayı çözeceğiz. Bu sefer elimizde java ile yazılmış bir program var. Bu programı JD-GUI ile decompile ettiğimizde aşağıdaki kodlar ile karşılaşıyoruz.

```java
import java.util.Scanner;

public class KeygenMe {
  public static void main(String[] paramArrayOfString) {
    Scanner scanner = new Scanner(System.in);
    System.out.println("Enter key:");
    String str = scanner.nextLine();
    if (str.length() != 34) {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(33) != '}') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(32) != 'e') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(31) != 'b') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(30) != '6') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(29) != 'a') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(28) != '2') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(27) != '3') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(26) != '3') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(25) != '9') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(24) != '_') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(23) != 'd') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(22) != '3') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(21) != 'r') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(20) != '1') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(19) != 'u') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(18) != 'q') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(17) != '3') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(16) != 'r') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(15) != '_') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(14) != 'g') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(13) != 'n') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(12) != '1') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(11) != 'l') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(10) != '0') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(9) != '0') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(8) != '7') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(7) != '{') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(6) != 'F') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(5) != 'T') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(4) != 'C') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(3) != 'o') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(2) != 'c') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(1) != 'i') {
      System.out.println("Invalid key");
      return;
    } 
    if (str.charAt(0) != 'p') {
      System.out.println("Invalid key");
      return;
    } 
    System.out.println("Valid key");
  }
}
```

Gördüğünüz üzere bir anahtar istiyor ve girilen anahtarın hangi karaktere denk geldiğini kontrol ediyor. Sonrada ekrana "Invalid Key" yazısını yazdırıyor. Sondan başlayarak kontrol edilen karakterleri birleştirdiğimizde bayrağı elde ediyoruz.
