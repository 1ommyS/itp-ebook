# Готовые решения задач Go

Здесь собраны эталонные реализации 30 обычных задач. Итоговый Order Service намеренно не решён. Каждый блок показывает ядро решения; имена module/package при переносе подстройте под свой репозиторий.

## 1. Модуль конвертации температур { #go-01 }

```go
// temperature/temperature.go
package temperature

import "fmt"

func CelsiusToFahrenheit(c float64) float64 { return c*9/5 + 32 }
func FahrenheitToCelsius(f float64) float64 { return (f - 32) * 5 / 9 }

func ExampleCelsiusToFahrenheit() {
    fmt.Printf("%.1f\n", CelsiusToFahrenheit(100))
    // Output: 212.0
}
```

```go
// cmd/temperature/main.go
package main

import (
    "flag"
    "fmt"
    "log"
    "example.com/temperature/temperature"
)

func main() {
    from := flag.String("from", "c", "c or f")
    value := flag.Float64("value", 0, "temperature")
    flag.Parse()
    switch *from {
    case "c": fmt.Println(temperature.CelsiusToFahrenheit(*value))
    case "f": fmt.Println(temperature.FahrenheitToCelsius(*value))
    default: log.Fatal("from must be c or f")
    }
}
```

## 2. Анализатор текста { #go-02 }

```go
package textstat

import (
    "sort"
    "strings"
    "unicode"
)

type Frequency struct { Rune rune; Count int }
type Stats struct { Runes, Words int; Frequency []Frequency }

func Analyze(s string) Stats {
    counts := map[rune]int{}
    runes := 0
    for _, r := range s {
        runes++
        if !unicode.IsSpace(r) { counts[unicode.ToLower(r)]++ }
    }
    frequencies := make([]Frequency, 0, len(counts))
    for r, count := range counts { frequencies = append(frequencies, Frequency{r, count}) }
    sort.Slice(frequencies, func(i, j int) bool {
        if frequencies[i].Count != frequencies[j].Count { return frequencies[i].Count > frequencies[j].Count }
        return frequencies[i].Rune < frequencies[j].Rune
    })
    return Stats{Runes: runes, Words: len(strings.Fields(s)), Frequency: frequencies}
}
```

## 3. Владение slices { #go-03 }

```go
package slicesx

import "sort"

func AppendUnique(values []int, value int) []int {
    for _, current := range values { if current == value { return values } }
    return append(values, value)
}

func CloneAndSort(values []int) []int {
    clone := append([]int(nil), values...)
    sort.Ints(clone)
    return clone
}

func Window(values []int, from, to int) []int {
    if from < 0 || to < from || to > len(values) { return nil }
    return append([]int(nil), values[from:to]...)
}
```

## 4. LRU cache { #go-04 }

```go
package lru

import (
    "container/list"
    "sync"
)

type entry[K comparable, V any] struct { key K; value V }
type Cache[K comparable, V any] struct {
    mu sync.Mutex
    capacity int
    order *list.List
    items map[K]*list.Element
}

func New[K comparable, V any](capacity int) *Cache[K, V] {
    if capacity < 0 { capacity = 0 }
    return &Cache[K,V]{capacity: capacity, order: list.New(), items: make(map[K]*list.Element)}
}
func (c *Cache[K,V]) Get(key K) (V, bool) {
    c.mu.Lock(); defer c.mu.Unlock()
    element, ok := c.items[key]
    if !ok { var zero V; return zero, false }
    c.order.MoveToFront(element)
    return element.Value.(entry[K,V]).value, true
}
func (c *Cache[K,V]) Put(key K, value V) {
    c.mu.Lock(); defer c.mu.Unlock()
    if c.capacity == 0 { return }
    if element, ok := c.items[key]; ok {
        element.Value = entry[K,V]{key, value}; c.order.MoveToFront(element); return
    }
    c.items[key] = c.order.PushFront(entry[K,V]{key, value})
    if c.order.Len() > c.capacity {
        last := c.order.Back(); c.order.Remove(last); delete(c.items, last.Value.(entry[K,V]).key)
    }
}
```

## 5. Domain types { #go-05 }

```go
package order

import (
    "errors"
    "regexp"
)

type ID string
type Money struct { Cents int64; Currency string }
type Status string
const (Created Status = "created"; Paid Status = "paid"; Cancelled Status = "cancelled")
type Order struct { ID ID; Total Money; Status Status }

var idPattern = regexp.MustCompile(`^ORD-[0-9]{8}$`)
func New(id string, cents int64, currency string) (Order, error) {
    if !idPattern.MatchString(id) { return Order{}, errors.New("invalid id") }
    if cents < 0 { return Order{}, errors.New("negative total") }
    if currency == "" { return Order{}, errors.New("currency required") }
    return Order{ID: ID(id), Total: Money{cents, currency}, Status: Created}, nil
}
```

## 6. Маленькие interfaces { #go-06 }

```go
package order

import "context"

type Reader interface { ByID(context.Context, ID) (Order, error) }
type EventSender interface { Send(context.Context, string, Order) error }
type Service struct { reader Reader; sender EventSender }

func NewService(reader Reader, sender EventSender) *Service { return &Service{reader, sender} }
func (s *Service) Publish(ctx context.Context, id ID) error {
    order, err := s.reader.ByID(ctx, id)
    if err != nil { return err }
    return s.sender.Send(ctx, "order.loaded", order)
}
```

## 7. Цепочка ошибок { #go-07 }

```go
package order

import (
    "errors"
    "fmt"
)

var ErrNotFound = errors.New("order not found")
type ValidationError struct { Field, Message string }
func (e *ValidationError) Error() string { return e.Field + ": " + e.Message }

func load(id string) error {
    if id == "" { return &ValidationError{Field: "id", Message: "required"} }
    return fmt.Errorf("load %q: %w", id, ErrNotFound)
}
```

## 8. Generic Set { #go-08 }

```go
package set

type Set[T comparable] map[T]struct{}
func New[T comparable](values ...T) Set[T] {
    result := make(Set[T], len(values)); for _, value := range values { result[value] = struct{}{} }; return result
}
func (s Set[T]) Has(value T) bool { _, ok := s[value]; return ok }
func (s Set[T]) Union(other Set[T]) Set[T] {
    result := New[T](); for value := range s { result[value] = struct{}{} }; for value := range other { result[value] = struct{}{} }; return result
}
func (s Set[T]) Intersection(other Set[T]) Set[T] {
    result := New[T](); for value := range s { if other.Has(value) { result[value] = struct{}{} } }; return result
}
```

## 9. Ограниченный ParallelMap { #go-09 }

```go
package parallel

import (
    "context"
    "sync"
)

func Map[T, R any](ctx context.Context, values []T, limit int, fn func(context.Context, T) (R, error)) ([]R, error) {
    if limit < 1 { limit = 1 }
    ctx, cancel := context.WithCancel(ctx); defer cancel()
    out := make([]R, len(values)); jobs := make(chan int); var wg sync.WaitGroup
    var first error; var once sync.Once
    worker := func() { defer wg.Done(); for index := range jobs {
        value, err := fn(ctx, values[index]); if err != nil { once.Do(func(){ first = err; cancel() }); continue }; out[index] = value
    }}
    wg.Add(limit); for i := 0; i < limit; i++ { go worker() }
sendLoop:
    for index := range values { select { case jobs <- index: case <-ctx.Done(): break sendLoop } }
    close(jobs); wg.Wait()
    if first == nil && ctx.Err() != nil { first = ctx.Err() }
    return out, first
}
```

## 10. Fan-out/fan-in checksum { #go-10 }

```go
package checksum

import (
    "context"
    "crypto/sha256"
    "os"
    "sync"
)

type Result struct { Path string; Sum [32]byte; Err error }
func Files(ctx context.Context, paths []string, workers int) <-chan Result {
    jobs := make(chan string); results := make(chan Result); var wg sync.WaitGroup
    worker := func() { defer wg.Done(); for path := range jobs {
        data, err := os.ReadFile(path); result := Result{Path:path, Err:err}; if err == nil { result.Sum = sha256.Sum256(data) }
        select { case results <- result: case <-ctx.Done(): return }
    }}
    wg.Add(workers); for i:=0; i<workers; i++ { go worker() }
    go func(){ defer close(jobs); for _, path := range paths { select { case jobs <- path: case <-ctx.Done(): return } } }()
    go func(){ wg.Wait(); close(results) }()
    return results
}
```

## 11. Отменяемый worker pool { #go-11 }

```go
package worker

import (
    "context"
    "sync"
)

func Run[T,R any](ctx context.Context, count int, jobs <-chan T, fn func(context.Context,T) R) <-chan R {
    out := make(chan R); var wg sync.WaitGroup
    worker := func(){ defer wg.Done(); for { select {
    case <-ctx.Done(): return
    case job, ok := <-jobs: if !ok { return }; result := fn(ctx,job); select { case out <- result: case <-ctx.Done(): return }
    }}}
    wg.Add(count); for i:=0;i<count;i++ { go worker() }
    go func(){ wg.Wait(); close(out) }()
    return out
}
```

## 12. Token bucket { #go-12 }

```go
package limiter

import (
    "sync"
    "time"
)

type Bucket struct { mu sync.Mutex; capacity, tokens int; every time.Duration; last time.Time }
func New(capacity int, every time.Duration, now time.Time) *Bucket { return &Bucket{capacity:capacity,tokens:capacity,every:every,last:now} }
func (b *Bucket) Allow(now time.Time) bool {
    b.mu.Lock(); defer b.mu.Unlock()
    refill := int(now.Sub(b.last)/b.every); if refill > 0 { b.tokens = min(b.capacity,b.tokens+refill); b.last = b.last.Add(time.Duration(refill)*b.every) }
    if b.tokens == 0 { return false }; b.tokens--; return true
}
```

## 13. Single-flight loader { #go-13 }

```go
package loader

import "sync"

type call[V any] struct { done chan struct{}; value V; err error }
type Loader[K comparable,V any] struct { mu sync.Mutex; calls map[K]*call[V] }
func New[K comparable,V any]() *Loader[K,V] { return &Loader[K,V]{calls:make(map[K]*call[V])} }
func (l *Loader[K,V]) Do(key K, fn func()(V,error)) (V,error) {
    l.mu.Lock(); if existing := l.calls[key]; existing != nil { l.mu.Unlock(); <-existing.done; return existing.value,existing.err }
    current := &call[V]{done:make(chan struct{})}; l.calls[key]=current; l.mu.Unlock()
    current.value,current.err=fn(); close(current.done)
    l.mu.Lock(); delete(l.calls,key); l.mu.Unlock(); return current.value,current.err
}
```

## 14. Concurrent ledger { #go-14 }

```go
package ledger

import (
    "errors"
    "sync"
)

type Ledger struct { mu sync.Mutex; balances map[string]int64 }
func New(values map[string]int64) *Ledger { clone:=map[string]int64{}; for k,v:=range values { clone[k]=v }; return &Ledger{balances:clone} }
func (l *Ledger) Transfer(from,to string,amount int64) error {
    l.mu.Lock(); defer l.mu.Unlock()
    if amount<=0 { return errors.New("amount must be positive") }; if l.balances[from]<amount { return errors.New("insufficient funds") }
    l.balances[from]-=amount; l.balances[to]+=amount; return nil
}
func (l *Ledger) Total() int64 { l.mu.Lock(); defer l.mu.Unlock(); var total int64; for _,v:=range l.balances { total+=v }; return total }
```

## 15. Устранение goroutine leak { #go-15 }

```go
package first

import "context"

func First[T any](parent context.Context, producers ...func(context.Context) (T,error)) (T,error) {
    ctx,cancel:=context.WithCancel(parent); defer cancel()
    type result struct{ value T; err error }; out:=make(chan result,1)
    for _,producer:=range producers { go func(fn func(context.Context)(T,error)){
        value,err:=fn(ctx); select { case out<-result{value,err}: case <-ctx.Done(): }
    }(producer) }
    select { case result:=<-out: return result.value,result.err; case <-ctx.Done(): var zero T; return zero,ctx.Err() }
}
```

## 16. Graceful background job { #go-16 }

```go
package background

import (
    "context"
    "sync"
    "time"
)

type Runner struct { mu sync.RWMutex; lastErr error }
func (r *Runner) Run(ctx context.Context, every time.Duration, job func(context.Context) error) {
    ticker:=time.NewTicker(every); defer ticker.Stop()
    for { select {
    case <-ctx.Done(): return
    case <-ticker.C: err:=job(ctx); r.mu.Lock(); r.lastErr=err; r.mu.Unlock()
    }}
}
func (r *Runner) LastError() error { r.mu.RLock(); defer r.mu.RUnlock(); return r.lastErr }
```

## 17. Strict JSON endpoint { #go-17 }

```go
package api

import (
    "encoding/json"
    "errors"
    "io"
    "net/http"
    "strings"
)

type createOrderRequest struct { CustomerID string `json:"customerId"`; Amount int64 `json:"amount"` }
func decodeOne(w http.ResponseWriter,r *http.Request,dst any) error {
    r.Body=http.MaxBytesReader(w,r.Body,1<<20); decoder:=json.NewDecoder(r.Body); decoder.DisallowUnknownFields()
    if err:=decoder.Decode(dst); err!=nil { return err }
    if err:=decoder.Decode(&struct{}{}); !errors.Is(err,io.EOF) { return errors.New("body must contain one JSON object") }
    return nil
}
func CreateOrder(w http.ResponseWriter,r *http.Request) {
    var request createOrderRequest
    if err:=decodeOne(w,r,&request); err!=nil { http.Error(w,"invalid JSON",http.StatusBadRequest); return }
    if strings.TrimSpace(request.CustomerID)=="" || request.Amount<=0 { http.Error(w,"validation failed",http.StatusUnprocessableEntity); return }
    w.Header().Set("Content-Type","application/json"); w.WriteHeader(http.StatusCreated); _=json.NewEncoder(w).Encode(map[string]any{"status":"created"})
}
```

## 18. Middleware chain { #go-18 }

```go
package middleware

import (
    "context"
    "log/slog"
    "net/http"
    "runtime/debug"
    "time"
)

type key struct{}
func Chain(next http.Handler) http.Handler {
    next = recoverer(next); next = accessLog(next); next = timeout(3*time.Second,next); return requestID(next)
}
func requestID(next http.Handler) http.Handler { return http.HandlerFunc(func(w http.ResponseWriter,r *http.Request){
    id:=r.Header.Get("X-Request-ID"); if id=="" { id=time.Now().UTC().Format("20060102150405.000000000") }
    next.ServeHTTP(w,r.WithContext(context.WithValue(r.Context(),key{},id)))
})}
func timeout(duration time.Duration,next http.Handler) http.Handler { return http.TimeoutHandler(next,duration,`{"error":"timeout"}`) }
func accessLog(next http.Handler) http.Handler { return http.HandlerFunc(func(w http.ResponseWriter,r *http.Request){ start:=time.Now(); next.ServeHTTP(w,r); slog.Info("request", "method",r.Method,"path",r.URL.Path,"duration",time.Since(start)) }) }
func recoverer(next http.Handler) http.Handler { return http.HandlerFunc(func(w http.ResponseWriter,r *http.Request){ defer func(){ if value:=recover(); value!=nil { slog.Error("panic","value",value,"stack",string(debug.Stack())); http.Error(w,"internal error",500) } }(); next.ServeHTTP(w,r) }) }
```

## 19. Idempotent create { #go-19 }

```go
package idempotency

import (
    "crypto/sha256"
    "errors"
    "sync"
)

var ErrConflict=errors.New("idempotency key reused with another payload")
type Result struct{ Status int; Body []byte }
type record struct{ hash [32]byte; result Result }
type Store struct{ mu sync.Mutex; records map[string]record }
func New() *Store { return &Store{records:map[string]record{}} }
func (s *Store) Create(key string,payload []byte,fn func()Result)(Result,error){
    s.mu.Lock(); defer s.mu.Unlock(); hash:=sha256.Sum256(payload)
    if old,ok:=s.records[key]; ok { if old.hash!=hash { return Result{},ErrConflict }; return old.result,nil }
    result:=fn(); s.records[key]=record{hash,result}; return result,nil
}
```

Для production используйте уникальное ограничение БД и храните состояние `in_progress/completed`, чтобы не держать mutex во время бизнес-операции.

## 20. Подписанный cursor { #go-20 }

```go
package cursor

import (
    "crypto/hmac"
    "crypto/sha256"
    "encoding/base64"
    "encoding/json"
    "errors"
    "time"
)

type Cursor struct{ CreatedAt time.Time `json:"createdAt"`; ID int64 `json:"id"`; ExpiresAt time.Time `json:"expiresAt"` }
func Encode(value Cursor,secret []byte)(string,error){
    payload,err:=json.Marshal(value); if err!=nil{return "",err}; mac:=hmac.New(sha256.New,secret); mac.Write(payload)
    raw:=append(payload,mac.Sum(nil)...); return base64.RawURLEncoding.EncodeToString(raw),nil
}
func Decode(raw string,secret []byte,now time.Time)(Cursor,error){
    decoded,err:=base64.RawURLEncoding.DecodeString(raw); if err!=nil||len(decoded)<sha256.Size{return Cursor{},errors.New("invalid cursor")}
    payload,signature:=decoded[:len(decoded)-sha256.Size],decoded[len(decoded)-sha256.Size:]; mac:=hmac.New(sha256.New,secret); mac.Write(payload)
    if !hmac.Equal(signature,mac.Sum(nil)){return Cursor{},errors.New("invalid signature")}; var value Cursor
    if json.Unmarshal(payload,&value)!=nil||now.After(value.ExpiresAt){return Cursor{},errors.New("expired cursor")}; return value,nil
}
```

## 21. Надёжный HTTP client { #go-21 }

```go
package downstream

import (
    "context"
    "fmt"
    "io"
    "net/http"
    "time"
)

type Client struct{ HTTP *http.Client; BaseURL string }
func (c *Client) Get(ctx context.Context,path string)([]byte,error){
    var last error
    for attempt:=0; attempt<3; attempt++ {
        request,err:=http.NewRequestWithContext(ctx,http.MethodGet,c.BaseURL+path,nil); if err!=nil{return nil,err}
        response,err:=c.HTTP.Do(request)
        if err==nil {
            body,readErr:=io.ReadAll(io.LimitReader(response.Body,2<<20)); response.Body.Close()
            if readErr==nil && response.StatusCode<500 { if response.StatusCode>=400{return nil,fmt.Errorf("downstream status %d",response.StatusCode)}; return body,nil }
            if readErr!=nil { err=readErr } else { err=fmt.Errorf("temporary status %d",response.StatusCode) }
        }
        last=err; select { case <-time.After(time.Duration(1<<attempt)*50*time.Millisecond): case <-ctx.Done(): return nil,ctx.Err() }
    }
    return nil,last
}
```

## 22. Тест graceful shutdown { #go-22 }

```go
package shutdown_test

import (
    "context"
    "net"
    "net/http"
    "sync"
    "testing"
    "time"
)

func TestShutdownWaitsForRequest(t *testing.T){
    started:=make(chan struct{}); release:=make(chan struct{}); var once sync.Once
    server:=&http.Server{Handler:http.HandlerFunc(func(w http.ResponseWriter,r *http.Request){ once.Do(func(){close(started)}); <-release; w.WriteHeader(204) })}
    listener,err:=net.Listen("tcp","127.0.0.1:0"); if err!=nil{t.Fatal(err)}; go server.Serve(listener)
    done:=make(chan error,1); go func(){ response,err:=http.Get("http://"+listener.Addr().String()); if err==nil{response.Body.Close()}; done<-err }()
    <-started; shutdownDone:=make(chan error,1); go func(){ctx,cancel:=context.WithTimeout(context.Background(),time.Second);defer cancel();shutdownDone<-server.Shutdown(ctx)}()
    close(release); if err:=<-done;err!=nil{t.Fatal(err)}; if err:=<-shutdownDone;err!=nil{t.Fatal(err)}
}
```

## 23. SQL repository { #go-23 }

```go
package postgres

import (
    "context"
    "database/sql"
    "errors"
    "fmt"
)

var ErrNotFound=errors.New("order not found")
type Order struct{ ID int64; Status string }
type Repository struct{ db *sql.DB }
func New(db *sql.DB)*Repository{return &Repository{db}}
func (r *Repository) ByID(ctx context.Context,id int64)(Order,error){
    var order Order; err:=r.db.QueryRowContext(ctx,"SELECT id,status FROM orders WHERE id=$1",id).Scan(&order.ID,&order.Status)
    if errors.Is(err,sql.ErrNoRows){return Order{},ErrNotFound}; if err!=nil{return Order{},fmt.Errorf("query order: %w",err)}; return order,nil
}
func (r *Repository) List(ctx context.Context)([]Order,error){
    rows,err:=r.db.QueryContext(ctx,"SELECT id,status FROM orders ORDER BY id"); if err!=nil{return nil,err}; defer rows.Close(); var result []Order
    for rows.Next(){var order Order;if err:=rows.Scan(&order.ID,&order.Status);err!=nil{return nil,err};result=append(result,order)}
    if err:=rows.Err();err!=nil{return nil,err};return result,nil
}
```

## 24. Transactional transfer { #go-24 }

```go
package transfer

import (
    "context"
    "database/sql"
    "errors"
)

func Transfer(ctx context.Context,db *sql.DB,from,to,amount int64)(err error){
    if amount<=0{return errors.New("amount must be positive")}; tx,err:=db.BeginTx(ctx,nil);if err!=nil{return err}
    defer func(){if err!=nil{_ = tx.Rollback()}}(); var balance int64
    if err=tx.QueryRowContext(ctx,"SELECT balance FROM accounts WHERE id=$1 FOR UPDATE",from).Scan(&balance);err!=nil{return err}
    if balance<amount{return errors.New("insufficient funds")}
    if _,err=tx.ExecContext(ctx,"UPDATE accounts SET balance=balance-$1 WHERE id=$2",amount,from);err!=nil{return err}
    result,execErr:=tx.ExecContext(ctx,"UPDATE accounts SET balance=balance+$1 WHERE id=$2",amount,to);if execErr!=nil{return execErr};if count,_:=result.RowsAffected();count!=1{return errors.New("recipient not found")}
    if _,err=tx.ExecContext(ctx,"INSERT INTO transfer_audit(sender,recipient,amount) VALUES($1,$2,$3)",from,to,amount);err!=nil{return err}
    err=tx.Commit();return err
}
```

## 25. Transactional outbox { #go-25 }

```go
package outbox

import (
    "context"
    "database/sql"
    "encoding/json"
)

type Event struct{ ID int64; Topic string; Payload []byte }
func Confirm(ctx context.Context,db *sql.DB,orderID int64) error {
    tx,err:=db.BeginTx(ctx,nil);if err!=nil{return err};defer tx.Rollback()
    if _,err=tx.ExecContext(ctx,"UPDATE orders SET status='confirmed' WHERE id=$1",orderID);err!=nil{return err}
    payload,_:=json.Marshal(map[string]any{"orderId":orderID})
    if _,err=tx.ExecContext(ctx,"INSERT INTO outbox(topic,payload) VALUES($1,$2)","OrderConfirmed",payload);err!=nil{return err};return tx.Commit()
}
type Publisher interface{ Publish(context.Context,int64,string,[]byte) error }
func PublishBatch(ctx context.Context,db *sql.DB,publisher Publisher) error {
    rows,err:=db.QueryContext(ctx,"SELECT id,topic,payload FROM outbox WHERE published_at IS NULL ORDER BY id LIMIT 100");if err!=nil{return err};defer rows.Close()
    for rows.Next(){var event Event;if err:=rows.Scan(&event.ID,&event.Topic,&event.Payload);err!=nil{return err};if err:=publisher.Publish(ctx,event.ID,event.Topic,event.Payload);err!=nil{return err};if _,err:=db.ExecContext(ctx,"UPDATE outbox SET published_at=now() WHERE id=$1 AND published_at IS NULL",event.ID);err!=nil{return err}}
    return rows.Err()
}
```

Повтор между `Publish` и `UPDATE` возможен, поэтому consumer обязан дедуплицировать событие по стабильному event ID.

## 26. Нагрузка на pool { #go-26 }

```go
package poolload

import (
    "context"
    "database/sql"
    "sync"
    "time"
)

type Result struct{ Duration time.Duration; Stats sql.DBStats; Err error }
func Run(ctx context.Context,db *sql.DB,workers,operations int) Result {
    started:=time.Now();jobs:=make(chan struct{});errorsCh:=make(chan error,1);var wg sync.WaitGroup
    worker:=func(){defer wg.Done();for range jobs{var value int;if err:=db.QueryRowContext(ctx,"SELECT 1").Scan(&value);err!=nil{select{case errorsCh<-err:default:};return}}}
    wg.Add(workers);for i:=0;i<workers;i++{go worker()}
feed:
    for i:=0;i<operations;i++{select{case jobs<-struct{}{}:case <-ctx.Done():break feed}}
    close(jobs);wg.Wait();var err error;select{case err=<-errorsCh:default:};if err==nil{err=ctx.Err()}
    return Result{Duration:time.Since(started),Stats:db.Stats(),Err:err}
}
```

## 27. Fuzz parser { #go-27 }

```go
package cursor_test

import (
    "testing"
    "time"
    "example.com/app/cursor"
)

func FuzzDecode(f *testing.F){
    secret:=[]byte("01234567890123456789012345678901");valid,_:=cursor.Encode(cursor.Cursor{ID:1,ExpiresAt:time.Now().Add(time.Hour)},secret)
    f.Add(valid);f.Add("");f.Add("💥")
    f.Fuzz(func(t *testing.T,raw string){ value,err:=cursor.Decode(raw,secret,time.Now());if err==nil&&value.ID<0{t.Fatalf("negative id: %d",value.ID)} })
}
```

## 28. Benchmark аллокаций { #go-28 }

```go
package encode_test

import (
    "bytes"
    "encoding/json"
    "strconv"
    "testing"
)

type Item struct{ ID int `json:"id"` }
func BenchmarkJSON(b *testing.B){item:=Item{42};b.ReportAllocs();for i:=0;i<b.N;i++{data,_:=json.Marshal(item);_ = data}}
func BenchmarkManual(b *testing.B){b.ReportAllocs();for i:=0;i<b.N;i++{var out bytes.Buffer;out.WriteString(`{"id":`);out.WriteString(strconv.Itoa(42));out.WriteByte('}');_ = out.Bytes()}}
```

## 29. Наблюдаемость { #go-29 }

```go
package observe

import (
    "log/slog"
    "net/http"
    "sync/atomic"
    "time"
)

type Metrics struct{ Requests,Errors atomic.Uint64; DurationNanos atomic.Uint64 }
func (m *Metrics) Middleware(next http.Handler) http.Handler{return http.HandlerFunc(func(w http.ResponseWriter,r *http.Request){
    start:=time.Now();m.Requests.Add(1);recorder:=&statusWriter{ResponseWriter:w,status:200};next.ServeHTTP(recorder,r);duration:=time.Since(start);m.DurationNanos.Add(uint64(duration))
    if recorder.status>=500{m.Errors.Add(1)};slog.InfoContext(r.Context(),"http request","method",r.Method,"route",r.Pattern,"status",recorder.status,"duration",duration)
})}
type statusWriter struct{http.ResponseWriter;status int}
func (w *statusWriter) WriteHeader(status int){w.status=status;w.ResponseWriter.WriteHeader(status)}
```

## 30. Failure drill { #go-30 }

```go
package drill_test

import (
    "context"
    "errors"
    "testing"
    "time"
)

func CallWithBudget(ctx context.Context,dependency func(context.Context)error)error{
    ctx,cancel:=context.WithTimeout(ctx,50*time.Millisecond);defer cancel();result:=make(chan error,1)
    go func(){result<-dependency(ctx)}();select{case err:=<-result:return err;case <-ctx.Done():return ctx.Err()}
}
func TestHungDependencyIsCancelled(t *testing.T){
    started:=time.Now();err:=CallWithBudget(context.Background(),func(ctx context.Context)error{<-ctx.Done();return ctx.Err()})
    if !errors.Is(err,context.DeadlineExceeded){t.Fatalf("got %v",err)};if time.Since(started)>200*time.Millisecond{t.Fatal("cancellation was not propagated")}
}
```

Для полного drill повторите тот же шаблон с fake DB error, заполненным bounded channel и shutdown context; ожидаемые logs/metrics фиксируйте в assertions.
