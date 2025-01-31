# Nomev Labs Contributor License Agreement

This repository contains the Nomev Labs CLA. The full document can be found
[here](./CLA.md).

## GitHub Action Setup

It is possible to setup a GitHub action to automatically check for CLA
signatures in a repository.

The following steps describe how to enable a
["lite" version of the CLA Assistant](https://github.com/contributor-assistant/github-action)
and automatically exempt bots.

1. Add a GitHub Action workflow file at `.github/workflows/cla.yml`:
   ```yml
   name: "cla"

   on:
     issue_comment:
       types: [created]
     pull_request_target:
       types: [opened, closed, synchronize]

   permissions:
     actions: write
     contents: write
     pull-requests: write
     statuses: write

   jobs:
     cla:
       runs-on: ubuntu-latest
       steps:
         - name: "CLA Assistant"
           if: (github.event.comment.body == 'recheck' || github.event.comment.body == 'I have read the CLA Document and I hereby sign the CLA') || github.event_name == 'pull_request_target'
           uses: contributor-assistant/github-action@v2.2.1
           env:
             GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
             PERSONAL_ACCESS_TOKEN : ${{ secrets.ORG_TOKEN }}
           with:
             branch: 'cla-signatures'
             path-to-signatures: 'signatures/version1/cla.json'
             path-to-document: 'https://github.com/cowprotocol/cla/blob/main/CLA.md'
             allowlist: '*[bot]'
   ```
2. Create the branch `cla-signatures` where the signatures JSON will be updated:
   ```sh
   git switch -f --orphan cla-signatures
   git commit -m "CLA signatures" --allow-empty
   git push -u origin cla-signatures
   ```
3. Profit $$$

## Troubleshoot

###  Error: Could not retrieve repository contents: Not Found. Status: 404

If this happened on a repository with no contributors, you can try to create an
empty contributor file for the action to populate. 

```sh
git checkout cla-signatures
mkdir -p signatures/version1/
echo $'{\n  "signedContributors": []\n}' > signatures/version1/cla.json
```

Then commit and push. [This](https://github.com/cowprotocol/flash-loan-wrapper-solver/pull/1)
is an example PR where this workaround helped.
