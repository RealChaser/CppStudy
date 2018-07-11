파일 전체 읽은 후 tchar(wchar_t)로 변환하기
-----------------------------------------
	#include <tchar.h>
	#include <string>
	#include <atlstr.h>
	#include <fstream>

	using namespace std;
	
	void testFunc(const TCHAR* buffer)
	{
		_asm nop;
	}
	
	int main()
	{
		std::ifstream t("test.json");
		std::string str((std::istreambuf_iterator<char>(t)), std::istreambuf_iterator<char>());	
		wstring wstr(CA2CT(str.c_str()));
	
		testFunc(wstr.c_str());
	
		return 0;
	}