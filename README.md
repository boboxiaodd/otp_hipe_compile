# otp_hipe_compile
erlang kernel,stdlib use hipe recompile


```bash
#!/bin/bash
ERLC="erlc +native +{hipe,[to_llvm]} +warnings_as_errors +debug_info"
ERL_SRC_LIB="/root/otp_src_19.3/lib"
KERNEL_DIR="$ERL_SRC_LIB/kernel/src"
KERNEL_EBIN="/usr/local/lib/erlang/lib/kernel-5.2/ebin"
STDLIB_DIR="$ERL_SRC_LIB/stdlib/src"
STDLIB_EBIN="/usr/local/lib/erlang/lib/stdlib-3.3/ebin"

echo "Enter $KERNEL_DIR"
cd $KERNEL_DIR
for filename in *.erl;
do
	if [ $filename = "erl_epmd.erl" ]; then
		echo "ERLC $filename"
 		$ERLC -Derlang_daemon_port=4369  -Depmd_dist_high=5 -Depmd_dist_low=5 -Depmd_node_type=110 -Depmd_port_no=4369 -o $KERNEL_EBIN -I $ERL_SRC_LIB/stdlib/include -I $KERNEL_DIR/kernel/include -I $KERNEL_DIR/kernel/src $filename
 	else
 		echo "ERLC $filename"
 		$ERLC -o $KERNEL_EBIN -I $ERL_SRC_LIB/stdlib/include -I $ERL_SRC_LIB/kernel/include -I $ERL_SRC_LIB/kernel/src $filename
 	fi
 	done
done
echo "Enter $STDLIB_DIR"
cd $STDLIB_DIR
for filename in *.erl; 
do
	echo "ERLC $filename"
 	$ERLC -o $STDLIB_EBIN -I $ERL_SRC_LIB/kernel/include -I $ERL_SRC_LIB/stdlib/include -I $ERL_SRC_LIB/stdlib/src $filename
done
```
