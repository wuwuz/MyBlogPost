# Installing Golang

I am using WSL and VS Code. My shell is zsh.

(Ref : https://medium.com/@betakuang/setup-go-development-environment-with-vs-code-and-wsl-on-windows-62bd4625c6a7)

### Install Golang in WSL

Don't use `apt-get`.

Download the binary from https://golang.org/dl/. 

Extract the package and install in `/usr/local`.

```
wget https://dl.google.com/go/go1.13.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf https://dl.google.com/go/go1.13.linux-amd64.tar.gz
```

Set up the environment variable. (Don't set the GOROOT, it causes problems in some cases.)

Add the following command in `~/.zshrc` : 

```
export GOPATH="$HOME/Go" # or any directory to put your Go code
export PATH="$PATH:/usr/local/go/bin:$GOPATH/bin"
```

Then `source ~/.zshrc`. 

Now check `go version`.

Notice : `GOPATH` is where you store the Go project . It also contains some tools necessary.  `GOROOT` is where the Go compiler at and in normal case you shouldn't set it manually.

### Install Remote-WSL and Go extension in VS-code

Firstly install `Remote-WSL` in VS-code.

Then click the bottom-left button, and click `Remote-WSL: New Window`.

Now we have the remote-VS-code.

Install `Go` extension in  this remote-VS-code.

Set up the `setting.json `: 

```json
    "terminal.integrated.shell.linux": "/usr/bin/zsh",
    "go.gopath": "/home/<name>/Go",
    "go.goroot": "/usr/local/go",
    "go.formatTool": "goformat",
    "go.autocompleteUnimportedPackages": true,
    "[go]": {
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.organizeImports": true
        }
    }
```

### Write the Hello-World Code

Create a folder under `$GOPATH/src/`. For example, `mkdir $GOPATH/src/hello`.

Then write the first Go code in this folder.

```go
package main

import "fmt"

func main(){
	fmt.Printf("Hello, world!\n")
}
```

Then in zsh, type `go build`. You should see one executable file was created. 

Try `./hello`.

### Install some tools for Go

In VS-code, use `ctrl+shift+p` or `view->Command Pallete`, type `Go: Install/Upgrade tools`.

Notice : If you suffer from network problems, e.g. Firewall, you can try the following solutions. 

1. https://github.com/golang/lint/issues/288
2. In `~/.zshrc`, add two extra command :

```json
export GOPROXY="https://goproxy.io"
export GO111MODULE=on
```

Those two solution solve my problems partially. I still suffer from network problems in installing some packages.