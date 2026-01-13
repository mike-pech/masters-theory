Дерево отрезков (Segment tree) — это мощная структура данных, позволяющая не только хранить массив чисел за $N log N$ и очень быстро находить эти числа за $log N$, но и гибко менять структуру подлежащего массива, меняя один элемент или реструктурируя сразу промежуток элементов
# Структура
## Простой пример

Для задачи "нахождения количества уникальных элементов в массиве" хорошо подходит стратегия *разделяй и властвуй* — разделим наш массив $a[0…n-1]$ на две половинки $a[0…n/2-1]$ и $a[n/2…n-1]$ — для каждой высчитаем кол-во уникальных элементов и сохраним в узле

> Получаем бинарное дерево!

…точнее его идею, ведь на самом деле это скорее опять же массив чисел

```
[ 1000  2  3  1  1000  1000 ]
// The idea
									[ 1...6 ] — 4
							[ 1...3 ] — 3 [ 4...6 ] — 2
					[ 1...2 ] — 2 [ 3...4 ] — 2 [ 5...6 ] — 1
[ 1..1 ] — 1 [ 2..2 ] — 1 [ 3..3 ] — 1 [ 4..4 ] — 1 [ 5..5 ] — 1 [ 6..6 ] — 1
// Implemented as an array
[ 4 3 2 2 2 1 1 1 1 1 1 1] // 4 * n чисел
```
## Как сконструировать?

Определившись со структурой и хранением элементов можно сделать функцию построения

1. Начинаем обрабатывать подлежащий массив с двух листьев дерева
2. Считаем кол-во уникальных элементов для них
3. Объединяем в одно дерево
4. Берём ещё схожее поддерево 
5. Повторяем шаги 2–4 для них, пока не обработаем массив

> [!IMPORTANT] Загвоздка задачи
> Нас интересуют уникальные элементы, а сами они гарантированно не повторяются только на самом нижнем уровне, который работает с индивидуальными элементами массива — мы не можем просто сложить полученные результаты в поддеревьях
### Реализация
Для суммы в массиве

```c
/*
 * Простейшая реализация сегментного дерева для суммы чисел в массиве через подобный бинарной
 * куче массив
 */
#include <stdlib.h>
#include <stdio.h>

#define MAXN 8

int n, t[4*MAXN];

int min(int a, int b) { return (a < b) ? a : b; }
int max(int a, int b) { return (a > b) ? a : b; }

// Рекурсивное построение a[] от вершины v с границами tl и tr
void build(int a[], int v, int tl, int tr) {
	if (tl == tr) t[v] = a[tl];	// Значение листа == значение родительского узла
	else {
		int tm = (tl + tr) / 2;
		build(a, v*2, tl, tm);		// Рекурсивно строим налево до m
		build(a, v*2+1, tm+1, tr); 	// и направо от m+1
		t[v] = t[v*2] + t[v*2+1];
	}
}

// Обработка запросов на сумму сегментов с границами tl и tr
int sum(int v, int tl, int tr, int l, int r) {
	if (l > r) return 0;	// Некорректный запрос
	if (l == tl && r == tr) return t[v]; // Запрос в рамках сегмента
	int tm = (tl + tr) / 2;
	return sum(v*2, tl, tm, l, min(r, tm)) + sum(v*2+1, tm+1, tr, max(l, tm+1), r);
}

void update(int v, int tl, int tr, int pos, int new_val) {
	if (tl == tr) t[v] = new_val;
	else {
		int tm = (tl + tr) / 2;
		if (pos <= tm)
			update(v*2, tl, tm, pos, new_val);
		else 
			update(v*2+1, tm+1, tr, pos, new_val);
		t[v] = t[v*2] + t[v*2+1];
	}
}

int main() {
	printf("Please enter a number under 8: ");
	scanf("%d", &n);

	printf("\nEnter %d numbers divided by spaces\n", n);
	for (int i = 0; i < n; i++) {
		int e;
		scanf("%d", &e);
		build(t, 1, 0, n-1);
	}

	int m;
	printf("\nPlease enter a number of queries: ");
	scanf("%d", &m);
	printf("\nEnter query borders divided by spaces:\n");
	for (int i = 0; i < m; i++) {
		int ql, qr;
		scanf("%d %d", &ql, &qr);
		printf("%d\n", sum(1, 0, n, ql, qr));
	}

	return 0;
}
```
#### Lazy propagation

Для обновления всех значений сегментного дерева не обязательно проходить весь массив — достаточно просто изменить сегменты в нашем промежутке — что выйдет за всё те же $O(log\ n)$ 

К примеру, чтобы добавить число ко всем узлам в промежутке:
```c
void build(int a[], int v, int tl, int tr) {
    if (tl == tr) {
        t[v] = a[tl];
    } else {
        int tm = (tl + tr) / 2;
        build(a, v*2, tl, tm);
        build(a, v*2+1, tm+1, tr);
        t[v] = 0;
    }
}

void update(int v, int tl, int tr, int l, int r, int add) {
    if (l > r)
        return;
    if (l == tl && r == tr) {
        t[v] += add;
    } else {
        int tm = (tl + tr) / 2;
        update(v*2, tl, tm, l, min(r, tm), add);
        update(v*2+1, tm+1, tr, max(l, tm+1), r, add);
    }
}

int get(int v, int tl, int tr, int pos) {
    if (tl == tr)
        return t[v];
    int tm = (tl + tr) / 2;
    if (pos <= tm)
        return t[v] + get(v*2, tl, tm, pos);
    else
        return t[v] + get(v*2+1, tm+1, tr, pos);
}
```
#### Lazy update

Аналогично поступаем с обновлениями, но в другой плоскости — если нам надо обновить промежуток $[0…n-1]$, то мы просто обновляем корень дерева и возвращаемся. При следующем обновлении значений в промежутке $[0…n/2]$ предыдущее изменение в корне мы пропихиваем вниз по дереву

> В общем случае обновление значения мы пропагируем вниз от корня поддерева к обоим его дочерним узлам

Для этого нам нужна маркировка узлов для обновления в отдельном массиве — сложность по памяти теперь $O(n^2)$ 

```c
void push(int v) {
    if (marked[v]) {
        t[v*2] = t[v*2+1] = t[v];
        marked[v*2] = marked[v*2+1] = true;
        marked[v] = false;
    }
}

void update(int v, int tl, int tr, int l, int r, int new_val) {
    if (l > r) 
        return;
    if (l == tl && tr == r) {
        t[v] = new_val;
        marked[v] = true;
    } else {
        push(v);
        int tm = (tl + tr) / 2;
        update(v*2, tl, tm, l, min(r, tm), new_val);
        update(v*2+1, tm+1, tr, max(l, tm+1), r, new_val);
    }
}

int get(int v, int tl, int tr, int pos) {
    if (tl == tr) {
        return t[v];
    }
    push(v);
    int tm = (tl + tr) / 2;
    if (pos <= tm) 
        return get(v*2, tl, tm, pos);
    else
        return get(v*2+1, tm+1, tr, pos);
}
```
### Несколько измерений

Дерево отрезков можно растянуть до нескольких измерений, если того требует задача (например, индексация матрицы, а не массива) — тогда мы строим дерево отрезков первого измерения из деревьев отрезков второго измерения

Например для $x$ и $y$
```c
void build_y(int vx, int lx, int rx, int vy, int ly, int ry) {
    if (ly == ry) {
        if (lx == rx)
            t[vx][vy] = a[lx][ly];
        else
            t[vx][vy] = t[vx*2][vy] + t[vx*2+1][vy];
    } else {
        int my = (ly + ry) / 2;
        build_y(vx, lx, rx, vy*2, ly, my);
        build_y(vx, lx, rx, vy*2+1, my+1, ry);
        t[vx][vy] = t[vx][vy*2] + t[vx][vy*2+1];
    }
}

void build_x(int vx, int lx, int rx) {
    if (lx != rx) {
        int mx = (lx + rx) / 2;
        build_x(vx*2, lx, mx);
        build_x(vx*2+1, mx+1, rx);
    }
    build_y(vx, lx, rx, 1, 0, m-1);
}
```

Для суммы
```c
int sum_y(int vx, int vy, int tly, int try_, int ly, int ry) {
    if (ly > ry) 
        return 0;
    if (ly == tly && try_ == ry)
        return t[vx][vy];
    int tmy = (tly + try_) / 2;
    return sum_y(vx, vy*2, tly, tmy, ly, min(ry, tmy))
         + sum_y(vx, vy*2+1, tmy+1, try_, max(ly, tmy+1), ry);
}

int sum_x(int vx, int tlx, int trx, int lx, int rx, int ly, int ry) {
    if (lx > rx)
        return 0;
    if (lx == tlx && trx == rx)
        return sum_y(vx, 1, 0, m-1, ly, ry);
    int tmx = (tlx + trx) / 2;
    return sum_x(vx*2, tlx, tmx, lx, min(rx, tmx), ly, ry)
         + sum_x(vx*2+1, tmx+1, trx, max(lx, tmx+1), rx, ly, ry);
}
```

Для обновлений
```c
void update_y(int vx, int lx, int rx, int vy, int ly, int ry, int x, int y, int new_val) {
    if (ly == ry) {
        if (lx == rx)
            t[vx][vy] = new_val;
        else
            t[vx][vy] = t[vx*2][vy] + t[vx*2+1][vy];
    } else {
        int my = (ly + ry) / 2;
        if (y <= my)
            update_y(vx, lx, rx, vy*2, ly, my, x, y, new_val);
        else
            update_y(vx, lx, rx, vy*2+1, my+1, ry, x, y, new_val);
        t[vx][vy] = t[vx][vy*2] + t[vx][vy*2+1];
    }
}

void update_x(int vx, int lx, int rx, int x, int y, int new_val) {
    if (lx != rx) {
        int mx = (lx + rx) / 2;
        if (x <= mx)
            update_x(vx*2, lx, mx, x, y, new_val);
        else
            update_x(vx*2+1, mx+1, rx, x, y, new_val);
    }
    update_y(vx, lx, rx, 1, 0, m-1, x, y, new_val);
}
```