---
title: Rest API using Golang and Fiber framework
published: true
---

## Getting Started

For the purpose of this tutorial, we are going to build a simple todo list RESTful API. To achieve that we are going to use Golang and Fiber framework.

## What is REST
REST is acronym for REpresentational State Transfer. It is architectural style for distributed hypermedia systems and was first presented by Roy Fielding in 2000 in his famous dissertation you can read more [Here](https://restfulapi.net/).

## Installing Dependencies
You need to install **golang v1.14** or higher and install **Fiber framework** using
`go get github.com/gofiber/fiber/v2`

## Project Layout
Let’s create some folders inside the project, so we are going to have 3 folders
`api/` inside this folder we put all the models.
`cmd/` this folder contains the entrypoint for the service.
`pkg/` this one contains all business logic.
in the end, we get a folder structure like this

![structure](https://github.com/idirall22/hacker-blog/blob/master/assets/01.PNG "structure")

## Models
In `api/` folder let’s create a file **task.go** and add a task structure, we have two struct the Task struct and the Form struct.

```go
package task

import "time"

//Task struct
type Task struct {
	ID          int64     `json:"id"`
	Title       string    `json:"title"`
	Description string    `json:"description"`
	Done        bool      `json:"done"`
	CreatedAt   time.Time `json:"created_at"`
}

// Form struct
type Form struct {
	Title       string `json:"title"`
	Description string `json:"description"`
}
```

## Store
Let's add a folder `store/` in the `pkg/` folder and inside a file **store.go**
Now let’s define the Store interface.


```go
package store

import (
	task "github.com/idirall22/products_api/api"
)

// Store interface
type Store interface {
	Create(form task.Form) (int64, error)
	Get(id int64) task.Task
	List() []task.Task
	Update(task task.Task) error
	Delete(id int64) error
	ToggleDone(id int64) error
}
```

Not finished yet
