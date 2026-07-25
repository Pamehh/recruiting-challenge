# Decision Log — <Pamela>

## Authorship declaration
> - *"I wrote this decision log entirely without AI assistance. The only tool I used on it was spell-check."*
---

## Issues addressed

> Defects, security smells, architectural problems, missing pieces, scaling risks — anything you decided was worth your time. For each, fill in **every** sub-field. An empty field is a worse signal than an awkward answer.

- **Issue 1 — <Crossed access to tenant's information>**
  - What was wrong or weak: the base configuration gave any merchant access to any merchant's order by typing the orderID because the query only filtered by order ID, not by merchant ID>
  - Shape of my improvement: Modified the data access layer so that it is necessary to get both the order id and the merchant's id. Updated the callers related and added a text to check that other merchants cannot access the order. 
  - **Confidence (1–10):** 9
  - **What would falsify this fix**: if it was designed like this on purpose and any merchant accessing any merchant's order is correct. If another merchant can access or guesses other merchant's ID's due to obvious configuration.  
  - **I disagreed with Claude on:** did not disagree.
  - Alternatives I considered and rejected: the place of the merchant check function, I initially thought of letting it in the route. But that wouldn't protect every caller like enforcing it in the DAL. 

- **Issue 2 — <Refund calculation>**
  - What was wrong or weak: The configuration of refund was making the revenue and customer metrics incorrect by being considered as positive because the revenue was adding them instead of substracting it.
  - Shape of my improvement: modified aggregate queries to treat refunds as negative values with a CASE expression. Added a test to check this with a sale of 10 and a refund of 4 to get the correct revenue of 6.
  - **Confidence (1–10):**9
  - **What would falsify this fix:** if refunds increase the revenue or if the aggregate queries interpret the refunds as positive. 
  - **I disagreed with Claude on:** Did not disagree. 
  - Alternatives I considered and rejected: changing the sign in how the refunds are stored to have easier access to refunds and checking if any other block of code assumes that refunds are positive. The first one would be a data migration that would require more planning for the application, the second one would require more time.

- **Issue 3 — <Storage of emails in dashboard>**
  - What was wrong or weak: the user's email was being inserted using innerHTML which gives way to security issues by allowing JS or HTML code stored in the database and executable in browsers.
  - Shape of my improvement: changed the approach by creating an element that renders as plain text in the browsers.
  - **Confidence (1–10):** 8
  - **What would falsify this fix:** the XSS only being accesible to the targeted user who inputed it.
  - **I disagreed with Claude on:** the severity of this issue because XSS are not common malware and also it would be hard to escalate outside of the user's with malicious intent in this case. 
  - Alternatives I considered and rejected:

## Feature chosen

- **Feature:** A. CSV export of orders
- **Why this one and not the others:** It implements one of the changes I added earlier, the refunds logic.
- **What I cut to ship it in budget:** kept only necessary features and simple UI.
- **Confidence (1–10) that the shape I picked is the right one:** 8
- **What would change my mind:** havinga large database, changes in export format or in the workflow.

## Things I noticed but did NOT fix

> Class-of-bug instances you saw and chose not to touch. For each, name the *reason* you cut it (scope / time / dependency / "needs a larger conversation").

- 

## Docs / code I left alone deliberately

-

## What I'd do with another 6 hours

-I would have a closer look to the connections in the database, input validation, I would spend more time in design and testing of my current implementations. I would check the SQueries as well to make sure indesex are correct and the right data is being gotten, I would check if I can simplify the code.

## Where I felt uncertain

> At least three places in this submission where you were not confident. Genuine uncertainty is a strength signal. "Nothing — I was confident everywhere" is itself a red flag and will be probed.

-I had my questions about all three implementations because I would need more context on the goals of this app to make changes like limiting access to orders, maybe in a context some merchants could be administrators and they would need to have access to other merchant's orders. 
- If the replacement of innerHTML for textContent is the right approach or there is a simpler one that reinforces security or tone that automatically checks the input.
-I am unsure on what is the best approach to handle negative and positive values in our app, I checked whether my approach was correct on financial situations using the web. I appears that the use of CASE is just a temporary solution because it puts wheight on the database and makes queries slower oblviously and also it is necessary to keep this consistent in future reports, which would need devs to be careful or just one dev to be in charge of them, impractical.
