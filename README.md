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
1.	The rectangle area which is highlighted we can get by using the code below and by executing the code we will get the early rise_delay and input ports start we will get the boundaries of that rectangle box
 ``` bash 
 set clock_start [lindex [lindex [constraints search all CLOCKS] 0] 1] 
set clock_start_column [lindex [lindex [constraints search all CLOCKS] 0 ] 0] 
set clock_period_start [lindex [lindex [constraints search all frequency] 0] 0] 
set duty_cycle_start [lindex [lindex [constraints search all duty_cycle] 0] 0] 
 ```
2.	We are opening the openMSp430.sdc in write mode by using the command set sdc_file [open $outputDirectory/$DesignName.sdc “w” now we will take the various values from the csv file and write it to sdc constraints which has /port/ signal /variable names picked from the excel file and corresponding values of the constraints. The csv file is input and the sdc constraints are the output.
- clock creation:
![pic18](https://github.com/user-attachments/assets/2f7fba7e-469a-46cb-abec-de3a5dd61eb2)
![pic19](https://github.com/user-attachments/assets/5341b324-f602-4597-bbc3-bb81b5246312)
- Algorithm to categorize input ports as bits and bussed
i.	In the inputs ports there are multibit and single bit ports, and the multibit ports are called bus for eg: dbg_i2c_addr and dbg_i2c_broadcast 
ii.	If there is an n bit bus, then put “*” in front of port else if it single bit then keep the port as it is. 
iii.	Related clock is the expression we follow to get that rectangular box. In that rectangular box we need to find all the keywords which are presented in each column.
iv.	 We will get the input port for eg.cpu_en &dbg_i2c_addr=>$pattern1 and will Open the each .V file and grep for $pattern1 in each file then split the string which you get in output using “;” as a delimiter and get the index[0] for that we need to grep the input in patterns generated by above step and then replace multiple spaces by single space in order to do that I need to count no of elements in the string and save it as $count then use the condition like if the $count is grater that 2 then put “* ” in front of the port  else keep the port as it is.
![pic21](https://github.com/user-attachments/assets/14ec62e5-8650-4a0c-b62d-b8473d0f7937)
v.	Now investigate the script below which will explain how we can get one input without any spaces eg: “input cpu_en;”
![pic20](https://github.com/user-attachments/assets/a3677e67-a0f2-48f6-a7a7-df71a21082a0)
vi.	and after grep for replace multiple places, we go the output file.
![pic22](https://github.com/user-attachments/assets/cb2b21b2-2b57-4b74-8bba-850dd020461b)
vii. We need to open the tmp1 in read mode and then open the tmp2 file in write mode where we can dump the files. Then we will split the elements in tmp1 using the new line and then uniquify and sort the elements and then join the elements and will write in tmp2 file this will allow us to remove the duplicate elements. Close tmp1 and tmp2 file and again open the tmp2 file in read mode the count the length of the element if it’s greater than 2 then put “*” in front of that port the
![pic23](https://github.com/user-attachments/assets/1ec87b12-3b06-43a0-b5f1-2b29924dee4c)
![pic24](https://github.com/user-attachments/assets/1216c03a-1209-4ed3-9df8-3e86d3b64444)
-splitting:
![pic25](https://github.com/user-attachments/assets/ac3bbbfd-4693-4937-8044-15da6f8131aa)
-sorting:
![pic26](https://github.com/user-attachments/assets/f1fd7fdb-929e-4435-8c22-3635220852cd)
-joining:
![pic27](https://github.com/user-attachments/assets/924f7118-a6e4-47a1-b008-f216c8de02f3)
viii.	Finally, all these constraints will be saved in sdc file which will show the constraints in file name  $outputDirectory/$DesignName.sdc after the code is fully executed
![pic29](https://github.com/user-attachments/assets/38eac404-2ce6-4927-8a8a-0a552dd2109a)
-A snip of the sdc
![pic30](https://github.com/user-attachments/assets/f0fea9a2-d630-4050-ab50-f661c8be20c8)
file pic31
- Constraints generation logic for the output port and conclusion!!
The script is exactly same as the input, but the only difference is we are replacing the input with the output.
![pic32](https://github.com/user-attachments/assets/f41e8ed8-e6fe-42a7-91c8-b168ff30309c)

## Day4: Completing Scripting and Yosys Synthesis Introduction
### Yosys synthesis toolbox: 
take the RTL and put that in yosys synthesis toolbox will execute the output. The tool will read all the Verilog file will provide the synthesis Verilog netlist as the output 
i.	We will create a hierarchy check script to be used by yosys pic33 show the code for the hierarchy and after running the code will the output shown in pic 34 when we open the file it looks like the below image pic 35
Error handling in hierarchy check:
i.	Checking whether the top-level hierarchy is being well connected with all hierarchy which is present file if not which module is missing
ii.	If any module not found error flag is set to 1, If all modules found error flag is set to 0 pic 36
iii.	If we change the module name to omsp_clock_mocule to omsp_clock_mocule_vsd, see what the flag value is pic37 and we can see the error in log file pic38 we can remove the error in the Verilog file and will run again then the flag is 0 and we did not see any error in hierarchy log file 
iv.	We will check if the hierarchy passes or fail the code is pic 39 the output pic 40 and pic 41

DAY 5:
Main synthesis script:
i.	We are running the synthesis and seeing if the synthesis is passing or failing based on the hierarchy PIC43,42

3.	Convert format[1] and SDC to format[2] and pass to timing tool ’opentimer’
Edit Output File
I.	We will remove all the  “*” in the synth.v file  pic44
II.	Will open the new file which is final.synth.v in write mode and will open tmp file is read mode pic46 pic 47 is the final.synth.v
PROCS:
I.	It is used to create commands in EDA tools in TCL you can create procname once and will use it multiple times in the overall code 
Eg: set_multi_cpu_usage PROC  this is used for distributed operations
Pic48 is code which will provide the length of the $args and pic49 is output of procs
II.	Set_multi_cpu_usage 
Pic50,51 for output
Open .conf file to see the output pic52 after removing all the puts commands then the output is pic53
III.	Read_lib_PROC:
It will set the paths to early and late libraries of .csv file pic54 if we comment the reopenStdout then this will appear on the screen and there is no .conf file that is created pic 55 
IV.	Read_verilog_Proc
Pic56 is the output, and the code is given below and pic57 is code 
V.	Read_SDC_Proc
We will get rid of the square brackets 

VI.	.conf file creation
Master file that is supplied to opentimer tool to do STA the code to create a spec file is given in pic58 the output is pic59 the spef file looks like this pic60 and conf file is pic61
Quality of Results(QOR)generation
I.	Runtime(pic62)
II.	Report Formatting (pic63)




## Day5: Advanced Scripting Techniques and Quality of Results Generation

