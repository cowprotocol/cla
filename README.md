# Nomev Labs Contributor License Agreement

This repository contains the Nomev Labs CLA.
The full document can be found [here](./Nomev%20Labs%20CLA.md).

## GitHub Action Setup

It is possible to setup a GitHub action to automatically check for CLA
signatures in a repository.

The following steps describe how to enable a ["lite" version of the CLA Assistant](https://github.com/contributor-assistant/github-action)
and automatically exempt team members and bots.

1. Add a GitHub Action workflow file at `.github/workflows/cla.yml`:
    ```yml
    name: "cla"

    on:
      issue_comment:
        types: [created]
      pull_request_target:
        types: [opened, closed, synchronize]

    jobs:
      cla:
        runs-on: ubuntu-latest
        steps:
          - name: "Get Team Members"
            id: team
            uses: actions/github-script@v6
            with:
              github-token: ${{ secrets.ORG_TOKEN }}
              result-encoding: string
              script: |
                const members = await github.paginate(
                  github.rest.orgs.listMembers,
                  { org: "cowprotocol" },
                );
                return members.map(m => m.login).join(",");

          - name: "CLA Assistant"
            if: (github.event.comment.body == 'recheck' || github.event.comment.body == 'I have read the CLA Document and I hereby sign the CLA') || github.event_name == 'pull_request_target'
            uses: contributor-assistant/github-action@v2.1.3-beta
            env:
              GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
              PERSONAL_ACCESS_TOKEN : ${{ secrets.ORG_TOKEN }}
            with:
              branch: 'cla-signatures'
              path-to-signatures: 'signatures/version1/cla.json'
              path-to-document: 'https://github.com/cowprotocol/cla/blob/main/Nomev%20Labs%20CLA.md'
              allowlist: '${{ steps.team.outputs.result }},*[bot]'
    ```
2. Create the branch `cla-signatures` where the signatures JSON will be updated:
    ```sh
    git switch -f --orphan cla-signatures
    git commit -m "CLA signatures" --allow-empty
    git push -u origin cla-signatures
    ```
3. Profit $$$
