# Twizz Chat

**Chat moderation that catches the tricks without punishing honest hosts.**

Wayzyy Solo Developer Challenge

---

## Quick summary

| | |
| --- | --- |
| Your ten test cases | All ten covered. Section 5 shows how each one is caught. |
| Running cost | About ₹325 a month at a million messages |
| Speed | Most messages decided in about a thousandth of a second |
| The main idea | The same sentence can be fine or not fine depending on where the person is in their booking. Nothing else works this way. |

Every number here is either checked against a named source or worked out from one. Where something is a target rather than a measurement, it says so. Nothing is built yet: this is the design and the plan.

---

## 1. Your brief already says what is wrong

> "Airbnb's current system often locks innocent messages while letting clever digit-splitting slip through."

Two failures are named there. Only one of them is about catching tricks.

The other one, **locking innocent messages**, is the more expensive failure, because it is the one that makes good hosts leave. Your homepage is largely an essay about hosts leaving.

So this is not a pitch about catching more phone numbers. Most submissions will be that. This is a pitch about catching the tricks **without generating false accusations against honest hosts**, and about proving the second half rather than claiming it.

### What a careless filter blocks

```
"our flight lands at 9:40 pm"                just a time
"we are 4 adults and 2 kids"                 just a count
"is 15000 for 3 nights negotiable?"          just a price
"gate code is 4471"                          the guest is standing outside the villa
"AC is broken, can I get a partial refund?"  a fair complaint, not a threat
"403516 is the pincode for Assagao"          a pincode
```

Every one of these has to go through untouched. That is harder than catching the tricks, and it is what we are optimising for.

---

## 2. The idea nobody else will bring

> **A phone number in chat is not always a problem. It depends on when it was sent.**

Before someone books, swapping numbers is how you get cut out. Worth stopping.

After the booking is confirmed, **Wayzyy gives both people each other's numbers anyway.** Hiding a number at that point achieves nothing, because the guest already has it on their screen. You have only annoyed someone.

Most filters apply one rule everywhere, forever. That is exactly why they get it wrong in both directions at the same time.

```
"call me on 98765 43210"

  Before booking     ->  hide it, and explain they'll get the number after booking
  After booking      ->  allow, they already have the number
  During the stay    ->  allow, they might be locked outside at midnight
  During a dispute   ->  allow, never block someone collecting evidence
```

Same sentence. Four different correct answers. A filter that cannot tell these apart has to choose between annoying honest people and letting real leakage through, and by your own description the incumbent manages to do both.

**There is a fairness argument here too.** If we wrongly hide a number before booking, the user loses almost nothing, because they get it automatically a minute later. So we can be firm early. But wrongly blocking a host of three years sending a gate code at midnight is the exact experience described on your homepage. So we are gentle later. The booking stage tells us how much caution we can afford.

---

## 3. The booking stages we track

| Stage | What is happening | If someone shares a number |
| --- | --- | --- |
| **Browsing** | Not even enquired yet | **Block.** No innocent reason to swap numbers with a stranger |
| **Enquiry** | Asked about a villa, nothing booked | **Hide and explain.** The risky window, where "let's do this directly" happens |
| **Requested** | Booking sent, waiting on the host | **Hide and explain.** Still not committed |
| **Confirmed** | Booking is done | **Allow.** Wayzyy hands over both numbers at this exact moment anyway |
| **During the stay** | Guest is at the villa | **Allow.** Gate codes, wifi passwords, "I'm at the gate" |
| **After the stay** | Stay is over | **Allow, with a gentle note** |
| **Dispute** | Something went wrong | **Allow everything. Log everything.** Never block someone collecting evidence |

**Two things stay blocked in every single stage:**

- **Payment outside Wayzyy.** UPI IDs, "pay me in cash", bank details. There is no point in a booking where this is fine, because it cancels the guest's protection completely.
- **Threats and blackmail.** Never acceptable, at any stage.

---

## 4. How it works

Four stages. Almost every message walks straight through the first two. Very few ever reach the last one.

| Stage | What it does | How many messages |
| --- | --- | --- |
| **1. The Cleaner** | Undoes the disguise. Turns `nine eight 7` into `987`. Pulls junk digits out of words. Strips invisible characters, dashes, emoji digits, lookalike letters from other alphabets, and spelled-out symbols like `(dot)` | All of them |
| **2. The Checklist** | Looks for known things: phone numbers, UPI IDs, social handles, emails, links, Aadhaar. And *verifies* each one, so a price or a pincode is never mistaken for a phone number | All of them. **Most stop here** |
| **3. The Cheap AI** | Understands meaning, not just patterns. Catches "let's just talk directly", which has no number in it at all. Runs on an ordinary computer and costs nothing | The suspicious ones |
| **4. The Expert** | A real AI. Only for messages that are genuinely confusing, plus anything that might be blackmail or a threat. Slow, and the only part that costs money | **About 3 in 100** |

Each stage is smarter and more expensive than the one before it, so the expensive one only ever sees what the cheap ones could not resolve. **That is the whole trick.** It is why this is fast, and why it costs about ₹325 a month instead of ₹8,000.

### A useful extra from the Cleaner

The Cleaner also measures *how much* it had to clean. A normal message barely changes. A disguised message changes a lot. That measurement by itself is a strong clue, and it is what tells "flight lands at nine forty" (barely changed, harmless) apart from a disguised phone number (heavily changed).

### Then it decides, and it explains itself

Five possible responses, not two: allow, a gentle note, hide part of it, hold for review, or block. Blocking is deliberately rare.

Unlike the incumbent, text never just silently disappears:

```
"We've hidden a phone number here. Wayzyy shares both numbers automatically
 the moment the booking is confirmed, usually a couple of taps away.

                                              Was this a mistake?  ->"
```

**That button is also how the system improves.** Every tap gives us a real human-labelled example of something we got wrong, tied to the exact message and the exact reasons we flagged it. After a couple of hundred, accuracy measurably improves, with no extra engineering and nobody hired to label data. The polite version of the product is also the version that gets better fastest.

---

## 5. Your ten test cases

All ten are covered. Here is how.

| # | The trick | What catches it |
| --- | --- | --- |
| 1 | Junk digits inside words, plus numbers written as words | Cleaner separates letters from digits, giving both `am akshay` and the number. A partial run of 7 or more digits next to "call me on" is enough |
| 2 | `insta:` and `telegram @`, with `(dot)` and underscores | Cleaner converts `(dot)` and splits on underscores. A dedicated check recognises social handles. "cannot share phone here but" is itself a signal |
| 3 | "send payment to UPI 9876543210" with a discount offer | A bare mobile number next to `UPI` or `GPay` counts as a payment address. **Blocked in every booking stage** |
| 4 | "refund or I'll trash the place and post fake reviews" | All three blackmail ingredients present: leverage, a money demand, and "or" linking them. Plus a property damage threat |
| 5 | `alex [at] gmail [dot] com` and a rival website | Cleaner converts the spelled-out symbols back into a real email and a real domain |
| 6 | Number emojis and dashes | Cleaner turns emoji digits into ordinary digits and strips the dashes. Both halves resolve to the same valid number |
| 7 | `ch@t w!th m30n c@ll: 987 654 3210` | Cleaner undoes the letter-for-number swaps, but only on chunks that are mostly letters, so the real number survives. See the trap below |
| 8 | `whtsapp` misspelled, plus a wa.me link | We match common misspellings of platform names, and recognise the WhatsApp link format |
| 9 | "I know where you live, transfer 5000 or I come to your office" | A separate category for threats to turn up in person: a safety matter, not a business dispute |
| 10 | `wayzyy-verify-payment-auth.online` | Contains your brand but is not a Wayzyy domain. That alone is enough, before looking at anything else |

### Test case 7 contains a hidden trap

Undoing letter-for-number swaps normally means turning `0` into `o` and `3` into `e`. Do that carelessly and the real phone number `3210` becomes `e21o`. The violation vanishes and the filter reports everything is fine.

We only apply those swaps to chunks that are mostly letters, so `c@ll` gets fixed while `3210` is left alone. That is the kind of bug that quietly breaks a filter without anyone noticing.

### What your test cases changed

Working through your ten cases changed the design in seven concrete ways. Being straight with you: three of the ten were not fully handled before.

| What was added | Which cases it fixes |
| --- | --- |
| Converting spelled-out symbols: `(dot)`, `[at]`, "dot", "at" | 2 and 5 |
| Accepting partial numbers of 7 or more digits next to "call me" | 1 |
| Treating a bare mobile number next to "UPI" or "GPay" as a payment address | 3 |
| A dedicated check for social handles | 2 |
| Splitting on underscores and dashes inside handles before reading numbers | 2 |
| Matching misspelled platform names | 8 |
| A new category for threats of physical harm and property damage | 4 and 9 |

Your cases also say something useful about where to spend the two days. **Eight of the ten are won at the Cleaner stage.** They are character-level disguises, not questions of meaning. Only cases 4 and 9 genuinely need the AI. So the Cleaner gets built first and gets built properly.

---

## 6. The hardest case, and how we handle it honestly

Consider this message:

```
"booking ID WZY-9876543210"
```

Ten digits, starting with 9, which is a valid Indian mobile shape. A simple filter fires immediately and blocks an innocent message.

But it could genuinely be a hidden phone number. Anyone can type `WZY-` in front of their number. **The prefix proves nothing.** So here is how it is actually resolved, in order:

1. **Format check, no lookup needed.** Wayzyy generates its own booking IDs, so you know the format. If real ones are `WZY-` plus 8 alphanumeric characters, then ten digits does not match and is immediately suspicious. This alone kills most of the attack, because the attacker has to fit a 10 digit number starting with 6 to 9 into your exact ID format.
2. **Existence lookup, definitive.** Does that booking ID exist, and does it belong to this thread? One query, and it is settled. An ID that does not resolve, or resolves to a different booking, is a disguise.
3. **The obvious question.** Why is a guest typing their own booking ID into the thread that is already attached to that booking? The platform already knows. Quoting it is genuinely unusual behaviour, and mildly suspicious on its own.
4. **If none of that settles it**, the message is hidden rather than blocked, with the appeal button. For a genuinely ambiguous case, the right answer is the cheap reversible one, not a confident verdict.

**The principle underneath this:** anything that looks like a platform identifier gets checked against the platform, not against a pattern. Booking IDs, property IDs, invoice numbers are all things Wayzyy issued and can therefore verify. A filter that trusts a prefix is trusting the attacker.

---

## 7. Built for India, not translated into it

You list villas in Goa, verify guests with Aadhaar, and pay out to Indian banks. Most submissions this week will contain an American phone number pattern.

| What generic filters miss | Why it matters here |
| --- | --- |
| **UPI is how money actually leaks out.** `9876543210@ybl` is a PhonePe user identified by their mobile number | It is two problems at once, a payment route and a phone number leak, and it looks like an email address to every standard tool |
| **Spoken UPI:** "nine eight seven, at why bee ell" | No pattern matcher catches this. The word to digit step has to run first |
| **Hinglish counting:** ek, do, teen, chaar, paanch, chhe, saat, aath, nau | Romanised Hindi has no fixed spelling, so "chhe", "che" and "chah" all mean six |
| **Intent without a number:** "number bhej do", "advance UPI kar do", "cash de dena" | Should raise suspicion rather than block outright |
| **Indian mobile numbers start with 6, 7, 8 or 9** | One rule that removes a whole class of false alarms. Goa pincodes start with 4 |
| **Aadhaar in chat is your liability** | A helpful guest typing their Aadhaar number creates a data protection problem you never asked for. It has a built-in maths check, so we spot it almost perfectly and remove it before it is ever saved |

---

## 8. Beyond phone numbers: the receipt

Your brief mentions coercion once, in passing. Your homepage devotes four sections to it: refund-for-review extortion, staged evidence, deleted warnings, "the decision is final."

Most submissions will chase the phone number, because that is where the memorable example is. The extortion problem looks like the reason your company exists.

It cannot be caught by looking for anger, because **the classic extortion message is polite.** We look for three things together instead:

```
1. Something the sender controls that you care about   (a review, a rating)
2. A demand worth money                                (a refund, a free night)
3. A link between them                                 (do this, or that happens)

"The pool was closed, I'd like a partial refund"
        demand only               ->  a fair complaint, let it through

"Refund me or I'll mention this in my review"
        all three                 ->  hold it, save the evidence, tell your team
```

**The framing:** your homepage says hosts leave because receipts do not matter. This makes the chat itself the receipt, timestamped, categorised, and handed to whoever reviews the dispute. Every decision the system makes is written down with its reasons, so if a host asks in March why something happened in January, you can answer precisely instead of guessing.

---

## 9. What it costs

| Messages per month | Twizz Chat | Sending every message to an AI | Cheaper by |
| --- | --- | --- | --- |
| 100,000 | **$1.15** | $10.36 | 9 times |
| 1,000,000 | **$3.71** (about ₹325) | $95.86 | **26 times** |
| 10,000,000 | **$29.36** | $950.86 | 32 times |

Worked out from Google's published prices, checked on 12 August 2026, using the actual size of the requests we would send. The comparison column is deliberately generous to the alternative, because we gave it the same efficiency tricks we use ourselves.

### If we are wrong about the 3%

Everything depends on only about 3 messages in 100 reaching the paid stage. That is a target we will measure, not a result we have. So:

| Messages reaching the AI | Cost per month at 1M | Still cheaper by |
| --- | --- | --- |
| 1 in 100 | $1.81 | 53 times |
| **3 in 100 (our target)** | **$3.71** | **26 times** |
| 10 in 100 | $10.36 | 9 times |
| 20 in 100 | $19.86 | 5 times |

Wrong by a factor of seven, and it is still five times cheaper. **The argument survives being wrong**, which matters more than the headline number.

**Why cost is a design constraint here and not a footnote:** you take about 2% where you say the big platforms take around 18%. A platform on 18% can afford an expensive safety system. A platform on 2% cannot. We designed backwards from your economics.

### Could we use open source AI instead?

Mostly we already do. The first three stages run on free, open software on an ordinary computer, covering about 97 messages in 100.

For the last stage, running our own would be **more** expensive at your size, not less. Renting a suitable machine costs a few hundred dollars a month whether it is busy or not, and it only becomes worthwhile somewhere past 80 million messages a month.

So we build it to be swappable in an afternoon, use the paid service now, and switch if the volume justifies it, or if you decide you would rather Indian chat data never left your own machines. That second one is your call, not ours.

---

## 10. How we would prove it

Anyone can claim 99% accuracy in a pitch. We would rather build the thing that lets you check.

Alongside the system: a test set of realistic Goa rental chat, disguised violations generated using about 22 techniques including all ten of yours, and, the part that matters most, **a set of innocent messages written by hand specifically to trip up a careless filter.**

How often we wrongly flag those is the headline number, because it is the failure you actually complained about. We will publish it next to the same numbers for simpler approaches, so you can see what the complexity is buying.

### And an honest list of what it cannot do

| Limitation | When we would fix it |
| --- | --- |
| **Numbers inside photos**, such as a screenshot of a UPI QR code. This is the biggest gap and the obvious next move for anyone trying to get around it | Week 3 |
| Voice notes | Week 5 |
| Numbers spread across many messages over a long period | Week 2 |
| Blackmail built up across several messages rather than said in one | Week 4 |
| New UPI app endings not yet on our list | Ongoing |

Every other submission will claim there are no weaknesses. Naming ours is the cheapest honest signal we can give you.

---

## 11. The plan

### Two days

**Day 1.** The Cleaner, complete. **Your ten test cases pass by lunchtime.** Then the Checklist, the booking-stage rules, and the test set. First honest numbers by the end of the day.

**Day 2.** The two AI stages. Conversation memory for numbers split across messages. Explanations, the appeal button, decision records. Then **a page you can try yourself**: a chat box, a booking-stage switch, and a button for each of your ten cases. Plus results and the limitations list.

**Deliberately not built in two days:** login and accounts, a real database, reading numbers out of images, custom AI training, exhaustive Hinglish. Named as deferred, not forgotten. Better to hand you a small thing that works completely than a large thing that works partly.

### After that

| When | What | Why |
| --- | --- | --- |
| Week 1 | Run it quietly beside real traffic without acting | Tune it on your real messages before it touches anyone |
| Week 2 | Turn on appeals and feed them back | Accuracy improves on its own |
| Week 3 | Read numbers and QR codes out of photos | Closes the biggest gap |
| Week 4 | A console for your team | Search past decisions, see reasons, handle appeals |
| Week 6 | Teach the cheap AI from the expensive one | **Fewer messages reach the paid stage, so it gets cheaper over time** |
| Week 8 | Hindi and Konkani explanations | Goa is not English only |

---

## 12. What we would want to know from you

1. Roughly how many messages a month? The right setup differs a lot between 100,000 and 10 million.
2. Should moderation hold up the send, or run just behind it and pull a message back if needed? We have assumed the second.
3. Should contact details unlock automatically once a booking is confirmed? We have assumed yes, and we think it is a real advantage over the incumbent, but it is your policy call.
4. Is this a standalone service, or something to build into your existing backend?
5. What happens to the trial work if we are not the ones you hire?

---

## In one paragraph

Cheap, simple checks handle almost every message instantly and for free. A real AI only sees the few that are genuinely confusing, which is why it costs about ₹325 a month rather than ₹8,000. Before deciding anything, it asks where in the booking the message was sent, because the same sentence can be fine or not fine depending on when it arrives, and that is the thing that stops honest hosts getting blocked. When it does act, it explains itself and offers a one-tap way to say it got it wrong, which is also how it gets better. It understands UPI, Hinglish and Aadhaar because your users are in Goa. And it ships with a test set and an honest list of its own weaknesses, because that is the only way you can check any of it.

---

## Sources

| Claim | Source |
| --- | --- |
| Contact details released after confirmation, and pre-confirmation off-platform solicitation is against policy | airbnb.com/help/article/147 and /4155 |
| Unicode confusables, and homoglyph bypass rates | Unicode UTS-39, arXiv:2508.14070 |
| The small AI model's size, speed and language coverage | github.com/MinishLab/model2vec |
| UPI address format and provider handles | Razorpay, Paytm and NPCI documentation |
| API pricing | ai.google.dev/gemini-api/docs/pricing, checked 12 August 2026 |
| GPU rental rates | RunPod and Vast.ai published rates, August 2026 |

Currency at ₹87.5 to the dollar. The roughly 18% figure for the big platforms is your own published number, not one we have verified. API pricing moves, so it is worth re-checking before either of us relies on it.