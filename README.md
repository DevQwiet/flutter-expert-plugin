# flutter-expert-plugin

Claude Code plugin — Flutter 3.x + Riverpod + Supabase expert agent and skill.

## Install

```bash
npx skills add evannarozny/flutter-expert-plugin
```

## What's included

### Agent: `flutter-expert`
A Claude Code subagent specialized in:
- Flutter 3.6+ best practices (official Flutter team rules)
- Riverpod 2.x codegen patterns and common pitfalls
- Supabase Flutter 2.x integration
- Google Sign-In 7.x nonce flow
- Flutter 3.27+ API migration (`.withValues` over `.withOpacity`, `SharePlus.ShareParams`)
- Accessibility and performance patterns

Use it for: adding screens, fixing providers, implementing auth, fixing analyzer warnings.

### Skill: `flutter-riverpod`
Inline skill that enforces Flutter + Riverpod conventions within the current agent context.

Triggers on: `flutter`, `riverpod`, `supabase flutter`, `ConsumerWidget`, `AsyncNotifier`, `go_router`, `Google Sign-In`, `share_plus`.

## Manual install

Copy files to your Claude config directories:

```bash
# Agent
cp agents/flutter-expert.md ~/.claude/agents/flutter-expert.md

# Skill
cp -r skills/flutter-riverpod ~/.claude/skills/flutter-riverpod
```

Restart Claude Code after copying.
