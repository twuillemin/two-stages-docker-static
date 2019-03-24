# Small Go Static build for Docker
This short snippet intends to build a small and statically linked executable to be run in Docker. 

Give me the code: [Dockerfile](https://github.com/twuillemin/two-stages-docker-static/blob/master/build/package/Dockerfile)

# What is the issue?
## A simple static build
For running an application in Docker, the application must be statically linked, as while executing it won't be able to 
access shared libraries. Sounds simple, no?

So after a bit of search in internet, you find something like:
```bash
CGO_ENABLED=0 GOOS=linux go build -a -ldflags '-extldflags "-static"' .
```

which sounds a reasonable answer to the previous reasonable requirement. And it works well for a lot of cases

## What is the CGO_ENABLED=0 by the way?
Roughly, there are two compilers in go:

 * The standard Go Compiler (`CGO_ENABLED=0`): the standard way of building, good for almost everything
 * The CGo compiler (`CGO_ENABLED=1`): based on gcc, enables the creation of Go packages that call C code and also cross 
 compiling.

As long as your application does not have dependencies requiring to build against C code, life is sweet. Unfortunately,
some packages (such as SQLite for example), can only be built with CGO_ENABLED=1.

## What happens with CGO_ENABLED=1?
When building with `CGO_ENABLED=1`, gcc try to link the generated executable against the standard glic. But, 
for a lot of good (or bad) technical reasons, the glibc cannot be statically linked. On a side note, as the glibc is 
GPL, this may be an issue depending of your licensing model.

So are we stuck in the middle of the desert without any solution to build a nice docker image when using some libraries?
Fortunately, there is a solution. The glibc is not the only implementation of the base C functions. There is an 
alternative named [Musl](https://www.musl-libc.org/) which allows to statically link against it easily.

# The Musl build
Unfortunately, there are few, or none, desktop friendly musl-linux distribution that could allows us to statically build 
the executable. The solution is to build the container in two steps:

 * A first container is a Musl-linux based container having go. This container is used to statically build the 
 application executable
 * A second container is a scratch container just having the generated application
 
# Reduce the size of the application
As a good to have, we can also reduce the size of the application.  

 * `strip`: for discarding symbols from object files
 * `upx`: for compressing the executable


# License

Copyright 2018 Thomas Wuillemin  <thomas.wuillemin@gmail.com>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this project or its content except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
