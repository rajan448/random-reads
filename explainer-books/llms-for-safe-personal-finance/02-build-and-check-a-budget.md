# 2. Build and verify a monthly budget

> **TL;DR:** Use the LLM to suggest options; use your records and calculator to establish the numbers.

## Prepare a reliable starting point

A **budget** is your plan for income, spending, and saving over a period. **Cash flow** tracks when money arrives and leaves.

Begin with one complete month. Record take-home income, spending, required debt payments, and money already assigned to savings. Keep the original records private.

If you categorize transactions, prevent double counting. A purchase and the later payment of its credit card bill usually represent the same spending. Transfers between your accounts are not new income. Refunds need consistent treatment.

Mark uncertain transactions for your review. Do not let the model guess their purpose.

## Work through a fictional month

Suppose you have these monthly amounts in rupees. Every number is fictional.

| Item | Amount |
| --- | ---: |
| Take-home income | ₹60,000 |
| Essential spending | ₹36,000 |
| Minimum debt payments | ₹4,000 |
| Optional spending | ₹8,000 |
| Unassigned money | ₹12,000 |

Verify independently:

```text
60,000 − 36,000 − 4,000 − 8,000 = 12,000
```

“Unassigned” does not mean available to invest. You may still owe annual insurance, repairs, taxes, or other irregular bills.

A **sinking fund** is money set aside gradually for a known future expense. For a fictional ₹24,000 bill due in twelve months, starting from zero:

```text
24,000 ÷ 12 = 2,000 per month
12,000 − 2,000 = 10,000 still unassigned
```

If the bill is due in three months, the same starting balance requires ₹8,000 monthly. Timing changes the plan.

## Ask for choices with explicit assumptions

Paste only the summary and this prompt:

```text
Use the fictional monthly totals above. Assume they do not
overlap. There is also a 24,000 annual bill due in 12 months,
with nothing saved for it yet. All figures are rupees.

1. Show the arithmetic and list missing information.
2. Include the annual bill's monthly provision.
3. Draft two allocations of the remaining money:
   one emphasizing emergency savings and one debt repayment.
4. Keep required payments and essential spending covered.
5. Label proposed allocations as examples, not recommendations.
6. Identify assumptions that could change the comparison.
Do not select investments or take financial actions.
```

**Emergency savings** are money reserved for unexpected essential costs or income loss. The suitable amount depends on your circumstances.

Before choosing between the drafts, review your cash buffer, debt rates, penalties, income stability, and upcoming expenses. Ask a qualified professional when the tradeoff has significant consequences.

## Check the result against reality

Treat the assistant's draft like a route suggested by a map. Check the road before turning.

Confirm that allocations add up to income. Check payment dates against payday: a monthly surplus does not prevent a shortage before rent is due.

Try a stress test: “What happens if income falls to ₹50,000 next month?” In this example, only ₹2,000 remains before the annual-bill provision. That provision consumes the balance.

Save the version you verify. Review actual spending weekly, then adjust next month's draft. Keep a separate record of completed payments; a chat message does not prove payment.

## Key takeaways

- Reconcile totals before asking for suggestions.
- Include irregular bills and payment timing.
- Verify arithmetic outside the LLM.
- Choose allocations after checking your circumstances.

**Next: [Check advice, permissions, and scams](./03-check-advice-and-scams.md)**
