# STL<!-- omit in toc -->
- [デバッグが遅いとき](#デバッグが遅いとき)
  - [debug](#debug)
  - [relsease](#relsease)
- [関数アダプターと関数バインダー](#関数アダプターと関数バインダー)
- [関数バインダーの否定回路](#関数バインダーの否定回路)
- [std::bind で参照を](#stdbind-で参照を)
- [配列の初期化](#配列の初期化)
- [例外処理](#例外処理)
- [I/O マニピュレーター](#io-マニピュレーター)
- [iterator](#iterator)
- [min/maxマクロ](#minmaxマクロ)
- [lower\_bound/upper\_bound](#lower_boundupper_bound)
- [numeric\_limits](#numeric_limits)

## デバッグが遅いとき
どうもSTLは，releaseビルド時にはインライン展開されているらしいが，debugビルド時にはインライン展開されていないらしい．しかも，__forceinlineをつけても展開されていなかったりする．  
それでマクロを使用して，イテレーターのチェックなんかを無効にすることによって，debug時のSTLの挙動を早くすることができる．
|マクロの値|意味|
|---|---|
|_SECURE_SCL=0|イテレーターをチェックしない|
|_SECURE_SCL=1|イテレーターをチェックする|
|_HAS_ITERATOR_DEBUGGING=0|debugビルド時にイテレーターdebugを無効にする|
|_HAS_ITERATOR_DEBUGGING=1|debugビルド時にイテレーターdebugを有効にする|

### debug
|新しいマクロ|古いマクロ|
|---|---|
|_ITERATOR_DEBUG_LEVEL=0|_SECURE_SCL=0, _HAS_ITERATOR_DEBUGGING=0|
|_ITERATOR_DEBUG_LEVEL=1|_SECURE_SCL=1, _HAS_ITERATOR_DEBUGGING=0|
|_ITERATOR_DEBUG_LEVEL=2（規定）|_SECURE_SCL=??, _HAS_ITERATOR_DEBUGGING=1|

### relsease
|新しいマクロ|古いマクロ|
|---|---|
|_ITERATOR_DEBUG_LEVEL=0（規定）|_SECURE_SCL=0|
|_ITERATOR_DEBUG_LEVEL=1|_SECURE_SCL=1|

通常，C/C++ - プリプロセッサ - プリプロセッサの定義に，_ITERATOR_DEBUG_LEVEL=1なんかを定義するのだが，_ITERATOR_DEBUG_LEVELを規定以外にする際には，リンクしているライブラリーのすべてで，_ITERATOR_DEBUG_LEVELを同じにしなければならない．さもないと，リンクエラーが発生する．  
その場合，STLのヘッダーをincludeする直前に#define _HAS_ITERATOR_DEBUGGING (0)なんかを定義してもいい．  

## 関数アダプターと関数バインダー
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>

// 整数クラス
class Integer {
public:
    explicit Integer(int n) :value(n) {};
    virtual ~Integer() {}
    void print() const {std::cout << value << std::endl;}   // 自分の値を表示する
    void print_plus(int plus) const {std::cout << value+plus << std::endl;} // 自分の値に足して表示する
    void print_plus_Integer(const Integer plus) const {std::cout << value+plus.value << std::endl;} // 自分の値に足して表示する
    void print_plus_Integer_r(const Integer& plus) const {std::cout << value+plus.value << std::endl;}  // 自分の値に足して表示する
    void print_plus_Integer_p(const Integer* plus) const {std::cout << value+plus->value << std::endl;} // 自分の値に足して表示する
private:
    int value;
};
// 整数表示クラス
class IntegerPrinter {
public:
    void operator()(const Integer& i) const {i.print();}
};
// 整数表示関数群
void print_r(const Integer& int1) {int1.print();}
void print_p(const Integer* int1) {int1->print();}
void print_plus_1st(int int1, const Integer int2) {int2.print_plus(int1);}  // C++03以前のptr_funでは参照型が渡せない
void print_plus_2nd(const Integer int1, int int2) {int1.print_plus(int2);}  // C++03以前のptr_funでは参照型が渡せない
void print_plus_1st_r(int int1, const Integer& int2) {int2.print_plus(int1);}
void print_plus_2nd_r(const Integer& int1, int int2) {int1.print_plus(int2);}
void print_plus_1st_p(int int1, const Integer* int2) {int2->print_plus(int1);}
void print_plus_2nd_p(const Integer* int1, int int2) {int1->print_plus(int2);}
void deleteObject(void* pObject) {if (pObject!=NULL) delete pObject;}

int _tmain(int argc, _TCHAR* argv[])
{
    std::vector<Integer> v;
    v.push_back(Integer(3));
    v.push_back(Integer(4));
    v.push_back(Integer(5));
    // C++03以前（非バインド）
    std::for_each(v.begin(), v.end(), IntegerPrinter());
    std::for_each(v.begin(), v.end(), print_r);
    std::for_each(v.begin(), v.end(), std::mem_fun_ref(&Integer::print));
    // C++03以前（バインド）
    std::for_each(v.begin(), v.end(), std::bind1st(std::ptr_fun(print_plus_1st),10));   // C++03以前のptr_funでは参照型が渡せない
    std::for_each(v.begin(), v.end(), std::bind2nd(std::ptr_fun(print_plus_2nd),10));   // C++03以前のptr_funでは参照型が渡せない
    std::for_each(v.begin(), v.end(), std::bind1st(std::mem_fun_ref(&Integer::print_plus_Integer),Integer(10)));    // C++03以前のmem_fun_refでは参照型が渡せない
    std::for_each(v.begin(), v.end(), std::bind2nd(std::mem_fun_ref(&Integer::print_plus),10));
    // C++TR1(2007)以降（非バインド）
    std::for_each(v.begin(), v.end(), IntegerPrinter());
    std::for_each(v.begin(), v.end(), print_r);
    std::for_each(v.begin(), v.end(), std::mem_fn(&Integer::print));
    // C++TR1(2007)以降（バインド）
    std::for_each(v.begin(), v.end(), std::bind(print_plus_1st_r,10,std::placeholders::_1));
    std::for_each(v.begin(), v.end(), std::bind(print_plus_2nd_r,std::placeholders::_1,10));
    std::for_each(v.begin(), v.end(), std::bind(std::mem_fn(&Integer::print_plus_Integer_r),Integer(10),std::placeholders::_1));
    std::for_each(v.begin(), v.end(), std::bind(std::mem_fn(&Integer::print_plus),std::placeholders::_1,10));

    std::vector<Integer*> v_p;
    v_p.push_back(new Integer(3));
    v_p.push_back(new Integer(4));
    v_p.push_back(new Integer(5));
    // C++03以前（非バインド）
//  std::for_each(v_p.begin(), v_p.end(), IntegerPrinter());
    std::for_each(v_p.begin(), v_p.end(), print_p);
    std::for_each(v_p.begin(), v_p.end(), std::mem_fun(&Integer::print));
    // C++03以前（バインド）
    std::for_each(v_p.begin(), v_p.end(), std::bind1st(std::ptr_fun(print_plus_1st_p),10));
    std::for_each(v_p.begin(), v_p.end(), std::bind2nd(std::ptr_fun(print_plus_2nd_p),10));
    std::for_each(v_p.begin(), v_p.end(), std::bind1st(std::mem_fun_ref(&Integer::print_plus_Integer_p),Integer(10)));
    std::for_each(v_p.begin(), v_p.end(), std::bind2nd(std::mem_fun(&Integer::print_plus),10));
    // C++TR1(2007)以降（非バインド）
//  std::for_each(v_p.begin(), v_p.end(), IntegerPrinter());
    std::for_each(v_p.begin(), v_p.end(), print_p);
    std::for_each(v_p.begin(), v_p.end(), std::mem_fn(&Integer::print));
    // C++TR1(2007)以降（バインド）
    std::for_each(v_p.begin(), v_p.end(), std::bind(print_plus_1st_p,10,std::placeholders::_1));
    std::for_each(v_p.begin(), v_p.end(), std::bind(print_plus_2nd_p,std::placeholders::_1,10));
    std::for_each(v_p.begin(), v_p.end(), std::bind(std::mem_fn(&Integer::print_plus_Integer_p),Integer(10),std::placeholders::_1));
    std::for_each(v_p.begin(), v_p.end(), std::bind(std::mem_fn(&Integer::print_plus),std::placeholders::_1,10));

    // v_pの後始末をする
    std::for_each(v_p.begin(), v_p.end(), deleteObject);
    return 0;
}
```
上記の関数バインダーの関数オブジェクトを変数定義するならば，以下のように書き表せられる． 
```cpp
    // C++03以前（バインド）
    std::binder1st<std::pointer_to_binary_function<int,Integer,void,void(__cdecl*)(int,Integer)>>   func_print_plus_1st = std::bind1st(std::ptr_fun(print_plus_1st),10);    // C++03以前のptr_funでは参照型が渡せない
    std::for_each(v.begin(), v.end(), func_print_plus_1st);
    std::binder2nd<std::pointer_to_binary_function<Integer,int,void,void(__cdecl*)(Integer,int)>>   func_print_plus_2nd = std::bind2nd(std::ptr_fun(print_plus_2nd),10);    // C++03以前のptr_funでは参照型が渡せない
    std::for_each(v.begin(), v.end(), func_print_plus_2nd);
    std::binder1st<std::const_mem_fun1_ref_t<void,Integer,Integer>> func_print_plus_Integer_m = std::bind1st(std::mem_fun_ref(&Integer::print_plus_Integer),Integer(10));   // C++03以前のmem_fun_refでは参照型が渡せない
    std::for_each(v.begin(), v.end(), func_print_plus_Integer_m);
    std::binder2nd<std::const_mem_fun1_ref_t<void,Integer,int>> func_print_plus_m = std::bind2nd(std::mem_fun_ref(&Integer::print_plus),10);
    std::for_each(v.begin(), v.end(), func_print_plus_m);
    // C++TR1(2007)以降（バインド）
    std::function<void(const Integer&)> func_print_plus_1st_r = std::bind(print_plus_1st_r,10,std::placeholders::_1);
    std::for_each(v.begin(), v.end(), func_print_plus_1st_r);
    std::function<void(const Integer&)> func_print_plus_2nd_r = std::bind(print_plus_2nd_r,std::placeholders::_1,10);
    std::for_each(v.begin(), v.end(), func_print_plus_2nd_r);
    std::function<void(const Integer&)> func_print_plus_1st_r_m = std::bind(std::mem_fn(&Integer::print_plus_Integer_r),Integer(10),std::placeholders::_1);
    std::for_each(v.begin(), v.end(), func_print_plus_1st_r_m);
    std::function<void(const Integer&)> func_print_plus_2nd_r_m = std::bind(std::mem_fn(&Integer::print_plus),std::placeholders::_1,10);
    std::for_each(v.begin(), v.end(), func_print_plus_2nd_r_m);

    // C++03以前（バインド）
    std::binder1st<std::pointer_to_binary_function<int,const Integer*,void,void(__cdecl*)(int,const Integer*)>> func_print_plus_1st_p_ = std::bind1st(std::ptr_fun(print_plus_1st_p),10);
    std::for_each(v_p.begin(), v_p.end(), func_print_plus_1st_p_);
    std::binder2nd<std::pointer_to_binary_function<const Integer*,int,void,void(__cdecl*)(const Integer*,int)>> func_print_plus_2nd_p_ = std::bind2nd(std::ptr_fun(print_plus_2nd_p),10);
    std::for_each(v_p.begin(), v_p.end(), func_print_plus_2nd_p_);
    std::binder1st<std::const_mem_fun1_ref_t<void,Integer,const Integer*>>  func_print_plus_Integer_p_m = std::bind1st(std::mem_fun_ref(&Integer::print_plus_Integer_p),Integer(10));
    std::for_each(v_p.begin(), v_p.end(), func_print_plus_Integer_p_m);
    std::binder2nd<std::const_mem_fun1_t<void,Integer,int>> func_print_plus_p_m = std::bind2nd(std::mem_fun(&Integer::print_plus),10);
    std::for_each(v_p.begin(), v_p.end(), func_print_plus_p_m);
    // C++TR1(2007)以降（バインド）
    std::function<void(const Integer*)> func_print_plus_1st_p = std::bind(print_plus_1st_p,10,std::placeholders::_1);
    std::for_each(v_p.begin(), v_p.end(), func_print_plus_1st_p);
    std::function<void(const Integer*)> func_print_plus_2nd_p = std::bind(print_plus_2nd_p,std::placeholders::_1,10);
    std::for_each(v_p.begin(), v_p.end(), func_print_plus_2nd_p);
    std::function<void(const Integer*)> func_print_plus_1st_p_m = std::bind(std::mem_fn(&Integer::print_plus_Integer_p),Integer(10),std::placeholders::_1);
    std::for_each(v_p.begin(), v_p.end(), func_print_plus_1st_p_m);
    std::function<void(const Integer*)> func_print_plus_2nd_p_m = std::bind(std::mem_fn(&Integer::print_plus),std::placeholders::_1,10);
    std::for_each(v_p.begin(), v_p.end(), func_print_plus_2nd_p_m);
```

## 関数バインダーの否定回路
- `std::not1` 単項演算子用（関数バインダーは単項演算子とみなされる）
- `std::not2` 二項演算子用
```cpp
auto isEven = [](int value)->bool{return value%2==0;};
std::array<int,3> values{0,1,2};
std::count_if(std::begin(values), std::end(values), isEven); // 2が返る
std::count_if(std::begin(values), std::end(values), std::not1(isEven)); // 1が返る
```
```cpp
std::array<int,3> values{1,2,0};
std::sort(std::begin(values), std::end(values), std::less<int>());
std::sort(std::begin(values), std::end(values), std::not2(std::greater_equal<int>()));
// 上記二つとも同等の処理となる
```

## std::bind で参照を
参照を引数に取る関数に対し`std::bind`すると，単なる値渡しに解釈されてしまう．  
それを回避するために`std::ref`を使う．  
```cpp
void hoge(int& rHoge) {rHoge = 1;}
{
    int h = 0;
    std::bind(hoge, h); // hは0のまま
    std::bind(hoge, std::ref(h)); // hは1になっている
}
```

## 配列の初期化
```cpp
int data[] = {0, 1, 2};
std::vector<int> data(data, data+3);
```
VS2010以降(?)ではstd::endが使用できる．これにより個数の変化に柔軟に対応できる．（std::beginは使用できない？）
```cpp
int data[] = {0, 1, 2};
std::vector<int> data(data, std::end(data));
```
C++11以降(VSでは2013以降)では初期化子リスト（initializer_list）が使用できる． 
```cpp
std::vector<int> data{0, 1, 2};
```

## 例外処理
```cpp
#include <crtdbg.h>
#include <exception>
#include <iostream>

class myException : public std::exception
{
};
class myException2 : public std::logic_error
{
public:
    myException2(const std::string& cause) :std::logic_error(cause) {}
};

int _tmain(int argc, _TCHAR* argv[])
{
    try {
        int i = 2;
        throw myException();
        int j = 3;
    } catch (const myException& e) {
        std::cout << e.what() << std::endl;
        return 0;
    }

    try {
        int i = 2;
        throw myException2("エラー");
        int j = 3;
    } catch (const myException& e) {
        std::cout << e.what() << std::endl;
        return 0;
    }

    return 0;
}
```

## I/O マニピュレーター
`std::ios_base::fmtflags`より，フラグを一部抜粋  
`std::ios_base::setf/unsetf`及び`<<`で設定可能．つまりインスタンスのライフサイクルで有効．
元に戻す方法
- 事前に`std::ios_base::flags`で取得して，使用後`std::ios_base::flags/setf/unsetf`して元に戻す．
- `setiosflags(fmtflgs)`or`resetiosflags(fmtflags)`で，フラグの付け外しをする．これらは，`operator <<`や`operator >>`に直接流し込める．

|||備考|
|---|---|---|
|基数表記|oct, dec(*), hex|basefield(=oct\|dec\|hex)|
|寄せ表記|left, right(*), internal|adjustfield(=left\|right\|internal)
|浮動小数点表記|fixed, scientific, hexfloat, defaultfloat(*)|floatfield(=fixed\|scientific\|hexfloat\|defaultfloat)|
|二値表示|noboolalpha(*), boolalpha||
|基数表示|noshowbase(*), showbase||
|小数点表示|noshowpoint(*), showpoint||
|正符号表示|noshowpos(*), showpos||
|大文字表示|nouppercase(*), uppercase||

||設定|取得|備考|
|---|---|---|---|
|基数表記|std::setbase()|std::ios_base::flags()|oct, dec(*), hexの代替|
|入出力文字数|std::setw()|std::ios_base::width()|次の入出力にのみ影響|
|埋め込み文字|std::setfill()|std::basic_ios::fill() |
|精度|std::setprecision()|std::ios_base::precision()|

```cpp
#include <sstream>
#include <iostream>
#include <iomanip>

int _tmain(int argc, _TCHAR* argv[])
{
    std::basic_ostringstream<TCHAR> os;
    std::ios_base::fmtflags fmtFlg = os.flags();    // 現在のフラグをとっておく
    auto    resetstringstream = [&os,fmtFlg](){
        os.str(_T("")); // initialize
        os.flags(fmtFlg);   // flagsは完全に置き換える
    };

    // 基数
    int integer = 16;
    os << std::oct << integer << _T(",") << // 20
          std::dec << integer << _T(",") << // 16
          std::hex << integer << _T(",") << std::endl;  // 10
    os << std::showbase << std::showpos;
    os << std::oct << integer << _T(",") << // 020
          std::dec << integer << _T(",") << // +16
          std::hex << integer << _T(",") << std::endl;  // 0x10
    std::wcout << os.str();
    resetstringstream();    // フラグを元に戻す

    // 寄せと基数と正符号
    os << std::dec << std::showpos;
    os << std::left     << std::setw(10) << integer << std::endl;   // +16_______
    os << std::internal << std::setw(10) << integer << std::endl;   // +_______16
    os << std::right    << std::setw(10) << integer << std::endl;   // _______+16
    os << std::hex << std::showbase;
    os << std::left     << std::setw(10) << integer << std::endl;   // 0x10______
    os << std::internal << std::setw(10) << integer << std::endl;   // 0x______10
    os << std::right    << std::setw(10) << integer << std::endl;   // ______0x10
    std::wcout << os.str();
    resetstringstream();    // フラグを元に戻す

    // 入出力文字数と埋め込みと小数点
    double  real = 1.0;
    os                  << real << std::endl;   // 1
    os << std::showpoint;
    os                  << real << std::endl;   // 1.00000
    os << std::setw(10) << real << std::endl;   // ___1.00000
    os << std::setfill(_T('*'));
    os << std::setw(10) << real << std::endl;   // ***1.00000
    std::wcout << os.str();
    resetstringstream();    // フラグを元に戻す

    // 精度
    double  pi = std::acos(-1.0);
    os << pi << std::endl;  // 3.14159
    os << std::setprecision(10) << pi << std::endl; // 3.141592654
    os << std::setprecision(std::numeric_limits<double>::digits10) << pi << std::endl;  // 3.14159265358979
    std::wcout << os.str();
    resetstringstream();    // フラグを元に戻す

    // 浮動小数点
    os << std::fixed        << 0.01 << std::endl;   // 0.010000
    os << std::scientific   << 0.01 << std::endl;   // 1.000000e-002
    os << std::hexfloat     << 0.01 << std::endl;   // 0x1.47ae147ae147b00p-7
    os << std::defaultfloat << 0.01 << std::endl;   // 0.01
    std::wcout << os.str();
    resetstringstream();    // フラグを元に戻す

    // 精度とfixedの組み合わせで，小数点以下の桁数指定になる
    // しかも(指定桁+1)番の数値を四捨五入する
    os << std::fixed << std::setprecision(3) << pi << std::endl;  // 3.142
    std::wcout << os.str();
    resetstringstream();    // フラグを元に戻す

    // 二値
    bool    boolean = false;
    os << boolean << std::endl; // 0
    os << std::boolalpha;
    os << boolean << std::endl; // false
    std::wcout << os.str();
    resetstringstream();    // フラグを元に戻す

    return 0;
}
```

## iterator
- コンテナアダプター(`stack` `queue` `priority_queue`)は，`iterator`を持たないためアルゴリズムを使用できない．
- 各種アルゴリズムは，コンテナにランダムアクセス/双方向反復子があるかどうかによって，アルゴリズムが利用できるかできないかが決定される． 

`std::make_reverse_iterator ` iteratorをreverse_iteratorに変換する  
`std::reverse_iterator<Iter>::base ` reverse_iteratorをiteratorに変換する

- output iterator（出力）
- input iterator（入力）
- forward iterator（前方）  
  forward iterator = output iterator | input iterator | ()
- bidirectionalI iterator（双方向）  
  bidirectionalI iterator = forward iterator | (一つ手前の要素へ戻る機能) 
- random access iterator（ランダムアクセス）  
  random access iterator = bidirectionalI iterator | (ランダムアクセス機能) 
- contiguous iterator（隣接）  
  contiguous iterator = random access iterator | (各要素がメモリ上で隣接していることを保証) 

## min/maxマクロ
min/maxマクロは，次のようにwindef.h内で定義されているが，次のように使用すると意図しない（直感的でない）動作をしてしまう． 
```cpp
#define max(a,b) (((a) > (b)) ? (a) : (b))
#define min(a,b) (((a) < (b)) ? (a) : (b))

int x = 10, y = 20;
int z = min(++x, y);  // zには11が入って欲しいが，意図に反して12が入ってしまう
```
よって，std::min,std::maxを使用するようにする．   
しかし，そのまま使用するとコンパイルエラーが発生するので，以下の方法で回避する．  
1. min/maxが定義されている"windef.h"（大抵は"stdafx.h"に定義されている"windows.h"に含まれる）がincludeされる前に`#define NOMINMAX`を書いておく．  
2. "windef.h"がincludeされた後に，`#undef min` `#undef max`にて，min/maxマクロのundefを行う．  
3. min/maxを含む式を括弧でくくり，マクロとして扱われないようにする．  
   `(std::max)(1,2)`

## lower_bound/upper_bound
`lower_bound` `upper_bound`ともに，ソートされていることが必要である．しかし，同値を持っていても良い．
- `lower_bound`は，指定値以上で最小の値の位置を返す．（初めて指定値以上となるイテレーターを返す）
- `upper_bound`は，指定値より大きい最初の値の位置を返す．（初めて指定値を超えるイテレーターを返す）

この二つの違いは，同値が含まれているときに大きく異る．  
比較関数を指定しない場合は，他アルゴリズム同様std::lessをしているのと同等である．  
結果として，次が言える．
- `lower_bound(less)` $\equiv$ `upper_bound(less_equal)`
- `lower_bound(less_equal)` $\equiv$ `upper_bound(less)`

## numeric_limits
https://en.cppreference.com/cpp/types/numeric_limits