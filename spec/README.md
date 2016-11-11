# Diacritics Database Specification

> File specification for the diacritics database

## 1. What Information Must Be Collected?

Since there's no trustworthy and complete source, it's necessary to collect all diacritics mapping information manually. It's also necessary to collect meta information for each language such as links to sources documenting the diacritics.

## 2. Master Branch

The master branch is used as the branch for source files.

### 2.1 Project Structure

The structure of this branch is divided into the following folders:

```
database/
├── spec/
└── src/
```

#### 2.1.1 `spec/` Folder

Used to specify the structure.

#### 2.1.2 `src/` Folder

The structure in the `src/` folder can look like:

```
src/
├── de/
│   └── de.js
└── fr/
    └── fr.js
```

If there are language variants it could look like:

```
src/
├── de/
│   ├── de.js
│   ├── at.js
│   └── ch.js
└── fr/
    └── fr.js
```

- As there might be multiple files for a language, each language has its own folder
- The language files in this folder will be in a `.js` format to allow comments and to make sure text editors allow formatting them. The containing source is JSON though
- Folder and file names must be according to [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)
- Each language variant has its own file. If the variants (e.g. de_DE, de_AT, de_CH) don't have any differences, only one file must be added
- When creating a language variant file all mappings need to be created (redundant) – not incrementally
- Changes should only be done within the source files

Files within the `src/` folder are called **language files**.

##### 2.1.2.1 Language File Specification

Example structure for the German language file, which only contains lower case characters for brevity:

```javascript
{
    "metadata": {
        "alphabet": "Latn",
        "continent": "EU",
        "language": "German",
        "native": "Deutsch",
        "sources": [
            "https://en.wikipedia.org/wiki/German_orthography#Special_characters",
            "https://en.wikipedia.org/wiki/German_orthography#Sorting"
        ]
    },
    "data": {
        "ü": {
            "mapping": {
                "base": "u",
                "decompose": "ue"
            }
        },
        "ö": {
            "mapping": {
                "base": "o",
                "decompose": "oe"
            }
        },
        "ä": {
            "mapping": {
                "base": "a",
                "decompose": "ae"
            }
        },
        "ß": {
            "mapping": {
                "decompose": "ss"
            }
        }
    }
}
```

###### metadata.alphabet

Required  
Type: `String`

An alphabet code based on [ISO 15924](https://en.wikipedia.org/wiki/ISO_15924)
that specifies the associated alphabet.

###### metadata.continent

Required  
Type: `String` or `Array` of `String`

A continent code based on [ISO-3166](https://en.wikipedia.org/wiki/List_of_sovereign_states_and_dependent_territories_by_continent_%28data_file%29)
that specifies the associated continent.

###### metadata.language

Required  
Type: `String`

The associated language written in English.

###### metadata.native

Required  
Type: `String`

The associated language written in the native language.

###### metadata.sources

Required  
Type: `Array`

An array containing links to diacritic sources including mapping. Include an empty array if no sources are to be listed.

###### data

Required  
Type: `Object`

An object containing the actual mapping information. Every diacritic has its own
object key. The value for each diacritic is an object specified below:

###### data.{character}.mapping

Required  
Type: `Object`

An object containing mapping values for the given diacritic. This must contain a
base or decompose value, or both. It can not be empty.

###### data.{character}.mapping.base

Optional  
Type: String

This is the base of the diacritic character (e.g. `ü` has a base of `u`, an
unaccented character).

###### data.{character}.mapping.decompose

Optional  
Type: String

This is the character, or combination of characters used to represent the
diacritic character (e.g. `ö` decomposes into `oe` in German).

## 3. Dist Branch

The `dist` branch contains generated data from master's `src/` files combined into a single file, `diacritics.json`. This file has almost identical structure like the files within `src/` and will be in a valid `.json` format which can not contain comments.

This distribution file is automatically generated when a pull request targeting the master branch is merged using a build that is running on Travis CI. This workflow has been chosen to allow contributions without a Git clone or command line things in general.

The file will be generated to a uniquely named folder, e.g. `v1`. In case the JSON structure changes, it's then possible to generate it to a different folder. This makes sure that components depending on the old file structure will still work.

While the dist branch contains all data, a server-side component is used to serve and filter them.

### 3.1 diacritics.json

The structure will use the language file name
([ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)) as the key,
and the internal file data as the value. In this example, German and French
languages are included (truncated to avoid repetition):

```js
{
    "de": {
        "metadata": {...},
        "data": {...}
    },
    "fr": {
        "metadata": {...},
        "data": {...}
    }
}
```

#### 3.1.1 Visual Diacritic Equivalents

Beside of generating all the data into one file the build _automatically_ adds visual diacritic equivalents to `diacritics.json`. Behind each diacritic may be several identical looking characters that have a different Unicode code point. To make sure the mapping process will catch all of them, these equivalents need to be mapped too.

These equivalents are added to the language file under `data.{character}.equivalents` (`{character}` is a placeholder for a diacritic).

Example (only showing one lower case diacritic):

```javascript
{
    "de": {
        "metadata": {...},
        "data": {
            "ü": {
                "mapping": {
                    "base": "u",
                    "decompose": "ue"
                },
                "equivalents": [
                    {
                        "raw": "ü"
                        "unicode": "\u00fc",
                        "html_decimal": "&#252;",
                        "html_hex": "&#xfc;",
                        "encoded_uri": "%C3%BC",
                        "html_entity": "&uuml;"
                    },
                    {
                        "raw": "ü",
                        "unicode": "u\\u0308",
                        "html_decimal": "u&#776;",
                        "html_hex": "u&#x0308;",
                        "encoded_uri": "u%CC%88"
                    }
                ]
            }
        }
    }
}
```

##### equivalents

Required  
Type: `Array`

Contains an array containing equivalents objects consisting of the original diacritic character and any additional characters created through normalization.

##### equivalents[index].raw

Required  
Type: `String`

Contains a rendered equivalent character (e.g. `ü`).

##### equivalents[index].unicode

Required  
Type: `String`

Contains an escaped unicode (hex) value of the character (e.g. `\u00f6`).

##### equivalents[index].html_decimal

Required  
Type: `String`

Contains a HTML entity in decimal format (e.g. `&#246;`).

##### equivalents[index].html_hex

Required  
Type: `String`

Contains a HTML entity in hex format (e.g. `&#xf6;`).

##### equivalents[index].html_entity

Required  
Type: `String`

Contains a named HTML entity if one exists (e.g. `&ouml;`) - [ref](https://dev.w3.org/html5/html-author/charref).

##### equivalents[index].encoded_uri

Required  
Type: `String`

Contains a URL encoded value (e.g. `%C3%B6`) - see [percent encoding](https://en.wikipedia.org/wiki/Percent-encoding).