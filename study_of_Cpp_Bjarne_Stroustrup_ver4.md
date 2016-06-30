# ビャーネ・ストラウストラップ プログラミング言語C++ [第4版]

## abstract
 C++のお勉強のために「ビャーネ・ストラウストラップ プログラミング言語C\+\+[第4版]」を読んで, ためになった部分をここにまとめていく.

## Index
* [第1章 本書の読み進め方](#第1章-本書の読み進め方)
* [第2章 C\+\+を探検しよう:基本](#第2章-c++を探検しよう:基本)
* [第3章 C\+\+を探検しよう:抽象化のメカニズム](#第3章-c++を探検しよう:抽象化のメカニズム)
* [第4章 C\+\+を探検しよう:コンテナとアルゴリズム](#第4章-c++を探検しよう:コンテナとアルゴリズム)

## 第1章 本書の読み進め方
- **1.3.2 C++11の新機能について**
 1.　コンストラクタによって不変条件を確立する(メンバの範囲等を明らかにする). (§2.4.3.2)
 2.　コンストラクタとデストラクタの組み合わせによって, 資源管理を単純化する. (§5.2)
 3.　裸のnewとdelete演算子を避ける. (§3.2.1.2, §11.2.1)
 4.　大規模オブジェクトのコピーを避けるために, ムーブセマンティクスを使用する. (§3.2.2, §17.5.2)
 5.　多層型のオブジェクトの参照に, unique\_ptrを使用する. (§5.2.1)
 6.　解体の責任を持つ所有権を一意に特定できない共有オブジェクトの参照には, shared\_ptrを使用する. (§5.2.1)
 7.　C\+\+98とC\+\+11の詳しい差は (§44.2)

## 第2章-c++を探検しよう:基本
- **2.2.3 定数**
 -　const : この値を変更しないことをコンパイラに対して約束する.
 -　constexpr : この値はコンパイル時に評価される.
```cpp
const int dmv = 17; //dmvは名前の付いた定数
int val = 17; //varは定数でない
constexpr double max1 = 1.4*square(dmv); //square(17)が定数であればOK
constexpr double max2 = 1.4*square(var); //エラー : valは定数でない
const double max3 = 1.4*square(var); //OK : 実行時に評価される
double sum(const vector<double>&); //sumは実引数を変更しない
vector<double> v {1.2, 3.4, 4.5};
const double s1 = sum(v); //OK : 実行時に評価される
constexpr double s2 = sum(v); //エラー : sum(v)は定数式でない
```
 -　constexprは例えば, 単に演算結果をreturnするi行だけのようなできるだけ単純なものがいい.
```cpp
constexpr double square(double x) {return x*x;}
```
- **2.3.2 構造体**
 -　構造体/クラス関数への値渡し, ポインタ渡し, 参照渡しの違い
 　-　値渡し : 渡された変数の値が別のアドレスにコピーされる. だから関数内で変更しても, 呼び出しもとの変数の値は変化しない.
 　-　ポインタ渡し : 渡された変数のアドレスを指すポインタが別のアドレスに割り当てられてそれを通じて呼び出し元の変数の値を操作できる, だから関数内で変更すると, 呼び出し元も変更される.
 　-　参照渡し : 渡された変数は呼び出し元の変数と値, アドレス含め全く同じになる. これはコピーではないので当然関数ないで変更すると呼び出し元の値も変化する.
```cpp
void f(Vector v, Vector&rv, Vector* pv){
	int i1 = v.sz;
	int i2 = rv.sz;
	int i3 = pv->sz;
}
```
- **2.3.3 列挙体**
 -　クラス以外のユーザ定義型で比較的小さな整数値の集合を表すのに好都合.
 -　列挙型はユーザ定義型なので, 演算子も定義できる.
 -　オペレータの前に&は別につけなくてもいいが, 意味合いとしてオペレータは参照を表すのでこの書き方に従ったほうが良い.
```cpp
enum class Traffic_light{green, yellow, red};
Traffic_light light = Traffic_light::red;
　
Traffic_light& operator++(Traffic_light& t){
	switch(t){
		case Traffic_light::green: return t=Traffic_light::yellos;
		case Traffic_light::yellow: return t=Traffic_light::red;
		case Traffic_light::red: return t=Traffic_light::green;
	}
}
　
Traffic_light next = ++light;
```

## 第3章 C++を探検しよう:抽象化のメカニズム
- **3.2.4 クラス階層**
  -　空き領域に確保したオブジェクトのポインタを関数が返却することは以下の点で危険である.
  　-　ユーザは返却されたポインタのdeleteに失敗するかもしれない.
  　-　返却されたポインタが示すオブジェクトをユーザが認知せずdeleteしないかもしれない.
  -　そこでポインタを返却する場合にはunique_ptrを用いると解決できる.
```cpp
enum class Kind {circle, triangle, smiley};
　
unique_ptr<Shape>read_shape(istream& is){
	//... shapeの先頭部をisから読み込んでKind kを判断
	switch (k){
		case Kind::circle:
			// circleのデータ{Point,int}をpとrに読み込む
			return unique_ptr<Shape>{new Circle{p,r}};
		//...
	}
}
　
void user(){
	vector<unique_ptr<Shape>> v;
	while (cin)
		v.push_back(read_shape(cin));
	draw_all(v); // 全要素に対してdraw()を呼び出す
} // すべてのShapeは暗黙裏に解体される
```
- **3.3.1 コンテナのコピー**
  -　あるクラスがポインタ経由でオブジェクトにアクセスしなければならないクラスであれば, デフォルトのメンバ単位のコピーは, 災難となる場合がほとんど.
  -　よってコンテナ等のコピーには**コピーコンストラクタ**と**コピー代入**を定義する.
```cpp
class Vector{
provate:
	double* elem; // elemはsz個のdouble配列へのポインタ
	int sz;
public:
	Vector(int s); // コンストラクタ : 不変条件を確立して資源を獲得
	=Vector(){delete[] elem;} //デストラクタ : 資源を開放
　
	Vector(const Vector& a); //コピーコンストラクタ
	Vector& operator=(const Vector& a); //コピー代入
　
	double& operator[](int i);
	const double& operator[]{int i} const;
　
	int size() const;
};
　
Vector::Vector(const Vector& a)
	:elem{new double[a.sz]},
	 sz {a.sz}
{
	for (int i=0;i!=sz;++i){
		elem[i] = a.elem[i];
	}
}
　
Vector& Vector::operator=(const Vector& a){
	double* p = new double[a.sz];
	for (int i=0;i!=a.sz;++i)
		p[i] = a.elem[i];
	delete[] elem; //古い要素をデリート
	elem = p;
	sz = a.sz;
	return *this;
}
```
- **3.3.2 コンテナのムーブ**
  -　コピーコンストラクタ等を用いてコンテナのコピーができるが, コンテナが大規模の場合には厄介となる. ムーブコンストラクタによって単に関数から演算結果を取り出すことができる.
  -　以下のように定義すると, 関数の外に演算結果を転送する際に, コンパイラはムーブコンストラクタを使うようになる. 例えばr=x+y+ZではVectorは一切コピーされることなく, ムーブされる.
  -　"&&"は”右辺値参照"を意味し, 他の誰も代入を行えない何かを参照するものであり, 安全に値を盗むことができる.
```cpp
class Vector{
	//...
	Vector(const Vector& a); //コピーコンストラクタ
	Vector& operator=(const Vector& a); //コピー代入
　
	Vector(Vector&& a); //ムーブコンストラクタ
	Vector& operator=(Vector&& a); //ムーブ代入
}
　
Vector::Vector(Vector&& a)
	:elem{a.elem}, //aから要素を盗む
	sz{a.sz}
{
	a.elem = nullptr; // もはやaには要素は全くない
	a.sz = 0;
}
```
- **3.3.4 演算子の抑制**
  -　階層内のクラスに対して, デフォルトのコピーやムーブを利用すると惨事につながる.
  -　以下のように書くと, デフォルトのコピー演算, ムーブ演算を削除できる(デフォルト定義の削除).
```cpp
class Shape{
public:
	Shape(const Shape& a) =delete;
	Shape& operator=(const Shape&) =delete;
　
	Shape(Shape&&) =delete;
	Shape& operator=(Shape&&) =delete;
}
```
- **3.4.4 可変個引数テンプレート**
  -　任意の型と任意の個数の引数を受け取ることができる.
  -　先頭のheadに対して何等かの処理を行って, それ以降の引数tailに対してf()を再帰的に呼びだす.
```cpp
void f(){} //何も行わない
template<typename T>
void g(T x){cout << x << " ";}
　
template<typename T, typename... Tail>
void f(T head, Tail... tail){
	g(head);
	f(tail...);
}
　
int main(){
	f(1, 2.2, "hello"); //output: 1 2.2 hello
	f(0.2, 'c', "yuck!", 0, 1, 2); // output: 0.2 c yuck! 0 1 2
}
```
- **3.4.5 別名**
  -　usingで型やテンプレートに同義語を与える.
```cpp
using size_t = unsigned int;
```
  -　例えば標準コンテナは, 自身が保持する値の型の名前としてvalue_typeを提供し, この規約に従うあらゆるコンテナに対して動作するコードが記述できる.
```cpp
template<typename T>
class Vector{
public:
	using value_type = T;
	// ...
}
　
template<typename C>
using Value_type = typename C::valuetype; // Cの要素の型
　
template<typename Container>
void algo(Container& c){
	Vector<Value_type<Container>> vec; // ここに結果を保存
	// ... vecを利用 ...
}
```
  -　別名は, テンプレートの引数の一部またはすべてをバインドして, 新しいテンプレートを定義できる.
```cpp
template<typename Key, typename Value>
class Map{
	// ...
};
　
template<typename Value>
using String_map = Map<string, Value>;
　
String_map<int> m; // mはMap<string, int>
```

## 第4章 C++を探検しよう:コンテナとアルゴリズム
- **to be continued ...**