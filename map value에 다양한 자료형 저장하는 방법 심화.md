std::map의 value로 상속 관계의 자료형을 value로 갖는 multimap(map) 집어넣기
결국 map의 value로 아무거나 넣을수 있도록 한다.

-----------------------------------
	#include <map>
	#include <string>
	#include <memory>
	#include <iostream>
	#include <vector>
	
	using namespace std;
	
	class DataBase
	{
	public:	
		virtual const char* _GetName() = 0;
		virtual ~DataBase(){}
	};
	
	class DataItem: public DataBase
	{
	public:
		virtual const char* _GetName()
		{
			return GetName();
		}
		static const char* GetName()
		{
			return "Item";
		}
	};
	
	class AnyBase
	{
	public:
		virtual ~AnyBase() = 0;	//이 기본 자료형으로의 생성을 막고 상속받아서 사용해야 하므로 소멸자를 순수가상으로 지정
	};
	inline AnyBase::~AnyBase(){}
	
	template<class T>
	class Any: public AnyBase
	{
	public:
		explicit Any(const T& data): data(data) {}
	
	public:
		T& GetTypeData()
		{
			return data;
		}
	
	private:
		T data;
	};
	
	std::map<std::string, std::shared_ptr<AnyBase>> anymap;
	
	template<typename T>
	void MakeFullLoadDB(vector<shared_ptr<T>>& vec)
	{
		multimap<int, shared_ptr<T>> setMultiMap;
	
		for(unsigned int i = 0; i < vec.size(); ++i)
		{
			setMultiMap.insert({i, vec[i]});
		}
	
		anymap.insert({T::GetName(), shared_ptr<Any<multimap<int, shared_ptr<T>>>>(new Any<multimap<int, shared_ptr<T>>>(setMultiMap))});
	}
	
	template<typename T>
	multimap<int, shared_ptr<T>> GetFullLoadedDB()
	{
		return dynamic_cast<Any<multimap<int, shared_ptr<DataItem>>>&>(*anymap[T::GetName()]).GetTypeData();
	}
	
	int main()
	{
		{
		vector<shared_ptr<DataItem>> vec;	
		vec.push_back(shared_ptr<DataItem>(new DataItem()));
		vec.push_back(shared_ptr<DataItem>(new DataItem()));
		vec.push_back(shared_ptr<DataItem>(new DataItem()));
		MakeFullLoadDB<DataItem>(vec);
		}	
	
		auto loadedDB = GetFullLoadedDB<DataItem>();
		shared_ptr<DataItem> item = loadedDB.find(2)->second;
		cout << item->GetName() << endl;
	
		//-----------------------------------------------------------------
		//Use Example
		//anymap["number"].reset(new Any<int>(5));
		//int value = dynamic_cast<Any<int>&>(*anymap["number"]).data;
		//-----------------------------------------------------------------
	}
