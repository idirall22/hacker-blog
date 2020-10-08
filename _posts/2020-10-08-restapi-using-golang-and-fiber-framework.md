---
title: Rest API using Golang and Fiber framework
published: true
---

## Getting Started

For the purpose of this tutorial, we are going to build a simple todo list RESTful API. To achieve that we are going to use Golang and Fiber framework.
Fiber is an Express inspired web framework built on top of Fasthttp, the fastest HTTP engine for Go. Designed to ease things up for fast development with zero memory allocation and performance in mind.

## What is REST
REST is acronym for REpresentational State Transfer. It is architectural style for distributed hypermedia systems and was first presented by Roy Fielding in 2000 in his famous dissertation you can read more [Here](https://restfulapi.net/).

## Installing Dependencies
First of all, download and install Go. 1.14 or higher is required.
Installation is done using the go get command:
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

## Implement The Store Interface
To implement an interface in Go, we just need to implement all the methods in the interface.
In `pkg/store/` folder let's add a new folder `memory/` and inside a file `memory.go`.

```go
package memory

import (
	"fmt"
	"time"

	task "github.com/idirall22/todo_api/api"
)

// Memory store struct.
type Memory struct {
	tasks []task.Task
}

// NewMemoryStore create memory store.
func NewMemoryStore() *Memory {
	return &Memory{
		tasks: []task.Task{},
	}
}

// Create task
func (m *Memory) Create(form task.Form) (int64, error) {
    // calculate th id from (length of the tasks + 1)
	var id int64 = int64(len(m.tasks)) + 1
	task := task.Task{
		ID:          id,
		Title:       form.Title,
		Description: form.Description,
		Done:        false,
		CreatedAt:   time.Now(),
	}

	m.tasks = append(m.tasks, task)
	return id, nil
}

// Get task
func (m *Memory) Get(id int64) (task.Task, error) {
	for _, task := range m.tasks {
		if task.ID == id {
			return task, nil
		}
	}
	return task.Task{}, fmt.Errorf("There is no task with id: %d", id)
}

// List the tasks.
func (m *Memory) List() []task.Task {
	return m.tasks
}

// Update a single task.
func (m *Memory) Update(id int64, t task.Form) error {
	for index, ts := range m.tasks {
		if ts.ID == id {
			m.tasks[index].Title = t.Title
			m.tasks[index].Description = t.Description
			break
		}
	}
	return nil
}

// Delete a task using the id.
func (m *Memory) Delete(id int64) error {
	for index, ts := range m.tasks {
		if ts.ID == id {
			m.tasks = append(m.tasks[:index], m.tasks[index+1:]...)
			break
		}
	}
	return nil
}

// ToggleDone toggle task status.
func (m *Memory) ToggleDone(id int64) error {
	for index, ts := range m.tasks {
		if ts.ID == id {
			m.tasks[index].Done = !m.tasks[index].Done
		}
	}
	return nil
}

```

## The Service
Now we should create the todo service, inside we will add the business logic.

```go
package service

import (
	task "github.com/idirall22/todo_api/api"
	"github.com/idirall22/todo_api/pkg/store"
)

// Service struct
type Service struct {
	store store.Store
}

// NewService create new todo service.
func NewService(store store.Store) *Service {
	return &Service{store}
}

// CreateTask task
func (s *Service) CreateTask(formTask task.Form) (int64, error) {
	return s.store.Create(formTask)
}

// GetTask single task
func (s *Service) GetTask(id int64) (task.Task, error) {
	return s.store.Get(id)
}

// ListTask Tasks
func (s *Service) ListTask() []task.Task {
	return s.store.List()
}

// UpdateTask single task
func (s *Service) UpdateTask(id int64, form task.Form) error {
	return s.store.Update(id, form)
}

// DeleteTask single task
func (s *Service) DeleteTask(id int64) error {
	return s.store.Delete(id)
}

// ToggleDoneTask single task
func (s *Service) ToggleDoneTask(id int64) error {
	return s.store.ToggleDone(id)
}

```
If you look closer to the service you should see that we are implementing the same methods as the Store interface.

## Service Routes
Now we have the loogic in place let's add a file `main.go` inside the `cmd/` folder
and create the service routes using **Fiber**

```go
package main

import (
	"encoding/json"
	"strconv"

	"github.com/gofiber/fiber/v2"

	task "github.com/idirall22/todo_api/api"
	service "github.com/idirall22/todo_api/pkg"
	"github.com/idirall22/todo_api/pkg/store/memory"
)

func main() {
	app := fiber.New()
	service := service.NewService(memory.NewMemoryStore())
	api := app.Group("/api")
	v1 := api.Group("/v1")
	app.Listen(":3000")
}
```
we start by creating a Fiber instance and todo service, then we add a group `api` and wrap it inside another group `v1` at the end we get an url like `/api/v1` so we do not need to this URL to all the routes manually.

### 1. Create task route
we start with the **create task route**, we define the function that handle the `POST` method and set the route URL to `/todos`,  then in habdler function first we parse the request body into the Form struct then we call the service **Create Task** finally we return the new task id just created.

```go
// POST create a task
	// curl -d '{"title": "first task", "descritption": "simple"}' -H "Content-Type: application/json" -X POST 127.0.0.1:3000/api/v1/todos
	v1.Post("/todos", func(c *fiber.Ctx) error {
		form := task.Form{}
		// we parse the request body
		if err := c.BodyParser(&form); err != nil {
			return c.SendStatus(400)
		}
		id, _ := service.CreateTask(form)

		return c.JSON(
			map[string]interface{}{
				"id": id,
			},
		)
	})
```
### 2.Get Single Task route
Here we choose the `GET` method and for the url `/todos/:id`, the `:id` will be used to passe the task id. First we start by parseing the id and check if it's a **int64** if it's not we return a `400 status Bad Request Error`. then we call the service **Get Task** and finally we return the task.
```go
	// GET single task
	// curl -H "Content-Type: application/json" 127.0.0.1:3000/api/v1/todos/1
	v1.Get("/todos/:id", func(c *fiber.Ctx) error {
		id, err := strconv.ParseInt(c.Params("id"), 10, 64)
		if err != nil {
			return c.SendStatus(400)
		}
		task, err := service.GetTask(id)
		if err != nil {
			c = c.Status(404)
			return c.SendString(err.Error())
		}

		return c.JSON(task)
	})
```
### 3.List Tasks.
Here we choose the `GET` method and for the url `/todos`, then we call the service **List Task** and finally we return a list of tasks.
```go
	// List all tasks
	// curl -H "Content-Type: application/json" 127.0.0.1:3000/api/v1/todos
	v1.Get("/todos", func(c *fiber.Ctx) error {
		tasks := service.ListTask()
		return c.JSON(tasks)
	})
```

### 4.Update a Task
Here we choose the `PUT` method and for the url `/todos/:id`, First we start by parseing the id and check if it's a **int64** if it's not we return a `400 status Bad Request Error`, then we parse the request body into the Form struct and we pass it to the service **Update Task**.

```go
	// Update a task
	/*
		curl \
		-d '{"title": "first task updated", "descritption": "simple updated"}' \
		-H "Content-Type: application/json" \
		-X PUT \
		127.0.0.1:3000/api/v1/todos
	*/
	v1.Put("/todos/:id", func(c *fiber.Ctx) error {
		id, err := strconv.ParseInt(c.Params("id"), 10, 64)
		if err != nil {
			return c.SendStatus(400)
		}
		form := task.Form{}
		if err := c.BodyParser(&form); err != nil {
			return c.SendStatus(400)
		}
		_ = service.UpdateTask(id, form)
		return c.SendStatus(204)
	})
```
### 5.Delete Task
Here we choose the `DELETE` method and for the url `/todos/:id`, First we start by parseing the id and check if it's a **int64** if it's not we return a `400 status Bad Request Error`, then we pass it to the service **Delete Task**.

```go
	// Delete a task
	/*
		curl \
		-H "Content-Type: application/json" \
		-X DELETE \
		127.0.0.1:3000/api/v1/todos/1
	*/
	v1.Delete("/todos/:id", func(c *fiber.Ctx) error {
		id, err := strconv.ParseInt(c.Params("id"), 10, 64)
		if err != nil {
			return c.SendStatus(400)
		}
		_ = service.DeleteTask(id)
		return c.SendStatus(204)
	})

```
### 6. Toggle Task Status
Finnaly the toggle route, we choose the `POST` method and for the url `/todos/toggle/:id`, First we start by parseing the id and check if it's a **int64** if it's not we return a `400 status Bad Request Error`, then we pass it to the service **Toggle Done Task**
```go
	// Toggle done task
	/*
		curl \
		-H "Content-Type: application/json" \
		-X POST \
		127.0.0.1:3000/api/v1/todos/toggle/1
	*/
	v1.Post("/todos/toggle/:id", func(c *fiber.Ctx) error {
		id, err := strconv.ParseInt(c.Params("id"), 10, 64)
		if err != nil {
			return c.SendStatus(400)
		}
		_ = service.ToggleDoneTask(id)
		return c.SendStatus(204)
	})
```

## Run The Service
in the bash let's run the command `go cmd/main.go`

![bash](https://github.com/idirall22/hacker-blog/blob/master/assets/02.PNG "bash")

## Next Steps
With the tools and methods covered in this tutorial, you should now be able to create simple REST APIs using Golang and FIber Framework. A lot of best practices that are not essential to the process were skipped, so don’t forget to:
- Implement proper validations (e.g., make sure to not allows a task with empty title)
- Implement unit testing and error reporting.
