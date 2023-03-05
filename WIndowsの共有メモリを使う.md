#単純なサンプル
```C++
//2019-05-03 gpsnmeajp
//These codes are licensed under CC0.
//http://creativecommons.org/publicdomain/zero/1.0/deed.ja
#include <windows.h>
#include <stdio.h>
#include <conio.h>

int main(void){
	//--------------共有メモリ------------
	DWORD MaxSizeLow = 100;
	//ハンドルのオープン
	HANDLE FileHandle = CreateFileMappingA(INVALID_HANDLE_VALUE, NULL, PAGE_READWRITE, 0, MaxSizeLow, "MEM_SEND");
	//ハンドルのオープンに失敗
	if (FileHandle == NULL)
	{
		return -1;
	}

	//メモリのマッピング
	LPVOID Mapping = MapViewOfFile(FileHandle, FILE_MAP_ALL_ACCESS, 0, 0, 0);
	//マッピングに失敗
	if (Mapping == NULL)
	{
		CloseHandle(FileHandle);
		return -1;
	}
	char *SharedRam = (char*)Mapping;

	SharedRam[0] = '\0';
	while (!_kbhit())
	{
		printf("<-%s\n", SharedRam);
		//Sleep(100);
	}

	//メモリの開放
	UnmapViewOfFile(Mapping);
	CloseHandle(FileHandle);


	return 0;
}
```

#クラス化

```C++
//2019-05-04 gpsnmeajp
//These codes are licensed under CC0.
//http://creativecommons.org/publicdomain/zero/1.0/deed.ja

#include <windows.h>
#include <stdio.h>
#include <conio.h>

//----------共有メモリ-----------

class SharedMemory{
private:
	DWORD SharedMemorySize = 16 * 1024; //16KB
	HANDLE SharedMemoryHandle = NULL;
	LPVOID SharedMemoryBuffer = NULL;

public:
	SharedMemory()
	{
	}

	SharedMemory(const char* Pipename)
	{
		printf(Pipename);
		open(Pipename);
	}

	~SharedMemory()
	{
		close();
	}

	LPVOID get_pointer(){
		return SharedMemoryBuffer;
	}

	bool is_open(){
		return (SharedMemoryBuffer != NULL) && (SharedMemoryHandle != NULL);
	}

	void set_size(DWORD _SharedMemorySize)
	{
		SharedMemorySize = _SharedMemorySize;
	}

	DWORD get_size()
	{
		return SharedMemorySize;
	}

	void print(const char *fmt, ...){
		va_list va;
		va_start(va, fmt);
		vsprintf_s((char*)SharedMemoryBuffer, SharedMemorySize-1, fmt, va);
		va_end(va);
	}

	void open(const char* Pipename){
		//もし仮にすでに開いている場合閉じる
		close();

		//ハンドルのオープン
		SharedMemoryHandle = CreateFileMappingA(INVALID_HANDLE_VALUE, NULL, PAGE_READWRITE, 0, SharedMemorySize, Pipename);
		//ハンドルのオープンに失敗
		if (SharedMemoryHandle == NULL)
		{
			return;
		}

		//メモリのマッピング
		SharedMemoryBuffer = MapViewOfFile(SharedMemoryHandle, FILE_MAP_ALL_ACCESS, 0, 0, 0);
		//マッピングに失敗
		if (SharedMemoryBuffer == NULL)
		{
			//先に開いたハンドルを閉じる
			CloseHandle(SharedMemoryHandle);
			return;
		}
	}

	void close(){
		if (SharedMemoryBuffer != NULL){
			UnmapViewOfFile(SharedMemoryBuffer);
		}
		if (SharedMemoryHandle != NULL){
			CloseHandle(SharedMemoryHandle);
		}
	}
};

int main(void){
	SharedMemory comm("pip1");
	if (!comm.is_open())
	{
		return -1;
	}

	char *SharedRam = (char*)comm.get_pointer();

	int i = 0;
	while (!_kbhit())
	{
		comm.print("Hello World %d", i);
		printf("%s\n", SharedRam);
		i++;
	}

	return 0;
}

```

#C＃
MS公式サンプルの改造

```csharp
//https://docs.microsoft.com/ja-jp/dotnet/api/system.io.memorymappedfiles.memorymappedfile?view=netframework-4.8

using System;
using System.IO;
using System.Text;
using System.IO.MemoryMappedFiles;
using System.Runtime.InteropServices;


namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            using (var mmf = MemoryMappedFile.CreateOrOpen(@"pip1", 1024 * 16))
            {
                using (var accessor = mmf.CreateViewAccessor(0, 0))
                {
                    while (!Console.KeyAvailable)
                    {
                        byte[] buf = new byte[1024 * 16];
                        accessor.ReadArray(0, buf, 0, 1024 * 16);

                        int count = 0;
                        for (int i = 0; i < buf.Length; i++)
                        {
                            if (buf[i] == '\0')
                            {
                                Array.Resize<byte>(ref buf, i); //切り詰め
                                break;
                            }
                        }

                        string s = Encoding.ASCII.GetString(buf);
                        Console.WriteLine(s);
                    }
                }
            }
        }
    }
}

```

