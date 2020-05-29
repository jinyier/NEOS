# ** Netlist encryption and obfuscation suite ** #

* This project is an object-oriented framework written in modern C++ for gate-level netlist obfuscation/deobfuscation. The tool contains a set of obfuscation and deobfuscation utilities and can be used through the command line. We are working on open sourcing the tool. For now there is a binary available which is tested on ubuntu16 and centos7. We will do our best to address outstanding issues if reported.

* Version 1.0

* Contacts: 
    - Kaveh Shamsi : kshamsi@ufl.edu
    - Yier Jin : yier.jin@ece.ufl.edu

# Upgrade Note #

A version 2.0 was recently released at https://bitbucket.org/kavehshm/neos. Please use the binary in the new repo for testing. With the new version being released, we will not maintain the version 1.0 in this repo. 

# Capabilities #

* neos has a parser written in boost::qi for reading in netlist files in Bench and Verilog formats supporting combinational circuits that use AND/NAND/OR/NOR/BUF/NOT/XOR/XNOR gates. A set of benchmark circuits are included in the /bench directory.

* Combinational and Sequential SAT based deobfuscation using the Glucouse SAT solver with various key-condition-crunching techniques. Sequential deobfuscation is based on model-checking using integrated bounded-model-checking (BMC) routines. 

* various locking schemes including: interconnect locking (cross-bar insertion, mux-flooding), LUT insertion, XOR/XNOR-AND/OR random insertion and insertion based on flip-rates or symbolic probability values, plus SAT-resilient schemes such as antisat etc.

* miscellaneous functionalities such as drawing .dot graph files, statistical subroutines, FSM reconstruction etc. Many of these will be better accessible from the source code once it is released.

# Usage #

there are two main ways to use neos. First is by simply passing flags and arguments, and second is using the interactive mode. The interactive mode dynamically links to libreadline and the non-interactive mode uses boost_program_options which is statically linked. 

For Ubuntu18 users, Ubuntu18 by default has `libreadline5` and `libreadline7` without `libreadline6`. Until this issue is resolved, you can manually copy `libreadline.so.7` or `libreadline.so.5` in your system to `libreadline.so.6` to get around this issue. 
These files should be in `/usr/lib/x86_64-linux-gnu/`

We start with the non-interactive mode:

```
./bin/neos -h
usage: neos [options] <positional_args>
Allowed options:
  -h [ --help ]               see help
  -i [ --int ]                interactive mode
  -c [ --cmd ]                script execution mode (open interactive shell and
                              run commands)
  -e [ --enc ] [=arg(=undef)] encryption/obfuscation mode
  -d [ --dec ] [=arg(=undef)] decryption/deobfuscation mode: <method> : 
                              decryption method
  -q [ --eq ]                 equivalence checker (key verification)
  -t [ --test ]               test (debug) mode
  --b2v                       bench to verilog converter
  --use_verilog               use verilog as input and output
  -v [ --verbose ] arg        verbosity level
  --cir_stats                 print circuit stats
  --to arg                    set timeout in minutes
  --mo arg                    set memory limit in MB
  --pos_args arg              positional arguments

```

We can mix the `-h` flag with other flags to get more help. For the encryption mode we have:

```
./bin/neos -e -h
usage: neos -e <lock_method> <input_bench> <output_bench> [args]
   antisat        : Anti-SAT locking
   antisat_xor    : Anti-SAT + XOR locking
   aor            : AND/OR insertion
   co             : Corrected output locking
   coxor          : CO + XOR locking
   crlk           : Cross-bar locking (crosslock)
   cstwr          : cost-function based interconnect obfuscation
   cyc            : Cyclic locking (N loops of length L)
   cyc2           : Cyclic locking (N loops of length L) + cyclify original
   deg            : degree-driven insertion
   dl             : DL encryption
   dlxor          : DL+XOR encryption
   int            : interconnect obfuscation suite
   lut            : Random k-LUT insertion
   mmux           : MUX insertion
   mmux_cg        : graph-density MUX insertion
   par            : Cross-bar locking (crosslock)
   shfk           : input-output permutation insertion
   univ           : Universal circuit locking
   xor            : Random XOR/XNOR insertion
   xorflip        : fliprate XOR/XNOR insertion
   xorprob        : XOR insertion with symbolic probability propagation
```

We can now obfuscate some circuits. Create a new directory `mkdir ./bench/testing`. We can then do a simple random XOR/XNOR insertion with 30 key-bits with the following :

```
#!bash
./bin/neos -e xor ./bench/comb/c432.bench ./bench/testing/c432_enc.bench 20
#width xor: 20
encryption complete for ./bench/comb/c432.bench
number of inserted key-bits: 20
overhead: 12%
```

Hopefully the terminal colloring works on your machine as well. 
We can now deobfsucate this circuit with :

```
#!bash
./bin/neos -d ex ./bench/comb/c432.bench ./bench/testing/c432_enc.bench 50
running combinatonal attack...
pos arg: ./bench/comb/c432.bench
pos arg: ./bench/testing/c432_xor.bench
pos arg: 20
iteration:    1; variables: 445 clauses: 1224 assignments: 1  propagations: 445
iteration:    2; variables: 823 clauses: 1949 assignments: 225  propagations: 1657
iteration:    3; variables: 1201 clauses: 2797 assignments: 389  propagations: 2633
iteration:    4; variables: 1579 clauses: 3527 assignments: 619  propagations: 4968
iteration:    5; variables: 1957 clauses: 4371 assignments: 783  propagations: 6306
iteration:    6; variables: 2335 clauses: 5215 assignments: 943  propagations: 8381
iteration:    7; variables: 2713 clauses: 6041 assignments: 1115  propagations: 10596
iteration:    8; variables: 3091 clauses: 6873 assignments: 1287  propagations: 13340
iteration:    9; variables: 3469 clauses: 7741 assignments: 1433  propagations: 17623

Finished! 
iteration=9
key=01101001111100001100

equivalent

time: 0.0468594
```

We can do the same for the `s27` sequential benchmark:
```
./bin/neos -e xor ./bench/seq/s27.bench ./bench/testing/s27_enc.bench 10
#width xor: 20
encryption complete for ./bench/seq/s27.bench
number of inserted key-bits: 10
overhead: 15%
```

We can deobfuscate this with simple unrolling:
```
./bin/neos -d int ./bench/seq/s27.bench ./bench/testing/s27_enc.bench 100
enc-ckt has DFFs. running sequnetial attack...
pos arg: ./bench/testing/s27.bench
pos arg: ./bench/testing/s27_enc.bench
pos arg: 100
dec method is: int
vebrosity: 0
key-condition sweeping mode: 0
BMC sweeping mode: 0
AIG mode : 0
Relative to kckt SAT-sweeping: 0
with propagation limit: -1
with mitter sweep: 0
BDD analysis option: 0
KC BDD size limit: 0
BMC bound limit: 600
called mcdec_int_ckt mitter build
called mcdec_int mitter build
frame: 0  stats -> variables: 76 clauses: 165 assignments: 0  propagations: 0
output mittOut asserted at f: 0

adding termination links
output mittOut asserted at f: 0

DIS iter: 0
  DIP clk_0 -> 1010  1
dis  : 1  stats -> variables: 114 clauses: 267 assignments: 5  propagations: 344
output mittOut asserted at f: 0

DIS iter: 1
  DIP clk_0 -> 1111  1
dis  : 2  stats -> variables: 150 clauses: 351 assignments: 13  propagations: 841
output mittOut asserted at f: 0

DIS iter: 2
  DIP clk_0 -> 0110  1
dis  : 3  stats -> variables: 186 clauses: 439 assignments: 19  propagations: 1330
output mittOut asserted at f: 0

DIS iter: 3
  DIP clk_0 -> 0011  0
dis  : 4  stats -> variables: 222 clauses: 531 assignments: 25  propagations: 2028
frame: 1  stats -> variables: 273 clauses: 686 assignments: 37  propagations: 3577
output mittOut asserted at f: 1

DIS iter: 4
  DIP clk_0 -> 1011  0
  DIP clk_1 -> 1110  1
dis  : 5  stats -> variables: 345 clauses: 864 assignments: 51  propagations: 4121
output mittOut asserted at f: 1

DIS iter: 5
  DIP clk_0 -> 1011  0
  DIP clk_1 -> 1011  0
dis  : 6  stats -> variables: 417 clauses: 1046 assignments: 67  propagations: 5471
output mittOut asserted at f: 1

DIS iter: 6
  DIP clk_0 -> 0011  0
  DIP clk_1 -> 1011  0
dis  : 7  stats -> variables: 489 clauses: 1228 assignments: 233  propagations: 7047
no trace up to bound 2
increasing bmc bound to 3
termination checking: 1  checked 1
reached combinational termination!

sat_time:  0.000000
term_time: 0.000198
gen_time:  0.000000
bmc_time:  0.001653
simp_time: 0.000000
total_time: 0.006386

dis_count: 7
dis_maxlen: 2

key=111000101
checking key

NEOS : /home/kaveh/Development/eclipse/neos/bin/neos
UC Berkeley, ABC 1.01 (compiled Oct  1 2019 23:03:06)
abc 01> dsec ./tmp29878/cir1.bench  ./tmp29878/cir2.bench
Warning: 6 registers in this network have don't-care init values.
The don't-care are assumed to be 0. The result may not verify.
Use command "print_latch" to see the init values of registers.
Use command "zero" to convert or "init" to change the values.
Networks are equivalent.  Time =     0.01 sec
NETWORKS ARE EQUIVALENT

equivalent

done

```
The sequential attack performs queries at different depths and does an equivalence checking using ABC at the end. 



neos also supports an interactive mode using `libreadline`. If your system supports this you can try 
```
./bin/neos -i 
```
 for the interactive mode. Tab-completion on file-names and commands can help you around.
 We can do the above sequential obfuscation using the command mode with the `-c` flag which simply takes a string in double-quotes and passes it to the interactive-mode interpreter. 
 
 ```
 ./bin/neos -c "read_bench ./bench/seq/s27.bench; draw_graph; enc xor 10; draw_graph; quit;"
 ```

make sure to have `xdot` installed on your linux system to view the dot files. In ubuntu you can run:

`sudo apt-get install xdot`


