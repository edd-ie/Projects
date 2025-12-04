_A deception-enabled competitive extension of Rock–Paper–Scissors–Lizard–Spock._

---

# **1. Game Basics**

Each round, each bot selects one of five actions:  
**Rock, Paper, Scissors, Lizard, Spock**

Scoring uses standard RPSLS outcomes:

- Win: **+1**
- Loss: **–1**
- Draw: **0**

Matches run for **10,000 rounds**.
Scores are **hidden** from bots during the match.

---

# **2. Dual-Action System: Real Moves & Shadow Moves**

Each bot outputs two fields per round:

1. **real_move**  
    Used for actual scoring.

2. **shadow_request** (boolean)  
    If true, the bot is attempting to substitute the visible “last move” shown to the opponent.

3. **shadow_move**  
    The move that will appear to the opponent **if** the shadow request succeeds.


### **Shadow Move Rules**

- Each bot starts the match with **50 deception tokens**.
- A successful shadow request **consumes 1 token**.
- Shadow substitutions **never fail** and always take effect when tokens remain.
- If a bot has **0 tokens**, shadow requests are ignored (no substitution occurs).

### **What the Opponent Sees**
Opponents see **opponent_last_visible**, which may be:

- the opponent’s real last move, or
- a shadowed move if a token was used.

**All scoring always uses real moves only.**

---

# **3. Information Provided to Bots**

Each round, a bot receives:

```json
{
  "round": <int>,
  "opponent_last_visible": <move>,
  "self_last_real": <move or null>, 
  "opponent_deception_bucket": <"HIGH"|"MEDIUM"|"LOW"|"EMPTY">
}
```

### **Interpretation**

- **opponent_last_visible:** the last visible action, real or shadowed.
- **self_last_real:** the bot’s actual last move (useful for consistency checks).
- **opponent_deception_bucket:** a coarse estimate of the opponent’s remaining deception tokens:

    - **HIGH** = 40–50 tokens
    - **MEDIUM** = 20–39
    - **LOW** = 1–19
    - **EMPTY** = 0

Exact token counts are never shown.
Scores remain hidden for the entire match.

---

# **4. Deception Token Mechanics (Clarified)**

### **4.1 Tokens Are Per-Bot, Not Shared**

Every bot has its own 50-token pool **for each match**.
Using a shadow move does not affect:

- the opponent’s tokens
- global resources
- other matches
### **4.2 Token Accounting**

- Tokens only reduce when your bot uses a shadow request and tokens > 0.
- The bucket level changes only when **your opponent** spends a token.
- This allows inference from bucket movement (e.g., MEDIUM → LOW).

### **4.3 Shadow Move Use Cases**

Strategic uses include:

- Breaking opponent’s Markov predictions
- Masking losing streaks
- Creating false pattern signals
- Triggering model collapse right before a pivot
- Concealing adaptation timing

---

# **5. Strategic Model**

Chaos League creates a **partially observed learning environment**. Bots must:

- Infer opponent deception timing
- Detect inconsistencies in visible history
- Estimate opponent token depletion from bucket transitions
- Decide when deception is worth spending
- Maintain entropy to avoid exploitation
- Preserve long-term expected value without knowing the score

The system rewards:

- Clean predictive modeling
- Unpredictable randomness when needed
- Smart resource management
- Well-timed deception

---

# **6. Scoring Model (Revised & Tightened)**

Final match score is a weighted combination of three components:

### **6.1 Standard RPSLS Score (70%)**

Your total net points across all rounds.

### **6.2 Anti-Exploitation Bonus (20%)**

Measures how well your bot:

- adapts under corrupted history
- maintains stable performance when deception occurs
- avoids becoming pattern exploitable
- performs well when the opponent’s bucket transitions downward

This is computed via statistical comparison between:

- rounds with visible deception pressure
- rounds without such pressure

### **6.3 Deception Efficiency Score (10%)**

Rewards strategic use of tokens:

- Well-timed shadow moves that mislead opponent predictors
- Token use correlated with opponent misreads
- Avoiding wasteful or spammy shadowing
- Causing measurable divergence between what your opponent sees and what you actually do

Bots that spam or randomly burn tokens score poorly here.

---

# **7. Final Score Formula**

```
Final = 
  (0.70 * Standard Score)
+ (0.20 * Anti-Exploitation)
+ (0.10 * Deception Efficiency)
```

This keeps the gameplay skill-based while keeping deception relevant but not dominant.

---

# **8. Match Format**

### Pool Stage

Round-robin: each bot plays every other bot once or twice.

### Finals

Top bots advance to best-of-three or best-of-five matches.  
Matches in finals use **longer rounds** (15,000–20,000) for stability.

---

# **9. Fairness Guarantees**

- No random failure mechanics exist.
- Deception is fully deterministic.
- Token pools are individual and hidden but inferable.
- Visible history is always consistent with the rules.
- RNG (if used by bots) is seeded externally per match for reproducibility.

---
