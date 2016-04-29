# Epitran

A library and tool for transliterating orthographic text as IPA (International Phonetic Alphabet).

## Usage

The principle script for transliterating orthographic text as IPA is `epitranscriber.py`. It takes one argument, the ISO 639-3 code for the language of the orthographic text, takes orthographic text at standard in and writes Unicode IPA to standard out.

```
$ echo "Düğün olur bayram gelir" | epitranscribe.py "tur-Latn" dyɰyn oluɾ bajɾam ɟeliɾ
$ epitranscribe.py "tur-Latn" < orthography.txt > phonetic.txt
```

Additionally, the small Python modules ```epitran``` and ```epitran.vector``` can be used to easily write more sophisticated Python programs for deploying the **Epitran** mapping tables. This is documented below.

## Using the `epitran` Module

The functionality in the `epitran` module is encapsulated in the very simple `Epitran` class. Its constructor takes one argument, `code`, the ISO 639-3 code of the language to be transliterated plus a hyphen plus a four letter code for the script (e.g. 'Latn' for Latin script, 'Cyrl' for Cyrillic script, and 'Arab' for a Person-Arabic script).

```
>>> import epitran
>>> epi = epitran.Epitran('tur-Latn')
```

The `Epitran` class has only a few "public" method (to the extent that such a concept exists in Python). The most important are ``transliterate`` and ``word_to_tuples``:

**transliterate**(text):
Convert `text` (in Unicode-encoded orthography of the language specified in the constructor) to IPA, which is returned.

```
>>> epi.transliterate(u'Düğün')
u'd\xfc\u011f\xfcn'
>>> print(epi.transliterate(u'Düğün'))
düğün
```

**word_to_tuples**(word):
Takes a `word` (a Unicode string) in a supported orthography as input and returns a list of tuples with each tuple corresponding to an IPA segment of the word. The tuples have the following structure:
```
(
    character_category :: String,
    is_upper :: Integer,
    orthographic_form :: Unicode String,
    phonetic_form :: Unicode String,
    segments :: List<Tuples>
)
```
The codes for `character_category` are from the initial characters of the two character sequences listed in the "General Category" codes found in [Chapter 4 of the Unicode Standard](http://www.unicode.org/versions/Unicode8.0.0/ch04.pdf#G134153). For example, "L" corresponds to letters and "P" corresponds to production marks. The above data structure is likely to change in subsequent versions of the library. The structure of ```segments``` is as follows:
```
(
    segment :: Unicode String,
    vector :: List<Integer>
)
```

Here is an example of an interaction with ```word_to_tuples```:
```
>>> import epitran
>>> epi = epitran.Epitran('uzb-Latn')
>>> epi.word_to_tuples(u'Düğün')
[(u'L', 1, u'D', u'd\u032a', [(u'd\u032a', [-1, -1, 1, -1, -1, -1, -1, -1, 1, -1, -1, 1, 1, 1, -1, -1, -1, -1, -1, 0, -1])]), (u'L', 0, u'u', u'u', [(u'u', [1, 1, -1, 1, -1, -1, -1, 0, 1, -1, -1, -1, -1, -1, 1, 1, -1, 1, 1, 1, -1])]), (u'M', 0, u'\u0308', u'', [(-1, [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0])]), (u'L', 0, u'g', u'\u0261', [(u'\u0261', [-1, -1, 1, -1, -1, -1, -1, 0, 1, -1, -1, -1, -1, 0, -1, 1, -1, 1, -1, 0, -1])]), (u'M', 0, u'\u0306', u'', [(-1, [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0])]), (u'L', 0, u'u', u'u', [(u'u', [1, 1, -1, 1, -1, -1, -1, 0, 1, -1, -1, -1, -1, -1, 1, 1, -1, 1, 1, 1, -1])]), (u'M', 0, u'\u0308', u'', [(-1, [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0])]), (u'L', 0, u'n', u'n', [(u'n', [-1, 1, 1, -1, -1, -1, 1, -1, 1, -1, -1, 1, 1, -1, -1, -1, -1, -1, -1, 0, -1])])]
```

## Using the ```epitran.vector``` Module

The ```epitran.vector``` module is also very simple. It contains one class, ```VectorsWithIPASpace```, including one method of interest, ```word_to_segs```. The constructor for ```VectorsWithIPASpace``` takes two arguments:
- ```code```: the language-script code for the language to be processed.
- ```space```: the code for the punctuation/symbol/IPA space in which the characters/segments from the data are expected to reside. The available spaces are listed [below](#language-support).

A typical interaction with the ```VectorsWithIPASpace``` object is illustrated here:
```
>>> import epitran.vector
>>> vwis = epitran.vector.VectorsWithIPASpace('uzb-Latn', 'uzb-with_attached_suffixes-space')
>>> vwis.word_to_segs(u'darë')
[(u'L', 0, u'd', u'd\u032a', u'40', [-1, -1, 1, -1, -1, -1, -1, -1, 1, -1, -1, 1, 1, 1, -1, -1, -1, -1, -1, 0, -1]), (u'L', 0, u'a', u'a', u'37', [1, 1, -1, 1, -1, -1, -1, 0, 1, -1, -1, -1, -1, -1, -1, -1, 1, 1, -1, 1, -1]), (u'L', 0, u'r', u'r', u'54', [-1, 1, 1, 1, 0, -1, -1, -1, 1, -1, -1, 1, 1, -1, -1, 0, 0, 0, -1, 0, -1]), (u'L', 0, u'e\u0308', u'ja', u'46', [-1, 1, -1, 1, -1, -1, -1, 0, 1, -1, -1, -1, -1, 0, -1, 1, -1, -1, -1, 0, -1]), (u'L', 0, u'e\u0308', u'ja', u'37', [1, 1, -1, 1, -1, -1, -1, 0, 1, -1, -1, -1, -1, -1, -1, -1, 1, 1, -1, 1, -1])]
```
(It is important to note that, though the word that serves as input--*darë*--has four letters, the output contains four tuples because the last letter in *darë* actually corresponds to two IPA segments, /j/ and /a/.) The returned data structure is a list of tuples, each with the following structure:

```
(
    character_category :: String,
    is_upper :: Integer,
    orthographic_form :: Unicode String,
    phonetic_form :: Unicode String,
    in_ipa_punc_space :: Integer,
    phonological_feature_vector :: List<Integer>
)
```

A few notes are in order regarding this data structure:
- ```character_category``` is defined as part of the Unicode standard ([Chapter 4](http://www.unicode.org/versions/Unicode8.0.0/ch04.pdf#G134153)). It consists of a single, uppercase letter from the set {'L', 'M', 'N', 'P', 'S', 'Z', 'C'}.. The most frequent of these are 'L' (letter), 'N' (number), 'P' (punctuation), and 'Z' (separator [including separating white space]).
- ```is_upper``` consists only of integers from the set {0, 1}, with 0 indicating lowercase and 1 indicating uppercase.
- The integer in ```in_ipa_punc_space``` is an index to a list of known characters/segments such that, barring degenerate cases, each character or segment is assignmed a unique and globally consistant number. In cases where a character is encountered which is not in the known space, this field has the value -1.
- The length of the list ```phonological_feature_vector``` should be constant for any instantiation of the class (it is based on the number of features defined in panphon) but is--in principles--variable. The integers in this list are drawn from the set {-1, 0, 1}, with -1 corresponding to '-', 0 corresponding to '0', and 1 corresponding to '+'. For characters with no IPA equivalent, all values in the list are 0.


## <a name='language-support'></a>Language Support

### Transliteration Languages

| Code     | Language              |
|----------|-----------------------|
| hau-Latn | Hausa                 |
| ind-Latn | Indonesian            |
| jav-Latn | Javanese              |
| kaz-Cyrl | Kazakh (Cyrillic)     |
| kaz-Latn | Kazakh (Latin)        |
| kir-Arab | Kyrgyz (Perso-Arabic) |
| kir-Cyrl | Kyrgyz (Cyrillic)     |
| kir-Latn | Kyrgyz (Latin)        |
| tuk-Cyrl | Turkmen (Cyrillic)    |
| tuk-Latn | Turkmen (Latin)       |
| tur-Latn | Turkish (Latin)       |
| yor-Latn | Yoruba                |
| uig-Arab | Uyghur (Perso-Arabic) |
| uzb-Cyrl | Uzbek (Cyrillic)      |
| uzb-Latn | Uzbek (Latin)         |

### Language "Spaces"

| Code                                | Language | Note                                 |
|-------------------------------------|----------|--------------------------------------|
| tur-with_attached_suffixes-space    | Turkish  | Based on data with suffixes attached |
| tur-without_attached_suffixes-space | Turkish  | Based on data with suffixes removed  |
| uzb-with_attached_suffixes-space    | Uzbek    | Based on data with suffixes attached |
