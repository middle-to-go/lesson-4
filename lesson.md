









[TOC]









---

















📽️ - посмотри воркшоу

⚗️ - проведи эксперимент

🔬 - изучи внимательно

📖 - прочитай документация

🪙 - подумай о сложности

🐞 - запомни ошибку

🔨 - запомни решение

🏔️ - обойди камень предкновенья

⏰ - сделай перерыв

🏡 - попробуй дома

💡 - обсуди светлые идеи

🙋 - задай вопрос

⚡ - запомни панику











---



















## Интерфейсы



























---

### Определение

> An **interface type** is defined as a set of method signatures
>
> A **value** of interface type can hold any value that implements those methods

```go
type Stringer interface {
  String() string
}

func (u *User) String() string {
  if u == nil {
  	return u.Name
  }
}

type User struct {
	Name string
}

func foo(s Stringer) {
  fmt.Println(s.String())
}

func main(){
  user := &User{"Anna"}
  foo(user)
}
```

Количество методов:

- один - суффиксы - `er`,` or`
- больше одного



...

---

### Что под капотом

```go
 stringer                                    user
+--------+        +--------------+         +-----+
| data   |------->| Name: "Rita" |<--------| ptr |
+--------+        +--------------+         +-----+
| itable |
+--------+
    |
    |                  +---------+         +--------+
    +----------------->| type    |-------->| *User  |
                       +---------+         +--------+
                       | ...     |
                       +---------+         +--------------+
                       | func[0] |-------->| *User.String |
                       +---------+         +--------------+


func main() {
	user := &User{"Rita"}
	var stringer Stringer
	stringer = user
	user.Name = "Masha"
	fmt.Println(stringer)
}
```

- Статический тип - `Stringer`

- Динамический тип - `User`

- Значение динамичегоского типа - `user`

- Значение типа интерфейса - `stringer`

  

- [ ] Попробуй данный пример для случая `user := User{"Rita"}` 🏡

---

### Равенство **nil**

```go
 stringer                                    user
+--------+                                 +-----+
| nil    |                                 | nil |
+--------+                                 +-----+
| nil    |
+--------+


func main() {
	var user *User
	var stringer Stringer
  if stringer == nil {
    fmt.Println("stringer is nil")
  }
}
```

- Статический тип - `Stringer`
- Динамический тип - `nil`
- Значение динамичегоского типа - `nil`
- Значение типа интерфейса - `nil`























---

### Равенство **nil**

```go
 stringer                                    user
+--------+                                 +-----+
| nil    |                                 | nil |
+--------+                                 +-----+
| itable |
+--------+
    |
    |                  +---------+         +--------+
    +----------------->| type    |-------->| *User  |
                       +---------+         +--------+
                       | ...     |
                       +---------+         +--------------+
                       | func[0] |-------->| *User.String |
                       +---------+         +--------------+

func main() {
	var user *User // nil
	var stringer Stringer
	stringer = user
  if stringer != nil {
    fmt.Println("stringer isn't nil")
  }
}
```

- Статический тип - `Stringer`
- Динамический тип - `User`
- Значение динамичегоского типа - `nil`
- Значение типа интерфейса - `stringer`

> interface type value is **nil** if both dynamic value and dynamic type are **nil**

---

### Наследование

```go
type Reader interface {
  Read(b []byte) (n int, err error)
}

type Closer interface {
  Сlose()
}

type ReadCloser interface {
  Reader
  Closer
}
```

Дополнительные методы



Пересечение методов

- Наименования и аргументы совпадают
- Совпадают только наименования
- Полностью совпадают

Наследование уже существующего - пример для стандартной библиотеки



```go
import "fmt"

type Bar interface {
  fmt.Stringer
}
```

---

### Конструирование

Добавим интерфейс **Repo** для работы с хранилищем данных

```go
package repo

type Repo interface {
	AddTask(task models.Task) (uint64, error)
}
```

Добавим интерфейс **Flusher** для сброса задач в хранилище данных

```go
package flusher

type Flusher interface {
	Flush(tasks []models.Task) []models.Task
}

type flusher struct {
  repo repo.Repo
}

func New(repo repo.Repo) Flusher {
  return &flusher{
    repo: repo
  }
}
```

**New** - для конструирования с защитой от пустого динамического значения

Имя структуры не экспортируется для инкапсуляции

При необходимости для инициализации добавляются методы:

- Init
- Close

> Стоит ли делать проверку на **nil** для **New**, коротрые возвращают интерфейс  ? 💡

---

### Стандартная библиотека

```go
type error interface {
    Error() string
}

type Stringer interface {
    String() string
}
type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}

type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}

//...
```

Список интерфейсов из стандартной библиотеки:

- `error`
- `fmt.Stringer`
- `io.Reader`, `io.Writer`, `io.Closer`, `io.ReadWriteCloser`
- `http.Handler`, `http.RoundTripper`
- `sort.Interface`
- `heap.Interface`
- `net.Addr`, `net.Conn`, `net.Listener`, `net.Error`

- [ ] Попробуй каждый интерфейс в свободное время 🏡

---

### Пустой интерфейс

```go
interface{}

func foo(a interface{})

t := i.(T) // ⚡ If i does not hold a T, the statement will trigger a panic.
t, ok := i.(T) // If i does not hold a T, the statement will not trigger a panic.
```

```go
// object -> map[string]interface{} -> object

type ContentType uint

const (
  TypeDocument ContentType = iota
  TypeVideo
  TypeQuestion
  TypeTask
)

type SlideInfo struct {
  Type    ContentType
	Content map[string]interface{}
}

type VideoContent struct {
  Link string
}

type QuestionContent struct {
  Author string
}
```

---

```sh
➜ go get github.com/mitchellh/mapstructure
```

```go
switch slide.Type {
  case TypeVideo:
  	var content VideoContent
		if err := Decode(slide.Content, &content); err != nil {
			return err
		}
  	fmt.Println(content.Link)
  //...
}
```

### Switch

```go
package main

import "fmt"

type StatusType uint

const (
	StatusRunning StatusType = iota
	StatusStopped
)

// 1 -> Running <- "running"
// 0 -> Stopped <- "stopped"
func ...(status interface{}) (StatusType, error) {
	switch v := status.(type) {
	case int:
		//...
	case string:
		//...
	default:
    return newErrInvalidStatus(status)
	}
}
```

> Пример, когда приходит либо строка, либо числовой идентификатор

---





















## ⏰

























---



















## Пример





























---

Пакет для работы с хранилищем

```go
package repo

type Repo interface {
  AddTasks(task []models.Task) error
  RemoveTask(taskId uint64) error
  DescribeTask(taskId uint64) (*models.Task, error)
 	ListTasks(limit, offset uint64) ([]models.Task, error)
}
```

Пакет для отправкии данных в хранилище

```go
package flusher

type Flusher interface {
	Flush(tasks []models.Task) []models.Task
}
```

Пакет для публикации метрик

```go
package metrics

type Publisher interface {
	PublishFlushing(count int)
}
```

internal

- flusher
- metrics
- publisher



Имплементация **internal/flusher/flusher.go**



---



















## Тестирование





























---

### Testing

```go
// where Xxx does not start with a lowercase letter. 
// The function name serves to identify the test routine
func TestXxx(*testing.T)

func BenchmarkXxx(*testing.B)

func ExampleXxx()
```

```go
func TestAbs(t *testing.T) {
    got := Abs(-1)
    if got != 1 {
        t.Errorf("Abs(-1) = %d; want 1", got)
    }
}
```

```sh
➜ go test [-s] [-v]
➜ go test -run ''      # Run all tests.
➜ go test -run Foo/A=  # For top-level tests matching "Foo", run subtests matching "A="
```

```
-cpu, -list,
-parallel, -run, -short, and -v
```

`t.Error*` will report test failures but continue executing the test. 

`t.Fatal*` will report test failures and stop the test immediately.

> Test files that declare a package with the suffix **"_test"** will be compiled as a
> separate package, and then linked and run with the main test binary.

---

### Методы

Общие методы для **testing.T** и **testing.B** представлены в виде интерфейса **TB**:

```go
type TB interface {
    Cleanup(func())
    Error(args ...interface{})
    Errorf(format string, args ...interface{})
    Fail()
    FailNow()
    Failed() bool
    Fatal(args ...interface{})
    Fatalf(format string, args ...interface{})
    Helper()
    Log(args ...interface{})
    Logf(format string, args ...interface{})
    Name() string
    Skip(args ...interface{})
    SkipNow()
    Skipf(format string, args ...interface{})
    Skipped() bool
    TempDir() string
}
```

Специализированные методы для **testing.T**:

- Deadline
- Run / Parallel

Специализированные методы для **testing.B**:

- StartTimer / StopTimer / ResetTimer
- SetBytes
- ReportMetric / ReportAllocs

---

### Testify

```sh
➜ go get github.com/stretchr/testify
```

```go
import (
  "testing"
  "github.com/stretchr/testify/assert"
)

func TestSomething(t *testing.T) {
  // assert equality
  assert.Equal(t, 123, 123, "they should be equal")
  // assert inequality
  assert.NotEqual(t, 123, 456, "they should not be equal")
  // assert for nil (good for errors)
  assert.Nil(t, object)
  // assert for not nil (good when you expect something)
  if assert.NotNil(t, object) {
    // now we know that object isn't nil, we are safe to make
    // further assertions without causing any errors
    assert.Equal(t, "Something", object.Value)
  }
}
```

[`assert`](http://godoc.org/github.com/stretchr/testify/assert) package

- Prints friendly, easy to read failure descriptions
- Allows for very readable code
- Optionally annotate each assertion with a message

[`mock`](http://godoc.org/github.com/stretchr/testify/mock) package vs **mockgen**

[`suite`](http://godoc.org/github.com/stretchr/testify/suite) package **...**

[`require`](http://godoc.org/github.com/stretchr/testify/require) package

> The `require` package provides same global functions as the `assert` package, but instead of returning a boolean result they terminate current test.

---

### Ginkgo

```sh
➜ go get github.com/onsi/ginkgo/ginkgo # установка ginkgo
```

```shell
➜ cd internal/flusher 
➜ ginkgo bootstrap 				 # создаем flusher_suite_test.go
➜ ginkgo generate flusher  # генерируем flusher_test.go
➜ ginkgo 									 # запускаем тестовые сценарии
```

Содержимое файла **flusher_suite_test.go**:

```go
package flusher_test

import (
	"testing"
	. "github.com/onsi/ginkgo"
)

func TestFlusher(t *testing.T) {
	RegisterFailHandler(Fail)
	RunSpecs(t, "Flusher Suite")
}
```

Содержимое файла **flusher_test.go**:

```go
package flusher_test

import (
	. "github.com/onsi/ginkgo"
	"github.com/ozoncp/ocp-task-api/internal/flusher"
)

var _ = Describe("Flusher", func() {
  
})
```

---

```go
BeforeEach(func() {
	// ...  
})

JustBeforeEach(func(){
  // ...  
})

AfterEach(func(){
})

Context("", func() {
  BeforeEach(func() { 
    // ...
  })
  
  JustBeforeEach(func(){
    // ...
	})
  
  It("", func() {
    // ...
  ))
})
```

1. `BeforeEach`
2. `JustBeforeEach`
3. `JustAfterEach`
4. `AfterEach`



1. `BeforeSuite`
2. `AfterSuite`

---

### Порядок вызовов

```go
BeforeEach(func() {
	// ...1️⃣
})

JustBeforeEach(func(){
  // ...3️⃣
})

AfterEach(func(){
  // ...6️⃣
})

Context("case", func() {
  BeforeEach(func() { 
    // ...2️⃣
  })
  
  JustBeforeEach(func(){
    // ...4️⃣
	})
  
  It("do", func() {
    // ...5️⃣
  ))
})
```

```go
var _ = BeforeSuite(func() {
  // ...0️⃣
}
                    
var _ = AfterSuite(func() {
  // ...7️⃣
})
```

---

### Gomega

```sh
➜ go get github.com/onsi/gomega/...
```

Установка только пакетов необходимых для использования **gomega**

```go
import "github.com/onsi/gomega"

Expect(5.1).Should(BeEquivalentTo(5))
Expect(5).ShouldNot(BeEquivalentTo(5.1))
```

|                                                              |                       |
| ------------------------------------------------------------ | --------------------- |
| Should / To, ShouldNot / ToNot                               |                       |
| Equal, BeEquivalentTo, BeIdenticalTo, BeAssignableToTypeOf   | Equivalence           |
| BeNil, BeZero, BeTrue, BeFalse                               | Presence / Truthiness |
| HaveOccurred, Succeed, MatchError                            | Error                 |
| BeEmpty, HaveLen, HaveCap, ContainElement / ContainElements, ConsistOf, HaveKey, HaveKeyWithValue | Collection            |
| Panic, PanicWith                                             | Panic                 |
| ContainSubstring, HavePrefix, HaveSuffix, MatchRegexp, MatchJSON, MatchYAML | String, Yaml, Json    |
| And / SatisfyAll, Or / SatisfyAny, WithTransform             | Composing Matchers    |
| ...                                                          | ...                   |

- Добавление анотаций
- Ассинхронные утверждения `Eventually`, `TIMEOUT`, `POLLING_INTERVAL`
- Пользовательские матчеры 
- Тестирование HTTP клиентов - **ghttp**
- Тестирование внешних процессов - **gexec**
- Упрощает тестирование больших и вложенных структур и срезов - **gstruct**

> Note that `Should` and `To` are just syntactic sugar and are functionally identical. Same is the case for `ToNot` and `NotTo`

---

### Mockgen

```sh
➜ go install github.com/golang/mock/mockgen@v1.5.0 # установка
```

Добавляем в **internal/mockgen.go** список интерфесов

```go
package internal

//go:generate mockgen -destination=./mocks/flusher_mock.go -package=mocks github.com/ozoncp/ocp-task-api/internal/flusher Flusher

//...github.com/ozoncp/ocp-task-api/internal/repo Repo

//...github.com/ozoncp/ocp-task-api/internal/metrics Publisher
```

```sh
➜ mkdir internal/mocks
➜ go genereate ./...
```

```shell
➜ tree internal/mocks
```

Возможности:

Возможности:

- Переопределить имя мока
- Указать выходное имя файла
- Переопределить пакет
- Генерировать моки не только для локальных сущностей
- Поддержка двух режимов: **source mode** и **reflect mode**
- Поддержка dry run флагов и возможности добавления **Copyright** заголовка

---

### Пример

```go
import (
	"github.com/ozoncp/ocp-task-api/internal/mocks"
)

var _ = Describe("Flusher", func() {

	var (
		ctrl *gomock.Controller
    
    mockRepo      *mocks.MockRepo
		mockPublisher *mocks.MockPublisher
  )
  
  BeforeEach(func() {
		ctrl = gomock.NewController(GinkgoT())

		mockRepo = mocks.NewMockRepo(ctrl)
		mockPublisher = mocks.NewMockPublisher(ctrl)
	})
  
  AfterEach(func() {
		ctrl.Finish()
	})
  
	Context("", func() {
		
    BeforeEach(func() {
		})

		It("", func() {
		})
	})
})

```

```go
mockRepo.EXPECT().
	AddTasks(gomock.Any(), gomock.Any()).
	Return(nil)

mockPublisher.EXPECT().
	PublishFlushing(gomock.Any())
```

```go
func (c *Call) After(preReq *Call) *Call
func (c *Call) AnyTimes() *Call
func (c *Call) Do(f interface{}) *Call
func (c *Call) DoAndReturn(f interface{}) *Call
func (c *Call) MaxTimes(n int) *Call
func (c *Call) MinTimes(n int) *Call
func (c *Call) Return(rets ...interface{}) *Call
func (c *Call) SetArg(n int, value interface{}) *Call
func (c *Call) String() string
func (c *Call) Times(n int) *Call
```
