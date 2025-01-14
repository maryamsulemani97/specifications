# Search API

## 1. Summary

The search endpoints retrieve documents from an index. Their returned documents are considered relevant based on the settings of the index and the provided search parameters.

## 2. Motivation
N/A

## 3. Functional Specification

Meilisearch exposes 2 routes to perform search requests:

- GET `indexes/:index_uid/search`
- POST `indexes/:index_uid/search`

- 🔴 If the index does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

If a master key is used to secure a Meilisearch instance, the auth layer returns the following errors:

- 🔴 Accessing these routes without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing these routes with a key that does not have permissions (i.e. other than the master key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

`POST` HTTP verb errors:

- 🔴 Omitting the Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.

### 3.1. Search Payload Parameters

| Field                                                 | Type                      | Required |
|-------------------------------------------------------|---------------------------|----------|
| [`q`](#311-q)                                         | String                    | False    |
| [`filter`](#312-filter)                               | Array of String - String  | False    |
| [`sort`](#313-sort)                                   | Array of String - String  | False    |
| [`facets`](#314-facets)                               | Array of String - String  | False    |
| [`limit`](#315-limit)                                 | Integer                   | False    |
| [`offset`](#316-offset)                               | Integer                   | False    |
| [`attributesToRetrieve`](#317-attributestoretrieve)   | Array of String - String  | False    |
| [`attributesToHighlight`](#318-attributestohighlight) | Array of String - String  | False    |
| [`highlightPreTag`](#319-highlightpretag)             | String                    | False    |
| [`highlightPostTag`](#3110-highlightposttag)          | String                    | False    |
| [`attributesToCrop`](#3111-attributestocrop)          | Array of String - String  | False    |
| [`cropLength`](#3112-croplength)                      | Integer                   | False    |
| [`cropMarker`](#3113-cropmarker)                      | String                    | False    |
| [`showMatchesPosition`](#3114-showmatchesposition)                | Boolean                   | False    |

#### 3.1.1. `q`

- Type: String
- Required: False
- Default: `null`

`q` contains the terms to search within the index documents.

- 🔴 Sending a value with a different type than `String` or `null` for `q` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

> When q isn't specified, Meilisearch performs a **placeholder search**. A placeholder search returns all searchable documents in an index, modified by any search parameters used and sorted by that index's custom ranking rules. If the index has no sort search parameter or custom ranking rules, the results are returned in the order of their internal database position.

> Meilisearch only considers the first ten words of any given search query to deliver a fast search-as-you-type experience.

> `q` supports the [Phrase Query](0043-phrase-query.md) expression.

- 🔴 Sending a value with a different type than `String` or `null` for `q` returns an [bad_request](0061-error-format-and-definitions.md#bad_request) error.

#### 3.1.2. `filter`

- Type: Array of String (POST) | String (POST/GET)
- Required: False
- Default: `[]|null`

`filter` contains a filter expression written as a string or an array of strings. It permits to refine search results.

Attributes used as filter criteria must be added to the `filterableAttributes` list of an index settings. See [Filterable Attributes Setting API](0123-filterable-attributes-setting-api.md).

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `filter` returns an [invalid_filter](0061-error-format-and-definitions.md#invalid_filter) error.
- 🔴 Sending an invalid syntax for `filter` returns an [invalid_filter](0061-error-format-and-definitions.md#invalid_filter) error.
- 🔴 Sending a field not defined as a `filterableAttributes` for `filter` returns an [invalid_filter](0061-error-format-and-definitions.md#invalid_filter) error.

##### 3.1.2.1. Supported Operators And Syntaxes.

###### 3.1.2.1.1. Supported Operators

- `<`, `<=`, `>`, `>=`, `TO`; only operate on number values. MeiliSearch returns only documents that have numbers in this field.
- `=`, `!=`/`NOT`; operate on string and number values. MeiliSearch returns only documents that have numbers, strings, or arrays of strings in this field.
- `AND`/`OR`; permits to cumulate several operations.

`filter` cannot operate on `null`, arrays of "undefined elements" (ex: array of `null`).

###### 3.1.2.1.2. Supported Syntaxes

Three syntaxes will be accepted for the `filter` parameter during search. `String syntax`, `Array syntax` and `Mixed syntax`.

**String syntax**

The string syntax uses operators combined with parentheses to express a search filter.

Example:
```json
{
    "filter": "(genres = Comedy OR genres = Romance) AND director = 'Mati Diop'"
}
```

**Array syntax**

The array syntax uses dimensional array to express logical connectives.

- Inner arrays elements are connected by an `OR` operator (e.g. [["genres:Comedy", "genres:Romance"]]).
- Outer arrays elements are connected by an `AND` operator (e.g. ["genres:Romance", "director:Mati Diop"]).

Example:
```json
{
    "filter": [["genres = Comedy", "genres = Romance"], "director = 'Mati Diop'"]
}
```

**Mixed syntax**

The mixed syntax can mix string and array syntaxes.

Let's say that we want to translate
```json
{
    "filter": "((genres = Comedy AND genres = Romance) OR genres = Action) AND director != 'Mati Diop'"
}
```
Example:
```json
{
    "filter": [["genres = Comedy AND genres = Romance", "genres = Action"], "NOT director = comedy"]
}
```
> Note that string values that are longer than a single word need to be enclosed by quote. `"director = Mati Diop"` will lead to a parsing error. The valid syntax is `"director = 'Mati Diop'"`.

#### 3.1.3. `sort`

- Type: Array of String (POST) | String (GET)
- Required: False
- Default: `[]|null`

`sort` contains a sort expression written as a string or an array of strings. It sorts the search results at query time according to the specified attributes and indicated order.

Attributes used as sort criteria must be added to the `sortableAttributes list of an index settings. See [Sortable Attributes Setting API](0123-sortable-attributes-setting-api.md).

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `sort` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.
- 🔴 Sending an invalid syntax for `sort` returns an [invalid_sort](0061-error-format-and-definitions.md#invalid_sort) error.
- 🔴 Sending a field not defined as a `sortableAttributes` for `sort` returns an [invalid_sort](0061-error-format-and-definitions.md#invalid_sort) error.

> See [Sort](0055-sort.md)

#### 3.1.4. `facets`

- Type: Array of String (POST) | String (GET)
- Required: False
- Default: `[]|null`

`facets` permits to specify facets to be computed for the current search query.

It returns the number of documents matching the current search query for each specified facet.

This parameter can take two values:

- An array of attributes: `facets=["attributeA", "attributeB", …]`
- A wildcard `"*"` — this returns a count for all facets present in `filterableAttributes`

Attributes used in `facets` must be added to the `filterableAttributes` list of an index settings. See [Filterable Attributes Setting API](0123-filterable-attributes-setting-api.md).

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `facets` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.
- 🔴 Sending a field not defined as a `filterableAttributes` for `facets` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

The distribution of the different facets is returned in the `facetDistribution` response parameter.

#### 3.1.5. `limit`

- Type: Integer
- Required: False
- Default: `20`

Sets the maximum number of documents to be returned for the search query.

- 🔴 Sending a value with a different type than `Integer` for `limit` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

#### 3.1.6. `offset`

- Type: Integer
- Required: False
- Default: `0`

Sets the starting point in the search results, effectively skipping over a given number of documents.

- 🔴 Sending a value with a different type than `Integer` for `offset` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

#### 3.1.7. `attributesToRetrieve`

- Type: Array of String (POST) | String (GET)
- Required: False
- Default: `["*"]`, meaning all the attributes

Configures which attributes will be retrieved in the returned documents.

If no value is specified, the default value of `attributesToRetrieve` is used (`["*"]`). This corresponds to the `displayedAttributes` index setting, which by default contains all attributes found in the documents.

> If an attribute is missing from `displayedAttributes` index setting, `attributesToRetrieve` silently ignore it, and the field doesn't appear in the returned search results.

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `attributesToRetrieve` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

#### 3.1.8. `attributesToHighlight`

- Type: Array of String (POST) | String(GET)
- Required: False
- Default: `[]|null`

Configures which fields may have highlighted parts, given that they match the requested query terms (i.e. the terms in the [`q`](#311-q) search parameter). Pre/post highlighting tags are applied around each word corresponding to a query term.

If `attributesToHighlight` is present in the search query, the search results will include a `_formatted` object containing the attributes and their highlighted parts. For more detailed regarding the `_formatted` behavior, see the [3.2.1.1.2. `_formatted`](#32112-formatted) section.

If `"*"` is provided as a value (`attributesToHighlight=["*"]`), all the attributes present in `displayedAttributes` setting will be highlighted.

Highlighted parts are surrounded by the [`highlightPreTag`](#319-highlightpretag) and [`highlightPostTag`](#3110-highlightposttag) parameters.

`attributesToHighlight` only works on values of the following types: `string`, `number`, `array`, `object`. When highlighted, number attributes are transformed to string.

- 🔴 Sending a value with a different type than `Array[String]`(POST), `String`(GET) or `null` for `attributesToHighlight` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

##### 3.1.8.1. searchableAttributes

Attributes not defined in the `searchableAttributes` index setting are also highlighted if assigned to `attributesToHighlight`.

##### 3.1.8.2. stopWords

Attributes defined in the `stopWords` index setting are also highlighted if matched.

##### 3.1.8.3. Tokenizer Separators

Tokenizer separators are not highlighted.

##### 3.1.8.4. synonyms

Synonyms are also highlighted.

#### 3.1.9. `highlightPreTag`

- Type: String
- Required: False
- Default: `"<em>"`

Specifies the string to put **before** every highlighted query terms.

This parameter is applied to the fields configured in `attributesToHighlight`. If there are none, this parameter has no effect. See [3.1.8. `attributesToHighlight`](#318-attributestohighlight) section.

- 🔴 Sending a value with a different type than `String` for `highlightPreTag` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

If `attributesToHighlight` is omitted while `highlightPreTag` is specified, there is no error.

#### 3.1.10. `highlightPostTag`

- Type: String
- Required: False
- Default: `"</em>"`

Specifies the string to put **after** the highlighted query terms.

This parameter is applied to the fields from `attributesToHighlight`. If there are none, this parameter has no effect. See [3.1.8. `attributesToHighlight`](#318-attributestohighlight) section.

- 🔴 Sending a value with a different type than `String` for `highlightPostTag` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

If `attributesToHighlight` is omitted while `highlightPostTag` is specified, there is no error.

#### 3.1.11. `attributesToCrop`

- Type: Array[String]|String
- Required: False
- Default: `[]|null`

Defines document attributes to be cropped. Cropped attributes have their values shortened around query terms.

If `attributesToCrop` is present in the search query, the search results will include a `_formatted` object containing the attributes and their cropped parts. For more detailed regarding the `_formatted` behavior, see the [3.2.1.1.2. `_formatted`](#32112-formatted) section.

If `"*"` is provided as a value (`attributesToCrop=["*"]`), all the attributes present in `displayedAttributes` setting will be cropped.

The number of words contained in the cropped value is defined by the `cropLength` parameter. See [3.1.1.12. `cropLength`](#3112-croplength) section.

The value of `cropLength` can be customized per attribute. See [3.1.12.1. Custom `cropLength` Defined Per Cropped Attribute](#31121-custom-croplength-defined-per-attribute) section.

The engine adds a marker by default in front of and/or behind the part selected by the cropper. This marker is customizable. See [3.1.13. `cropMarker`](#3113-cropmarker) section.

- 🔴 Sending a value with a different type than `Array[String]`(POST), `String`(GET) or `null` for `attributesToCrop` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

##### 3.1.11.2. searchableAttributes

Attributes configured in `attributesToCrop` are cropped even if not present in the `searchableAttributes` index setting.

##### 3.1.11.3. stopWords

Terms defined in the `stopWords` index setting are counted as words regarding `cropLength`.

##### 3.1.11.3. Tokenizer Separators

Tokenizer separators aren't counted as words regarding `cropLength`.

#### 3.1.12. `cropLength`

- Type: Integer
- Required: False
- Default: `10`

Sets the total number of **words** to keep for the cropped part of an attribute specified in the `attributesToCrop` parameter. It means that if `10` is set for `cropLength`, the cropped part returned in `_formatted` will only be 10 words long.

This parameter is applied to the fields from `attributesToCrop`. If there are none, this parameter has no effect. See [3.1.11. `attributesToCrop`](#3111-attributestocrop) section.

Sending a `0` value deactivates the cropping unless a custom crop length is defined for an attribute inside `attributesToCrop`.

- 🔴 Sending a value with a different type than `Integer` for `cropLength` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

##### 3.1.12.1. Custom `cropLength` Defined Per Attribute.

Optionally, indicating a custom crop length for any of the listed attributes is possible:

`"attributesToCrop":["attributeNameA:15", "attributeNameB:30"]`

A custom crop length set in this way has priority over the `cropLength` parameter.

##### 3.1.12.2 Examples

###### 3.1.12.1.1. Extending around

Given an attribute defined in `attributesToCrop` containing:

`"In his ravenous hatred he found no peace, and with boiling blood he scoured the umbral plains, seeking vengence afgainst the dark lords who had robbed him."`

With `croplength` defined as `5` and `q` defined as `boiling blood`, the cropped value will be:

`"…and with boiling blood he…"`

Cropped query terms are counted as a word regarding `cropLength`.

Sending more query terms than the `cropLength` value has no impact. The cropped part will contain the `cropLength` number.

###### 3.1.12.1.2. Keeping a phrase context

After Meilisearch has chosen the best possible match window (some number of words < `cropLength`), it will add words from before or after the match window until the total number is equal to `cropLength`. In doing so, it will attempt to add context to the match window by choosing words from the same sentence(s) where the match window occurs.

For instance, for the matching word `Split` the text:

`"Natalie risk her future. Split The World is a book written by Emily Henry. I never read it."`

will be cropped like:

`…Split The World is a book written by Emily Henry…`

and not like:

`Natalie risk her future. Split The World is a book…`

#### 3.1.13. `cropMarker`

- Type: String
- Required: False
- Default: `"…"` (U+2026)

Sets which string to add before and/or after the cropped text. See [3.1.11. `attributesToCrop`](#3111-attributestocrop) section.

The specified crop marker is applied by following rules outline in section [3.1.13.1. Applying `cropMarker`](#31131-applying-cropmarker).

Specifying `cropMarker` to `""` or `null` implies that no marker will be applied to the cropped part.

This parameter is applied to the fields configured in `attributesToCrop`. If there are none, this parameter has no effect. See [3.1.11. `attributesToCrop`](#3111-attributestocrop) section.

- 🔴 Sending a value with a different type than `String` or `null` for `cropMarker` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

##### 3.1.13.1. Applying `cropMarker`

###### 3.1.13.1.1. Matched Part To Be Cropped

The cropping algorithm tries to match the window with the highest density of query terms within the `cropLength` limit.

The cropping algorithm tries to find the crop window that contains the most relevant matches.

1. That has the highest count of unique matches

For example, for the query terms `split the world`, then the interval `the split the split the` has `5` matches but only `2` unique matches (`1` for `split` and `1` for `the`) where the interval `split of the world` has `3` matches and `3` unique matches. So the interval `split of the world` is considered better.

2. That have the minimum distance between matches

For example, for the query terms `split the world`, then the interval `split of the world` has a distance of `3` (`2` between `split` and `the`, and `1` between `the` and `world`) where the interval `split the world` has a distance of `2`. So the interval `split the world` is considered better.

3. That have the highest count of ordered matches

For example, for the query terms `split the world`, then the interval `the world split` has `2` ordered words where the interval `split the world` has `3`. So the interval `split the world` is considered better.

Only one cropped part from an attribute is returned.

If no part is found when selecting a part to be cropped, the returned value in `_formatted` will start at the beginning of the attribute and include a number of words equal to `cropLength`.

###### 3.1.13.1.2. Positioning Markers

If the cropped part has been matched against query terms and contains the beginning of the attribute to be cropped, the `cropMarker` is not placed to the left of the cropped part.

If the cropped part has been matched against query terms and contains the end of the attribute to be cropped, the `cropMarker` is not placed to the right of the cropped part.

#### 3.1.14. `showMatchesPosition`

- Type: Boolean
- Required: False
- Default: `false`

Adds a `_matchesPosition` object to the search response that contains the location of each occurrence of queried terms across all fields. The given positions are in bytes.

It's useful when more control is needed than offered by the built-in highlighting/cropping features.

- 🔴 Sending a value with a different type than `Boolean` or `null` for `showMatchesPosition` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.


### 3.2. Search Response Properties

| Field                                                 | Type                         | Required |
|-------------------------------------------------------|------------------------------|----------|
| [`hits`](#321-hits)                                   | Array[Hit]                   | True     |
| [`limit`](#322-limit)                                 | Integer                      | True     |
| [`offset`](#323-offset)                               | Integer                      | True     |
| [`estimatedTotalHits`](#324-estimatedTotalHits)       | Integer                      | True     |
| [`facetDistribution`](#325-facetdistribution)       | Object                       | False    |
| [`processingTimeMs`](#326-processingtimems)           | Integer                      | True     |
| [`query`](#327-query)                                 | String                       | True     |

#### 3.2.1. `hits`

- Type: Array[Hit]
- Required: True

Results of the search query as an array of documents.

> Hit object represents a matched document as a search result.

> The search parameters `attributesToRetrieve` influence the returned payload for a hit. See [3.1.7. `attributesToRetrieve`](#317-attributestoretrieve) section.

A search result can contain special properties. See [3.2.1.1. `hit` Special Properties](#3211-hits-special-properties) section.

##### 3.2.1.1. `hit` Special Properties

| Field                                | Type        | Required |
|--------------------------------------|-------------|----------|
| [`_geoDistance`](#32111-geodistance) | Integer     | False    |
| [`_formatted`](#32112-formatted)     | Object      | False    |
| [`_matchesPosition`](#32113-matchesposition) | Object      | False    |

###### 3.2.1.1.1. `_geoDistance`

- Type: Integer
- Required: False

Search queries using `_geoPoint` returns a `_geoDistance` field containing the distance in meters between the document `_geo` coordinates and the specified `_geoPoint`.

> See [GeoSearch](0059-geo-search.md)

###### 3.2.1.1.2. `_formatted`

- Type: Object
- Required: False

`_formatted` is an object returned in the search response, only if at least one of the following paramaters has been set in the search query:
- `attributesToHighlight`
- `attributesToCrop`

If `attributesToHighlight` and `attributesToCrop` are not set, `_formatted` is not returned.

This `_formatted` object will be present in each returned document in the `hits` field.

Example:

```json
{
    "attributesToCrop": ["title"]
}
```

```json
{
    "hits": [
        {
            "id": 2,
            "title": "Pride and Prejudice",
            "_formatted": {
                "id": "2",
                "title": "Pride and Prejudice"
            }
        },
        {
            "id": 456,
            "title": "Le Petit Prince",
            "_formatted": {
                "id": "456",
                "title": "Le Petit Prince",
            }
        }
    ],
    ...
}
```

Which attributes are present in `_formatted`?

*Remember the main rule: `_formatted` is only present if `attributesToHighlight` or `attributesToCrop` is set.*

The `_formatted` object contains attributes coming from the original document, depending on the parameters the users set during the search query. Indeed, **`_formatted` contains all the attributes present in `attributesToRetrieve`, `attributesToHighlight`, and `attributesToCrop` combined**.

Knowing the default value of `attributesToRetrieve` is `["*"]` (so all the attributes present in `displayedAttributes`), if no `attributesToRetrieve` are set in the search query, `_formatted` will return all the `displayedAttributes`.

Returning attributes in the `_formatted` object does not mean these attributes will be necessarily highlighted or cropped, see the next point.

Which attributes are highlighted or cropped in `_formatted`?

No matter which attributes are retrieved in `_formatted` (according to the previous section "Which attributes are present in `_formatted`?"):
- Only the attributes present in `attributesToHighlight` are highlighted.
- Only the attributes present in `attributesToCrop` are cropped.
- Attributes present in both are cropped and highlighted at the same time.

Some edge cases:
- If cumulated fields in `attributesToHighlight` and `attributesToCrop` resolve to only having non-existent fields, `_formatted` is not returned.

Some examples:
*The examples work the same with `attributesToCrop`*

Example 1:

```json
{
    "q": "t",
    "attributesToHighlight": ["title"]
}
```

```json
{
    "hits": [
        {
            "id": 1,
            "title": "The Hobbit",
            "author": "J. R. R. Tolkien",
            "_formatted": {
                "id": "1",
                "title": "<em>T</em>he Hobbit",
                "author": "J. R. R. Tolkien"
            }
        }
    ],
    ...
}
```
-> All the attributes (so `id`, `title` and `author`) are returned in `_formatted` because by default `attributesToRetrieve` is set to `["*"]`.
-> Only `title` is highlighted.

Example 2:

```json
{
    "q": "t",
    "attributesToHighlight": ["*"]
}
```

```json
{
    "hits": [
        {
            "id": 1,
            "title": "The Hobbit",
            "author": "J. R. R. Tolkien",
            "_formatted": {
                "id": "1",
                "title": "<em>T</em>he Hobbit",
                "author": "J. R. R. <em>T</em>olkien"
            }
        }
    ],
    ...
}
```
-> `id`, `title` and `author` are returned in `_formatted` because`attributesToHighlight` is set to `["*"]` (but also `attributesToRetrieve` by default).
-> Both `title` and `author` are highlighted because `attributesToHighlight` is set to `["*"]`.

Example 3:

```json
{
    "q": "t",
    "attributesToRetrieve": ["author"],
    "attributesToHighlight": ["title"]
}
```

```json
{
    "hits": [
        {
            "author": "J. R. R. Tolkien",
            "_formatted": {
                "title": "<em>T</em>he Hobbit",
                "author": "J. R. R. Tolkien"
            }
        }
    ],
    ...
}
```
-> Only `author` is returned at the root of the document because defined in the `attributesToRetrieve`.
-> Only `author` and `title` are returned in `_formatted` because the addition of `attributesToRetrieve` and `attributesToHighlight`.
-> Only `title` is highlighted because the only one defined in `attributesToHighlight`.

Example 4:

```json
{
    "q": "t",
    "attributesToRetrieve": [],
    "attributesToHighlight": ["*"]
}
```

```json
{
    "hits": [
        {
            "_formatted": {
                "id": "1",
                "title": "<em>T</em>he Hobbit",
                "author": "J. R. R. <em>T</em>olkien"
            }
        }
    ],
    ...
}
```
-> No attributes are returned at the root of the document because `attributesToRetrieve` is set to `[]`.
-> All the attributes are returned in `_formatted` because `attributesToHighlight` is set to `["*"]`.
-> All the attributes are highlighted because `attributesToHighlight` is set to `["*"]`.


###### 3.2.1.1.3. `_matchesPosition`

- Type: Object
- Required: False

Contains the location of each occurrence of queried terms across all fields. The `_matchesPosition` object is added to a search result when the `showMatchesPosition` search parameter is specified to true.

The beginning of a matching term within a field is indicated by `start`, and its `length` by length.

`start` and `length` are measured in bytes and not the number of characters. For example, `ü` represents two bytes but one character.

> See [3.1.14. `showMatchesPosition`](#3114-showmatchesposition) section.

#### 3.2.2. `limit`

- Type: Integer
- Required: True

Returns the `limit` search parameter used for the query.

> See [3.1.5. `limit`](#315-limit) section.

#### 3.2.3. `offset`

- Type: Integer
- Required: True

Returns the `offset` search parameter used for the query.

> See [3.1.6. `offset` section](#316-offset) section.

#### 3.2.4. `estimatedTotalHits`

- Type: Integer
- Required: True

Returns the estimated number of candidates for the search query.

#### 3.2.5. `facetDistribution`

- Type: Object
- Required: False

Added to the search response when `facets` is set for a search query. It contains the number of remaining candidates for each specified facet in the `facets` search parameter.

If a field distributed as a facet contains no value, it is returned as a `facetDistribution` field with an empty object as value.

> See [3.1.4. `facet`](#314-facets) section.

#### 3.2.6. `processingTimeMs`

- Type: Integer
- Required: True

Processing time of the search query in **milliseconds**.

#### 3.2.7. `query`

- Type: String
- Required: True
- Default: `""`

Query originating the response. Equals to the `q` search parameter.

> See [3.1.1. `q`](#311-q) section.

## 2. Technical Details
n/a

## 3. Future Possibilities

- Add dedicated errors to replace `bad_request` error.

### 3.1. Formatting Search Results

- Replaces `_matchesPosition` with chars position instead of bytes. It could also be a `mode` to choose `byte` or `char`.
- Move `attributesToHighlight`, `highlightPreTag`, `highlightPostTag`, `attributesToCrop`, `cropLength` and `cropMarker` into a `formatter` objet.
- Add an option to only highlight complete query term.
- Expose the `formatter` resource as an index setting.
- Highlight a phrase search as a single highlighted section.
