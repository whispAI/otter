# otter 🦦

OpenSearch and Lucene-based DBMS
benefit from cost-savings via external stor-
age locations, such as UltraWarm which
relies on AWS S3. These alternative
tiers, however, are unsuitable for many
use-cases as indices are immutable. We
present Otter, the OpenSearch Tier Triage
and Edit Router. Otter splits indices into
small fragments which can be easily mi-
grated between hot and UltraWarm tiers.
By providing an asynchronous endpoint
for write operations, Otter dynamically
manages fragments, enabling update and
delete operations on UltraWarm indices,
bypassing immutability.

## 1 Introduction
OpenSearch, Elasticsearch, and other Lucene-
based database management systems are highly
performant for full-text and semantic search appli-
cations. Scaling of such systems, however, proves
resource-intensive due to the requirement that data
is stored on-node, as opposed to cost-effective
storage solutions such as Amazon S3. Amazon
have made some headway in this regard with the
introduction of AWS OpenSearch UltraWarm.
Unfortunately, the UltraWarm tier (UW) was de-
signed primarily for logging use-cases, and in-
dices stored within are immutable.

We present Otter, the OpenSearch Tier Triage
and Edit Router, a system to handle read and write
requests to an OpenSearch cluster with UltraWarm
nodes. Otter enables documents to be stored in an
UW tier, without becoming immutable. In prac-
tice, Otter allows administrators and systems ar-
chitects to avail of cost-savings associated with
UltraWarm nodes, without the need to navigate in-
dex lifecycle policies and migration.

## 2 Conceptual Overview
Otter is designed to sit in front of a cluster, as a
single endpoint for all CRUD operations. It acts
as a pass-through service for queries and read op-
erations, with marginal latency impact. For write
operations, the system operates asynchronously.
Otter treats clusters as single index systems,
wherein all documents must adhere to a single
index mapping, and CRUD operations assume a
monolithic index.

While a single index is used for API calls, Ot-
ter uses a fragmentation process to split an in-
dex into numerous smaller fragments whose size
is determined by the speed of Hot-UltraWarm mi-
gration. All fragments are served by a single alias
for ease of operation. Fragments are migrated to
the UltraWarm tier once a pre-determined thresh-
old is reached, known as Time Since Edit (TSE).
This enables migration at speeds several orders of
magnitude faster, while read/query operations re-
main unaffected.

Update and deletion operations are handled
asynchronously. Should the fragment of a target
document reside in hot storage, the request is re-
layed to the cluster near-instantaneously. How-
ever, if said fragment is in UltraWarm, the migra-
tion API is called and the request is placed in a
message queue. Once returned to hot storage, re-
quests are processed in a First-In-First-Out (FIFO)
manner. This resets the TSE clock.
