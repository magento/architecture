### Overview

At the time of this writing only 85% of the `*.phtml` files in Magento core are being statically 
tested due to `@codingStandardsIgnore` present in over 700 templates. There is a current security initiative 
that requires a large number of templates to be modified and they need to be covered by static checks moving 
forward as part of the Acceptance Criteria of the initiate. 

However, due to the lack of current static template checks the codebase has thousands of static template errors and will require many 
hours to address every issue in every file. Additionally, many of the tests are not reasonable, checking incorrectly, or simply gratuitous for `*.phtml` templates. 


### Design

To address this issue for the immediate efforts as well as for all developers in the future I am proposing to remove the following sniffs
for `*.phtml` files.

|Sniff|Rationale|
|-----|---------|
| `Magento2.Files.LineLength` |HTML is frequently much cleaner when kept on single lines even if they are longer. This sniff also incorrectly counts lines containing PHP.|
| <ul><li>`Squiz.ControlStructures.ControlSignature.NewlineAfterOpenBrace`</li><li> `Squiz.WhiteSpace.ScopeClosingBrace`</li><li>`PEAR.ControlStructures.ControlSignature`</li></ul>|Inline PHP can be cleaner and easier to read in many cases within a PHTML document. <br/><br/>  Additionally, the `PEAR.ControlStructures.ControlSignature.Found` sniff disallows any inline control structures but allows the alternative control syntax (if (foo): blah endif; ) as well as ternaries (foo ? bar : baz) which are the same thing. <br/> <br/> These sniffs also incorrectly detect the control signatures leading to a mixed set of cases where it is actually flagged. It seems to only flag some of these when the endif or } is on the same line as the if|
| `Squiz.Operators.IncrementDecrementUsage`|Very common in templating|
| `Magento2.Security.LanguageConstruct.DirectOutput`|This makes sense for PHP files but not for templates where the whole purpose is to output content.|

#### Acceptance Criteria Fulfillment

1. Acceptance Criteria 1
 The proposed sniffs exclude `*phtml` files preferably in a way that doesn't affect the `magento-coding-standards` repo.
