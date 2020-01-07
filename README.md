
 RPN Calculator
 Description

This code simulates a simple calculator for unlimited-precision unsigned integers.
The code is written entirely in assembly language.  

Reverse Polish notation (RPN) is a mathematical notation in which every operator follows all its operands, for example "3 + 4" would be presented as "3 4 +".For simplicity, each operator will appear on a separate line of input. Input and output operands are in hexadecimal representation. 

The program is prompt ‘calc: ‘ and waiting for input. Each number or operator is entered in a separate line. For example, to enter a number “0x7A+9” a user should type:
calc: 7A
calc: 09
calc: + 
Operations are performed as is standard for an RPN calculator: any input number is pushed onto an operand stack. Each operation is performed on operands which are popped from the operand stack. The result, if any, is pushed onto the operand stack. The output should contain no leading zeroes, but the input may have some leading zeroes.
I needed to implement a separate oerand stack inside a static array, allocated in (e.g.) .bss. The operand stack size is 5 slots.
The program printinting "Error: Operand Stack Overflow" if the calculation attempts to push operands onto the operand stack and there is no free space on the operand stack.
The program printinting "Error: Insufficient Number of Arguments on Stack" if an operation attempts to pop an empty stack. 
In any case of error, The program returns the stack to its previous state (as it was before the failed action).
The program also counting the number of operations (both successful and unsuccessful) performed. This is the return valued which returned to function main. The size of the operands is unbounded, except by the size of available heap space on the virtual memory.
Operations:
The operations that supported by the calculator are:
‘q’ – quit
‘+’ – unsigned addition 
pop two operands from operand stack, and push one result, their sum
‘p’ – pop-and-print
pop one operand from the operand stack, and print its value to stdout
‘d’ – duplicate
push a copy of the top of the operand stack onto the top of the operand stack
‘^’ - X*2^Y, with X being the top of operand stack and Y the element next to x in the operand stack. If Y>200 this is considered an error, in which case i print out an error message and leave the operand stack unaffected.
pop two operands from the operand stack, and push one result
‘v’ – X*2^(-Y), with X and Y as above. This number may be not an integer. You are required to truncate the fraction part of it and keep only the integer part.
pop two operands from the operand stack, and push one result
‘n’ – number of '1' bits
pop one operand from the operand stack, and push one result
‘sr’ – square root (bonus item*)
pop one operand from the operand stack, and push one result (only the integer part)
Assumptions
I assumed that the input is correct (i.e. numbers in hexa, no illegal characters)
Each input line is no more than 80 characters in length
Debug option
The program allows “-d” command line argument, which means a debug option. When "-d" option is set, it prints out to stderr various debugging messages (as a minimum, print out every number read from the user, and every result pushed onto the operand stack).


Compile and link your assembly file calc.s as follows:

     nasm -f elf calc.s -o calc.o 
     gcc -m32 -Wall -g calc.o -o calc 


