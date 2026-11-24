# Investigation and Evidence

## Investigative question

Can a controlled file change on the lab endpoint be detected in real time, matched by a built-in Wazuh rule, and found as the same event in the indexer?

The live experiment answered this question for creation, modification, and deletion of one synthetic file.

## Timeline

All timestamps below are UTC on November 24, 2026.

| Operation | Endpoint action | Manager alert | Rule | First indexed observation |
| --- | --- | --- | --- | --- |
| Create | 22:17:50.357 | 22:17:50.752 | 554 | 9.76 seconds after action |
| Modify | 22:17:59.944 | 22:18:00.339 | 550 | 10.80 seconds after action |
| Delete | 22:18:10.729 | 22:18:11.117 | 553 | 16.46 seconds after action |

The first indexed observation includes polling and API query overhead. Clock alignment also influences cross-system timestamp comparisons. These are single-run observations.

## Rule interpretation

| Rule | Event | Level | Observed meaning |
| --- | --- | --- | --- |
| 554 | Added | 5 | The monitored file appeared |
| 550 | Modified | 7 | The monitored file's integrity attributes changed |
| 553 | Deleted | 7 | The monitored file was removed |

The modification event listed size, modification time, and checksum changes. All events reported `realtime` mode.

These events resulted from authorized training actions. Rule severity provides an investigation signal. Determining malicious intent requires the surrounding activity and authorization context.

## Hash evidence

The created file's SHA-256 was:

```text
9318894ef031add1a321e237b3dc3148bbcd79f6c1063a9ffbfa009a5f1adde9
```

After modification, its SHA-256 was:

```text
ec72eb32063fc660d192a538e58ec6e428c1eb4c7e284254d7fe6965f8a6a496
```

The modification alert's `sha256_before` matched the original value, and `sha256_after` matched the modified file. The deletion alert retained the modified file's last recorded hash. The endpoint's independently calculated hashes matched the reported values.

## Search the evidence

In the Wazuh dashboard, select a time window covering the exercise and filter by the actual lab agent and test path. An initial event search can use:

```text
agent.name:"LAB_AGENT_NAME" AND rule.groups:"syscheck"
```

Replace `LAB_AGENT_NAME` with the deployed lab endpoint name. Use the actual unique test filename when narrowing the results.

An equivalent indexer search uses `wazuh-alerts-*` and filters on both the agent name and `syscheck.path`. Retrieve the rule, timestamp, event type, and hashes, then compare the document's event ID privately with the manager alert.

The project performed this comparison through authenticated indexer API queries. The published [results](../evidence/results.json) record successful matches, hashes, and timings. Actual event IDs, addresses, and unrelated logs are omitted.

## Outcome and scope

The experiment verified the agent-to-manager-to-indexer path for all three file operations. The watched directory was empty afterward. It used built-in rules and the real-time FIM module.

Attribution of the person or process responsible for a change requires additional audit context. This experiment establishes file-change detection and evidence consistency. A subsequent attribution exercise can add who-data monitoring and compare its evidence.
