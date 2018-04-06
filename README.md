This repository is created by initiative of Magento architects to discuss with the Magento community any open questions around Magento 2 architecture such as design documents, proposals, or any other architectural artifacts.

---
NOTE: We do not guarantee that approved changes will be delivered into Magento code base. The purpose of the repository is to start open discussions with the Magento community around architectural concepts of the Magento 2 platform.
---

## Documents in the Repository

*New documents* are processed through PRs targeting the `master` branch.
As a result, the `master` branch must contain content approved by Magento architects only.

All closed (not merged) PRs represent *declined documents*.
Note that abandoned PRs may be closed as irrelevant.

All open PRs represent *open discussions*. 

## Glossary

author
: a Magento core engineer, or any community member

## The Workflow

1. Fork the repository and add or edit a document in your branch
   1. Design Documents must be stored in `design-documents` directory. Contributions are expected from Magento core engineers mostly, although the community members are able to contribute.
   1. Proposals are contributed into the `proposals` directory. Contributions are expected from both Magento core engineers and the community members.
   1. We may add more categories in future
1. Create a PR with the new or updated document to discuss
1. Share the PR with internal team(s) and the Magento community through existing channels (Twitter, Slack, blog post, etc)
   1. Suggestion: include deadline for receiving feedback
1. Get a feedback. We expect a feedback as:
   1. Comments into the PR thread
   1. Likes/dislikes/other emojis to the comments or the PR itself. We kindly encourage contributors to submit explanations about pros and cons that they have noticed.
1. Update the PR to the received feedback accordingly or submit a reply if the proposed changes are not applicable with clear explanation.
1. When all participants of the discussion have come to agreement and confirmed their approval, add "approved" label to the PR.
1. A Magento core architect merges the PR. 

The implementation process is out of scope in this project.

After approval of the document, a new discussion may be raised basing on the issues occurred during implementation.
It is also possible in case of new informational updates that discover hidden sides of the future implementation.
If it is the case, a new PR should be opened to update existing document. The PR should include explained reasons for the proposed change.
