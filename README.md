# CST8917 Lab 4 Real-Time Trip Event Analysis

## Executive Summary

This lab implements a real-time event-driven system for monitoring and analyzing taxi trip data. The solution leverages Azure Event Hub for data ingestion, Azure Functions for intelligent analysis, Logic Apps for workflow orchestration, and Microsoft Teams for instant notifications. The system successfully identifies suspicious patterns in trip data and alerts operations staff in real-time.
## Youtube Demo
[Demo Video Link](https://youtu.be/XXAvi55zHVQ)

### Logic AppworkFlow
<img width="1799" height="1005" alt="Screenshot 2025-08-01 at 7 28 53 PM" src="https://github.com/user-attachments/assets/eb28bc64-af92-4113-a479-2cbf103e1e2d" />


## Architecture Overview

### System Components

1. **Azure Event Hub (triipeventhub)**
   - Ingests trip events in JSON format
   - Configured with default consumer group
   - Batch processing with up to 10 events per pull

2. **Azure Logic App**
   - Polls Event Hub every minute
   - Orchestrates the analysis workflow
   - Routes notifications based on analysis results

3. **Azure Function (taxi-app-analyze_trip)**
   - Performs intelligent trip analysis
   - Identifies patterns and suspicious activities
   - Returns enriched data with insights

4. **Microsoft Teams Integration**
   - Receives Adaptive Card notifications
   - Three distinct card types for different scenarios
   - Direct chat integration with Flow bot

### Data Flow
```
Event Hub → Logic App Trigger → Azure Function → For Each Loop → Conditional Logic → Teams Notifications
```

## Logic App Workflow Details
### 1. Input

This is the input data (Yellow Taxi)  sent to the Event Hub (from Azure Data Explorer)

```json
{
    "vendorID": "2",
    "tpepPickupDateTime": 1528119858000,
    "tpepDropoffDateTime": 1528121148000,
    "passengerCount": 1,
    "tripDistance": 1.24,
    "puLocationId": "186",
    "doLocationId": "230",
    "startLon": null,
    "startLat": null,
    "endLon": null,
    "endLat": null,
    "rateCodeId": 1,
    "storeAndFwdFlag": "N",
    "paymentType": "1",
    "fareAmount": 13.5,
    "extra": 0,
    "mtaTax": 0.5,
    "improvementSurcharge": "0.3",
    "tipAmount": 2.86,
    "tollsAmount": 0,
    "totalAmount": 17.16
}
```
This is the custom payload for generating the suspicious vendor activity.
```
{
    "vendorID": "2",
    "tpepPickupDateTime": 1528119858000,
    "tpepDropoffDateTime": 1528121148000,
    "passengerCount": 3,
    "tripDistance": 0.93,
    "puLocationId": "186",
    "doLocationId": "230",
    "startLon": null,
    "startLat": null,
    "endLon": null,
    "endLat": null,
    "rateCodeId": 1,
    "storeAndFwdFlag": "N",
    "paymentType": 2,
    "fareAmount": 13.5,
    "extra": 0,
    "mtaTax": 0.5,
    "improvementSurcharge": "0.3",
    "tipAmount": 2.86,
    "tollsAmount": 0,
    "totalAmount": 17.16
}
```
### 2. Event Hub Trigger
```json
"When_events_are_available_in_Event_Hub": {
    "recurrence": {
        "interval": 1,
        "frequency": "Minute"
    },
    "queries": {
        "maximumEventsCount": 10,
        "consumerGroupName": "$Default"
    }
}
```

- **Frequency**: Polls every minute for new events
- **Batch Size**: Processes up to 10 events per execution
- **Content Type**: JSON format for structured data

### 3. Azure Function Call
```json
"taxi-app-analyze_trip": {
    "type": "Function",
    "inputs": {
        "body": "@triggerBody()"
    }
}
```

- Passes the entire event batch to the function
- Function processes each trip and returns analysis results

### 4. For Each Loop Processing

Iterates through each analyzed trip result and implements nested conditional logic:

#### Primary Condition: `isItInteresting`
- **True Branch**: Further evaluates if it's suspicious
- **False Branch**: Posts "No Issues" notification

#### Secondary Condition: `IsSus` (Is Suspicious)
- **True Branch**: Posts "Suspicious Vendor Activity" card
- **False Branch**: Posts "Interesting Trip Detected" card

## Azure Function Logic Description

The Azure Function (`analyze_trip.py`) implements the following analysis logic:


### Input Processing
- Accepts batch of trip events from Event Hub
- Extracts trip data from ContentData property
- Handles both single events and arrays

### Analysis Rules

1. **Long Trip Detection**
   - **Condition**: `tripDistance > 10` miles
   - **Insight**: "LongTrip"

2. **Group Ride Identification**
   - **Condition**: `passengerCount > 4`
   - **Insight**: "GroupRide"

3. **Cash Payment Tracking**
   - **Condition**: `paymentType == "2"`
   - **Insight**: "CashPayment"

4. **Suspicious Vendor Activity**
   - **Condition**: `paymentType == "2" AND tripDistance < 1`
   - **Insight**: "SuspiciousVendorActivity"
   - **Purpose**: Flags potential fare manipulation

## Microsoft Teams Integration
### Output 
<img width="1799" height="1005" alt="Screenshot 2025-08-01 at 7 28 23 PM" src="https://github.com/user-attachments/assets/97dcedc6-29af-4477-88e2-6bd90b9504e7" />


