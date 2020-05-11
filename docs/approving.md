# APPROVING
Guide for approvers, and what they should do

## APPROVERS
The GitHub repo will use [CODEOWNERS](/CODEOWNERS) to determine the approvers. Initially, new services won't have CODEOWNERS, so it will default to the [aws_playbook_core](https://github.com/orgs/open-itg/teams/aws_playbook_core) team.

If there is only the core team currently set as approver, the Risk Analyst shall determine the correct platform approver, if any, and assign them to the Pull Request too. They'll also update the [CODEOWNERS](/CODEOWNERS) file in the source branch for the Pull Request to add that approver for the service going forward.

## REVIEW
(Define what to look for- is there sufficient info for prevent, detect, etcetera to actually be implemented? Does it match controls)

## MERGING
The final approver will merge to master