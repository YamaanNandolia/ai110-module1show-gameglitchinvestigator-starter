# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

The first time I ran the game, Streamlit opened a page titled "Game Glitch Investigator" with a guess input, Submit/New Game buttons, and a sidebar for difficulty settings. The Developer Debug Info expander showed the secret number, which made it possible to test whether hints and win logic were trustworthy. Even when I typed the exact secret from debug info on odd-numbered attempts, the hints often pointed the wrong way, and on even-numbered attempts the game seemed to compare my guess against a different "secret" than the one displayed.

Concrete bugs noticed at the start:

1. **Hints are backwards** — when my guess was too high, the app told me to go HIGHER; when too low, it told me to go LOWER.
2. **Even attempts use string comparison** — on the 2nd, 4th, 6th… submit, the secret is converted to a string, so hints follow lexicographic rules (e.g., guessing `9` against secret `50` says "Too High" because `"9" > "5"`).
3. **UI range text is wrong** — the info banner always says "Guess a number between 1 and 100" even on Easy (1–20) or Hard (1–50).
4. **Hard mode range is smaller than Normal** — Hard uses 1–50 while Normal uses 1–100, which is the opposite of what "Hard" suggests.
5. **Attempt counter starts at 1** — on Normal (8 attempts allowed), the first screen shows "Attempts left: 7" instead of 8.

**Bug Reproduction Log**

Document at least 3 bugs you found. Add rows as needed.

| Input | Expected Behavior | Actual Behavior | Console Output / Error |
|-------|-------------------|-----------------|------------------------|
| Secret = 50, guess = 60 (1st attempt, hints on) | Outcome "Too High"; hint says go **lower** (e.g., "Go LOWER!") | Outcome "Too High"; hint says "📈 Go HIGHER!" | none |
| Secret = 50, guess = 40 (2nd attempt, hints on) | Outcome "Too Low"; hint says go **higher** | Outcome "Too Low"; hint says "📉 Go LOWER!" (wrong direction) | none |
| Secret = 50, guess = 9 (2nd attempt — secret passed as `"50"`) | "Too Low" (9 < 50 numerically); hint says go higher | "Too High" with "📈 Go HIGHER!" (string compare: `"9" > "5"`) | none |
| Difficulty = Easy (range 1–20), open game | Info banner says guess between **1 and 20** | Banner still says "Guess a number between **1 and 100**" | none |
| Difficulty = Normal, fresh game | "Attempts left: **8**" | "Attempts left: **7**" (`attempts` initializes to 1) | none |

---

## 2. How did you use AI as a teammate?

I used **Cursor Agent** (in VS Code) as my primary AI teammate for Phase 2, attaching `app.py`, `logic_utils.py`, and `tests/test_game_logic.py` so the assistant could see both UI and logic together.

**Correct suggestion:** The AI traced the backwards-hint bug to `check_guess` in `app.py`, where a guess of 60 against secret 50 returned `"Too High"` with the message `"Go HIGHER!"` instead of `"Go LOWER!"`. It suggested swapping the hint strings and removing the broken `TypeError` fallback that compared strings on even attempts. I verified this by running new pytest cases (`test_too_high_hint_says_lower` and `test_numeric_compare_on_small_guess`) and by playing the Streamlit app with Developer Debug Info open — hints now point the right way on every attempt.

**Incorrect / misleading suggestion:** Early in the session the AI briefly suggested keeping the `TypeError` handler but "fixing" only the message strings inside the `except` block. That would still leave even-attempt guesses using lexicographic string comparison (e.g., guess 9 vs secret `"50"` would stay wrong). I rejected that approach after reading the submit handler in `app.py` and confirming the real fix was to always pass the integer secret and delete the string fallback entirely. A quick manual test on attempt 2 confirmed guess 9 now correctly returns "Too Low."

---

## 3. Debugging and testing your fixes

I treated a bug as fixed only when **both** automated tests and a manual Streamlit run agreed. For logic bugs I relied on pytest first because it is fast and repeatable; for UI/state issues (attempt counter, range banner) I reloaded the app and checked the sidebar and info banner against the active difficulty.

The most useful test was `test_too_high_hint_says_lower`, which asserts that `check_guess(60, 50)` returns outcome `"Too High"` and a message containing `"LOWER"`. Before the fix this test would have failed because the message said `"HIGHER"`. Running `pytest tests/ -v` gave 9 passing tests, including the three starter cases updated to unpack the `(outcome, message)` tuple. The AI helped draft the new test names and assertions; I kept them focused on one behavior each instead of one giant test.

Manual verification: I ran `python -m streamlit run app.py`, opened Developer Debug Info, entered the displayed secret on attempts 1 and 2, and confirmed a win with balloons. I also switched to Easy and confirmed the banner reads "between 1 and 20" and Normal shows 8 attempts left on a fresh game.

---

## 4. What did you learn about Streamlit and state?

- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?

---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?
  - This could be a testing habit, a prompting strategy, or a way you used Git.
- What is one thing you would do differently next time you work with AI on a coding task?
- In one or two sentences, describe how this project changed the way you think about AI generated code.
