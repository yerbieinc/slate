---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - java

toc_footers:
  - <a href='https://www.yerbie.dev'>Yerbie Home</a>

includes:
  - quickstart
  - retrying
search: true

code_clipboard: true
---

# Introduction

Yerbie is a job queue and scheduler. A system like Yerbie is used to distribute work across threads or machines, or to schedule code to run at a certain point in the future.

To initiate a job, a client sends a request to Yerbie indicating a queue, data and delay, and Yerbie then makes that data available on the queue after the specified delay.

A worker polls, and runs a job associated with the data when it is available on the queue.

Currently, Yerbie has a Java client library with more language support planned.

