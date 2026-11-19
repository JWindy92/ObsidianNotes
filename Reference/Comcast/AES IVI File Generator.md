---
tags:
  - ivi
  - aes
  - generator
  - comcast
  - work
---
### Overview
- Daily run
- Sends file to [FreeWheel]
	- Describes the TV Schedule
	- Used by FW to decide what programs advertisers can buy ads on
		- #task How do they use it?
- Made up of 7 different Lambdas
	- #task name them here
- Tech Stack
	- AWS Lambda
	- Python (3.12)
	- Triggered by EventBridge Scheduler

### Live Event Targeting
- This work is to provide support for additional filtering and behavior for live programs
- Allows advertisers to filter on live events to further refine their search for programs to purchase on

##### Before
```
<fwTitles>
  <titleItem><title>VG: Provider Group: Unknown: CBS HD</title>   <titleType>Group</titleType></titleItem>
  <titleItem><title>VG: Stream Type: Live</title>                 <titleType>Group</titleType></titleItem>
  <titleItem><title>VG: Program Type: SportingEvent</title>       <titleType>Group</titleType></titleItem>
  <titleItem><title>VG: Airing Type: Live</title>                 <titleType>Group</titleType></titleItem>
  <titleItem><title>VG: Airing Type: Repeat</title>               <titleType>Group</titleType></titleItem>
  <titleItem><title>Golden Globes 2026</title>                    <titleType>Episode Title1</titleType></titleItem>
</fwTitles>
```

##### After
```
<fwTitles>
  <titleItem><title>VG: Provider Group: Unknown: CBS HD</title>   <titleType>Group</titleType></titleItem>
  <titleItem><title>VG: Stream Type: Live</title>                 <titleType>Group</titleType></titleItem>
  <titleItem><title>VG: Program Type: SportingEvent</title>       <titleType>Group</titleType></titleItem>
  <titleItem><title>VG: Airing Type: Live</title>                 <titleType>Group</titleType></titleItem>
  <titleItem><title>VG: Airing Type: Repeat</title>               <titleType>Group</titleType></titleItem>
  <titleItem><title>VG: Program Title: Golden Globes 2026</title> <titleType>Group</titleType></titleItem>   <!-- new -->
  <titleItem><title>VG: Episode Title: Golden Globes</title>      <titleType>Group</titleType></titleItem>   <!-- new -->
  <titleItem><title>VG: Synopsis: Awards ceremony recognizing film and television</title> <titleType>Group</titleType></titleItem>   <!-- new -->
  <titleItem><title>Golden Globes 2026</title>                    <titleType>Episode Title1</titleType></titleItem>
</fwTitles>
```