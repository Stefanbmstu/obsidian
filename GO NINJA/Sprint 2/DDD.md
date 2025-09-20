Go — язык, который ценит простоту, но при работе со сложной бизнес-логикой требуется структурированный подход. Domain-Driven Design (DDD) — это именно такой подход, фокусирующийся на ядре приложения — его доменной области. На собеседованиях вопросы про DDD помогают понять, умеет ли кандидат моделировать сложную бизнес-логику и выстраивать коммуникацию с экспертами предметной области. Давайте разберём ключевые идеи DDD и их применение в Go.

### Domain-Driven Design (DDD)

Domain-Driven Design (Проектирование, управляемое предметной областью) — это методология разработки ПО, предложенная Эриком Эвансом, которая ставит во главу угла **домен** (предметную область) приложения. Основная цель — справиться со сложностью путём тесного сотрудничества разработчиков и экспертов предметной области, используя общий язык (**Ubiquitous Language**).

**Ключевые концепции DDD (Тактические паттерны)**:

- **Entities (Сущности)**: Объекты, обладающие уникальной идентичностью, которая сохраняется во времени (например, User, Order). В Go это обычно структуры с полем ID.
- **Value Objects (Объекты-значения)**: Объекты, определяемые своими атрибутами, не имеющие собственной идентичности (например, Address, Money). Часто делаются неизменяемыми (immutable). В Go это структуры, сравниваемые по значению полей.
- **Aggregates (Агрегаты)**: Кластер из одной или нескольких сущностей и объектов-значений, который рассматривается как единое целое. У агрегата есть **корень** (Aggregate Root) — сущность, через которую происходит всё взаимодействие с агрегатом. Это обеспечивает целостность данных внутри агрегата (например, Order с его OrderItems).
- **Repositories (Репозитории)**: Абстракция для доступа к данным агрегатов, имитирующая коллекцию объектов в памяти. Скрывает детали хранения (БД, файлы и т.д.). В Go это интерфейсы.
- **Domain Services (Доменные Сервисы)**: Логика, которая не принадлежит естественным образом ни одной сущности или объекту-значению. В Go это могут быть функции или методы структур-сервисов.
- **Domain Events (События Домена)**: Отражают значимые события, произошедшие в домене (например, OrderPlaced, UserRegistered).

**Пример в Go**:

Представим систему управления заказами.

```go
import "errors"

// Value Object: Money представляет сумму денег (неизменяемый)
type Money struct {
	Amount   int
	Currency string
}

func NewMoney(amount int, currency string) Money {
	// Здесь могут быть проверки
	return Money{Amount: amount, Currency: currency}
}

// Entity: OrderItem представляет позицию в заказе
type OrderItem struct {
	ProductID string
	Price     Money
	Quantity  int
}

// Aggregate Root: Order представляет заказ
type Order struct {
	ID         string
	CustomerID string
	Items      []OrderItem
	TotalPrice Money
	// ... другие поля статуса и т.д.
}

// Фабричный метод для создания нового заказа
func NewOrder(id, customerID string) *Order {
	return &Order{
		ID:         id,
		CustomerID: customerID,
		Items:      make([]OrderItem, 0),
		TotalPrice: NewMoney(0, "USD"), // Пример валюты по умолчанию
	}
}

// Метод на агрегате для добавления позиции (инкапсулирует логику)
func (o *Order) AddItem(productID string, price Money, quantity int) error {
	if quantity <= 0 {
		return errors.New("quantity must be positive")
	}
	if price.Currency != o.TotalPrice.Currency && o.TotalPrice.Amount != 0 {
        return errors.New("cannot mix currencies in an order")
    }
    if o.TotalPrice.Amount == 0 { // Если первая позиция, устанавливаем валюту заказа
        o.TotalPrice.Currency = price.Currency
    }

	item := OrderItem{
		ProductID: productID,
		Price:     price,
		Quantity:  quantity,
	}
	o.Items = append(o.Items, item)
	o.recalculateTotal() // Обновляем общую стоимость
	// Здесь можно генерировать Domain Event: OrderItemAdded
	return nil
}

func (o *Order) recalculateTotal() {
	total := 0
	for _, item := range o.Items {
		total += item.Price.Amount * item.Quantity
	}
	o.TotalPrice.Amount = total
}

package ports // Или infrastructure/repository

import "domain"

// Repository: интерфейс для работы с хранилищем заказов
type OrderRepository interface {
	Save(order *domain.Order) error
	FindByID(id string) (*domain.Order, error)
}

package application

import (
    "domain"
    "ports"
)

// Application Service: Оркестрирует использование доменных объектов
type OrderService struct {
    repo ports.OrderRepository
    // Здесь могут быть другие зависимости, например, для генерации ID
}

func NewOrderService(repo ports.OrderRepository) *OrderService {
    return &OrderService{repo: repo}
}

// Пример Use Case: Создать новый заказ
func (s *OrderService) CreateNewOrder(customerID string) (*domain.Order, error) {
    orderID := "some_generated_id" // В реальности генерация ID
    order := domain.NewOrder(orderID, customerID)
    err := s.repo.Save(order)
    if err != nil {
        return nil, err
    }
    // Здесь можно публиковать Domain Event: OrderCreated
    return order, nil
}

// Пример Use Case: Добавить товар в заказ
func (s *OrderService) AddItemToOrder(orderID, productID string, amount int, currency string, quantity int) error {
	order, err := s.repo.FindByID(orderID)
	if err != nil {
		return err // Заказ не найден
	}

	price := domain.NewMoney(amount, currency)
	err = order.AddItem(productID, price, quantity)
	if err != nil {
		return err // Ошибка бизнес-логики
	}

	// Сохраняем измененный агрегат
	return s.repo.Save(order)
}

package main // Или infrastructure/persistence

import (
	"fmt"
	"domain"
    "ports"
    "application"
)

// Пример реализации репозитория (InMemory)
type InMemoryOrderRepo struct {
	orders map[string]*domain.Order
}

func NewInMemoryOrderRepo() *InMemoryOrderRepo {
	return &InMemoryOrderRepo{orders: make(map[string]*domain.Order)}
}

func (r *InMemoryOrderRepo) Save(order *domain.Order) error {
	r.orders[order.ID] = order // Клонирование может быть полезно для избежания мутаций
	fmt.Printf("Order %s saved\\n", order.ID)
	return nil
}

func (r *InMemoryOrderRepo) FindByID(id string) (*domain.Order, error) {
	if order, exists := r.orders[id]; exists {
		return order, nil // Возвращаем копию для защиты? Зависит от стратегии.
	}
	return nil, fmt.Errorf("order with id %s not found", id)
}

func main() {
	repo := NewInMemoryOrderRepo()
	orderService := application.NewOrderService(repo)

	// Используем сервис для создания заказа
	order, err := orderService.CreateNewOrder("customer-123")
	if err != nil {
		fmt.Println("Error creating order:", err)
		return
	}
	fmt.Printf("Created order: %s for customer %s\\n", order.ID, order.CustomerID)

	// Используем сервис для добавления товара
	err = orderService.AddItemToOrder(order.ID, "product-abc", 1000, "USD", 2) // 2 штуки по $10.00
    if err != nil {
        fmt.Println("Error adding item:", err)
        return
    }

	err = orderService.AddItemToOrder(order.ID, "product-xyz", 550, "USD", 1) // 1 штука по $5.50
    if err != nil {
        fmt.Println("Error adding item:", err)
        return
    }

	// Получаем заказ снова, чтобы увидеть изменения
	updatedOrder, _ := repo.FindByID(order.ID)
	fmt.Printf("Updated order %s total price: %d %s\\n", updatedOrder.ID, updatedOrder.TotalPrice.Amount, updatedOrder.TotalPrice.Currency)
    fmt.Printf("Items in order: %d\\n", len(updatedOrder.Items))
}
```

**Почему это полезно для собеседований**:

- Показывает, что вы можете **моделировать сложную бизнес-логику** и выделять ядро системы.
- Демонстрирует понимание **Ubiquitous Language** и важность коммуникации с бизнесом.
- Помогает объяснить, как **защитить бизнес-правила** внутри агрегатов.
- Позволяет структурировать код так, чтобы он **отражал предметную область**, делая его более понятным и поддерживаемым.
- Подчеркивает умение работать с **абстракциями** (репозитории, доменные сервисы).
- Легко объяснить: "Я использую DDD, чтобы изолировать и защитить сложную бизнес-логику, сделав код ближе к реальным бизнес-процессам".