---
title: "How to use System SCORE functions as inter-call"
excerpt: ""
---

## Overview

This document shows how a SCORE uses the external functions of System SCORE via inter-call.
A SCORE can stake, delegate and claim-iscore instead of a user through System SCORE.

## Intended Audience

ICON dApp developers who want to implement a SCORE which uses the IISS API features that System SCORE provides

## Purpose

After reading this document, you can understand how your SCORE can use the external functions of System SCORE
and implement a SCORE which can stake, delegate or claim-iscore instead of a user.

## Prerequisites

We assume that you are familiar with SCORE development.
If you do not have enough knowledge, please read the below documents:

* [SCORE Overview](doc:score-overview) 
* [SCORE Quickstart](doc:score-quickstart)
* [Writing SCORE](doc:writing-score)
* [Token & Crowdsale](doc:token-crowdsale) - Reference implementation

## Make your SCORE support IISS features
