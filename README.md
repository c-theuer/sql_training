# SQL Training Program

**Instructor:** Catherine Theuer
**Dataset:** theLook eCommerce on BigQuery (`bigquery-public-data.thelook_ecommerce`)

A SQL training series that teaches business and technical users how to use SQL to answer business questions. All exercises run against theLook eCommerce, a fictional online clothing store, in the free BigQuery sandbox.

## Program history

The program started on Monday, October 5, 2026 with Session 0: an introduction to databases vs. data warehouses, how a SELECT statement works, how to limit the data a query retrieves, and setting up BigQuery. Based on that class, the program is splitting into two tracks:

- **Beginner track** on Mondays
- **Advanced track** on Wednesdays

## Schedule

| Date | Track | Session | Materials |
| --- | --- | --- | --- |
| Mon, Oct 5, 2026 | All students | Session 0: SQL basics and BigQuery setup | [Outline](session0_outline.md) · [Answers](session0_answers.md) |
| Mon, Oct 12, 2026 | Beginner | Session 1: Getting data out | [Outline](beginner/session1_outline.md) · [Answers](beginner/session1_answers.md) |
| Mon, Oct 19, 2026 | Beginner | Session 2: Summarizing data | [Outline](beginner/session2_outline.md) · [Answers](beginner/session2_answers.md) |
| Mon, Oct 26, 2026 | Beginner | Session 3: Combining tables with JOINs | [Outline](beginner/session3_outline.md) · [Answers](beginner/session3_answers.md) |
| Mon, Nov 2, 2026 | Beginner | Session 4: Real analysis and capstone | [Outline](beginner/session4_outline.md) · [Answers](beginner/session4_answers.md) |
| Wednesdays (dates TBD) | Advanced | Topics to be set from the [student survey](https://docs.google.com/forms/d/e/1FAIpQLSct6cHzqfSbkK2TBo2PyUMQo385kfjETJ7csArEWdHt6no6yw/viewform?usp=dialog) | Coming soon |

## Beginner track (Mondays)

Four 2-hour sessions that take students from their first SELECT to a multi-table business analysis.

1. **Getting data out:** SELECT, WHERE, ORDER BY, LIMIT, DISTINCT
2. **Summarizing data:** COUNT, SUM, AVG, GROUP BY, HAVING, NULLs
3. **Combining tables:** INNER JOIN, LEFT JOIN, keys, avoiding double-counting
4. **Real analysis:** dates, CASE WHEN, CTEs, and a capstone answering a marketing question

Each session has a student outline with concepts, examples and exercises, and a separate answer file with runnable queries.

## Advanced track (Wednesdays)

A [survey](https://docs.google.com/forms/d/e/1FAIpQLSct6cHzqfSbkK2TBo2PyUMQo385kfjETJ7csArEWdHt6no6yw/viewform?usp=dialog) goes out to advanced students on Monday, October 12, 2026 to find out what they want to learn. Topics and dates will be set from the results. Directions raised in Session 0 include more technical work (coding with databases) and data science (pandas/polars).

## Getting started

1. Set up the BigQuery sandbox: [docs.cloud.google.com/bigquery/docs/sandbox](https://docs.cloud.google.com/bigquery/docs/sandbox)
2. Add the `bigquery-public-data` project and star the `thelook_ecommerce` dataset.
3. Start with the [Session 0 outline](session0_outline.md).

## Files

| File | Contents |
| --- | --- |
| `session0_outline.md`, `session0_answers.md` | Session 0 notes and challenge answers |
| `beginner/session1_outline.md` to `beginner/session4_outline.md` | Beginner track student outlines |
| `beginner/session1_answers.md` to `beginner/session4_answers.md` | Beginner track answer keys |
| `images/` | Screenshots used in the Session 0 files |

## Notes

theLook data is synthetic and regenerated regularly, so query results change over time. The answer files give correct queries rather than fixed numbers.

## More practice

- [SQLZoo](https://sqlzoo.net/wiki/SQL_Tutorial)
- [DataLemur](https://datalemur.com/)
