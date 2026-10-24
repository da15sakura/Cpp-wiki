# Unicode<!-- omit in toc -->
UTF … Unicode Transformation Format  
UTF … UCS(Universal Coded Character Set) Transformation Format  
0x000000～0x10FFFF(21bit)が定義域である．  
CL(Microsoft Visual C++)では，wchar_tはUTF-16LEで表現される．  

- [UCS-2](#ucs-2)
- [UTF-8](#utf-8)
- [UTF-16](#utf-16)
- [UTF-32](#utf-32)
- [エンディアン](#エンディアン)
- [BOM](#bom)
  - [UTF-8](#utf-8-1)
  - [UTF-16](#utf-16-1)
  - [UTF-32](#utf-32-1)
- [BMP](#bmp)


## UCS-2
U+xxxxと表記可能なBMP(Basic Miltilingual Plane：基本多言語面)領域を表す．2011年の改訂で廃止予定とされた．   

## UTF-8
Unicodeスカラ値を1～4符号単位の可変長で表す．1符号単位は8bit(1byte)である．  
表現する際に，最低のbit数が8bitであることから，UTF-'8'と表する．  
各byteでの表現を二進数で表すと以下の通り．（当項では二進数表現を で表している）  
ここで，x...x（xのみを抽出して並べたもの）がUnicodeスカラ値である．  
```
0xxxxxxx                            0x000000～0x00007F  7bit
110xxxxx 10xxxxxx                   0x000080～0x0007FF 11bit
1110xxxx 10xxxxxx 10xxxxxx          0x000800～0x00FFFF 16bit
11110xxx 10xxxxxx 10xxxxxx 10xxxxxx 0x010000～0x10FFFF 21bit
```
ASCIIコードは1byteで表現でき，そのコードは同じである．  
2～4byteでは，先頭から0が現れるまでの1の数が，その文字を表すバイト数を表す．  
2～4byteでは，2byte目以降は先頭は10で表す．このため，任意のバイト部が1byte表現文字と同一bit列になることはない．つまり，各byteの先頭が0なら1byte表現部，10なら2～4byte表現の2～4byte部，110なら2byte表現の先頭部，1110なら3byte表現の先頭部，11110なら4byte表現の先頭部ということが容易にわかる．  
2byteでは，xの先頭4bitはすべて0であってはいけない．3～4byteでは，xの先頭5bitはすべて0であってはいけない．  
なお，4byteでは，xの先頭5bitの最大値は10000（このため，最大値は0x10FFFF）．（2byteでは1111，3byteでは11111）  

## UTF-16
Unicodeスカラ値を1または2符号単位の可変長で表す．1符号単位は16bit(2byte)である．  
表現する際に，最低のbit数が16bitであることから，UTF-'16'と表する．  
```
//0x0000～0xFFFFのとき
xxxxxxxx xxxxxxxx (16bit)

//0x010000～0x10FFFFのとき
ハイサロゲート = ("Unicodeスカラ値"-0x10000)/0x400+0xD800 0xD800～0xDBFF
ローサロゲート = ("Unicodeスカラ値"-0x10000)%0x400+0xDC00 0xDC00～0xDFFF
[ハイサロゲート][ローサロゲート] (32bit)
```
両サロゲートは，開始位置分をオフセットして，"/0x400""%0x400"で前部10bit,後部10bitに分けて，Unicodeスカラー値として定義されていない0xD800～0xDFFFを使ってユニーク表現に落とし込んでいる．前後10bitの組み合わせなので，1024*1024=1048576通りの表現が可能になる．  

## UTF-32
Unicodeスカラ値を1符号単位の固定長で表す．1符号単位は32bit(4byte)である．  
表現する際に，最低のbit数が32bitであることから，UTF-'32'と表する．  
UTF-32はUCS-4の部分集合である．  
内容はUnicodeスカラ値そのものなので，UTF-8,UTF-16と比べて，サイズが少なくとも同じか大きくなる．  

## エンディアン
Little Endianness と Big Endianness がある．  
UTF-16またはUTF-32に付与される．各々，UTF-16LE, UTF-16BE, UTF-32LE, UTF-32BEと表す．エンディアンが指定されていない場合にはビッグエンディアンとして扱う．  
1234ABCDの4byteを，byte単位で1234ABCDと上位から表すのをビッグエンディアン，byte単位で3412CDABと下位から表すのをリトルエンディアンと呼ぶ．  

## BOM
バイト順マーク(Byte Order Mark)

### UTF-8
エンディアンの必要のないUTF-8ではBOMは必要ないが，データストリームの先頭にEFBBBFをつけることもある．（EFBBBFはUnicodeスカラー値0xFFFEをUTF-8で表現したものである）   

### UTF-16
データストリームの先頭に2byte付与される．  
先頭2byteがFFFEならリトルエンディアン，FEFFならビッグエンディアンとして扱う．  
BOMがなければ，ビッグエンディアンとして扱う．   

### UTF-32
データストリームの先頭に4byte付与される．  
先頭4byteがFFFE0000ならリトルエンディアン，0000FEFFならビッグエンディアンとして扱う．  
BOMがなければ，ビッグエンディアンとして扱う．   

## BMP
"Basic Multilingual Plane" "基本多言語面" "基本的な文字" 