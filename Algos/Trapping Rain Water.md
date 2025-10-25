> Дан массив стенок n×1. Найти максимальное количество клеток, которые можно заполнить водой

![[Pasted image 20251025205049.png]]
Вот и вся задача. А рейтинг на литкоде `hard` если чё

Решается просто за O(n) — проходимся по массиву двумя указателями, выявляем наиболее высокие стенки, ищем объём клетки, идём дальше к центру, пока указатели не столкнутся

> Объём клетки = самая высокая стена - стена в текущей клетке

```c++
#include <iostream>
using namespace std;

int main () {
	int n;
	cin >> n;

	int bricks[n];
	for (int i = 0; i < n; i++) {
		cin >> bricks[i];
	}

	int cc = 0;				// *в куболитрах
	int left = 0, right = n-1;		// Указатели
	int max_left = 0, max_right = 0;	// Макс высота стен

	while (left <= right) {
		if (bricks[left] <= bricks[right]) {	// Можно вместить воду
			if (bricks[left] > max_left) {	// Обновляем макс высоту
				max_left = bricks[left];
			} else { // Объём клетки = самая высокая стена - стена в текущей клетке
				cc += max_left - bricks[left];
			}
			left++;
		} else {	// Аналогично, но для правого указателя
			if (bricks[right] > max_right) {
				max_right = bricks[right];
			} else {
				cc += max_right - bricks[right];
			}
			right--;
		}
	}

	cout << cc << endl;

	return 0;
}	
```