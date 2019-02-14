# Unicode Filter
PHP Unicode string filter library based on [Unicode Blocks](http://ftp.unicode.org/Public/UNIDATA/Blocks.txt) defined by Unicode 11.0 standard

# Usage

## Basic Usage

`UnicodeFilter::whitelist($input, $filters = [], $excepts = [], $replacement = '')`

- Keep only characters in BASIC_LATIN block
```php
echo UnicodeFilter::whitelist("Hello World! 😃", [
    UnicodeFilter::BASIC_LATIN
]);

// Hello World! 
```

- Keep only characters in BASIC_LATIN block and replace everything else with underscore "_"
```php
echo UnicodeFilter::whitelist("Hello World! 😃", [
    UnicodeFilter::BASIC_LATIN
], [], "_");

// Hello World! _
```

`UnicodeFilter::blacklist($input, $filters = [], $excepts = [], $replacement = '')`

- Remove only characters in EMOTICONS block
```php
echo UnicodeFilter::blacklist("Hello World! 😃", [
    UnicodeFilter::EMOTICONS
]);

// Hello World! 
```

- Return `true`/`false` if string is processed

`UnicodeFilter::isWhitelistProcessed($input, $filters = [], $excepts = [])`

`UnicodeFilter::isBlacklistProcessed($input, $filters = [], $excepts = [])`

- `$filter` and `$excepts` can accept array of following formats
  - Block Name (e.g. UnicodeFilter::BASIC_LATIN)
  - Arbitary decimal codepoint (e.g. 0x200b, mb_ord("好"))
  - Arbitary decimal codepoint range (e.g. [0x2000, 0x200F])

## Advanced Usage

- Keep only characters in BASIC_LATIN block but excepted range U+00..U+20, replace everything else with underscore "_"
```php
echo UnicodeFilter::whitelist("Hello\nWorld! 😃", [
    UnicodeFilter::BASIC_LATIN
], [
    [0x00, 0x20]
], "_");

// Hello_World! _
```

- Keep only (most) characters in English, Chinese, Japanese and Korean
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
// (Thai is not included so it's removed) 
```

- Keep only (most) characters in English, Chinese, Japanese, Korean, Thai and also [General Punctuation](https://www.compart.com/en/unicode/block/U+2000) 
    and an additional 😃 character
    but excepted characters in range U+2000..U+200F and U+205F..U+206F (Unprintable characters)
    and finally replace any other characters with underscore
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
], "_");

// ‷Hello_您好_こんにちは_안녕하세요_สวัสดีค่ะ‴ 😃
```

- Generate array of details (codepoint and block) for each characters of given string

`analysis($string)`

```
array(14) {
  [0]=>
  array(3) {
    ["character"]=>
    string(1) "H"
    ["codepoint"]=>
    int(72)
    ["block"]=>
    string(11) "BASIC_LATIN"
  }
  ...
}
```

- Generate detail of how whitelist/blacklist is processed and it's results

`whitelistInfo($input, $filters = [], $excepts = [], $replacement = '')`

`blacklistInfo($input, $filters = [], $excepts = [], $replacement = '')`

```
array(6) {
  ["input"]=>
  string(12) "Hello 您好"
  ["output"]=>
  string(6) "Hello "
  ["pattern"]=>
  string(18) "/[^\x{0}-\x{7f}]/u"
  ["isProcessed"]=>
  bool(true)
  ["processedCount"]=>
  int(2)
  ["processedCharacters"]=>
  string(6) "您好"
}
```

## Debug Functions

- Dump whitelist/blacklist info to console

`dumpWhitelistInfo($input, $filters = [], $excepts = [], $replacement = '')`

`dumpBlacklistInfo($input, $filters = [], $excepts = [], $replacement = '')`

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

## Common Issues

- Whitelist is more preferred way to work with, because there are too many characters (137,374 characters as of Unicode 11.0)

- Some language, especailly Chinese and sorth-east asia language have characters spread over multiple blocks
  For example there are CJK_COMPATIBILITY, CJK_UNIFIED_IDEOGRAPHS_EXTENSION_A, CJK_UNIFIED_IDEOGRAPHS, CJK_COMPATIBILITY_IDEOGRAPHS... blocks.
  Therefore multiple tests needed to include all blocks you may actually need
  

## Reference

- https://unicode.org/
- https://www.compart.com/en/unicode/block
