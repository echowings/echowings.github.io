---
title: "Learning Golang 01"
date: 2021-04-07T18:07:23+08:00
lastmode: 2021-04-07T18:07:23+08:00
draft: false
tags: [ "go", "golang", "programming", "dev" ]
categories: ["programming"]
reward: true
mathjax: true
---

#Learning Golang notebook --001

## How to install golang 

### Set go workspace
Go still expects there to be a single workspace for third-party Go tools installe via `go install`.By default,this workspace is located in `$HOME/go`.

```bash
$HOME/go/src
$HOME/go/bin
$HOME
```

1.  Install golang with homebrew

  ```bash
  brew install go
  ```

2 set `GOPATH` on unix like os.

  ```bash
  cat << EOF >> ~/.zshrc
  export GOPATH=$HOME/go
  export PATH=$PATH:%GOPATH/bin
  ```
  set `GOPATH` on window
  
  ```bash
  setx GOPATH $USERPROFILE%\go
  setx path "%PATH%;%USERPROFILE%\bin"
  ``` 
  
  
  ## GO command
  
   - `go run`
   - `go build`

 ## `hello world`
 
 ```bash
 # hello.go
 pachage main
 import "fmt"
 func main(){
 	fmt.Println("Hello, world!")
 }
 
 ```
 ```bash
 #Just run go code,without build binary in currently directory path.but build it in temporary direcory.
 go run hello.go
 
 #Build binary without run 
 go build hello.go
 
 # Build specific binary name
 go build -o hellworld hello.go
 
 ```
 ## Install Third-Party Go Tools
 
  - Install 3rd go tools
 
  ```bash
  go install github.com/rakyll/hey@latest
  ```
  - Upgrade 3rd go tools by run the install command again 

  ```bash
  go install github.com/rakyll/hey@latest
  
  # Install goimports
  go install golang.org/x/tools/cmd/goimports@latest
  #go: downloading golang.org/x/tools v0.1.0
  #go: downloading golang.org/x/sys v0.0.0-20210119212857-b64e53b001e4
  #go: downloading golang.org/x/mod v0.3.0
  #go: downloading golang.org/x/xerrors v0.0.0-20200804184101-5ec99f83aff1
  ```
  
  
 ##  `goimport`
  
  ```bash
  goimports -l -w .
  # The `-l` flag tells `goimports` to print the files with incorrect formatting to the console
  # The `-w` flag tells `goimports` to modify the files in-place.
  # The . specifies the files to be scanned: everything in the current directory and all of its sub-directories.
  ```
 ## `golint`
 
 - Install golint

 ```bash
 go install golang.org/x/lint/golint@latest
 ```
 - Run it

 ```bash
 # Run golint over your entire project
 golint ./...
 ```
  
  
  
  
  