Приоритетная очередь — это структура данных, элементы в которую добавляются и удаляются из неё согласно приоритету (часто по убыванию — сначала наибольший и всё меньше)

```c++
#include <iostream>
#include <queue>
using namespace std;

int main() {
    // Create a max-heap priority queue (default)
    priority_queue<int> pq;
	
    // Insert elements
    pq.push(30);
    pq.push(10);
    pq.push(20);
    pq.push(40);
	
    cout << "Elements removed from priority queue in order:\n";
    // Remove elements (largest element comes out first)
    while (!pq.empty())
    {
        cout << pq.top() << " ";
        pq.pop();
    }
    return 0;
}
```

По-умолчанию в C++ в основе приоритетной очереди лежит макс-куча ([[Heap]]), которая и сортирует элементы по убыванию. Тем самым удаление элементов из очереди гарантирует сортировку их по убыванию

...ну, это если ты будешь доставать через `top` и `pop`. А в C++ и очередь можно использовать как стек, и молоток как зубило...