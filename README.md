# 🔐 Week 4 – Automating Access Review Decisions with PowerShell and Microsoft Graph API

![Microsoft Entra ID](https://img.shields.io/badge/Microsoft%20Entra%20ID-IAM-blue)
![Microsoft Graph](https://img.shields.io/badge/Microsoft-Graph-0078D4)
![PowerShell](https://img.shields.io/badge/PowerShell-Automation-5391FE?logo=powershell&logoColor=white)
![Access Reviews](https://img.shields.io/badge/Access-Reviews-success)
![CSV Export](https://img.shields.io/badge/Output-CSV-orange)
![Status](https://img.shields.io/badge/Project-Completed-brightgreen)

## Overview

This project focuses on **automating the extraction of Microsoft Entra ID Access Review decisions** using **PowerShell** and the **Microsoft Graph API**.

The goal of this lab was to move from a manual governance process to an automated one by authenticating to Microsoft Graph, retrieving Access Review definitions, instances, and decisions, and exporting the results into a **CSV file** for reporting and analysis.

This project demonstrates how automation can strengthen **Identity Governance** by making access review data easier to audit, track, and analyze.

---

## Real-World Scenario

Imagine working in an IAM or security governance team where quarterly access reviews must be completed for privileged users or sensitive groups.

Manually checking every reviewer decision is time-consuming and error-prone, especially when many accounts are involved.

This automation helps the team to:

- automatically retrieve Access Review decision data,
- consolidate reviewer and target user information,
- export results into a reusable CSV format,
- support governance meetings and audit preparation,
- identify unusual cases such as missing justifications or pending decisions.

---

## Project Objectives

- Authenticate securely to Microsoft Graph with PowerShell
- Query Access Review definitions and review instances
- Retrieve Access Review decision results
- Export the results into CSV for analysis and reporting

---

## Technologies Used

- **Microsoft Entra ID**
- **Microsoft Graph API**
- **PowerShell**
- **MSAL.PS**
- **CSV / Excel**

---

## What I Built

In this home lab, I built a PowerShell-based workflow that:

1. authenticates interactively to Microsoft Graph,
2. lists Access Review definitions,
3. retrieves the target review instance,
4. collects decision results,
5. exports the decision data to a CSV file.

This simulates a realistic IAM governance automation task for reporting and audit support.

---

## Lab Prerequisites

Before starting Week 4, I already had:

- a Microsoft Entra tenant,
- a test Access Review created in a previous lab,
- a test group / test users,
- a reviewer configured in Microsoft Entra ID Governance.

This project builds directly on the Access Review created in **Week 3**.

---

## Implementation Steps

### 1. App Registration in Microsoft Entra ID

To allow PowerShell to authenticate against Microsoft Graph, I created an **App Registration** in Microsoft Entra ID.

Configuration used:

- **Single tenant**
- **Public client / native application**
- Redirect URI configured for interactive authentication

<img width="500" height="303" alt="image" src="https://github.com/user-attachments/assets/0d4f0f91-5b83-4069-a912-3d80c3abefb2" />


This app registration provided:

- **Application (client) ID**
- **Directory (tenant) ID**

<img width="1100" height="247" alt="image" src="https://github.com/user-attachments/assets/99afd6d4-89bd-444c-8537-381cd0686dd3" />


These values were later used inside the PowerShell script.

---

### 2. Microsoft Graph API Permission

I added the following delegated Microsoft Graph permission:

- `AccessReview.Read.All`

This permission allows the script to read Access Review data such as definitions, instances, reviewers, and decisions.

Admin consent was granted so the permission could be used properly during authentication.

<img width="1111" height="381" alt="image" src="https://github.com/user-attachments/assets/b257e399-4e33-4e80-a7d1-07101e128148" />


---

### 3. PowerShell Module Setup

I installed the required PowerShell components:

- **NuGet provider**
- **MSAL.PS module**

During setup, PowerShell prompted for:

- NuGet provider installation
- PSGallery trust confirmation

After accepting both prompts, the module installation completed successfully.

---

### 4. Fixing the Authentication Issue

During authentication testing, I encountered the following error:

- `AADSTS50011: The redirect URI 'https://login.microsoftonline.com/common/oauth2/nativeclient' specified in the request does not match the redirect URIs configured for the application '4e81d661-7c54-408d-a937-af4011a0b87c'. Make sure the redirect URI sent in the request matches one added to your application in the Azure portal. Navigate to https://aka.ms/redirectUriMismatchError to learn more about how to fix this.`

The issue came from the fact that the redirect URI used by the authentication flow did not match the redirect URI configured in the app registration.

To fix it, I added the correct redirect URI for the public client flow:

https://login.microsoftonline.com/common/oauth2/nativeclient

I also kept http://localhost as an additional valid redirect URI.

<img width="1118" height="474" alt="image" src="https://github.com/user-attachments/assets/c65693d9-c452-4eab-b51f-328bda024127" />

After fixing this, the authentication flow worked correctly.

---

###5. Interactive Authentication Test

I first created a small authentication test script to verify that the connection worked before querying Graph.

The script used:

Get-MsalToken -TenantId $tenantId -ClientId $clientId -Scopes $scope -Interactive

Once the script returned a non-empty access token, I confirmed that:

the app registration was valid,
the client ID and tenant ID were correct,
the redirect URI issue was resolved,
interactive authentication was working.

A token length of around 2430 characters confirmed that a real access token had been issued.

<img width="349" height="43" alt="TEST1" src="https://github.com/user-attachments/assets/5d869fa2-81cf-4ff9-8146-d2a7138fb8d9" />


---

### 6. Listing Access Review Definitions

Before exporting decisions, I needed to retrieve the correct Access Review Definition ID.

I used Microsoft Graph to list all review definitions and identify the relevant one by its display name.

Example endpoint used:

GET https://graph.microsoft.com/v1.0/identityGovernance/accessReviews/definitions

This step allowed me to identify the review definition associated with my first project Access Review.

<img width="1051" height="446" alt="ACCESS_review_def" src="https://github.com/user-attachments/assets/7e5dfd3e-262a-4d8a-914b-c84354a0c49a" />

<img width="548" height="97" alt="test2" src="https://github.com/user-attachments/assets/5b354e83-09c1-4094-a3de-5fc7992d87d0" />


---

### 7. Retrieving the Access Review Instance

Once the definition ID was known, I queried the review instances to get the correct instance ID.

Example endpoint used:

GET https://graph.microsoft.com/v1.0/identityGovernance/accessReviews/definitions/{definitionId}/instances

This provided the active or most recent review instance to target.

<img width="548" height="97" alt="test2" src="https://github.com/user-attachments/assets/7a7b5494-5e7f-466b-a57a-514c8d2856b4" />

---

### 8. Exporting Access Review Decisions

Finally, I queried the decision items and exported the results into a CSV file.

Example endpoint used:

GET https://graph.microsoft.com/v1.0/identityGovernance/accessReviews/definitions/{definitionId}/instances/{instanceId}/decisions

This output can be used for:

audit evidence,
governance review meetings,
anomaly detection,
recurring access review reporting.
Final PowerShell Script

<img width="754" height="753" alt="image" src="https://github.com/user-attachments/assets/82519cc0-7ee2-4a6b-b96b-645e567cd620" />


## Anomaly Detection for Governance Meetings


To make the export more useful for governance teams, I added a simple anomaly detection step after the CSV export.

The purpose is to help managers and reviewers quickly identify unusual or potentially risky review outcomes during governance meetings, such as:

approved access without justification,
reviews that were not completed,
decisions that do not match the recommendation,
accounts that may still keep unnecessary access.

This makes the report more actionable by helping teams focus on access decisions that may require manual validation, remediation, or privilege reduction.

Instead of reviewing every single line manually, governance stakeholders can directly investigate a smaller subset of suspicious or incomplete decisions.

This approach supports:

access governance meetings,
audit preparation,
privilege cleanup,
reduction of unnecessary access exposure.

I need to add a script code highlighted in red square. 
After exporting the Access Review results to CSV, I added a second step to generate an anomaly-focused report.

## Detection rules used


The script flags entries when:

decision = Approve and justification is empty
decision = NotReviewed
recommendation and decision do not match

## Why the Anomaly Report Matters


The anomaly report helps governance and IAM teams focus on decisions that may introduce unnecessary risk.

For example:

an approval without justification may indicate weak review quality,
a pending or missing review may indicate a governance gap,
a decision that contradicts the recommendation may require managerial follow-up.

This additional report improves the operational value of the automation by turning raw review data into a more decision-oriented governance output.

The given generated CSV file Result looks like:

<img width="1134" height="160" alt="image" src="https://github.com/user-attachments/assets/9385410e-b905-469d-ba4d-a35527ff77e7" />

and for the  generated CSV file Anomalies:

<img width="1396" height="119" alt="image" src="https://github.com/user-attachments/assets/05cc0ba8-8b85-441b-8ceb-cd8718a9fa16" />



---

## Key IAM Concepts Practiced
This project helped reinforce several important IAM concepts:

Identity Governance
Access Reviews
Delegated Microsoft Graph permissions
Interactive authentication
Public client / native application
OAuth 2.0 / OIDC concepts
JWT access tokens
CSV-based governance reporting
Why This Project Matters

This automation project shows how IAM teams can reduce manual governance work by using scripts and APIs to retrieve review data consistently.

Instead of manually opening every review in the portal, teams can generate structured outputs and build repeatable governance processes around:
access certification,
privileged access validation,
compliance evidence,
governance reporting.

---

## Possible next steps include:

automatically looping through all Access Review definitions,

filtering for the latest active instance,

replacing interactive authentication with a more production-ready workflow.
