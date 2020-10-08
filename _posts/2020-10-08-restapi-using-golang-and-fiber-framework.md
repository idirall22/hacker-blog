---
title: Rest API using Golang and Fiber framework
published: true
---

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

IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII
## Models
In `api/` folder let’s create a file **task.go** and add a task structure, we have two struct the Task struct and the Form struct.

---go
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
---

## Store
Let's add a folder `store/` in the `pkg/` folder and inside a file **store.go**
Now let’s define the Store interface.

---go
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
---




## Getting Started

For the purpose of this tutorial, we are going to build a simple todo list RESTful API. To achieve that we are going to use Golang and Fiber framework.


Text can be **bold**, _italic_, ~~strikethrough~~ or `keyword`.

[Link to another page](another-page).

There should be whitespace between paragraphs.

There should be whitespace between paragraphs. We recommend including a README, or a file with information about your project.

# [](#header-1)Header 1

This is a normal paragraph following a header. GitHub is a code hosting platform for version control and collaboration. It lets you and others work together on projects from anywhere.

## [](#header-2)Header 2

> This is a blockquote following a header.
>
> When something is important enough, you do it even if the odds are not in your favor.

### [](#header-3)Header 3

```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require('./lang/' + l)
  return true;
}
```

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### [](#header-4)Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### [](#header-5)Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### [](#header-6)Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![](https://assets-cdn.github.com/images/icons/emoji/octocat.png)

### Large image

![](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```
