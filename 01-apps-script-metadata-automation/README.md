# Apps Script Metadata Automation

Automates daily collection of Meta Ads (Facebook/Instagram) performance data via the Meta Marketing API, aggregating results across multiple ad accounts and campaigns into Google Sheets.

메타 애즈 API를 통해 여러 광고 계정·캠페인의 성과 데이터를 자동으로 취합하여 구글시트에 정리하는 앱스크립트 프로젝트입니다.

## Overview

Previously, checking Meta Ads performance meant manually logging into Ads Manager, pulling reports for each account/campaign, and consolidating them by hand — a repetitive process prone to delay and error. This project automates that entire workflow.

## What It Does

- Pulls campaign performance data from the **Meta Marketing API**
- Aggregates results across **multiple ad accounts and campaigns** in a single run
- Writes the data into **Google Sheets** in a structured, ready-to-use format
- Runs automatically every day at **00:05**, updating the **previous day's** data via a time-driven trigger

## Tech Stack

`Google Apps Script` `Meta Marketing API` `Google Sheets API` `Time-driven Triggers`

## How It Works

1. A time-based trigger runs the script daily at 00:05.
2. The script calls the Meta Marketing API for each configured ad account/campaign.
3. Responses are parsed and normalized into a consistent row format.
4. Data is appended/updated in the target Google Sheet.

## Notes

- API credentials (access tokens, account IDs) are managed separately and are **not included** in this repository.
- Sheet structure and account/campaign IDs shown in the code are illustrative; actual values are configured privately.
