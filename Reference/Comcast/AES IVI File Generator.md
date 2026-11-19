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
- 