**Meo**

## Level 8 - Bitwise and
Without using the following instructions:
```mov```, ```xchg```
Please perform the following:
```
rax = rdi AND rsi
```
i.e. Set ```rax``` to the value of ```(rdi AND rsi)```

```python
import pwn
pwn.context.update(arch="amd64")
code = pwn.asm("""
    and rdi, rsi
    xor rax, rax
    or rax, rdi
""")
process = pwn.process("/challenge/run")
process.write(code)
print(process.readallS())
```

## Level 9 - Bitwise logic

Using only the following instructions:
```and```, ```or```, ```xor```
Implement the following logic:
```
if x is even then
  y = 1
else
  y = 0
where:
x = rdi
y = rax
```

We can see that ``` x is even ``` means ```x and 1 = 0```
So we have ```y = ~(x & 1)``` or ```y = (x & 1) ^ 1```

```python
import pwn
pwn.context.update(arch="amd64")
code = pwn.asm("""
    xor rax, rax
    or rax, 0x1
    and rdi, 0x1
    xor rax, rdi
""")
process = pwn.process("/challenge/run")
process.write(code)
print(process.readallS())
```

**Flag:** ```pwn.college{Evf77iwpmYbnXtXY3XswSRtb9Lu.0lMwIDL1QTMyMzW}``

## Level 10 - Memory reads and writes

In x86 we can access the thing at a memory location, called dereferencing, like so:
```mov rax, [some_address]```        <=>     Moves the thing at 'some_address' into rax
This also works with things in registers:
```mov rax, [rdi]```         <=>     Moves the thing stored at the address of what ```rdi``` holds to ```rax```
This works the same for writing:
```mov [rax], rdi```         <=>     Moves rdi to the address of what rax holds.
So if ```rax``` was ```0xdeadbeef```, then ```rdi``` would get stored at the address 0xdeadbeef:
```[0xdeadbeef] = rdi```
Note: memory is linear, and in x86, it goes from ```0 - 0xffffffffffffffff``` (yes, huge).

The challenge requires performing the following:
1. Place the value stored at ```0x404000``` into ```rax```
2. Increment the value stored at the address ```0x404000``` by ```0x1337```
Make sure the value in ```rax``` is the original value stored at ```0x404000``` and make sure
that ```[0x404000]``` now has the incremented value.

```python
import pwn
pwn.context.update(arch="amd64")
code = pwn.asm("""
    mov rax, [0x404000]
    addq [0x404000], 0x1337
""")
process = pwn.process("/challenge/run")
process.write(code)
print(process.readallS())
```

**Flag:** pwn.college{00TL4RYpQD0kaMsJGjJXkeFMEjf.01MwIDL1QTMyMzW}
```
Executing your code...
---------------- CODE ----------------
0x400000:    mov       rax, qword ptr [0x404000]
0x400008:    add       qword ptr [0x404000], 0x1337
--------------------------------------
```
```
We add 0x1337 to qword ptr [0x404000] -> so we use addq.
``` 

## Level 11 - Data sizes
```
Recall that registers in x86_64 are 64 bits wide, meaning they can store 64 bits in them.
Similarly, each memory location is 64 bits wide. We refer to something that is 64 bits
(8 bytes) as a quad word. Here is the breakdown of the names of memory sizes:
* Quad Word = 8 Bytes = 64 bits
* Double Word = 4 bytes = 32 bits
* Word = 2 bytes = 16 bits
* Byte = 1 byte = 8 bits
In x86_64, you can access each of these sizes when dereferencing an address, just like using
bigger or smaller register accesses:
mov al, [address]        <=>         moves the least significant byte from address to rax
mov ax, [address]        <=>         moves the least significant word from address to rax
mov eax, [address]        <=>         moves the least significant double word from address to rax
mov rax, [address]        <=>         moves the full quad word from address to rax
Remember that moving only into al for instance does not fully clear the upper bytes.
```

```python
import pwn
pwn.context.update(arch="amd64")
code = pwn.asm("""
    mov al, [0x404000]
    mov bx, [0x404000]
    mov ecx, [0x404000]
    mov rdx, [0x404000]
""")
process = pwn.process("/challenge/run")
process.write(code)
print(process.readallS())
```

**Flag:** pwn.college{YePlV_90W5rnCZiEQ9i9loicul7.0FNwIDL1QTMyMzW}
```
Executing your code...
---------------- CODE ----------------
0x400000:    mov       al, byte ptr [0x404000]
0x400007:    mov       bx, word ptr [0x404000]
0x40000f:    mov       ecx, dword ptr [0x404000]
0x400016:    mov       rdx, qword ptr [0x404000]
--------------------------------------
```

## Level 12 - Dynamic address memory writes
```
It is worth noting, as you may have noticed, that values are stored in reverse order of how we
represent them. As an example, say:
[0x1330] = 0x00000000deadc0de
If you examined how it actually looked in memory, you would see:
[0x1330] = 0xde 0xc0 0xad 0xde 0x00 0x00 0x00 0x00
This format of storing things in 'reverse' is intentional in x86, and its called Little Endian.

For this challenge we will give you two addresses created dynamically each run. The first address
will be placed in rdi. The second will be placed in rsi.
Using the earlier mentioned info, perform the following:
1. set [rdi] = 0xdeadbeef00001337
2. set [rsi] = 0xc0ffee0000
Hint: it may require some tricks to assign a big constant to a dereferenced register. Try setting
a register to the constant then assigning that register to the derefed register.
```

```python
import pwn
pwn.context.update(arch="amd64")
code = pwn.asm("""
    mov rax, 0xdeadbeef00001337
    movq [rdi], rax
    mov rax, 0xc0ffee0000
    movq [rsi], rax
""")
process = pwn.process("/challenge/run")
process.write(code)
print(process.readallS())
```

**Flag:** pwn.college{cvR-ud92z7Paho-4tSdBn4jQRiD.0VNwIDL1QTMyMzW}
```
Executing your code...
---------------- CODE ----------------
0x400000:    movabs    rax, 0xdeadbeef00001337
0x40000a:    mov       qword ptr [rdi], rax
0x40000d:    movabs    rax, 0xc0ffee0000
0x400017:    mov       qword ptr [rsi], rax
--------------------------------------
```
Maybe:
```python
import pwn
pwn.context.update(arch="amd64")
code = pwn.asm("""
    movb [rdi], 0x37
    movb [rdi + 1], 0x13
    movb [rdi + 2], 0x00
    movb [rdi + 3], 0x00
    movb [rdi + 4], 0xEF
    movb [rdi + 5], 0xBE
    movb [rdi + 6], 0xAD
    movb [rdi + 7], 0xDE
    movb [rsi], 0x00
    movb [rsi + 1], 0x00
    movb [rsi + 2], 0xEE
    movb [rsi + 3], 0xFF
    movb [rsi + 4], 0xC0
    movb [rsi + 5], 0x00
    movb [rsi + 6], 0x00
    movb [rsi + 7], 0x00
""")
process = pwn.process("/challenge/run")
process.write(code)
print(process.readallS())
```

```
Executing your code...
---------------- CODE ----------------
0x400000:    mov       byte ptr [rdi], 0x37
0x400003:    mov       byte ptr [rdi + 1], 0x13
0x400007:    mov       byte ptr [rdi + 2], 0
0x40000b:    mov       byte ptr [rdi + 3], 0
0x40000f:    mov       byte ptr [rdi + 4], 0xef
0x400013:    mov       byte ptr [rdi + 5], 0xbe
0x400017:    mov       byte ptr [rdi + 6], 0xad
0x40001b:    mov       byte ptr [rdi + 7], 0xde
0x40001f:    mov       byte ptr [rsi], 0
0x400022:    mov       byte ptr [rsi + 1], 0
0x400026:    mov       byte ptr [rsi + 2], 0xee
0x40002a:    mov       byte ptr [rsi + 3], 0xff
0x40002e:    mov       byte ptr [rsi + 4], 0xc0
0x400032:    mov       byte ptr [rsi + 5], 0
0x400036:    mov       byte ptr [rsi + 6], 0
0x40003a:    mov       byte ptr [rsi + 7], 0
--------------------------------------
```

## Level 13 - Consecutive memory reads

```
Recall that memory is stored linearly. What does that mean? Say we access the quad word at 0x1337:
[0x1337] = 0x00000000deadbeef
The real way memory is layed out is byte by byte, little endian:
[0x1337] = 0xef
[0x1337 + 1] = 0xbe
[0x1337 + 2] = 0xad
...
[0x1337 + 7] = 0x00
What does this do for us? Well, it means that we can access things next to each other using offsets,
like what was shown above. Say you want the 5th *byte* from an address, you can access it like:
mov al, [address+4]
Remember, offsets start at 0.

Perform the following:
1. load two consecutive quad words from the address stored in rdi
2. calculate the sum of the previous steps quad words.
3. store the sum at the address in rsi
```

```python
import pwn
pwn.context.update(arch="amd64")
code = pwn.asm("""
    mov rax, [rdi]
    add rax, [rdi + 8]
    mov [rsi], rax
""")
process = pwn.process("/challenge/run")
process.write(code)
print(process.readallS())
```
**Flag:** pwn.college{w7SeIC4venOY9ww1Q3WRNbY35-c.0lNwIDL1QTMyMzW}
```
Executing your code...
---------------- CODE ----------------
0x400000:    mov       rax, qword ptr [rdi]
0x400003:    add       rax, qword ptr [rdi + 8]
0x400007:    mov       qword ptr [rsi], rax
--------------------------------------
```

## Level 14 - The stack
```
In this level you will be working with the Stack, the memory region that dynamically expands
and shrinks. You will be required to read and write to the Stack, which may require you to use
the pop & push instructions. You may also need to utilize rsp to know where the stack is pointing.

Replace the top value of the stack with (top value of the stack - rdi). (dấu này là dấu trừ)

We will now set the following in preparation for your code:
rdi = 0x3c44
(stack) [0x7fffff1ffff8] = 0x2f56971e (0x2f56971e can be replaced)
```
```python
import pwn
pwn.context.update(arch="amd64")
code = pwn.asm("""
    pop rax
    sub rax, rdi
    push rax
""")
process = pwn.process("/challenge/run")
process.write(code)
print(process.readallS())
```

## Level 16 - Memory reads and writes with the stack

In the previous levels you used push and pop to store and load data from the stack
however you can also access the stack directly using the stack pointer.
The stack pointer is stored in the special register rsp.
rsp always stores the memory address to the top of the stack,
i.e. the memory address of the last value pushed.
Similar to the memory levels we can use [rsp] to access the value at the memory address in rsp.

Without using pop please calculate the average of 4 consecutive quad words stored on the stack.
Store the average on the top of the stack. Hint:
RSP+0x?? Quad Word A
RSP+0x?? Quad Word B
RSP+0x?? Quad Word C
RSP      Quad Word D
RSP-0x?? Average

```python
import pwn
pwn.context.update(arch="amd64")
code = pwn.asm("""
    mov rax, 0
    add rax, [rsp]
    add rax, [rsp+0x8]
    add rax, [rsp+0x10]
    add rax, [rsp+0x18]
    shr rax, 2 
    mov [rsp-0x8], rax
    # sub rsp, 0x8 dong nay co cung duoc ma khong co cung duoc
""")
process = pwn.process("/challenge/run")
process.write(code)
print(process.readallS())
```
