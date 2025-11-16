# Solr 9.7.0 deprecated API audit

## SolrMetricProducer#initializeMetrics(SolrMetricsContext, String)
* **Context.** `SolrMetricProducer` exposes the two-argument `initializeMetrics` signature that accepts an explicit scope. Solr commit history prior to 9.x noted that the scope argument would be retired once all producers used child `SolrMetricsContext`s.
* **Evidence it still ships.** `SolrCoreMetricManager#registerMetricProducer` (lines 123-140) still invokes `producer.initializeMetrics(solrMetricsContext, scope)` and carries an inline note `// use deprecated method for back-compat, remove in 9.0`. The interface itself lives in `solr/core/src/java/org/apache/solr/metrics/SolrMetricProducer.java` and is still the only contract implementors can satisfy.
* **Why it matters.** Downstream plugins that depend on Solr 9.7.0 can still call the legacy overload even though operator guidance since 9.0 promised its removal. Any effort to delete the scope parameter would now be a breaking change because the shipped public interface still requires it.

## Overseer internal work queue helpers
* **Context.** The `Overseer.ClusterStateUpdater` historically mirrored operations from the live `/overseer/queue` into a `/overseer/queue-work` safety net. Zookeeper pressure fixes in the 8.x line marked the second queue for deletion in 9.0.
* **Evidence it still ships.** In `solr/core/src/java/org/apache/solr/cloud/Overseer.java` lines 184-224, the `workQueue` field is still constructed with `getInternalWorkQueue(zkClient, zkStats)` next to the comment `// TODO remove in 9.0, we do not push message into this queue anymore`. Every helper that exposes the redundant queue (`getInternalWorkQueue`, `getWorkQueueStats`) remains public.
* **Why it matters.** The unused work-queue API continues to leak Zookeeper ACL surface area in cloud deployments and prevents the cleanup promised to operators upgrading from 8.x.
