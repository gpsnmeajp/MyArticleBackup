#はじめに
例外の差異に例外クラスを使ってthrowする必要を感じたので，どうしたら良いか調べようと思っても中々思ってる情報が出てこないので書きました．
C++はほぼ全くの初心者です．Better Cとして使ってます．
間違い等があればご指摘いただけると幸いです．

#意味
- throw "hogehoge";とするのは簡単だが，受け取った側は「あ，なんか例外来た．ユーザーに表示して終了しよう」くらいしかできない．
- あるいは握りつぶすか，タイミングで推測するか
- 受け取り側のプログラムにエラーの内容がわかるように例外を投げる→例外クラスを使う

#作り方
**コメントにて指摘があったので追記**

- std::exceptionを継承する**か，あるいはstd::runtime_errorを継承する**
- そうすることで，std::exceptionをcatchすることでまとめて捕まえることができて便利．(握りつぶしたいときとか，とりあえず全部キャッチしたいときとか)
- ~~std::exceptionはそのままではメッセージくらいしか出せない．e.what()で読める．~~ **→ これはMSの独自仕様らしい．std::runtime_errorなどは標準の仕様でメッセージが出せる．**
- デバッグする人間のためにe.what()で概要をつかめるようにすべき
- 追加のデータ構造は自分で持たせる

```cpp:
	class PCSCCommandException : public std::runtime_error
	{
	public:     //元々std::runtime_errorにある文字列↓   ↓追加の情報
		PCSCCommandException(const char *_Message, int res)
			: _Errinfo(res), runtime_error(_Message)
                                      //↑こうすることでwhat()で人間向けメッセージが読める
		{}

           //↓追加のエラー情報を返すGetter
		int returncode()
		{
			return _Errinfo;
		}
	private:
		int _Errinfo;
	};
```

- 本来こういうのは良くないのかもしれないが，自分は詳細なエラー情報を別途持たせたりしている．

```cpp:
	class PCSCStateException : public std::runtime_error
	{
	public:
		enum
		{
			FAILD_TO_ESTABLISH,
			NOT_CONNECTED_TO_SERVICE,
			NOT_CONNECTED_TO_READER,
			NOT_CONNECTED_TO_CARD,
			NO_READERS_AVAILABLE,
			FAILED_TO_DETECT_READER,
			FAILED_TO_GET_CARD_STATUS,
			READER_DISCONNECTED,
			UNKNOWN_STATUS,
			NO_CARD_FOUND,
			UNKNOWN_ERROR,
			FAILED_TO_DIRECT_CONNECTION,
			CARD_REMOVED,
		};

		PCSCStateException(const char *_Message, int errorcode, LONG ret)
			: _Errinfo(), runtime_error(_Message)
		{
			_Errinfo.errorcode = errorcode;
			_Errinfo.ret = ret;
		}

		int errorcode()
		{
			return _Errinfo.errorcode;
		}
		LONG returncode()
		{
			return _Errinfo.ret;
		}
	private:
		struct PCSCStateError_struct
		{
			int errorcode;
			LONG ret;
		};

		PCSCStateError_struct _Errinfo;
	};

```
