# TCL Scripting WorkShop from Fundamentals to Advanced Techniques
Tool Command Language (Tcl) is a simple and flexible scripting language widely used in EDA, software development, and system administration. It enables automation, command execution, and data manipulation through easy-to-write scripts.
## Day1: Introduction and VSDSYNTH Toolbox Usage
### Task1: Create command of vsdsynth and pass the csv file from UNIX shell to TCL script
1. A custom command (sandy) was created to pass a CSV file from the UNIX shell to the Tcl script, following an algorithm that first informs the system it's a UNIX script.
``` bash 
 #!/bin/tcsh -f  
 ```
2. Created the logo
![pic4](https://github.com/user-attachments/assets/4dd7bf58-10c8-4bee-8090-062191ff3b47)
3. verifying three general scenarios for a user POV
- User Not provided the .csv file as input:
If user type only ./sandysynth without any csv file, then this line of the code will work saying provide the csv file.
![pic6](https://github.com/user-attachments/assets/f0dc111d-ff44-4223-a62a-a5c53525666f)
![image](https://github.com/user-attachments/assets/afa54068-1669-4ac4-81e7-a65e131f94ae)
 - Users provide a .csv file which doesn’t exist:
If user have provided the csv file with ./sandysynth my.csv then first it will check for the argument if it is true, then go to next line and then again check whether the argument 1 is not equal to -help then the error will print by telling the user that cannot find the csv file my.csv
![pic7](https://github.com/user-attachments/assets/46f69159-7fd3-463c-b7d1-4dc90b4f644a)
![image](https://github.com/user-attachments/assets/b8bbeb68-8dd3-4f77-9d62-fded5642bdf1)
- Type “-help” to find out usage:
If we user wants to find out the usage, then type ./sandysynth -help the section is true and enter else loop
![pic8](https://github.com/user-attachments/assets/7413b840-868b-4f4e-9960-f7161221cc1b)
![image](https://github.com/user-attachments/assets/c769aa79-a17e-4a33-a594-4130c7a2c361)
4. source the Unix shell to the Tcl script by passing the required csv file
  ``` bash 
 tclsh pandabro.tcl $argv[1]  
 ```
Note : Make sure the file is executable by using the command chmod -R 777 panda

## Day2: Variable Creation and Processing Constraints from CSV
### Task2: Convert all files to format[1] and SDC format, and pass to synthesis tool “yosys”
#### SUB TASKS INVOLVED IN TASK 2 and 3:
- Create variables
- Check if directories and files mentioned in .csv, exists or not
- Read "Constraints File" for above .csv and convert to SDC format
- Read all files in "Netlist Directory"
- Create main synthesis script in format|2]
- Pass this script to Yosys
1. Create the variables 
![pic10](https://github.com/user-attachments/assets/ac523b71-9a07-4aef-9012-51f25125ef54)
2. Check if directories and files mentioned in .csv exist or not 
![pic12](https://github.com/user-attachments/assets/54ab1b60-f7f1-462f-8035-c9f973bbd7bc)
3. Read constraints file for above .csv and convert to SDC format
![WhatsApp Image 2025-05-09 at 5 50 36 PM](https://github.com/user-attachments/assets/dfefaf2f-b369-401f-8ff3-aa35acaddfa9)
We need to get row number for the inputs like clocks, inputs and outputs and need to process them separately
![pic16](https://github.com/user-attachments/assets/22dae1f1-bc28-43b4-aeb5-401ee7ccb086)

## Day3: Processing Clock and Input Constraints
### Read all files in netlist directory
-Clock ports: 
![image](https://github.com/user-attachments/assets/060c5c73-c099-45e0-ade6-0bb2b24e3c1e)
###### The rectangle area which is highlighted we can get by using the code below and by executing the code we will get the early rise_delay and input ports start we will get the boundaries of that rectangle box
 ``` bash 
 set clock_start [lindex [lindex [constraints search all CLOCKS] 0] 1] 
set clock_start_column [lindex [lindex [constraints search all CLOCKS] 0 ] 0] 
set clock_period_start [lindex [lindex [constraints search all frequency] 0] 0] 
set duty_cycle_start [lindex [lindex [constraints search all duty_cycle] 0] 0] 
 ```
###### We are opening the openMSp430.sdc in write mode by using the command set sdc_file [open $outputDirectory/$DesignName.sdc “w” now we will take the various values from the csv file and write it to sdc constraints which has /port/ signal /variable names picked from the excel file and corresponding values of the constraints. The csv file is input and the sdc constraints are the output.
- clock creation:
![pic18](https://github.com/user-attachments/assets/2f7fba7e-469a-46cb-abec-de3a5dd61eb2)
![pic19](https://github.com/user-attachments/assets/5341b324-f602-4597-bbc3-bb81b5246312)
- Algorithm to categorize input ports as bits and bussed
1. In the inputs ports there are multibit and single bit ports, and the multibit ports are called bus for eg: dbg_i2c_addr and dbg_i2c_broadcast
2. If there is an n bit bus, then put “*” in front of port else if it single bit then keep the port as it is.
3. Related clock is the expression we follow to get that rectangular box. In that rectangular box we need to find all the keywords which are presented in each column.
4. We will get the input port for eg.cpu_en &dbg_i2c_addr=>$pattern1 and will Open the each .V file and grep for $pattern1 in each file then split the string which you get in output using “;” as a delimiter and get the index[0] for that we need to grep the input in patterns generated by above step and then replace multiple spaces by single space in order to do that I need to count no of elements in the string and save it as $count then use the condition like if the $count is grater that 2 then put “* ” in front of the port  else keep the port as it is.
![pic21](https://github.com/user-attachments/assets/14ec62e5-8650-4a0c-b62d-b8473d0f7937)
5. Now investigate the script below which will explain how we can get one input without any spaces eg: “input cpu_en;”
![pic20](https://github.com/user-attachments/assets/a3677e67-a0f2-48f6-a7a7-df71a21082a0)
6. and after grep for replace multiple places, we go the output file.
![pic22](https://github.com/user-attachments/assets/cb2b21b2-2b57-4b74-8bba-850dd020461b)
7. We need to open the tmp1 in read mode and then open the tmp2 file in write mode where we can dump the files. Then we will split the elements in tmp1 using the new line and then uniquify and sort the elements and then join the elements and will write in tmp2 file this will allow us to remove the duplicate elements. Close tmp1 and tmp2 file and again open the tmp2 file in read mode the count the length of the element if it’s greater than 2 then put “*” in front of that port the
![pic23](https://github.com/user-attachments/assets/1ec87b12-3b06-43a0-b5f1-2b29924dee4c)
![pic24](https://github.com/user-attachments/assets/1216c03a-1209-4ed3-9df8-3e86d3b64444)
8. splitting:

![pic25](https://github.com/user-attachments/assets/ac3bbbfd-4693-4937-8044-15da6f8131aa)

9. sorting:

![pic26](https://github.com/user-attachments/assets/f1fd7fdb-929e-4435-8c22-3635220852cd)

10. joining:

![pic27](https://github.com/user-attachments/assets/924f7118-a6e4-47a1-b008-f216c8de02f3)

11. Finally, all these constraints will be saved in sdc file which will show the constraints in file name  $outputDirectory/$DesignName.sdc after the code is fully executed

![pic29](https://github.com/user-attachments/assets/38eac404-2ce6-4927-8a8a-0a552dd2109a)

12. A snippet of the sdc

![pic30](https://github.com/user-attachments/assets/f0fea9a2-d630-4050-ab50-f661c8be20c8)

- Constraints generation logic for the output port and conclusion!!
The script is exactly same as the input, but the only difference is we are replacing the input with the output.
![pic32](https://github.com/user-attachments/assets/f41e8ed8-e6fe-42a7-91c8-b168ff30309c)

## Day4: Completing Scripting and Yosys Synthesis Introduction
### Yosys synthesis toolbox: 
YOSYS is an open-source RTL synthesis and formal verification framework for digital circuits. It takes RTL descriptions (e.g., Verilog) as input and performs synthesis to generate a gate-level netlist. YOSYS supports technology mapping, optimization, and formal verification. It has a scripting interface, integrates with other EDA tools, and is widely used in academia and industry for digital design tasks.
Take the RTL and put that in yosys synthesis toolbox will execute the output. The tool will read all the Verilog file will provide the synthesis Verilog netlist as the output 
- We will create a hierarchy check script to be used by yosys.
![pic33](https://github.com/user-attachments/assets/c8f76632-3429-42c4-a947-9e93acd57071)
-synthesis output:
![pic34](https://github.com/user-attachments/assets/c09a699b-e050-489c-b6eb-65344203d1e2)
- Error handling in hierarchy check:
1. Checking whether the top-level hierarchy is being well connected with all hierarchy which is present file if not which module is missing.
2. If any module not found error flag is set to 1, If all modules found error flag is set to 0.
![pic36](https://github.com/user-attachments/assets/2b84daea-9d65-43f4-960a-8d9989706781)
3. If we change the module name to omsp_clock_mocule to omsp_clock_mocule_vsd, we can see the error in log file and will remove the error in the Verilog file and will run again then the flag is 0 and we did not see any error in hierarchy log file
![pic37](https://github.com/user-attachments/assets/0f5550ba-3536-4cf2-bb70-24515fcb2999)
- Hierarchy log file
 ![pic38](https://github.com/user-attachments/assets/b77d34d6-c886-4275-8659-64914fe0099e)
4. We will check if the hierarchy passes or fail.
Case 1: When all the modules are present and called correctly
![pic40](https://github.com/user-attachments/assets/e1788623-de2b-49f8-abee-676f0e3fd157)
Case 2: referenced in module error due to calling incorrectly or module doesnt exist
![pic41](https://github.com/user-attachments/assets/56d2d7db-b151-4e3d-9c3b-18cd4acd0f29)

## Day5: Advanced Scripting Techniques and Quality of Results Generation
### Main synthesis script:
1. We are running the synthesis and seeing if the synthesis is pass or fail
Case 1: error encountered in hierarchy
![pic42](https://github.com/user-attachments/assets/6fa878e1-c682-46ca-b7ad-c3363f6990e2)
Case 2: No error in hierarchy
![pic43](https://github.com/user-attachments/assets/9d7a3279-47ce-47b6-be93-940c05de23c0)
### Task3: Convert format[1] and SDC to format[2] and pass to timing tool ’opentimer’
Edit Output File
1. We will remove all the  “*” in the synth.v file.
![pic44](https://github.com/user-attachments/assets/9485c4fa-5854-4c05-8072-364f75ae6f51)
2. Will open the new file which is final.synth.v in write mode and will open tmp file is read mode.
![pic46](https://github.com/user-attachments/assets/1f74c35b-b9dc-4db1-820b-3c1b645f00ff)
3. Final.synth.v file:
![pic47](https://github.com/user-attachments/assets/74a7648b-8c02-4573-a92f-7a9dcd251e49)

### PROCS:
To generate a script for OpenTimer I will be making use of procs. Procs are an external tcl file that perform an operation that is specified in it when sourced to the main tcl file. It works similar to how a function works in Python Programming. An example of a proc would be read_liberty where options like -lib, -late, -early and /or can be passed as an arguememt to the proc. Once the proc is sourced in the main tcl script the read_liberty command will be executed by referring to the proc and mapping the arguements to the external tcl script(proc script). At the end of the proc command, the main tcl script will be left with the output of the proc.

![image](https://github.com/user-attachments/assets/61c50a5a-29d8-4b56-b536-09ac646a8912)

Eg: set_multi_cpu_usage PROC  this is used for distributed operations

![image](https://github.com/user-attachments/assets/3d61730c-981f-4155-8520-14305bb6510e)

1. reopenStdout.proc
- The reopenStdout proc is a simple proc which is used to close the main terminal stdout and open a file in write mode
``` bash 
 proc reopenStdout {file} {
  #closes the main terminal window where all the puts statements were being displayed as info to user
  close stdout
  #opens $file in write mode
  open $file w       
}
 ```
![pic49](https://github.com/user-attachments/assets/79c2342c-2e01-4991-aea8-861fc1830000)

2. Set_multi_cpu_usage 

- "array set options { -localCpu <num_of_threads> -help "" }" --> set an array named options. options is a list of key-value pairs, where each key is a string representing the element's name, and each value is the corresponding value to assign to that element. eg, "-localCpu is linked to <num_of_threads>" and "-help" is linked to "".
- "switch -glob -- [lindex $args 0]" --> globbing is used to get the term inside [] so that switch can map to the corresponding case. Takes only the ket of the key-value pair
- "set args [lassign $args - options(-localCpu)]" --> assigning new value to args after removing the array element which was used to enter the loop

``` bash 
 proc set_multi_cpu_usage {args} {
    array set options {-localCpu <num_of_threads> -help "" }
    foreach {switch value} [array get options] {
    puts "Option $switch is $value"
    }
    while {[llength $args]} {
    puts "llength is [llength $args]"
    puts "lindex 0 of \"$args\" is [lindex $args 0]"
        switch -glob -- [lindex $args 0] {
          -localCpu {
              puts "old args is $args"
              set args [lassign $args - options(-localCpu)]
              puts "new args is \"$args\""
              puts "set_num_threads $options(-localCpu)"
              }
          -help {
              puts "old args is $args"
              set args [lassign $args - options(-help) ]
              puts "new args is \"$args\""
              puts "Usage: set_multi_cpu_usage -localCpu <num_of_threads>"
              }
        }
    }
}       
}
```
![pic50](https://github.com/user-attachments/assets/2d9f9a51-194e-45c7-9a53-bb5f8db43bb3)
![pic51](https://github.com/user-attachments/assets/d5d0b15d-749e-456e-bd15-ab0be546bc8d)

- Open .conf file to see the output 

![image](https://github.com/user-attachments/assets/787030aa-7296-49aa-979e-eb0a9db57f14)

- after removing all the puts commands.

![pic53](https://github.com/user-attachments/assets/3fbc9549-de05-4d65-be1a-98ebd28195d2)

3. Read_lib_PROC:
  
- Similar to the set_num_threads proc , the read_lib proc will have 3 options i.e late early and help the proc ensures to read the late and early lib file for STA and write it in a file

``` bash
  proc read_lib args {
	array set options {-late <late_lib_path> -early <early_lib_path> -help ""}
	while {[llength $args]} {
		switch -glob -- [lindex $args 0] {
		-late {
			set args [lassign $args - options(-late) ]
			puts "set_late_celllib_fpath $options(-late)"
		      }
		-early {
			set args [lassign $args - options(-early) ]
			puts "set_early_celllib_fpath $options(-early)"
		       }
		-help {
			set args [lassign $args - options(-help) ]
			puts "Usage: read_lib -late <late_lib_path> -early <early_lib_path>"
			puts "-late <provide late library path>"
			puts "-early <provide early library path>"
		      }	
		default break
		}
	}
} 
```
- It will set the paths to early and late libraries of .csv file.

![image](https://github.com/user-attachments/assets/11d0122a-779f-4c15-b6f4-4cae9e29887c)

- if we comment the reopenStdout then this will appear on the screen and there is no .conf file that is created.

![pic55](https://github.com/user-attachments/assets/d21a2f4d-7723-43b1-9427-e884cc926e7a)

4. Read_verilog_Proc:

- This proc enters the puts statement followed by the netlist file

``` bash
proc read_verilog arg1 {
  puts "set_verilog_fpath $arg1"
}
```
![image](https://github.com/user-attachments/assets/e853c8c5-3ffa-4df8-92b4-88aaacea37c9)

5. Read_SDC_Proc:

- The read_sdc proc is a large proc file which will be covered in parts. This is done to convert sdc file into OpenTimer format

``` bash
proc read_sdc {arg1} {
set sdc_dirname [file dirname $arg1]
#"file tail" is used to get the last file of the arguement given
set sdc_filename [lindex [split [file tail $arg1] .] 0 ]
#set sdc_filename [lindex [split [lindex [split $arg1 /] [expr {[llength [split $arg1 /]] -1}]] .] 0]
set sdc [open $arg1 r]
set tmp_file [open ./temp/4 "w"] 
puts -nonewline $tmp_file [string map {"\[" "" "\]" " "} [read $sdc]]     
close $tmp_file
}
```
- setting directory and filename for sdc , also replacing the " [] " with "" in a temp file

- special mapping done so that it can diffrentiate between "abc" and "abc_en". Refer the block of code.

![image](https://github.com/user-attachments/assets/4fff2126-0b5c-4c3e-96f1-1cc14765f2eb)

- converting create_clock constraints
``` bash
set tmp_file [open /tmp/4 r]
set timing_file [open /tmp/3 w]
set lines [split [read $tmp_file] "\n"]
set find_clocks [lsearch -all -inline $lines "create_clock*"]
foreach elem $find_clocks {
	set clock_port_name [lindex $elem [expr {[lsearch $elem "get_ports"]+1}]]
	set clock_period [lindex $elem [expr {[lsearch $elem "-period"]+1}]]
	set duty_cycle [expr { 100 - [expr {[lindex [lindex $elem [expr {[lsearch $elem "-waveform"]+1}]] 1]*100/$clock_period}]}]
	puts $timing_file "clock $clock_port_name $clock_period $duty_cycle"
	}
close $tmp_file
}
```
- creating clock_latency constraints
  
``` bash
set find_keyword [lsearch -all -inline $lines "set_clock_latency*"]
#puts $find_keyword
set tmp2_file [open ./temp/5 "w"]
set new_port_name ""
foreach elem $find_keyword {
        set port_name [lindex $elem [expr {[lsearch $elem "-clock"]+1}]]
        #puts "b = $port_name"
        #puts "c = $new_port_name"
	    if {![string match $new_port_name $port_name]} {
        	set new_port_name $port_name 
        	set delays_list [lsearch -all -inline $find_keyword [join [list "*" " " $port_name " " "*"] ""]]
            #puts "d = $delays_list"
        	set delay_value ""
        	foreach new_elem $delays_list {
        		set port_index [lsearch $new_elem "-clock"]
        		lappend delay_value [lindex $new_elem [expr {$port_index-1}]]
			#puts "e= $delay_value"
        	}
		puts -nonewline $tmp2_file "\nat $port_name $delay_value"
	}
}

close $tmp2_file
set tmp2_file [open ./temp/5 ]
puts -nonewline $timing_file [read $tmp2_file]
close $tmp2_file
```
- output of the script:
  
![image](https://github.com/user-attachments/assets/331c2d01-6e55-4435-81e3-11f6f1c31908)

- similarly the constraints for "set_clock_transition", "set_input_delay", "set_input_transition", "set_output_delay" and "set_load" are written in same fashion

``` bash
set ot_timing_file [open $sdc_dirname/$sdc_filename.timing w]
set timing_file [open /tmp/3 r]
while {[gets $timing_file line] != -1} {
        if {[regexp -all -- {\*} $line]} {
                set bussed [lindex [lindex [split $line "*"] 0] 1]
                set final_synth_netlist [open $sdc_dirname/$sdc_filename.final.synth.v r]
                while {[gets $final_synth_netlist line2] != -1 } {
                        if {[regexp -all -- $bussed $line2] && [regexp -all -- {input} $line2] && ![string match "" $line]} {
                        puts -nonewline $ot_timing_file "\n[lindex [lindex [split $line "*"] 0 ] 0 ] [lindex [lindex [split $line2 ";"] 0 ] 1 ] [lindex [split $line "*"] 1 ]"
                        } elseif {[regexp -all -- $bussed $line2] && [regexp -all -- {output} $line2] && ![string match "" $line]} {
                        puts -nonewline $ot_timing_file "\n[lindex [lindex [split $line "*"] 0 ] 0 ] [lindex [lindex [split $line2 ";"] 0 ] 1 ] [lindex [split $line "*"] 1 ]"
                        }
                }
        } else {
        puts -nonewline $ot_timing_file  "\n$line"
        }
}

close $timing_file
puts "set_timing_fpath $sdc_dirname/$sdc_filename.timing"
}
}
```
- Main file:

``` bash
puts "\nInfo: Timing Analysis Started ... "
puts "\nInfo: initializing number of threads, libraries, sdc, verilog netlist path..."
source /home/vsduser/vsdsynth/procs/reopenStdout.proc
source /home/vsduser/vsdsynth/procs/set_num_threads.proc
reopenStdout $OutputDirectory/$DesignName.conf
set_multi_cpu_usage -localCpu 8

source /home/vsduser/vsdsynth/procs/read_lib.proc
read_lib -early /home/vsduser/vsdsynth/osu018_stdcells.lib

read_lib -late /home/vsduser/vsdsynth/osu018_stdcells.lib

source /home/vsduser/vsdsynth/procs/read_verilog.proc
read_verilog $OutputDirectory/$DesignName.final.synth.v

source /home/vsduser/vsdsynth/procs/read_sdc.proc
read_sdc $OutputDirectory/$DesignName.sdc
reopenStdout /dev/tty
}

```
6. Creating the spef file and config file

``` bash
if {$enable_prelayout_timing == 1} {
	puts "\nInfo: enable prelayout_timing is $enable_prelayout_timing. Enabling zero-wire load parasitics"
	set spef_file [open $OutputDirectory/$DesignName.spef w]
	puts $spef_file "*SPEF \"IEEE 1481-1998\""
	puts $spef_file "*DESIGN \"$DesignName\""
	puts $spef_file "*DATE \"Sun Jun 11 11:59:00 2023\""
	puts $spef_file "*VENDOR \"VLSI System Design\""
	puts $spef_file "*PROGRAM \"TCL Workshop\""
	puts $spef_file "*DATE \"0.0\""
	puts $spef_file "*DESIGN FLOW \"NETLIST_TYPE_VERILOG\""
	puts $spef_file "*DIVIDER /"
	puts $spef_file "*DELIMITER : "
	puts $spef_file "*BUS_DELIMITER [ ]"
	puts $spef_file "*T_UNIT 1 PS"
	puts $spef_file "*C_UNIT 1 FF"
	puts $spef_file "*R_UNIT 1 KOHM"
	puts $spef_file "*L_UNIT 1 UH"
}
close $spef_file

set conf_file [open $OutputDirectory/$DesignName.conf a]
puts $conf_file "set_spef_fpath $OutputDirectory/$DesignName.spef"
puts $conf_file "init_timer"
puts $conf_file "report_timer"
puts $conf_file "report_wns"
puts $conf_file "report_tns"
puts $conf_file "report_worst_paths -numPaths 10000 " 
close $conf_file
}
```
![pic59](https://github.com/user-attachments/assets/e7f8f872-b683-4ab0-a083-6205cc971579)
![image](https://github.com/user-attachments/assets/ea301503-5d66-4614-86b9-3cbfb3e906b0)

### Quality of Results(QOR) generation:

- Running sta analysis:
  
 ``` bash 
 set tcl_precision 3
set time_elapsed_in_us [time {exec /home/vsduser/OpenTimer-1.0.5/bin/OpenTimer < $OutputDirectory/$DesignName.conf >& $OutputDirectory/$DesignName.results} 1]
puts "time_elapsed_in_us is $time_elapsed_in_us"
set time_elapsed_in_sec "[expr {[lindex $time_elapsed_in_us 0]/100000}]sec"
#puts "time_elapsed_in_sec is $time_elapsed_in_sec"
puts "\nInfo: STA finished in $time_elapsed_in_sec seconds"      
}
```
- outputs of STA analysis from the .results file:

 ``` bash 
 #-------------------------find worst output violation--------------------------------#
set worst_RAT_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r]
set pattern {RAT}
while {[gets $report_file line] != -1} {
	if {[regexp $pattern $line]} {
		set worst_RAT_slack "[expr {[lindex $line 3]/1000}]ns"
		break
	} else {
		continue
	}
}
close $report_file

#-------------------------find number of output violations--------------------------------#	
set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
	incr count [regexp -all -- $pattern $line]
}
set Number_output_violations $count
close $report_file

#-------------------------find worst setup violation--------------------------------#
set worst_negative_setup_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r] 
set pattern {Setup}
while {[gets $report_file line] != -1} {
	if {[regexp $pattern $line]} {
		set worst_negative_setup_slack "[expr {[lindex $line 3]/1000}]ns"
		break
	} else {
		continue
	}
}
close $report_file

#-------------------------find number of setup violations--------------------------------#
set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
	incr count [regexp -all -- $pattern $line]
}
set Number_of_setup_violations $count
close $report_file

#-------------------------find worst hold violation--------------------------------#
set worst_negative_hold_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r] 
set pattern {Hold}
while {[gets $report_file line] != -1} {
	if {[regexp $pattern $line]} {
		set worst_negative_hold_slack "[expr {[lindex $line 3]/1000}]ns"
		break
	} else {
		continue
	}
}
close $report_file

#-------------------------find number of hold violations--------------------------------#
set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
	incr count [regexp -all -- $pattern $line]
}
set Number_of_hold_violations $count
close $report_file

#-------------------------find number of instances--------------------------------#

set pattern {Num of gates}
set report_file [open $OutputDirectory/$DesignName.results r] 
while {[gets $report_file line] != -1} {
	if {[regexp $pattern $line]} {
		set Instance_count "[lindex [join $line " "] 4 ]"
		break
	} else {
		continue
	}
}
close $report_file    
}

```
1. output data is taken from .results file.
   
![pic62](https://github.com/user-attachments/assets/77d2904c-966b-408f-a970-bb2fa7072b9d)

``` bash 
 puts "\n"
puts "						****PRELAYOUT TIMING RESULTS**** 					"
set formatStr "%15s %15s %15s %15s %15s %15s %15s %15s %15s"

puts [format $formatStr "----------" "-------" "--------------" "---------" "---------" "--------" "--------" "-------" "-------"]
puts [format $formatStr "DesignName" "Runtime" "Instance Count" "WNS Setup" "FEP Setup" "WNS Hold" "FEP Hold" "WNS RAT" "FEP RAT"]
puts [format $formatStr "----------" "-------" "--------------" "---------" "---------" "--------" "--------" "-------" "-------"]
foreach design_name $DesignName runtime $time_elapsed_in_sec instance_count $Instance_count wns_setup $worst_negative_setup_slack fep_setup $Number_of_setup_violations wns_hold $worst_negative_hold_slack fep_hold $Number_of_hold_violations wns_rat $worst_RAT_slack fep_rat $Number_output_violations {
	puts [format $formatStr $design_name $runtime $instance_count $wns_setup $fep_setup $wns_hold $fep_hold $wns_rat $fep_rat]
}

puts [format $formatStr "----------" "-------" "--------------" "---------" "---------" "--------" "--------" "-------" "-------"]
puts "\n"

```

2. Report Formatting.
   
![pic63](https://github.com/user-attachments/assets/7c2cf0e7-1ff7-4fca-80ec-5709dee0d0fa)

## Conclusion
- All tasks for the Tcl workshop have been completed
- All simulation errors were resolved.






