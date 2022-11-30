# Docker Compose
## Background
We usually create a single container application such as Docker when we create an application. Previous tasks enabled us to create a single container application using Docker, but you need to learn how to make multiplex container applications. Today's lesson aims to create multiplex container applications using docker-compose.

## Task
Based on [ref](https://qiita.com/muroya2355/items/d48c384a4a82c7ed34ae), please build a multiplex container including Golang and PostgreSQL using Docker Compose. (In your feature, you can build an environment for React-Go application.)

### Step0: Install Golang
https://go.dev/doc/install
### Step1: Create golang + postgreSQL
https://qiita.com/muroya2355/items/d48c384a4a82c7ed34ae
If you don't know GOPATH, you can get GOPATH by executing the below command.
```
go env GOPATH
```

<details><summary> Answer </summary>

### Modify the golang dockerfile
If the error has been ocurred, you should replace below command.
- Error
```
executor failed running [/bin/sh -c go get github.com/lib/pq]: exit code: 2
ERROR: Service 'app' failed to build : Build failed 
```
- Dockerfile
```
# Specify the base image for the go app.
FROM golang:1.15

WORKDIR /go/src/github.com/postgres-go
# Copy everything from this project into the filesystem of the container.
COPY . .
# Obtain the package needed to run code. Alternatively use GO Modules. 
RUN go get -u github.com/lib/pq

WORKDIR /go/src/yuta/go
# Add the contents of the host OS . /go contents to the working directory
ADD ./go .
```
### Modify the postgreSQL dockerfile
If the error has been ocurred, you should replace below command.
- Error
```
initdb: error: invalid locale settings; check LANG and LC_* environment variables
```
- Dockerfile
```
FROM postgres:latest

ENV LC_ALL en_US.UTF-8
ENV LC_CTYPE en_US.UTF-8
RUN localedef -i ja_JP -c -f UTF-8 -A /usr/share/locale/locale.alias ja_JP.UTF-8
ENV LANG ja_JP.UTF-

# Copy the initialization sql file to the specified directory in the container
COPY ./docker/postgres/init/*.sql /docker-entrypoint-initdb.d/
```
</details>

### Step2 Build a server in the golang container
You modify /go/main.go, Dockerfile, docker-compose.yml for building a server.
First, you copy a below code and paste it in /go/main.go.(Some parts omitted.)
We can complete building a server.
The detail of explanation is described in [this](https://note.com/miso_hijiki/n/nf816bdf23430)
```
package main

import (
	"database/sql"
	"fmt"
	"log"
	"net/http"

	// postgres ドライバ
	_ "github.com/lib/pq"
)

// TestUser : テーブルデータ
type TestUser struct {
	UserID   int
	Password string
}

// main 関数
func main() {
	http.HandleFunc("/a", handler)
	http.ListenAndServe(":8181", nil)
}

// handler 関数
func handler(w http.ResponseWriter, r *http.Request) {

	// Db: データベースに接続するためのハンドラ
	var Db *sql.DB
	// Dbの初期化
	Db, err := sql.Open("postgres", "host=postgres user=app_user password=password dbname=app_db sslmode=disable")
	if err != nil {
		log.Fatal(err)
	}

	// SQL文の構築
	sql := "SELECT user_id, user_password FROM TEST_USER WHERE user_id=$1;"

	// preparedstatement の生成
	pstatement, err := Db.Prepare(sql)
	if err != nil {
		log.Fatal(err)
	}

	// 検索パラメータ（ユーザID）
	queryID := 1
	// 検索結果格納用の TestUser
	var testUser TestUser

	// queryID を埋め込み SQL の実行、検索結果1件の取得
	err = pstatement.QueryRow(queryID).Scan(&testUser.UserID, &testUser.Password)
	if err != nil {
		log.Fatal(err)
	}

	// 検索結果の表示
	fmt.Fprintf(w, testUser.Password)
}

```
But, we didn't access the server as below in local terminal.
```
% curl localhost:8181/a  
curl: (7) Failed to connect to localhost port 8181: Connection refused
```
So, please consider this problem by referring to [it](https://docs.docker.com/compose/gettingstarted/#step-3-define-services-in-a-compose-file)

The task will be terminated when the following message is displayed
```
% curl localhost:8181/a                           
password1
```

<details><summary> Answer </summary>
- Answer(docker-compose.yml)
Add ports under services.postgres
```
services:
  postgres:
    ports:
      - "8181:8181"
```
</details>