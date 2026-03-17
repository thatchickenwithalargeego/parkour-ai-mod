# Parkour AI Mod 🤖

A Fabric (client-side) mod that adds a **reinforcement-learning bot** to your Minecraft client. The bot learns to complete parkour courses on public servers — and you train it in real time by giving feedback, or by playing the course yourself for it to imitate.

---

## How it works

The AI uses **Q-Learning**, a classic reinforcement learning algorithm:

- The bot observes its *state*: is it on the ground? Is there a block ahead? How far from the goal?
- It picks an *action*: sprint, jump, turn, etc.
- It receives a *reward*: positive for moving toward the goal or gaining height, negative for falling or getting stuck.
- Over thousands of steps it builds up a table of "which action is best in each situation."

**You** can inject manual rewards at any time — `/ai good` or `/ai bad` — to guide learning. You can also let it watch you play the course first (teach mode) to give it a head start.

Learning is saved automatically every 500 steps (`config/parkouraid_qtable.bin`) and persists across sessions.

---

## Requirements

- Minecraft **1.21.1**
- [Fabric Loader](https://fabricmc.net/use/installer/) 0.16.5+
- [Fabric API](https://modrinth.com/mod/fabric-api) 0.102.0+
- Java 21+

---

## Installation

1. Install Fabric for Minecraft 1.20.4
2. Download Fabric API and put it in your `mods/` folder
3. **Build this mod** (see below) and put the resulting `.jar` in `mods/`
4. Launch Minecraft — you'll see `[ParkourAI] Ready!` in the log

### Building from source

```bash
./gradlew build
# Jar is in build/libs/parkour-ai-1.0.0.jar
```

---

## Quick start guide

### Step 1 — Join a server with a parkour course

Join any public Minecraft server that has a parkour map. Walk up to the start.

### Step 2 — Set the goal

Stand at the **end** of the parkour (or wherever you want the bot to try to reach), then type:

```
/ai goal
```

This saves your current position as the goal. Alternatively use coordinates:

```
/ai goal 120 65 -340
```

### Step 3 — Teach the bot (recommended first)

Walk back to the start. Type `/ai teach`, then **play the parkour yourself** as best you can. The bot records every move you make. When done:

```
/ai learn
```

This injects your movement data directly into the bot's brain as a positive example.

### Step 4 — Let the bot run

```
/ai start
```

The bot now controls your character. Watch it attempt the parkour. It will fail a lot at first — that's normal and expected. Each failure teaches it something.

### Step 5 — Give feedback

While the bot is running, you can coach it:

| You type | Effect |
|---|---|
| `/ai good` | Rewards the last action it took |
| `/ai bad` | Punishes the last action (e.g. it jumped the wrong way) |
| `/ai checkpoint` | Manual bonus reward (e.g. it landed a hard jump) |

### Step 6 — Check progress

```
/ai status
```

Shows exploration rate (ε), Q-table size, steps taken. As ε approaches 0.05 the bot is mostly exploiting its learned knowledge rather than exploring randomly.

---

## All commands

| Command | Description |
|---|---|
| `/ai help` | Show all commands |
| `/ai start` | AI takes control |
| `/ai stop` | You take control back |
| `/ai good` | +reward for last action |
| `/ai bad` | −reward for last action |
| `/ai teach` | Start recording your play |
| `/ai learn` | Stop teaching, absorb lesson |
| `/ai goal` | Set goal to current position |
| `/ai goal x y z` | Set goal to coordinates |
| `/ai checkpoint` | Manual checkpoint reward |
| `/ai status` | Stats printout |
| `/ai save` | Force-save Q-table |
| `/ai reset` | Wipe all learning |

---

## Important notes for public servers

- **This is a client-side mod** — the server does not need to have it installed.
- The bot controls YOUR character (your account). It does not use a separate bot account.
- Some servers have anti-cheat that detects unnatural movement patterns. Start with a permissive server or one that explicitly allows bots/clients.
- The bot will not respond to server chat, open GUIs, or do anything except move.
- Long sessions may trigger AFK kick — the bot's movement usually prevents this.

---

## How the AI improves over time

| Phase | What happens |
|---|---|
| Early (ε ≈ 0.9) | 90% random exploration — chaotic and clumsy |
| Mid (ε ≈ 0.4) | Mix of exploration and learned patterns |
| Late (ε ≈ 0.05) | Mostly exploiting best-known policy |
| After teach mode | Q-values pre-seeded from your demo — improves much faster |

The Q-table is saved persistently, so you can quit and come back — the bot picks up where it left off.

---

## Troubleshooting

**Bot doesn't move at all** → Check `/ai status`, make sure `Active: YES`. Check that `/ai goal` was set.

**Bot keeps falling off the same block** → Type `/ai bad` right as it makes the bad move. Give it a few attempts and it will learn to avoid that action.

**Bot is too random** → Use teach mode first (`/ai teach`, play it, `/ai learn`). This bootstraps the Q-table.

**Server says "Unknown command /ai"** → The mod intercepts `/ai` commands before they reach the server. If this still happens, you may have a conflicting mod.

---

## Architecture

```
ParkourAIMod          ← Fabric entry point, wires everything together
├── GameState         ← Encodes what the bot sees (11 boolean/int fields)
├── Action            ← 12 discrete actions (move, jump, turn, sprint-jump...)
├── ParkourAgent      ← Q-Learning brain (Q-table, ε-greedy, reward update)
├── BotController     ← Translates actions to key presses, computes auto-rewards
└── TrainerCommands   ← Intercepts /ai chat commands for real-time training
```
