Download Link: https://assignmentchef.com/product/solved-cmu15213-lab-1-understanding-buffer-overflow-bugs
<br>
<span class="kksr-muted">Rate this product</span>

1 Introduction

This assignment involves generating a total of five attacks on two programs having different security vul- nerabilities. Outcomes you will gain from this lab include:

<ul>

 <li>You will learn different ways that attackers can exploit security vulnerabilities when programs do not safeguard themselves well enough against buffer overflows.</li>

 <li>Through this, you will get a better understanding of how to write programs that are more secure, as well as some of the features provided by compilers and operating systems to make programs less vulnerable.</li>

 <li>You will gain a deeper understanding of the stack and parameter-passing mechanisms of x86-64 machine code.</li>

 <li>You will gain a deeper understanding of how x86-64 instructions are encoded.</li>

 <li>You will gain more experience with debugging tools such as GDB and OBJDUMP.Note: In this lab, you will gain firsthand experience with methods used to exploit security weaknesses in operating systems and network servers. Our purpose is to help you learn about the runtime operation of programs and to understand the nature of these security weaknesses so that you can avoid them when you write system code. We do not condone the use of any other form of attack to gain unauthorized access to any system resources.You will want to study Sections 3.10.3 and 3.10.4 of the CS:APP3e book as reference material for this lab.</li>

</ul>

1

2 Logistics

As usual, this is an individual project. You will generate attacks for target programs that are custom gener- ated for you.

2.1 Getting Files

You can obtain your files by pointing your Web browser at:

<pre>    http://$Attacklab::SERVER_NAME:15513/</pre>

<pre>INSTRUCTOR: $Attacklab::SERVER_NAME is the machine that runs the attacklab servers. You define it in attacklab/Attacklab.pm and in attacklab/src/build/driverhdrs.h</pre>

The server will build your files and return them to your browser in a tar file called targetk.tar, where k is the unique number of your target programs.

Note: It takes a few seconds to build and download your target, so please be patient.

Save the targetk.tar file in a (protected) Linux directory in which you plan to do your work. Then give the command: tar -xvf targetk.tar. This will extract a directory targetk containing the files described below.

You should only download one set of files. If for some reason you download multiple targets, choose one target to work on and delete the rest.

Warning: If you expand your targetk.tar on a PC, by using a utility such as Winzip, or letting your browser do the extraction, you’ll risk resetting permission bits on the executable files.

The files in targetk include:

README.txt: A file describing the contents of the directory

ctarget: An executable program vulnerable to code-injection attacks

rtarget: An executable program vulnerable to return-oriented-programming attacks

cookie.txt: An 8-digit hex code that you will use as a unique identifier in your attacks.

farm.c: The source code of your target’s “gadget farm,” which you will use in generating return-oriented programming attacks.

hex2raw: A utility to generate attack strings.In the following instructions, we will assume that you have copied the files to a protected local directory,

and that you are executing the programs in that local directory.

2

2.2 Important Points

Here is a summary of some important rules regarding valid solutions for this lab. These points will not make much sense when you read this document for the first time. They are presented here as a central reference of rules once you get started.

3

<ul>

 <li>You must do the assignment on a machine that is similar to the one that generated your targets.</li>

 <li>Your solutions may not use attacks to circumvent the validation code in the programs. Specifically, any address you incorporate into an attack string for use by a ret instruction should be to one of the following destinations:– The addresses for functions touch1, touch2, or touch3. – The address of your injected code– The address of one of your gadgets from the gadget farm.</li>

 <li>You may only construct gadgets from file rtarget with addresses ranging between those for func- tions start_farm and end_farm.Target Programs</li>

</ul>

Both CTARGET and RTARGET read strings from standard input. They do so with the function getbuf defined below:

1 unsigned getbuf() 2{

<ol start="3">

 <li>3  char buf[BUFFER_SIZE];</li>

 <li>4  Gets(buf);</li>

 <li>5  return 1;</li>

</ol>

6}

The function Gets is similar to the standard library function gets—it reads a string from standard input (terminated by ‘
’ or end-of-file) and stores it (along with a null terminator) at the specified destination. In this code, you can see that the destination is an array buf, declared as having BUFFER_SIZE bytes. At the time your targets were generated, BUFFER_SIZE was a compile-time constant specific to your version of the programs.

Functions Gets() and gets() have no way to determine whether their destination buffers are large enough to store the string they read. They simply copy sequences of bytes, possibly overrunning the bounds of the storage allocated at the destinations.

If the string typed by the user and read by getbuf is sufficiently short, it is clear that getbuf will return 1, as shown by the following execution examples:

unix&gt; ./ctarget

3

Cookie: 0x1a7dd803Type string: Keep it short!No exploit. Getbuf returned 0x1 Normal return

Typically an error occurs if you type a long string:

unix&gt; ./ctargetCookie: 0x1a7dd803Type string: This is not a very interesting string, but it has the property … Ouch!: You caused a segmentation fault!Better luck next time

(Note that the value of the cookie shown will differ from yours.) Program RTARGET will have the same behavior. As the error message indicates, overrunning the buffer typically causes the program state to be corrupted, leading to a memory access error. Your task is to be more clever with the strings you feed CTARGET and RTARGET so that they do more interesting things. These are called exploit strings.

Both CTARGET and RTARGET take several different command line arguments:

-h: Print list of possible command line arguments-q: Don’t send results to the grading server-i FILE: Supply input from a file, rather than from standard input

Your exploit strings will typically contain byte values that do not correspond to the ASCII values for printing characters. The program HEX2RAW will enable you to generate these raw strings. See Appendix A for more information on how to use HEX2RAW.

Important points:

• Your exploit string must not contain byte value 0x0a at any intermediate position, since this is the ASCII code for newline (‘
’). When Gets encounters this byte, it will assume you intended to terminate the string.

• HEX2RAW expects two-digit hex values separated by one or more white spaces. So if you want to create a byte with a hex value of 0, you need to write it as 00. To create the word 0xdeadbeef you should pass “ef be ad de” to HEX2RAW (note the reversal required for little-endian byte ordering).

When you have correctly solved one of the levels, your target program will automatically send a notification to the grading server. For example:

unix&gt; ./hex2raw &lt; ctarget.l2.txt | ./ctargetCookie: 0x1a7dd803Type string:Touch2!: You called touch2(0x1a7dd803) Valid solution for level 2 with target ctargetPASSED: Sent exploit string to server to be validated. NICE JOB!

4

<table>

 <tbody>

  <tr>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

  </tr>

 </tbody>

</table>

Phase

Program

Level

Method

Function

Points

1 2 3

CTARGET CTARGET CTARGET

1 2 3

CI CI CI

<pre>touch1touch2touch3</pre>

10 25 25

4 5

RTARGET RTARGET

2 3

ROP ROP

<pre>touch2touch3</pre>

35 5

CI: Code injectionROP: Return-oriented programming

Figure 1: Summary of attack lab phases

The server will test your exploit string to make sure it really works, and it will update the Attacklab score- board page indicating that your userid (listed by your target number for anonymity) has completed this phase.

You can view the scoreboard by pointing your Web browser at

<pre>    http://$Attacklab::SERVER_NAME:15513/scoreboard</pre>

Unlike the Bomb Lab, there is no penalty for making mistakes in this lab. Feel free to fire away at CTARGET and RTARGET with any strings you like.

IMPORTANT NOTE: You can work on your solution on any Linux machine, but in order to submit your solution, you will need to be running on one of the following machines:

<pre>INSTRUCTOR: Insert the list of the legal domain names that you established in buflab/src/config.c.</pre>

Figure 1 summarizes the five phases of the lab. As can be seen, the first three involve code-injection (CI) attacks on CTARGET, while the last two involve return-oriented-programming (ROP) attacks on RTARGET.

4 Part I: Code Injection Attacks

For the first three phases, your exploit strings will attack CTARGET. This program is set up in a way that the stack positions will be consistent from one run to the next and so that data on the stack can be treated as executable code. These features make the program vulnerable to attacks where the exploit strings contain the byte encodings of executable code.

4.1 Level 1

For Phase 1, you will not inject new code. Instead, your exploit string will redirect the program to execute an existing procedure.

Function getbuf is called within CTARGET by a function test having the following C code: 5

1 void test() 2{

<ol start="3">

 <li>3  int val;</li>

 <li>4  val = getbuf();</li>

 <li>5  printf(“No exploit. Getbuf returned 0x%x
”, val);</li>

</ol>

6}

When getbuf executes its return statement (line 5 of getbuf), the program ordinarily resumes execution within function test (at line 5 of this function). We want to change this behavior. Within the file ctarget, there is code for a function touch1 having the following C representation:

1 void touch1() 2{

<ol start="3">

 <li>3  vlevel = 1; /* Part of validation protocol */</li>

 <li>4  printf(“Touch1!: You called touch1()
”);</li>

 <li>5  validate(1);</li>

 <li>6  exit(0);</li>

</ol>

7}

Your task is to get CTARGET to execute the code for touch1 when getbuf executes its return statement, rather than returning to test. Note that your exploit string may also corrupt parts of the stack not directly related to this stage, but this will not cause a problem, since touch1 causes the program to exit directly.

Some Advice:

4.2

• •

• •

•

All the information you need to devise your exploit string for this level can be determined by exam- iningadisassembledversionofCTARGET.Useobjdump -dtogetthisdissembledversion.

The idea is to position a byte representation of the starting address for touch1 so that the ret instruction at the end of the code for getbuf will transfer control to touch1.

Be careful about byte ordering.

You might want to use GDB to step the program through the last few instructions of getbuf to make sure it is doing the right thing.

The placement of buf within the stack frame for getbuf depends on the value of compile-time constant BUFFER_SIZE, as well the allocation strategy used by GCC. You will need to examine the disassembled code to determine its position.

Level 2

Phase 2 involves injecting a small amount of code as part of your exploit string.Within the file ctarget there is code for a function touch2 having the following C representation:

1 void touch2(unsigned val)

6

4.3

2{ 345

6 7 8 9

10 11 12

<pre>vlevel = 2;       /* Part of validation protocol */if (val == cookie) {</pre>

•

• •

•

•

You will want to position a byte representation of the address of your injected code in such a way that ret instruction at the end of the code for getbuf will transfer control to it.

Recall that the first argument to a function is passed in register %rdi.Your injected code should set the register to your cookie, and then use a ret instruction to transfer

control to the first instruction in touch2.

Do not attempt to use jmp or call instructions in your exploit code. The encodings of destination addresses for these instructions are difficult to formulate. Use ret instructions for all transfers of control, even when you are not returning from a call.

See the discussion in Appendix B on how to use tools to generate the byte-level representations of instruction sequences.

Level 3

<pre>    printf("Touch2!:</pre>

validate(2); }else{

<pre>    printf("Misfire:</pre>

fail(2); }

<pre>You called touch2(0x%.8x)
", val);</pre>

<pre>You called touch2(0x%.8x)
", val);</pre>

exit(0); }

Your task is to get CTARGET to execute the code for touch2 rather than returning to test. In this case, however, you must make it appear to touch2 as if you have passed your cookie as its argument.

Some Advice:

Phase 3 also involves a code injection attack, but passing a string as argument.

Within the file ctarget there is code for functions hexmatch and touch3 having the following C representations:

1 /* Compare string to hex represention of unsigned value */

2 int 3{45

678 9}

<pre>hexmatch(unsigned val, char *sval)</pre>

<pre>char cbuf[110];/* Make position of check string unpredictable */char *s = cbuf + random() % 100;sprintf(s, "%.8x", val);return strncmp(sval, s, 9) == 0;</pre>

7

5

consist of the eight hexadecimal digits (ordered from most to least significant) without a leading “0x.”

<ul>

 <li>Recall that a string is represented in C as a sequence of bytes followed by a byte with value 0. Type“man ascii”onanyLinuxmachinetoseethebyterepresentationsofthecharactersyouneed.</li>

 <li>Your injected code should set register %rdi to the address of this string.</li>

 <li>When functions hexmatch and strncmp are called, they push data onto the stack, overwriting portions of memory that held the buffer used by getbuf. As a result, you will need to be careful where you place the string representation of your cookie.Part II: Return-Oriented Programming</li>

</ul>

<pre>10111213141516171819202122</pre>

<pre>void touch3(char *sval){</pre>

<pre>    vlevel = 3;       /* Part of validation protocol */    if (hexmatch(cookie, sval)) {</pre>

<pre>        printf("Touch3!: You called touch3("%s")
", sval);</pre>

<pre>        validate(3);    } else {</pre>

<pre>        printf("Misfire: You called touch3("%s")
", sval);</pre>

fail(3); }

exit(0); }

Your task is to get CTARGET to execute the code for touch3 rather than returning to test. You must make it appear to touch3 as if you have passed a string representation of your cookie as its argument.

Some Advice:• Youwillneedtoincludeastringrepresentationofyourcookieinyourexploitstring.Thestringshould

Performing code-injection attacks on program RTARGET is much more difficult than it is for CTARGET, because it uses two techniques to thwart such attacks:

<ul>

 <li>It uses randomization so that the stack positions differ from one run to another. This makes it impos- sible to determine where your injected code will be located.</li>

 <li>It marks the section of memory holding the stack as nonexecutable, so even if you could set the program counter to the start of your injected code, the program would fail with a segmentation fault.Fortunately, clever people have devised strategies for getting useful things done in a program by executing existing code, rather than injecting new code. The most general form of this is referred to as return-oriented programming (ROP) [1, 2]. The strategy with ROP is to identify byte sequences within an existing program that consist of one or more instructions followed by the instruction ret. Such a segment is referred to as a</li>

</ul>

8

<table>

 <tbody>

  <tr>

   <td></td>

  </tr>

  <tr>

   <td></td>

  </tr>

  <tr>

   <td></td>

  </tr>

  <tr>

   <td></td>

  </tr>

 </tbody>

</table>

Stack

  

%rsp

Gadget n code

Gadget 2 code

Gadget 1 code

c3

c3

c3

Figure 2: Setting up sequence of gadgets for execution. Byte value 0xc3 encodes the ret instruction.

gadget. Figure 2 illustrates how the stack can be set up to execute a sequence of n gadgets. In this figure, the stack contains a sequence of gadget addresses. Each gadget consists of a series of instruction bytes, with the final one being 0xc3, encoding the ret instruction. When the program executes a ret instruction starting with this configuration, it will initiate a chain of gadget executions, with the ret instruction at the end of each gadget causing the program to jump to the beginning of the next.

A gadget can make use of code corresponding to assembly-language statements generated by the compiler, especially ones at the ends of functions. In practice, there may be some useful gadgets of this form, but not enough to implement many important operations. For example, it is highly unlikely that a compiled function would have popq %rdi as its last instruction before ret. Fortunately, with a byte-oriented instruction set, such as x86-64, a gadget can often be found by extracting patterns from other parts of the instruction byte sequence.

For example, one version of rtarget contains code generated for the following C function:

<pre>void setval_210(unsigned *p){</pre>

<pre>    *p = 3347663060U;}</pre>

The chances of this function being useful for attacking a system seem pretty slim. But, the disassembled machine code for this function shows an interesting byte sequence:

<pre>0000000000400f15 &lt;setval_210&gt;:  400f15:       c7 07 d4 48 89 c7       movl   $0xc78948d4,(%rdi)  400f1b:       c3                      retq</pre>

The byte sequence 48 89 c7 encodes the instruction movq %rax, %rdi. (See Figure 3A for the encodings of useful movq instructions.) This sequence is followed by byte value c3, which encodes the ret instruction. The function starts at address 0x400f15, and the sequence starts on the fourth byte of the function. Thus, this code contains a gadget, having a starting address of 0x400f18, that will copy the 64-bit value in register %rax to register %rdi.

9

Your code for RTARGET contains a number of functions similar to the setval_210 function shown above in a region we refer to as the gadget farm. Your job will be to identify useful gadgets in the gadget farm and use these to perform attacks similar to those you did in Phases 2 and 3.

Important: The gadget farm is demarcated by functions start_farm and end_farm in your copy of rtarget. Do not attempt to construct gadgets from other portions of the program code.

5.1 Level 2

For Phase 4, you will repeat the attack of Phase 2, but do so on program RTARGET using gadgets from your gadget farm. You can construct your solution using gadgets consisting of the following instruction types, and using only the first eight x86-64 registers (%rax–%rdi).

movq : The codes for these are shown in Figure 3A.

popq : The codes for these are shown in Figure 3B.

ret : This instruction is encoded by the single byte 0xc3.

nop : This instruction (pronounced “no op,” which is short for “no operation”) is encoded by the single byte 0x90. Its only effect is to cause the program counter to be incremented by 1.

Some Advice:

5.2

•

• •

All the gadgets you need can be found in the region of the code for rtarget demarcated by the functions start_farm and mid_farm.

You can do this attack with just two gadgets.

When a gadget uses a popq instruction, it will pop data from the stack. As a result, your exploit string will contain a combination of gadget addresses and data.

Level 3

Before you take on the Phase 5, pause to consider what you have accomplished so far. In Phases 2 and 3, you caused a program to execute machine code of your own design. If CTARGET had been a network server, you could have injected your own code into a distant machine. In Phase 4, you circumvented two of the main devices modern systems use to thwart buffer overflow attacks. Although you did not inject your own code, you were able inject a type of program that operates by stitching together sequences of existing code. You have also gotten 95/100 points for the lab. That’s a good score. If you have other pressing obligations consider stopping right now.

Phase 5 requires you to do an ROP attack on RTARGET to invoke function touch3 with a pointer to a string representation of your cookie. That may not seem significantly more difficult than using an ROP attack to invoke touch2, except that we have made it so. Moreover, Phase 5 counts for only 5 points, which is not a true measure of the effort it will require. Think of it as more an extra credit problem for those who want to go beyond the normal expectations for the course.

10

<table>

 <tbody>

  <tr>

   <td colspan="1" rowspan="2"></td>

   <td colspan="8" rowspan="1"></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

  </tr>

 </tbody>

</table>

<table>

 <tbody>

  <tr>

   <td colspan="1" rowspan="2"></td>

   <td colspan="8" rowspan="1"></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

  </tr>

 </tbody>

</table>

<table>

 <tbody>

  <tr>

   <td colspan="1" rowspan="2"></td>

   <td colspan="8" rowspan="1"></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

  </tr>

 </tbody>

</table>

<table>

 <tbody>

  <tr>

   <td colspan="1" rowspan="2"></td>

   <td colspan="4" rowspan="1"></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

   <td></td>

  </tr>

 </tbody>

</table>

A. Encodings of movq instructions movq S, D

Source

S

Destination D

%rax

%rcx

%rdx

%rbx

%rsp

%rbp

%rsi

%rdi

<pre>%rax%rcx%rdx%rbx%rsp%rbp%rsi%rdi</pre>

<pre>48 89 c048 89 c848 89 d048 89 d848 89 e048 89 e848 89 f048 89 f8</pre>

<pre>48 89 c148 89 c948 89 d148 89 d948 89 e148 89 e948 89 f148 89 f9</pre>

<pre>48 89 c248 89 ca48 89 d248 89 da48 89 e248 89 ea48 89 f248 89 fa</pre>

<pre>48 89 c348 89 cb48 89 d348 89 db48 89 e348 89 eb48 89 f348 89 fb</pre>

<pre>48 89 c448 89 cc48 89 d448 89 dc48 89 e448 89 ec48 89 f448 89 fc</pre>

<pre>48 89 c548 89 cd48 89 d548 89 dd48 89 e548 89 ed48 89 f548 89 fd</pre>

<pre>48 89 c648 89 ce48 89 d648 89 de48 89 e648 89 ee48 89 f648 89 fe</pre>

<pre>48 89 c748 89 cf48 89 d748 89 df48 89 e748 89 ef48 89 f748 89 ff</pre>

B. Encodings of popq instructions

C. Encodings of movl instructions movl S, D

Operation

Register R

%rax

%rcx

%rdx

%rbx

%rsp

%rbp

%rsi

%rdi

popq R

58

59

5a

5b

5c

5d

5e

5f

Source

S

Destination D

%eax

%ecx

%edx

%ebx

%esp

%ebp

%esi

%edi

<pre>%eax%ecx%edx%ebx%esp%ebp%esi%edi</pre>

<pre>89 c089 c889 d089 d889 e089 e889 f089 f8</pre>

<pre>89 c189 c989 d189 d989 e189 e989 f189 f9</pre>

<pre>89 c289 ca89 d289 da89 e289 ea89 f289 fa</pre>

<pre>89 c389 cb89 d389 db89 e389 eb89 f389 fb</pre>

<pre>89 c489 cc89 d489 dc89 e489 ec89 f489 fc</pre>

<pre>89 c589 cd89 d589 dd89 e589 ed89 f589 fd</pre>

<pre>89 c689 ce89 d689 de89 e689 ee89 f689 fe</pre>

<pre>89 c789 cf89 d789 df89 e789 ef89 f789 ff</pre>

D. Encodings of 2-byte functional nop instructions

Figure 3: Byte encodings of instructions. All values are shown in hexadecimal.

Operation

Register R

%al

%cl

%dl

%bl

andb R, R orb R, R cmpb R, R testb R, R

<pre>20 c008 c038 c084 c0</pre>

<pre>20 c908 c938 c984 c9</pre>

<pre>20 d208 d238 d284 d2</pre>

<pre>20 db08 db38 db84 db</pre>

11

To solve Phase 5, you can use gadgets in the region of the code in rtarget demarcated by functions start_farm and end_farm. In addition to the gadgets used in Phase 4, this expanded farm includes the encodings of different movl instructions, as shown in Figure 3C. The byte sequences in this part of the farm also contain 2-byte instructions that serve as functional nops, i.e., they do not change any register or memoryvalues.Theseincludeinstructions,showninFigure3D,suchasandb %al,%al,thatoperateon the low-order bytes of some of the registers but do not change their values.

Some Advice:

• You’ll want to review the effect a movl instruction has on the upper 4 bytes of a register, as is described on page 183 of the text.

• The official solution requires eight gadgets (not all of which are unique). Good luck and have fun!

A Using HEX2RAW

HEX2RAW takes as input a hex-formatted string. In this format, each byte value is represented by two hex digits. For example, the string “012345” could be entered in hex format as “30 31 32 33 34 35 00.” (Recall that the ASCII code for decimal digit x is 0x3x, and that the end of a string is indicated by a null byte.)

The hex characters you pass to HEX2RAW should be separated by whitespace (blanks or newlines). We recommend separating different parts of your exploit string with newlines while you’re working on it. HEX2RAW supports C-style block comments, so you can mark off sections of your exploit string. For example:

48 c7 c1 f0 11 40 00 /* mov $0x40011f0,%rcx */Be sure to leave space around both the starting and ending comment strings (“/*”, “*/”), so that the

comments will be properly ignored.

If you generate a hex-formatted exploit string in the file exploit.txt, you can apply the raw string to CTARGET or RTARGET in several different ways:

1. You can set up a series of pipes to pass the string through HEX2RAW.

unix&gt; cat exploit.txt | ./hex2raw | ./ctarget 2. You can store the raw string in a file and use I/O redirection:

unix&gt; ./hex2raw &lt; exploit.txt &gt; exploit-raw.txt unix&gt; ./ctarget &lt; exploit-raw.txt

This approach can also be used when running from within GDB: 12

B

unix&gt; gdb ctarget(gdb) run &lt; exploit-raw.txt

3. You can store the raw string in a file and provide the file name as a command-line argument:

unix&gt; ./hex2raw &lt; exploit.txt &gt; exploit-raw.txt unix&gt; ./ctarget -i exploit-raw.txt

This approach also can be used when running from within GDB.

Generating Byte Codes

Using GCC as an assembler and OBJDUMP as a disassembler makes it convenient to generate the byte codes for instruction sequences. For example, suppose you write a file example.s containing the following assembly code:

<pre># Example of hand-generated assembly code</pre>

<pre>pushq   $0xabcdefaddq    $17,%raxmovl    %eax,%edx</pre>

<pre># Push value onto stack# Add 17 to %rax# Copy lower 32 bits to %edx</pre>

The code can contain a mixture of instructions and data. Anything to the right of a ‘#’ character is a comment.

You can now assemble and disassemble this file:

unix&gt; gcc -c example.sunix&gt; objdump -d example.o &gt; example.d

The generated file example.d contains the following: example.o: file format elf64-x86-64

<pre>Disassembly of section .text:</pre>

<pre>0000000000000000 &lt;.text&gt;:   0: 68 ef cd ab 00   5: 48 83 c0 11   9: 89 c2</pre>

<pre>pushq  $0xabcdefadd    $0x11,%raxmov    %eax,%edx</pre>

The lines at the bottom show the machine code generated from the assembly language instructions. Each line has a hexadecimal number on the left indicating the instruction’s starting address (starting with 0), while

13

the hex digits after the ‘:’ character indicate the byte codes for the instruction. Thus, we can see that the instructionpush $0xABCDEFhashex-formattedbytecode68 ef cd ab 00.

From this file, you can get the byte sequence for the code:

<pre>68 ef cd ab 00 48 83 c0 11 89 c2</pre>

This string can then be passed through HEX2RAW to generate an input string for the target programs.. Alter- natively, you can edit example.d to omit extraneous values and to contain C-style comments for readability, yielding:

<pre>   68 ef cd ab 00   /* pushq  $0xabcdef  */   48 83 c0 11      /* add    $0x11,%rax */   89 c2            /* mov    %eax,%edx  */</pre>

This is also a valid input you can pass through HEX2RAW before sending to one of the target programs.

References

[1] R. Roemer, E. Buchanan, H. Shacham, and S. Savage. Return-oriented programming: Systems, lan- guages, and applications. ACM Transactions on Information System Security, 15(1):2:1–2:34, March 2012.

[2] E. J. Schwartz, T. Avgerinos, and D. Brumley. Q: Exploit hardening made easy. In USENIX Security Symposium, 2011.