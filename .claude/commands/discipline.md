---
command: discipline
description: Toggle discipline mode — auto-invoke verification, brainstorming-before-coding, and review checks
---

# Discipline Mode

Toggle automatic skill invocation for development discipline.

## Usage

```
/octo:discipline on     — enable auto-invoke discipline checks
/octo:discipline off    — disable (back to manual invoke only)
/octo:discipline status — show current state
```

## What Discipline Mode Does

When **on**, you MUST follow these rules automatically — no user prompt needed:

### Before Implementation (brainstorm gate)
**Before writing ANY code or making changes**, check:
- Has the approach been discussed/planned? If not, invoke `skill-thought-partner` or `skill-writing-plans`
- Is the scope clear? If not, ask clarifying questions first
- This applies even for "simple" changes — simple is where unexamined assumptions cause the most waste

### Before Completion Claims (verification gate)
**Before saying "done", "fixed", "passing", or committing**, invoke `skill-verification-gate`:
- Run the actual verification command
- Read the full output
- Only claim success with evidence in hand

### After Implementation (review gate)
**After completing any non-trivial code change**, automatically:
- Run spec compliance check: does the change match what was asked?
- Run code quality review via subagent
- Flag issues before presenting results

### When Receiving Feedback (response gate)
**When receiving code review feedback or error reports**, invoke `skill-review-response`:
- Verify the feedback against actual code before implementing
- Never blindly agree — evaluate technically first

### When Debugging (investigation gate)
**When encountering ANY bug, error, or test failure**, invoke `skill-debug`:
- Root cause investigation before proposing fixes
- No guessing, no "try this and see"

## How It Works

When the user runs `/octo:discipline on`, persist the setting:

```bash
mkdir -p ~/.claude-octopus/config
echo "OCTOPUS_DISCIPLINE=on" > ~/.claude-octopus/config/discipline.conf
```

The SessionStart hook reads this file and injects the discipline directive into the session context. The directive is ~30 lines (not 200+) — lightweight enough to not bloat context.

When off:
```bash
echo "OCTOPUS_DISCIPLINE=off" > ~/.claude-octopus/config/discipline.conf
```

## What Discipline Mode Does NOT Do

- Does not add new commands or skills — uses existing ones
- Does not slow down quick tasks — `/octo:quick` bypasses discipline checks
- Does not force multi-provider workflows — discipline is about rigor, not providers
- Does not fire on every single turn — only at the 5 gates above

## Execution Contract

When the user invokes `/octo:discipline`:

1. Parse the argument: `on`, `off`, or `status`
2. For `on`: write config file, confirm with banner
3. For `off`: write config file, confirm
4. For `status`: read config file, display current state
5. No args: show status

```bash
DISCIPLINE_CONF="${HOME}/.claude-octopus/config/discipline.conf"
mkdir -p "$(dirname "$DISCIPLINE_CONF")"

case "${1:-status}" in
    on)
        echo "OCTOPUS_DISCIPLINE=on" > "$DISCIPLINE_CONF"
        echo "🐙 Discipline mode: ON"
        echo "  ✓ Brainstorm gate — plan before coding"
        echo "  ✓ Verification gate — evidence before claims"
        echo "  ✓ Review gate — check after implementing"
        echo "  ✓ Response gate — verify before agreeing"
        echo "  ✓ Investigation gate — root cause before fixing"
        ;;
    off)
        echo "OCTOPUS_DISCIPLINE=off" > "$DISCIPLINE_CONF"
        echo "🐙 Discipline mode: OFF — manual skill invocation only"
        ;;
    status|"")
        if [[ -f "$DISCIPLINE_CONF" ]] && grep -q "OCTOPUS_DISCIPLINE=on" "$DISCIPLINE_CONF" 2>/dev/null; then
            echo "🐙 Discipline mode: ON"
        else
            echo "🐙 Discipline mode: OFF"
        fi
        ;;
esac
```
