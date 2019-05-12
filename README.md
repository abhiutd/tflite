# MLModelScope TFLite Agent

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)


## Installation


### Build from source

Refer to `$GOPATH/src/github.com/rai-project/go-pytorch/dockerfiles` to know how to build `Libtorch` from source. Note that one can also use `build_libtorch.py` script provided as part of the Pytorch repository to do the same.

Place the extracted/built library to `/opt/libtorch/`.

Configure the linker environmental variables since Libtorch library has been extracted to a non-system directory. Place the following in either your `~/.bashrc` or `~/.zshrc` file

Linux

```
export LIBRARY_PATH=$LIBRARY_PATH:/opt/libtorch/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/libtorch/lib
```

macOS

```
export LIBRARY_PATH=$LIBRARY_PATH:/opt/libtorch/lib
export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/opt/libtorch/lib
```

Or use [Dep](https://github.com/golang/dep).

```
dep ensure -v
```

This installs the dependency in `vendor/`.

#### libjpeg-turbo

[libjpeg-turbo](https://github.com/libjpeg-turbo/libjpeg-turbo) is a JPEG image codec that uses SIMD instructions (MMX, SSE2, AVX2, NEON, AltiVec) to accelerate baseline JPEG compression and decompression. It outperforms libjpeg by a significant amount.

The default is to use libjpeg-turb, to opt-out, use build tag `nolibjpeg`.

To install libjpeg-turbo, refer to [libjpeg-turbo](https://github.com/libjpeg-turbo/libjpeg-turbo/releases).

Linux

```
  cd /tmp
  wget http://downloads.sourceforge.net/project/libjpeg-turbo/$1/libjpeg-turbo-official_$1_amd64.deb
  dpkg -x libjpeg-turbo-official_$1_amd64.deb /tmp/libjpeg-turbo-official
  mv /tmp/libjpeg-turbo-official/opt/libjpeg-turbo /tmp/libjpeg-turbo
```

macOS

```
brew install jpeg-turbo
```

## External services

MLModelScope relies on a few external services.
These services provide tracing, registry, and database servers.

#### Installing Docker

[Install Docker](https://docs.docker.com/engine/installation/). An easy way is using

```
curl -fsSL get.docker.com -o get-docker.sh | sudo sh
sudo usermod -aG docker $USER
```

#### Configuration

You must have a `carml` config file called `.carml_config.yml` under your home directory. An example config file `carml_config.yml.example` is in [github.com/rai-project/MLModelScope](https://github.com/rai-project/MLModelScope) . You can move it to `~/.carml_config.yml`.

The following configuration file can be placed in `$HOME/.carml_config.yml` or can be specified via the `--config="path"` option.

```yaml
app:
    name: carml
    debug: true
    verbose: true
    tempdir: ~/data/carml
registry:
    provider: consul
    endpoints:
        - localhost:8500
    timeout: 20s
    serializer: jsonpb
database:
    provider: mongodb
    endpoints:
        - localhost
tracer:
    enabled: true
    provider: jaeger
    endpoints:
        - localhost:9411
    level: FULL_TRACE
logger:
    hooks:
        - syslog
```

#### Starting Trace Server

-   On x86 (e.g. intel) machines, start [jaeger](http://jaeger.readthedocs.io/en/latest/getting_started/) by

```
docker run -d -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 -p5775:5775/udp -p6831:6831/udp -p6832:6832/udp \
  -p5778:5778 -p16686:16686 -p14268:14268 -p9411:9411 jaegertracing/all-in-one:latest
```

-   On ppc64le (e.g. minsky) machines, start [jaeger](http://jaeger.readthedocs.io/en/latest/getting_started/) machine by

```
docker run -d -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 -p5775:5775/udp -p6831:6831/udp -p6832:6832/udp \
  -p5778:5778 -p16686:16686 -p14268:14268 -p9411:9411 MLModelScope/jaeger:ppc64le-latest
```

The trace server runs on http://localhost:16686

#### Starting Registry Server

-   On x86 (e.g. intel) machines, start [consul](https://hub.docker.com/_/consul/) by

```
docker run -p 8500:8500 -p 8600:8600 -d consul
```

-   On ppc64le (e.g. minsky) machines, start [consul](https://hub.docker.com/_/consul/) by

```
docker run -p 8500:8500 -p 8600:8600 -d MLModelScope/consul:ppc64le-latest
```

The registry server runs on http://localhost:8500

#### Starting Database Server

-   On x86 (e.g. intel) machines, start [mongodb](https://hub.docker.com/_/mongo/) by

```
docker run -p 27017:27017 --restart always -d mongo:3.0
```

You can also mount the database volume to a local directory using

```
docker run -p 27017:27017 --restart always -d  -v $HOME/data/MLModelScope/mongo:/data/db mongo:3.0
```

## Usage

Run the agent with GPU enabled

```
cd $GOPATH/src/github.com/rai-project/pytorch
go run pytorch-agent/main.go -l -d -v
```

Run the agent ithout GPU or libjpeg-turbo

```
cd $GOPATH/src/github.com/rai-project/pytorch
go run -tags="nogpu nolibjpeg" pytorch-agent/main.go -l -d -v
```
