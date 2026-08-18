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

Architecture
        Same architecture pattern, one library per granularity level
┌────────────────────┐  ┌────────────────────┐  ┌────────────────────┐
│  Campaign Library   │  │  Ad Set Library     │  │  Ad Creative Library│
│  (this repo)        │  │  level: "adset"     │  │  level: "ad"         │
│  · Retry/backoff     │  │  · Retry/backoff     │  │  · Retry/backoff     │
│  · Pagination         │  │  · Pagination         │  │  · Pagination         │
│  · Action normalize   │  │  · Action normalize   │  │  · Action normalize   │
│  · Logging, mapping   │  │  · Logging, mapping   │  │  · Logging, mapping   │
└──────────┬───────────┘  └──────────┬───────────┘  └──────────┬───────────┘
           │                          │                          │
           └──────────────┬───────────┴──────────────┬───────────┘
                           │  imported as Apps Script libraries
                ┌──────────┼──────────────┬──────────────┐
                ▼          ▼               ▼              ▼
            Client A    Client B       Client C   ...  Client N
        (own Apps Script project: config object,
         daily time-driven trigger, imports the level(s) it needs)
Data store: Google Sheets (per-client spreadsheet, referenced by TARGET_SHEET_ID)
Data granularity: this repository is the campaign-level library (level: "campaign"). Ad set and ad creative levels are separate sibling libraries built on the same core pattern (retry logic, pagination, logging, config-driven design), differing mainly in the Graph API level parameter and the fields requested.
Trigger: each client's Apps Script project runs the imported library(ies) daily via a time-driven trigger, pulling the previous day's data.

## Notes

- API credentials (access tokens, account IDs) are managed separately and are **not included** in this repository.
- Sheet structure and account/campaign IDs shown in the code are illustrative; actual values are configured privately.

## My Role

Designed and built this shared library from scratch — the API integration, retry/rate-limit strategy, metric normalization logic, and the campaign-mapping system — and established the reusable architecture pattern that the sibling ad set– and ad creative–level libraries are also built on. Rolled it out as a reusable Apps Script library consumed by per-client scripts, now running daily in production across 6+ client accounts.

## What I Learned / Challenges
Meta's rate limits are aggressive and inconsistently signaled — some errors return HTTP 4xx with an ambiguous error code rather than a clean 429, which required parsing the error body to detect "soft" rate-limit conditions (code 4, code 613, keyword matching on the error message) before deciding whether to retry.
Action-type data is not standardized across campaign objectives — the same conversion event can appear under several different action_type strings depending on objective/optimization goal, which required building a normalization layer (sumByTypes + per-metric candidate lists) instead of relying on a single field name.
Designing for reuse across many clients pushed the library toward a strict config-driven pattern (no hardcoded sheet names, IDs, or tokens) so new clients can be onboarded by adding a config object rather than duplicating code.

## Version History
Library	Version	Date
Campaign-level (this repo)	v2.08	2026-02-02
Ad Set-level	v2.09	2026-03-12
Ad Creative-level	v2.07	2026-01-12

## 프로젝트 요약 (한국어)
무엇: 메타 광고(페이스북/인스타그램) 캠페인 데이터를 매일 자동 수집해 구글 시트에 적재하는 Google Apps Script 공용 라이브러리
구조: 캠페인/세트/소재 레벨별로 동일한 아키텍처를 공유하는 독립 라이브러리가 각각 존재하며, 이 저장소는 캠페인 레벨 라이브러리. 각 클라이언트는 자신의 앱스크립트 프로젝트에서 필요한 레벨의 라이브러리를 가져다 씀 (멀티테넌트 구조)
핵심 기능: 레이트리밋 대응 재시도(지수 백오프+지터), 페이지네이션 처리, 전환 액션 타입 정규화, 캠페인 카테고리 자동 매핑, 실행 로그 기록
규모: 6개 이상 클라이언트 계정에 적용, 매일 Time-driven Trigger로 자동 실행 중
기술: Google Apps Script, Meta Graph API v23.0, Google Sheets API
