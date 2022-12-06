> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [learnku.com](https://learnku.com/articles/64405)

> 之前项目有一个需求，业务人员使用中文编写一些自定义公式，然后需要我们后台执行将结果返回到界面上，于是就基于有限状态机写了这个词法分析器，比较简单，希望能够抛砖引玉。

最初的想法是使用字符串替换的方式，将中文关键字替换成 php 的关键字，然后调用 eval 执行，这样确实也是可以的，但是总觉得不是很美丽，并且不能实现动态解析。就想着自己实现一个简单的词法分析，然后结合 ast 将词法转换成 php 代码执行，岂不快哉。当前版本没有用到抽象语法树来生成代码，全部使用字符串拼接。

```
<?php

/**
 * Class Lexer
 * @package Sett\OaLang
 * 词法分析器
 */
class Lexer {
    // 内置关键字集合
    public $keywordList = [];
    // 内置操作符集合
    public $operatorList = [
        "+", "-", "*", "/", "=", ">", "<", "!", "(", ")", "{", "}", ",", ";"
    ];
    // 源代码
    private $input;
    // 当前的字符
    private $currChar;
    // 当前字符位置
    private $currCharPos = 0;
    // 结束符
    private $eof = "eof";
    // 当前编码
    private $currEncode  = "UTF-8";

    // 内置关键字
    public const VAR = "variable";
    public const STR = "string";
    public const KW  = "keyword";
    public const OPR = "operator";
    public const INT = "integer";
    public const NIL = "null";


    /**
     * Lexer constructor.
     * @param string $input
     */
    public function __construct(string $input) {
        $this->input    = $input;
        $this->currChar = mb_substr($this->input, $this->currCharPos, 1);
    }

    /**
     * @param array $keywordList
     */
    public function setKeywordList($keywordList) {
        $this->keywordList = $keywordList;
    }

    /**
     * @return array
     * @throws Exception
     */
    public function parseInput() {
        if ($this->input == "") {
            throw new Exception("code can not be empty");
        }
        $tokens = [];
        do {
            $token = $this->nextToken();
            if ($token["type"] != "eof") {
                $tokens[] = $token;
            }
            if ($token["type"] == self::KW) {
                $tokens[] = $this->makeToken(self::NIL, " ");
            }
        } while ($token["type"] != "eof");
        return $tokens;
    }

    /**
     * @return array
     */
    public function nextToken() {
        $this->skipBlankChar();
        $this->currChar == "" && $this->currChar = $this->eof;
        if ($this->isCnLetter()) {
            $word = $this->matchUntilNextCharIsNotCn();
            if ($this->isKeyword($word)) {
                $this->currCharPos -= 1;
                return $this->currToken(static::KW, $word);
            }
            // 不是关键字的全部归为变量
            return $this->makeToken(static::VAR, $word);
        }
        // 如果是操作符
        if ($this->isOperator()) {
            return $this->currToken(static::OPR, $this->currChar);
        }
        // 如果是数字
        if ($this->isNumber()) {
            return $this->currToken(static::INT, $this->currChar);
        }
        // 如果是字符串
        if ($str = $this->isStr()) {
            return $this->currToken(static::STR, $str);
        }
        // 如果是变量
        if ($this->isVar()) {
            $word = $this->matchVar();
            if ($this->isKeyword($word)) {
                return $this->currToken(static::KW, $word);
            }
            return $this->makeToken(static::VAR, $word);
        }
        if ($this->currChar == $this->eof) {
            return $this->currToken('eof', $this->currChar);
        }
        return $this->currToken(static::VAR, $this->currChar);
    }

    /**
     * @param string $input
     * @return string
     */
    private function matchVar(string $input = "") {
        $word = $input ?: '';
        while ($this->isVar()) {
            $word .= $this->currChar;
            $this->nextChar();
        }
        return $word;
    }

    /**
     * @return bool
     * 是否为普通变量
     */
    private function isVar() {
        return $this->isCnLetter() || $this->isEnLetter();
    }


    /**
     * 跳过空白字符
     */
    private function skipBlankChar() {
        while (ord($this->currChar) == 10 ||
            ord($this->currChar) == 13 ||
            ord($this->currChar) == 32) {
            $this->nextChar();
        }
    }

    /**
     * @param string $type
     * @param $word
     * @return array
     * 记录当前token和下一个字符
     */
    private function currToken(string $type, $word) {
        $token = $this->makeToken($type, $word);
        $this->nextChar();
        return $token;
    }

    /**
     * @param string $type
     * @param string $char
     * @return array
     */
    private function makeToken(string $type, string $char) {
        return ["type" => $type, "char" => $char, "pos" => $this->currCharPos];
    }


    /**
     * @return bool
     * 判断是否是英文字符
     */
    private function isEnLetter() {
        if ($this->currChar == "" || $this->currChar == $this->eof) {
            return false;
        }
        $ord = mb_ord($this->currChar, $this->currEncode);
        if ($ord > ord('a') && $ord < ord('z')) {
            return true;
        }
        return false;
    }

    /**
     * @return false|int
     * 是否中文字符
     */
    private function isCnLetter() {
        return preg_match("/^[\x{4e00}-\x{9fa5}]+$/u", $this->currChar);
    }

    /**
     * @return bool
     * 是否为数字
     */
    private function isNumber() {
        return is_numeric($this->currChar);
    }

    /**
     * @return bool
     * 是否是字符串
     */
    private function isStr() {
        return $this->matchCompleteStr();
    }

    /**
     * @return string
     * 匹配完整字符串
     */
    private function matchCompleteStr() {
        $char = "";
        if ($this->currChar == "\"") {
            $this->nextChar();
            while ($this->currChar != "\"") {
                if ($this->currChar != "\"") {
                    $char .= $this->currChar;
                }
                $this->nextChar();
            }
            return $char;
        }
        return $char;
    }

    /**
     * @return bool
     * 是否是操作符
     */
    private function isOperator() {
        return in_array($this->currChar, $this->operatorList);
    }

    /**
     * @return string
     * 匹配中文字符
     */
    private function matchUntilNextCharIsNotCn() {
        $char = "";
        while ($this->isCnLetter()) {
            $char .= $this->currChar;
            $this->nextChar();
        }
        return $char;
    }

    /**
     * @return void 获取下一个字符
     * 获取下一个字符
     */
    private function nextChar() {
        $this->currCharPos += 1;
        $this->currChar    = mb_substr($this->input, $this->currCharPos, 1);
        if ($this->currChar == "") {
            $this->currChar = $this->eof;
        }
    }

    /**
     * @param string $input
     * @return bool
     * 是否是关键字
     */
    private function isKeyword(string $input) {
        return ($this->keywordList[$input] ?? "") != "";
    }

    public function convert(array $tokens) {
        $code = "";
        foreach ($this->lexerIterator($tokens) as $generator) {
            switch ($generator["type"]) {
                case static::KW:
                    $code .= $this->keywordList[$generator["char"]];
                    break;
                case static::VAR:
                    $code .= sprintf("$%s", $generator["char"]);
                    break;
                case static::OPR:
                    $code .= $this->replace($generator["char"]);
                    break;
                case static::INT:
                    $code .= $generator["char"];
                    break;
                case static::STR:
                    $code .= sprintf("\"%s\"", $generator["char"]);
                    break;
                default:
                    $code .= $generator["char"];
            }
        }
        return $code;
    }

    private function replace(string $char) {
        return str_replace("+", ".", $char);
    }

    /**
     * @param array $tokens
     * @return \Generator
     */
    private function lexerIterator(array $tokens) {
        foreach ($tokens as $index => $token) {
            yield $token;
        }
    }
}

```

```
[{
    "type": "variable",
    "char": "姓名",
    "pos": 2
}, {
    "type": "operator",
    "char": "=",
    "pos": 2
}, {
    "type": "string",
    "char": "腕豪",
    "pos": 7
}, {
    "type": "operator",
    "char": ";",
    "pos": 8
}, {
    "type": "variable",
    "char": "问候",
    "pos": 13
}, {
    "type": "operator",
    "char": "=",
    "pos": 13
}, {
    "typ e": "string",
    "char": "你好啊",
    "pos": 17
}, {
    "type": "operator",
    "char": ";",
    "pos": 18
}, {
    "type": "variable",
    "char": "地址",
    "pos": 23
}, {
    "type": "operator",
    "char": "=",
    "pos": 23
}, {
    "type": "operator",
    "char": "(",
    "pos": 24
}, {
    "type": "integer",
    "char": "1",
    "pos": 25
}, {
    "type": "operator",
    "char": " +",
    "pos": 26
}, {
    "type": "integer",
    "char": "2",
    "pos": 27
}, {
    "type": "operator",
    "char": ")",
    "pos": 28
}, {
    "type": "operator",
    "char": "*",
    "pos": 30
}, {
    "type": "integer",
    "char": "3",
    "pos": 32
}, {
    "type": "operator",
    "char": ";",
    "pos": 33
}, {
    "type": "keyword",
    "char": "如果",
    "pos": 37
}, {
    "type": "nul l",
    "char": " ",
    "pos": 38
}, {
    "type": "operator",
    "char": "(",
    "pos": 38
}, {
    "type": "variable",
    "char": "地址",
    "pos": 41
}, {
    "type": "operator",
    "char": ">",
    "pos": 42
}, {
    "type": "integer",
    "char": "3",
    "pos": 44
}, {
    "type": "operator",
    "char": ")",
    "pos": 45
}, {
    "type": "operator",
    "char": "{",
    "pos": 46
}, {
    "type": "variable",
    "char": "地址",
    "pos": 55
}, {
    "type": "operator",
    "char": "=",
    "pos": 55
}, {
    "type": "integer",
    "char": "1",
    "pos": 56
}, {
    "type": "operator",
    "char": ";",
    "pos": 57
}, {
    "type": "operator",
    "char": "}",
    "pos": 60
}, {
    "type": "keyword",
    "char": "否则",
    "pos": 62
}, {
    "type": "null",
    "char ": " ",
    "pos": 63
}, {
    "type": "operator",
    "char": "{",
    "pos": 63
}, {
    "type": "variable",
    "char": "地址",
    "pos": 72
}, {
    "type": "operator",
    "char": "=",
    "pos": 72
}, {
    "type": "string",
    "char": "艾欧尼亚",
    "pos": 78
}, {
    "type": "operator",
    "char": ";",
    "pos": 79
}, {
    "type": "operator",
    "char": "}",
    "pos": 82
}, {
    "type": "variable",
    "char": "说话",
    "pos": 87
}, {
    "type": "operator",
    "char": "=",
    "pos": 88
}, {
    "type": "operator",
    "char": "(",
    "pos": 90
}, {
    "type": "string",
    "char": "我",
    "pos": 93
}, {
    "type": "operator",
    "char": "+",
    "pos": 94
}, {
    "type": "string",
    "char": "爱",
    "pos": 97
}, {
    "type": "operator",
    "char": ")",
    "pos": 98
}, {
    "type": "operator",
    "char": "+",
    "pos": 99
}, {
    "type": "string",
    "char": "你",
    "pos": 102
}, {
    "type": "operator",
    "char": ";",
    "pos": 103
}, {
    "type": "keyword",
    "char": "返回",
    "pos": 107
}, {
    "type": "null",
    "char": " ",
    "pos": 108
}, {
    "type": "variable",
    "char": "姓名",
    "pos": 111
}, {
    "typ e": "operator",
    "char": "+",
    "pos": 111
}, {
    "type": "variable",
    "char": "年龄",
    "pos": 114
}, {
    "type": "operator",
    "char": ";",
    "pos": 114
}]

```