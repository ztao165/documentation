---
layout: ../../../layouts/MainLayout.astro
section: anyApi
date: Last Modified
title: "Data Provider Nodes"
permalink: "docs/any-api/data-providers/introduction/"
---

Chainlink has facilitated the launch of several new oracle data services that allow dApps to access rich data from external data sources. These oracles expand the types of data available on Chainlink and include data and use cases that do not fit into the traditional Chainlink feed model.

Data provider nodes allow customizable, on-demand API calls and the ability to deliver different data points on each request. This enables use cases where each requestor requires a different piece of data.

:::caution[Select quality data feeds]
Be aware of the quality of the data that you use. [Learn more about making responsible data quality decisions.](/data-feeds/selecting-data-feeds/)
:::

## Data Provider Nodes List

| Data Category | Name                                                             | Supported Blockchains        |
| ------------- | ---------------------------------------------------------------- | ---------------------------- |
| DNS Lookup    | [DNS Ownership Oracle](/any-api/data-providers/dns-ownership/)   | Ethereum, BNB Chain, Polygon |
| Weather       | [Google Weather Oracle](/any-api/data-providers/google-weather/) | Ethereum                     |
