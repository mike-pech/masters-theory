# Проблема

> Проблема N+1 — имеется алгоритм загрузки N записей. Каждая запись имеет множество вложенных записей другого домена (например, посты и их комментарии). Проблема такого подхода в том, что чтобы подгрузить по 1 вложенной записи для N верхнеуровенвых записей потребуется сделать N+1 запросов, что неоправданно нагружает базу данных и вводит ненужные задержки

Допустим для примера с постами и их авторами:

```go
type Post struct {
    ID       int
    Title    string
    AuthorID int
}

type Author struct {
    ID   int
    Name string
}
```

Для загрузки информации об авторе поста для каждого поста (допустим в выдаче):

```go
// The inefficient N+1 approach
func getPostsAndAuthors_N_Plus_1(db *sql.DB) {
    // 1. The "1" query: Fetch all posts
    rows, _ := db.Query("SELECT id, title, author_id FROM posts")
    defer rows.Close()

    var posts []Post
    for rows.Next() {
        var p Post
        rows.Scan(&p.ID, &p.Title, &p.AuthorID)
        posts = append(posts, p)
    }

    // 2. The "N" queries: Fetch author for each post
    for _, p := range posts {
        var authorName string
        // This query runs inside the loop for every single post!
        db.QueryRow("SELECT name FROM authors WHERE id = ?", p.AuthorID).Scan(&authorName)
        fmt.Printf("post: %s, author: %s\n", p.Title, authorName)
    }
}
```

Вследствие чего для каждого поста мы должны подгружать N имён авторов

```sql
SELECT id, title, author_id FROM posts;
SELECT name FROM authors WHERE id = 101;
SELECT name FROM authors WHERE id = 102;
SELECT name FROM authors WHERE id = 101;
-- ... и так далее для каждого поста
```
# Решение — Eager Loading

Подгрузка наперёд — запросить с БД партию из N верхнеуровневых записей, запомнить связки (внешние ключи), потом запросить N вложенных записей по массиву ключей — итого две операции сложностью O(n) — *Ого, алгосы!

В примере с постами мы просто загружаем партию постов, закидываем ID авторов в массив, а потом загружаем авторов по массиву ID

```go
// The efficient eager loading approach
func getPostsAndAuthors_Optimized(db *sql.DB) {
    // 1. Still fetch all posts
    rows, _ := db.Query("SELECT id, title, author_id FROM posts")
    defer rows.Close()

    var posts []Post
    // Collect all the unique author IDs we will need
    authorIDs := make(map[int]bool) 
    for rows.Next() {
        var p Post
        rows.Scan(&p.ID, &p.Title, &p.AuthorID)
        posts = append(posts, p)
        authorIDs[p.AuthorID] = true
    }

    // Create a map to easily look up authors by their ID
    authorMap := make(map[int]string)

    // 2. Fetch all required authors in a SINGLE second query
    // ... code to build the IN clause and run the query ...

    // 3. Now, combine the data in memory — no more database calls!
    for _, p := range posts {
        authorName := authorMap[p.AuthorID]
        fmt.Printf("post: %s, author: %s\n", p.Title, authorName)
    }
}
```

Итого два запроса

```sql
-- Query 1
SELECT id, title, author_id FROM posts;

-- Query 2
SELECT id, name FROM authors WHERE id IN (101, 102, ...);
```
# Вывод

> [!IMPORTANT] При запросе N записей с базы данных всегда задавай вопрос "Понадобится ли мне загружать вместе с ними N вложенных записей?"
> Если да, то стратегия подгрузки записей должна быть не такой прямолинейной во избежание проблемы N+1
