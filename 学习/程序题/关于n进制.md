将一个整数转换成n进制
```python
def decimal_to_base(n, base):
    if n == 0:
        return '0'
    digits = []
    while n > 0:
        digits.append(str(n % base))
        n = n // base
    return ''.join(reversed(digits))

# 示例
print(decimal_to_base(10, 4))   # 输出 22（四进制）
print(decimal_to_base(10, 2))   # 输出 1010（二进制）./////
```
将n进制转换成十进制整数,注意那里要是字符串
```
print(int("n进制数字",n))
```