# MSP430 Memory Manager

This script translates references to selected variables in C source files to function calls which access external SPI memory 
allowing C variables to be stored externally. This is useful because the MSP430 lacks an external memory bus, so there is no way
to have the C compiler use external memory to hold declared variables.

The script accepts #pragma statements in the input file to change functionality. These are ignored by the C compiler, so source files that rely on 
the script for preprocessing can also be compiled without preprocessing. These are the #pragmas the script recognizes:

```C
#pragma MM_READ func       //specifies the function used to read external memory
#pragma MM_WRITE func      //specifies the function used to write external memory
#pragma MM_ON              //enables the functionality of the script
#pragma MM_OFF             //disables the functionality of the script
#pragma MM_DECLARE         //variables declared afterward are stored externally
#pragma MM_END             //variables declared afterward are stored internally
#pragma MM_GLOBALS         //variables declared afterward are globals stored externally
#pragma MM_ASSIGN_GLOBALS  //inserts code to assign pointer addresses to globals stored externally
#pragma MM_VAR var         //specifies one variable to be stored externally
#pragma MM_OFFSET offset   //specifies beginning offset for variables stored externally
```


After a variable has been assigned to external memory, the script replaces all reads and writes to it 
with the functions that access external memory. For example:
```C
//BEFORE PROCESSING
void puts(unsigned char *msg)
{
     #pragma MM_VAR msg
     for (int i=0;msg[i];i++) putchar(msg[i]);
}

int main()
{    
     #pragma MM_OFFSET 200
     #pragma MM_DECLARE
     unsigned char text1[30];
     unsigned char text2[30];
     #pragma MM_END

     text1[0]='A';
     text1[1]=0;
     puts(text1);
}

//AFTER PROCESSING
void puts(unsigned char *msg)
{
     #pragma MM_VAR msg
     for (int i=0;RAM_Read(msg+i);i++) putchar(RAM_Read(msg+i));   //reference to msg translated
}

int main()
{    
     #pragma MM_OFFSET 200
     #pragma MM_DECLARE
     unsigned char *text1=(unsigned char*)200;                    //declaration of text1 changed to external pointer
     unsigned char *text2=(unsigned char*)230;                    //declaration of text2 changed to external pointer
     #pragma MM_END

     RAM_Write(text1+0,'A');                                      //reference to text1 translated
     RAM_Write(text1+1,0);                                        //reference to text1 translated
     puts(text1);
}
```
