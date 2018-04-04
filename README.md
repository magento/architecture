The purpose of the Architecture repository is to support architectural discussions around design documents, proposals or any other architectural documents

---

Disclaimer: there is no guarantee that approved proposals/design documents will be implemented by Magento core team. The purpose of the repository is to support discussions, and does not guarantee resources dedicated to the implementation.

---

## Documents in the Repository

New documents are processed through PRs targeting `master` branch.
As a result, all documents that are present in the repository are the ones that have been approved.
All closed (not merged) PRs represent declined documents.
All open PRs represent open/abandoned documents. Abandoned PRs may be closed as irrelevant.

## The Workflow

Terms:
* **Author** - a Magento core engineer, or any community member

1. Fork the repository and create a document in your branch
   1. Design Documents in "design-documents" folder. Expected from Magento core engineers, though not prohibited for community members
   1. Proposals in "proposals" folder. Expected from Magento core engineers or community members
   1. More folders may be added in future if necessary
1. Create a PR with the document to discuss to the repository
1. Share the PR with internal team(s) and the Magento community through existing channels (Twitter, Slack, blog post, etc)
   1. Suggestion: include deadline for receiving feedback
1. The feedback is left as:
   1. Comments to the PR
   1. Likes/dislikes/other emojis to the comments or the PR itself. It's encouraged to add an explanation
1. Update the PR according to the received feedback or reply to the feedback that is not going to be incorporated (with reasons)
1. When everybody come to an agreement, add "approved" label to the PR
1. The PR is merged by a Magento core architect and implementation may start based on the decision
The implementation process is out of scope of this project. The document is kept in the repository for historical reasons

After a document is approved, new discussion may be necessary based on discoveries during the implementation or new information being available.
In this case, a new PR should be opened that modifies existing document. The PR should include reasons for the change.
