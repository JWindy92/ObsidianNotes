![[Pasted image 20261005115948.png]]\


### Basic Workflow
1) dig up the schedule insert from the json processing time
2) insert data
3) re-process json schedule
4) update verif file in final 
5) monitor for filewatcher to pick up
6) confirm verif processor has correct lines



Each line, download file from ESVP

Log String: FileKey FARH TRU 20261003 7911
  - 1 hr surrounding window
  - Ingest: 102 followed by SendToProcessor -> Get Log stream name

"Found # LOIs" = num spot ids in JSON file

all above to get to Ingest: 440 (Insert Statement) (only 1 per lambda run)
  - Grab insert command




Im working on TBS

aws-prd-schedule-load


---

# verif-reinsert

  

Pulls the `INSERT INTO aes_schedule` statement out of CloudWatch Lambda logs for schedules that failed to land in the database, and saves each one as a `.sql` file you can paste into a DB client.

  

## The short version

  

```bash

conda activate aws-scripting

python main.py --logs --max-rows 0

```

  

Results land in `output/sql/`, one file per schedule.

  

Everything below explains what that actually does and how to fix it when it doesn't work.

  

## Background: how the data flows

  

When a schedule is ingested, a Go Lambda writes several log lines per run. Three of them matter:

  

1. **`[Ingest: 102]`** — "Processing FileKey:{...}". The start of a run. Contains the OU, date, zone, syscode, and channel.

2. **`[SendToProcessor:NNNN]`** — the message handed to the next stage. Contains the same `FileKeyJson` fields, plus the **Lambda ID** for that run.

3. **`[ingest:440]`** — `insertSQL:INSERT INTO aes_schedule ...`. The statement we want. Exactly one per Lambda run.

  

Every line is prefixed with `Lambda ID: [<uuid>]`. That UUID is what ties the SendToProcessor line to its `[ingest:440]` line. Two lines from the same run share a Lambda ID; they do **not** necessarily share a timestamp or a log stream.

  

## Finding the SQL by hand

  

Do this once to sanity-check, or when the script finds nothing and you need to know whether the data is even there.

  

### 1. Get the timestamp

  

Open the schedule's JSON file and read `blendTimestamp`:

  

```json

"blendTimestamp": "2026-10-03T16:09:17.5201718Z"

```

  

This is UTC, and it's roughly when the Lambda ran. Everything gets searched in a window around it.

  

### 2. Build the FileKey search string

  

From the verification CSV, take four columns and assemble them in this order:

  

```

FileKey <zone code> <ChannelName> <Date> <SysCode>

```

  

The zone code is the bit inside the parentheses of `ZoneName`, before `-BP`:

  

```

Danbury BYPASS (DNJ-BP) -> DNJ

Far Hills-Longhill BYPASS (FARH-BP) -> FARH

```

  

So a row with ZoneName `Danbury BYPASS (DNJ-BP)`, SysCode `151`, ChannelName `TBSC`, Date `20261003` gives:

  

```

FileKey DNJ TBSC 20261003 151

```

  

### 3. Search the log group

  

In the CloudWatch console, open the Lambda's log group, set the time range to about an hour around the blendTimestamp, and paste that string into the filter box.

  

> **Why this works:** CloudWatch treats unquoted, space-separated words as terms that must **all** appear somewhere in the event. It is not a phrase match. So `DNJ` matches inside `(DNJ-BP)`, `151` matches inside `SysCode:151`, and so on. Wrapping the whole thing in quotes would make it a literal phrase and match nothing.

  

Add `SendToProcessor` as a sixth term to jump straight to the line you want:

  

```

FileKey DNJ TBSC 20261003 151 SendToProcessor

```

  

### 4. Copy the Lambda ID

  

From that line:

  

```

Lambda ID: [4bfecc26-9407-54a8-97c0-af1dbed8b22d] 2026/10/03 12:09:17 [SendToProcessor:2711] kermit:{Message:{FileKeyJson:{OU:NY_BYPASS ...

```

  

Grab `4bfecc26-9407-54a8-97c0-af1dbed8b22d`.

  

### 5. Search again for the INSERT

  

```

"4bfecc26-9407-54a8-97c0-af1dbed8b22d" 440

```

  

Quotes around the UUID, `440` bare. That returns one line:

  

```

Lambda ID: [4bfecc26-...] 2026/10/03 12:09:17 [ingest:440] insertSQL:INSERT INTO aes_schedule (ou, schedule, date, ...) VALUES (...),(...),(...)

```

  

Everything from `INSERT INTO` onward is the statement. Note it has **no trailing semicolon** in the log.

  

## Running the script

  

The script automates all five steps above for every row in a CSV.

  

### One-time setup

  

```bash

conda activate aws-scripting

conda install -y -c conda-forge pandas boto3 python-dotenv requests

cp .env.example .env

```

  

Then edit `.env`. The values that matter:

  

| Variable | What it is |

|---|---|

| `INPUT_CSV_PATH` | Path to the verification CSV, e.g. `input/20261003_tb_LOI).csv` |

| `CHANNEL_NAME` | Which channel to keep. `TBSC` |

| `LOG_GROUP_NAME` | The Lambda log group, e.g. `/aws/lambda/aes-prd-schedule-json-processor` |

| `LOG_TIMESTAMP` | `blendTimestamp` from the schedule JSON |

| `WINDOW_MINUTES` | How far to search either side of it. `30` = a one-hour window |

| `AWS_PROFILE` / `AWS_REGION` | Which credentials to use |

| `SQL_DIR` | Where `.sql` files get written. Defaults to `output/sql` |

  

`.env` is gitignored. Never commit it.

  

### Each time you run it

  

1. **Drop the new CSV in `input/`** and point `INPUT_CSV_PATH` at it.

2. **Find the blendTimestamp.** Open any of the schedule JSONs for that batch and copy `blendTimestamp` into `LOG_TIMESTAMP`. All the Lambdas in one batch run within a few minutes of each other, so one timestamp covers the whole file.

3. **Run it**, first for a single row to check things work, then for all of them:

  

```bash

python main.py --logs

python main.py --logs --max-rows 0

```

  

Output looks like:

  

```

17 row(s) with ChannelName=TBSC

FileKey DNJ TBSC 20261003 151

...

searching +/-30m around 2026-10-03 16:09:17 UTC

  

=== FileKey DNJ TBSC 20261003 151 ===

stream: 2026/10/03/[$LATEST]b2c86c27...

saved output/sql/FileKey_DNJ_TBSC_20261003_151.sql

```

  

4. **Open the files in `output/sql/`** and paste into your DB client. The script appends the trailing `;` that the log line is missing.

  

## Options

  

| Flag | Effect |

|---|---|

| `--max-rows N` | Process the first N rows. `0` = all. Default `1` |

| `--logs` | Do the CloudWatch lookup. Without it, only the keys are printed |

| `--print-sql` | Echo the statement to the terminal as well as saving it |

| `--debug` | Show the filter patterns, sample matched log lines, and Lambda IDs |

| `--timestamp` / `--window` | Override `LOG_TIMESTAMP` / `WINDOW_MINUTES` |

| `--input` / `--channel` | Override the CSV path / channel filter |

| `--sql-dir` | Write `.sql` files somewhere else |

  

## When it doesn't work

  

Run with `--debug` first. It prints the exact filter pattern sent to AWS, which you can paste into the CloudWatch console to compare.

  

**`no SendToProcessor event found`** — the first search came back empty. Usually the time window. Check `LOG_TIMESTAMP` is set and is the right day, and widen it with `--window 240`. Also confirm `LOG_GROUP_NAME` is the right Lambda.

  

**`no 440 line for <uuid>`** — the second search came back empty, or found the line but couldn't parse it. `--debug` distinguishes these, since it reports how many events the search saw. If it saw 1 and still failed, the regex is the problem. Check that `SQL_REGEX` in `.env` is `(INSERT\s+INTO\b[\s\S]*)`. An older version required a trailing `;`, which the log line does not have, so it matched nothing.

  

**Nothing matches but the console finds it** — quoting. Bare terms are ANDed; quoted terms are literal phrases. `"ingest:440"` quoted does not match, `440` bare does. Use `probe.py` to test a pattern directly:

  

```bash

python probe.py '"4bfecc26-9407-54a8-97c0-af1dbed8b22d" 440'

python probe.py 'FileKey DNJ TBSC 20261003 151' 240

```

  

The second argument overrides the window in minutes.

  

## Downloading schedule JSON (optional)

  

There's also a downloader for the schedule files, ported from the ESVP web app's API call. It needs a bearer token copied from a browser session, which expires quickly, so it's fiddly. The only thing it was really needed for was reading `blendTimestamp`.

  

```bash

# .env: API_BASE_URL and BEARER_TOKEN

python main.py --download --filename 20261003_91_14_12817_151.json

```

  

Filenames follow `{date}_{tbChannel}_{tbHeadend}_{sourceId}_{syscode}.json`, where the middle three values come from inside the JSON itself, which is awkward since you need the file to know its own name. Easier to grab it from the browser's network tab.

  

## Layout

  

```

main.py entry point

probe.py run a raw CloudWatch filter pattern

input/ verification CSVs

output/sql/ generated .sql files

src/verif_reinsert/

config.py reads .env

input_reader.py CSV load, TBSC filter, composite key

cloudwatch.py log queries, timestamp parsing

pipeline.py key -> SendToProcessor -> Lambda ID -> 440

sql_extract.py regex the INSERT, write the .sql file

executor.py stub, does not run SQL yet

cli.py argument parsing

```

  

Executing the statements is **not** implemented, `executor.py` is a placeholder. Everything this tool does is read-only.


### Root Cause

- Merged causing duplicate windows