# otp_hipe_compile
erlang kernel,stdlib use hipe compile

Install llvm..

```shell
yum install llvm
```

Download otp source http://www.erlang.org/download

Enter /path/to/lib/kernel/src/
And edit Makefile

```shell
152 #ifeq ($(NATIVE_LIBS_ENABLED),yes)
153 ERL_COMPILE_FLAGS += +native +"{hipe,[to_llvm]}"
154 #endif
155 ERL_COMPILE_FLAGS += -I../include -Werror
```

Enter /path/to/lib/stdlib/src/
Edit Makefile ...(same of kernel)


Other module ,such as sasl,inets same as above

```shell
./configure
make
make install
```

if you rebuild source
```shell
./configure
make clean
./otp_build autoconf
make
make install
```
