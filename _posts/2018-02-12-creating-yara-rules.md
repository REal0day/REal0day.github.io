---
layout: post
title: Creating Yara Rules for Malware Detection
date: 2018-02-12 21:28:16
description: Tutorial on Creating Yara Rules
tags: snort hacking yara reverse
categories: blog
---

![Yara Girl](/assets/img/yara-girl.gif "Yara Girl"){:class="featured-image"}

# Introduction
We all know it's way more fun to hack shit than to patch shit. That said, not all employers will be satisfied with a hacker who can only compromise systems. Some companies want security researchers that are able to apply patches based on malware samples/breach data they have collected or have found in the wild.

**Author Assigned Level**: Newbie or Wannabe

**Required Skills**:
There really aren't too many skills required for this. The deeper you understand malware anlaysis and reverse engineering, the more capable you'll be at finding unique ways to catch malware. But this won't hinder you from writing amazing yara rules. Most of the rules I've seen are pretty basic. Most look like a python script that took 5 minutes to write. The skill and detail comes in the analysis. Not in the actual yara rule itself.

* GNU Linux
* Familiar with C syntax (not required, but useful)
* Regex (not required, but useful)

**Disclaimer**: I learned yara on the streets, not in the schools. I have about 30hrs expereince with yara. A weekend for me.

# The Paper
I'll be going over the following:

1. **Rule Identifiers**
2. **Yara Keywords**
3. **Strings**
   - Hexadecimal
   - Text Strings
   - String Modifiers
   - Regular Expressions
   - Sets of Strings
   - Anonymous Strings
4. **Conditions**
   - Boolean
   - Counting String Instances
   - String Offsets or Virtual Addresses
   - Match Length
   - File Size
   - Executable `entry_point`
   - Accessing Data at a Given Position
   - Applying One Condition Across Many Strings
   - Iterating Over String Occurrences
5. **Referencing Other Rules**
6. **Yara Essentials**
   - Global Rules
   - Private Rules
   - Rule Tags
   - Metadata
   - Using Modules
   - Undefined Values
   - External/Argument Values
   - Including Files

2. **Private Rules**
3. **Rule Tags**
4. **Metadata**
5. **Using Modules**
6. **Undefined Values**
7. **External/Argument Values**
8. **Including Files**

Let's get started. I want to do something else tonight besides just doc.

## Writing Yara Rules

Yara mostly resembles the syntax of the C language. Here is a simple rule that does nothing:

```c
rule HelloRule {
    condition:
        false
}
```
---

## Rule Identifiers
The word the follows rule, in this case “dummy”, is known as the rule identifier. They can be:

* alphanumeric characters
* underscore character
* first char can't be a digit
* case-sensitive
* cannot exceed 128 characters

## Yara Keywords
The following can't be used as a rule identifier because they're special to the yara language.

* all
* and
* any
* ascii
* at
* condition
* contains
* entrypoint
* false
* filesize
* fullword
* for
* global
* in
* import
* include
* int8
* int16
* int32
* int8be
* int16be
* int32be
* matches
* meta
* nocase
* not
* or
* of
* private
* rule
* strings
* them
* true
* uint8
* uint16
* uint32
* uint8be
* uint16be
* uint32be
* wide

Generally, yara has two sections: **strings definition **and condition.

```bash
rule HelloRule2    // This is an example
{
    strings:
        $my_text_string = "text here"
        $my_hex_string = { E2 34 A1 C8 23 FB }

    condition:
        $my_text_string or $my_hex_string
}
```

This rule will be active when either string is found.
As you can see, you can also add comments.

## Hexadecimal Strings
### Wildcards
Acceptable uses for hex-strings are wildcards, which are represented with a `?` mark.
```bash
rule GambitWildcard
{
    strings:
       $hex_string = { EF 44 ?? D8 A? FB }

    condition:
       $hex_string
}
```
This will catch any of the following:

```
EF 44 01 D8 AA FB
EF 44 AA D8 AB FB
```

### Unknown Length of Wildcard
Strings with an unknown length can be represented as the following:

```bash
rule MarioJump
{
        strings:
           $hex_string = { F4 23 [4-6] 62 B4 }

        condition:
           $hex_string
}
```
This will catch any of the following:
```
F4 23 01 02 03 04 62 B4
F4 23 AA BB CC DD EE FF 62 B4
```
**Infinite** is also possible.
```bash
rule BuzzLightyear
{
        strings:
           $hex_string = { F4 23 [-] 62 B4 }

        condition:
           $hex_string
}
```
This will catch any of the following:
```
F4 23 AA FF 62 B4
F4 23 AA AA AA AA AA...FF FF 62 B4
```
## Conditional Strings
You can create 1 to as many statements as you like.
```bash
rule WorriedRabbit
{
    strings:
       $hex_string = { BA 21 ( DA BC | C6 ) A5 }

    condition:
       $hex_string
}
```
This will catch any of the following:
```
BA 21 DA BC A5
BA 21 C6 A5
```

### Mixing it all up
You can also combine them all, of course.
```bash
rule WorriedGabmitLightyearJump
{
    strings:
       $hex_string = { BA ?? ( DA [2-4] | C6 ) A5 }

    condition:
       $hex_string
}
```
This will catch any of the following:

```
BA 01 DA 01 02 03 04 A5
BA AA C6 A5
BA FF DA 01 02 A5
```

## Text Strings
An alternative to hex-strings, one can also use text strings.
```bash
rule KimPossible
{
    strings:
        $alert_string = "Whats the Sitch"

    condition:
       $alert_string
}
```


One can also use the following escape sequences, just like in C:

* `*\` **Double Quotes
* `**` Backslash
* `\t` Horizontal Tab
* `\n` New line
* `\xdd` Any byte in hexadecimal notation

## Modifiers
### Case-insensitive strings
By default, Yara is case-sensitive, but you can turn that off.
```bash
rule ThickSkin
{
    strings:
        $strong_string = "Iron" nocase

    condition:
        $strong_string
}
```
### Wide-character strings
The wide modifer can be used to search for strings encoded with two bytes per character, something typically in many executable binaries. If the string “FatTony” appears encoded as two bytes per character, it will be caught if we use the modifer wide. Let's also add the nocase modifier as “FatTony” might be “fattony” and we wouldn't want to miss that.
```bash
rule FatTony
{
    strings:
        $fat_villain = "FatTony" wide nocase

    condition:
        $fat_villain
}
```

**[!] Important [!]** - Keep in mind that this modifier interleaves the `ASCII` codes of the characters in the string with zeroes, it does not support truly UTF-16 strings containing non-English characters. To add a search for strings in both `ASCII` and `wide`, use the following:
```bash
rule ASCIIFatTony
{
    strings:
        $fat_villain = "FatTony" wide ascii nocase

    condition:
        $fat_villain
}
```
**ASCII is assumed by default** so you don't have to add ascii if you want to search for FatTony by ascii alone.
```bash
rule ASCIIFatTony
{
    strings:
        $fat_villain = "FatTony"

    condition:
        $fat_villain
}
```
This works if you want to search without the wide and nocase modifiers.

### Fullwords Modifier
This modifier will catch on words that DO NOT have prepend and append the word with a character.
```bash
rule ShadyDomain
{
    strings:
        $shady_domain = "faceebook" fullword

    condition:
       $shady_domain
}
```
This will catch any of the following:
```
www.faceebook.com
www.myportal.faceebook.com
https://secure.faceebook.com
```
This will **NOT CATCH** any of the following:
```
www.myfaceebook.com
thefaceebook.com
```
The difference is that that the fullword is prepended or appended by a *special character*, not a regular character.

## Regular Expression
Enclosed in forward slashes instead of double quotes, (like Perl Programming), yara allows for RegEx.
```bash
rule RegularShow
{
    strings:
        $re1 = /md5: [0-9a-fA-F]{32}/
        $re2 = /state: (on|off)/

    condition:
        $re1 and $re2
}
```
This will catch any md5 string it finds, in either state.

One can also apply text modifiers such as nocase,** ascii**,** wide**,** **and **fullword **to RegEx as well.

## Metacharacters
A metacharacter is a character that has a special meaning (instead of a literal meaning) to a computer program. For RegEx, these are the following meanings

* `**`: Quote the next metacharacter
* `^`: Match the beginning of the file
* `$`: Match the end of the file
* `|`: Alternation
* `()`: Grouping
* `[]`: Bracketed character class

The following quantifiers are also recognized:

* `*`: Match 0 or more times
* `+`: Match 1 or more times
* `?`: Match 0 or 1 times
* `{n}`: Match exactly n-times
* `{n, }`: Match at least n-times
* `{ ,m}`: Match at most m-times
* `{n,m}`: Match n to m-times

The following escape sequences are recognized:

* `\t`: Tab (HT, TAB)
* `\n`: New Line (LF, NL)
* `**\r`: **Return (CR)
* `\f`: Form feed (FF)
* `\a`: Alarm bell
* `\xNN`:  Character whose ordinal number is the given hexadecimal number

These are the recognized character classes:
* `\w`: Match a _word _character (alphanumeric plus “_”)
* `\W`: Match a non-word character
* `\s`: Match a whitespace character
* `\S`: Match a non-whitespace character
* `**\d`: Match a decimal digit character
* `\D`: Match a non-digit character
* `\b`: Match a word boundary
* `\B`: Match except at a word boundary

## Sets of strings
If the event where you want a certain number of strings from a list to be hit, you can implement the following:

```bash
rule MigosPresent
{
    strings:
        $m1 = "Quavo"
        $m2 = "Offset"
        $m3 = "Takeoff"

    condition:
        2 of ($m1,$m2,$m3)
}
```
If any of the two Migos members are present, then the Migos are present.
You can also use wildcards to represent a set. Used this way, you would use the * wildcard.
```bash
rule MigosPresent
{
    strings:
        $m1 = "Quavo"
        $m2 = "Offset"
        $m3 = "Takeoff"

    condition:
        2 of ($m*)
}
```
To represent all variables in strings, you can use the them keyword.

```bash
rule ThreeRappersPresent
{
    strings:
        $m1 = "Quavo"
        $m2 = "Offset"
        $m3 = "Takeoff"
        $q1 = "Cardi B"

    condition:
        3 of them // equivalent to 3 of ($*)
}
```
Any expression that returns a numeric value can be used. Here is an example of the keywords any and **all **being used.
```bash
rule Squad
{
    strings:
        $m1 = "Quavo"
        $m2 = "Offset"
        $m3 = "Takeoff"
        $q1 = "Cardi B"

    condition:
        3 of them // equivalent to 3 of ($*)
        all of them
        any of ($*) and 2 of ($*)    // Fancy way of using any in a rule that requires 3.
}
```

## Anonymous strings with of and for…of
If the event where you are not specifically referencing strings, you can just use $ to reference them all.
```bash
rule AnonymousStrings
{
    strings:
        $ = "dummy1"
        $ = "dummy2"

    condition:
        1 of them
}
```
## Conditions
Yara allows for boolean expressions via the operators, and, or, and not and relational. Arithmetic operators (+,-,*,,%) and bitwise operators (&, |, <<, >>, ~, ^) can also be used on numerical expressions.
### Boolean
String identifiers can also be used within a condition, acting as a Boolean variables whose value depends on the presence or not of the associated string in a file.
```bash
rule Example
{
    strings:
        $hero1a = "Batman"
        $hero1b = "Robin"
        $hero2a = "Edward"
        $hero2b = "Alphonse"

    condition:
        ($hero1a or $hero1b) and ($hero2a or $hero2b)
}
```
## Counting string instances
Sometimes we need to know not only if a certain string is present or not, but how many times the string appears in the file or process memory. The number of occurrences of each string is represented by a variable whose name is the string identifier but with a # character in place of the $ character. For example:
```bash
rule Ransomware
{
    strings:
        $a = "encrypted"
        $b = "btc"

    condition:
        #a == 2 and #b > 2
}
```
This rule matches any file or process containing the string $a exactly two times, and more than two occurrences of string `$b`.

## String offsets or virtual addresses
In the majority of cases, when a string identifier is used in a condition, we are willing to know if the associated string is anywhere within the file or process memory, but sometimes we need to know if the string is at some specific offset of the file or at some virtual address within the process address space. In such situtations the operator at is what we need.
```bash
rule Offset
{
    strings:
        $a = "encrypted"
        $b = "btc"

    condition:
        $a at 100 and $b at 200
}
```
If string `$a` is found at offset 100 within the file (or at virtual address 100 if applied to a running process), it will catch. The string `$b` should also be at offset 200. You can also use hexadecimal instead of decimal notation.
```bash
rule Offset
{
    strings:
        $a = "encrypted"
        $b = "btc"

    condition:
        $a at 0x64 and $b at 0xC8
}
```
While the at operator is very specific, you can use the **in **operator to specify a range the string can be located at.
```bash
rule InExample
{
    strings:
        $a = "encrypted"
        $b = "btc"

    condition:
        $a in (0..100) and $b in (100..filesize)
}
```
String $a must be found at offset between 0-100, while string $b must be at an offset between 100 and the end of the file EOF.

You can also get the offset or virtual address of the i-th occurrence of string $a by using @a[ i ]. The indexes are one-based, so the first occurrence would be @a[1], the second being @a[2], and so on. It doesn't start at @a[0]. If you provide an index greater than the number of occurrences of the string, the result will be a NaN (Not a Number) value.

## Match Length
For many regular expressions and hex strings containing jumps, the length of the match is variable. If you have the regular expression /fo*/ the strings “fo”, “foo” and “fooo” can be matches, all of them with a different length.

You can use the length of the matches as part of your condition by using the character ! in front of the string identifier, in a similar way you use the @ character for the offset. !a[1] is the length for the first match of $a, !a[2] is the length for the second match, and so on. !a is a abbreviated form of !a[1].
```bash
rule Hak5
{
    strings:
        $re1 = /hack*/    // Will catch on hacker, hacked, hack, hack*

    condition:
        !re1[1] == 4 and !re1[2] > 6
}
```
This will catch the following:
```
We hack things. We are hackers.
```
The first instance of 'hack' is re1 and it's equal to length 4. the second instance of 'hack' has at least length 6.

## File size
String identifiers are not the only variables that can appear in the condition (in fact, rules can be defined without any string definition), there are other special variables that can be used as well. filesize holds the size of the file being scanned. The size is expressed in bytes.
```bash
rule FileSizeExample
{
    condition:
       filesize > 200KB
}
```
We use the KB postfix to set the size in which the file will be caught on to 200KB. It automatically multiples the value of the constant by 1024. The MB postfix can be used to multiply the value by 2^20. Both prefixes can be used only with decimal constants.

**[!] Important [!]** - filesize **only works when the rule is applied to a file. If applied to a running process, it won’t ever match.

## Executable entry_point
If the file is a **Portable Executable** (PE) or **Executable and Linkable Format** (ELF), this variable holds the raw offset of the executable’s entry point in case we are scanning a file. If we’re scanning a running process, the entry_point will hold the virtual address of the main executable’s entry point. _A typical use of this variable is to look for some pattern at the entry point to detect packers or simple file infectors. _The current way to use entry_point is by importing the lib for PE and/or ELF and use their respective functions. Yara’s entrypoint function is depreciated starting at version 3. This is how it looks pre-version 3.
```bash
rule EntryPointExample1
{
    strings:
        $a = { E8 00 00 00 00 }

    condition:
       $a at entrypoint
}

rule EntryPointExample2
{
    strings:
        $a = { 9C 50 66 A1 ?? ?? ?? 00 66 A9 ?? ?? 58 0F 85 }

    condition:
       $a in (entrypoint..entrypoint + 10)
}
```
**[!] Important [!]** Again, don’t use yara’s entrypoint. Import `PE` or `ELF` and use `pe.entry_point` and/or `elf.entry_point`.

## Accessing data at a given position
If you want to read data from a specific offset and save it as a variable you can use one of the following:
```c
int8(<offset or virtual address>)
int16(<offset or virtual address>)
int32(<offset or virtual address>)

uint8(<offset or virtual address>)
uint16(<offset or virtual address>)
uint32(<offset or virtual address>)

int8be(<offset or virtual address>)
int16be(<offset or virtual address>)
int32be(<offset or virtual address>)

uint8be(<offset or virtual address>)
uint16be(<offset or virtual address>)
uint32be(<offset or virtual address>)
```
Default is little-endian. If you want to read a big-endian integer use the corresponding function ending in be.

The `<offset or virtual address>` parameter can be any expression returning an unsigned integer, including the return value of one the uintXX functions itself.
```bash
rule IsPE
{
  condition:
     // MZ signature at offset 0 and ...
     uint16(0) == 0x5A4D and
     // ... PE signature at offset stored in MZ header at 0x3C
     uint32(uint32(0x3C)) == 0x00004550
}
```
## for…of: Applying one condition across many strings
The **boolean_expression** is evaluated for every string in string_set and there must be at least num of them true.
One can also exchange num with other keywords such as **all or any**.
```bash
for any of ($a,$b,$c) : ( $ at elf.entry_point  )
```
The `$` represents all of the strings in the set. In this example, it’s strings `$a`, `$b`, and `$c`.

You can also employ the symbols `#` and `@` to make reference to the number of occurrences and the first offset of each string.
```bash
for all of them : ( # > 3 )
for all of ($a*) : ( @ > @b )
```
## Iterating over string occurrences
If you want to iterate over offsets and test a condition, one can do the following:
```bash
rule Three_Peat
{
    strings:
        $a = "dummy1"
        $b = "dummy2"

    condition:
        for all i in (1,2,3) : ( @a[i] + 10 == @b[i] )
}
```
This rule says that the first three occurrences of $b should be 10 bytes away from the first three occurrences of $a. Another way to write this is the following:
```bash
for all i in (1..3) : ( @a[i] + 10 == @b[i] )
```
We can also use expression as well. In this example, we are iterating over every occurrence of `$a` (remember that `#a` represents the number of occurrences of `$a`). This rule is specifying that every occurrence of `$a` should be within the first 100 bytes of the file.
```bash
for all i in (1..#a) : ( @a[i] < 100 )
```
You can also set it so it’s a set amount of occurrence for the first 100 bytes.
```bash
for any i in (1..#a) : ( @a[i] < 100 )
for 2 i in (1..#a) : ( @a[i] < 100 )
```
## Referencing other rules
Just like in C when referencing functions, the function, or in this case the rule, must be defined prior to being used.
```bash
rule Rule1
{
    strings:
        $a = "dummy1"

    condition:
        $a
}

rule Rule2
{
    strings:
        $a = "dummy2"

    condition:
        $a and Rule1
}
```
## Yara Essentials
### Global Rules
Allows users to impose restrictions in all the rules. If you want all your rules to ignore the files that exceed a certain size limit, you could go rule by rule making the required modifications to their conditions, or just write a global rule like this one:
```bash
global rule SizeLimit
{
    condition:
        filesize < 2MB
}
```
You can define as many global rules as you want. They’ll run before the other rules.

### Private Rules
Private rules don’t have an output when they match. When paired with referencing other rules, this can allow for a cleaner output. Such that, to get to superMalicious, maybe one private rule is that file must be ELF. Once that is confirmed, then the next rule will execute. But we don’t want to see ELF, in output. We just want to know if it’s superMalicious or not. To create a private rule, just add private in front of rule.
```bash
private rule PrivateRule
{
    ...
}
```

### Rule tags
You can tag your rules in case you only want to see the output of type ruleName.
```bash
rule TagsExample1 : Foo Bar Baz
{
    ...
}

rule TagsExample2 : Bar
{
    ...
}
```
### Metadata
This allows for additional data to be stored in a rule.
```bash
rule MetadataExample
{
    meta:
        my_identifier_1 = "Some string data"
        my_identifier_2 = 24
        my_identifier_3 = true

    strings:
        $my_text_string = "text here"
        $my_hex_string = { E2 34 A1 C8 23 FB }

    condition:
        $my_text_string or $my_hex_string
}
```

### Using Modules
Some modules are officially distributed with YARA like PE and Cuckoo. They can be imported just like python, but add double quotes.
```python
import "pe"
import "cuckoo"
```
Once imported, you can use the feature by using its name prior to the function.
```python
pe.entry_point == 0x1000
cuckoo.http_request(/someregexp/)
```
### Undefined Values
Some values are left as undefined when they are ran. If the following rule executes on a file that’s of type ELF but it finds the string, it will result in something like TRUE & Undefined.
```python
import "pe"

rule Test
{
  strings:
      $a = "some string"

  condition:
      $a and pe.entry_point == 0x1000
}
```
Be careful.

### External Variables
External variables allow you to define rules which depend on values provided from ‘the other side’.
```bash
rule ExternalVariable1
{
    condition:
       ext_var == 10
}
```
ext_var is an external variable whos value is assigned at runtime, (use -d on the command line and parameter of **compile** and match methods in yara-python). External variables could be of types: int, str, or boolean.

External variables can be used with the operators: contains and **matches**. Contains returns true if the string contains the specified substring. **Matches** returns true if the string matches the given **regular expression**.
```bash
rule ExternalVariable2
{
    condition:
        string_ext_var contains "text"
}

rule ExternalVariable3
{
    condition:
        string_ext_var matches /[a-z]+/
}
```
**Contains **is True for ExternalVariable2 and matches is True for ExternalVariable3

You can also use regex modifiers along with the matches operator.
```bash
rule ExternalVariableExample5
{
    condition:
        /* case insensitive single-line mode */
        string_ext_var matches /[a-z]+/is
}
```
This will match for case-insensitive due to the i.

Remember, you must define all external variables at run-time. This can be done with the **-d **argument.

### Including files
Of course, you can include other files in yara, using the C-type import, #include…but without the # and with double quotes. You can use relative paths, absolute paths, and if windows, paths with drives.
```python
include "Migos.yar"
include "../CardiB.yar"
include "/home/user/yara/IsRapper.yar"
include "c:\\yara\\includes\\oldRappers.yar"
include "c://yara/includes/oldRappers.yar"
```
# Conclusions
Alright, now you know how to write some Yara Rules.
Here’s some malware repos, rules, and tools that allow you to generate yara rules. If you install yarGen, just point it at the malware, and it will the write a signature for that malware. If you want to catch a family of malware, it’s better to generalize it across the entire family.

## Resources

### Worm Descriptions
- [F-Secure: Worm W32 Downadup AL](https://www.f-secure.com/v-descs/worm_w32_downadup_al.shtml)
- [F-Secure: Worm W32 Downadup](https://www.f-secure.com/v-descs/worm_w32_downadup.shtml)
- [Microsoft Support: Win32 Conficker Worm Alert](https://support.microsoft.com/en-us/help/962007/virus-alert-about-the-win32-conficker-worm)
- [F-Secure: Worm W32 Downadup A](https://www.f-secure.com/v-descs/worm_w32_downadup_a.shtml)
- [F-Secure: Worm W32 Downadup Gen](https://www.f-secure.com/v-descs/worm_w32_downadup_gen.shtml)
- [F-Secure: Worm W32 DownadupRun A](https://www.f-secure.com/v-descs/worm_w32_downaduprun_a.shtml)

### Yara
- [Experts Exchange: How to Test Yara Rules](https://www.experts-exchange.com/questions/29042297/How-to-test-yara-rule.html)
- [Security Artwork: Yara 101](https://www.securityartwork.es/2013/10/11/yara-101/)
- [STIX Project: Yara Test Mechanism](https://stixproject.github.io/documentation/idioms/yara-test-mechanism/)
- [yarGen by Neo23x0](https://github.com/Neo23x0/yarGen)
- [Radare2 Yara Documentation](https://github.com/radare/radare2/blob/master/doc/yara.md)
- [BSK Consulting: Writing Simple Yara Rules - Part 1](https://www.bsk-consulting.de/2015/02/16/write-simple-sound-yara-rules/)
- [BSK Consulting: Writing Simple Yara Rules - Part 2](https://www.bsk-consulting.de/2015/10/17/how-to-write-simple-but-sound-yara-rules-part-2/)
- [BSK Consulting: Writing Simple Yara Rules - Part 3](https://www.bsk-consulting.de/2016/04/15/how-to-write-simple-but-sound-yara-rules-part-3/)

### xxd
- [Linux Man Pages: xxd](https://www.systutorials.com/docs/linux/man/1-xxd/)

### Malware repos
- [MalShare](https://github.com/Malshare/MalShare-Toolkit.git)

### Malware Submission
- [VirusTotal](https://www.virustotal.com/)







