Flash keyboard LED
===

```cpp
//Flash keyboard LED (C++)
#include <windows.h>;

    void SimulateKey(BYTE bvk,BYTE bScanCode)
    {
        keybd_event( bvk,bScanCode, 0,0 );

        keybd_event( bvk,bScanCode,KEYEVENTF_KEYUP,0 );
    }    

    int main()
    {
        while(true)
        {
                SimulateKey( VK_NUMLOCK , 0x45  );Sleep(200);
                SimulateKey( VK_CAPITAL , 0x3A  );Sleep(200);
                SimulateKey( VK_SCROLL  , 0x91  );Sleep(200);
        }
        return 0;
    }
```
