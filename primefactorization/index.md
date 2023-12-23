# 分解质因数


> **算术基本定理**：任何一个大于1的自然数 N，如果N不为质数，那么N可以唯一分解成有限个质数的乘积

```java
List<Integer> primeFactorization(int n) {
    // 分解得到的质数列表（不去重）
    List<Integer> res = new ArrayList<Integer>();
    
    /* 短除法：从最小的质数开始分解 */
    for (int i = 2; i * i <= n; i++) {
        // 判断n（即模余数）是否为质数只需要遍历到sqrt(n)
        while (n % i == 0) {
            res.add(i);
            n /= i;
        }
    }
    return res;
}
```


