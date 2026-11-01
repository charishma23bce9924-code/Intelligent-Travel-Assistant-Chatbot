# Intelligent Travel Assistant Chatbot (Fuzzy Logic)

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/charishma23bce9924-code/Intelligent-Travel-Assistant-Chatbot/blob/main/Travel_Assistant_Chatbot.ipynb)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Gradio](https://img.shields.io/badge/UI-Gradio-F97316)
![scikit-fuzzy](https://img.shields.io/badge/Fuzzy%20Logic-scikit--fuzzy-6C63FF)

A conversational travel recommender for India. Tell it your **budget**, preferred **climate** and your **interest in adventure/heritage**, and a **fuzzy-logic inference engine** ranks 50 curated destinations and suggests the best compromise. Once you pick a place, it shows **live weather**, a **5-day forecast**, **stay links** and **3 / 5 / 7-day itineraries**. If you reject a suggestion, the bot adapts for the rest of the session and offers a better alternative.

Final project (CSE2009), B.Tech Computer Science & Engineering, VIT-AP University.

## Why fuzzy logic?

Travel preferences are vague: a budget of "around ₹50,000", a climate that is "cool but not freezing". Boolean filters force hard cut-offs and either return nothing or the wrong place. Fuzzy logic scores *how well* each destination matches each preference (a degree of membership between 0 and 1), so the system can find the best overall compromise across all three criteria.

## Screenshots

| Start | Recommendation + live weather | Explore menu |
|---|---|---|
| ![Budget prompt](docs/screenshots/01-budget-prompt.png) | ![Recommendation](docs/screenshots/02-recommendation.png) | ![Explore menu](docs/screenshots/03-explore-menu.png) |

| 5-day forecast | 7-day itinerary |
|---|---|
| ![Forecast](docs/screenshots/04-weather-forecast.png) | ![Itinerary](docs/screenshots/05-seven-day-itinerary.png) |

## Features

- Guided, stateful chat built with Gradio (`gr.ChatInterface` + `gr.State`)
- Fuzzy matching over **budget (INR)**, **climate (°C)** and **interest (0-100)**
- Dataset of **50 Indian destinations**, each with scores plus 3-, 5- and 7-day itineraries
- **Live weather** and a **5-day forecast** via the OpenWeatherMap API
- Stay-search links (Google Maps, Booking.com, Airbnb)
- **Adaptive feedback loop**: rejecting a suggestion nudges that destination's scores for the session
- Type `new` / `reset` at any time to start over, `exit` to finish

## How it works

```
User input ──► Scale to 0-100 ──► Triangular membership per destination ──► Sum ──► Top-ranked place
 (INR, °C, 0-100)                  (trimf, half-width 20, 3 attributes)      │
                                                                             └──► Weather API / itinerary / feedback loop
```

1. **Fuzzification / scaling.** Budget (₹2,000-₹2,00,000) and climate (2-60 °C) are normalised to 0-100. Interest is already on that scale.
2. **Rule base = dataset.** There is no hand-written `IF budget IS medium...` rule list. Each destination is its own implicit rule, e.g. *Manali = budget 40, climate 20, interest 85*.
3. **Inference.** For every destination, three triangular membership functions (`skfuzzy.trimf`, centred on its scores, ±20) are evaluated at the user's values with `interp_membership`, giving `b_score`, `c_score`, `i_score`.
4. **Aggregation + defuzzification.** `total = b_score + c_score + i_score`; winner-take-all ranking picks the highest total.
5. **Adaptive feedback.** On "another", the rejected place goes to a `disliked_places` list and `simulate_backpropagation` nudges its scores by 5 away from the user's preference. This is session-only and resets with a new session.

**Worked example.** ₹95,000, 15 °C, interest 5 → scaled to 47.0 / 22.4 / 5. Manali (40 / 20 / 85) scores 0.65 + 0.88 + 0.00 ≈ 1.53, the best overall compromise, so it is recommended.

### Conversation states

| State | Bot asks | Next |
|---|---|---|
| `start` | Greets, asks budget | `get_budget` |
| `get_budget` | Climate preference | `get_climate` |
| `get_climate` | Interest 0-100 | `get_interest` |
| `get_interest` | Runs fuzzy engine, shows pick + weather | `feedback` |
| `feedback` | "like" or "another"? | `explore` or next pick |
| `explore` | `forecast`, `stays`, `3-day`, `5-day`, `7-day`, `exit` | stays in `explore` |

## Tech stack

| Tool | Purpose |
|---|---|
| Python | Core language |
| Gradio | Web chat UI |
| scikit-fuzzy | Membership functions and interpolation |
| NumPy | Numeric arrays for the universe of discourse |
| Requests | HTTP calls to OpenWeatherMap |
| OpenWeatherMap API | Current weather and 5-day / 3-hour forecast |

## Getting started

You need a free [OpenWeatherMap API key](https://openweathermap.org/api).

**Option 1: Google Colab (easiest).** Click the *Open in Colab* badge, add your key as a Colab Secret named `OPENWEATHER_API_KEY` (key icon in the left sidebar), then `Runtime → Run all`. A public `gradio.live` link appears when the last cell runs.

**Option 2: locally.**

```bash
git clone https://github.com/charishma23bce9924-code/Intelligent-Travel-Assistant-Chatbot.git
cd Intelligent-Travel-Assistant-Chatbot
pip install -r requirements.txt

export OPENWEATHER_API_KEY="your_key_here"      # Windows PowerShell: $env:OPENWEATHER_API_KEY="your_key_here"
jupyter notebook Travel_Assistant_Chatbot.ipynb
```

If no key is found, the notebook prompts for one. **Never commit your API key.**

## Project structure

```
.
├── Travel_Assistant_Chatbot.ipynb   # dataset, fuzzy engine, chat logic, Gradio UI
├── requirements.txt
├── docs/screenshots/                # UI screenshots used in this README
└── README.md
```

## Limitations

- Conversation flow is a keyword-driven state machine, not natural-language understanding.
- Destination scores are hand-curated and static; the 50-place dataset is a prototype knowledge base.
- Feedback learning lives only in the current session; nothing is persisted.
- Weather features need internet access and a valid OpenWeatherMap key.

## Future scope

- **Persistent user profiles** (SQLite / MongoDB) to keep preferences and nudged scores across sessions
- **NLU front end** (spaCy, Rasa or a transformer) so users can say "a cheap, cold place for trekking" in one sentence
- **Weighted preferences**: let users rate the importance of budget, climate and interest, e.g. `total = w_b·b + w_c·c + w_i·i`
- **Dynamic dataset** connected to a live destinations database or travel API
- Multilingual and voice support, and booking-platform integration

## Team

Developed at VIT-AP University by **Rishitha Uppala, Jyothi Shravani Mamidi, Charishma Jagarapu and Charmesh Chejarla**, under the guidance of **Dr. Radha Mohan Pattanayak** (Assistant Professor, School of Computer Science and Engineering).

## References

- [scikit-fuzzy documentation](https://pythonhosted.org/scikit-fuzzy/)
- [Gradio documentation](https://www.gradio.app/)
- [OpenWeatherMap API](https://openweathermap.org/api)
- L. A. Zadeh, "Fuzzy sets", *Information and Control*, 1965
