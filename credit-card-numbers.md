
正規表示法: 信用卡號文章說明
==========================
這篇文章保留了原文，原始出處來自 http://www.richardsramblings.com/regex/credit-card-numbers/
為了避免該站不見了，故保留原文備用

----------


Regex: Credit Card Numbers
==========================
No pure software solution can identify all credit card numbers with perfect accuracy. No regular expression can do it. No algorithm, no matter how sophisticated, can do it. Solely financial network and payment gateways provide the greatest assurance of valid account numbers and even their databases may be 30 days out of date. Only after you accept that should you move forward with home-spun validation.

To make matters worse, many international credit or debit cards are issued under dual banking systems. China Construction Bank issues a joint China UnionPay and Japan Credit Bureau card under IIN 356895, generally classified as a JCB account number. The same bank issues a joint China UnionPay and Discover Network card under IIN 622286, which falls into the UnionPay network. The Bank of Beijing issues a dual VISA UnionPay debit card under IIN 602969, a bucket not belonging to any of the major financial networks.

Using the wrong regular expression can be pointless, aggravating, or — in the worst cases — disastrous. Be sure to first read [The Perfect Credit Card Number RegEx](the-perfect-credit-card-number-regex.md) to understand how to use different types of regular expressions, and why none of these regular expressions may be suited for your purpose.

## Single Card Types ##

### VISA Cards ###

VISA account numbers start with a “4″. According to VISA’s developer API documentation, valid account numbers range in length from 13 to 19 digits, and their hardware transaction device specifications require support for PANs with as few as 12 digits. However, due to the overwhelming prevalence of 16-digits PANs, in-depth coverage of any lengths other than 16 digits are purposefully omitted. For supporting other card number lengths, review other bank card types.

- The most common input-validation regex for VISA card numbers needs only allow numbers with 16 digits. No spaces or dashes are allowed, e.g. “4012888888881881″.
~~~
    ^4\d{15}$
     
    <!-- Assert starting position is the beginning of the string or line. Match the number "4". Match on 15 other digits (0..9). Assert position is the end of the string or line. -->
~~~

- A best-practice input-validation regex for VISA card numbers that allows for optional matching delimiters of spaces or dashes between number groups, e.g. “4012-8888-8888-1881″. You will need to strip out the spaces or dashes prior to verifying the PAN through VISA payment gateway.
~~~
    ^4\d{3}([\ \-]?)\d{4}\1\d{4}\1\d{4}$
     
    <!-- Assert starting position is the beginning of the string or line. Match the number "4". Match on 3 other digits (0..9). Match on a space or dash ("the delimiter") or nothing if neither is present. Match on 4 digits. Optionally match on the same delimiter as before, if any. Match on 4 more digits. Optionally match on the delimiter. Match on 4 more digits. Assert position is the end of the string or line. -->
~~~

- Despite the fact that I haven’t personally seen a 13-digit VISA card in decades, if you truly need support for 13-digit PANs, use the following input-validation regex for 13or 16-digit VISA card numbers that allows for optional matching delimiters of spaces or dashes between number groups, e.g. “4012-8888-8888-1″.
~~~
    ^4\d{3}([\ \-]?)(?:\d{4}\1){2}\d(?:\d{3})?$
~~~

- A data-mask regex for scrubbing 16-digit VISA card numbers with optional spaces or dashes as matching delimiters between the number groups. Certain repetitive and sequential groups common to many false CCNs are ignored, impacting approximately 0.34% of potentially valid numbers. The second scrubbing regex contains fewer exceptions, matching potentially more false data at the expense of decreased masking.
~~~
    ^4(\d{3})(?!\1{3})([\ \-]?)(?!(\d)\3{3})(\d{4})\2(?!\4|(\d)\5{3}|1234|2345|3456|5678|7890)(\d{4})\2(?!\6|(\d)\7{3}|1234|3456)\d{4}$
     
    <!-- Assert starting position is the beginning of the string or line. Match the number "4". Match on 3 other digits (0..9). Assert that the next 9 digits cannot be three groups of three digits that are identical to the previous group of three digits (e.g. "123123123"). Match on a space or dash ("the delimiter"), if either present. Assert that there are three more groups of four digits ("groups 2, 3, and 4"), each separated by the last with the same delimiter as the one previously matched, if any. Assert that none of the last three groups of four have identical digits (e.g. "9999"). Assert that the 3rd group can not be "2345", "5678", or "7890". Assert that neither the 3rd or 4th groups can be "1234" or "3456". Assert that the 3rd group cannot match the 2nd group. Assert that the 4th group cannot match the 3rd group. Assert position is the end of the string or line. -->
~~~
~~~
    ^4(\d{3})(?!\1{3})([\ \-]?)(\d{4})\2(?!\3)(\d{4})\2(?!\4|(\d)\5{3})\d{4}$
     
    <!-- Assert starting position is the beginning of the string or line. Match the number "4". Match on 3 other digits (0..9). Assert that the next 9 digits cannot be three groups of three digits that are identical to the previous group of three digits (e.g. "123123123"). Match on a space or dash ("the delimiter"), if either present. Assert that there are three more groups of four digits ("groups 2, 3, and 4"), each separated by the last with the same delimiter as the one previously matched, if any. Assert that the 3rd group cannot match the 2nd group. Assert that the 4th group cannot match the 3rd group. Assert that the last group of four does not have identical digits (e.g. "9999"). Assert position is the end of the string or line. -->
~~~

- Content-inspection regular expressions for 16-digit VISA card numbers with optional spaces or dashes as matching delimiters between the number sections, and also with the same general ruleset as the one above. What makes thus a bold regex is the restrictions based on surrounding characters.
~~~
    \b(?<!\-|\.)4(\d{3})(?!\1{3})([\ \-]?)(?<!\d\ \d{4}\ )(?!(\d)\3{3})(\d{4})\2(?!\4|(\d)\5{3}|1234|2345|3456|5678|7890)(\d{4})(?!\ \d{4}\ \d)\2(?!\6|(\d)\7{3}|1234|3456)\d{4}(?!\-)(?!\.\d)\b
     
    <!-- Assert starting position is at a word boundary. Assert that the previous character is not a period or dash. Match the number "4". Match on 3 other digits (0..9). Assert that the next 9 digits cannot be three groups of three digits that are identical to the previous group of three digits. Match on a space or dash ("the delimiter"), if either present. Assert that the previous seven characters are not a digit, a space, four digits, and then a space. Match four digits and the previously seen delimiter, if any. Assert that the 3rd group cannot match the 2nd group. Assert that the next four digits are not identical. Assert that the 3rd group can not be "1234", "2345", "3456', "5678", or "7890". Match on four more digits. Assert that the next  six characters are not a space, four digits and a space. Match the delimiter. Assert that the 4th group cannot match the 3rd group. Assert that the last group of four does not have identical digits. Assert that the 4th group cannot be "1234" or "3456". Assert the next character is not a dash. Assert that the next two characters are not a period followed by a number. Assert ending position is at a word boundary.
~~~

### MasterCard ###

MasterCard account numbers start with prefixes ranging from “51″ to “55″, and are 16 digits in length.

- A basic input-validation regex for 16-digit MasterCard account numbers. No spaces or dashes are allowed, e.g. “5111005111051128″.
~~~
    ^5[1-5]\d{14}$
~~~

- For each of the VISA regexes above that contains “4(\d{3})” near the beginning, substitute with “5([1-5]\d{2})” for an equivalent MasterCard regular expression. For regexes without capturing groups, substitute “4\d{3}” with “5[1-5]\d{2}” as appropriate. All of the above VISA regexes can be transformed into MasterCard regexes using this substitution method. For example, input validation for MasterCard account numbers with optional matching delimiters of spaces or dashes between number groups mirrors VISA validation:
~~~
    ^4\d{3}([\ \-]?)\d{4}\1\d{4}\1\d{4}$
        becomes
    ^5[1-5]\d{2}([\ \-]?)\d{4}\1\d{4}\1\d{4}$
~~~

### Discover Card ###

The first six digits of any credit or debit card identify the issuing authority or bank. According to the Discover Network’s developer documentation, Discover Card’s issuer identification numbers start with 6011, 622126-622925 (Discover cards in this range are dual-branded with UnionPay), 644-649, or 65.

- Not as intuitively simple as VISA or MasterCard regexes due to the full-length IINs, an input-validation regex for 16-digit Discover card numbers needs to account for the range of six-digit issuer identifiers. No spaces or dashes are allowed, e.g. “6011000990139424″.
~~~
    ^6(?:011\d\d|5\d{4}|4[4-9]\d{3}|22(?:1(?:2[6-9]|[3-9]\d)|[2-8]\d\d|9(?:[01]\d|2[0-5])))\d{10}$
~~~

- Discover Card regexes can be derived from each of the VISA regexes above that contains “4(\d{3})” by substituting “6(011|22(?:1(?=[\ \-]?(?:2[6-9]|[3-9]))|[2-8]|9(?=[\ \-]?(?:[01]|2[0-5])))|4[4-9]\d|5\d\d)”. For regexes without capturing groups, substitute “4\d{3}” with “6(?:011|22(?:1(?=[\ \-]?(?:2[6-9]|[3-9]))|[2-8]|9(?=[\ \-]?(?:[01]|2[0-5])))|4[4-9]\d|5\d\d)” as appropriate. For example, input validation for Discover Cards with optional matching delimiters of spaces or dashes between number group, starting with the VISA regex:
~~~
    ^4\d{3}([\ \-]?)\d{4}\1\d{4}\1\d{4}$
        becomes
    ^6(?:011|22(?:1(?=[\ \-]?(?:2[6-9]|[3-9]))|[2-8]|9(?=[\ \-]?(?:[01]|2[0-5])))|4[4-9]\d|5\d\d)([\ \-]?)\d{4}\1\d{4}\1\d{4}$
~~~

### Japan Credit Bureau ###

Japan Credit Bureau (JCB) account numbers have IIN prefixes ranging from “3528″ to “3589″.

- A basic input-validation regex for 16-digit JCB account numbers. No spaces or dashes are allowed, e.g. “3566002020360505″.
~~~
    ^35(?:2[89]|[3-8]\d)\d{12}$
~~~
- For each of the VISA regexes above that contains “4(\d{3})” near the beginning, substitute with “3(5(?:2[89]|[3-8]\d))” for an equivalent JCB regular expression. For regexes without capturing groups, substitute “4\d{3}” with “35(?:2[89]|[3-8]\d)” as appropriate. All of the above VISA regexes can be transformed into Japan Credit Bureau regexes using this substitution method. For example, input validation for JCB card numbers with optional matching delimiters of spaces or dashes between number groups mirrors VISA validation:
~~~
    ^4\d{3}([\ \-]?)\d{4}\1\d{4}\1\d{4}$
        becomes
    ^35(?:2[89]|[3-8]\d)([\ \-]?)\d{4}\1\d{4}\1\d{4}$
~~~

### American Express ###

American Express credit card account numbers are 15 digits in lengths, and generally start with either “34″ or “37″.

- An input-validation regex for 15-digit American Express card numbers, e.g. “371449635398431″.
~~~
    ^3[47]\d{13}$
     
    <!-- Assert starting position is the beginning of the string or line. Match the number "3". Match on a number "4" or "7". Match on 13 other digits (0..9). Assert position is the end of the string or line. -->
~~~

- A best-practice input-validation regex for AmEx card numbers that allows for optional matching delimiters of spaces or dashes between number groups, e.g. “3714-496353-98431″.
~~~
    ^3[47]\d\d([\ \-]?)\d{6}\1\d{5}$
     
    <!-- Assert starting position is the beginning of the string or line. Match the number "3". Match on a number "4" or "7". Match on 2 other digits (0..9). Match on a space or dash ("the delimiter") or nothing if neither is present. Match on 6 digits. Optionally match on the same delimiter as before, if any. Match on 5 more digits. Assert position is the end of the string or line. -->
~~~

- A data-mask regex for scrubbing 15-digit American Express card numbers with optional spaces or dashes as matching delimiters between the three number groups. Certain repetitive and sequential groups common to many test CCNs are ignored. The sequential number groups are probably large enough not to warrant a less-aggressive scrubber.
~~~
    ^3[47]\d\d([\ \-]?)(?!(\d)\2{5}|123456|234567|345678)\d{6}\1(?!(\d)\3{4}|12345|56789)\d{5}$
~~~

- A content-inspection regular expression with the same rules as the data-mask regex above, plus additional restrictions on surrounding characters.
~~~
    \b(?<!\-|\.)3[47]\d\d([\ \-]?)(?<!\d\ \d{4}\ )(?!(\d)\2{5}|123456|234567|345678)\d{6}(?!\ \d{5}\ \d)\1(?!(\d)\3{4}|12345|56789)\d{5}(?!\-)(?!\.\d)\b
~~~

### China UnionPay ###

According to recent statistics, almost every Chinese citizen in China (including Hong Kong and Macau) has at least one UnionPay card. There are approximately 3.1 billion UnionPay credit and debit cards issued worldwide by more than 250 international and Chinese domestic member banks. Excluding cards issued under dual networks, most China UnionPay card number have prefixes from “620″ through “625″, and range in length from 16 to 19 characters.

Form-input validation or data masking of variable-length card numbers is nearly as simple and effective as fixed-length numbers, but variable-length numbers pose significant false-positive issues when performing free-form inspection or discovery.

- A basic input-validation regex for 16- to 19-digit UnionPay account numbers. No spaces or dashes are allowed, e.g. “6212341111111111111″.
~~~
    ^62[0-5]\d{13,16}$
~~~

### Maestro ###

Maestro card numbers have several prefixes, including 50, 56-58, 6390, and 67. They are frequently between 16 and 19 digits in length, but are allowed to have as few as 12 digits. Since 2009, all new Laser debit cards (an Irish financial network) have been dual branded with Maestro, so the 6304 Laser prefix is bundled into the Maestro regex.

- A basic input-validation regex for 12- to 19-digit Maestro card numbers. No spaces or dashes are allowed, e.g. “5019717010103742″.
~~~
    ^(?:5[0678]\d\d|6304|6390|67\d\d)\d{8,15}$
~~~

### Diner’s Club International ###

According to Discover Network, due to an alliance with Discover, MasterCard, and Diner’s Club, as of October 2009, the IIN ranges previously used by Diner’s Club (300-305, 3095, 36, 38-39) have been retired and are “for development purposes only”. Any current Diner’s Club account numbers have been reissued from number ranges assigned to Discover. As such, these numbers no longer require the same level of protection as other account numbers, so I am no longer supporting regexes for DCI numbers.

## Multiple Card Types ##
### Visa, MasterCard, American Express, and Discover Cards ###

- The most basic input-validation regex for the most common 16-digit card numbers and 15-digit AmEx cards. Older 13-digit card numbers are omitted. No spaces or dashes are allowed, e.g. “4012888888881881″ or “378282246310005″.
~~~
    \b(?:3[47]\d|(?:4\d|5[1-5]|65)\d{2}|6011)\d{12}\b
~~~

- The same as above, but with optional spaces or dashes as matching delimiters between the most common number sections for the card type, e.g. “4012-8888-8888-1881″ or “3782 822463 10005″.
~~~
    \b(?:3[47]\d{2}([\ \-]?)\d{6}\1\d|(?:(?:4\d|5[1-5]|65)\d{2}|6011)([\ \-]?)\d{4}\2\d{4}\2)\d{4}\b
~~~

### The Kitchen Sink ###

This complicated content-inspection regular expression matches optionally delimited 15-digit American Express numbers and 16-digit VISA, MasterCard, Discover, and Japan Credit Bureau card numbers. China UnionPay and Maestro are not included. It includes much of the filtering from the above scrubbing and filtering regexes with a few minor modifications required to combine the rules while maintaining the overall flavor of functionality. This is far too complex to explain each expression token in depth, so you’re on your own in deciphering or modifying this behemoth.
~~~
\b(?<!\-|\.)(?:(?:(?:4\d|5[1-5]|65)(\d\d)(?!\1{3})|35(?:2[89]|[3-8]\d)|6(?:011|4[4-9]\d|22(?:1(?!1\d|2[1-5])|[2-8]|9(?=1\d|2[1-5]))))([\ \-]?)(?<!\d\ \d{4}\ )(?!(\d)\3{3})(\d{4})\2(?!\4|(\d)\5{3}|1234|2345|3456|5678|7890)(\d{4})(?!\ \d{4}\ \d)\2(?!\6|(\d)\7{3}|1234|3456)|3[47]\d{2}([\ \-]?)(?<!\d\ \d{4}\ )(?!(\d)\9{5}|123456|234567|345678)\d{6}(?!\ \d{5}\ \d)\8(?!(\d)\10{4}|12345|56789|67890)\d)\d{4}(?!\-)(?!\.\d)\b
~~~

> **Author’s Note**: The regex above contains ten capturing groups. Some regex engines limit the number of capturing groups to nine or fewer; attempting to reference the tenth capturing group (“\10″) with such an engine may be split and interpreted instead as: Match the data captured in the first group (“\1″), followed by the number zero (“0″). While not catastrophic in this case, the behavior is generally undesirable.

### Modifying the Kitchen Sink ###

If you want to add support for more two-digit prefixes for 16-digit card numbers, add additional alternatives within “(?:4\d|5[1-5]|65)” near the start of the regex, .e.g. “(?:4\d|5[1-5]|62|65)” to also look for cards starting with “62″. To add more four-digit prefixes for 16-digit numbers, add alternatives within “))))”, e.g. “)))|7789)” to include numbers starting with “7789″. To exclude more four-digit suffixes, add alternatives within “(?!\6|(\d)\7{3}|1234|3456)” near the middle, .e.g. “(?!\6|(\d)\7{3}|1234|3456|6789)” to skip card numbers ending in “6789″.

The even longer regex below is the same as the behemoth above, plus extremely basic China UnionPay and Maestro support; validation for the two networks is limited to 19-digit non-delimited card numbers ending in four non-repeating digits. Additional restrictions on surrounding characters have been added. I do not recommend using this for content inspection or discovery as I expect the CUP and Maestro numbers to generate too many false positives; it is included only to demonstrate how and where to make some of the modifications.
~~~
\b(?<![\$\&\+\_\--\/\<\>\?])(?:(?:(?:4\d|5[1-5]|65)(\d\d)(?!\1{3})|35(?:2[89]|[3-8]\d)|6(?:011|4[4-9]\d|22(?:1(?!1\d|2[1-5])|[2-8]|9(?=1\d|2[1-5]))))([\ \-]?)(?<!\d\ \d{4}\ )(?!(\d)\3{3})(\d{4})\2(?!\4|(\d)\5{3}|1234|2345|3456|5678|7890)(\d{4})(?!\ \d{4}\ \d)\2(?!\6|(\d)\7{3}|1234|3456)|3[47]\d{2}([\ \-]?)(?<!\d\ \d{4}\ )(?!(\d)\9{5}|123456|234567|345678)\d{6}(?!\ \d{5}\ \d)\8(?!(\d)\10{4}|12345|56789|67890)\d|(?:(?:5[0678]|6[27])\d\d|6304|6390)\d{11}(?!(\d)\11{3}))\d{4}(?![\$\&\+\_\-\/\<\>])(?![\.\?]\d)\b
~~~

## Wrapping It All Up ##

These regular expressions are designed for matching credit and debit cards — gift cards, SIM cards, and loyalty or reward cards are intentionally not considered. That said, similar techniques used to match debit and credit cards can be used in matching similarly formatted account numbers outside the standard IIN buckets. For example, the Russian supermarket chain Перекресток issues loyalty cards with 16-digit account numbers that begin with 778900. The following regex format should look quite familiar by now:
~~~
^778900\d{10}$
~~~

Lastly, don’t take my word for anything (or anyone else’s) regarding regular expressions. If you don’t understand regexes and how and when to use them, if you can’t break them apart into their components and comprehend them, don’t use them at all.

> **Author’s Note**: I spend considerable time writing these articles not so you can copy/paste my hard work into your code, but so you can study the techniques and gain a deeper understanding of how regular expressions work.
> 
> If you use my regexes in your commercial product, please give credit where credit is due, minimally by name, optionally accompanied by a link to or URL of this page. I wouldn’t refuse a free copy/license of your product, either. If you otherwise find the fruits of my labor useful in your personal or professional daily life, please make a donation using the button at the bottom of the page. Just please don’t profit from the end result of my long, hard work without even a “Thank You!”