# Задание:

Численно решить дифференциальное уравнение:

$$
y' = 2\frac{y}{x} - x, \quad y(1) = 0
$$

на отрезке \([1; 2]\) с шагом \(h = 0.2\) методом Эйлера, модифицированным методом Эйлера и методом Рунге-Кутта.

Найти точное решение \(y = y(x)\) и сравнить значения точного и приближенных решений в точке \(x = 2\).

Найти абсолютную и относительную погрешности в этой точке для каждого метода. Вычисления вести с четырьмя десятичными знаками.

# Решение:

Для начала я решил уравнение аналитически, чтобы в дальнейшем рассчитывать погрешности, для х = 2 ответ получился -2.7726

## Метод Эйлера

Использует линейное приближение функции:

$$
y_{i+1} = y_i + h \cdot f(x_i, y_i),
$$

где h — шаг интегрирования.

```
import pandas as pd

x0 = 1
y0 = 0
h = 0.2
n = 5 

def f(x, y):
    return 2 * (y / x) - x

results = {
    "i": [],
    "x_i": [],
    "y_i": [],
    "f(x_i, y_i)": [],
    "Δy_i": []
}

x, y = x0, y0
for i in range(n + 1):
    results["i"].append(i)
    results["x_i"].append(round(x, 4))
    results["y_i"].append(round(y, 4))
    f_value = f(x, y)
    results["f(x_i, y_i)"].append(round(f_value, 4))
    delta_y = h * f_value
    results["Δy_i"].append(round(delta_y, 4))

    y += delta_y
    x += h

df = pd.DataFrame(results)
print(df)
```

![image](https://github.com/user-attachments/assets/20fce7ce-af72-4938-a494-c0ad9599f6f2)

Абсолютная погрешность:

$$
|\text{Точное значение} - \text{Приближенное значение}| = |-2.1741 - (-2.7726)| = 0.5985
$$

Относительная погрешность:

$$
\left| \frac{\text{Точное значение} - \text{Приближенное значение}}{\text{Точное значение}} \right| \cdot 100\% = 
\left| \frac{-2.1741 - (-2.7726)}{-2.7726} \right| \cdot 100\% \approx 21.5862367452\% \approx 21.5862\%
$$

## Модифицированный метод Эйлера
Улучшает точность за счёт использования средней точки:

$$
y_{i+1} = y_i + h \cdot \frac{f(x_i, y_i) + f(x_i + h, y_i + h \cdot f(x_i, y_i))}{2}.
$$

Учитывает дополнительную информацию о наклоне на конце интервала, что повышает точность.

```
import pandas as pd

x0 = 1.0
y0 = 0.0
h = 0.2
n = 5  # кол-во шагов


# Функция f(x, y)
def f(x, y):
    return (2 * y / x) - x

x_values = [x0]
y_values = [y0]
f_values = [f(x0, y0)]
delta_y_values = [0]

# Метод модифицированного Эйлера
for i in range(n):
    x = x_values[-1]
    y = y_values[-1]

    # Шаг 1: вычисляем промежуточные значения
    x_half = x + h / 2
    y_half = y + (h / 2) * f(x, y)
    f_half = f(x_half, y_half)

    # Шаг 2: вычисляем приращение Δy и новое значение y
    delta_y = h * f_half
    y_new = y + delta_y

    x_values.append(round(x + h, 4))
    y_values.append(round(y_new, 4))
    f_values.append(round(f_half, 4))
    delta_y_values.append(round(delta_y, 4))

data = {
    'i': list(range(n + 1)),
    'x_i': [round(x, 4) for x in x_values],
    'f_i = f(x_i, y_i)': [round(f, 4) for f in f_values],
    'Δy_i': delta_y_values,
    'y_i': y_values,
}

table = pd.DataFrame(data)
print(table)
```

![image](https://github.com/user-attachments/assets/ca28be2d-d77d-4dae-bf9d-016b0020a592)

Абсолютная погрешность:

$$
|\text{Точное значение} - \text{Приближенное значение}| = |-2.7243 - (-2.7726)| = 0.0483
$$

Относительная погрешность:

$$
\left| \frac{\text{Точное значение} - \text{Приближенное значение}}{\text{Точное значение}} \right| \cdot 100\% 
= \left| \frac{-2.7243 - (-2.7726)}{-2.7726} \right| \cdot 100\% 
\approx 1.7420\%
$$



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

Обеспечивает высокую точность при большом шаге h

```
def runge_kutta(f, y0, x0, xn, h):
    x_values = []
    y_values = []

    x = x0
    y = y0

    while x <= xn:
        x_values.append(x)
        y_values.append(y)

        k1 = h * f(x, y)
        k2 = h * f(x + h / 2, y + k1 / 2)
        k3 = h * f(x + h / 2, y + k2 / 2)
        k4 = h * f(x + h, y + k3)

        y = y + (k1 + 2 * k2 + 2 * k3 + k4) / 6
        x = x + h

    return x_values, y_values

# Определение функции f(x, y)
def f(x, y):
    return (2 * y / x) - x

x0 = 1
y0 = 0
xn = 2
h = 0.2

# Решение уравнения
x_values, y_values = runge_kutta(f, y0, x0, xn, h)

print("i | x_i | y_i")
print("---|-----|-----")
for i, (x, y) in enumerate(zip(x_values, y_values)):
    print(f"{i} | {x:.1f} | {y:.4f}")
```

![image](https://github.com/user-attachments/assets/57ae19e6-fb2b-4a63-88b6-105d8a22f9dd)

Абсолютная погрешность:

$$
|\text{Точное значение} - \text{Приближенное значение}| = |-2.7722 - (-2.7726)| = 0.0004
$$

Относительная погрешность:

$$
\left| \frac{\text{Точное значение} - \text{Приближенное значение}}{\text{Точное значение}} \right| \cdot 100\%
= \left| \frac{-2.7722 - (-2.7726)}{-2.7726} \right| \cdot 100\%
\approx 0.0144\%
$$

```
import matplotlib.pyplot as plt

def runge_kutta(f, y0, x0, xn, h):
    x_values = []
    y_values = []

    x = x0
    y = y0

    while x <= xn:
        x_values.append(x)
        y_values.append(y)

        k1 = h * f(x, y)
        k2 = h * f(x + h / 2, y + k1 / 2)
        k3 = h * f(x + h / 2, y + k2 / 2)
        k4 = h * f(x + h, y + k3)

        y = y + (k1 + 2 * k2 + 2 * k3 + k4) / 6
        x = x + h

    return x_values, y_values

# Определение функции f(x, y)
def f(x, y):
    return (2 * y / x) - x

# Начальные условия
x0 = 1
y0 = 0
xn = 2
h = 0.2

# Решение уравнения
x_values, y_values = runge_kutta(f, y0, x0, xn, h)

plt.figure(figsize=(10, 6))
plt.plot(x_values, y_values, marker='o', label="Численное решение (Метод Рунге-Кутта 4-го порядка)")
plt.title("Решение дифференциального уравнения методом Рунге-Кутта")
plt.xlabel("x")
plt.ylabel("y")
plt.grid(True)
plt.legend()
plt.show()
```

Сам график:

![image](https://github.com/user-attachments/assets/7973ffd1-2025-4fab-9400-1a6aeda9f8fb)

