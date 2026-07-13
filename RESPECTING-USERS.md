# Respecting Users

The center of these standards is [the standing What](WHAT-MATTERS-MOST.md#the-standing-what): it must cost less to create and sustain than the value it produces. Respecting users means recognizing that they are running the same equation, on you. Your artifact must be a win *for them*: better than what it replaces, and by enough to offset their cost of change (the learning, the migration, the risk they take on you). Their time the standing What already prices; the sections below are the other line items on their side of the ledger.

## Their Privacy

Their data is not your asset; it is your liability, accepted on their behalf, a cost they are lending you. Three rules follow:

* **Don't ask for data you don't need.** Every field you collect is a liability you accepted and a promise you must now keep. The data you never collect can't leak, can't be subpoenaed, can't be breached; minimization is the simplest reasonable answer applied to other people's information.
* **Don't keep knowledge you don't need to keep.** Retention is collection stretched over time; everything said above applies for as long as you hold it. Expire what has served its purpose.
* **Encrypt what you do hold. Really encrypt.** At rest and in transit alike, real encryption means a proven implementation of a proven algorithm. **Never implement your own.** Rolling your own encryption is the ultimate trap for the confident ([Choosing Dependencies](CHOOSING-DEPENDENCIES.md)): it fails silently, it is tested only by attackers, and by the time you learn it was broken, it had been broken all along.

A privacy failure spends the unbounded asset. A user whose data you leaked does not return, and neither do the users they talk to.

## Their Mistakes

Users err: a mistyped command, the wrong file, a misread option. The artifact sets the exchange rate between the slip and what it costs them: an error message that names the problem in the user's terms and points at the next step; recovery that is cheap (undo where possible, retries that don't punish, no lost work as the default). A cryptic error converts a five-second mistake into an afternoon, billed to them. And your own defects run the same account, with interest, because the user pays first in confusion and again in the workaround.

## Telemetry

Telemetry is a legitimate instrument of [Tests and Measurement](WHAT-MATTERS-MOST.md#tests-and-measurement): production is where the truth lives, and some questions can be answered nowhere else. But it is measurement *of people*, and that puts it under two principles at once. Privacy governs the payload: every field of telemetry is collected data, and the minimization rules above apply to each one. [Honesty](WHAT-MATTERS-MOST.md#honesty) governs the existence: telemetry must be overt, disclosed plainly, visible in documentation, never buried. And overt is not enough: telemetry is **opt-in**. Collect nothing until the user has indicated understanding and consent by some real means (a visible request in the UI, a choice made before the artifact first runs); any means qualifies, so long as it is a genuine choice and not fine print. Telemetry is important; it is not as important as the user. A hidden cost falsifies the user's equation: they agreed to a price that wasn't the real price. A user who discovers undisclosed telemetry has discovered a broken promise, and that discovery converts a measurement tool into a betrayal, retroactively, for every measurement you ever took.

Nothing here is a new value: only Honesty and minimization meeting at the user's boundary. And the test of respect is the oldest one in these documents: you are not done when your equation balances. You are done when both do.
