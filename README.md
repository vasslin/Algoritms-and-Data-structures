# Вопросы к экзамену

## 1 Вопрос. 
### Хеширование. Полиномиальная хеш-функция. Поиск подстроки в строке при помощи хешей (алгоритм Рабина-Карпа). Оценка времени выполнения.
**Хэширование** - процесс превращения объекта (данных) в хэш - последовательность символов определенной длины, полученная с помощью определенных преобразований.

**Полиномиальная хэш-функция** - хэш-функция, вычисляющая хэш как перевод из p-ной системы счисления в 10-ую:
hash(s)
```rb
p = 131 # простое число, большее мощности алфавита, взаимно простое с mod (опционально вообще простым)
mod = 10**9 + 7 # модуль, по которому выполняются все операции. должно быть большым и взаимно простым с p (опционально вообще простым)

def hash(s):
  n = len(s)
  hash = 0
  for i in range(n):
    hash = (hash * p + ord(s[i])) % mod
return hash
```

**Поиск подстроки в строке при помощи хешей (алгоритм Рабина-Карпа)**
_Идея алгоритма:_ сначала сравниваем подстроку исходной строки с нужной, если хэши равны, то производим посимвольное сравнение
```rb
t = input() 
s = input()
# задача: найти индексы вхождений строки s в строку t
N = len(t)
M = len(s)
hs = hash(s)
ht = hash(s[:M])
for i in range(0, N - M + 1):
  if hs == ht and s[i:i + M] == h:
    print(i)
  if i != N - M: #пересчитываем хэш
    ht = ((ht - ord(t[i]) * p **(M - 1)) * p + ord(t[i + M])) % mod
```
**Оценка времени выполнения:** в лучшем случае О(N) - никакие хэши не равны, в худшем - О(N * M) - все хэши равны, идет проверка всех подстрок

## 2 Вопрос. 
### Z-функция, понятие z-блока. Пи-функция. Определения. Алгоритмы построения. Оценка времени выполнения.
**Z-функция** - функция, возвращающая max длину префикса строки, равного префиксу подстроки s[i::]
**Z-блок** - отрезок с границами l и r включительно, где r - наибольшееотдальение от начала строки, равное len(s[:l:]) + длина префикса, l - индекс элемента с макс отдалением

```rb
def z_func(s):
  N = len(s)
  z = [0] * N
  l, r = 0
  for i in range(1, N):
    ind = 0
    if i <= r:
      ind = min(z[i - l], r - i + 1)
    while ind + i < N and s[ind] == s[i + ind]:
      ind += 1
    if i + ind > r:
      l = i
      r = i + ind - 1
    z[i] = ind

```
**Оценка времени выполнения:** -  в алгоритме мы делаем столько же действий, сколько раз сдвигается правая граница z-блока - О(N)


**Префикс-функция** - функция, возвращающая max длину префикса, равного постфиксу подстроки s[:i]
```rb
def pref_func(s):
  N = len(s)
  p = [0] * (N + 1)
  p[0] = -1
  for i in range(1, N + 1):
    k = p[i - 1]
    while k != -1 and s[k] != s[i - 1]:
      k = p[k]
    p[i] = k + 1
```

**Оценка времени выполнения:** O(N)

**Алгоритм КМП**
_Идея_: склеиваем строки s и t с помощью символа, не встречающегося в этих строках, в строку ans
Тогда вхождения строки s в строку t будет там, где:
+ значение префикс-функции строки ans равно длине s (в этом месте вхождение заканчивается, начинается в ans[i - p[i]]
+ значение z-функции строки ans равно длине s (в этом месте начинается)
**КМП с префикс-функцией**
```rb
l = s + "#" + t
ans = pref_func(l)
for i in range(len(l) + 1):
    if ans[i] == len(s):
        print(i - len(s) - 1 - len(s))
```

**КМП с z-функцией**
```rb
l = s + "#" + t
ans = z_func(l)
for i in range(len(l) + 1):
    if ans[i] == len(s):
        print(i - len(s) - 1)
```

## 3 Вопрос.
### Алгоритм Дейкстры. Постановка задачи, описание алгоритма за O(V^2 + E). Улучшение алгоритма для разреженных графов за O(ElogV).

**Алгоритм Дейкстры** - алгоритм поиска кратчайшего пути в графе от вершины V1 до всех остальных.

**Постановка задачи, описание алгоритма за O(V^2 + E)**
_Суть_: находим вершину curr, от которой расстояние до нужной вершины А наименьшее, для каждого соседа curr сравниваем расстояние до А по прямой и через соседа, находим минимальное.\
Вершину с минимальным расстояние до А делаем curr, если она еще не была такой (если она не посещена)
массив **dist** хранит текущие минимальные расстояния от вершины  до остальных вершин, изначально они равны бесконечности, dist[A] = 0
массив **visited** хранит информацию о посещенных вершинах: 
+ visited[i] = True, если вершина уже была curr (посещена)
+ visited[i] = False, если вершина еще не была curr (не посещена)

```rb
n = int(input())
adj_list = [[] for _ in range(n)]
for i in range(n):
    m = int(input())
    for j in range(m):
        adj_list[i].append(tuple(map(int, input().split())))
        # (номер, вес)

curr = int(input())
dist = [float('inf')] * n
dist[curr] = 0
min_v = curr

visited = [0] * n
count = n

while count > 0:
    for to, weight in adj_list[curr]:
        if not visited[to]:
            if dist[to] > dist[curr] + weight:
                dist[to] = dist[curr] + weight
    visited[curr] = True
    count -= 1
    # ищем новую минимальную вершину
    for i in range(n):
        if min_v == curr or not visited[i] and dist[i] < dist[min_v]:
            min_v = i
    curr = min_v

print(*dist)
```
**Асимптотика** - O(V^2 + E)

**Улучшение алгоритма для разреженных графов за O(ElogV)**
_Идея_: создаем min-кучу,добавляем к нее элементы (weight, to) при изменении расстояния в dist
пока куча полная: 
достаем элемент
  если вершина не посещена:
    посещаем всех ее сосдей:
      если сосед не посещен и расстояние от А до соседа > расстояния от А до вершины + расстояние от вершины до соседа:
        заменяем dist[соседа]
        добавляем в кучу (сосед, dist[сосед])
    отмечаем вершину посещенной
    


```rb
n, m = [int(el) for el in input().split()]
adj_list = [[] for _ in range(n)]
visited = [False for _ in range(n)]
dist = [float('inf') for _ in range(n)]
dist[0] = 0
heap = []
count_of_visited = 0
heapq.heappush(heap, (0, 0))
for j in range(m):
    f, t, w = [int(el) for el in input().split()]
    adj_list[f].append([w, t])

while heap:
    hp = heapq.heappop(heap)
    weight, to = hp[0], hp[1]
    if not visited[to]:
        for el in adj_list[to]:
            w, t = el[0], el[1]
            if not visited[t] and dist[t] > dist[to] + w:
                dist[t] = dist[to] + w
                heapq.heappush(heap, (dist[t], t))
        visited[to] = True
        count_of_visited += 1
    if count_of_visited == n: break
print(*dist)
```
**Асимптотика** -  O(ElogV)

## 4 Вопрос.
### Динамическое программирование. Основные понятия. Задача о кузнечике. Задача о черепашке. Оценка времени выполнения.
**Динамическое программирование** - способ решения сложных задач с помощью деления их на более маленькие и простые подзадачи.
**Основные понятия**:
+ база - начальные данные, которые можно найти
+ состояние - данные, доступные на i-ом шаге
+ переход - действия (формула) для того, чтобы получить данные из другого состояния


**Задача о кузнечике.**

На вершине лесенки, содержащей n ступенек, сидит кузнечик, который хочет спуститься вниз, к основанию. Кузнечик умеет прыгать на следующую ступеньку, на ступеньку через одну или через две.
Задача - посчитать количество всевозможных маршрутов кузнечика для спуска с последней ступеньки до основания.
(необходимо вывести ответ по модулю 10**9 + 7)

_Решение_:

**база**: dp[0] = 1 - количество способов добраться до первой ступеньки

**состояние**: dp[i] - количество способов добраться до i-ой ступеньки

**переход**: dp[i + j] += dp[i] (j in range(1,4)) - добавляем к возможным для прыжка ступенькам количество способов добраться до пердыдущей ступеньки

```rb
n = int(input()) + 1 # кол-во ступенек
dp = [0] * n
dp[0] = 1 # база
# dp[i] - количество способов добраться до i-й ступеньки
for i in range(n):
    for j in range(1, 4):
        if i + j < n:
            dp[i + j] += dp[i] # переход
print(dp[-1])
```

*Оценка времени выполнения* - O(N)


**Задача о черепашке**
В верхнем левом углу прямоугольной таблицы размером n∗m сидит черепашка и предвкушает несметные богатства. В каждой ячейке таблицы лежит некоторое количество монет. Черепашка может двигаться либо в правую соседнюю клетку, либо в нижнюю соседнюю и никуда иначе. Она должна закончить маршрут в правой нижней клетке таблицы.
Задача: определить максимальное количество монеток, которое сможет собрать черепашка, и построить маршрут, на котором это максимальное количество монеток будет достигнуто.


**база**: dp[1][1] - макс количесвто монеток, которое можно получить, находясь в начальной ячейке (все монетки в ней)

**состояние**: dp[i][j] - максимальное количество монеток, которое можно получить, попав на [i][j] ячейку

**переход**: dp[i][j] += max(dp[i][j - 1], dp[i - 1][j])

```rb
n, m = list(map(int, input().split()))
dp = [[0] * (m + 1)]
for _ in range(n):
    dp.append([0] + list(map(int, input().split())))

for i in range(1, n + 1):
    for j in range(1, m + 1):
        dp[i][j] += max(dp[i - 1][j], dp[i][j - 1])
print(dp[n][m])

steps = []

# Восстановление ответа
i, j = n, m
while i != 1 or j != 1:
    if i == 1:
        j -= 1
        steps.append("R")
    elif j == 1:
        i -= 1
        steps.append("D")
    else:
        if dp[i - 1][j] > dp[i][j - 1]:
            steps.append("D")
            i -= 1
        else:
            steps.append("R")
            j -= 1

steps.reverse()
print(*steps)
```

*Оценка времени выполнения* - O(N*M)

## 5 Вопрос.
### Задача о рюкзаке: постановка классической задачи, решение методом динамического программирования, оценка времени работы.

**Постановка задачи**
Есть рюкзак, который может поднять максимально возможный вес w. Также задан набор предметов, каждый предмет имеет два параметра – вес p и стоимость c. Необходимо найти набор предметов максимальной стоимости, вес набора предметов не больше w.


**cостояние:**
dp[i][j] - максимальное количество денег, которое можно получить, рассмотрев первые i вещей и рюкзак вместимостью j кг

**база:**
dp[0][j] = 0, j in range(0, n + 1)
dp[i][0] = 0, i in range(0, n + 1)

**переход:**
_if j >= weight[i]:_
    dp[i][j] = max(dp[i - 1][j - weight[i]] + coasts[i], dp[i - 1][j]

dp[i - 1][j - weight[i]] + coasts[i] - берем предмет (dp[i - 1][j - weight[i]] - макс стоимость с оставшейся вместимостью при рассмотрении оставшихся предметов, coasts[i] - стоимость предмета)

_else:_
    dp[i][j] = dp[i - 1][j] - не берем предмет, так как его вес больше вместимости рюкзака


```rb
n = int(input()) # количество вещей
w = int(input()) # максимальный вес, который выдерживает рюкзак
weight = [0] + list(map(int, input().split()))
coasts = [0] + list(map(int, input().split()))

dp = [[0] * (w + 1) for _ in range(n + 1)]

for i in range(1, n + 1):
    for j in range(1, w + 1):
        if weight[i] <= j:
            dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + coasts[i])
        else:
            dp[i][j] = dp[i - 1][j]

# восстановление ответа
belongings = []
i = n
j = w
while i != 0 and j != 0:
    if j >= weight[i] and dp[i - 1][j] < dp[i - 1][j - weight[i]] + coasts[i]:
        belongings.append(i)
        j -= weight[i] # уменьшаем свободное место на вес вещи
        i -= 1 # уменьшаем  количество вещей
    else:
        i -= 1 # уменьшаем только количество вещей, вес остается прежним

summ_weight = 0
summ_costs = 0
for ind in belongings:
    summ_weight += weight[ind]
    summ_costs += coasts[ind]
print(summ_costs)
print(summ_weight)
print(len(belongings))
print(*[weight[k] for k in belongings])
print(*[coasts[g] for g in belongings])
```

**Оценка времени работы** - O(N * W)


## 6 Вопрос.
### Задача поиска наибольшей возрастающей подпоследовательности в массиве. Решение за O(N^2) методом динамического программирования. Решение за O(NlogN) методом динамического программирования.

**Задача:** найти наибольшую возрастающую подпоследовательность в массиве.
  

**Решение за O(N^2) методом динамического программирования.**

Используется:

+ a[n] - массив элементов последовательности
+ dp[n]

**Состояние:** 
dp[i] - длина НВП последовательности a[:i + 1]

**База:** 
dp[0] = 1 - длина НВП последовательности из 1 элемента

**Переход:**

if a[j] < a[i]:
    dp[i] = max(dp[i], dp[j] + 1)

Если элемент a[i] может быть продолжением подпоследовательности, оканчивающейся на элемент a[j], то dp[i] равно максимуму из суммы длины этой подпоследовательности dp[j] и 1 (новый элемент a[i]) и текущей длины dp[i]

```rb
n = int(input())
a = [int(el) for el in input().split()]
dp = [0] * n
dp[0] = 1

max_len = 0

for i in range(n):
    for j in range(i):
        if a[j] < a[i]:
            dp[i] = max(dp[j] + 1, dp[i])
        max_len = max(max_len, dp[i])

print(max_len)
```

**Решение за O(NlogN) методом динамического программирования.**

**Состояние:** 
d[i] - min значение, на которое может быть закончена НВП длины i

**База:** 

d = [INF] * (n + 1)
d[0] = -INF

**Переход:**

Находим r, где d[r] - первый элемент, больший a[i]
d[r] = a[i] - заменяем окончание подпоследовательности длины i на более маленькое значение
lis[r] = lis[r - 1] + [a[i]]


Используется:
+ a[n] - подпоследовательность
+ d[n]


```rb
n = int(input())
a = [int(el) for el in input().split()]
INF = float('inf')
d = [INF] * (n + 1)
d[0] = -INF
lis = [[] for _ in range(n + 1)] # хранит НВП длины i

lis[1] = [a[0]] # делаем для i = 0 отдельно, тк в цикле ниже рассматриваем i с единицы
# (индекс первого большего элемента в d равен 1)

for i in range(1, n):
    l = -1
    r = i + 1
    while r - l > 1: # ищем в d первый элемент, больший a[i]
        m = (l + r) // 2
        if d[m] >= a[i]:
            r = m
        else:
            l = m
    d[r] = a[i]
    lis[r] = lis[r - 1] + [a[i]]

ind = 0
while d[ind] != INF:
    ind += 1
print("Длина НВП:", ind)
print("НВП:", lis[ind])
```

## 7 Вопрос.
### Задача поиска длины наибольшей общей подпоследовательности двух строк. Решение за O(N*M) методом динамического программирования.

**Состояние:**

dp[i][j] - длина наибольшоей общей подпоследовательности строк t[0..i] и s[0..j]

**База:** 

dp[0][j] = 0
dp[i][0] = 0 


**Переход:**

if t[i - 1] == s[j - 1]:
  dp[i][j] = dp[i - 1][j - 1] + 1

else:
  dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
  
  

```rb
m = int(input())
s = input().split()

n = int(input())
t = input().split()

dp = [[0] * (m + 1) for _ in range(n + 1)]

for i in range(1, n + 1):
    for j in range(1, m + 1):
        if s[j - 1] == t[i - 1]:
            dp[i][j] = dp[i - 1][j - 1] + 1
        else:
            dp[i][j] = min(dp[i - 1][j], dp[i][j - 1])

print("Длина НОП:", dp[n][m])

i = len(t)
j = len(s)

print(t, s)
lcs = ""

while i != 0 and j != 0:
    if s[j - 1] == t[i - 1]:
        lcs = s[j - 1] + " " + lcs
        i -= 1
        j -= 1
    else:
        if dp[i - 1][j] > dp[i][j - 1]:
            i -= 1
        else:
            j -= 1
print("НОП:", lcs)
```

## 8 Вопрос.
### Поиск суммы на отрезке + изменение значения в массиве. Постановка задачи, решение за O(logN) при помощи дерева отрезков. Построение дерева отрезков. Оценка времени выполнения.

**Постановка задачи**:
Дан массив, нужно найти сумму элементов на отрезке [l; r)

**Решение за O(logN) при помощи дерева отрезков**
Функция def sum_(..) _см [segment tree(sum)](https://github.com/vasslin/Algoritms-and-Data-structures/blob/main/segment%20tree%20(sum))_





