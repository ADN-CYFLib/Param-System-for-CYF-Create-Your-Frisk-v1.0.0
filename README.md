Param System for CYF(Create Your Frisk)

CYFでゲームを作成している皆さん、こんばんは。ADNです。
このGitでは、例えばAnimations/Sans.luaにあるファイルを、どんなファイルからでも同じ環境にアクセスできるように出来るコードを紹介するGitです。

ただ単にWaves/Wave1.luaでlocal Sans = require("Animations/Sans")をしただけじゃ、Waves/Wave2.luaで同じようにrequireしたとしても、同じスコープにアクセス出来ません。

そこで目をつけたのが、ゲーム全体を通じてスコープが破棄されないEncounter.luaです。

Encounter.luaならEncounter.Call()で他のファイルから自由にアクセスできるので、それを利用して同じ環境にアクセスし、Waveを超えても常に同じ環境を操作できるようになります。

そこでexpr:matchの出番です。
本来Encounter.Call()は、("Function名", "数値、または文字")しか使えませんが、expr:matchを使うと、("function名", "数値1,文字2,数値3")などと自由に入力し、実行することが出来ます。

それで私は、expr:matchを利用し、単なる数式ではなく、320+50 やmath.pi*2といった数学的な式をパラメーターとして渡すことを可能にしたコードを作りました。

────────────────
```
local function evalExpression(expr)
    expr = (expr and expr:match("^%s*(.-)%s*$")) or ""
    if expr == "" then return nil end

    if not expr:match("^[%d%+%-%*/%%%(%)%.%s^math]+$") then
        return nil
    end

    local env = { math = math }

    local f = load("return " .. expr, "expr", "t", env)
    if f then
        local ok, result = pcall(f)
        if ok then
            return result
        end
    end
    return nil
end
```
────────────────
expr = (expr and expr:match("^%s*(.-)%s*$")) or ""
ここでexprが存在するか確認し、その文字列の前後にある空白文字を全て削除します。

if expr == "" then return nil end
文字列が空っぽだったら何も計算するものがないのでnilを返して終了します。

if not expr:match("^[%d%+%-%*/%%%(%)%.%s^math]+$") then
        return nil
end
式の中に許可されていない文字が含まれていないかをチェックし、含まれていたらnilを返します。

local env = { math = math }
計算を実行するための環境を準備します。

local f = load("return " .. expr, "expr", "t", env)
"return "という文字列と、expr(例: 320+100)を結合し、それをload()関数を使って実行可能な関数(f)にコンパイルします。envによってサンドボックス内で実行されます。

if f then
コンパイルが成功し、実行可能な関数が作成されたかを確認します。

local ok, result = pcall(f)
pcallを使って関数fを実行します。pcallを使っているので、計算中にエラーが起きてもゲーム全体がクラッシュするのを防げます。

if ok then
    return result
end
return nil
計算がエラーなく成功したかを確認し、計算結果(例:420)を返します。
コンパイルに失敗した場合、または pcall で計算エラーが起きた場合に、nilを返して終了します。
────────────────
​導入方法と使い方

​このParam Systemの導入は簡単です。以下のコードを Encounter.lua の適切な場所にコピー＆ペーストし、その後に必要な関数を定義するだけです。

​詳しい関数の定義や実装方法については、Gitをダウンロードしてご確認ください。

Androidの方は、
パッケージにあるzipをダウンロードし、zipの中のModsにある
evalExpressionPC_and_Androidを
/storage/emulated/0/Android/data/com.YXY_123.CYF/files/Mods/
または、
/storage/emulated/0/Android/data/com.HadrianPorts.CreateYourDroid/files/Mods/
にGitのファイルを置いて下さい。

PCの方は、
パッケージにあるzipをダウンロードし、zipの中の適切なModsフォルダにGitを置いてexeを起動してください。



## MIT License

Copyright (c) 2025 ADN-UT

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
