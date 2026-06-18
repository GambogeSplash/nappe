# Taste

A design-engineering standard for Claude. Drop it into a project and Claude does front-end and design work to a senior bar: considered visuals instead of generic defaults, motion that feels right, real accessibility, and a habit of looking at the rendered result before calling it done.

It teaches judgment, not a fixed house style. A house style copied into every project is just a new kind of generic, so this gives Claude the principles a senior design engineer reasons with, plus the specific numbers (motion durations, contrast ratios, hit targets, Core Web Vitals) to aim at.

## Why

Given a UI task with no direction, language models converge on the same look: a default SaaS sans, a purple gradient on white, evenly rounded cards, a centered hero, a sparkle icon on anything related to AI. It is competent and forgettable. This standard pushes past that average, and it bakes in the one habit most AI design work skips: render it, screenshot it, compare it to the intent, and fix the gap.

## What is inside

```
skills/design-engineering/SKILL.md   The standard itself, ready to use as a skill
.claude-plugin/plugin.json           Plugin manifest, for installing as a Claude plugin
README.md                            This file
LICENSE                              MIT
```

The standard covers how to operate (frame first, kill the generic default, verify with your own eyes, finish the sweep), avoiding generic design, motion and animation, micro-interactions, interaction details, visual craft, accessibility, performance, and a pre-ship checklist.

## Install

Pick whichever fits how you use Claude. All three are portable across stacks.

**1. As a skill (recommended).** Loads only when the work is actually design or front-end, so it never bloats unrelated sessions. Copy the skill folder into your project (or your home config):

```bash
# project-local
cp -r skills/design-engineering .claude/skills/

# or for every project on your machine
cp -r skills/design-engineering ~/.claude/skills/
```

Claude picks it up automatically when you ask for UI, frontend, components, animation, or visual design.

**2. As an always-on rule.** Put it where Claude reads project rules, optionally scoped to front-end files so it only loads when relevant:

```bash
cp skills/design-engineering/SKILL.md .claude/rules/design-engineering.md
```

**3. From CLAUDE.md.** Keep `CLAUDE.md` short and import the standard so the detail lives in one place:

```markdown
@skills/design-engineering/SKILL.md
```

**4. As a Claude plugin.** This repo ships a `.claude-plugin/plugin.json` manifest, so it can be installed as a plugin and the skill is registered automatically. Point your plugin install at this repo.

## Using it

Once installed, work normally. Ask Claude to build, review, or refine an interface and it will apply the standard, including the render-and-compare verification loop. To pull it in explicitly, mention design, frontend, UI, animation, or visual work, or reference the skill by name.

## Extending it

Keep it focused. Add a rule when you have actually watched the work go wrong, not speculatively, and cut any line whose removal would not change the output. Long always-loaded files dilute the rules that matter, which is exactly why the detail lives in a skill that loads on demand.

## License

MIT. See [LICENSE](LICENSE). Use it, fork it, adapt it to your own taste.

By [John Wright-Nyingifa](https://twitter.com/itriple9) ([@itriple9](https://twitter.com/itriple9)).
