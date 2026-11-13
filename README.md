# Riccardo Gravinese

Senior Software Engineer and AI tech lead at CP Sistemi, near Bologna, Italy.
I build AI systems for manufacturing plants and take them from the first demo on the
customer's data to production.

- A multi-agent assistant on plant energy data, in production for a year, now running for seven
  customers, one plant each.
- Text-to-SQL at 95% on the plant benchmark, with deterministic rules and a validator between
  the model and the database.
- Python, LangGraph, FastAPI, SQL Server, Qdrant, Ignition SCADA.

## Public work

**[agentic-ems](https://github.com/riccardogravinesebvu-afk/agentic-ems)**: plain-language
questions on a synthetic plant's energy data, answered three ways by the same model and scored
against reference values: a semantic layer that compiles the SQL, text-to-SQL with the rules in
the prompt, and text-to-SQL from the schema alone. 55 questions, five repeats each. With the rules
the model that writes SQL made no mistake; without them it returned 12 wrong values in 275 runs,
4 of them double counts. The semantic layer cannot double count, but outside its four metrics it
refuses or returns ingredients, not answers. Written from scratch on synthetic data, with a
[hosted demo](https://agentic-ems-demo.vercel.app) that replays four recorded answers.

**[harness-loop](https://github.com/riccardogravinesebvu-afk/harness-loop)**: an optimizer agent
that improves another agent by iterating on evals. 44 eval cases, 6 loops, 19 hypotheses, about
€8.7 of API spend. Accuracy went from 45% to 87% overall, and from 40% to 67% on the cases the
optimizer never sees. What I learned matters more than the curve: a single-run gate let a
regression through, the judge's rubric moved the score more than most hypotheses did, and most of
the gain came from one edit. Every number points to the results file behind it.

## A case from proposal to production

The code and the data belong to the customer, so this is the story without them. The customer is
a food plant and is not named.

**The problem.** The plant monitors energy on Ignition SCADA. Meters write increments to SQL Server
every fifteen minutes, with fixed dashboards on top. Any question outside the dashboards went to a
technician, who wrote the query by hand in about half an hour. Sometimes the answer was wrong,
because meters form a hierarchy: add a main meter to its children and you count the same kWh twice.

**How it started.** Nobody had asked for this. I saw how many requests ended up on the
technician's desk, built a demo on the energy manager's real questions, showed it to my employer and
then to the customer, and wrote the proposal. Before writing it I sat twice with the energy manager
and the production manager to learn which questions they really ask and which aggregation rules
they apply in their heads.

**The system.** A Python service with read-only access to the SCADA database, a vector store with
the plant manuals, and a web interface that streams the answer with a chart when one helps. A
supervisor graph decides whether a question goes to the database, to the manuals or to both.
Everything runs in containers inside the customer's network.

**Where it breaks.** The model writes plausible SQL and makes the same mistakes again and again. I
fixed them with deterministic rules instead of hoping for a better prompt, one test per rule. On
top sits a validator that reads a hand-written domain model (the meter hierarchy and how each
metric aggregates) and blocks a query that sums different levels. On "total plant consumption" the
first version inflated the result about 13 times. One of the double-counting cases we mapped is
still open; until it is closed the system says it cannot answer instead of answering wrong.

**How I know it works.** A benchmark of plant questions with expected answers, a separate
benchmark for retrieval from the manuals (recall 86%, MRR 0.88), and an automatic judge whose
stability I measure by running it again. A change to a prompt or to retrieval goes in only if
these numbers do not go down.

**Result.** A reporting request outside the dashboards went from about 30 minutes through a
technician to about 3 minutes, self-served. For the customer that is 160 hours of technical time
and more than 10k EUR a year. It went to production about three and a half months after the
proposal, with the junior developers I coordinated and trained.

**What did not work.** I rewrote some database views to remove a scalar function and nothing
changed, because SQL Server was already inlining it. The real bottleneck was a join on a long text
column that cannot be indexed. Materialized tables fixed it, and response times dropped by an order
of magnitude.
