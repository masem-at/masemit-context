# Build cycle Workflow

This document describes how to implement a single story.

## Flow
|Step|Agent|Activity|Name|
|-|-|-|-|
|1|SM|create-story|Bob|
|2|SM|hands this document and context over to DEV|Bob|
|3|DEV|dev-story|Amelia|
|4|DEV|hands this document and context over to DEV (Adversarial mode)|Amelia|
|5|DEV (Adversarial mode)|code-ewview|Amelia|
|6|DEV (Adversarial mode)|Automatically fix|Amelia|
|7|DEV (Adversarial mode)|commit and push to develop|Amelia|
|8|DEV (Adversarial mode)|Hands this document and context over to SM|Amelia|
|9|SM|Gives summary of last Story and STOPS|Bob|

## Exceptions
If there are questions for me or issues, then stop immediately!