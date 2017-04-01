# otp_hipe_compile
erlang kernel,stdlib use hipe compile

Install llvm..

```shell
yum install llvm
```

Download otp source http://www.erlang.org/download

Enter <code>/path/to/lib/kernel/src/</code>
And edit Makefile

```shell
152 #ifeq ($(NATIVE_LIBS_ENABLED),yes)
153 ERL_COMPILE_FLAGS += +native +"{hipe,[to_llvm]}"
154 #endif
155 ERL_COMPILE_FLAGS += -I../include -Werror
```

Enter <code>/path/to/lib/stdlib/src/</code>
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



If you want to show hipe compile detail, edit <code>/path/to/lib/hipe/llvm/hipe_llvm_main.erl</code>
remove io:format() comment
