# go-mnist-client

An example of tensorflow_serving client in Go. 

It demonstrates how to connect to the `tensorflow_model_server` launched
as described in 
[Serving a TensorFlow Model](https://www.tensorflow.org/serving/serving_basic) on
TensorFlow documentation webpage.

The main problem with tensorflow_serving is a lack of Go package ready to 
import and run.

This example is based on:
[github.com/tobegit3hub/tensorflow_template_application](https://github.com/tobegit3hub/tensorflow_template_application)

## Setup fake workspace through a`$GOPATH` or copy files

For a convenience, I provide the generated files under the `gopath` 
subdirectory. You can copy files directly to your `$GOPATH`.

```bash
cp gopath/src $GOPATH
```

## Generating from sources

For a purpose of generating proto we will use a `docker` container `grpc/go` with 
`protoc`, `go` and [`gRPC`](https://grpc.io/) protoc extension installed.

Remember that a version of TensorFlow Serving, TensorFlow C++ library and
and Go TensorFlow library versions should match.

```
go get -u golang.org/x/net/context
go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
go get -u google.golang.org/grpc
go get -u github.com/tensorflow/tensorflow/tensorflow/go
go get -u go.uber.org/ratelimit
```


Let's start a docker container mounting a fake gopath:
```bash
docker run --rm -it -v $PWD/gopath:/gopath:ro -v $PWD/gen:/gen grpc/go /bin/bash
```
and copy paste:
```
export GOPATH=/gopath

git clone https://github.com/tensorflow/serving
git clone https://github.com/tensorflow/tensorflow

IN=$PWD
OUT=/gen
protoc -I=$IN/serving -I $IN/tensorflow \
    --go_out=plugins=grpc:$OUT \
    $IN/serving/tensorflow_serving/apis/*.proto

IN=$PWD/tensorflow
protoc -I=$IN --go_out=plugins=grpc:$OUT \
    $IN/tensorflow/core/framework/*.proto
    
protoc -I=$IN --go_out=plugins=grpc:$OUT \
    $IN/tensorflow/core/protobuf/{saver,meta_graph}.proto
    
protoc -I=$IN --go_out=plugins=grpc:$OUT \
    $IN/tensorflow/core/example/*.proto
```

In most cases we have to fix permissions:
```bash
sudo chown -R $USER.$USER gen
```

Most of the tensorflow_serving and tensorflow is not prepared to be
imported in go directly, therefore we must **fix incorrect import paths**:

``` 
grep -rl '"tensorflow/' --include \*.go gen | 
xargs sed -i 's/"tensorflow\//"github.com\/tensorflow\/tensorflow\/tensorflow\/go\//g'
```

And now we need to copy files into the GOPATH:
```bash
mkdir -p $GOPATH/src/github.com/tensorflow/serving
cp -R gen/tensorflow_serving $GOPATH/src/github.com/tensorflow/serving/
mkdir -p $GOPATH/src/github.com/tensorflow/tensorflow/tensorflow/go/
cp -R gen/tensorflow/core $GOPATH/src/github.com/tensorflow/tensorflow/tensorflow/go/
```