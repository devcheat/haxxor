**Dll Injection Tutorial**
===
* * * * *



# Introduction

In this tutorial i'll try to cover all of the known methods(or at least, those that I know =p) of injecting dll's into a process.\
Dll injection is incredibly useful for TONS of stuff(game hacking, function hooking, code patching, keygenning, unpacking, etc..).\
Though there are scattered tutorials on these techniques available throughout the web, I have yet to see any complete tutorials detailing\
all of them(there may even be more out there than I have here, of course), and comparing their respective strength's and weakness's.\
This is precisely what i'll attempt to do for you in this paper. You are free to reproduce or copy this paper, so long as proper\
credit is given and you don't modify it without speaking to me first.

# The CreateRemoteThread method

I've used this in tons of stuff, and I only recently realized that a lot of people have never seen it, or know how to do it.\
I can't take credit for thinking it up...I got it from an article on codeproject, but it's a neat trick that I think more\
people should know how to use.

The trick is simple, and elegant. The windows API provides us with a function called CreateRemoteThread(). This allows you\
to start a thread in another process. For our purposes, i'll assume you know how threading works, and how to use functions like\
CreateThread(if not, you can go here ). The main disadvantage of this method is that it will work only on windows NT and above.\
To prevent it from crashing, you should use this function to check to make sure you're on an NT-based system(thanks to CatID for\
pointing this out):

**Code:** 
```cpp
bool IsWindowsNT()\
{
   // check current version of Windows
   DWORD version = GetVersion();
   // parse return
   DWORD majorVersion = (DWORD)(LOBYTE(LOWORD(version)));
   DWORD minorVersion = (DWORD)(HIBYTE(LOWORD(version)));
   return (version < 0x80000000);
}
```

The MSDN definition for CreateRemoteThread is as follows:

**Code:**
```cpp
HANDLE CreateRemoteThread( HANDLE hProcess, LPSECURITY_ATTRIBUTES lpThreadAttributes, SIZE_T dwStackSize,
                           LPTHREAD_START_ROUTINE lpStartAddress, LPVOID lpParameter, DWORD dwCreationFlags,
                           LPDWORD lpThreadId );
```

So, it's essentially CreateThread, with an hProcess argument, so that we can tell it in which process to create the new thread.\
Now, normally we would want to start the thread executing on some internal function of the process that we are interacting with.\
However, to inject a dll, we have to do something a little bit different.

**Code:**
```cpp
BOOL InjectDLL(DWORD ProcessID)
{
   HANDLE Proc;
   char buf[50]={0};
   LPVOID RemoteString, LoadLibAddy;

   if(!ProcessID)
      return false;

   Proc = OpenProcess(CREATE_THREAD_ACCESS, FALSE, ProcessID);

   if(!Proc)
   {
      sprintf(buf, "OpenProcess() failed: %d", GetLastError());
      MessageBox(NULL, buf, "Loader", NULL);
      return false;
   }

   LoadLibAddy = (LPVOID)GetProcAddress(GetModuleHandle("kernel32.dll"), "LoadLibraryA");

   RemoteString = (LPVOID)VirtualAllocEx(Proc, NULL, strlen(DLL_NAME), MEM_RESERVE|MEM_COMMIT, PAGE_READWRITE);
   WriteProcessMemory(Proc, (LPVOID)RemoteString, DLL_NAME,strlen(DLL_NAME), NULL);
   CreateRemoteThread(Proc, NULL, NULL, (LPTHREAD_START_ROUTINE)LoadLibAddy, (LPVOID)RemoteString, NULL, NULL);

   CloseHandle(Proc);

   return true;
}
```

This code, calls CreateRemoteThread() with a lpStartAddress of LoadLibrary(). So, it starts a new thread in the remote process\
and executes the LoadLibrary() function. Luckily for us, this function takes only one argument, the name of the dll to load. We can\
pass this in the arg field of CreateRemoteThread(). However, there is a minor dilemma. Since this thread will not be executing in\
our address space, it won't be able to refer to strings(such as the name of the dll) that are in our address space. So, before calling\
CreateRemoteThread(), we have to allocate space in the other process, using VirtualAllocEx(), and write our string there. Finally,\
we pass the pointer to the string inside the remote process in the single arg field of CreateRemoteThread(), and voila...Our dll is\
now loaded and running smoothly within the remote process. This is the generic loader program I use whenever I need to load a dll.

# The SetWindowsHookEx method

The SetWindowsHookEx method is a little bit more intrusive than the first, and creates more of a commotion in the injected\
process, which we normally don't want. However, it is a little bit easier to use than the first, and does have it's own advantages\
(like being able to inject into every process on the system in one shot). The SetWindowsHookEx() function is designed to allow you\
to "hook" windows messages for a given thread. This requires that you inject a dll into that process's address space, so\
SetWindowsHookEx() handles all that for us. The dll must have a function for the hook that it created though, otherwise it will\
crash.

**Code:**
```cpp
HHOOK SetWindowsHookEx(

    int idHook,
    HOOKPROC lpfn,
    HINSTANCE hMod,
    DWORD dwThreadId
);
```

idHook is just that, the ID of the message that we want to hook. There are many of them(for a complete list, go\
here ), however we'll want to use one that's as unintrusive as possible, and has the least likelihood of causing alarm bells to\
go off in any AV software(SetWindowsHookEx is the staple of all ring3 keyloggers). The WH_CBT message seems innocuous enough.

**Quote:**
```cpp
WH_CBT
```
Installs a hook procedure that receives notifications useful to a computer-based training (CBT) application. For more information,\
see the CBTProc hook procedure.

So, we'll need to create a placebo CBT hook proc in our dll, so that when the hook is called, we can handle it properly.

**Code:**
```cpp
LRESULT CALLBACK CBTProc(int nCode, WPARAM wParam, LPARAM lParam)
{
    return CallNextHookEx(0, nCode, wParam, lParam);
};
```

All we're doing is calling the next hook in the chain of hooks that exist for this message. Getting back to the SetWindowsHookEx()\
function, the next parameter we see is lpfn. lpfn is exactly as it sounds "long pointer to function". That's a pointer to our CBT\
hook proc function. So, to get this, we'll have to either hardcode the address, or load the dll first ourselves. Hardcoding anything\
is a bad idea, so we'll load the dll using LoadLibrary(), and use GetProcAddress() to get the address of our function.

| **Code:** |
```cpp
HMODULE hDll;
unsigned long cbtProcAddr;

hDll        = LoadLibrary("injected.dll");
cbtProcAddr = GetProcAddress(hDll, "CBTProc");
```

Now, in cbtProcAddr we have the address of our function. Parameter 3, of SetWindowsHookEx() is a handle to our dll, we've\
already obtained this in the process of getting the address of CBTProc(hDll is a handle to our dll, returned by LoadLibrary). Now,\
there is only one parameter left in the SetWindowsHookEx() function, the dwThreadId parameter. If you want to inject your dll into\
every process on the system(useful for global function hooks, keyloggers, trojans, rootkits, etc..) you can simply specify 0 for\
this parameter. If you want to target a specific process, you'll need to get the ID of one of it's threads. There are many ways\
of doing this, and i'll try to enumerate as many as I can think of in Appendix B. So,\
to put it all together into one neat little function:

**Code:**
```cpp
BOOL InjectDll(char *dllName)
{
    HMODULE hDll;
    unsigned long cbtProcAddr;

    hDll        = LoadLibrary(dllName);
    cbtProcAddr = GetProcAddress(hDll, "CBTProc");

    SetWindowsHookEx(WH_CBT, cbtProcAddr, hDll, GetTargetThreadIdFromWindow("targetApp"));

    return TRUE;
}
```

# The code cave method

Instead of exploiting a windows API function to force the process to load our Dll, this time we'll allocate a little chunk\
memory inside the target application, and inject a little stub that loads our dll. The advantage of this approach is that it\
will work on any version of windows, and it's also the least detectable of any of the methods mentioned thus far. Our stub will\
look like this:

**Code:**
```cpp
__declspec(naked) loadDll(void)
{
   _asm{
      //   Placeholder for the return address
      push 0xDEADBEEF

      //   Save the flags and registers
      pushfd
      pushad

      //   Placeholder for the string address and LoadLibrary
      push 0xDEADBEEF
      mov eax, 0xDEADBEEF

      //   Call LoadLibrary with the string parameter
      call eax

      //   Restore the registers and flags
      popad
      popfd

      //   Return control to the hijacked thread
      ret
   }
}
```

0xDEADBEEF is just there to mark addresses that we can't know beforehand, and have to patch-in at runtime. Ok, let's make a list\
of the things that we need to do to make this work:

To allocate space inside the target, we'll use VirtualAllocEx(). We'll need to open a handle to the process\
with the VM_OPERATION privelege specified, in order to do this. For our dllName string, we'll only need read and write priveleges.\
For the stub however, we'll need read, write, and execute priveleges. Then we'll write in our dllName string, so that we can reference\
it from the stub once it's inserted.

**Code:**
```cpp
void *dllString, *stub;
unsigned long wowID;
HANDLE hProcess

//See Appendix A for
//this function
wowID    = GetTargetProcessIdFromProcname(PROC_NAME);

hProcess = OpenProcess((PROCESS_VM_WRITE | PROCESS_VM_OPERATION), false, wowID);

dllString = VirtualAllocEx(hProcess, NULL, (strlen(DLL_NAME) + 1), MEM_COMMIT, PAGE_READWRITE);
stub      = VirtualAllocEx(hProcess, NULL, stubLen, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
WriteProcessMemory(hProcess, dllString, DLL_NAME, strlen(DLL_NAME), NULL);
```

To accomplish our next few tasks, we'll need a handle to one of our target's threads. We can use the functions from Appendix B

to get the ID of one such thread, and then use the OpenThread API to get a handle. We'll need to be able to get and set context, and\
also suspend and resume the thread.

**Code:**
```cpp
unsigned long threadID;
HANDLE hThread;

threadID = GetTargetThreadIdFromProcname(PROC_NAME);
hThread   = OpenThread((THREAD_GET_CONTEXT | THREAD_SET_CONTEXT | THREAD_SUSPEND_RESUME), false, threadID);
```

Now, we need to pause the thread in order to get it's "context". The context of a thread is the current state of all of it's\
registers, as well as other peripheral information. However, we're mostly concerned with the EIP register, which points to the\
next instruction to be executed. So, if we don't suspend the thread before retrieving its context information, it'll continue\
executing and by the time we get the information, it'll be invalid. Once we've paused the thread, we'll retrieve it's context\
information using the GetThreadContext() function. We'll grab the value\
of the current next instruction to be executed, so that we know where our stub should return to. Then it's just a matter of\
patching up the stub to have all of the proper pointers, and forcing the thread to execute it:

**Code:**
```cpp
SuspendThread(hThread);

ctx.ContextFlags = CONTEXT_CONTROL;
GetThreadContext(hThread, &ctx);
oldIP   = ctx.Eip;

//Set the EIP of the context to the address of our stub
ctx.Eip = (DWORD)stub;
ctx.ContextFlags = CONTEXT_CONTROL;

//Right now loadDll is code, which isn't writable.  We needto change that.
VirtualProtect(loadDll, stubLen, PAGE_EXECUTE_READWRITE, &oldprot);

//Patch the first push instruction
memcpy((void *)((unsigned long)loadDll + 1), &oldIP, 4);
//Patch the 2nd push instruction
memcpy((void *)((unsigned long)loadDll + 8), &dllString, 4);
//Patch the mov eax, 0xDEADBEEF to mov eax, LoadLibrary
memcpy((void *)((unsigned long)loadDll + 13), &loadLibAddy, 4);

WriteProcessMemory(hProcess, stub, loadDll, stubLen, NULL);         //Write the stub into the target
//Set the new context of the target's thread
SetThreadContext(hThread, &ctx);
//Let the target thread continue execution, starting at our stub
ResumeThread(hThread);
```

All that's left now, is to cleanup the evidence. Before we do that though, we should pause the injector for a bit, to be sure\
that the target has time to execute our stub(don't want any nasty race conditions). We'll use Sleep() to pause for 8 seconds before\
unmapping the memory that we allocated, and exiting the injector.

**Code:**
```cpp
Sleep(8000);
VirtualFreeEx(hProcess, dllString, strlen(DLL_NAME), MEM_DECOMMIT);
VirtualFreeEx(hProcess, stub, stubLen, MEM_DECOMMIT);
CloseHandle(hProcess);
CloseHandle(hThread);
```

This method should work on any version of windows, and should be the least likely to trigger any A/V alarms or cause the program\
to malfunction. If you can understand it and implement it properly, this is definitely the best of the three methods.

Appendix A - Methods of obtaining a process ID

If the process you're targeting has a window, you can use the FindWindow function, in conjunction with GetWindowThreadProcessId,\
as shown here:

**Code:**
```cpp
unsigned long GetTargetProcessIdFromWindow(char *className, char *windowName)
{
    unsigned long procID;
    HWND targetWnd;

    targetWnd = FindWindow(className, windowName);
    GetWindowThreadProcessId(targetWnd, &procId);

    return procID;
}
```

If you only know the name of the executable file or it just doesn't have a window:

**Code:**
```cpp
unsigned long GetTargetProcessIdFromProcname(char *procName)\
{\
   PROCESSENTRY32 pe;\
   HANDLE thSnapshot;\
   BOOL retval, ProcFound = false;

   thSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

   if(thSnapshot == INVALID_HANDLE_VALUE)\
   {\
      MessageBox(NULL, "Error: unable to create toolhelp snapshot", "Loader", NULL);\
      return false;\
   }

   pe.dwSize = sizeof(PROCESSENTRY32);

    retval = Process32First(thSnapshot, &pe);

   while(retval)\
   {\
      if(StrStrI(pe.szExeFile, procName) )\
      {\
         ProcFound = true;\
         break;\
      }

      retval    = Process32Next(thSnapshot,&pe);\
      pe.dwSize = sizeof(PROCESSENTRY32);\
   }

   return pe.th32ProcessID;\
}\
```

Appendix B - Methods of obtaining a thread ID

If the process you're targeting has a window, you can use the FindWindow function, in conjunction with GetWindowThreadProcessId\
and the toolhelp API, as shown here:

**Code:**
```cpp
unsigned long GetTargetThreadIdFromWindow(char *className, char *windowName)\
{\
    HWND targetWnd;\
    HANDLE hProcess\
    unsigned long processId, pTID, threadID;

    targetWnd = FindWindow(className, windowName);\
    GetWindowThreadProcessId(targetWnd, &processId);

    _asm {\
   mov eax, fs:[0x18]\
   add eax, 36\
   mov [pTID], eax\
    }

    hProcess = OpenProcess(PROCESS_VM_READ, false, processID);\
    ReadProcessMemory(hProcess, (const void *)pTID, &threadID, 4, NULL);\
    CloseHandle(hProcess);

    return threadID;\
}\
```

If you only know the name of the executable of your target, then you can use this code to locate it:

**Code:**
```cpp
unsigned long GetTargetThreadIdFromProcname(char *procName)\
{\
   PROCESSENTRY32 pe;\
   HANDLE thSnapshot, hProcess;\
   BOOL retval, ProcFound = false;\
   unsigned long pTID, threadID;

   thSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

   if(thSnapshot == INVALID_HANDLE_VALUE)\
   {\
      MessageBox(NULL, "Error: unable to create toolhelp snapshot", "Loader", NULL);\
      return false;\
   }

   pe.dwSize = sizeof(PROCESSENTRY32);

    retval = Process32First(thSnapshot, &pe);

   while(retval)\
   {\
      if(StrStrI(pe.szExeFile, procName) )\
      {\
         ProcFound = true;\
         break;\
      }

      retval    = Process32Next(thSnapshot,&pe);\
      pe.dwSize = sizeof(PROCESSENTRY32);\
   }

   CloseHandle(thSnapshot);

   _asm {\
      mov eax, fs:[0x18]\
      add eax, 36\
      mov [pTID], eax\
   }

   hProcess = OpenProcess(PROCESS_VM_READ, false, pe.th32ProcessID);\
   ReadProcessMemory(hProcess, (const void *)pTID, &threadID, 4, NULL);\
   CloseHandle(hProcess);

   return threadID;\
}\
```

Appendix C - CreateRemoteThread complete example source code

**Code:**
```cpp
#include <windows.h>\
#include <stdio.h>\
#include <tlhelp32.h>\
#include <shlwapi.h>

#define PROCESS_NAME "target.exe"\
#define DLL_NAME "injected.dll"

//I could just use PROCESS_ALL_ACCESS but it's always best to use the absolute bare minimum of priveleges, so that your code works in as\
//many circumstances as possible.\
#define CREATE_THREAD_ACCESS (PROCESS_CREATE_THREAD | PROCESS_QUERY_INFORMATION | PROCESS_VM_OPERATION | PROCESS_VM_WRITE | PROCESS_VM_READ)

BOOL WriteProcessBYTES(HANDLE hProcess,LPVOID lpBaseAddress,LPCVOID lpBuffer,SIZE_T nSize);

BOOL LoadDll(char *procName, char *dllName);\
BOOL InjectDLL(DWORD ProcessID, char *dllName);\
unsigned long GetTargetProcessIdFromProcname(char *procName);

bool IsWindowsNT()\
{\
   // check current version of Windows\
   DWORD version = GetVersion();\
   // parse return\
   DWORD majorVersion = (DWORD)(LOBYTE(LOWORD(version)));\
   DWORD minorVersion = (DWORD)(HIBYTE(LOWORD(version)));\
   return (version < 0x80000000);\
}

int WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)\
{\
    if(IsWindowsNT())\
       LoadDll(PROCESS_NAME, DLL_NAME);\
    else\
   MessageBox(0, "Your system does not support this method", "Error!", 0);

    return 0;\
}

BOOL LoadDll(char *procName, char *dllName)\
{\
   DWORD ProcID = 0;

   ProcID = GetProcID(procName);

   if(!(InjectDLL(ProcID, dllName)))\
      MessageBox(NULL, "Process located, but injection failed", "Loader", NULL);

   return true;\
}

BOOL InjectDLL(DWORD ProcessID, char *dllName)\
{\
   HANDLE Proc;\
   char buf[50]={0};\
   LPVOID RemoteString, LoadLibAddy;

   if(!ProcessID)\
      return false;

   Proc = OpenProcess(CREATE_THREAD_ACCESS, FALSE, ProcessID);

   if(!Proc)\
   {\
      sprintf(buf, "OpenProcess() failed: %d", GetLastError());\
      MessageBox(NULL, buf, "Loader", NULL);\
      return false;\
   }

   LoadLibAddy = (LPVOID)GetProcAddress(GetModuleHandle("kernel32.dll"), "LoadLibraryA");

   RemoteString = (LPVOID)VirtualAllocEx(Proc, NULL, strlen(DLL_NAME), MEM_RESERVE|MEM_COMMIT, PAGE_READWRITE);\
   WriteProcessMemory(Proc, (LPVOID)RemoteString, dllName, strlen(dllName), NULL);\
        CreateRemoteThread(Proc, NULL, NULL, (LPTHREAD_START_ROUTINE)LoadLibAddy, (LPVOID)RemoteString, NULL, NULL);

   CloseHandle(Proc);

   return true;\
}

unsigned long GetTargetProcessIdFromProcname(char *procName)\
{\
   PROCESSENTRY32 pe;\
   HANDLE thSnapshot;\
   BOOL retval, ProcFound = false;

   thSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

   if(thSnapshot == INVALID_HANDLE_VALUE)\
   {\
      MessageBox(NULL, "Error: unable to create toolhelp snapshot", "Loader", NULL);\
      return false;\
   }

   pe.dwSize = sizeof(PROCESSENTRY32);

   retval = Process32First(thSnapshot, &pe);

   while(retval)\
   {\
      if(StrStrI(pe.szExeFile, procName) )\
      {\
         ProcFound = true;\
         break;\
      }

      retval    = Process32Next(thSnapshot,&pe);\
      pe.dwSize = sizeof(PROCESSENTRY32);\
   }

   return pe.th32ProcessID;\
}\
```

Appendix D - SetWindowsHookEx complete example source code

**Code:**
```cpp
#include <windows.h>\
#include <tlhelp32.h>

#define PROC_NAME "target.exe"\
#define DLL_NAME "injected.dll"

void LoadDll(char *procName, char *dllName);\
unsigned long GetTargetThreadIdFromProcname(char *procName);

int WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)\
{\
    LoadDll(PROC_NAME, DLL_NAME);

    return 0;\
}

void LoadDll(char *procName, char *dllName)\
{\
    HMODULE hDll;\
    unsigned long cbtProcAddr;

    hDll        = LoadLibrary(dllName);\
    cbtProcAddr = GetProcAddress(hDll, "CBTProc");

    SetWindowsHookEx(WH_CBT, cbtProcAddr, hDll, GetTargetThreadIdFromProcName(procName));

    return TRUE;\
}

unsigned long GetTargetThreadIdFromProcname(char *procName)\
{\
   PROCESSENTRY32 pe;\
   HANDLE thSnapshot, hProcess;\
   BOOL retval, ProcFound = false;\
   unsigned long pTID, threadID;

   thSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

   if(thSnapshot == INVALID_HANDLE_VALUE)\
   {\
      MessageBox(NULL, "Error: unable to create toolhelp snapshot", "Loader", NULL);\
      return false;\
   }

   pe.dwSize = sizeof(PROCESSENTRY32);

    retval = Process32First(thSnapshot, &pe);

   while(retval)\
   {\
      if(StrStrI(pe.szExeFile, procName) )\
      {\
         ProcFound = true;\
         break;\
      }

      retval    = Process32Next(thSnapshot,&pe);\
      pe.dwSize = sizeof(PROCESSENTRY32);\
   }

   CloseHandle(thSnapshot);

   _asm {\
      mov eax, fs:[0x18]\
      add eax, 36\
      mov [pTID], eax\
   }

   hProcess = OpenProcess(PROCESS_VM_READ, false, pe.th32ProcessID);\
   ReadProcessMemory(hProcess, (const void *)pTID, &threadID, 4, NULL);\
   CloseHandle(hProcess);

   return threadID;\
}\
```

Appendix E - Code cave example source code

**Code:**
```cpp
#include <windows.h>\
#include <tlhelp32.h>\
#include <shlwapi.h>

#define PROC_NAME "target.exe"\
#define DLL_NAME "injected.dll"

unsigned long GetTargetProcessIdFromProcname(char *procName);\
unsigned long GetTargetThreadIdFromProcname(char *procName);

__declspec(naked) loadDll(void)\
{\
   _asm{\
      //   Placeholder for the return address\
      push 0xDEADBEEF

      //   Save the flags and registers\
      pushfd\
      pushad

      //   Placeholder for the string address and LoadLibrary\
      push 0xDEADBEEF\
      mov eax, 0xDEADBEEF

      //   Call LoadLibrary with the string parameter\
      call eax

      //   Restore the registers and flags\
      popad\
      popfd

      //   Return control to the hijacked thread\
      ret\
   }\
}

__declspec(naked) loadDll_end(void)\
{\
}

int WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)\
{\
   void *dllString;\
   void *stub;\
   unsigned long wowID, threadID, stubLen, oldIP, oldprot, loadLibAddy;\
    HANDLE hProcess, hThread;\
   CONTEXT ctx;

   stubLen = (unsigned long)loadDll_end - (unsigned long)loadDll;

   loadLibAddy = (unsigned long)GetProcAddress(GetModuleHandle("kernel32.dll"), "LoadLibraryA");

   wowID    = GetTargetProcessIdFromProcname(PROC_NAME);\
   hProcess = OpenProcess((PROCESS_VM_WRITE | PROCESS_VM_OPERATION), false, wowID);

   dllString = VirtualAllocEx(hProcess, NULL, (strlen(DLL_NAME) + 1), MEM_COMMIT, PAGE_READWRITE);\
   stub      = VirtualAllocEx(hProcess, NULL, stubLen, MEM_COMMIT, PAGE_EXECUTE_READWRITE);\
   WriteProcessMemory(hProcess, dllString, DLL_NAME, strlen(DLL_NAME), NULL);

   threadID = GetTargetThreadIdFromProcname(PROC_NAME);\
   hThread   = OpenThread((THREAD_GET_CONTEXT | THREAD_SET_CONTEXT | THREAD_SUSPEND_RESUME), false, threadID);\
   SuspendThread(hThread);

   ctx.ContextFlags = CONTEXT_CONTROL;\
   GetThreadContext(hThread, &ctx);\
   oldIP   = ctx.Eip;\
   ctx.Eip = (DWORD)stub;\
   ctx.ContextFlags = CONTEXT_CONTROL;

   VirtualProtect(loadDll, stubLen, PAGE_EXECUTE_READWRITE, &oldprot);\
   memcpy((void *)((unsigned long)loadDll + 1), &oldIP, 4);\
   memcpy((void *)((unsigned long)loadDll + 8), &dllString, 4);\
   memcpy((void *)((unsigned long)loadDll + 13), &loadLibAddy, 4);

    WriteProcessMemory(hProcess, stub, loadDll, stubLen, NULL);\
   SetThreadContext(hThread, &ctx);

   ResumeThread(hThread);

   Sleep(8000);

   VirtualFreeEx(hProcess, dllString, strlen(DLL_NAME), MEM_DECOMMIT);\
   VirtualFreeEx(hProcess, stub, stubLen, MEM_DECOMMIT);\
   CloseHandle(hProcess);\
   CloseHandle(hThread);

    return 0;\
}

unsigned long GetTargetProcessIdFromProcname(char *procName)\
{\
   PROCESSENTRY32 pe;\
   HANDLE thSnapshot;\
   BOOL retval, ProcFound = false;

   thSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

   if(thSnapshot == INVALID_HANDLE_VALUE)\
   {\
      MessageBox(NULL, "Error: unable to create toolhelp snapshot", "Loader", NULL);\
      return false;\
   }

   pe.dwSize = sizeof(PROCESSENTRY32);

    retval = Process32First(thSnapshot, &pe);

   while(retval)\
   {\
      if(StrStrI(pe.szExeFile, procName) )\
      {\
         ProcFound = true;\
         break;\
      }

      retval    = Process32Next(thSnapshot,&pe);\
      pe.dwSize = sizeof(PROCESSENTRY32);\
   }

   CloseHandle(thSnapshot);\
   return pe.th32ProcessID;\
}

unsigned long GetTargetThreadIdFromProcname(char *procName)\
{\
   PROCESSENTRY32 pe;\
   HANDLE thSnapshot, hProcess;\
   BOOL retval, ProcFound = false;\
   unsigned long pTID, threadID;

   thSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

   if(thSnapshot == INVALID_HANDLE_VALUE)\
   {\
      MessageBox(NULL, "Error: unable to create toolhelp snapshot", "Loader", NULL);\
      return false;\
   }

   pe.dwSize = sizeof(PROCESSENTRY32);

    retval = Process32First(thSnapshot, &pe);

   while(retval)\
   {\
      if(StrStrI(pe.szExeFile, procName) )\
      {\
         ProcFound = true;\
         break;\
      }

      retval    = Process32Next(thSnapshot,&pe);\
      pe.dwSize = sizeof(PROCESSENTRY32);\
   }

   CloseHandle(thSnapshot);

   _asm {\
      mov eax, fs:[0x18]\
      add eax, 36\
      mov [pTID], eax\
   }

   hProcess = OpenProcess(PROCESS_VM_READ, false, pe.th32ProcessID);\
   ReadProcessMemory(hProcess, (const void *)pTID, &threadID, 4, NULL);\
   CloseHandle(hProcess);

   return threadID;\
}
```

----
