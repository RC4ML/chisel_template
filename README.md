# RC4ML Chisel Template

1. Please make sure you have already installed mill https://com-lihaoyi.github.io/mill/mill/Intro_to_Mill.html

2. Clone some repos
	```
	$ git clone https://github.com/RC4ML/chisel_template.git
	$ cd chisel_template
	$ git submodule add git@github.com:carlzhang4/common.git common
	$ git submodule add git@github.com:RC4ML/qdma.git qdma
	```

3. Generate your first module:
	```
	$ mill project_foo Foo
	```
	Corresponding sv file Foo.sv can be found under Verilog folder

	When you write a new module and want to generate its verilog, edit Elaborate.scala
	For example a new Bar module, put following code near other cases in Elaborate.scala.
	```
	"case "Foo" => stage.execute(arr,Seq(ChiselGeneratorAnnotation(() => new Foo()),dir)) "
	```


# How to generate a QDMA benchmark project

1. First, create a vivado project, vivado version must either of 2020.01/2020.02/2021.01(recommended)

2. Modify one line in ./qdma/src/examples/basic/QDMATop.scala, replace with your vivado version and PCIe width, e.g.
	```
	val qdma = Module(new QDMA(VIVADO_VERSION="202101", PCIE_WIDTH=16)
	```  
	Note that `VIVADO_VERSION` should match your Vivado version, and `PCIE_WIDTH` is 16 for a PCIe Gen 3 machine (or 8 for a PCIe Gen 4 machine).

3. Generate Top module
	```
	$ mill project_foo QDMATop
	```
	This will generate a QDMATop.sv under Verilog folder, copy and add it to your vivado project. 

	And it will also print several tcl commands, starts with create_ip xxxx, ends with update_compile_order -fileset sources_1.

	Copy these tcls and execute them in your vivado project's tcl console, this will help you to generate a QDMA IP.

4. Copy xdc file (./qdma/sv) to your vivado project and add it as constraint.

	Then you can generate your bitstream in vivado.

5. Corresponding benchmark software and corresponding driver could be found in https://github.com/RC4ML/rc4ml_qdma

	You can also write your own top file by refering QDMATop.scala.


# How to simplify your workflow with several scripts


### postElaborating.py

1. Create a config.json file in the project root dir, fill it with following contents
	```
	{
		"project_foo":{
			"destIPRepoPath" : "/path to your vivado project/example.srcs/sources_1/ip",
			"destSrcPath" : "/path to your vivado project/example.srcs/sources_1/new",
			"moniterDelay" : 6,
			"projectGeneration" : "Versal"
		}
	}
	```
	destSrcPath is where you want to put your sv file
	destIPRepoPath is where you want to put your vivado ip if you have used chisel based ila in your project.
	projectGeneration should be "ultrascale" if your target board is UltraScale+ boards (e.g., VCU128, U50, U280) or "versal" if your target borad is Versal boards (e.g., VHK158, V80)

2. 
	```
	$ python3 postElaborating.py project_foo QDMATop -t -p
	```
	Above command will help you move sv file to your vivado project and generate some tcl if you have used it.


### instant.py
This script aims to generate some basic testbench content automatically.

1. Move ./common/sv/TESTER.sv to your vivado project, and add it as simulation source

2. If your sv module file has been generated, run following command to generate a testbench, replace Foo with your module name. Copy the output to your testbench file.
	```
	$ python3 instant.py Foo
	```

3. Control the signals by yourself.
