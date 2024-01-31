<h1 align="center">GORM</h1>

## Installation

```
go get -u gorm.io/gorm
```

## Connecting to Database [🔗](https://gorm.io/docs/connecting_to_the_database.html)

In order to connect to the database, a driver is required that matches the selected database

### MySQL

```go
import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {
  dsn := "user:pass@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{}) // *gorm.DB
}
```

### PostgreSQL

```go
import (
  "gorm.io/driver/postgres"
  "gorm.io/gorm"
)

dsn := "host=localhost user=gorm password=gorm dbname=gorm port=9920 sslmode=disable TimeZone=Asia/Jakarta"
db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{}) // *gorm.DB
```

## Create [🔗](https://gorm.io/docs/create.html)

### Create record

```go
// Insert data
(*gorm.DB).Create(&Model{}) // pass pointer

// Batch insert
(*gorm.DB).Create(&[]Model{})

// Batch in size (100)
(*gorm.DB).CreateInBatches(models, 100)

// Selected field
(*gorm.DB).Select("field1", "field2").Create(&Model{})

// Associations
(*gorm.DB).Create(&Model{
  AnotherModel: AnotherModel{}
})
```

### Default value [🔗](https://gorm.io/docs/create.html#Default-Values)

```go
type User struct {
  gorm.Model // extend
  Name string `gorm:"default:guest"`
}
```

### Hooks [🔗](https://gorm.io/docs/create.html#Create-Hooks)

GORM allows user defined hooks to be implemented for `BeforeSave`, `BeforeCreate`, `AfterSave`, `AfterCreate`.

```go
func (m *Model) BeforeCreate(tx *gorm.DB) (err error) {
  // logic here
}

// skip hooks
(*gorm.DB).Session(&gorm.Session{SkipHooks: true}).Create(&Model{})
```

### Upsert [🔗](https://gorm.io/docs/create.html#Upsert-x2F-On-Conflict)

```go
import "gorm.io/gorm/clause"

(*gorm.DB).Clauses(clause.OnConflict{/* logic here */}).Create(&Model{})
```
