# Auto Ticket Classification (ServiceNow Flow Designer)

Automatically classifies new IT incidents and routes them to the right support group, with no scripting.

## Problem
Service desk agents waste time manually reading tickets and assigning groups. Wrong or late assignment delays resolution.

## Solution
A Flow Designer flow triggers when an incident is created, identifies the category (from the Category field, or from keywords in the Short description if Category is blank), then sets the Category and Assignment Group. Unmatched tickets go to Service Desk for manual triage.

## Architecture

```
New Incident created
        |
   Flow Trigger (Incident, Created)
        |
   Classification logic (If / Else If / Else)
        |
 +------------+-------------+-------------+-----------------+
 | Hardware   | Software    | Network     | No match        |
 +------------+-------------+-------------+-----------------+
       |             |            |               |
 Hardware      Software      Network        Service Desk
 Support       Support       Support        + work note
```

## Tech
- ServiceNow Personal Developer Instance (free: developer.servicenow.com)
- Flow Designer (Record trigger, Flow Logic, Update Record action)

## Setup

### 1. Create assignment groups
All > User Administration > Groups > New. Create:
- Hardware Support
- Software Support
- Network Support
- Service Desk

### 2. Create the flow
All > Flow Designer > New > Flow
- Name: `Auto Ticket Classification`
- Description: Automatically classify IT incidents and assign them to the appropriate support group.

### 3. Trigger
Add Trigger > Record > Created
- Table: Incident [incident]
- Condition: none (runs on every new incident)

### 4. Classification logic
Add Flow Logic > If, then add Else If branches in this order.

| Branch | Condition | Update Record (Incident) |
|---|---|---|
| IF 1 | Category is Hardware **OR** (Category is empty AND Short description contains `laptop` / `keyboard` / `screen` / `mouse`) | Category = Hardware, Assignment group = Hardware Support |
| ELSE IF 2 | Category is Software **OR** (Category is empty AND Short description contains `install` / `software` / `app` / `license`) | Category = Software, Assignment group = Software Support |
| ELSE IF 3 | Category is Network **OR** (Category is empty AND Short description contains `wifi` / `vpn` / `network` / `internet`) | Category = Network, Assignment group = Network Support |
| ELSE | (no match) | Assignment group = Service Desk, Work notes = "Auto-classification failed. Manual triage needed." |

Tips:
- Each keyword is a separate OR row in the condition builder.
- Update Record target = Trigger > Incident Record.
- Click **Save**, then **Activate**. The flow will not run until activated.

### 5. Test
Create incidents (All > Incident > New) and check Flow Designer > Executions.

| # | Short description | Category | Expected group |
|---|---|---|---|
| 1 | Laptop not working | Hardware | Hardware Support |
| 2 | Software installation problem | Software | Software Support |
| 3 | WiFi connection problem | Network | Network Support |
| 4 | WiFi not connecting | (blank) | Network Support, Category auto-set to Network |
| 5 | Keyboard keys stuck | (blank) | Hardware Support |
| 6 | Random issue | (blank) | Service Desk + work note |

## Results
Fill this in after testing: add screenshots of the flow, the Executions tab, and a classified incident.

## Future improvements
- Priority assignment based on keywords (e.g. "server down" = P1)
- Email notification to the assigned group
- Predictive Intelligence / ML-based classification
- Dashboard for auto-classified vs manual tickets

## Author
Sanjay, IT student, Tamil Nadu, India
