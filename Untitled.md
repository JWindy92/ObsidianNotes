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