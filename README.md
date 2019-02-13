# Unicode Filter
PHP Unicode string filter library base on [Unicode Blocks](ftp://ftp.unicode.org/Public/UNIDATA/Blocks.txt) defined by Unicode 11.0 standard

# Usage

## Basic Usage

`UnicodeFilter::whitelist($input, $filters = [], $excepts = [], $replacement = '')`

- Remove everything out of BASIC_LATIN block
```php
echo UnicodeFilter::whitelist("Hello·World!", [
    UnicodeFilter::BASIC_LATIN
]);

// HelloWorld! 
```

- Replace everything out of BASIC_LATIN block and replace them as space " "
```php
echo UnicodeFilter::whitelist("Hello·World!", [
    UnicodeFilter::BASIC_LATIN
], [], " ");

// Hello World! 
```

`UnicodeFilter::blacklist($input, $filters = [], $excepts = [], $replacement = '')`

- Remove all characters in EMOTICONS block
```php
echo UnicodeFilter::blacklist("Hello World! 😃", [
    UnicodeFilter::EMOTICONS
]);

// Hello World! 
```

## Advanced Usage

- Allow most characters in English, Chinese, Japanese and Korean (Thai is not included so it's removed) 
```php
echo UnicodeFilter::whitelist("Hello 您好 こんにちは 안녕하세요 สวัสดีค่ะ", [
    UnicodeFilter::BASIC_LATIN,
    UnicodeFilter::CJK_UNIFIED_IDEOGRAPHS,
    UnicodeFilter::CJK_COMPATIBILITY,
    UnicodeFilter::HIRAGANA,
    UnicodeFilter::KATAKANA,
    UnicodeFilter::HANGUL_SYLLABLES
]);

// Hello 您好 こんにちは 안녕하세요
```

- Allow most characters in English, Chinese, Japanese, Korean, Thai and also [General Punctuation](https://www.compart.com/en/unicode/block/U+2000) 
  finally in additional allow 😃 character
  but not allow characters in range U+2000..U+200F and U+205F..U+206F (Unprintable characters)
  replace other characters as underscore
```php
echo UnicodeFilter::whitelist("‷Hello×您好×こんにちは×안녕하세요×สวัสดีค่ะ‴ 😃", [
    UnicodeFilter::BASIC_LATIN,
    UnicodeFilter::CJK_UNIFIED_IDEOGRAPHS,
    UnicodeFilter::CJK_COMPATIBILITY,
    UnicodeFilter::HIRAGANA,
    UnicodeFilter::KATAKANA,
    UnicodeFilter::HANGUL_SYLLABLES,
    UnicodeFilter::THAI,
    UnicodeFilter::GENERAL_PUNCTUATION,
    mb_ord('😃')
], [
    [0x2000, 0x200F],
    [0x205F, 0x206F]
], "?");

// ‷Hello_您好_こんにちは_안녕하세요_สวัสดีค่ะ‴ 😃
```

## Debug Functions

`dumpWhitelistInfo($input, $filters = [], $excepts = [])`

`dumpBlacklistInfo($input, $filters = [], $excepts = [])`

```php
echo UnicodeFilter::dumpWhitelistInfo("Hello 您好", [
    UnicodeFilter::BASIC_LATIN
]);
```

```Input:  Hello 您好 (8)
Output: Hello  (6)
Pattern: /[^\x{0}-\x{7f}]/u
Processed: Yes
Processed Characters: 2
您 (U+60a8) in block CJK_UNIFIED_IDEOGRAPHS
好 (U+597d) in block CJK_UNIFIED_IDEOGRAPHS
```

`dumpString($string)`

```php
echo UnicodeFilter::dumpString("Hello×您好×こんにちは", [
    UnicodeFilter::BASIC_LATIN
]);
```

```
H (U+48) in block BASIC_LATIN
e (U+65) in block BASIC_LATIN
l (U+6c) in block BASIC_LATIN
l (U+6c) in block BASIC_LATIN
o (U+6f) in block BASIC_LATIN
× (U+d7) in block LATIN_1_SUPPLEMENT
您 (U+60a8) in block CJK_UNIFIED_IDEOGRAPHS
好 (U+597d) in block CJK_UNIFIED_IDEOGRAPHS
× (U+d7) in block LATIN_1_SUPPLEMENT
こ (U+3053) in block HIRAGANA
ん (U+3093) in block HIRAGANA
に (U+306b) in block HIRAGANA
ち (U+3061) in block HIRAGANA
は (U+306f) in block HIRAGANA
```

`dumpFilters($filters = [])`

```php
echo UnicodeFilter::dumpFilters([
    UnicodeFilter::BASIC_LATIN,
    UnicodeFilter::LATIN_1_SUPPLEMENT,
    UnicodeFilter::LATIN_EXTENDED_A,
    UnicodeFilter::LATIN_EXTENDED_B
]);
```

```
BASIC_LATIN / U+0..U+7f
LATIN_1_SUPPLEMENT / U+80..U+ff
LATIN_EXTENDED_A / U+100..U+17f
LATIN_EXTENDED_B / U+180..U+24f
```

## Reference

- https://www.compart.com/en/unicode/block
