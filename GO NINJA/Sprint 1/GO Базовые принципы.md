## ** Объявления: переменные и константы**

```go
var a int          // zero-value = 0
a := 42            // короткая форма
const Pi = 3.14    // константа
```

* := используется ТОЛЬКО внутри функций

* Неиспользуемые переменные — ошибка компиляции

* Все переменные в Go инициализируются **zero-value**:

  * int → 0
  * string → ""
  * bool → false
  * pointer/slice/map/interface → nil

* Константы (`const`) применимы только для **компилируемых выражений** (без вызовов функций)

---

### **📏 Размеры переменных (на 64-битной архитектуре)**

| **Тип** | **Размер** | **Примечание**            |
| ------- | ---------- | ------------------------- |
| int     | 8 байт     | зависит от архитектуры    |
| int32   | 4 байта    |                           |
| int64   | 8 байт     |                           |
| float64 | 8 байт     | стандарт для вещественных |
| bool    | 1 байт     |                           |
| string  | 16 байт    | указатель + длина         |
| slice   | 24 байта   | ptr + len + cap           |
| map     | 8 байт     | указатель на хеш-таблицу  |
| pointer | 8 байт     |                           |

---

## **🔤 Байты и Руны**

```go
var b byte = 'A'     // ASCII, byte == uint8 → 65
var r rune = 'Ж'     // Unicode, rune == int32 → 1046
fmt.Printf("%c %U", r, r) // Ж U+0416
```

* string в Go — набор байт
* rune — символ Unicode (4 байта)
* Для корректной обработки Unicode используют `for _, r := range str`

---

## **🔢 Базовые Типы**

| **Тип** | **Пример**    | **Zero-value** |
| ------- | ------------- | -------------- |
| int     | var x int     | 0              |
| float   | var y float64 | 0.0            |
| string  | var s string  | ""             |
| bool    | var b bool    | false          |
| byte    | byte('A')     | 0              |
| rune    | rune('Ж')     | 0              |

* Все ссылочные типы (`slice`, `map`, `chan`, `func`, `interface`) имеют zero-value = `nil`

---

## **🔁 Преобразование типов**

```go
var x int = 42
var y float64 = float64(x)  // явное преобразование
```

* В Go **всегда нужно явно преобразовывать типы**
* Автоматического преобразования нет (в отличие от Python или C++)

---

## **🧾 Область Видимости**

* Глобальные: вне функций
* Локальные: внутри функций

```go
var global int
func main() {
    local := 5
    fmt.Println(global, local)
}
```

---

## **📍 Множественное Присваивание**

```go
a, b := 1, 2
a, b = b, a
x, _, z := 1, 2, 3
```

---

## **🧩 Функции и Возвраты**

```go
func sum(a, b int) int {
    return a + b
}

func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

---

## **🔁 Условия и Циклы**

```go
if x > 0 {
    fmt.Println("Positive")
} else if x == 0 {
    fmt.Println("Zero")
} else {
    fmt.Println("Negative")
}

for i := 0; i < 5; i++ { fmt.Println(i) }
for condition { ... }      // аналог while
for { ... }                // бесконечный
```

---

## **❗ Обработка Ошибок**

```go
val, err := someFunc()
if err != nil {
    log.Fatal(err)
}
```

* Тип `error` встроен
* По соглашению — последний в сигнатуре

---

## **🔄 switch**

```go
switch day := time.Now().Weekday(); day {
case time.Monday:
    fmt.Println("Start")
case time.Sunday:
    fmt.Println("Rest")
default:
    fmt.Println("Go!")
}
```

---

## **🌐 Map (словари)**

```go
users := map[string]int{"Max": 30}
age, ok := users["Max"]
users["John"] = 25
delete(users, "Max")
```

* Ключи уникальны
* Чтение несуществующего ключа → zero-value

---

## **📚 Слайсы (Slices)**

```go
nums := []int{1, 2, 3}
nums = append(nums, 4)
copyTo := make([]int, len(nums))
copy(copyTo, nums)
```

* `len` — длина, `cap` — вместимость
* `append()` может пересоздать slice (будет новый backing array)
* `copy()` работает только для slices
* Копирует min(len(dst), len(src)) элементов

```go
s := []int{1, 2, 3}
s = append(s, 4)
```

⚠️ Результат append нужно сохранять

### **Структура Slice**

```go
struct Slice {
    ptr *T
    len int
    cap int
}
```

---

## **📎 Указатели (Pointers)**

```go
func mutate(s *string) {
    *s += "!"
}
s := "Hi"
mutate(&s)
```

* `*ptr` — разыменование
* `&val` — адрес
* Указатели нельзя складывать и вычитать (в отличие от C)

---

## **🧱 Struct и Методы**

```go
type User struct {
    Name string
    Age  int
}
func (u *User) SetName(name string) { u.Name = name }
```

* Методы с `*T` — изменяют оригинал
* Методы с `T` — работают с копией

---

## **🧬 Интерфейсы и Полиморфизм**

```go
type Shape interface {
    Area() float64
}

func describe(s Shape) {
    fmt.Println(s.Area())
}
```

* Duck typing: тип реализует интерфейс, если у него есть нужные методы
* `interface{}` — любой тип (аналог any)
* С Go 1.18 можно писать `any` вместо `interface{}`

### **Type Assertion: Зачем и как**

```go
var i interface{} = "hi"
s, ok := i.(string)
```

* Необходима, когда переменная типа interface{}, а нужно конкретное значение
* `ok == false` → безопасный путь избежать panic

### **Type Switch**

```go
switch v := i.(type) {
case string:
    fmt.Println("строка", v)
case int:
    fmt.Println("число", v)
default:
    fmt.Println("неизвестно")
}
```

* Подходит для обработки разных типов внутри interface{}

---

## **🧰 Встроенные функции**

* `make()` — создание slice/map/channel

```go
s := make([]int, 3, 10)
m := make(map[string]int)
```

* `new()` — выделяет память и возвращает `*T`

```go
ptr := new(int)
```

---

## **📦 Пакеты и Модули**

```go
go mod init example.com/project
go get github.com/some/lib
```

* Файл go.mod = имя + зависимости
* Импорт внешних пакетов: `go get <url>`
* Только имена с **заглавной буквы** экспортируются
* Запуск проекта: `go run .`
* Сборка бинарника: `go build`
* Форматирование: `go fmt ./...`

---

## **🧠 Специальная функция init()**

```go
func init() {
    // вызывается до main()
}
```

* Для инициализации состояний
* В каждом файле может быть `init()` — вызываются в порядке импорта

---

## **✅ Отличия от Python**

| **Категория** | **Go**                   | **Python**              |
| ------------- | ------------------------ | ----------------------- |
| Типизация     | Статическая              | Динамическая            |
| Циклы         | Один for                 | for, while              |
| Ошибки        | error, возвращается явно | Исключения (try/except) |
| Структуры     | struct, методы отдельно  | Классы                  |
| Интерфейсы    | Композиция по методам    | Явное наследование      |
| Переменные    | Обязаны использоваться   | Можно игнорировать      |
