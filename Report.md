# EE6470 Homework 7 Report
###### tags: `EE6470`

## Problems and Solutions
### Platform installation
Q: How to use semaphore mutex
A: By the code in lab.
```
int sem_wait (uint32_t *__sem) __THROW{
  uint32_t value, success; //RV32A
  __asm__ __volatile__("\
L%=:\n\t\
     lr.w %[value],(%[__sem])            # load reserved\n\t\
     beqz %[value],L%=                   # if zero, try again\n\t\
     addi %[value],%[value],-1           # value --\n\t\
     sc.w %[success],%[value],(%[__sem]) # store conditionally\n\t\
     bnez %[success], L%=                # if the store failed, try again\n\t\
"
    : [value] "=r"(value), [success]"=r"(success)
    : [__sem] "r"(__sem)
    : "memory");
  return 0;
}

int sem_post (uint32_t *__sem) __THROW{
  uint32_t value, success; //RV32A
  __asm__ __volatile__("\
L%=:\n\t\
     lr.w %[value],(%[__sem])            # load reserved\n\t\
     addi %[value],%[value], 1           # value ++\n\t\
     sc.w %[success],%[value],(%[__sem]) # store conditionally\n\t\
     bnez %[success], L%=                # if the store failed, try again\n\t\
"
    : [value] "=r"(value), [success]"=r"(success)
    : [__sem] "r"(__sem)
    : "memory");
  return 0;
}


```

## Implementation details 

### Testbench in risc-v software
I feed data pixel by pixel with core id as below code.
![](https://i.imgur.com/ZtihuN5.png)

Then verify the answer by the golden.
![](https://i.imgur.com/onHC9CB.png)

Also in Testbench, it allow two data transferring mode: DMA and memory copy. But I add semaphore wait/signal to promise mutual exclusive when I access DMA.
![](https://i.imgur.com/NZB9IPd.png)



### Design in risc-v platform
* main.cpp
I add two filters and bind it to different ports and memory address.
![](https://i.imgur.com/zboZo1M.png)
![](https://i.imgur.com/N6SdfCj.png)
Also I add two cores to control it.
![](https://i.imgur.com/XoHWmpB.png)



## Additional features
### Different directory structure
For simplier testing, I set different directory structure, which is embedded prebuild risc-v platform platform.
```
hw7
   ├── platform 
   │   ├── RISC-V software base files
   │   └── Filter.h                 
   ├── software                    
   │   ├── Makefile                    # Makefile will include riscv-vp-hw6 in bin to run software.
   │   ├── main.cpp    
   │   ├── input_bitmap.h     
   │   └── output_bitmap.h         
   └── bin                  
       └── riscv-vp-hw7                # platform prebuild

   
```


## Experimental results
### Simulating times
* DMA
![](https://i.imgur.com/XA7YUUQ.png)
![](https://i.imgur.com/Jtms7vR.png)


* memcpy
![](https://i.imgur.com/P6EXKwg.png)
![](https://i.imgur.com/nZES7bk.png)


Both method can work successfully, but DMA method will take 0.84 times rather than memcoy method


## Discussions and conclusions
Multi-core system need mutex to promise critical section, which is a important technique in morderm computer, which need to share hardware access with multiple devices, for example, cores, I/O, uart,etc.
