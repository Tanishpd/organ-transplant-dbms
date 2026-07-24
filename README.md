# organ-transplant-dbms

A relational database for organ transplantation: it tracks donors, patients,
available organs, the hospitals and doctors involved, and the transactions that
match a donated organ to a recipient.

Built as a database-design exercise, so the interesting part is the schema
rather than the interface — the normalisation, the multi-valued attributes
pulled out into their own tables, and the queries that do the matching.

## Schema

Twelve tables. Phone numbers are separated into their own tables because a
user, doctor or organisation can have several — storing them inline would
violate 1NF.

| Table | Holds |
|---|---|
| `user` | Base record: name, date of birth, gender, address, city, state |
| `user_phone` | One row per phone number per user |
| `donor` | Organ offered, reason, linked user |
| `patient` | Organ needed, reason, linked user |
| `organ_available` | Organs currently in the pool |
| `doctor` | Treating doctors |
| `doctor_phone` | One row per phone number per doctor |
| `organization` | Hospitals and transplant centres |
| `organization_head` | Who runs each organisation |
| `organization_phone` | One row per phone number per organisation |
| `transaction` | A donor–recipient match and its outcome |

`er diagram.png` shows the entity relationships. `sql database/tables.sql`
creates the schema; the remaining `.sql` files populate individual tables.

## Setting it up

```bash
mysql -u root -p -e "CREATE DATABASE dbms_project;"
mysql -u root -p dbms_project < "sql database/tables.sql"

# then the per-table data, in this order (foreign keys)
for t in user user_phone doctor doctor_phone organization \
         organization__head organization_phone donor patient \
         organ_available transaction; do
  mysql -u root -p dbms_project < "sql database/$t.sql"
done
```

The `templates/` directory contains a Python front-end (`main.py`) over the
same schema.

## Data

**All data in this repository is synthetic.** Names are `Name-1`, `Name-2`;
addresses are `Street-1`; phone numbers are sequential from `7000000000`;
medical reasons are `Reason-1`. Nothing here relates to a real person, and no
real medical record was used.

## Documents

The coursework report and presentation are not committed here. They carry the
names and university registration numbers of the three other students on the
team, and publishing those in a public repository is not mine to do. Available
on request.

## Security note — coursework, do not deploy

This was a database-design assignment; the application layer in `templates/`
was not written to production standards and has known problems:

- **SQL injection.** Most queries interpolate request data into the SQL string
  rather than using parameters. The login query has been fixed, since that one
  is reachable without credentials, but the rest have not. Do not expose this
  to untrusted input.
- **Plaintext passwords.** The `login` table stores and compares passwords in
  the clear rather than hashing them.

Database credentials are read from the environment (`MYSQL_HOST`, `MYSQL_USER`,
`MYSQL_PASSWORD`, `MYSQL_DB`); they used to be hardcoded.

The schema and the ER model — the actual point of the assignment — are in
`sql database/` and `er diagram.png`.
