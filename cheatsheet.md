## Addresses

```
0000=A, 0001=B, 0002=X, 0003=PC
0200=Output
0201, 0202 & 0203=Flags A, B & X, b1=Carry, b0=Overflow
0377=Input
```

## Op-codes: octal PQR

```
----Bit Test & Manipulate----
P--           -Q-         --R
0 = Set 0   (bit number)
1 = Set 1
2 = Skip 0                2
3 = Skip 1

----Shifts/Rotates---- (one byte)
P--           -Q-         --R
0 = Rt Shift  0 = A:4
1 = Rt Rot    1 = A:1     1
2 = Lt Shift  2 = A:2
3 = Lt Rot    3 = A:3
              4 = B:4
              5 = B:1
              6 = B:2
              7 = B:3     (bits)

----Misc---- (one byte)
P--           -Q-         --R
0 = Halt    (don't care)  0
1 = Halt
2 = NOP
3 = NOP       360 = System extension

----Jumps----
P--           -Q-         --R
0 = A
1 = B
2 = X
3 = Unc                   3 != 0
              4 = JPD     4 == 0
              5 = JPI     5 =< 0
              6 = JMD     6 >= 0
              7 = JMI     7 > 0

----Or, And, LNeg----
P--           -Q-         --R
3             0 = Or
              1 = (NOP)
              2 = And
              3 - LNeg    (address mode)
                          3 = Const
                          4 = Mem
                          5 = Ind
                          6 = Idx
                          7 = Ind/Idx

----Add, Sub, Load, Store----
P--           -Q-         --R
0 = A         0 = Add
1 = B         1 = Sub
2 = X         2 = Load
              3 = Store   (address mode)
                          3 = Const
                          4 = Mem
                          5 = Ind
                          6 = Idx
                          7 = Ind/Idx
```

## Address modes; `LDA b        STA b`

```
3 Const:   A=b              Immed:  (over-write b)
4 Mem:     A=[b]                    [b]=A
5 Ind:     A=[[b]]          Defer:  [[b]]=A
6 Idx:     A=[b + X]                [b + X]=A
7 Ind/Idx: A=[[b] + X]              [[b] + X]=A
Bit Test & Manipulate op-codes are only Mem mode.
```

## Jump Modes; `jmp b`

```
4 jmp direct:          PC=[b]
5 jmp indirect:        PC=[[b]]
6 jmp direct & mark:   [b]=PC+2, PC=[b]+1
7 jmp indirect & mark: [[b]]=PC+2, PC=[[b]]+1
```

## System Extension ("NOOP" 0360)

```
Index in A, b7 clear = read into B, b7 set = write to B
A=
000..007 : DS1307 RTC registers;
  sec,min,hr-24,day,date,month,year,ctrl (BCD).
010 : Flags;  b0 set = buttons toggle bits.
011: EEPROM Page Map.
012..016: User data.
017: Control auto-run (see Buttons at Power On).
020: Control LEDs; Read N/A.
  Write b0:INP,b1:ADDR,b2:MEM,b3:RUN,b4-b7=~PWM.
021: Random; Read 0..255, write <seed> or 0.
022: Sleep; Read N/A. Write=delay in ms.
023: Serial byte; @38400baud.
0177: Extensions enabled; Reading sets A=0
```

## Buttons

```
STOP+CLR: All LEDs off.
CLR+STOR: Zero memory, PC=004, address=004, reset speed.
STOP+BitN: Loads pre-defined program:
  0:  Simple counter
  1:  Pattern
  2:  Counting Clock
  3:  BCD Clock
  4:  Binary Clock
  5:  Das Blinken Lights
  6:  Sieve of Eratosthenes
  7:  Set RTC (A=HH, B=MM)
BitN+STOR: Write memory to EEPROM page N.
BitN+READ: Reads memory from EEPROM page N.  Default pages
  0:    256
  1:    256
  2:    128
  3:    128
  4:    64
  5:    64
  6:    64
  7:    64
STOP+READ: System Extension read.  Index is address, result in Output.
STOP+STOR: System Extension write.  Index is address, value from Input.
BitN+STOP: Set CPU speed.  Cycle delay=2^N ms.
BitN+DISP/SET: Send/receive program memory at baud N (0/1/2/3=4800/9600/19k2/38k4)
```

## Buttons at Power On

```
STOP & BitN: auto-run built-in program N.
READ & BitN: auto-run from EEPROM slot N.
STOP       : turn auto-run off.
STOP & CLR : reset configuration to defaults.
```

----
[//]: # ( vim: set ts=4 sw=4 et cindent tw=80 ai si syn=markdown ft=markdown: )