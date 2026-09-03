# TunnelQuiz pricing page

A standalone static page for the TunnelQuiz pricing model. Single file, no build step -
`index.html` carries its own CSS and JS, and pulls only Google Fonts from the network.

- **`/`** - the customer-facing page. Plans, the proctoring ladder, the full feature sheet,
  top-up prices, and a market switch for India, the US, the UK and the EU.
**A second, unlisted path holds a temporary internal build** with cost-to-serve and
gross-margin figures. It carries `noindex, nofollow` plus a banner saying so, and nothing
links to it - but this repository is public, because free GitHub Pages serves only from a
public repo. **The path is visible in the repo tree and in git history, so treat those
numbers as published rather than hidden.**

It is meant to be short-lived. To remove it:

    git rm -r <internal-path>
    git commit --amend --no-edit
    git push --force-with-lease

Amending rather than adding a delete commit is what keeps the figures out of history.
Anything a crawler, fork or archive has already captured cannot be recalled.

The pricing model this page presents is specified in `product/Pricing-Model.md` in the
main repository. If the two disagree, the specification wins and this page needs updating.
