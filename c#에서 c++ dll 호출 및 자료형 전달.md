c#에서 c++ dll 호출 및 디버깅 하기
ExAdd -> int 2개를 받고 더해서 리턴
ExStr -> 문자열을 c++코드로 넘기고 c++에서 생성한 문자열을 리턴 받음

- x64, x86 상관없음
- 유니코드로 할거면 CharSet = CharSet.Unicode 필요하다 아니면 지워야 한다.
- c의 char 자료형은 c# 에선 stringbuilder와 대응된다.

----------------------------------
C# 코드
----------------------------------

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Runtime.InteropServices;
	using System.Text;
	using System.Threading.Tasks;
	
	/* c#에서 dll로드한 c++코드 디버깅 하기
	 하나의 솔루션에 c++ c# 프로젝트가 있어야 한다.
	  - 가장 중요한 것!! tools > options >  debugging > general > 마지막줄의 use managed compatibility mode 체크.
	  - 솔루션에 속성 설정 > common properties > project dependencies >  여기서 c# 프로젝트는 c++ dll 프로젝트에 의존한다고 체크
	  - c#프로젝트에 속성 설정 > debug > Enable Debuggers > Enable native code debugging 체크!!!
	  - c#프로젝트를 set as startup project
	 */
	
	namespace ConsoleApplication2
	{
	    class Program
	    {
	        [DllImport("DynamicExplicitDLL.dll", CallingConvention = CallingConvention.Cdecl)]
	        extern public static int ExAdd(int a, int b);
	
	        [DllImport("DynamicExplicitDLL.dll", CallingConvention = CallingConvention.Cdecl, CharSet = CharSet.Unicode)]
	        extern public static IntPtr ExStr(StringBuilder a);
	
	        static void Main(string[] args)
	        {
	            IntPtr a = ExStr(new StringBuilder("aaaaaa33aaaaaaaaa"));
	            string s = Marshal.PtrToStringUni(a);
	            System.Console.WriteLine(s);
	
	            System.Console.WriteLine(ExAdd(2, 5));
	        }
	    }
	}

---------------------------------------------------
C++ 코드
---------------------------------------------------
	//.h 파일
	#pragma once
	
	#ifdef	__cplusplus
	#define		EXTERN_FORM		extern "C"
	#else
	#define		EXTERN_FORM
	#endif
	
	#ifdef EX_EXPORTS
	#define DLL_API _declspec(dllexport)
	#else
	#define DLL_API _declspec(dllimport)
	#endif
	
	EXTERN_FORM DLL_API int __cdecl ExAdd( int a, int b );
	
	EXTERN_FORM DLL_API wchar_t* __cdecl ExStr(wchar_t* a);
	//명시하지 않으면 기본 호출 규약 __cdecl 이 된다.

----------------------------------------------------

	//.cpp 파일
	#define EX_EXPORTS
	#include "ExplicitDllEx.h"
	#include <tchar.h>
	#include <string>
	
	EXTERN_FORM DLL_API int __cdecl ExAdd( int a, int b )
	{
		return a + b;
	}
	
	EXTERN_FORM DLL_API wchar_t* __cdecl ExStr(wchar_t* a)
	{	
		static std::wstring ss(_T("asdfasd"));
		return const_cast<wchar_t*>(ss.c_str());
	}