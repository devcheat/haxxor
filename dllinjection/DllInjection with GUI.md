DLL injection with GUI
===

Example of DLL injection with a GUI in c++

```cpp
//DLL injection with GUI
#include 
#include 

/*  Declare Windows procedure  */
LRESULT CALLBACK WindowProcedure (HWND, UINT, WPARAM, LPARAM);

/*  Make the class name into a global variable  */
char szClassName[ ] = "WindowsApp";

int WINAPI WinMain (HINSTANCE hThisInstance,
                    HINSTANCE hPrevInstance,
                    LPSTR lpszArgument,
                    int nFunsterStil)

{
    HWND hwnd;            
    MSG messages;          
    WNDCLASSEX wincl;    
    wincl.hInstance = hThisInstance;
    wincl.lpszClassName = szClassName;
    wincl.lpfnWndProc = WindowProcedure;    
    wincl.style = CS_DBLCLKS;                
    wincl.cbSize = sizeof (WNDCLASSEX);
    wincl.hIcon = LoadIcon (NULL, IDI_APPLICATION);
    wincl.hIconSm = LoadIcon (NULL, IDI_APPLICATION);
    wincl.hCursor = LoadCursor (NULL, IDC_ARROW);
    wincl.lpszMenuName = NULL;                
    wincl.cbClsExtra = 0;                    
    wincl.cbWndExtra = 0;                    
    wincl.hbrBackground = (HBRUSH) COLOR_BACKGROUND+7;
  
    if (!RegisterClassEx (&wincl))
        return 0;
      hwnd = CreateWindowEx (
          0,                  
          szClassName,        
          "The Game Injector ",      
          WS_SYSMENU|WS_VISIBLE, 
          CW_USEDEFAULT,      
          CW_USEDEFAULT,      
          400,                
          200,              
          HWND_DESKTOP,        
          NULL,                
          hThisInstance,      
          NULL                
          );
    
    
    while (GetMessage (&messages, NULL, 0, 0))
    {
        TranslateMessage(&messages);
        DispatchMessage(&messages);
    }
    return messages.wParam;
}
HWND Input1,Input2;
HWND Inject;

BOOL SetPrivilege(LPSTR type) // more flexible 
{
HANDLE Htoken;
TOKEN_PRIVILEGES tokprivls;
if(!OpenProcessToken( GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES, &Htoken)){
                      return 0;
                      }
tokprivls.PrivilegeCount = 1;
LookupPrivilegeValue(NULL, type, &tokprivls.Privileges[0].Luid);
tokprivls.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
BOOL Success =AdjustTokenPrivileges( Htoken, FALSE, &tokprivls, sizeof(tokprivls), NULL, NULL);
CloseHandle(Htoken);
return Success;

}
HANDLE GetHandle(char *proc)
{
      PROCESSENTRY32 pe32;
      pe32.dwSize = sizeof(pe32);
      HANDLE Snap = CreateToolhelp32Snapshot(TH32CS_SNAPALL,0);
      Process32First(Snap,&pe32);
      do{
          if(stricmp(pe32.szExeFile,proc)==0)
          {
                                            SetPrivilege(SE_DEBUG_NAME);
                                            return OpenProcess(PROCESS_ALL_ACCESS,0,pe32.th32ProcessID);
          }}while(Process32Next(Snap,&pe32));CloseHandle(Snap);
}
void InjectDll(char* Name, char *path)
{
HANDLE hProcess = GetHandle(Name);
if(hProcess){
            int DllPath = strlen(path) + 20; 
            LPVOID MemSp = VirtualAllocEx(hProcess,NULL,DllPath,MEM_COMMIT,PAGE_READWRITE);
            WriteProcessMemory(hProcess,MemSp,path,DllPath,NULL);
            HANDLE hThread = CreateRemoteThread(hProcess,NULL,0,(LPTHREAD_START_ROUTINE)GetProcAddress(LoadLi
brary("Kernel32.dll"), "LoadLibraryA"), MemSp, 0, NULL);
            if(hThread){
                        WaitForSingleObject(hThread, 30000); 
                        CloseHandle(hThread);
                                }
                        VirtualFreeEx(hProcess, MemSp, 0, MEM_RELEASE);
            }
else {MessageBox(0,"Could not get the process handle .",0,0);}            
}
                        
char proc[50],dll[260];
LRESULT CALLBACK WindowProcedure (HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam)
{
    
  HWND hBmpStat;
    HBITMAP hBitmap;
    HFONT hFont ;
    switch (message)              
    {
        case WM_CREATE:
            hFont = CreateFont(20, 0, 0, 10, FW_DONTCARE, 0, 0, 0, ANSI_CHARSET, OUT_TT_PRECIS, CLIP_TT_ALWAYS, DEFAULT_QUALITY, FF_DONTCARE, "Microsoft Sans MS");
            
            hBitmap  = (HBITMAP) LoadImage(NULL, "C:\\WINDOWS\\system32\\setup.bmp", IMAGE_BITMAP, 0, 0, LR_LOADFROMFILE);
            // zomfg h4x
            hBmpStat = CreateWindowEx(0,"Static","",WS_VISIBLE | WS_CHILD | SS_BITMAP,
                      -200,-220,0,0,hwnd,0,0,0);
            
            SendMessage(hBmpStat, STM_SETIMAGE, IMAGE_BITMAP, (LPARAM) hBitmap);
            
            Inject = CreateWindow("Button","INJECT",WS_CHILD | WS_VISIBLE | WS_BORDER,
                      190, 20, 180, 38,hwnd,(HMENU)100,0,NULL);
            Input1 = CreateWindow("Edit", "wmplayer.exe",WS_CHILD | WS_VISIBLE | WS_BORDER,
                      10, 20, 180,18,hwnd,0,0,NULL);
            Input2 = CreateWindow("Edit", "c:\\sample.dll",WS_CHILD | WS_VISIBLE | WS_BORDER,
                      10, 40, 180,18,hwnd,0,0,NULL);
                      SendMessage(Inject,WM_SETFONT,WPARAM(hFont),0);
                      break;  
        case WM_DESTROY:
            PostQuitMessage (0);      
            break;
            case WM_COMMAND:
                switch(LOWORD(wParam))
                      {
                          case 100:
                              SendMessage(Input1,WM_GETTEXT,sizeof(proc),LPARAM(proc));
                              if(proc!=0)
                              {
                                          SendMessage(Input2,WM_GETTEXT,sizeof(dll),LPARAM(dll));
                                          if(dll!=0)
                                          InjectDll(proc,dll);
                              }break;
                        default:break;        
                                            }break;
                
        default:                    
            return DefWindowProc (hwnd, message, wParam, lParam);
    }
    return 0;
}
```
