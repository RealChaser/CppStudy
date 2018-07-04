템플릿으로 사용자 자료형 Get, Set 함수 만들기
----------------------------------------------------

	#include <iostream>
	using namespace std;
	
	struct CountType
	{
		int value;
	};
	
	class CItemData
	{
	public:
		template<typename T> struct TypeValue;
		template<typename T> T GetFunc();
		template<typename T> void SetFunc(T);
		template<typename T> typename TypeValue<T>::Type GetFuncValue();
		template<typename T> void SetFuncValue(typename TypeValue<T>::Type);
	
	public:
		template<>
		struct TypeValue<CountType>
		{
			using Type = int;
		};
	
		template<>
		typename TypeValue<CountType>::Type GetFuncValue<CountType>()
		{
			return countType.value;
		}
	
		template<>
		void SetFuncValue<CountType>(typename TypeValue<CountType>::Type _value)
		{
			countType.value = _value;
		}
	
		template<>
		CountType GetFunc()
		{
			return countType;
		}
	
		template<>
		void SetFunc(CountType _countType)
		{
			countType = _countType;
		}
	
	private:
		CountType countType;
	};
	
	void main()
	{
		CItemData itemData;
	
		CountType count;
		count.value = 5;
		
		itemData.SetFunc(count);
		cout << itemData.GetFunc<CountType>().value << endl;
	
		itemData.SetFuncValue<CountType>(66);
		cout << itemData.GetFuncValue<CountType>() << endl;
	}

---------------------------------------------
위 코드 매크로로 작성
----------------------------------------------
	#include <iostream>
	using namespace std;
	
	#define MakeTypeDefine(TypeName, ValueType, valueName)		\
	struct TypeName	{	ValueType valueName;	}
	
	#define MakeInstant(SpecializeType, InstantName)	\
	private:									\
		SpecializeType InstantName;	
	
	#define MakeBaseTemplate( GetFuncName, SetFuncName, GetFuncValueName, SetFuncValueName)	\
	public:																								\
		template<typename T> struct TypeValue;															\
		template<typename T> T GetFuncName();															\
		template<typename T> void SetFuncName(T);														\
		template<typename T> typename TypeValue<T>::Type GetFuncValueName();							\
		template<typename T> void SetFuncValueName(typename TypeValue<T>::Type);
	
	#define MakeSpecializeTemplate(InstantName, SpecializeType, ValueType, GetFuncName, SetFuncName, GetFuncValueName, SetFuncValueName)		\
	public:																																		\
		template<>	struct TypeValue<SpecializeType>	{ using Type = ValueType;	};															\
		template<>	typename TypeValue<SpecializeType>::Type GetFuncValueName<SpecializeType>(){	return InstantName.value;	}				\
		template<>	void SetFuncValueName<SpecializeType>(typename TypeValue<SpecializeType>::Type _value){	InstantName.value = _value;	}		\
		template<>	SpecializeType GetFuncName(){ return InstantName;	}																		\
		template<>	void SetFuncName(SpecializeType _type){	InstantName = _type;}																		
	
	#define MakeType( SpecializeType, InstantName, ValueType, GetFuncName, SetFuncName, GetFuncValueName, SetFuncValueName)				\
			MakeInstant(SpecializeType, InstantName)																							\
			MakeBaseTemplate( GetFuncName, SetFuncName, GetFuncValueName, SetFuncValueName)														\
			MakeSpecializeTemplate(InstantName, SpecializeType, ValueType, GetFuncName, SetFuncName, GetFuncValueName, SetFuncValueName)
	
	
	MakeTypeDefine(CountType, int, value);
	
	class CItemData
	{
		MakeType(CountType, countType, int, GetFunc, SetFunc, GetFuncValue, SetFuncValue);
	};
	
	void main()
	{
		CItemData itemData;
	
		CountType count;
		count.value = 5;
	
		itemData.SetFunc(count);
		cout << itemData.GetFunc<CountType>().value << endl;
	
		itemData.SetFuncValue<CountType>(66);
		cout << itemData.GetFuncValue<CountType>() << endl;
	}