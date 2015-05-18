# What is SLIP Trace? #

**SLIP Trace** is a tool for parsing **GTF SLIP STD traces for successful Branch** and viewing the results in _**useful**_ way.


---


# Main features #
  1. Presentation of trace(s) in readable format:
    * **"From"** and **"To"** fields give full information about the branch recorded by trace.
  1. Ability to narrow the scope of interest:
    * You can specify which **HLASM** module you wanted to trace and **offsets** in its listing, so the trace will be narrowed only to this information. **Note**: Narrowing to the level of HLASM module is _**compulsory**_.
    * When this information is specified the output will contain **only** relevant information, so you will not need to look through all the trace in order to find several points.


---


# Instructions #
Here are the instructions you need to follow in order to use SLIP Trace tool.
## Get the source code and compile it ##
  * All sources and JCL jobs for compilation and running the program are availible in code section of this project.
## Capture GTF SLIP STD Successfull Branch trace for needed load module ##
  * Let your module be named **`Mod1`** and the environment (the JCL job for it) be named **`Job1`**. Then, you will need to do the following:
    1. _**Make GTF**_ for capturing SLIP traces (specify **`TRACE=SLIP`** in options). Let **`GTFTRACE`** be the name of output dataset for this GTF;
    1. _**Start this GTF**_ (i.e., `/S TEMPGTF.GTFSLIP`);
    1. _**Create SLIP trap**_ for your module. This must be Succsessfull branch SLIP trap:
```
        /SLIP SET,SBT,ACTION=TRACE,ID=SLP1,JOBNAME=Job1,PVTMOD=Mod1,PRCNTLIM=65,END
```
    1. _**Start**_ your `Mod1` by executing `Job1`. Wait for it to finish its work.
    1. After `Job1` has finished its work, _**stop SLIP trap**_ by:
```
        /SLIP MOD,DISABLE,ID=SLP1,END
```
      * _**Optionally**_, you can _**delete this SLIP trap**_ by:
```
          /SLIP DEL,ID=SLI1,END
```
    1. Also, do not forget to _**stop the GTF**_ you started for this work:
```
        /P GTFSLIP
```
    1. As a result you will obtain `GTFTRACE` dataset containing trace for this SLIP trap.
## Modify sample JCL job for running SLIP Trap tool ##
  * Specify the real `GTFTRACE` as **`SYSIN` DD**;
  * Specify **`ASMOFFS` DD** correctly. To do so, **_look into_ `LINKLIST` _for_ `Mod1`** and make the table according to the rules given in sample JCL job;
  * Also, do not forget to _**make 1st non-comment string contain parameters**_ for running SLIP Trace tool.
## Run the edited JCL job ##
  * On output (which will be put into `SYSOUT`) you will receive the readeble and trace contents, narrowed to the scope specified by the JCL job.


---


# Requirements #
To use SLIP Trap tool without changes to the source code, you must follow the following requirements:
  1. `ASMOFFS` DD **must have LRECL=80** (as a JCL job);
  1. `SYSOUT` DD must have **LRECL>=80** for the output strings.


---


# Example #
Let's look at the sample program which can be found in the source code for verifying.
Code of `TEST`:
```
TEST     MODBGN AMODE=ANY,GBLVAR=N                                      00010005
         B     label1                                                   00011005
label1   DS 0H                                                          00012005
         WTO   'label1'                                                 00013005
         B     label7                                                   00014005
label2   DS 0H                                                          00015005
         WTO   'label2'                                                 00016005
         B     label6                                                   00017005
label3   DS 0H                                                          00018005
         WTO   'label3'                                                 00019005
         B     label8                                                   00020005
label4   DS 0H                                                          00030005
         WTO   'label4'                                                 00040005
         LA    R4,4                                                     00050005
label5   DS 0H                                                          00060005
         WTO   'label5'                                                 00060105
         BCT   R4,label5                                                00060205
         B     label2                                                   00060305
label6   DS 0H                                                          00060405
         WTO   'label6'                                                 00060505
         B     label3                                                   00060605
label7   DS 0H                                                          00060705
         WTO   'label7'                                                 00060805
         B     label4                                                   00060905
label8   DS 0H                                                          00061005
         WTO   'label8'                                                 00062005
         MODEND  RC=(R15)                 Return                        00063005
         END                                                            00064005
```

Listing of `TEST`:
```
 '  0                                      1 TEST     MODBGN AMODE=ANY,GBLVAR=N                                      00010005
 '   000000                00000 00128     2+TEST     CSECT ,                                                        01-MODBG
 '                                         3+TEST     AMODE ANY                                                      01-MODBG
 '                                         4+TEST     RMODE ANY                                                      01-MODBG
 '   000000                00000 00128     5+TEST_LOC    LOCTR ,                                                     01-MODBG
 @                         00000           7+R0       EQU   0                                                        02-REGS
 @                         00001           8+R1       EQU   1                                                        02-REGS
 @                         00002           9+R2       EQU   2                                                        02-REGS
 @                         00003          10+R3       EQU   3                                                        02-REGS
 @                         00004          11+R4       EQU   4                                                        02-REGS
 @                         00005          12+R5       EQU   5                                                        02-REGS
 @                         00006          13+R6       EQU   6                                                        02-REGS
 @                         00007          14+R7       EQU   7                                                        02-REGS
 @                         00008          15+R8       EQU   8                                                        02-REGS
 @                         00009          16+R9       EQU   9                                                        02-REGS
 @                         0000A          17+R10      EQU   10                                                       02-REGS
 @                         0000B          18+R11      EQU   11                                                       02-REGS
 @                         0000C          19+R12      EQU   12                                                       02-REGS
 @                         0000D          20+R13      EQU   13                                                       02-REGS
 @                         0000E          21+R14      EQU   14                                                       02-REGS
 @                         0000F          22+R15      EQU   15                                                       02-REGS
 '                                        24+         PUSH USING                                                     02-PROCB
 '   000000                00000 00048    25+TEST_MAIN_SAVE DSECT                             SaveArea               02-PROCB
 '   000000                               26+TEST_MAIN_SAVE_AREA DS XL72                                             02-PROCB
 '   000048                00048 00000    27+                    ORG TEST_MAIN_SAVE_AREA                             02-PROCB
 '   000000                               28+TEST_MAIN_SAVE_RESERVED DS F                     Reserved               02-PROCB
 '   000004                               29+TEST_MAIN_SAVE_OLD_SAVE DS A                     Old SaveArea addr      02-PROCB
 '   000008                               30+TEST_MAIN_SAVE_NEW_SAVE DS A                     Next SaveArea addr     02-PROCB
 '   00000C                               31+TEST_MAIN_SAVE_R14      DS F                                            02-PROCB
 '   000010                               32+TEST_MAIN_SAVE_R15      DS F                                            02-PROCB
 '   000014                               33+TEST_MAIN_SAVE_R0       DS F                                            02-PROCB
 '   000018                               34+TEST_MAIN_SAVE_R1       DS F                                            02-PROCB
 '   00001C                               35+TEST_MAIN_SAVE_R2       DS F                                            02-PROCB
 '   000020                               36+TEST_MAIN_SAVE_R3       DS F                                            02-PROCB
 '   000024                               37+TEST_MAIN_SAVE_R4       DS F                                            02-PROCB
 '   000028                               38+TEST_MAIN_SAVE_R5       DS F                                            02-PROCB
 '   00002C                               39+TEST_MAIN_SAVE_R6       DS F                                            02-PROCB
 '   000030                               40+TEST_MAIN_SAVE_R7       DS F                                            02-PROCB
 '   000034                               41+TEST_MAIN_SAVE_R8       DS F                                            02-PROCB
 '   000038                               42+TEST_MAIN_SAVE_R9       DS F                                            02-PROCB
 '   00003C                               43+TEST_MAIN_SAVE_R10      DS F                                            02-PROCB
 '   000040                               44+TEST_MAIN_SAVE_R11      DS F                                            02-PROCB
 '   000044                               45+TEST_MAIN_SAVE_R12      DS F                                            02-PROCB
 '                         00048          46+TEST_MAIN_SAVELEN EQU *-TEST_MAIN_SAVE           Length                 02-PROCB
                                         47+*
 '   000000                00000 00128    48+TEST     CSECT                                                          02-PROCB
 '   000000                00000 00128    49+TEST_LOC    LOCTR ,                                                     02-PROCB
 '   000000                00000 00128    50+TEST_MAIN_PROC_LOC LOCTR ,                                              02-PROCB
 '   000000                               51+TEST_MAIN DS 0D                                                         02-PROCB
 '                                        52+         POP   USING                                                    02-PROCB
                                         53+*
 '   000000 90EC D00C            0000C    54+         STM   R14,R12,12(R13)                   Save registers         02-PROCB
 '   000004 0DC0                          55+         BASR  R12,0                             Establish              02-PROCB
 '                    R:C  00006          56+         USING *,R12                               addresability        02-PROCB
 '   000006 0700                          59+         CNOP   0,4                                                     03-STORA
 '  1                                                                                                               Page    4
      Active Usings: TEST+X'6',R12
 '  0  Loc  Object Code    Addr1 Addr2  Stmt   Source Statement                                  HLASM R6.0  2014/04/28 16.09
 '  0000008 47F0 C00E            00014    60+         B      IHB0004B                     .BRANCH AROUND DATA        03-STORA
 '   00000C 00000048                      61+IHB0004L DC     A(TEST_MAIN_SAVELEN)         .STORAGE LENGTH            03-STORA
 '   000010 00                            62+IHB0004F DC     BL1'00000000'                                           03-STORA
 '   000011 00                            63+         DC     AL1(0*16)                    .KEY                       03-STORA
 '   000012 00                            64+         DC     AL1(0)                       .SUBPOOL                   03-STORA
 '   000013 02                            65+         DC     BL1'00000010'                .FLAGS                     03-STORA
 '   000014                               66+IHB0004B DS     0F                                                      03-STORA
 '   000014 5800 C006            0000C    67+         L      0,IHB0004L                   .STORAGE LENGTH       @P9C 03-STORA
 '   000018 58F0 C00A            00010    68+         L      15,IHB0004F                  .CONTROL INFORMATION  @P9C 03-STORA
 '   00001C 58E0 0010            00010    69+         L      14,16(0,0)                   .CVT ADDRESS               03-STORA
 '   000020 58EE 0304            00304    70+         L      14,772(14,0)                 .ADDR SYST LINKAGE TABLE   03-STORA
 '   000024 58EE 00A0            000A0    71+         L      14,160(14,0)                 .OBTAIN LX/EX FOR OBTAIN   03-STORA
 '   000028 B218 E000      00000          72+         PC     0(14)                        .PC TO STORAGE RTN         03-STORA
 '   00002C 18E1                          73+         LR    R14,R1                            Preserve R1            02-PROCB
 '   00002E 50DE 0004            00004    74+         ST    R13,4(R14)                        Save ptr to old SA     02-PROCB
 '   000032 50ED 0008            00008    75+         ST    R14,8(R13)                        Save ptr to new SA     02-PROCB
 '   000036 18DE                          76+         LR    R13,R14                           Set R13 to new SA @    02-PROCB
 '                    R:D  00000          77+         USING TEST_MAIN_SAVE,R13                                       02-PROCB
 '   000038 581D 0004            00004    78+         L     R1,4(R13)                         Get addr of old SA     02-PROCB
 '   00003C 9801 1014            00014    79+         LM    R0,R1,20(R1)                      Restore R0 and R1      02-PROCB
                                         80+*
                                          81+* Restore R0 and R1
 '   000040 581D 0004            00004    82+         L     R1,4(R13)                                                01-MODBG
 '   000044 9801 1014            00014    83+         LM    R0,R1,20(R1)                                             01-MODBG
 '   000048 47F0 C046            0004C    84          B     label1                                                   00011005
 '   00004C                               85 label1   DS 0H                                                          00012005
 '                                        86          WTO   'label1'                                                 00013005
 #   00004C                               88+         CNOP  0,4                                                      01-WTO
 #   00004C A715 0007            0005A    89+         BRAS  1,IHB0006A               BRANCH AROUND MESSAGE      @LCC 01-WTO
 #   000050 000A                          90+         DC    AL2(10)                  TEXT LENGTH            @YA17152 01-WTO
 #   000052 0000                          91+         DC    B'0000000000000000'      MCSFLAGS                        01-WTO
 #   000054 9381828593F1                  92+         DC    C'label1'                MESSAGE TEXT               @L6C 01-WTO
 #   00005A                               93+IHB0006A DS    0H                                                       01-WTO
 #   00005A 0A23                          94+         SVC   35                       ISSUE SVC 35               @L6A 01-WTO
 '   00005C 47F0 C0C2            000C8    95          B     label7                                                   00014005
 '   000060                               96 label2   DS 0H                                                          00015005
 '                                        97          WTO   'label2'                                                 00016005
 #   000060                               99+         CNOP  0,4                                                      01-WTO
 #   000060 A715 0007            0006E   100+         BRAS  1,IHB0008A               BRANCH AROUND MESSAGE      @LCC 01-WTO
 #   000064 000A                         101+         DC    AL2(10)                  TEXT LENGTH            @YA17152 01-WTO
 #   000066 0000                         102+         DC    B'0000000000000000'      MCSFLAGS                        01-WTO
 #   000068 9381828593F2                 103+         DC    C'label2'                MESSAGE TEXT               @L6C 01-WTO
 #   00006E                              104+IHB0008A DS    0H                                                       01-WTO
 #   00006E 0A23                         105+         SVC   35                       ISSUE SVC 35               @L6A 01-WTO
 '   000070 47F0 C0AE            000B4   106          B     label6                                                   00017005
 '   000074                              107 label3   DS 0H                                                          00018005
 '                                       108          WTO   'label3'                                                 00019005
 #   000074                              110+         CNOP  0,4                                                      01-WTO
 #   000074 A715 0007            00082   111+         BRAS  1,IHB0010A               BRANCH AROUND MESSAGE      @LCC 01-WTO
 #   000078 000A                         112+         DC    AL2(10)                  TEXT LENGTH            @YA17152 01-WTO
 #   00007A 0000                         113+         DC    B'0000000000000000'      MCSFLAGS                        01-WTO
 #   00007C 9381828593F3                 114+         DC    C'label3'                MESSAGE TEXT               @L6C 01-WTO
 #   000082                              115+IHB0010A DS    0H                                                       01-WTO
 #   000082 0A23                         116+         SVC   35                       ISSUE SVC 35               @L6A 01-WTO
 '   000084 47F0 C0D6            000DC   117          B     label8                                                   00020005
 '  1                                                                                                               Page    5
      Active Usings: TEST+X'6',R12  TEST_MAIN_SAVE,R13
 '  0  Loc  Object Code    Addr1 Addr2  Stmt   Source Statement                                  HLASM R6.0  2014/04/28 16.09
 '  0000088                              118 label4   DS 0H                                                          00030005
 '                                       119          WTO   'label4'                                                 00040005
 #   000088                              121+         CNOP  0,4                                                      01-WTO
 #   000088 A715 0007            00096   122+         BRAS  1,IHB0012A               BRANCH AROUND MESSAGE      @LCC 01-WTO
 #   00008C 000A                         123+         DC    AL2(10)                  TEXT LENGTH            @YA17152 01-WTO
 #   00008E 0000                         124+         DC    B'0000000000000000'      MCSFLAGS                        01-WTO
 #   000090 9381828593F4                 125+         DC    C'label4'                MESSAGE TEXT               @L6C 01-WTO
 #   000096                              126+IHB0012A DS    0H                                                       01-WTO
 #   000096 0A23                         127+         SVC   35                       ISSUE SVC 35               @L6A 01-WTO
 '   000098 4140 0004            00004   128          LA    R4,4                                                     00050005
 '   00009C                              129 label5   DS 0H                                                          00060005
 '                                       130          WTO   'label5'                                                 00060105
 #   00009C                              132+         CNOP  0,4                                                      01-WTO
 #   00009C A715 0007            000AA   133+         BRAS  1,IHB0014A               BRANCH AROUND MESSAGE      @LCC 01-WTO
 #   0000A0 000A                         134+         DC    AL2(10)                  TEXT LENGTH            @YA17152 01-WTO
 #   0000A2 0000                         135+         DC    B'0000000000000000'      MCSFLAGS                        01-WTO
 #   0000A4 9381828593F5                 136+         DC    C'label5'                MESSAGE TEXT               @L6C 01-WTO
 #   0000AA                              137+IHB0014A DS    0H                                                       01-WTO
 #   0000AA 0A23                         138+         SVC   35                       ISSUE SVC 35               @L6A 01-WTO
 '   0000AC 4640 C096            0009C   139          BCT   R4,label5                                                00060205
 '   0000B0 47F0 C05A            00060   140          B     label2                                                   00060305
 '   0000B4                              141 label6   DS 0H                                                          00060405
 '                                       142          WTO   'label6'                                                 00060505
 #   0000B4                              144+         CNOP  0,4                                                      01-WTO
 #   0000B4 A715 0007            000C2   145+         BRAS  1,IHB0016A               BRANCH AROUND MESSAGE      @LCC 01-WTO
 #   0000B8 000A                         146+         DC    AL2(10)                  TEXT LENGTH            @YA17152 01-WTO
 #   0000BA 0000                         147+         DC    B'0000000000000000'      MCSFLAGS                        01-WTO
 #   0000BC 9381828593F6                 148+         DC    C'label6'                MESSAGE TEXT               @L6C 01-WTO
 #   0000C2                              149+IHB0016A DS    0H                                                       01-WTO
 #   0000C2 0A23                         150+         SVC   35                       ISSUE SVC 35               @L6A 01-WTO
 '   0000C4 47F0 C06E            00074   151          B     label3                                                   00060605
 '   0000C8                              152 label7   DS 0H                                                          00060705
 '                                       153          WTO   'label7'                                                 00060805
 #   0000C8                              155+         CNOP  0,4                                                      01-WTO
 #   0000C8 A715 0007            000D6   156+         BRAS  1,IHB0018A               BRANCH AROUND MESSAGE      @LCC 01-WTO
 #   0000CC 000A                         157+         DC    AL2(10)                  TEXT LENGTH            @YA17152 01-WTO
 #   0000CE 0000                         158+         DC    B'0000000000000000'      MCSFLAGS                        01-WTO
 #   0000D0 9381828593F7                 159+         DC    C'label7'                MESSAGE TEXT               @L6C 01-WTO
 #   0000D6                              160+IHB0018A DS    0H                                                       01-WTO
 #   0000D6 0A23                         161+         SVC   35                       ISSUE SVC 35               @L6A 01-WTO
 '   0000D8 47F0 C082            00088   162          B     label4                                                   00060905
 '   0000DC                              163 label8   DS 0H                                                          00061005
 '                                       164          WTO   'label8'                                                 00062005
 #   0000DC                              166+         CNOP  0,4                                                      01-WTO
 #   0000DC A715 0007            000EA   167+         BRAS  1,IHB0020A               BRANCH AROUND MESSAGE      @LCC 01-WTO
 #   0000E0 000A                         168+         DC    AL2(10)                  TEXT LENGTH            @YA17152 01-WTO
 #   0000E2 0000                         169+         DC    B'0000000000000000'      MCSFLAGS                        01-WTO
 #   0000E4 9381828593F8                 170+         DC    C'label8'                MESSAGE TEXT               @L6C 01-WTO
 #   0000EA                              171+IHB0020A DS    0H                                                       01-WTO
 #   0000EA 0A23                         172+         SVC   35                       ISSUE SVC 35               @L6A 01-WTO
 '                                       173          MODEND  RC=(R15)                 Return                        00063005
 '   0000EC 18FF                         175+         LR    R15,R15                           Load RC from register  02-PROCE
 '   0000EE 58E0 D004            00004   176+         L     R14,TEST_MAIN_SAVE_OLD_SAVE       Get addr of old SA     02-PROCE
 '   0000F2 90F1 E010            00010   177+         STM   R15,R1,16(R14)                    Preserve R15-R1        02-PROCE
 '   0000F6 181D                         178+         LR    R1,R13                            Copy addr of SaveArea  02-PROCE
 '  1                                                                                                               Page    6
      Active Usings: TEST+X'6',R12  TEST_MAIN_SAVE,R13
 '  0  Loc  Object Code    Addr1 Addr2  Stmt   Source Statement                                  HLASM R6.0  2014/04/28 16.09
 '  00000F8 18DE                         179+         LR    R13,R14                           Get R13 to old SA      02-PROCE
 '   0000FA 0700                         182+         CNOP   0,4                                                     03-STORA
 '   0000FC 47F0 C102            00108   183+         B      IHB0024B                     .BRANCH AROUND DATA        03-STORA
 '   000100 00000048                     184+IHB0024L DC     A(TEST_MAIN_SAVELEN)         .STORAGE LENGTH            03-STORA
 '   000104 00                           185+IHB0024F DC     BL1'00000000'                                           03-STORA
 '   000105 00                           186+         DC     AL1(0*16)                    .KEY                       03-STORA
 '   000106 00                           187+         DC     AL1(0)                       .SUBPOOL                   03-STORA
 '   000107 03                           188+         DC     BL1'00000011'                .FLAGS                     03-STORA
 '   000108                              189+IHB0024B DS     0F                                                      03-STORA
 '   000108 5800 C0FA            00100   190+         L      0,IHB0024L                   .STORAGE LENGTH       @P9C 03-STORA
 '   00010C 1811                         191+         LR     1,R1                         .ADDRESS OF STORAGE        03-STORA
 '   00010E 58F0 C0FE            00104   192+         L      15,IHB0024F                  .CONTROL INFORMATION  @P9C 03-STORA
 '   000112 58E0 0010            00010   193+         L      14,16(0,0)                   .CVT ADDRESS               03-STORA
 '   000116 58EE 0304            00304   194+         L      14,772(14,0)                 .ADDR SYST LINKAGE TABLE   03-STORA
 '   00011A 58EE 00CC            000CC   195+         L      14,204(14,0)                 .OBTAIN LX/EX FOR RELEASE  03-STORA
 '   00011E B218 E000      00000         196+         PC     0(14)                        .PC TO STORAGE RTN         03-STORA
 '   000122 98EC D00C            0000C   197+         LM    R14,R12,12(R13)                   Restore R14-R12        02-PROCE
 '   000126 07FE                         198+         BR    R14                                                      02-PROCE
 '                                       199+         DROP  ,                                                        02-PROCE
                                        200+*
 '   000128                00000 00128   201+TEST     CSECT                                                          02-PROCE
 '   000000                00000 00128   202+TEST_LOC    LOCTR ,                                                     02-PROCE
 '   000128                00000 00128   203+TEST_MAIN_PROC_LOC LOCTR ,                                              02-PROCE
 '   000128                              204+         LTORG ,                                                        02-PROCE
 '                                       205          END                                                            00064005
```
After performing all actions described above, you should get the following output for this program:
```
LoadModule: TEST     AsmModule: TEST     Offs start: 00000000 Offs end: FFFFFFFF
 FROM                                  || TO TEST           19300ED8
 Asm Name Asm Offs | CSECT    OffInCs  || Asm Name Asm Offs | CSECT    OffInCs
 TEST     00000008 | TEST     00000008 || TEST     00000014 | TEST     00000014
 TEST     00000048 | TEST     00000048 || TEST     0000004C | TEST     0000004C
 TEST     0000004C | TEST     0000004C || TEST     0000005A | TEST     0000005A
 TEST     0000005C | TEST     0000005C || TEST     000000C8 | TEST     000000C8
 TEST     000000C8 | TEST     000000C8 || TEST     000000D6 | TEST     000000D6
 TEST     000000D8 | TEST     000000D8 || TEST     00000088 | TEST     00000088
 TEST     00000088 | TEST     00000088 || TEST     00000096 | TEST     00000096
 TEST     0000009C | TEST     0000009C || TEST     000000AA | TEST     000000AA
 TEST     000000AC | TEST     000000AC || TEST     0000009C | TEST     0000009C
 TEST     0000009C | TEST     0000009C || TEST     000000AA | TEST     000000AA
 TEST     000000AC | TEST     000000AC || TEST     0000009C | TEST     0000009C
 TEST     0000009C | TEST     0000009C || TEST     000000AA | TEST     000000AA
 TEST     000000AC | TEST     000000AC || TEST     0000009C | TEST     0000009C
 TEST     0000009C | TEST     0000009C || TEST     000000AA | TEST     000000AA
 TEST     000000B0 | TEST     000000B0 || TEST     00000060 | TEST     00000060
 TEST     00000060 | TEST     00000060 || TEST     0000006E | TEST     0000006E
 TEST     00000070 | TEST     00000070 || TEST     000000B4 | TEST     000000B4
 TEST     000000B4 | TEST     000000B4 || TEST     000000C2 | TEST     000000C2
 TEST     000000C4 | TEST     000000C4 || TEST     00000074 | TEST     00000074
 TEST     00000074 | TEST     00000074 || TEST     00000082 | TEST     00000082
 TEST     00000084 | TEST     00000084 || TEST     000000DC | TEST     000000DC
 TEST     000000DC | TEST     000000DC || TEST     000000EA | TEST     000000EA
 TEST     000000FC | TEST     000000FC || TEST     00000108 | TEST     00000108
```