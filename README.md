# otp_hipe_compile
erlang kernel,stdlib use hipe compile

Download otp source

Enter /path/to/lib/kernel/src/
Edit Makefile

```shell
152 #ifeq ($(NATIVE_LIBS_ENABLED),yes)
153 ERL_COMPILE_FLAGS += +native +"{hipe,[to_llvm]}"
154 #endif
155 ERL_COMPILE_FLAGS += -I../include -Werror
```

Enter /path/to/lib/stdlib/src/
Edit Makefile ...(same of kernel)

```shell
./configure
make
make install
```

if you rebuild source
```shell
./configure
./otp_build autoconf
make
make install
```
