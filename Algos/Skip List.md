>	Блин! Вот бы мой связный список был немного больше массивом
# Структура

Skip List или список с пропусками — это связный список, хранящий не только множество узлов со значениями и связями, но и т.н. «Экспресс-линию» — множество узлов со связями, которое позволяет пропустить некоторое количество элементов в поисках элемента с некоторым значением `v` 

Таким образом, при вставке или поиске элементов мы будем совершать бинарный поиск по экспресс-линии, чтобы найти место для нашего элемента или искомый элемент, например ищем 72:

> 72 > 14
> 72 < 79
>	72 > 50
>	72 < 79
>		72 > 66
>		72 < 79
>
> Значит, 72 должен быть в промежутке между 66 и 79, т.к. 66 < 72 < 79
> Идём от 66, пока не найдём 72 или не дойдём до 79, что будет означать, что 72 в списке нет

![[Pasted image 20251104104422.png]]

В целом структура достаточно эффективная, но реализация её накладная — требует отсортированного списка на каждом действии, что делает вставку элементов очень сложным процессом
## Асимптотика
### Вставка

Для начала нужно найти, куда вставить элемент в самом нижнем уровне списка (самом связном списке) — в какой промежуток элементов его протолкнуть

Пример:

![[Pasted image 20251104104931.png]]

Далее нужно перестроить индексы, чтобы в одном промежутке не скапливалась куча элементов. Для этого введём ..._монетку_

Если при проходе по промежутку мы кинули моентку и нам выпал «орёл», то возвращаем ссылку на текущий элемент, иначе — null. Если мы на самом нижнем уровне, просто вставляем элемент без броска монетки

Ранее при проходе элементов, если нам вернули ссылку на вставляемый элемент, то мы вставляем его на текущий уровень

Такой фокус с монеткой позволяет распределить пропуски между элементами и более-менее равномерно заполнить экспресс-линии 

Иногда при вставке нового элемента нам потребуется создать ещё уровень. Тогда мы создаём новый отсортированный список только с новым элементом и ссылаемся на него в высшем уровне списка 

В общем и целом — амортизированное O(log n)
### Поиск

1. Начинаем поиск элемента в самом верхнем уровне
2. Переходим к следующему элементу списка, пока значение в следующей ячейке меньше
3. Переместимся на один уровень вниз и перейти к шагу 2. Если мы уже на первом уровне — прекратим поиск и вернём ссылку на текущую вершину

Гордое O(log n)	— за счёт использования типа бинарного поиска по экспресс-линиям
### Удаление

1. Начинаем удалять элемент с верхнего уровня
2. Переходим к следующему элементу, пока значение следующего элемента меньше ключа
3. Если элемент существует на данном уровне — удаляем его с этого уровня. Если мы не на первом уровне, то удаляем элемент ещё с нижнего уровня.

O(log n)
### Память

Вне зависимости от количества уровней O(n) — т.к. уровней всегда меньше, чем n (иначе в них нет смысла). В общем случае рассматриваются 2–3 уровня
## Реализация

Кривенькая реализация на C++. TODO: сделать несколько уровней списков и многоуровневую вставку
```c++
class ListNode {
public:
	int value;
	int index;
	ListNode *next;
	ListNode *link;

	ListNode(int v, int i, ListNode *l = nullptr) {
		value = v;
		index = i;
		next = nullptr;
		link = l;
	}
};

class SeekList {
public:
	SeekList(int s) { 
		head = nullptr; 
		exp_head = nullptr; 
		size = 0;
		step = s;
	}

	~SeekList() {
		ListNode* current = head;
		while (current != nullptr) {
			ListNode* next = current->next;
			delete current;
			current = next;
		}
		head = nullptr;

		current = exp_head;
		while (current != nullptr) {
			ListNode* next = current->next;
			delete current;
			current = next;
		}
		exp_head = nullptr;
	}

	void push_back(int element) {
		ListNode* newNode = new ListNode(element, size);
		if (head == nullptr) {
			head = newNode;
		} else {
			ListNode* tail = walk(head);
			tail->next = newNode;
		}

		if (size % step == 0) {
			ListNode* newExpressNode = new ListNode(element, size, newNode);
			if (exp_head == nullptr) {
				exp_head = newExpressNode;
			} else {
				ListNode* exp_tail = walk(exp_head);
				exp_tail->next = newExpressNode;
			}
		}

		size++;
	}

	ListNode* walk(ListNode* curr) {
		if (curr == nullptr) {
			throw out_of_range("List empty!");
		}

		while (curr->next != nullptr) {
			curr = curr->next;
		}

		return curr;
	}

	int walk_for(int target) {
		if (empty()) {
			throw out_of_range("List empty!");
		}

		target--;	
		if (target > size) return -1;

		ListNode* curr = exp_head;
		ListNode* prev_exp = nullptr;

		while (curr != nullptr && curr->index <= target) {
			prev_exp = curr;
			curr = curr->next;
		}

		ListNode* search_start;
		if (prev_exp == nullptr) {
			search_start = head;
		} else {
			search_start = prev_exp->link;
		}

		curr = search_start;
		while (curr != nullptr && curr->index <= target) {
			if (curr->index == target) {
				return curr->value;
			}
			curr = curr->next;
		}

		return -1;
	}

	ListNode* get_node(int target) {
		if (empty() || target < 0 || target >= size) return nullptr;

		ListNode* curr = exp_head;
		ListNode* prev_exp = nullptr;

		while (curr != nullptr && curr->index <= target) {
			prev_exp = curr;
			curr = curr->next;
		}

		ListNode* search_start = (prev_exp == nullptr) ? head : prev_exp->link;
		curr = search_start;

		while (curr != nullptr && curr->index <= target) {
			if (curr->index == target) {
				return curr;
			}
			curr = curr->next;
		}

		return nullptr;
	}

	bool empty() {
		return head == nullptr;
	}

	ListNode* head;
private:
	int size;
	int step;
	ListNode* exp_head;
};
```
