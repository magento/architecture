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

facilitator
: a Magento architect who makes sure the process is followed (Olga Koplyova @buskamuza)

## The Workflow

1. Fork the repository and add or edit a document in your branch.
Contributions are expected from Magento core engineers mostly, although the community members are able to contribute.
1. Create a PR with the new or updated document to discuss
1. Share the PR with internal team(s) and the Magento community through existing channels (Twitter, Slack, blog post, etc)
   1. Suggestion: include deadline for receiving feedback
1. Get a feedback. We expect a feedback as:
   1. Comments into the PR thread
   1. Likes/dislikes/other emojis to the comments or the PR itself. We kindly encourage contributors to submit explanations about pros and cons that they have noticed.
1. Update the PR to the received feedback accordingly or submit a reply if the proposed changes are not applicable with clear explanation.
   1. Add `needs update` label while the PR is in work, so it's not 
1. When all participants of the discussion have come to agreement and confirmed their approval, a Magento core architect merges the PR.

### Design Documents Review Requirements

* Required reviewers are architects assigned to the [affected components](https://github.com/magento/magento2/wiki/Magento-Components-Assignment)
* Anybody else may join the review and provide feedback, but they're not obligated to review the entire document or to meet deadlines
* For existing (running or about to run) projects, a minimum time frame given for one round of review is 2 business days from the notification notice. The author may specify due date. For documents that do not block any existing projects (proposals or design documents that describe features for unforeseeable future), no due date is guaranteed.

### Assigned Reviewer Responsibilities

* Review the entire document by specified due date (if any)
   * If it is impossible, find a replacement
   * Contact the facilitator in case you can't find a replacement
   * If the due date is unreasonable for the size of the document, agree on another due date with the author
* Include a detailed feedback
   * Ensure the feedback is objective
   * Ensure the feedback provides value to the document. Avoid side-notes and off-topics (or mention explicitly that this is just a note for future)
   * If the document is rejected, include clear and objective explanation for it. Rejection is final

The implementation process is out of scope in this project.

After approval of the document, a new discussion may be raised basing on the issues occurred during implementation.
It is also possible in case of new informational updates that discover hidden sides of the future implementation.
If it is the case, a new PR should be opened to update existing document. The PR should include explained reasons for the proposed change.
