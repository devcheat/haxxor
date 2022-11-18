Hooking Notepad.exe
===
The following code shows how to hook the notepad.dll

```cpp
//Hooking Notepad in c++
#include <windows.h>;

    int main()
    {
        HOOKPROC hkprcSysMsg; 
        static HINSTANCE hinstDLL; 
        static HHOOK hhookSysMsg; 
    
        hinstDLL = LoadLibrary((LPCTSTR) "c:\\windows\\notepad.dll"); 
        hkprcSysMsg = (HOOKPROC)GetProcAddress(hinstDLL, "SysMessageProc"); 
        hhookSysMsg = SetWindowsHookEx(WH_SYSMSGFILTER,hkprcSysMsg,hinstDLL,0); 
        return  0;
    }
 ```
