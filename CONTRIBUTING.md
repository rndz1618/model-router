# Contributing

Thanks for helping keep the catalog accurate. Free model lineups change often, so small, well-sourced PRs are the most useful.

## Updating the model catalog

1. Run the live check from Step 1 of `SKILL.md` and note which free IDs exist today.
2. For each added or changed row, link a **primary source** in the PR description: the vendor announcement or the Hugging Face model card.
3. Set the "Open" column using these rules:
   - ✅ only if weights are downloadable and the license is stated on the model card.
   - ❓ if you cannot verify it. Do not guess.
   - ❌ if the model is closed or its identity is undisclosed.
4. Mark benchmark numbers as vendor claims unless an independent source reports them.
5. Remove models that no longer appear in the live check, and note it in `CHANGELOG.md`.

## Other changes

- Keep `SKILL.md` self-contained and under ~200 lines where possible.
- Do not add API keys, tokens, or personal paths anywhere.

## Reporting a problem

Open an issue with: OpenCode version, the output of the Step 1 command, and what the skill produced.
