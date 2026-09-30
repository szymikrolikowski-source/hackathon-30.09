# Booked → Ready · Trip Mode for KBC (Tectonic Hackathon prototype)

The moment a customer pays for a trip, Kate detects it and prepares everything in one screen:
suggested travel insurance (only the gaps), a pre-filled insurance form, and a foreign-currency
pocket for the trip. Customers can switch **Trip Mode** on or off at any time.

## Run it
```bash
pip install -r requirements.txt
cp .env.example .env          # set DEMO_PASSWORD (and optionally GEMINI_API_KEY)
cd backend
set -a && source ../.env && set +a
uvicorn app:app --reload
```
Open http://localhost:8000 and log in as `lotte`, `wout` or `emma` with your DEMO_PASSWORD.

## Demo script
1. Log in as **lotte** → no trips yet.
2. Tap **"Demo: simulate a flight booking"** → *Trip to Tokyo detected*, "11 of 13 fields filled for you".
3. **Review travel insurance** → recommended covers with reasons, full application pre-filled with the source of every field.
4. Confirm the estimated dates → **Sign & take out insurance** → policy number issued.
5. Back on the trip → **Open JPY pocket**.
6. **wout** has a yearly travel policy → "already covered", only a USD pocket.
7. **emma** has Trip Mode off; switch it on → phone and co-traveller are missing, so only those are asked.

## How it works
| Step | Where | AI? |
|---|---|---|
| Filter travel payments (merchant category codes, amount ≥ €150) | `engine.detect_trips` | No, cheap rules for millions of customers |
| Understand messy merchant names (`BKG*HTL SHINJUKU…` → Tokyo, JP) | `engine.parse_merchant_ai` | Yes, Gemini, JSON output validated with Pydantic; rule fallback without a key |
| Recommend policy + covers (assistance, cancellation, luggage) with reasons | `insurance.recommend` | No, deterministic code |
| Pre-fill the application from KBC profile, household and booking; mark each field filled / check / needed | `insurance.build_form` | No |
| Sign: server recomputes premium, blocks until nothing is missing or unconfirmed | `POST /api/me/trips/{id}/insurance/sign` | No |
| Currency pocket, card-replacement warning | `engine.py` | No |

## Security
- User identity comes only from the session token (`/api/me/...`), never from the URL.
- Every trip action checks ownership (no IDOR); the server re-validates chosen insurance codes.
- Password and API keys only via `.env`; constant-time password check; sessions expire after 1 h.
- Frontend renders with `textContent` only (no `innerHTML`).
- Merchant text sent to the AI is treated as data; output is schema-validated.

## Data
100% synthetic (`backend/data_gen.py`): 1,000 random customers with ~12% hidden trips + 3 demo users.
No real customer data.

## Unfinished / roadmap
- Real booking-document parsing (upload PDF → exact dates) — designed, not built
- During-trip features (low-balance top-up, delay claims), after-trip summary
- Real KBC policy pricing and Kate Wallet integration (premiums and FX rates here are placeholders)
- Persistent database (state is in memory)
