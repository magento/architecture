# Extended Configuration for ElasticSearch Analyzer

## Context

During work on [Japanese localization community project](https://github.com/magento/magento2-jp) we descovered requirement to modify default index settings created by `Magento\Elasticsearch\Model\Adapter\Index\BuilderInterface::build`.
First of all, this is caused by the unique nature of the Japanese writing system. Usage of standard analyzer and tokenizer do not provide sufficient search accuracy. To provide valid search results [Japanese (kuromoji) Analysis Elasticsearch plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-kuromoji.html#analysis-kuromoji) should be used which provides [set of tokenizers and token filters](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-kuromoji-analyzer.html) correctly processing Japanese texts.

Existing [Elasticsearch configuration in Magento 2](https://github.com/magento/magento2/blob/2.3-develop/app/code/Magento/Elasticsearch/etc/esconfig.xml) allows setting only default stemmer name and list of stop words. It introduces two top-level XML elements `stemmer` and `stopwords_file`. Each element includes elements with valid locale code as a name and `default` element which holds configuration that should be used if locale-specific configuration is not defined.

```xml
<config>
    <stemmer>
        <type>stemmer</type>
        <default>english</default><!-- used for all locales that does not have explicit config element-->
        <de_DE>german</de_DE><!-- stemmer name for de_DE locale -->
        <!-- elements for other locales -->
    </stemmer>
    <stopwords_file>
        <default>stopwords.csv</default><!-- stop words list to be used if locale does not defined own list -->
        <de_DE>stopwords_de_DE.csv</de_DE><!-- de_DE locale stop words -->
        <!-- elements for other locales -->
    </stopwords_file>
</config>
```

There may be much more use-cases when SI integrators would like to change default index settings to adjust search results.

To allow complex changes of default Elasticsearch configuration in Magento 2, such us required by Japanese localization project, at the current moment we have 2 options:

1. Use [pluginization](https://devdocs.magento.com/guides/v2.2/extension-dev-guide/plugins.html) mechanism on `\Magento\Elasticsearch\Model\Adapter\Index\Config\EsConfigInterface` and `Magento\Elasticsearch\Model\Adapter\Index\BuilderInterface`.
1. Introduce explicit extension points and extend possibilities of Magento 2 ElasticSearch configuration.

Plugins are powerful mechanism but they push developers to explore the internals of pluginized module that violates the Open-Closed principle. Usage of plugins would require a same code from project-to-project.

## Decision

Magento 2 should provide a possibility to configure analyzer for indexes. To not limit all the power of ElasticSearch without exposing all complexity of configuration should include:
- tokenizer configuration
- token filters configuration
    - customized filters configuration
    - list of token filters to be used by an analyzer
- char filters configuration
    - customized filters configuration
    - list of token filters to be used by an analyzer

The extended schema should be fully backward compatible and consistent with the current implementation.

### General Concepts
As all required configuration options are independent they should be expressed as top-level XML elements.

As all required configuration options have a strong dependency on a language and to be consistent with existing configuration elements each introduced elements should contain `default` element and elements with names corresponding to locale names.

As particular tokenizer and filters may require complex types for configuration XML schema should allow to do that and converting of available options described at ElasticSearch Documentation in JSON to XML elements should follow simple straight-forward rules.

### Tokenizer Configuration
A tokenizer is configured by `tokenizer` element which must include `default` element as a container for common configuration and may include one or more elements with valid locale codes as a name that are containers for locale-specific configuration.

Each direct child of `tokenizer` element must contain `type` element with the name of tokenizer to be used.

Each direct child of `tokenizer` element may contain element which represent configuration parameter available for specified tokenizer type. See [ElasticSearch parameters XML representation](#ElasticSearch-parameters-XML-representation) section for more information.

```xml
<config>
    <tokenizer>
        <default>
            <type>standard</type>
        </default>
        <jp_JP>
            <type>kuromoji_tokenizer</type>
            <mode xsi:type="string">extended</mode>
            <discard_punctuation xsi:type="boolean">false</discard_punctuation>
            <user_dictionary xsi:type="string">userdict_ja.txt</user_dictionary>
        </jp_JP>
    </tokenizer>
</config>
```

### Token Filters Configuration
Token filters are configured by `token_filters` element which must include `default` element as a container for common configuration and may include one or more elements with valid locale codes as a name that are containers for locale-specific configuration.

Each direct child of `token_filters` holds configuration of filter:

- standard filters that should be used by analyzer must be declared as an empty element with name equal to token filter name (e.g. `<lowercase/>`)
- customized filters declared by an element with name equal to custom filter name, required `type` child element which contains the name of customized token and optional parameters. See [ElasticSearch parameters XML representation](#ElasticSearch-parameters-XML-representation) section for more information.

```xml
<config>
    <toke_filters>
        <default>
            <lowercase /><!-- declare usage of standard filter -->
            <my_token_filter><!-- customized token filter -->
                <type>standard</type>
                <max_token_length xsi:type="number">5</max_token_length>
            </my_token_filter>
        </defualt>
        <en_US><!-- locale specific filters>
            <!-- ... -->
        </en_US>
    </toke_filters>
</config>
```

### Char Filters Configuration
Char filters configuration is similar to token filters but declared by `char_filters` element

```xml
<config>
    <char_filters>
        <default>
            <html_strip /><!-- declare usage of standard filter -->
            <my_char_filter><!-- customized filter to replece "-" by "_" -->
                <type>pattern_replace</type>
                <pattern>(\\d+)-(?=\\d)</pattern>
                <replacement>$1_</replacement>
            </my_char_filter>
        </defualt>
        <en_US><!-- locale specific filters>
            <!-- ... -->
        </en_US>
    </char_filters>
</config>
```

### ElasticSearch parameters XML representation

JSON configuration described at ElasticSearch configuration may be converted to XML configuration to specify tokenizer and filter parameters.

#### Objects

Objects or maps are key structure to declare ElasticSearch configuration. When converting JSON to XML keys became element names and values are presented as node values. An element may declare a type of value with `xsi:type` attribute but it may be autodetected by the parser.

| JSON        | XML           | 
| ------------- |-------------|
| `{key: {}}`  | `<key xsi:type="map"><!-- ... --></key>` |


#### Strings

String values are converted to text node:

| JSON        | XML           | 
| ------------- |-------------|
| `{key: "string value"}`  | `<key xsi:type="string">string value</key>` |

#### Numbers

Numeric values are converted to text node. This type utilize both integer and float numbers depend on provided decimal part:

| JSON        | XML           | 
| ------------- |-------------|
| `{key: 42}`  | `<key xsi:type="number">42</key>` |

#### Booleans

Boolean values are converted to text node and may contain "true" or "false":

| JSON        | XML           | 
| ------------- |-------------|
| `{key: true}`  | `<key xsi:type="boolean">true</key>` |

#### Least

Array values represented as series of `item` nodes:

| JSON        | XML           | 
| ------------- |-------------|
| `{key: ['v1', 'v2']}`  | `<key xsi:type="list"><item>v1</item><item>v2</item></key>` |

Item node may declare `ref` attribute that should be unique inside list. `ref` attribute should be used to provide a possibility to override list element by Magento configuration merging mechanism. If module B want to add element to list in ElasticSearch config declared by module A then `ref` attribute should be used as well.

```xml
<!-- module A-->
<articles xsi:type="list">
    <item ref="overridableArticleItem" xsi:type="string">may be overridden</item>
    <item>can not be referenced so is can not be changed</item>
</articles>

<!-- module B which has module A in sequence at module.xml -->
<articles xsi:type="list">
    <item ref="overridableArticleItem" xsi:type="string">overridden value</item>
    <item ref="addedArticleItem">added value</item><!-- after merge articles list will have 3 items -->
</articles>
```

### Disabling Configuration Element

As with proposed changes configuration may be complex and conflict with some requirements it is necessary to provide a mechanism of disabling some parts of configuration during the Magento configuration merge process.

To achieve this goal any element may declare `disabled` boolean attribute:

```xml
<config>
    <token_filters>
        <default>
            <my_token_filter disabled="true"/><!-- disable customized filter declared in other config file -->
        </default>
    </token_filters>
</config>
```

### Deprecation of Magento\Elasticsearch\Model\Adapter\Index\Config\EsConfigInterface

Current `@api` interface `Magento\Elasticsearch\Model\Adapter\Index\Config\EsConfigInterface` violates Interface Segregation principle. It extending is impossible as that would be backward incompatible.

As a solution `Magento\Elasticsearch\Model\Adapter\Index\Config\EsConfigInterface` should be deprecated and instead of it new set of interfaces should be introduced. Each new interface should be responsible for a single aspect of ElasticSearch configuration.

```php



/**
 * @api
 * @deprecated
 */
interface EsConfigInterface extends EsStemmerConfigInterface, EsStopWordsConfigInterface
{
    public function getStemmerInfo();
    public function getStopwordsInfo();
}

/**
 * @api
 */
interface EsStemmerConfigInterface
{
    public function getStemmerInfo();
}

/**
 * @api
 */
interface EsStopWordsConfigInterface
{
    public function getStopwordsInfo();
}

/**
 * @api
 */
interface EsTokenizerConfigInterface
{
    public function getTokenizerInfo(): array;
}

/**
 * @api
 */
interface EsTokenFilterConfigInterface
{
    public function getTokenFiltersInfo(): array;
    public function getTokenFiltersList(): array;
}

/**
 * @api
 */
interface EsCharFilterConfigInterface
{
    public function getCharFiltersInfo(): array;
    public function getCharFiltersList(): array;
}
```

Class `Magento\Elasticsearch\Model\Adapter\Index\Config\EsConfig` will implement all of these interfaces to simplify backward compatible implementation.

### Prototype

Prototype of proposed changes is implemented in [magento/magento2-l10n#1](https://github.com/magento/magento2-l10n/pull/1) by [Hirokazu Nishi](https://github.com/HirokazuNishi) from [Veriteworks](https://veriteworks.co.jp/) in collaboration wit Magento Community Engineering Team.

## Status

Proposed

## Consequences

Proposed changes to ElasticSearch XML configuration give a possibility to fully configure ElastoicSearch analyzer for indexes. It currently not supported multiples tokenizers. The current implementation also assumes usage of unified stemmer and stop words list declared in CSV files. Fixing these limitations is out of the scope of this proposal and should be addressed if real issue reported.

The main drawback of this proposal is an introduction of XML schema that not really match XML philosophy (not strictly defined elements structure). This is done by intention to be consistent with existing configuration and to provide straightforward, not verbose conversion of configuration in JSON described at ElasticSearch documentation to XML format expected by Magento. 