# Dockerised elasticsearch v7 cluster

## Background
For this task, the team upgraded their elasticsearch docker containers from version 6 to version 7. Since the replication between the two versions were different, I was required to fix their existing replication scripts.

Our infrastructure consists of three servers. Two servers have a passive-active relationship, and the other is responsible for coordinating the promotion and demotion of the servers. All data must be the same for both servers, therefore it was required to sync data across the elasticsearch docker containers between the two servers.

## Overview
This documentation 