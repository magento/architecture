# Decision-making

### Proposal submission
Proposal is submitted by opening pull request to this repository.
Initiator must present the idea on the architecture meeting.
The announcement should include problem description the proposal is trying to resolve as well as solution(s).

### Discussion
There must be minimum 2 weeks of discussion in the pull request thread.
After that initiator may decide when to start the vote.

### Voting
All eligible voters must get notification about the voting start.
A valid voting period must be announced when voting is started, must be greater that 2 weeks and must not be changed during the vote.

### Approval
The number of `approve` votes must be greater than or equal to the number of `reject` votes multiplied by two in order to merge pull request.
In case of multi-choice voting (implementation details) the decision may be taken by simple plurality. This means that the voting option with the most votes wins. If there are multiple options with the most number of votes, the initiator can choose one of them.
Once pull request is merged the proposal is accepted.

# Eligible voters
The group of eligible voters should be established. The group should represent the Magento community and include internal developers and architects (40%), Magento maintainers (30%), extension developers and partner contributors (30%).

## To Do
- GitHub application for survey.
- GitHub user group with eligible voters.

_Inspired by:_
- [Request for Comments: Voting on PHP features](https://wiki.php.net/rfc/voting)
- [PHP-FIG Public Survey](https://github.com/php-fig/fig-standards/blob/master/proposed/extended-coding-style-guide-meta.md#44-public-survey)
