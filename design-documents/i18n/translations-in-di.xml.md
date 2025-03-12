# Translations in `di.xml`

## Current Situation

Currently all XML files that support types, allow `translate` attribute for `string` type. This attribute leads to:

1. The specified string to be translated when XML file is read, so the client of the string receives a translated string automatically.
2. A translation tool recognizes such strings as translatable and collect them in a translation file. 

`di.xml`, instead, supports `translatable` attribute which:

1. Allows the translation tool to recognize the string as translatable and collect it.
2. Does NOT lead to automatic translation of the string for the client code. It is expected that the client code will translate the string.

The decision was based on the requirement that it us not DI responsibility to translate phrases, which makes sense.

## Problem Statement

Such behavior has two problems in my opinion:

1. It is confusing. `translate` sounds very similar to `translatable`, while the behavior is different. Without deep knowledge of the feature, it's impossible to expect such behavior.
2. Somebody can make a mistake (because attribute names sound very similar). Though the translation tool can recognize both attributes, so it will not cause any issue.

## Solution

Unify translation behaviors:

1. Allow `translate` attribute in `di.xml`.
2. `translate` attribute leads to the string being automatically translated during XML parsing and passed to the client code as translated phrase.
   2.1. If it is impossible to translate the phrase (e.g., the application is running in `global` area), the phrase is provided untranslated.
3. Remove `translatable` attribute. Temporarily, support it as deprecated with existing behavior.
