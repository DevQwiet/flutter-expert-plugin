# flutter-expert-plugin

![cover](./cover.png)

Claude Code plugin — Flutter 3.x + Riverpod + Accessibility expert agent and skill.

## Install

```bash
npx skills add DevQwiet/flutter-expert-plugin
```

## What's included

### Agent: `flutter-expert`
A Claude Code subagent specialized in:
- Flutter 3.6+ best practices (official Flutter team rules)
- Riverpod 2.x codegen patterns and common pitfalls
- Flutter 3.27+ API migration (`.withValues` over `.withOpacity`, `SharePlus.ShareParams`)
- WCAG AA accessibility standards (touch targets, contrast, semantics, animation guards)
- Widget composition, performance patterns, Dart modern syntax

Use it for: adding screens, fixing providers, fixing analyzer warnings, accessibility audits.

### Skill: `flutter-riverpod`
Inline skill that enforces Flutter + Riverpod + Accessibility conventions within the current agent context.

Triggers on: `flutter`, `riverpod`, `ConsumerWidget`, `AsyncNotifier`, `go_router`, `widget`, `accessibility`, `a11y`, `semantics`.

## Manual install

Copy files to your Claude config directories:

```bash
# Agent
cp agents/flutter-expert.md ~/.claude/agents/flutter-expert.md

# Skill
cp -r skills/flutter-riverpod ~/.claude/skills/flutter-riverpod
```

Restart Claude Code after copying.
