# Задание:

Методом Рунге-Кутта проинтегрировать дифференциальное уравнение:

$$
y'' = -4y + x^2, \quad y(0) = 1, \quad y'(0) = 2
$$

на отрезке \([0; 0,3]\) с шагом \(h = 0,1\). Найти аналитическое решение \(y = y(x)\) заданного уравнения и сравнить значения точного и приближенного решений в точках \(x_1 = 0,1\), \(x_2 = 0,2\), \(x_3 = 0,3\). Все вычисления вести с шестью десятичными знаками.

Сначала нужно получить ответ путём аналитического решения, им будет:

$$
y = \sin(2x) + \frac{9 \cos(2x)}{8} + \frac{x^2}{4} - \frac{1}{8}
$$

Подставим вместо x следующие значения: 0.1; 0.2; 0.3. Получаем: 1.178744; 1.310612; 1.390645 соответственно.
Теперь закодим решение методом Рунге-Кутта:

## Метод Рунге-Кутта

Использует четыре промежуточных вычисления:

$$
k_1 = h \cdot f(x_i, y_i), \quad k_2 = h \cdot f\left(x_i + \frac{h}{2}, y_i + \frac{k_1}{2}\right),
$$

$$
k_3 = h \cdot f\left(x_i + \frac{h}{2}, y_i + \frac{k_2}{2}\right), \quad k_4 = h \cdot f(x_i + h, y_i + k_3),
$$

$$
y_{i+1} = y_i + \frac{k_1 + 2k_2 + 2k_3 + k_4}{6}.
$$

Обеспечивает высокую точность при большом шаге h. Теперь закодим решение методом Рунге-Кутта, но в этот раз ещё вводим z для замены, потому что в этот раз дифференциальное уравнение 2-го порядка, а не 1-го:

```
def runge_kutta_system(f1, f2, y0, z0, x0, xn, h):
    x_values = []
    y_values = []
    z_values = []

    x = x0
    y = y0
    z = z0

    while x <= xn + 1e-9:
        x_values.append(x)
        y_values.append(y)
        z_values.append(z)

        k1_y = h * f1(x, y, z)
        k1_z = h * f2(x, y, z)

        k2_y = h * f1(x + h / 2, y + k1_y / 2, z + k1_z / 2)
        k2_z = h * f2(x + h / 2, y + k1_y / 2, z + k1_z / 2)

        k3_y = h * f1(x + h / 2, y + k2_y / 2, z + k2_z / 2)
        k3_z = h * f2(x + h / 2, y + k2_y / 2, z + k2_z / 2)

        k4_y = h * f1(x + h, y + k3_y, z + k3_z)
        k4_z = h * f2(x + h, y + k3_y, z + k3_z)

        y = y + (k1_y + 2 * k2_y + 2 * k3_y + k4_y) / 6
        z = z + (k1_z + 2 * k2_z + 2 * k3_z + k4_z) / 6

        x = x + h

    return x_values, y_values, z_values

def f1(x, y, z):
    return z

def f2(x, y, z):
    return -4 * y + x**2

x0 = 0
y0 = 1
z0 = 2
xn = 0.3
h = 0.1

x_values, y_values, z_values = runge_kutta_system(f1, f2, y0, z0, x0, xn, h)

print("i | x_i   | y_i     | z_i")
print("---|-------|---------|---------")
for i, (x, y, z) in enumerate(zip(x_values, y_values, z_values)):
    print(f"{i} | {x:.1f} | {y:.6f} | {z:.6f}")
```

Получаем:

![image](https://github.com/user-attachments/assets/d6a17700-6070-439d-b8c0-bded3a1eaac7)

i  | x_i   | y_i     | z_i
---|-------|---------|---------
0  | 0.0   | 1.000000| 2.000000
1  | 0.1   | 1.178742| 1.563133
2  | 0.2   | 1.310608| 1.065943
3  | 0.3   | 1.390641| 0.530246

Рассчитаем относительные и абсолютные погрешности, как они считаются - очевидно:
```
data = [
    {"i": 1, "x_i": 0.1, "y_i": 1.178742, "y_exact": 1.178744},
    {"i": 2, "x_i": 0.2, "y_i": 1.310608, "y_exact": 1.310612},
    {"i": 3, "x_i": 0.3, "y_i": 1.390641, "y_exact": 1.390645},
]

def calculate_errors(data):
    result = []
    for row in data:
        y_i = row["y_i"]
        y_exact = row["y_exact"]
        if y_exact is not None:
            abs_error = abs(y_exact - y_i)
            rel_error = abs_error / abs(y_exact) if y_exact != 0 else None
        else:
            abs_error = None
            rel_error = None
        result.append({
            "i": row["i"],
            "x_i": row["x_i"],
            "y_i": y_i,
            "y_exact": y_exact,
            "abs_error": abs_error,
            "rel_error": rel_error,
        })
    return result

results = calculate_errors(data)

print(f"{'i':<2} | {'x_i':<5} | {'y_i':<9} | {'y_exact':<9} | {'abs_error':<10} | {'rel_error':<10}")
print("-" * 60)
for row in results:
    abs_error = f"{row['abs_error']:.6f}" if row['abs_error'] is not None else "N/A"
    rel_error = f"{row['rel_error']:.6f}" if row['rel_error'] is not None else "N/A"
    y_exact = f"{row['y_exact']:.6f}" if row['y_exact'] is not None else "N/A"
    print(
        f"{row['i']:<2} | {row['x_i']:<5.1f} | {row['y_i']:<9.6f} | "
        f"{y_exact:<9} | {abs_error:<10} | {rel_error:<10}"
    )
```
Получилось:

![image](https://github.com/user-attachments/assets/3e73040d-cf98-4c3b-b48c-7916f10c440a)

i  | x_i   | y_i       | y_exact   | abs_error  | rel_error
---|-------|-----------|-----------|------------|----------------
1  | 0.1   | 1.178742  | 1.178744  | 0.000002   | 0.000002  
2  | 0.2   | 1.310608  | 1.310612  | 0.000004   | 0.000003  
3  | 0.3   | 1.390641  | 1.390645  | 0.000004   | 0.000003 

Где y_i - значение, которое было ранее получено при выполнении программы, y_exact - значение, которое было получено аналитически, abs_error - абсолютная погрешность, rel_error - относительная погрешность.

P.S.: Полный код:

```
def runge_kutta_system(f1, f2, y0, z0, x0, xn, h):
    x_values = []
    y_values = []
    z_values = []

    x = x0
    y = y0
    z = z0

    while x <= xn + 1e-9:
        x_values.append(x)
        y_values.append(y)
        z_values.append(z)

        k1_y = h * f1(x, y, z)
        k1_z = h * f2(x, y, z)

        k2_y = h * f1(x + h / 2, y + k1_y / 2, z + k1_z / 2)
        k2_z = h * f2(x + h / 2, y + k1_y / 2, z + k1_z / 2)

        k3_y = h * f1(x + h / 2, y + k2_y / 2, z + k2_z / 2)
        k3_z = h * f2(x + h / 2, y + k2_y / 2, z + k2_z / 2)

        k4_y = h * f1(x + h, y + k3_y, z + k3_z)
        k4_z = h * f2(x + h, y + k3_y, z + k3_z)

        y = y + (k1_y + 2 * k2_y + 2 * k3_y + k4_y) / 6
        z = z + (k1_z + 2 * k2_z + 2 * k3_z + k4_z) / 6

        x = x + h

    return x_values, y_values, z_values

def f1(x, y, z):
    return z

def f2(x, y, z):
    return -4 * y + x**2

x0 = 0
y0 = 1
z0 = 2
xn = 0.3
h = 0.1

x_values, y_values, z_values = runge_kutta_system(f1, f2, y0, z0, x0, xn, h)

print("i | x_i   | y_i     | z_i")
print("---|-------|---------|---------")
for i, (x, y, z) in enumerate(zip(x_values, y_values, z_values)):
    print(f"{i} | {x:.1f} | {y:.6f} | {z:.6f}")

data = [
    {"i": 1, "x_i": 0.1, "y_i": 1.178742, "y_exact": 1.178744},
    {"i": 2, "x_i": 0.2, "y_i": 1.310608, "y_exact": 1.310612},
    {"i": 3, "x_i": 0.3, "y_i": 1.390641, "y_exact": 1.390645},
]

def calculate_errors(data):
    result = []
    for row in data:
        y_i = row["y_i"]
        y_exact = row["y_exact"]
        if y_exact is not None:
            abs_error = abs(y_exact - y_i)
            rel_error = abs_error / abs(y_exact) if y_exact != 0 else None
        else:
            abs_error = None
            rel_error = None
        result.append({
            "i": row["i"],
            "x_i": row["x_i"],
            "y_i": y_i,
            "y_exact": y_exact,
            "abs_error": abs_error,
            "rel_error": rel_error,
        })
    return result

results = calculate_errors(data)

print(f"{'i':<2} | {'x_i':<5} | {'y_i':<9} | {'y_exact':<9} | {'abs_error':<10} | {'rel_error':<10}")
print("-" * 60)
for row in results:
    abs_error = f"{row['abs_error']:.6f}" if row['abs_error'] is not None else "N/A"
    rel_error = f"{row['rel_error']:.6f}" if row['rel_error'] is not None else "N/A"
    y_exact = f"{row['y_exact']:.6f}" if row['y_exact'] is not None else "N/A"
    print(
        f"{row['i']:<2} | {row['x_i']:<5.1f} | {row['y_i']:<9.6f} | "
        f"{y_exact:<9} | {abs_error:<10} | {rel_error:<10}"
    )
```


