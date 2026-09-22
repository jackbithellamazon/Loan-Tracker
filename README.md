# Loan Tracker

A single-file tool for working out what a revenue-advance offer actually costs.
Open `index.html` in a browser — no build step, no server, no account, no
backend of any kind. Everything you type is saved to that browser's
`localStorage` and stays on the machine.

## Why it exists

Revenue lenders label an offer "7 months". That label is not what you repay.
Two offers can both say "7 months" and have different fees, different numbers of
payment dates, different final payments, and payoff dates two weeks apart.

So nothing here is calculated from the displayed term. The schedule is built the
way the lender actually runs it:

```
principal + fee(repayment) = total to remit
then subtract the repayment every 14 days until the balance is zero
```

The number of payment dates is whatever falls out of that. Deployment day is not
a payment day — the first debit lands one full period later.

## The four tabs

| Tab | What it does |
|---|---|
| **Dashboard** | Live advances: what's owed, what's been repaid, when the next debit leaves the bank |
| **Manage Loans** | Add, edit and import advances; mark payments done/pending/scheduled |
| **Loan Calculator** | Model an offer against your stock ROI and cash-conversion cycle; compare two side by side |
| **Offer Slider** | Move the repayment and watch the real schedule, the fee and the payment-date boundaries move with it |

## Payment-date boundaries

A **boundary** is the repayment at which a whole future payment date disappears.
Finding one is not `total ÷ N`, because raising the repayment also lowers the
lender's fee. Both sides move, so it is solved by bisection on

```
N × repayment ≥ principal + fee(repayment)
```

The fee between two quoted points is interpolated linearly in `1/repayment` —
the fee tracks how long the money is out, and duration is roughly proportional
to `1/repayment`. Leave-one-out against the 14 seeded reference points: mean
error £2.53, worst £7.83. Straight-line interpolation on the repayment itself is
about six times worse, up to £93 out. Interpolated fees are labelled as
estimates; a figure you type in yourself always wins.

The boundaries are drawn as ticks on the slider and as steps on the chart —
every drop in the navy line is a payment date disappearing.

## Is paying more actually worth it?

A lower fee is not automatically the better deal. Paying more each fortnight
pulls working capital out of stock, so the tool prices that: it measures the
extra £-days of cash leaving the business early, values them at your own stock
return (ROI % per cycle ÷ cycle days) and sets that against the fee saved.

Treat the ROI and cycle-length inputs as a **worst case**. If your real return is
better — faster cycle, higher ROI — the cash you hand over early is worth more,
so paying more gets *worse*, not better. The verdict falls whichever way the
arithmetic goes; it does not just recommend the cheapest headline fee.

## A new advance on top of live ones

An advance is rarely taken in isolation, and the money leaves one bank account.
If there are live advances on the Dashboard, the calculator stacks the new one
on top of them and shows the month-by-month outflow, the worst month, and the
combined cash-flow pressure next to what the new advance would look like alone.

The cash-flow part of the score is judged on that combined load by default —
there's a **"Count my existing advances in the score"** switch on the panel, and
the score labels which basis it used, so it's never silently one or the other.

If the combined figure passes 100%, more cash leaves in the first stock cycle
than the advance brings in: the new money is part-funding the old repayments
before any of it reaches stock.

## The fee curve

The app ships with 14 example reference points for an £86,000 advance. They are
observations, not pricing — lenders re-quote constantly. Replace them on the
Offer Slider tab with the points from your own quote.

The box is deliberately forgiving: type `repayment, fee £, term`, or just select
the rows on the lender's page and paste. £ signs, tabs, percentages and
"7 months" are all read through. Both of these land on the same point:

```
5516, 7614.92, 8 months
£5,516   8 months   8.85%   £7,614.92   £93,614.92
```

"Pin current repayment as a point" adds whatever you are currently looking at,
turning an estimated fee into an exact one.

## Where your data lives

**This file ships empty.** No loans, no balances, no dates — `DEFAULT_LOANS` is
`[]`, so nothing real is in the published source. Open it and it asks you to
load your own data.

There is no server, no database and no account. The app makes **zero network
requests** and loads **zero external files** — no fonts, no CDN scripts, no
analytics. Open it from a local file with the wifi off and it works identically.

Your figures live in the browser's `localStorage`, which means they never leave
the machine. It also means they are not a backup: clearing site data, switching
browser, or moving to another machine loses the lot.

So the two buttons on **Manage Loans**:

- **Save a backup** writes everything — loans, payment history, offer-slider
  settings — to a `.json` file in your Downloads. Written by the browser
  straight to your own disk; nothing is uploaded.
- **Restore a backup** reads one back, after confirming. It rejects anything
  that isn't a Loan Tracker export and leaves your existing data untouched if
  the file is wrong.

That backup file is the only place your real figures exist outside the browser.
Treat it like a bank statement, and keep it out of this repo — if you ever put
one alongside the app, add a `.gitignore` with `loan-tracker-data-*.json`.

## Tests

The repayment engine is covered by 129 headless checks — the full balance table
of a real offer letter, the payment-date boundary solver, the paste parser, the
schedule edge cases, and a guard that this file still ships with no real data
in it. If a backup file is present they also reconcile it against the lender's
own remitted/remaining figures.

They're kept outside this repo, next to it rather than in it. To run them:

```bash
deno run --allow-read checks.js /path/to/index.html
```

`checks.js` finds `index.html` on its own if it sits beside it or one folder
across; otherwise pass the path.

## Bundled data

None. The app opens with nothing in it — that's deliberate, so the published
source carries no real figures. Load your own via **Restore a backup**, or add
a loan by hand.

The example fee curve on the Offer Slider is the one exception: fourteen
reference points for an £86,000 advance, so the boundary maths has something to
demonstrate on. They're lender pricing, not anyone's account. Replace them with
your own quote's numbers whenever you like.

## First payment isn't always a full period

Lenders differ on when the first debit lands, and it shifts every date in the
schedule:

- Some keep the **deployment's weekday** — deploy Thursday, first debit the
  Thursday a fortnight later, a clean 14 days.
- Some snap every remittance to a **fixed weekday**. Both advances above settle
  on a Sunday, so a Monday deployment gives a **13-day** first gap and 14 days
  after that.

Leave **First payment after (days)** blank for one full period, or set it to
match what your lender actually does.
