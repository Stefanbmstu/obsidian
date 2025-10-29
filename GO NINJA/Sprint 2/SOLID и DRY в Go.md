## Single Responsibility Principle (SRP)
**Принцип единственной ответственности**: класс или модуль должен иметь только одну причину для изменения.  
Если компонент отвечает за одну задачу — его проще модифицировать, не затрагивая остальное.
### ❌ Нарушение SRP
```go
package main

import (
    "fmt"
    "net/http"
)

type User struct {
    ID        int
    FirstName string
    LastName  string
}

// одновременно обрабатывает HTTP-запросы и управляет пользователями
func (u *User) HandleRequest(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case "GET":
        // получение данных пользователя
    case "POST":
        // создание нового пользователя
    }
}
````
### **✅ Правильное применение SRP**
```go
package main

import (
    "fmt"
    "net/http"
)

type User struct {
    ID        int
    FirstName string
    LastName  string
}

type UserHandler struct {}

// отвечает только за HTTP-запросы
func (h *UserHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case "GET":
        // получение данных пользователя
    case "POST":
        // создание нового пользователя
    }
}
```
## **Open/Closed Principle (OCP)**

**Принцип открытости/закрытости**:
сущности должны быть открыты для расширения, но закрыты для изменения.
### **❌ Нарушение OCP**
```go
type Printer struct {}

func (p *Printer) Print(data string) {
    fmt.Println("Data: ", data)
}

// новая функциональность требует изменения класса
func (p *Printer) PrintHTML(data string) {
    fmt.Println("<habr>" + data + "</habr>")
}
```
### **✅ Соблюдение OCP**
```go
type Printer interface {
    Print(data string)
}

type TextPrinter struct {}
func (p *TextPrinter) Print(data string) {
    fmt.Println("Data: ", data)
}

type HTMLPrinter struct {}
func (h *HTMLPrinter) Print(data string) {
    fmt.Println("<html>" + data + "</html>")
}
```
## **Liskov Substitution Principle (LSP)**
**Принцип подстановки Барбары Лисков**:
подклассы должны быть взаимозаменяемы с базовыми типами.
### **❌ Нарушение LSP**
```go
type Bird struct {}
func (b *Bird) Fly() {
    fmt.Println("Птица летит")
}

type Penguin struct {
    Bird
}

// пингвин не летает → нарушение LSP
```
### **✅ Соблюдение LSP**
```go
type Bird struct {}
func (b *Bird) MakeSound() {
    fmt.Println("Птица издает звук")
}

type FlyingBird interface {
    Fly()
}

type Sparrow struct {
    Bird
}
func (s *Sparrow) Fly() {
    fmt.Println("Воробей летит")
}

type Penguin struct {
    Bird
}
// пингвин не реализует интерфейс FlyingBird
```
## **Interface Segregation Principle (ISP)**

**Принцип разделения интерфейсов**:
пользователи не должны зависеть от интерфейсов, которые они не используют.
```go
type Printer interface {
    Print(document string)
}

type Scanner interface {
    Scan(document string)
}

type MultiFunctionDevice interface {
    Printer
    Scanner
}

type SimplePrinter struct {}
func (p *SimplePrinter) Print(document string) {}

type AdvancedPrinter struct {}
func (p *AdvancedPrinter) Print(document string) {}
func (p *AdvancedPrinter) Scan(document string) {}
```
## **Dependency Inversion Principle (DIP)**
**Принцип инверсии зависимостей**:
модули должны зависеть от абстракций, а не от деталей.
```go
type DataStorage interface {
    Save(data string)
}

type FileStorage struct {}
func (fs *FileStorage) Save(data string) {
    fmt.Println("Сохранение в файл:", data)
}

type DataManager struct {
    storage DataStorage
}
func (dm *DataManager) SaveData(data string) {
    dm.storage.Save(data)
}
```
# **DRY**
**Каждый кусочек знаний должен существовать в системе в одном месте.**
Повторяющийся код → источник ошибок.
### **❌ Нарушение DRY**
```go
type User struct {
    Name string
    Age  int
}

func (u User) PrintName() {
    fmt.Println(u.Name)
}

func (u User) PrintAge() {
    fmt.Println(u.Age)
}
```
### **✅ Соблюдение DRY**
```go
func (u User) PrintInfo() {
    fmt.Printf("Name: %s, Age: %d\n", u.Name, u.Age)
}
```