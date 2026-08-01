# OFF TTRPG — System Document

Combat engine and player classes. Encounters live in a separate document.

**Provenance.** Canonical, taken from OFF: the Batter's level 1 stat block, the competence damage formula, the CP cost ladder, the element ring, and the status list. Everything else is designed for this campaign — all growth curves, the five non-Purifier class stat lines, and every Move Power value.

---

# PART 1 — ENGINE

## Stats

| Stat | Full name | Governs |
|---|---|---|
| **HP** | Health Points | Damage a character can take before falling. Restored by items, healing competences, and rest zones. |
| **CP** | Competence Points | Spent to use competences. Restored only by items and rest zones — never by acting, defending, or waiting. |
| **ATK** | Attack | Damage of a normal Attack, before element, DEF, and crit. Also feeds competences that carry an Atk% share. Not a flat damage number on its own. |
| **DEF** | Defence | Percentage damage reduction, applied to attacks and competences alike. Range −25 to +75; negative values amplify incoming damage instead of reducing it. |
| **AGI** | Agility | Gauge fill rate. `gaugeSeconds = 40 / AGI`. Affects nothing else — not accuracy, not evasion, not turn order beyond frequency. |
| **ESP** | Esprit | Competence power. Feeds any competence carrying an Esp% share; a class with high ESP relative to ATK gains more from competences than from attacking. |
| **LCK** | Luck | Crit chance as a flat percentage, rolled once per gauge cycle. A crit doubles final damage. |
| **RES** | Resistance | Does two jobs: subtracted from a status's landing chance, and rolled each turn as the chance to shake an active status. |

**Derived values.** Two numbers are computed from the above rather than tracked separately:

| Derived | From |
|---|---|
| Gauge seconds | `40 / AGI`, floor 1.0 |
| Accuracy baseline | `100 − level 1 AGI`, fixed for the whole campaign |

## Damage

```
raw    = (BaseValue + ATK×Atk% + ESP×Esp%) × MovePower × (1 ± Var)
final  = raw × element × (1 − DEF/100) × crit
```

- **Accuracy** 85–95 across every attack in the game. Misses deal nothing. See below — player accuracy is anchored per class, not picked per move.
- **Crit** = LCK% chance, ×2 applied to the final value. **Rolled during gauge fill, not on use** — see below.
- **Element** = 2.0 / 1.0 / 0.5.
- **DEF ranges from −25 to +75.** All DEF modifiers are **flat point additions**, never multipliers.

**DEF is written in points, not percent.** DEF is already a percentage, so "DEF Up 30%" is ambiguous — 30% of the stat, or 30 points? Every DEF change in this system is stated as a flat value: **DEF Up +15**, **DEF Down −20**. ATK and AGI changes remain percentages, because those are raw numbers where a percentage is unambiguous.

**Negative DEF amplifies damage.** The formula is `× (1 − DEF/100)`, so a target at DEF −20 takes 120%. This keeps DEF shred useful against lightly-armoured targets instead of hitting a pointless floor at zero, and it means Gaussian Blur is worth casting on a Spectre as well as a Guardian.

The cap does the safety work. The Burnt at 39, plus Defend's +25 and an ally's Focus at +15, reaches 79 and clamps to 75 — one turn of heavy mitigation costing two party turns to set up, not the sustained near-immunity that multiplicative stacking produced.

A normal Attack is `BaseValue 0, Atk% 100, Esp% 0, MovePower 1.0, Var 10%` — it resolves to plain ATK before element and DEF. Every competence is priced against that.

## Accuracy

```
class accuracy = 100 − (level 1 AGI)
```

| Class | Base AGI | Accuracy |
|---|---|---|
| Epsilon | 5 | **95** |
| Omega | 6 | 94 |
| Burnt | 7 | 93 |
| Alpha | 9 | 91 |
| Purifier | 10 | 90 |
| Bandit | 13 | **87** |

**Uses level 1 AGI, held constant for the whole campaign** — accuracy must not decay as a class levels into being faster.

### Per-competence modifiers

Applied to the class baseline. Everything lands inside 85–95 with no clamping.

**Exactly one category penalty, plus the rider penalty if it applies.** The categories are mutually exclusive — an all-enemies competence takes −2 for being AoE and never also takes the single-target CP-tier penalty, whatever it costs. This is what keeps every value inside 85–95 without clamping.

**Category — pick one:**

| | Modifier |
|---|---|
| Normal Attack | +0 |
| Single-target damage, 14–20 CP | +0 |
| Single-target damage, 22–32 CP | −2 |
| All-enemies damage, any CP | −2 |

**Rider — stacks on the above:**

| | Modifier |
|---|---|
| Carries any attached effect — status, stat change, element change, or steal | −1 |

A rider is something the competence *applies to the target* beyond damage. A property of the attack itself is not a rider and takes no penalty: Perseus Mark ignoring elemental resistance, or Long Exposure doubling against a Blinded target, change how the damage resolves rather than adding an effect on top of it.

**No roll at all:**

| | |
|---|---|
| Non-damaging — heal, revive, buff, debuff, element change | cannot miss |
| Pure status application | uses the resistance tier instead |

Worked examples: Vela Shot is Bandit 87, AoE −2, no rider → **85**. Photographic Blur is Omega 94, AoE −2, Blinded rider −1 → **91**. Corvus Mark is Bandit 87, single-target at 24 CP −2 → **85**. Ursa Shot is Bandit 87, single-target at 16 CP +0, steal rider −1 → **86**.

**A miss consumes CP.** The competence is spent whether or not it connects, which makes an expensive attack a genuine risk rather than a free reroll — and is the main reason the accuracy floor sits at 85 rather than lower. A miss is a resolved action that failed. Clicking an already-dead target is not an action at all — the input is refused outright; see Targeting.

**AoE rolls accuracy per target, not once.** A single roll would make an all-enemies competence all-or-nothing, which at 32 CP is punishing variance. Per-target means an AoE at 93 reliably connects with most of a group and occasionally drops one.

**Nothing that isn't damage can miss.** A missed heal or a missed element change in a CP-scarce system with no between-fight recovery is miserable rather than tense, and pure status competences already roll against the resistance tier — an accuracy roll on top would be double jeopardy.

**A damage competence carrying a status applies both or neither.** If the attack misses, the rider doesn't land; the status only rolls its resistance tier once the hit connects.

The full range stays occupied at both ends: Epsilon's Tragedies at 93–95, and four of the Bandit's eight competences at 85.

**Why slow classes are more accurate.** A miss costs one turn regardless of who takes it, but not one *share*. Over a 90-second fight the Bandit acts 29 times and Epsilon 11, so a single miss is 3.4% of the Bandit's output against 9.1% of Epsilon's — 2.6× more expensive. With only 11 trials, variance doesn't average out either: a three-miss run is a real outcome for Epsilon and statistically almost impossible for the Bandit. The correction softens that gap from 2.58× to 2.36× rather than erasing it.

**Enemy accuracy is hand-set per move**, not derived. Enemy AGI runs from 35 to 200 and would produce nonsense through this formula; the source data already assigns accuracy directly — Common Spectre's Attack at 95%, Pastel's competences at 100%.

**Blinded's −20 points therefore bites unevenly**, taking the Bandit to 67 and Epsilon to 75. That's consistent rather than accidental: the class that already misses more suffers more from inaccuracy, which makes Blinded a soft counter to fast classes.

## The competence formula

**Why this formula and not flat `basePower × ESP`:** Move Power is a per-ability dial independent of class stats, so one competence can be nerfed without touching the class that owns it. The Atk%/Esp% split lets martial competences scale off ATK, which is the only thing that stops them decaying into uselessness by mid-campaign.

**DEF is percentage, not flat subtraction.** OFF's flat model creates hard immunity breakpoints — an enemy at DEF 120 subtracts a fixed amount and anyone under a certain ATK does literally nothing. Percentage scales smoothly and survives any rescaling.

## The Gauge

```
seconds = 40 ÷ AGI     (floor 1.0s)
```

| AGI | Seconds |
|---|---|
| 5 | 8.0 |
| 7 | 5.7 |
| 10 | 4.0 |
| 13 | 3.1 |
| 16 | 2.5 |
| 20 | 2.0 |

AGI 10 = 4.0s is the Batter's canonical level 1 speed and the system's anchor.

**The gauge holds at full.** Once it fills it stops and waits — there is no window, no timer, no penalty for thinking. The sole exception is Madness, which resolves automatically the moment the gauge fills. AGI governs how often your turn *comes up*, not how fast you have to decide once it does.

This is the difference between an action game and a real-time tactics game. Nobody is ever locked out of acting because someone else was slow, and nobody is rushed into a bad call. What creates pressure is that **the fight does not stop while you deliberate** — enemy gauges and your allies' gauges keep filling, so a long pause costs tempo without costing you your turn. Talking is free; taking forever is not.

A rate model is self-correcting: +1 AGI at 5 saves 1.33s, at 13 it saves 0.22s. Slow classes benefit more per point automatically, so AGI growth can be uniform across classes and the spread compresses on its own — 2.6× at level 1 down to 1.8× at level 20.

### Crits are rolled during gauge fill

**The crit roll happens when the gauge starts filling, and the result is shown to the player before they choose.** A charged crit is visible — the gauge fills in a different colour — so the player knows what they are holding.

Rolled once per gauge cycle, not per action. Whatever the character does with that turn, it either crits or it doesn't, and the roll is spent when the turn is spent.

**It applies to any action that deals damage**, whether that is a normal Attack or a competence. Non-damaging actions ignore it, and a charged crit spent on a heal or a buff is simply wasted.

This converts LCK from a passive damage bonus into a resource the player can spend deliberately: bank the crit for a Homerun instead of a normal Attack, or hold it for a target you actually want doubled. A high-LCK character now makes a decision every few seconds that a low-LCK character does not.

**A charged crit applies to every target of an AoE.** One roll, one flag, applied to each target the competence hits. Spending a crit on an all-enemies competence is therefore worth more than spending it on a single target, and against a large group it is the highest-value use of a crit in the system.

**A miss wastes a charged crit against that target only.** Accuracy rolls per target, so an AoE can crit some targets and miss others; the charged flag is spent when the turn is spent regardless.

Enemies roll the same way. Whether their charged crits are visible to players is a presentation choice rather than a rule — hiding them keeps enemy turns threatening, showing them makes the whole fight readable.

## The Four Buttons

**Attack** — ATK, one enemy, no cost.

**Defend** — **+25 DEF flat**. **Restores no CP.**

**Defend expires the moment your gauge next fills, whether or not you act.** Because the gauge holds at full, "until your next turn" would never arrive for a player who simply stops playing — a character could Defend once and sit at +25 DEF for the rest of the fight. Tying expiry to the gauge rather than to the action means the mitigation lasts exactly one gauge cycle and refusing to act buys nothing.

The same applies to any effect worded around a character's own next turn.

Defend restores nothing because any CP return above about 4 makes Defend-then-nuke beat straight attacking outright, while also reducing incoming damage — strictly better on both axes and the dullest possible input pattern.

The bonus is flat rather than a multiplier because doubling would give the Burnt +39 points and the Bandit +22, handing the most benefit to the class that needs it least. Flat makes the button equally valuable to everyone and keeps high-DEF classes off the cap.

**Competence** — spend CP. Costs your turn.

**Objects** — shared party inventory. The only CP source in combat.

## Elements

**Plastic → Metal → Smoke → Meat → Plastic.** Each deals 2× to the next and 0.5× to the previous; the other two are neutral. **Sugar** is neutral in both directions — never exploited, never exploitable.

**Same element against same element is neutral, 1.0×.** An attack only ever multiplies against the element it beats or the element that beats it; every other pairing, including a mirror match, resolves flat.

Confirmed against the Batter's Elemental Runs: Metal is effective against Spectres (so Spectres are Smoke), Smoke against Burnts (Burnts are Meat), Plastic against Beasts (Beasts are Metal).

### Having an element cuts both ways

Every character and every enemy has exactly one element. It determines two things at once:

- **Offence** — an elemental competence deals 2× to the element it beats, 0.5× to the element that beats it.
- **Defence** — you take 2× from the element that beats yours, 0.5× from the element you beat.

**Normal Attacks carry your class element.** So does every damaging competence unless it states otherwise. Matchups are therefore pervasive rather than occasional — walking into a fight against the element that beats yours means dealing half and taking double, a **4× swing in exchange rate**.

This is a change from the source, where elemental damage was special-cased. The payoff is that the party has four distinct modes depending on what it's facing: identifying an enemy's element decides who carries the fight.

**Only damage carries an element. Nothing else does.**

| Elementless | Sugar — deliberately neutral damage |
|---|---|
| All heals and revives | The Purifier's four Homeruns |
| All buffs | |
| All pure debuffs | |
| All pure status applications | |
| Wide Angle | |

The Purifier therefore has three offensive modes: its normal Attack is Metal, its Homeruns are Sugar and never care about matchup, and its Runs pick an element deliberately. Stronger-but-locked against weaker-but-flexible.

| Class | Element | Weak to |
|---|---|---|
| Purifier | Metal | Plastic |
| Alpha | Plastic | Meat |
| Omega | Smoke | Metal |
| Epsilon | Meat | Smoke |
| Bandit | Smoke | Metal |
| Burnt | Meat | Smoke |

**No class *is* Sugar**, but the Purifier's Homeruns deal Sugar damage — the only reliable way any class ignores matchup entirely. On the enemy side Sugar stays rare, matching the source.

**Two elements are doubled, so two weaknesses are shared.** Omega and the Bandit are both Smoke, so both are weak to Metal; Epsilon and the Burnt are both Meat, so both are weak to Smoke. Only the Purifier and Alpha hold an element alone.

**Six classes over four elements means two duplicates, so no element setting is universally good.** Setting an enemy to Plastic or Meat helps two allies and hurts one; Metal or Smoke helps one and hurts two. Any other distribution just moves which half is favoured — that asymmetry is the tension the Burnt plays with, not a flaw to correct.

## Statuses

| Status | Effect |
|---|---|
| **Poisoned** | Loses 1/25 max HP per turn |
| **Blinded** | Accuracy −20 points |
| **Muted** | Cannot use Competence |
| **Palsied** | Gauge fills normally, but on reaching full the turn is consumed with no action and the gauge resets to 0 |
| **Asleep** | Every second gauge fill resolves as Palsied — turn consumed, no action, gauge resets |
| **Furious** | The player picks a target; the engine substitutes a random affordable competence that can hit it |
| **Madness** | No input accepted. Each gauge fill resolves automatically as a normal Attack on a random party member, including itself |
| **Hasty** | Acts twice per gauge fill. A charged crit applies to the first action only; the second never crits |
| **Taunted** | Can only target the character who applied it |

Duration counts **the afflicted character's own turns**. Rounds don't exist in a real-time gauge system.

**Cure check** at the start of each of the target's turns: `RES% + 5 per turn already afflicted`, with a hard cap: at the start of the target's eighth turn afflicted the status clears automatically, with no roll. **One roll per status, each tracking its own escalation counter** — a target holding Poisoned, Blinded and Muted rolls three times a turn, and each counts turns from the moment that particular status landed. Statuses never share a timer and clearing one has no effect on the others. Flat RES alone is geometric — a 10 RES character has a 12% chance of sitting Palsied for 20+ turns, which in real time is minutes of a player watching.

**Palsied consumes turns rather than stopping them, and that's what makes durations work.** Because every duration in the system counts the target's own turns, a status that froze the gauge entirely would never tick down — Palsy would be permanent by accident. Burning the turn keeps the clock running: cure checks fire, Poison ticks, and other statuses expire on schedule. It also means Palsy is worse for a fast character than a slow one, since a high-AGI target burns through more wasted turns in the same wall-clock time.

**Madness and Taunted are mutually exclusive.** Taunted restricts you to one legal target; Madness ignores targeting entirely and strikes a random ally. **Neither can be applied to a character already holding the other** — the application simply fails, exactly as if the target were immune. Whichever landed first runs until it clears normally, and only then can the other be applied.

**Hasty is enemy-side only.** No player class grants it. Doubling actions per gauge fill is the strongest effect in the list — worth more than any damage competence at any cost — so it stays a threat rather than a tool, and there is no competence to look for.

**Taunted is enemy-side only.** No player class applies it. It exists so enemies can force a character to attack the wrong target, which is a problem the party has to solve rather than a tool it holds. It behaves as an ordinary status in every respect — it rolls to land, RES ticks it off, and it does not stack or refresh.

### One instance only

**A target can hold one instance of any given status, one instance of any given stat change type, and one element change.**

- **Statuses.** Reapplying a status the target already has does **nothing at all** — no second instance, no extension, no refresh. Four Palsies is one Palsy.
- **Stat changes.** One instance per stat per direction. The first application holds until it expires — a later ATK Down does nothing while an ATK Down is running, even a stronger one from a more expensive competence or an item.
- **Up and Down of the same stat cancel on contact.** Applying ATK Down to a target already holding ATK Up removes the Up and applies nothing; the incoming change is spent clearing the existing one. The reverse is identical. A target therefore never holds both directions of the same stat at once, and there is no netting, no multiplication, and no stacking to reason about.

  Cancellation costs the attacker a turn to strip a buff, which is a fair trade, and it gives stat changes a counterplay that mirrors Omega's Fixer without requiring a special competence. It also means buffing an ally *before* an enemy debuffs them is genuinely protective, and debuffing an enemy that is about to buff itself is worth doing early.
- **Element changes.** One instance per target — a target's element cannot be rewritten while a change is already running. **A change to the target's own native element cancels instead**, clearing the existing change and restoring it immediately, applying nothing in its place. This mirrors how Up and Down cancel: setting a Meat creature back to Meat costs a turn and undoes whatever was done to it.

  Without cancellation, an enemy-applied element change would be uncounterable for its full duration, and the party's element specialist would be the one class unable to answer it. With it, any Rush or Crash naming the target's native element is the counterplay, and no new competence is needed.
- **The RES escalation counter never resets** while a status is active. It counts from the moment the status landed, regardless of what else is cast.

Without this, every debuff has an infinite combo. Stacked AGI Down drives an enemy to zero turns and the fight is over before it starts; a refreshable Palsy is *cheaper to maintain than to establish*, so one Alpha locks a boss out permanently. Duration refresh is the more dangerous of the two and is why there is no refresh anywhere in the system.

Stat changes carry a duration, so a blocked application isn't a loss — it's a window. You use what's available while the current one runs, and the option reopens when it expires.

### Landing a status

```
landing chance = tier base − target's RES
```

| Tier | Base |
|---|---|
| Light vulnerability | 100 |
| Neutral | 80 |
| Light immunity | 45 |
| Strong immunity | never lands |

Floored at 5% except Strong immunity, which is absolute. Tiers are set per enemy; every enemy block needs **RES and LCK** values, which the public wiki doesn't list because they're code-side.

RES therefore does two jobs — resisting application and shaking it off — and they compound:

| Target | Lands (neutral) | Avg duration | Expected turns afflicted |
|---|---|---|---|
| Bandit, RES 8 | 72% | 4.64 | **3.34** |
| Alpha, RES 18 | 62% | 3.51 | **2.18** |
| Burnt, RES 32 | 48% | 2.59 | **1.24** |

A 2.7× spread across the party from one stat, without making anyone immune.

### Statuses have no duration. Stat changes do.

**Two vocabularies, on purpose:**

- **Statuses** — Poisoned, Blinded, Muted, Palsied, Asleep, Furious, Madness, Hasty, Taunted. No number. They run until the cure check passes, so the same status lasts different lengths on different targets, and a low-RES enemy is genuinely in trouble while a high-RES one shrugs.
- **Stat changes** — ATK/DEF/AGI Up or Down. Fixed timer, **2 turns from competences learned below level 10, 3 turns at level 10 and above.** They never roll to land and never roll to cure.

**A stat change is not a status.** Nothing that counts, cures, or keys off statuses sees them — Alpha's Absolute String counts statuses only, Focus and Sablé Rush cure and prevent statuses only, and an ATK Down neither blocks a status nor is blocked by one. The two systems share a target but never interact.

Stat changes are timed rather than formula-driven because they're the higher-value effect. A party-wide ATK Down is functionally a party-wide DEF buff, applied in one action — that needs to be predictable and bounded rather than something that might last eight turns on an unlucky roll.

**Hasty is the strongest status in the game** — doubling actions beats any damage competence at any cost. It's priced at the top of the ladder wherever it appears, and should be rare on the enemy side.

## Targeting

**The GM chooses enemy targets** from whatever the engine allows. Targeting restrictions are enforced system-side, exactly like Palsied skipping a turn — nobody is tracking or honouring anything by hand, illegal targets simply aren't selectable.

### Invalid targets

**A dead target is not selectable.** In real time a target can die between the moment a player decides and the moment they click. The input is refused — the button does not respond. No action is initiated, so there is nothing to spend and nothing to reverse. The player is still standing at a full gauge choosing what to do.

This is not a refund. It is not a failed action. It is the engine declining to register a click on something that isn't there, the same way a greyed-out button doesn't respond. Implementation follows from that: **validate the target before initiating the action**, never resolve-then-undo.

Contrast with a miss, which is a real action that happens and fails:

| | What happened | CP | Gauge |
|---|---|---|---|
| **Refused click** — target already dead | nothing | untouched | held at full |
| **Miss** — legal target, accuracy roll failed | the action resolved and failed | spent | reset |

These must look different in the UI. A refused click should read as unresponsive — dead targets greyed out, no flash, no sound. A miss should read as a resolved event with a visible MISS. If both produce silence, a player who clicked a corpse and a player who whiffed a 32 CP nuke draw the same wrong conclusion about what the game just did to them.

**AoE resolves against whoever is alive when the click lands.** If three of four enemies remain, Long String hits three. Only a **completely empty target set** refuses the input. A partially depleted group is a legal target, not a rejected one.

### Taunted ends when the applier dies

Taunted restricts you to a single legal target. If that target dies while you're still afflicted, every click you make would be illegal and you would be locked out entirely.

**Taunted therefore ends immediately when the character who applied it dies**, with no cure check. This is the only status that terminates on an external condition, and it exists to prevent a deadlock rather than for flavour.

### Furious and Madness

The two statuses take control away to different degrees, and in real time each needs an answer to who presses the button.

**Furious — the player still clicks.** They pick a target normally; the engine substitutes a random affordable competence **from those that can legally target that selection**, and randomises any choice embedded in it. Petit Four Crash picks its own element; an Elemental Run picks its own. CP is spent normally. If no affordable competence can target their selection, the turn is spent with no effect.

Drawing only from competences that fit the chosen target keeps Furious about *what* you do rather than *who* — the player still points, and only the action scrambles. Pressing a button and watching it come out wrong is the loss of control the status represents, and it keeps the player physically in the fight.

**Madness — the player does not click at all.** The character is gone. When the gauge fills it **resolves automatically**: a normal Attack against a random party member, which may be the afflicted character itself. No menu opens, no input is accepted, and the gauge resets and begins refilling immediately.

Two things follow, both deliberate:

- **No competences, ever.** Madness is Attack-only. A status that only redirected damaging actions would be nothing at all to Omega or the Burnt, whose kits are almost entirely non-damaging — they would keep buffing and curing on schedule while a Madness-afflicted Purifier tore into the party.
- **It cannot be waited out.** The gauge holds at full everywhere else in this system, so a player could otherwise sit and refuse to act until the cure check passed. Madness is the one place the engine acts without input, and that is what makes it a loss of control rather than an inconvenience.

Madness is therefore the most punishing status in the game for a support class and the most dangerous for the party when it lands on a striker. Price and resist it accordingly.

## Death

Characters are expected to die mid-combat. Revive items are common but not bulk.

- **Item revives restore 35% HP.** At 1 HP a revived character dies to the next hit and the item bought one turn — that's the spiral.
- **Revived characters return at 50% gauge.** An empty bar is a second hidden penalty that lands hardest on slow classes, who lose more wall-clock time per turn.
- **Competence revives must beat items** — higher HP restored, statuses cured — or they're a CP cost competing with common inventory and losing.

---

# PART 2 — PROGRESSION

## Growth shape

Roughly **3× from level 1 to 20.** HP 100 → 300, ATK 21 → 60 for the Purifier.

Growth **decelerates** — large early gains, tapering late. Base increment per stat, weighted:

| Levels | Weight |
|---|---|
| 2–5 | ×1.5 |
| 6–10 | ×1.2 |
| 11–15 | ×1.0 |
| 16–20 | ×0.8 |

Total across the campaign is 21 base increments, so `increment = (level 20 target − level 1 value) ÷ 21`.

**CP grows linearly** — competence costs are fixed for all twenty levels, so CP buys more casts, never bigger ones. **AGI grows at milestones only** (levels 5, 9, 13, 17, 20) and is the rarest stat in the game, in level-ups and in item bonuses alike.

**Why flat and not exponential:** at 3× growth a 2× element multiplier outweighs ten levels of stat gain, so elements stay the primary tactical layer for the whole campaign.

The ceiling is anchored to the source. Carnival, OFF's optional endgame superboss, has **1,450 HP in the original game** — that figure is canon, not a target for this campaign, and every enemy here will be restatted from scratch. It is cited only to fix the order of magnitude: for a fight of that scale to be threatening, endgame characters sit near 250–350 HP, which is 3× growth rather than 14×.

## Stat checkpoints

Values at levels 1 / 5 / 10 / 15 / 20.

### HP

| Class | 1 | 5 | 10 | 15 | 20 |
|---|---|---|---|---|---|
| Burnt | 120 | 194 | 268 | 330 | 380 |
| Purifier | 100 | 157 | 214 | 262 | 300 |
| Alpha | 85 | 133 | 181 | 221 | 253 |
| Epsilon | 85 | 133 | 181 | 221 | 253 |
| Omega | 80 | 125 | 170 | 208 | 238 |
| Bandit | 70 | 109 | 148 | 181 | 207 |

### ATK

| Class | 1 | 5 | 10 | 15 | 20 |
|---|---|---|---|---|---|
| Purifier | 21 | 32 | 43 | 52 | 60 |
| Bandit | 18 | 28 | 38 | 46 | 52 |
| Alpha | 16 | 24 | 32 | 39 | 44 |
| Epsilon | 15 | 22 | 29 | 35 | 40 |
| Burnt | 14 | 21 | 28 | 34 | 38 |
| Omega | 11 | 16 | 21 | 25 | 28 |

### ESP

| Class | 1 | 5 | 10 | 15 | 20 |
|---|---|---|---|---|---|
| Alpha | 26 | 39 | 52 | 63 | 72 |
| Omega | 26 | 39 | 52 | 63 | 72 |
| Epsilon | 24 | 36 | 48 | 58 | 66 |
| Purifier | 21 | 31 | 41 | 49 | 55 |
| Burnt | 18 | 27 | 35 | 42 | 48 |
| Bandit | 16 | 23 | 31 | 37 | 42 |

### DEF (percent)

| Class | 1 | 5 | 10 | 15 | 20 |
|---|---|---|---|---|---|
| Burnt | 18 | 24 | 30 | 35 | 39 |
| Purifier | 13 | 18 | 23 | 28 | 32 |
| Omega | 12 | 17 | 22 | 26 | 30 |
| Epsilon | 10 | 15 | 20 | 24 | 27 |
| Alpha | 9 | 14 | 18 | 22 | 25 |
| Bandit | 7 | 11 | 15 | 19 | 22 |

### AGI (gauge seconds in parentheses)

Milestone growth at levels 5, 9, 13, 17, 20.

| Class | 1 | 5 | 10 | 15 | 20 |
|---|---|---|---|---|---|
| Bandit | 13 (3.1) | 14 (2.9) | 15 (2.7) | 17 (2.4) | 18 (2.2) |
| Purifier | 10 (4.0) | 11 (3.6) | 12 (3.3) | 14 (2.9) | 15 (2.7) |
| Alpha | 9 (4.4) | 10 (4.0) | 11 (3.6) | 13 (3.1) | 14 (2.9) |
| Burnt | 7 (5.7) | 8 (5.0) | 9 (4.4) | 11 (3.6) | 12 (3.3) |
| Omega | 6 (6.7) | 7 (5.7) | 8 (5.0) | 10 (4.0) | 11 (3.6) |
| Epsilon | 5 (8.0) | 6 (6.7) | 7 (5.7) | 9 (4.4) | 10 (4.0) |

### CP (linear)

| Class | 1 | 5 | 10 | 15 | 20 |
|---|---|---|---|---|---|
| Burnt | 48 | 75 | 108 | 141 | 175 |
| Omega | 45 | 75 | 105 | 135 | 165 |
| Alpha | 42 | 70 | 98 | 127 | 155 |
| Epsilon | 42 | 70 | 98 | 127 | 155 |
| Purifier | 35 | 58 | 80 | 103 | 125 |
| Bandit | 28 | 46 | 64 | 82 | 100 |

At level 20, 155 CP buys Alpha roughly 5–11 competences depending on tier. That is the intended scarcity — the screenshot's 113 CP is 3.5 Ultimate Homeruns.

### LCK (crit %)

| Class | 1 | 5 | 10 | 15 | 20 |
|---|---|---|---|---|---|
| Bandit | 15 | 19 | 22 | 24 | 25 |
| Alpha | 10 | 13 | 15 | 17 | 18 |
| Purifier | 8 | 11 | 13 | 14 | 15 |
| Omega | 5 | 7 | 8 | 9 | 10 |
| Epsilon | 5 | 7 | 8 | 9 | 10 |
| Burnt | 3 | 5 | 6 | 7 | 8 |

### RES (status cure % per turn)

| Class | 1 | 5 | 10 | 15 | 20 |
|---|---|---|---|---|---|
| Burnt | 20 | 24 | 28 | 30 | 32 |
| Purifier | 15 | 19 | 22 | 24 | 25 |
| Omega | 15 | 19 | 22 | 24 | 25 |
| Alpha | 10 | 14 | 17 | 19 | 20 |
| Epsilon | 10 | 14 | 17 | 19 | 20 |
| Bandit | 8 | 12 | 15 | 17 | 18 |

---

# PART 3 — COMPETENCE PRICING

## The CP ladder

Taken directly from the Batter's real cost list.

| CP | Tier |
|---|---|
| 2 | Pure utility, no combat effect |
| 14 | Baseline damage in your own element |
| 16 | Baseline damage in a chosen or off-class element |
| 18 | Single-target heal, or damage with a rider |
| 20 | Average damage, or a status |
| 22 | Strong single heal, or party debuff |
| 24 | Party heal, or strong status |
| 26 | High damage |
| 30 | Revive, or party-wide buff |
| 32 | Ultimate |

The spread is only **2.3×** (14 → 32), not the 5× I originally used. Three consequences:

**There is no cheap spam option.** A 2-to-10 ladder makes the cheapest competence the most CP-efficient by a wide margin, so players spam it and hoard the rest. At 14–32 that behaviour doesn't exist.

**Healing costs more than damage.** Save First Base 18 against Furious Homerun 14; revive 30 against Magic Homerun 26. Preserved throughout.

**Elemental attacks are flat-priced.** All four of the Batter's Runs cost exactly 16. No element is cheaper, so the choice is purely matchup.

## Deriving Move Power

A competence's strength is expressed as a **target multiplier** against that class's own normal Attack.

| CP | Target multiplier | Per-target if all enemies |
|---|---|---|
| 14 | 1.6× | 0.85× |
| 16 | 1.5× + element | 0.80× + element |
| 20 | 2.1× | 1.15× |
| 22 | 2.3× | 1.25× |
| 24 | 2.6× | 1.40× |
| 26 | 2.9× | 1.55× |
| 32 | 3.8× | 2.05× |

```
MovePower = targetMultiplier × ATK ÷ PowerBase
PowerBase = BaseValue + ATK×Atk% + ESP×Esp%
```

Evaluate at **level 10**, then hold Move Power fixed for the whole campaign.

Because each class's ESP/ATK ratio is near-constant across levels, this holds its value automatically:

| Class | ESP÷ATK @1 | @20 | A 100%-Esp competence is worth |
|---|---|---|---|
| Omega | 2.36 | 2.57 | ~2.5 attacks |
| Alpha | 1.63 | 1.64 | ~1.6 attacks |
| Epsilon | 1.60 | 1.65 | ~1.6 attacks |
| Burnt | 1.29 | 1.26 | ~1.3 attacks |
| Purifier | 1.00 | 0.92 | ~1.0 attacks |
| Bandit | 0.89 | 0.81 | ~0.85 attacks |

**This is where the martial/caster split lives** — in stat ratios, not special rules. Omega's competences are worth 2.5 attacks, so it should almost never press Attack. Bandit's are worth 0.85, so its competences must carry riders or high Atk% to justify themselves.

### What the AoE column means

Per-target AoE sits at roughly **54% of the single-target value at the same CP**. No encounter size is assumed — the ratio scales on its own:

| Enemies | AoE output vs single-target at the same cost |
|---|---|
| 1 | 0.54× |
| 2 | 1.08× |
| 3 | 1.62× |
| 4 | 2.16× |
| 6 | 3.24× |

Break-even is just under 2. Below that, spending CP on AoE is a mistake; above it, the advantage compounds with no ceiling. That swing **is** the AoE identity, and it needs no encounter-size rule to work.

### Stat change magnitudes

DEF is stated in flat points because it is already a percentage. ATK and AGI are percentages because they are raw numbers spanning a huge range — a flat −10 ATK is a quarter of a Common Spectre and a sixteenth of Pastel.

| CP | ATK one / all | AGI one / all | DEF one / all |
|---|---|---|---|
| 14–18 | 20% / 15% | 15% / 10% | ±15 / ±10 |
| 20–22 | 25% / 20% | 20% / 15% | ±20 / ±15 |
| 24–26 | 30% / 25% | 25% / 20% | ±25 / ±20 |
| 30–32 | 35% / 30% | 30% / 25% | ±30 / ±25 |

**AGI sits one tier below ATK** at the same cost. An AGI change of X% alters how often a target acts by X%, which moves its damage exactly as an ATK change would — and also its statuses, its heals, and everything else it might have done.

**A competence carrying several stat changes discounts each**, the same way damage discounts for riders. Total Drama grants three at once and prices each roughly a tier down.

Sanity anchor: a mitigation competence should not outperform the healer at the same CP. Save Second Base restores 151 HP for 22 CP at level 20; a 24 CP ATK Down should prevent something near that over its life, not double it.

**Damage that carries a rider pays for it in damage.** A competence dealing full tier value *and* applying a status or stat change is underpriced — the rider is free. The standard is roughly **40% off the damage tier for one rider, 50% for two**. The Burnt's Nougat Crash deals 0.75× where its 22 CP tier grants 1.25×; Omega's Blurs are discounted to match.

**Status competences price one tier above their damage.** Palsied is a full stop and Hasty doubles actions; both belong at 30–32 regardless of what damage they carry. **Asleep prices at 25** — it costs the target half its actions, so it sits between the reliable single-effect statuses and a full stop. Furious and Madness are chaotic rather than reliably strong and price one tier down.

---

# PART 4 — THE CLASSES

Six classes. Competences unlock at the listed level; passives at 2 and 10.

Move Power values are computed at level 10 using each class's own stats. `Split` is Atk%/Esp%.

---

## THE PURIFIER — Metal

The Batter. Sustained damage, healing, and the only class that picks its element at cast time. Metal by default — it's a metal bat, and Run with Courage is the first Run learned — so it takes 2× from Plastic.

The Elemental Runs are not redundant with the Homeruns — they're the **element selector**. Four attacks at 16 CP, one per element, while Homeruns are non-elemental and hit harder. The per-fight decision is "match the weakness for 2×, or hit harder flat."

| Lv | Competence | CP | Split | Var | MP | Acc | Effect |
|---|---|---|---|---|---|---|---|
| 1 | Wide Angle | 2 | — | — | — | — | Reveal one enemy's stats, element, statuses |
| 1 | Powerful Homerun | 14 | 100/0 | 10% | 1.60 | 90 | One enemy, 1.6× **Sugar** |
| 3 | Save First Base | 18 | 0/100 | 5% | 1.90 | — | One ally, heal |
| 5 | Run with Courage | 16 | 100/0 | 10% | 1.50 | 90 | One enemy, **Metal** |
| 7 | Furious Homerun | 20 | 100/0 | 15% | 2.10 | 90 | One enemy, 2.1× **Sugar** |
| 9 | Run with Grace | 16 | 100/0 | 10% | 1.50 | 90 | One enemy, **Plastic** |
| 11 | Save Second Base | 22 | 0/100 | 5% | 2.75 | — | One ally, large heal |
| 13 | Run with Dementia | 16 | 100/0 | 10% | 1.50 | 90 | One enemy, **Smoke** |
| 15 | Special Homerun | 26 | 100/0 | 15% | 2.90 | 88 | One enemy, 2.9× **Sugar** |
| 17 | Run with Faith | 16 | 100/0 | 10% | 1.50 | 90 | One enemy, **Meat** |
| 18 | Save Third Base | 24 | 0/100 | 5% | 1.60 | — | All allies, heal |
| 19 | Save Fourth Base | 30 | 0/100 | — | — | — | Revive one ally at 50% HP |
| 20 | Ultimate Homerun | 32 | 100/0 | 20% | 3.80 | 88 | One enemy, 3.8× **Sugar** |

**Sacred Mission** (2) — Once per battle, survive lethal damage at 1 HP.
**Purification** (10) — +25% damage against enemies at or below 30% HP.

---

## THE ALPHA — Plastic

Status control. The most dangerous class in the game and the one to watch in playtest.

| Lv | Competence | CP | Split | Var | MP | Acc | Effect |
|---|---|---|---|---|---|---|---|
| 1 | Saturated String | 14 | 0/100 | 10% | 0.98 | 91 | One enemy, 1.6×, **Plastic** |
| 3 | Open Bracket | 18 | 0/100 | — | — | — | Poisoned |
| 5 | Converted String | 20 | 0/100 | 10% | 1.29 | 91 | One enemy, 2.1× **Plastic**, +25% if it holds a status |
| 7 | Closed Bracket | 20 | 0/100 | — | — | — | Blinded + ATK Down 25% (2 turns) |
| 9 | Long String | 24 | 0/100 | 15% | 0.86 | 89 | All enemies, 1.4× each, **Plastic** |
| 12 | Broken Bracket | 24 | 0/100 | — | — | — | Muted |
| 15 | Complete String | 26 | 0/100 | 10% | 1.78 | 89 | One enemy, 2.9× **Plastic** |
| 18 | Impossible Bracket | 32 | 0/100 | — | — | — | Palsied |
| 20 | Absolute String | 32 | 0/100 | 15% | 2.33 | 89 | One enemy, 3.8× **Plastic**, +0.5× per **status** on it — stat changes do not count |

**Chain Mastery** (2) — Targets afflicted by an Alpha status check their cure at **RES − 5**.
**Status Expertise** (12) — Alpha's status competences land at **+10**.
**Corrosion** (18) — Alpha's status competences treat **Light immunity as Neutral**.

Timed to the Bracket ladder: Status Expertise arrives with Broken Bracket, Corrosion with Impossible Bracket. Each higher tier brings the passive that makes it worth its cost — against a neutral RES 20 boss, Impossible Bracket lands 70% rather than 60%, so a 32 CP competence isn't a coin flip.

**Control is already the party contribution.** Palsy removes an enemy's turns, Mute removes its competences, Blind removes a fifth of its hits. Alpha's passives therefore sharpen its own control rather than amplifying the party's damage — they scale with Alpha's actions and CP.

Corrosion only matters against enemies that were resisting. **Strong immunity still blocks absolutely**, so any enemy that shouldn't be controllable stays that way.

---

## THE OMEGA — Smoke

Protection, cures, revival. Highest ESP-to-ATK ratio in the game at 2.5 — it should essentially never press Attack, and its ATK line reflects that.

| Lv | Competence | CP | Split | Var | MP | Acc | Effect |
|---|---|---|---|---|---|---|---|
| 1 | Optimized Blur | 14 | 0/100 | 10% | 0.40 | 93 | One enemy, 1.0× **Smoke** + AGI Down 15% (2 turns) |
| 3 | Focus | 18 | 0/100 | — | — | — | One ally, cure all statuses + **DEF Up +15** (2 turns) |
| 5 | Photographic Blur | 22 | 0/100 | 10% | 0.32 | 91 | All enemies, 0.8× each **Smoke** + Blinded |
| 7 | Depth of Field | 22 | — | — | — | — | All allies, **DEF Up +15** + AGI Up 15% (2 turns) |
| 9 | Gaussian Blur | 20 | — | — | — | — | One enemy, **DEF Down −20** (2 turns) |
| 12 | Long Exposure | 26 | 0/100 | 15% | 0.93 | 92 | One enemy, 2.3× **Smoke**, doubled if Blinded |
| 15 | Multiple Perspective | 30 | — | — | — | — | Revive one ally at 60% HP, cure all statuses |
| 18 | Radial Blur | 32 | 0/100 | 15% | 0.40 | 91 | All enemies, 1.0× each **Smoke** + **DEF Down −20** + AGI Down 25% (3 turns) |

**Fixer** (2) — Whenever Omega cures statuses, it also strips one stat **Down** from the same target, applying nothing in its place. Applies to **Focus** and **Multiple Perspective**.
**Negative Space** (10) — Enemies Blinded by Omega cannot land criticals. A charged crit held by an enemy is lost the moment Blinded lands, and none can be rolled while it holds.

**Negative Space is the only counter to LCK in the system.** It's worth 9–20% incoming damage reduction depending on enemy LCK, and it pays off the combo running through the kit: Photographic Blur blinds a whole group, Long Exposure doubles against a Blinded target, Negative Space turns that same Blinded into crit immunity. Three competences pointing at one status.

**Fixer is the only counterplay to a stat change in the system.** Statuses have Focus and the cure formula; a DEF Down or ATK Down otherwise sits for its full 2–3 turns with nothing anyone can do about it. It strips one Down, not all — Omega already cures every status at once, and doing both without limit would make a single 18 CP competence an answer to any setup an enemy can build.

Both passives are purely defensive, and both do nothing against an enemy that only deals damage.

Multiple Perspective beats item revives on HP restored and status cure, so it earns its 30 CP against common inventory.

---

## THE EPSILON — Meat

AoE and party buffs. Slowest gauge in the game, so every turn must be worth ~2.5 of the Bandit's.

| Lv | Competence | CP | Split | Var | MP | Acc | Effect |
|---|---|---|---|---|---|---|---|
| 1 | Petite Tragedy | 16 | 0/100 | 10% | 0.48 | 93 | All enemies, 0.8× each **Meat** |
| 3 | Comic Drama | 18 | — | — | — | — | All allies, ATK Up 15% (2 turns) |
| 5 | Grand Tragedy | 20 | 0/100 | 10% | 0.69 | 93 | All enemies, 1.15× each **Meat** |
| 7 | Baroque Drama | 20 | — | — | — | — | All allies, **DEF Up +15** (2 turns) |
| 9 | Absolute Tragedy | 24 | 0/100 | 15% | 0.85 | 93 | All enemies, 1.4× each **Meat** |
| 12 | Unrevokable Drama | 22 | — | — | — | — | All allies, AGI Up 15% (3 turns) |
| 15 | Total Drama | 30 | — | — | — | — | All allies, ATK Up 20% / **DEF Up +15** / AGI Up 15% (3 turns) |
| 18 | Final Tragedy | 32 | 0/100 | 15% | 1.24 | 93 | All enemies, 2.05× each **Meat** |

**Artistic Mastery** (2) — Epsilon's Dramas last 1 additional turn.
**Standing Ovation** (10) — While any Drama is active, Epsilon's Tragedies deal +20%.

**Epsilon is penalised twice, and the Dramas are why that's survivable.** Its per-target AoE is 54% of a single-target competence *and* it has the slowest gauge in the game — at level 20 it acts at 4.0s against the Bandit's 2.2s. Stacking those, Epsilon needs roughly **3.4 enemies** before its damage throughput matches a dedicated single-target class.

That makes the Drama line the fallback, not a bonus: against one or two enemies Epsilon is a party buffer whose Tragedies are filler, and against four or more it is the highest-throughput damage in the game. The class changes job with the enemy count rather than being mediocre at both.

---

## THE BANDIT — Smoke

Fastest gauge, highest crit, thinnest HP. Lowest ESP-to-ATK ratio at 0.85, so its competences are Atk-weighted and carry riders rather than raw multipliers.

**It generates items, not CP.** Stealing items feeds the economy the whole system rests on; generating CP directly would let one class opt out of it. Light Fingers extends this — it stretches the party's supply passively, without the Bandit spending a single turn on it.

| Lv | Competence | CP | Split | Var | MP | Acc | Effect |
|---|---|---|---|---|---|---|---|
| 1 | Orion's Mark | 14 | 100/0 | 15% | 1.60 | 87 | One enemy, 1.6× **Smoke** |
| 3 | Ursa Shot | 16 | 100/0 | 10% | 1.20 | 86 | One enemy, 1.2× **Smoke**, steal an Object into party inventory |
| 5 | Draco's Mark | 20 | 100/0 | 15% | 2.10 | 87 | One enemy, 2.1× **Smoke** |
| 7 | Lyra Shot | 18 | 100/0 | 10% | 1.40 | 87 | One enemy, 1.4× **Smoke** |
| 9 | Corvus Mark | 24 | 100/0 | 20% | 2.60 | 85 | One enemy, 2.6× **Smoke** |
| 12 | Vela Shot | 22 | 100/0 | 15% | 1.25 | 85 | All enemies, 1.25× each **Smoke** |
| 15 | Perseus Mark | 26 | 100/0 | 20% | 2.90 | 85 | One enemy, 2.9×, ignores elemental resistance |
| 18 | Hydra Shot | 32 | 100/0 | 25% | 3.80 | 85 | One enemy, 3.8× **Smoke** |

**Light Fingers** (2) — Whenever any party member uses an Object, 5% chance it is not consumed.
**Cutpurse** (10) — Enemies defeated by the Bandit drop one additional item.

At 207 HP against 380 for the Burnt, the Bandit is the class that dies. That's the trade for the fastest gauge and 25% crit.

---

## THE BURNT — Meat

A burnt Elsen, and Elsen are Meat. Weak to Smoke.

**The only class that interacts with elements.** Nothing else in the system can change a matchup once a fight has started — the Purifier picks its own element via the Runs, but nobody can alter anyone else's. The Burnt rewrites them on both sides of the field, which makes it the answer to a fight the party is otherwise simply losing.

A defensive and CP-heavy hybrid: the largest CP pool in the game, real durability, and no damage to speak of.

| Lv | Competence | CP | Split | Var | MP | Acc | Effect |
|---|---|---|---|---|---|---|---|
| 1 | Marzipan Rush | 16 | 0/100 | — | 1.20 | — | One ally: minor heal, set element to **Plastic** |
| 1 | Marzipan Crash | 16 | 0/100 | 10% | 0.56 | 92 | One enemy: 0.7× Meat, set element to **Plastic** |
| 3 | Torte Rush | 16 | 0/100 | — | 1.20 | — | One ally: minor heal, set element to **Meat** |
| 3 | Torte Crash | 16 | 0/100 | 10% | 0.56 | 92 | One enemy: 0.7× Meat, set element to **Meat** |
| 6 | Praline Rush | 16 | 0/100 | — | 1.20 | — | One ally: minor heal, set element to **Metal** |
| 6 | Praline Crash | 16 | 0/100 | 10% | 0.56 | 92 | One enemy: 0.7× Meat, set element to **Metal** |
| 9 | Meringue Rush | 16 | 0/100 | — | 1.20 | — | One ally: minor heal, set element to **Smoke** |
| 9 | Meringue Crash | 16 | 0/100 | 10% | 0.56 | 92 | One enemy: 0.7× Meat, set element to **Smoke** |
| 12 | Nougat Crash | 22 | 0/100 | 10% | 0.60 | 90 | All enemies: 0.75× each Meat + ATK Down 20% (3 turns) |
| 15 | Ganache Crash | 24 | 50/50 | 10% | 1.10 | 90 | One enemy: 1.2× Meat + **ATK Down 30%** (3 turns) |
| 18 | Sablé Rush | 26 | — | — | — | — | All allies: set element to one you choose |
| 20 | Petit Four Crash | 32 | — | — | — | — | All enemies: set element to one you choose |

**Sweet Madness** (2) — The Burnt's own element can never be changed, including by itself.
**Scorched** (10) — The Burnt's element changes last 3 of the target's turns instead of 2.

### How element control works

**Element changes last until the end of the changed character's second turn**, then revert. Duration counts the *target's* turns, so a fast enemy sheds the change quickly and a slow one carries it. Scorched extends this to a third turn.

**Damage resolves before the element change.** A Crash deals its Meat damage against the target's existing element, then rewrites it. Otherwise Marzipan Crash would set a target to Plastic and double its own hit on the same button.

**The change affects both what the target deals and what it takes.** Every check using that character's element is swapped — offence and defence alike. The one exception is a competence that forces a specific element regardless of its user, like the Purifier's Runs.

**The Burnt cannot overwrite its own change before it expires.** One element change per target, first application wins — a mistimed cast locks that target for two of its turns. The single exception is naming the target's native element, which cancels the change and restores it.

**Sugar is never a legal target.** Four Rushes and four Crashes, one per ring element. Nothing can be made un-exploitable.

### Why the flat 16 CP

Every element-setting competence costs the same, mirroring the Purifier's Runs. No element is cheaper, so the choice is purely matchup and never economics.

The pool carries the identity: 175 CP at level 20 is eleven casts. This class wants to act constantly and reshape a fight repeatedly, which is unaffordable on a martial pool.

Healing lands around 42 at level 10 against the Purifier's 78 — roughly half a healer, and elementless like all healing. Crash damage is 0.7× a normal attack. Neither is why you bring this class.

**Sablé Rush and Petit Four Crash are the same competence pointed opposite ways.** Every other element change in the kit is single-target, so retuning a full party costs one turn per ally — Sablé Rush does it in one. Its enemy-side mirror arrives two levels later.

The pair is what the ladder builds toward: single-target control from 1 through 9, mass control at 18 and 20. Sablé Rush is the defensive half — set the party to the element that resists what you're facing — and Petit Four Crash the offensive one, collapsing a mixed enemy group into a single matchup.

Neither carries damage or a rider, so neither rolls accuracy.

**Level 5 is empty.** The Burnt holds four competences by level 9 — more front-loading than any other class — so a gap there costs nothing and keeps the element ladder legible.

**Ganache Crash is mitigation, not repair.** ATK Down 30% on one enemy is the strongest single-target damage reduction in the game, and it keeps the Burnt reducing incoming damage rather than healing it back — the Purifier and Omega already own repair, and the Burnt's Rushes heal only incidentally.

It pairs with Nougat Crash at 12: the group version at 20% across everything, the focused version at 30% on whatever is hitting hardest. Against a single Guardian the focused one is worth roughly three times as much.

The 50/50 Atk/Esp split is deliberate. Every other Crash is pure Esp, so this one scales partly off the stat the Burnt is worst at — it's a mitigation button that happens to deal damage, not a damage button.

---

# PART 5 — IMPLEMENTATION

Every competence in this document fits one schema:

```
{
  name, level, cp,
  baseValue, atkPct, espPct, movePower, variance,
  target,        // one enemy | all enemies | one ally | all allies | self
  element,       // or null for non-elemental
  accuracy,      // null = cannot miss; otherwise 85–95, rolled per target
  statusTier,    // vulnerability | neutral | lightImmune | strongImmune
  effects[]      // statuses, stat changes, element changes
}
```

Balance patches become a data edit, never a code change. **If a competence needs custom code, reprice it until it doesn't** — that constraint is worth more than any single ability idea.

Build order: gauge loop and four buttons first, then the damage formula with elements and crits, then statuses keyed to the target's gauge fills, then competences as data rows, then Objects and inventory.
