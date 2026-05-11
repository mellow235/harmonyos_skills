# AGENTS.md - HarmonyOS 6.0 Development

## Project Overview
This is a comprehensive HarmonyOS 6.0 development skill repository with 22 modules covering all aspects of ArkTS/ArkUI app development.

## Skill Loading
- OpenCode: `.opencode/skills/harmonyos-dev/SKILL.md`
- Claude/Cline: `.claude/skills/harmonyos-dev/SKILL.md`
- Agent-compatible: `.agents/skills/harmonyos-dev/SKILL.md`
- Quick reference: `QUICK-REFERENCE.md`

## When to use this skill
Activate this skill when:
- Writing HarmonyOS application code (ArkTS/ArkUI)
- Implementing state management patterns (@State, @Prop, @Link, etc.)
- Building UI components or layouts
- Creating animations or transitions
- Setting up routing or navigation
- Working with data storage (preferences, DB, file)
- Making network requests
- Handling permissions or device features
- Using 3D graphics or AI/ML features
- Debugging or testing HarmonyOS apps

## Key Rules
1. Always use `@Entry` + `@Component struct` for pages
2. Use `$` prefix for `@Link` bindings
3. Paginate with `router.pushUrl` or `Navigation`
4. Import preferences from `@kit.ArkData`, HTTP from `@kit.NetworkKit`
5. Test with `@ohos/hypium`
6. Hide internals with `private`, expose with `public`
7. Always destroy HTTP request objects in `finally` blocks
8. Call `flush()` after preferences writes
9. Check permissions before accessing protected resources
10. Use `LazyForEach` for long lists, not `ForEach`

## Module Structure
When user asks about a specific topic, reference the corresponding module folder:
- ArkTS basics → `01-ArkTS-Basic/`
- UI development → `02-UI-Development/`
- State management → `03-State-Management/`
- Components → `04-Components/`
- Layout → `05-Layout/`
- Animation → `06-Animation/`
- Router → `07-Router/`
- Storage → `08-Storage/`
- Network → `09-Network/`
- Device → `10-Device/`
- Permission → `11-Permission/`
- Testing → `12-Testing/`
- 3D Graphics → `13-3D-Graphics/`
- AI Engine → `14-AI-Engine/`
- Concurrency → `15-Concurrency/`
- Runtime/GC → `16-Runtime-and-GC/`
- Native Extension → `17-Native-Extension/`
- Security → `18-Security/`
- Version Diff → `19-Version-Differences/`
- Project Engineering → `20-Project-Engineering/`
- App Architecture → `21-App-Architecture/`
- Quality Checklist → `22-Quality-Checklist/`
