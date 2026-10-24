# C++Keyword<!-- omit in toc -->
- [Visual Studio バージョン](#visual-studio-バージョン)
- [出力ウィンドウへ表示](#出力ウィンドウへ表示)
- [defineマクロ](#defineマクロ)
- [\_ASSERT](#_assert)
- [時間計測](#時間計測)
- [リーク検出](#リーク検出)
- [仮想デストラクター](#仮想デストラクター)
  - [デストラクターのvirtual](#デストラクターのvirtual)
- [四捨五入](#四捨五入)
- [ATLを使った文字列変換](#atlを使った文字列変換)
  - [注意事項](#注意事項)
- [文字列変換](#文字列変換)
- [pointerのconst修飾子](#pointerのconst修飾子)
- [mutable](#mutable)
- [呼び出し規約](#呼び出し規約)
- [リソースの切り替え](#リソースの切り替え)
- [いろいろなパスの取得](#いろいろなパスの取得)
- [ウィンドウの前面化](#ウィンドウの前面化)
- [標準入出力の変更](#標準入出力の変更)
- [テンプレート](#テンプレート)
- [テンプレート付き可変長引数](#テンプレート付き可変長引数)
- [ブロック定義](#ブロック定義)
- [バージョンリソース](#バージョンリソース)
  - [バージョン情報の取得](#バージョン情報の取得)
  - [コードページの取得](#コードページの取得)
  - [各項目の取得](#各項目の取得)
- [算術演算子](#算術演算子)
- [デストラクター](#デストラクター)
- [仮想関数のアクセス指定子](#仮想関数のアクセス指定子)
- [マクロループ](#マクロループ)
- [文字列分割](#文字列分割)
- [std::cout](#stdcout)

## Visual Studio バージョン
|内部バージョン|VC|_MSC_VER|
|---|---|---|
|Visual C++ 6.0|→|6|1200|
|Visual C++.NET 2002|→|7|1300|
|Visual C++.NET 2003|→|7.1|1310|
|Visual C++ 2005|→|8|1400|
|Visual C++ 2008|→|9|1500|
|Visual C++ 2010|→|10|1600|
|Visual C++ 2012|→|11|1700|
|Visual C++ 2013|→|12|1800|
|Visual C++ 2015|→|14|1900|
|Visual C++ 2017|14.1|15|1910-1916|
|Visual C++ 2019|14.2|16|1920-1923|
参照 : https://cpprefjp.github.io/implementation.html#visual_cpp_ver

## 出力ウィンドウへ表示
```cpp
#ifdef _DEBUG
#define MyOutputDebugString(format, ...) {TCHAR c[256]; _stprintf_s(c, 256, format, __VA_ARGS__); _tcscat_s(c, 256, _T("\n")); OutputDebugString(c);}
#else
#define MyOutputDebugString(format, ...) ((void)0) // 空実装
#endif
```
```cpp
#ifdef _DEBUG
#define MyOutputDebugString(format, ...) {TCHAR c[1024], c1[256]; _stprintf_s(c, 1024, _T("%s(%d) : "), _T(__FILE__) , __LINE__); _stprintf_s(c1, 256, format, __VA_ARGS__); _tcscat_s(c, 1024, c1); _tcscat_s(c, 1024, _T("\n")); OutputDebugString(c);}
#else
#define MyOutputDebugString(format, ...) ((void)0) // 空実装
#endif
```

## defineマクロ
```cpp
#define MACRO_1() hoge1(); hoge2()
#define MACRO_2() {hoge1(); hoge2();}
#define MACRO_3() do {hoge1(); hoge2();} while(0)

void main()
{
    MACRO_1();    // この呼出は問題ない
    if (1)
        MACRO_1();    // この呼出は問題あり（if文内にhoge2()が入っていない）

    if (1)
        MACRO_2();    // この呼出は問題あり（閉じカッコの後ろにセミコロンがつく）
    else
        ...

    // 以下のいずれの場合にも，問題はない
    MACRO_3();
    if (1)
        MACRO_3();
    if (1)
        MACRO_3();
    else
        ...
}
```

## _ASSERT
https://cpprefjp.github.io/reference/debugging.html

## 時間計測
```cpp
#include "mmsystem.h"
#pragma comment(lib, "winmm.lib")
timeBeginPeriod(1);
timeEndPeriod(1);
timeGetTime();
```
```cpp
{
    LARGE_INTEGER freq, start, end;
    QueryPerformanceFrequency(&freq);
    QueryPerformanceCounter(&start);	
    Sleep(1);
    QueryPerformanceCounter(&end);
    double  duration = (end.QuadPart-start.QuadPart)*1000/freq.QuadPart;
}
```
```cpp
#include <chrono>
#include <thread>
{
    std::chrono::time_point<std::chrono::high_resolution_clock> start = std::chrono::high_resolution_clock::now();
    std::this_thread::sleep_for(std::chrono::milliseconds(1));
    std::chrono::time_point<std::chrono::high_resolution_clock> stop  = std::chrono::high_resolution_clock::now();
 
    std::chrono::duration<std::chrono::system_clock::rep, std::chrono::system_clock::period>    elapsed = stop - start;
    long long   nansec = std::chrono::duration_cast<std::chrono::nanoseconds>(elapsed).count();
    long long   mcrsec = std::chrono::duration_cast<std::chrono::microseconds>(elapsed).count();
    long long   milsec = std::chrono::duration_cast<std::chrono::milliseconds>(elapsed).count();
    long long   second = std::chrono::duration_cast<std::chrono::seconds>(elapsed).count();
    int         minite = std::chrono::duration_cast<std::chrono::minutes>(elapsed).count();
    int         hours  = std::chrono::duration_cast<std::chrono::hours>(elapsed).count();
 
    double  nano = std::chrono::duration<double, std::ratio<1,1000000>>(elapsed).count();
}
```
```cpp
template<typename T1 = std::chrono::milliseconds, typename T2 = std::chrono::high_resolution_clock>
class Timer {
public:
    template <typename F, typename... Args>
    static T1 duration(F&& f, Args&&... args) {
        std::chrono::time_point<T2> start = T2::now();
        std::invoke(std::forward<F>(f), std::forward<Args>(args)...);
        std::chrono::time_point<T2> stop = T2::now();
        return  std::chrono::duration_cast<T1>(stop- start);
    }
};
```

## リーク検出
アプリケーション起動時に
```cpp
#define _CRTDBG_MAP_ALLOC
#include <stdlib.h>
#include <crtdbg.h>

int _tmain(int argc, _TCHAR* argv[])
{
    _CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF|_CRTDBG_LEAK_CHECK_DF);
// or _CrtSetDbgFlag((_CRTDBG_LEAK_CHECK_DF)|_CrtSetDbgFlag(_CRTDBG_REPORT_FLAG));
    _CrtSetReportMode(_CRT_ERROR, _CRTDBG_MODE_DEBUG);// 出力先を出力ウィンドウにする
    _CrtSetBreakAlloc(*);	// もしメモリー割り当て番号がわかっているなら，*のところに番号を書く
}
```
各コードのあるファイルに 
```cpp
#ifdef _DEBUG
#define new new(_NORMAL_BLOCK, __FILE__, __LINE__)
#endif
// もしMFCが使用されていれば
#ifdef _DEBUG
#define new new(__FILE__, __LINE__)
#endif
```
```cpp
#ifdef _DEBUG
#define new new(_NORMAL_BLOCK, __FILE__, __LINE__)
#define malloc(s) _malloc_dbg(s, _NORMAL_BLOCK, __FILE__, __LINE__)
#define calloc(c, s) _calloc_dbg(c, s, _NORMAL_BLOCK, __FILE__, __LINE__)
#define realloc(p, s) _realloc_dbg(p, s, _NORMAL_BLOCK, __FILE__, __LINE__)
#define _recalloc(p, c, s) _recalloc_dbg(p, c, s, _NORMAL_BLOCK, __FILE__, __LINE__)
#define _expand(p, s) _expand_dbg(p, s, _NORMAL_BLOCK, __FILE__, __LINE__)
#endif
```

## 仮想デストラクター
なぜデストラクターは仮想関数にしなければならないか  
->  
派生クラスにてコンストラクトした変数を基底クラスの変数型として保持しており，基底クラスの型のままデストラクトしたときには，基底クラスのデストラクターが呼ばれない．なので，基底クラスで確保したものはリークしてしまう．  
（通常の一般論として）一般的には，基底クラスに仮想メソッドが定義されているなら，派生クラスでコンストラクトした変数は，基底クラス型で持ちながら，仮想メソッドを使うことを想定しているはずなので，上記条件に当てはまるだろう．  

### デストラクターのvirtual
**virtualなメソッドが存在するなら，デストラクターもvirtualのすべき**  
virtualなメソッドがクラスに定義されているということは，「そのクラスを継承した子クラスを作成し，そのインスタンスを親クラスポインターにキャストする」ことを想定しているはずだ．  
そのため，破棄するときに親クラスのポインターのままだと，子クラス（そのクラスを実際に作成したクラス）のデストラクターが呼ばれなくなる．  
そのために，親クラスからvirtualでデストラクターを継承する必要がある．  

逆視点から言えば，virtualなデストラクターがないクラスでは，子クラスのポインターを親クラス（virtualなデストラクターがないクラス）にキャストしたまま破棄してはならない．  

もし，virtualなデストラクターがないクラス（クラスAとする）から，virtualデストラクターを使いたければ，そのクラスからの派生クラス（子クラス，クラスBとする）にvirtaulデストラクターを定義し，またその派生クラス（孫クラス，クラスCとする）のインスタンスを，クラスBポインターにキャストすれば，クラスBポインターを破棄するときに，クラスCのデストラクターが呼ばれる．  

**そして，上記以外の場合には，デストラクターをvirtual指定する必要性はない．**  

## 四捨五入
C++11では新しく`std::round`が追加されたので，それを使用すること  
https://en.cppreference.com/cpp/numeric/math/round  
`std::float_round_style`も見よ  
https://cppreference.com/cpp/types/numeric_limits/float_round_style  

## ATLを使った文字列変換
```
C{SourceType}2[C]{DestinationType}[EX]
```
|変換元の型/変換先の型|説明|
|---|---|
|A|ANSI 文字列です|
|W|Unicode 文字列です|
|T|汎用文字列です．_UNICODE が定義されている場合は W と等価です．それ以外の場合は A と等価です|
|OLE|OLE 文字列です．W と等価です|

### 注意事項
A<->Wには[C]はない  
OLEには，OLE<->Tしかない  

```cpp
#define _ATL_NO_AUTOMATIC_NAMESPACE // 既定で"using namespace ATL"が定義されるのを防ぐ
#include "atlbase.h"

int _tmain(int argc, _TCHAR* argv[])
{
    // ATL文字列変換マクロを使用するときは，コンパイルエラーが発生する可能性がある
    // これを防ぐため，関数の先頭に"USES_CONVERSION"を追加する
    // 但し，ATL7.0（Visual C++ .NET 2002）以降は必要ない
    USES_CONVERSION;
    ...
    char test[64];
    func(ATL::CA2WEX<64>(test));
    ...
}
```

## 文字列変換
`MultiByteToWideChar` MB,UTF-7,UTF-8 -> UTF-16  
`WideCharToMultiByte` MB,UTF-7,UTF-8 <- UTF-16  

## pointerのconst修飾子
`const int* pVal`と`int const *pVal`は同じように解釈される．型に対して前後どちらでもよい．`const int * pVal`と`int const * pVal`は同じ結果になるが，内部的にはint const *pValと解釈される．  
つまり，const 修飾子はその直前の型が表す変数が定数であることを示すものである．  
- `int const *pVal` `const int *pVal` … ポインターの参照先の書き換え禁止  
- `int* const pVal` … ポインターアドレスの書き換え禁止  
- `int const * const pVal` `const int* const pVal` … ポインターの参照先及びポインターアドレスの書き換え禁止  

## mutable
mutable 修飾子を指定したメンバ変数は、constメンバ関数内でも書き換え可能となる． 

## 呼び出し規約
__stdcall, __cdecl, __fastcall 

## リソースの切り替え
以下をすることにより，カレントのスコープ内では自身が持っているリソースを読むことになる．  
```cpp
AFX_MANAGE_STATE(AfxGetStaticModuleState());
```

## いろいろなパスの取得
```cpp
GetModuleFileName() // 自身のプロセスのパスの取得
GetTempPath() // テンポラリディレクトリを取得
GetTempFileName() // 任意の場所のテンポラリファイル名を取得
```

## ウィンドウの前面化
Windows98/2000の途中からWindowsAPIの仕様が変わったらしく，簡単に任意のウィンドウをアクティブ化しなくなり，代わりにタスクバーが点滅するようになった．   
従来
```cpp
SetForegroundWindow(hWnd); /*or BringWindowToTop(hWnd);*/
```
対策一（あまりうまくいかない） 
```cpp
UINT save_tm, cur_tm = 0;
SystemParametersInfo(SPI_GETFOREGROUNDLOCKTIMEOUT, 0, &save_tm, 0);
SystemParametersInfo(SPI_SETFOREGROUNDLOCKTIMEOUT, 0, &cur_tm, 0);
SetForegroundWindow(hWnd); /*or BringWindowToTop(hWnd);*/
SystemParametersInfo(SPI_SETFOREGROUNDLOCKTIMEOUT, 0, &save_tm, 0); //元に戻す
```
対策二（やや危険らしい） 
```cpp
HWND hActWnd = GetForegroundWindow();
DWORD dwActProcessId = GetWindowThreadProcessId(hActWnd, 0);
DWORD dwMeProcessId = GetWindowThreadProcessId(hWnd, 0);
if (dwActProcessId != dwMeProcessId) {	
    AttachThreadInput(dwMeProcessId, dwActProcessId, 1);
    SetForegroundWindow(hWnd); /*or BringWindowToTop(hWnd);*/
    AttachThreadInput(dwMeProcessId, dwActProcessId, 0);
}
```
対策三（無難） 
```cpp
SetWindowPos(hwnd, HWND_TOPMOST, 0, 0, 0, 0, SWP_NOMOVE|SWP_NOSIZE);
SetWindowPos(hwnd, HWND_NOTOPMOST, 0, 0, 0, 0, SWP_SHOWWINDOW|SWP_NOMOVE|SWP_NOSIZE);
```

## 標準入出力の変更
cout, cerr, clog, cin 

## テンプレート
```cpp
template<class T>
class CHoge
{
    // クラス定義内では"CHoge"は"CHoge<T>"と同等に扱われる．
    CHoge(const CHoge& crHoge);
}
// クラス定義外では，最初の一つを定義すれば，それ以降では"CHoge"は"CHoge<T>"と同等に扱われる．
template <typename T>
void CHoge<T>::CHoge(const CHoge& crHoge) {
}
```

## テンプレート付き可変長引数
最後の引数になるまで第二関数で再帰処理し，最後の引数のみ第一関数で変換し，第二関数を終了する． 
```cpp
template<typename T> int convert(const T& data) {return /*例えばTからintへの変換*/}
template<typename T1, typename... T2> std::vector<int> convert(const T1& data1, const T2&... data2) {std::vector<int> all; all.push_back(convert(data1)); all.push_back(convert(data2...)); return all;}
```

## ブロック定義
```cpp
#pragma region なんやかんや
...
#pragma endregion
```

## バージョンリソース
Visual Studioでのバージョンリソースから情報の取得  

### バージョン情報の取得
```cpp
TCHAR szModuleName[_MAX_PATH];
::GetModuleFileName(nullptr, szModuleName, _MAX_PATH);
::GetFileVersionInfoSize(szModuleName, nullptr);
pVersionInfo = new BYTE[dwSize];
if (pVersionInfo == nullptr) return FALSE;
::GetFileVersionInfo(szModuleName, 0, dwSize, pVersionInfo);
```

### コードページの取得
```cpp
struct {
    WORD Language; // 日本語(0411)，英語(0419)
    WORD CodePage;
} *lpTran;
UINT size;
::VerQueryValue(pVersionInfo, _T("\\VarFileInfo\\Translation"), reinterpret_cast<void**>(&lpTran), &size);
std::basic_ostringstream<TCHAR>	os;
os << std::hex << std::setfill(_T('0')) << std::setw(4) << lpTran[0].Language;
os << std::hex << std::setfill(_T('0')) << std::setw(4) << lpTran[0].CodePage;
std::basic_string<TCHAR> codeStr = os.str();
```

### 各項目の取得
```cpp
auto getSubBlock = [pVersionInfo,codeStr](LPCTSTR name)->LPTSTR{
    LPTSTR pStrInfo = nullptr;
    UINT size;
    ::VerQueryValue(pVersionInfo, std::basic_string<TCHAR>(_T("\\StringFileInfo\\")+codeStr+_T("\\")+name).c_str(), reinterpret_cast<void**>(&pStrInfo), &size);
    return pStrInfo;
};
LPTSTR pDescr = getSubBlock(_T("FileDescription")); _ASSERT(pDescr);	// ファイルの説明(FileDescription)
LPTSTR pVersi = getSubBlock(_T("FileVersion"));     _ASSERT(pVersi);	// ファイルバージョン(FileVersion)
LPTSTR pProdN = getSubBlock(_T("ProductName"));     _ASSERT(pProdN);	// 製品名(ProductName)
LPTSTR pProdV = getSubBlock(_T("ProductVersion"));  _ASSERT(pProdV);	// 製品バージョン(ProductVersion)
LPTSTR pCpyRt = getSubBlock(_T("LegalCopyright"));  _ASSERT(pCpyRt);	// 著作権(LegalCopyright)
LPTSTR pCompa = getSubBlock(_T("CompanyName"));     _ASSERT(pCompa);	// 会社名(CompanyName)
```

## 算術演算子
```cpp
T& operator=(const T& v) noexcept {if(this==&v) {/*_ASSERT(false);*/ return (*this);} CopyMemory(/*thisへコピー*/); return (*this);}
T operator-(void) const {return T(/*負値化*/);}
// 単純演算はインスタンスを作るが，代入演算子はインスタンスを作らないので，
// 単純演算子に委譲するよりも，代入演算子に委譲する方が良い
T operator+(const T& v) const {return T(*this) += v;}
T operator-(const T& v) const {return T(*this) -= v;}
T operator*(double value) const {return T(*this) *= v;}
T operator/(double value) const {return T(*this) /= v;}
T& operator+=(const T& v) {return /*代入足し算*/;}
T& operator-=(const T& v) {return /*代入引き算*//*(this->operator+=(-v));*/;}
T& operator*=(double value) {return /*代入掛け算*/;}
T& operator/=(double value) {return /*代入割り算*//*if(value==0.0) {_ASSERT(false); return (std::INFINITY);} return this->operator*=(1.0/value);*/;}
bool operator==(const T& v) const noexcept {return /*比較*/;}
bool operator!=(const T& v) const noexcept {return !T::operator==(v);}
bool operator<(const T& v) const noexcept {/*もし==なら，return false;*/}
```

## デストラクター
- 基本クラスのポインターを介したポリモフィズムを利用した削除を許可するか（デストラクターはvirtualかつpublicでなければならない）  
- 基本クラスのポインターを介したポリモフィズムを利用した削除を許可しないか（デストラクターは非virtualかつprotectedでなければならない）  
```cpp
class Base {.......};
class Derived : public Base {.......};
void hoge()
{
    Derived* pD = new Derived;
    Base* pB = pD;
    delete pB; // これを許可するかしないか
}
```

## 仮想関数のアクセス指定子
仮想関数へのアクセス指定子は，   
- C++では，オーバーライドする際に制約はない  
- Javaでは，オーバーライドされる側のアクセス修飾子よりも範囲の広い物であればよい  
- C#では，オーバーライド時のアクセス修飾子の変更は認められない  

## マクロループ
```cpp
#define CONCATENATE_MAIN(x, y) x##y
#define CONCATENATE(x, y) CONCATENATE_MAIN(x, y)

// MSVCでの__VA_ARGS__の展開には，これがいる（GCCでは不要？）
// __VA_ARGS__を含むマクロ全体を，EXPANDで囲む
#define EXPAND(x) x

// 引数の数によりずれることによって，引数の個数を取り出せる
#define DEF_INDEX() 10, 9, 8, 7, 6, 5, 4, 3, 2, 1
#define __VA_ARGS__SIZE_MAIN(ARG0, ARG1, ARG2, ARG3, ARG4, ARG5, ARG6, ARG7, ARG8, ARG9, N, ...) N
#define __VA_ARGS__SIZE_SUB(...) EXPAND(__VA_ARGS__SIZE_MAIN(__VA_ARGS__))
#define __VA_ARGS__SIZE(...) EXPAND(__VA_ARGS__SIZE_SUB(__VA_ARGS__, DEF_INDEX()))

// MY_CASTに，逐一自分のやりたいことを定義する
#define MY_CAST(TYPE, ARG1) static_cast<TYPE>(ARG1)
#define MACRO_1(TYPE, ARG1) MY_CAST(TYPE,ARG1)
#define MACRO_2(TYPE, ARG1, ...) MY_CAST(TYPE, ARG1), EXPAND(MACRO_1(TYPE, __VA_ARGS__))
#define MACRO_3(TYPE, ARG1, ...) MY_CAST(TYPE, ARG1), EXPAND(MACRO_2(TYPE, __VA_ARGS__))
// 以下，この例だと10まで定義可能
// 定義を増減させたいときには，DEF_INDEXと__VA_ARGS__SIZE_MAINの数を調整する必要がある

#define MY_MACRO(TYPE, ...) EXPAND(CONCATENATE(MACRO_, EXPAND(__VA_ARGS__SIZE(__VA_ARGS__)))(TYPE, __VA_ARGS__))

int main()
{
    std::initializer_list<double> data1{ MY_MACRO(double,1) };
    std::initializer_list<double> data2{ MY_MACRO(double,1,2) };
    std::initializer_list<double> data3{ MY_MACRO(double,1,2,3) };
}
```
https://www.google.co.jp/search?q=C%2B%2B+define+%E3%83%9E%E3%82%AF%E3%83%AD+%E7%B9%B0%E3%82%8A%E8%BF%94%E3%81%97&source=hp&ei=xtOQYprfMaDs2roPqaK1uAM&iflsig=AJiK0e8AAAAAYpDh1gbEZX8k287M578wbwrIpTzTZS6r&ved=0ahUKEwia2tGb5v_3AhUgtlYBHSlRDTcQ4dUDCAg&uact=5&oq=C%2B%2B+define+%E3%83%9E%E3%82%AF%E3%83%AD+%E7%B9%B0%E3%82%8A%E8%BF%94%E3%81%97&gs_lcp=Cgdnd3Mtd2l6EAM6BAgAEEM6DQgAEIAEELEDEIMBEAQ6CwgAEIAEELEDEIMBOgUIABCABDoKCAAQsQMQgwEQQzoHCAAQgAQQBDoGCAAQHhAEOggIABAeEAgQBDoECAAQHjoFCCEQoAE6BQgAEKIEOgQIIRAVUABY4VdglVtoAHAAeACAAZsBiAGZFpIBBDIwLjmYAQCgAQE&sclient=gws-wiz  
https://in-neuro.hatenablog.com/entry/2020/10/21/155651 

## 文字列分割
```cpp
std::string str("いろはにほへと ちりぬるを わかよたれそ つねならむ うゐのおくやま けふこえて あさきゆめみし ゑひもせす");
std::vector<std::string> splitStrs;
char* orgLocale = std::setlocale(LC_CTYPE, nullptr); // 現在のロケールは先に尋ねないといけない
std::string orgloc;
if (orgLocale != nullptr)
	orgloc.assign(orgLocale);
std::setlocale(LC_CTYPE, "");
_ASSERT(MB_CUR_MAX < splitNum);
std::vector<bool> splitables(str.size(), true);
size_t pos = 0;
while (pos < str.size()) {
    int multiNum = std::mblen(str.data() + pos, MB_CUR_MAX);
    if (1 < multiNum) {
        for (size_t cnt = 1; cnt < multiNum; cnt++)
            splitables.at(pos + cnt) = false;
        pos += multiNum;
    } else {
        pos += 1;
    }
}
std::setlocale(LC_CTYPE, orgloc.data());

static const int splitNum = 10;
size_t start = 0;
while (start < str.size()) {
    if (str.size() <= start + splitNum) {
        splitStrs.push_back(str.substr(start));
        break;
    }
    int offset = 0;
    if (splitables.at(start + splitNum) == false) {
        for (int cnt = -1; -splitNum < cnt; cnt--) {
            if (splitables.at(start + splitNum + cnt)) {
                offset = cnt;
                break;
            }
        }
        if (offset == 0) {
            assert(false);
        }
    }
    splitStrs.push_back(str.substr(start, splitNum + offset));
    start += splitNum + offset;
}
```

## std::cout
エスケープシーケンスをつけると細かく編集できる．エスケープシーケンスは“\033”または“\x1b”で指定する．  
'['が指定の始まり，'m'が指定の終了．その間に数値を';'で区切って編集を指定する． 
||| 
|---|---|
|0|リセット|
|1|太字|
|2|RGB指定|
|3|イタリック|
|4|下線|
|5|点滅|
|6||
|7||
|8||
|9||
|30|文字色 black|
|31|文字色 red|
|32|文字色 green|
|33|文字色 yellow|
|34|文字色 blue|
|35|文字色 magenta|
|36|文字色 cyan|
|37|文字色 white|
|40|背景色 black|
|41|背景色 red|
|42|背景色 green|
|43|背景色 yellow|
|44|背景色 blue|
|45|背景色 magenta|
|46|背景色 cyan|
|47|背景色 white|

例えば，`std::cout << "\033[31m hoge \033[0m" << std::endl;`とすれば，"hoge"が赤色で表示される．（最後に"\033[0m"をつけてリセットしておくのを勧める） 