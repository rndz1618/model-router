# Publishing to GitHub

Maintainer notes for putting this repo on GitHub as `rndz1618/model-router`.

## 1. Set your git identity (once)

Use the GitHub noreply address so commits link to your profile without exposing a private email:

```bash
git config --global user.name "rndz1618"
git config --global user.email "221454253+rndz1618@users.noreply.github.com"
```

To make GitHub reject pushes that expose a private email, enable "Block command line pushes that expose my email" in Settings > Emails.

## 2. Check the repo before pushing

```bash
git status
git log --format='%h %an <%ae> %s'
git ls-files
```

The author should be `rndz1618 <221454253+rndz1618@users.noreply.github.com>`. The tracked files should be only the project files, with no keys or tokens.

## 3. Create the repo and push

With the GitHub CLI:

```bash
gh auth login
gh repo create rndz1618/model-router --public --source=. --remote=origin --push \
  --description "Agent skill that maps roles to free OpenCode models, with open-source status and fallbacks"
```

With plain git: create an empty public repo named `model-router` on github.com (no README, no license, no .gitignore), then:

```bash
git remote add origin https://github.com/rndz1618/model-router.git
git push -u origin main
```

## 4. Set up the repo page

```bash
gh repo edit rndz1618/model-router \
  --add-topic opencode --add-topic agent-skills --add-topic llm \
  --add-topic open-source --add-topic ai-agents
```

In the repo settings, also confirm that the license is detected as MIT and that Issues are enabled.

## 5. Tag the first release

```bash
git tag -a v0.1.0 -m "v0.1.0"
git push origin v0.1.0
gh release create v0.1.0 --title "v0.1.0" --notes "Initial release. See CHANGELOG.md."
```

## 6. Releasing later

1. Edit the skill.
2. Add an entry to `CHANGELOG.md`.
3. Commit, tag the new version, push the tag, and create the release as above.
