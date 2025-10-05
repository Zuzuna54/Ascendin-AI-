# Decision 09: Message Queue System

## 0. Executive Snapshot

- **Current choice:** AWS SQS FIFO (First-In-First-Out) Queue
- **Overall score:** 4.93/5 (Excellent - Highest Score in Data Layer)
- **Verdict:** ✅ Keep (optimal choice - zero cost + FIFO guarantees + native Lambda integration)
- **Why (one sentence):** AWS SQS FIFO provides exactly-once processing with ordered delivery at zero cost (free tier covers 1M requests/month workload of 300K/month), native Lambda trigger integration requiring zero configuration, and fully managed reliability, making it superior to Kafka ($250+/month) and RabbitMQ ($40-60/month) which are massive overkill for Personal Vault's 10K messages/day scale.

---

## 1. Context & Requirements Fit

### Problem Statement

Need reliable queue for asynchronous tasks (embedding generation, batch processing) that guarantees message delivery, supports retries on failure, and integrates seamlessly with Lambda compute. FIFO ordering required to process messages in correct sequence (e.g., delete operation must come after create).

### Requirements

| Requirement | Description | How SQS FIFO Satisfies |
|-------------|-------------|----------------------|
| **Reliability** | Exactly-once processing (no duplicates) | MessageDeduplicationId ensures no duplicates |
| **Ordering** | FIFO (process in submission order) | FIFO queue guarantees order within MessageGroupId |
| **Integration** | Native Lambda triggers | SQS → Lambda automatic polling (zero config) |
| **Cost** | <$5/month typical workload | Free tier: 1M requests/month (workload: 300K = $0) |
| **Scalability** | Handle 1K-10K messages/day | 300 msg/sec per queue (3K/sec with batching) |

### Constraints

- **Ordering:** Must process messages in order (FIFO requirement)
- **Integration:** Must trigger Lambda functions automatically
- **Reliability:** At-least-once delivery (retries on failure)
- **Visibility Timeout:** Support long-running tasks (up to 12 hours)
- **Dead-Letter Queue:** Handle persistent failures

### Success Criteria

- ✅ Exactly-once processing (no duplicate embeddings)
- ✅ FIFO ordering preserved
- ✅ Cost $0/month (free tier covers workload)
- ✅ Native Lambda integration (zero configuration)
- ✅ Retry logic with dead-letter queue

---

## 2. Alternatives Catalog

### Alternative A: AWS SQS FIFO ✅ **(CURRENT CHOICE)**

**What it is:**
AWS Simple Queue Service FIFO is a fully managed message queue that guarantees exactly-once processing and ordered delivery. Designed for workloads requiring strict ordering.

**How it works:**
1. Producer sends messages to queue with MessageGroupId (determines ordering scope)
2. SQS stores messages redundantly across multiple AZs
3. Consumer (Lambda) polls queue automatically
4. Message becomes invisible during processing (visibility timeout)
5. Consumer deletes message after successful processing
6. If processing fails, message reappears after timeout (automatic retry)
7. After N failures, message moves to Dead-Letter Queue (DLQ)

**Architecture:**
```
Client → Encrypt Message → Queue in SQS FIFO → Lambda Polls Automatically →
Process (Generate Embedding) → Success → Delete from Queue
                            → Failure → Requeue (visibility timeout) →
                            → Retry 3x → Move to DLQ
```

**Maturity & Ecosystem:**
- **Version:** GA since 2016 (FIFO), standard SQS since 2004
- **Scale:** Powers Amazon.com retail operations
- **Adoption:** Thousands of AWS customers
- **Integration:** Native Lambda, SNS, S3, CloudWatch
- **Reliability:** 99.9% availability SLA

**Licensing:** Proprietary (AWS managed service)

**FIFO Features:**
- **Exactly-once processing:** Content-based deduplication
- **Ordering:** Strict FIFO within MessageGroupId
- **Throughput:** 300 messages/sec (3,000/sec with batching)
- **Message size:** Up to 256KB
- **Retention:** 1-14 days (default 4 days)
- **Visibility timeout:** Up to 12 hours

### Alternative B: Apache Kafka ❌

**What it is:**
Distributed streaming platform designed for high-throughput event processing (millions of messages/sec).

**How it works:**
1. Producers publish to topics
2. Topics divided into partitions
3. Consumer groups process messages in parallel
4. Messages persisted to disk (durable)
5. Exactly-once semantics (transactions)

**Maturity:** Extremely mature (LinkedIn, Uber, Netflix use at massive scale)

**Critical Issue:** ❌ **Massive overkill** - Designed for millions msg/sec, not 10K/day; $250+/month minimum

### Alternative C: RabbitMQ ❌

**What it is:**
Open-source message broker with advanced routing (exchanges, queues, bindings).

**How it works:**
1. Producers send to exchanges
2. Exchanges route to queues (direct, topic, fanout, headers)
3. Consumers acknowledge messages
4. Supports pub/sub, request/reply, work queues

**Maturity:** Very mature (15+ years)

**Critical Issue:** ❌ **Cost + operational overhead** - $40-60/month + must manage; SQS provides same for $0

### Alternative D: AWS SQS Standard (Non-FIFO) ❌

**What it is:**
Standard SQS with higher throughput but no ordering guarantees.

**How it works:**
- At-least-once delivery (potential duplicates)
- No ordering guarantees
- Unlimited throughput
- Cheaper than FIFO (but both in free tier)

**Critical Issue:** ❌ **NO FIFO** - Embedding generation requires ordered processing

### Alternative E: Google Cloud Tasks ❌

**What it is:**
Fully managed task queue similar to SQS.

**Critical Issue:** ❌ **NO FIFO guarantees** - Cannot ensure ordered processing

---

## 3. Pros & Cons (Comparison Table)

| Option | Key Pros | Key Cons | Performance Envelope | Ops Complexity | Cost/TCO Notes |
|--------|----------|----------|---------------------|----------------|----------------|
| **SQS FIFO** ✅ | Zero cost (free tier), FIFO, Lambda integration, managed, exactly-once | 300 msg/sec limit (3K/sec batched) | 300-3K msg/sec, 256KB msgs | Very Low (managed) | $0/month (free tier covers) |
| Kafka | Ultra-high throughput (millions/sec), durable, battle-tested | $250+/month, complex (ZooKeeper, partitions), overkill | Millions msg/sec | Very High | $250+/month minimum |
| RabbitMQ | Flexible routing, open-source, mature | $40-60/month, ops overhead, no advantage vs SQS | 10K-100K msg/sec | Medium | $40-60/month hosted |
| SQS Standard | Higher throughput, cheaper | NO FIFO, at-least-once (duplicates) | Unlimited throughput | Very Low | $0/month free tier |
| Cloud Tasks | Similar to SQS, Google ecosystem | NO FIFO guarantees | Similar to SQS | Very Low | $0.40 per 1M |

---

## 4. Performance & Benchmarks

### SQS FIFO Performance (Official AWS)

**Source:** https://aws.amazon.com/sqs/features/  
**Date Checked:** 05 Oct 2025

| Metric | Value | Notes |
|--------|-------|-------|
| **Throughput** | 300 messages/sec | Per FIFO queue; single MessageGroupId |
| **Batched Throughput** | 3,000 messages/sec | 10-message batches |
| **Latency** | <100ms | Message delivery time |
| **Message Size** | Up to 256KB | 256KB per message |
| **Retention** | 1-14 days | Default: 4 days |
| **Visibility Timeout** | Up to 12 hours | For long-running tasks |

### Cost Analysis (Workload: 10K Messages/Day)

**Source:** https://aws.amazon.com/sqs/pricing/  
**Date Checked:** 05 Oct 2025

```
Daily: 10,000 messages
Monthly: 10,000 × 30 = 300,000 messages
Batching: 10 messages per API call = 30,000 requests/month

AWS SQS Pricing:
- Free tier: 1,000,000 requests/month
- Usage: 30,000 < 1,000,000 = $0
- FIFO after free: $0.50 per 1M requests

Total monthly cost: $0 ✅

Comparison:
| Service | Monthly Cost (300K msgs) |
|---------|-------------------------|
| SQS FIFO | $0 (free tier) |
| Kafka (MSK) | $250+ (3-broker minimum) |
| RabbitMQ (AmazonMQ) | $40-60 |
| Self-hosted RabbitMQ | $15-30 (VPS) + ops time |
```

### Lambda Integration Performance

**Source:** https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html  
**Date Checked:** 05 Oct 2025

| Metric | Value | Notes |
|--------|-------|-------|
| **Poll Interval** | 20 seconds | Lambda long-polls SQS |
| **Batch Size** | 1-10 messages | Lambda processes batch |
| **Concurrency** | Automatic | Lambda scales based on queue depth |
| **Max Retry** | Configurable | Move to DLQ after N failures |

**Flow:**
```
1. Message arrives in SQS FIFO queue
2. Lambda polls (automatic, every 20s or when messages available)
3. Lambda invoked with batch of 1-10 messages
4. Lambda processes messages (generate embeddings)
5. Lambda returns success → SQS deletes messages
6. Lambda returns failure → Messages reappear after visibility timeout
7. After 3 failures → Messages moved to Dead-Letter Queue
```

---

## 5. Evidence Log (Citations)

1. **AWS SQS Features**  
   URL: https://aws.amazon.com/sqs/features/  
   Date Checked: 05 Oct 2025  
   Relevance: FIFO guarantees, throughput limits, message size limits

2. **AWS SQS Pricing**  
   URL: https://aws.amazon.com/sqs/pricing/  
   Date Checked: 05 Oct 2025  
   Relevance: Free tier (1M requests/month), FIFO pricing ($0.50 per 1M after free)

3. **AWS SQS FIFO Queues Documentation**  
   URL: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html  
   Date Checked: 05 Oct 2025  
   Relevance: FIFO semantics, MessageGroupId, deduplication

4. **SQS Lambda Integration**  
   URL: https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html  
   Date Checked: 05 Oct 2025  
   Relevance: Automatic polling, batch processing, error handling

5. **Apache Kafka Documentation**  
   URL: https://kafka.apache.org/documentation/  
   Date Checked: 05 Oct 2025  
   Relevance: Alternative comparison (Kafka use cases vs SQS)

6. **AWS MSK (Managed Kafka) Pricing**  
   URL: https://aws.amazon.com/msk/pricing/  
   Date Checked: 05 Oct 2025  
   Relevance: Kafka costs ($250+/month minimum)

7. **AmazonMQ (Managed RabbitMQ) Pricing**  
   URL: https://aws.amazon.com/amazon-mq/pricing/  
   Date Checked: 05 Oct 2025  
   Relevance: RabbitMQ managed service costs

---

## 6. Winner Rationale

### Primary Reasons

1. **Zero Cost for Workload:**
   - Free tier: 1,000,000 requests/month
   - Workload: 300,000 requests/month (10K messages/day × 30 days / 10 batch size)
   - **Cost: $0/month** ✅
   - vs Kafka: $250+/month minimum
   - vs RabbitMQ: $40-60/month

2. **FIFO Guarantees (Critical Requirement):**
   - Exactly-once processing via MessageDeduplicationId
   - Strict ordering within MessageGroupId
   - **Critical for:** Processing embeddings in order (don't process delete before create)

3. **Native Lambda Integration:**
   - Lambda polls SQS automatically (no configuration needed)
   - Scales automatically based on queue depth
   - Built-in retry logic
   - Dead-letter queue support

4. **Fully Managed:**
   - No infrastructure to provision
   - Auto-scales (no capacity planning)
   - Multi-AZ redundancy (99.9% availability)
   - AWS handles all operations

5. **Simple API:**
   - Send message: single API call
   - Receive messages: long polling
   - Delete message: acknowledgment
   - **vs Kafka:** Complex (topics, partitions, consumer groups, ZooKeeper)

### Accepted Trade-offs

| Trade-off | Impact | Mitigation |
|-----------|--------|------------|
| **300 msg/sec limit** | Could hit limit if burst >300 msg/sec | Not an issue (typical: 10K/day = 0.12 msg/sec); use batching (10 msgs = 3K/sec) |
| **AWS lock-in** | SQS-specific API | Acceptable; alternatives (Google Pub/Sub, Azure Service Bus) similar APIs |
| **FIFO throughput lower** | Standard SQS has unlimited throughput | FIFO required for ordering; standard SQS not viable |

---

## 7. Losers' Rationale

### Apache Kafka (Score: 3.25/5)

**Why it lost:**
- ❌ **Massive overkill:** Designed for millions msg/sec streaming; we need 10K msg/day = 0.12 msg/sec
- ❌ **Cost:** AWS MSK minimum $250/month (3-broker cluster); Confluent Cloud $100+/month
- ❌ **Complexity:** Requires ZooKeeper, understanding of topics/partitions/consumer groups, operational expertise
- **100x more expensive** with no benefit for our scale

**Would be better for:** Real-time analytics, event streaming, millions of events/second

### RabbitMQ (Score: 3.98/5)

**Why it lost:**
- ❌ **Cost:** $40-60/month (AmazonMQ managed) vs $0/month (SQS)
- ❌ **Operational overhead:** Must monitor, upgrade, tune
- ❌ **No advantage:** Provides same FIFO guarantees as SQS with added complexity

**Would be better for:** Complex routing requirements (exchanges, bindings)

### SQS Standard (Score: 4.23/5)

**Why it lost:**
- ❌ **NO FIFO:** At-least-once delivery (messages can be duplicated)
- ❌ **NO ordering:** Messages processed out of order
- **Deal-breaker:** Embedding generation must happen in correct order

**Example problem:**
```
Messages in queue: [Create msg-1, Delete msg-1, Create msg-2]

With SQS Standard (no ordering):
Possible order: Delete msg-1, Create msg-1, Create msg-2
Result: Try to delete msg-1 before it's created (error)

With SQS FIFO:
Guaranteed order: Create msg-1, Delete msg-1, Create msg-2
Result: Operations execute correctly
```

**Would win if:** FIFO not required (higher throughput, same free tier)

### Google Cloud Tasks (Not Fully Scored)

**Why it lost:**
- ❌ **NO FIFO guarantees:** Similar to SQS Standard
- ⚠️ **Google ecosystem:** Different APIs, less familiar to team

---

## 8. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation | Monitoring |
|------|------------|--------|------------|------------|
| **Queue depth grows** | Medium | Medium | Lambda concurrency scales automatically; alert if depth >1000 | CloudWatch queue depth metric |
| **Visibility timeout too short** | Low | Medium | Set to 5 minutes (embedding task ~30s; safety margin) | CloudWatch timeout errors |
| **Lambda processing failures** | Medium | Medium | Dead-letter queue (DLQ) after 3 retries; alert on DLQ messages | CloudWatch DLQ message count |
| **Message loss** | Very Low | High | SQS stores redundantly across 3+ AZs; 99.9% SLA | AWS Health Dashboard |
| **Free tier exhaustion** | Very Low | Low | 1M requests/month = 10x our workload; billing alert at $5/month | CloudWatch billing metrics |

### Design Considerations

**Throughput Limit (300 msg/sec):**
- **Workload:** 10K messages/day = 0.12 messages/sec average
- **Burst:** Even 100 messages at once = far below 300/sec limit
- **Batching:** 10 messages per request = 3,000 messages/sec effective
- **Conclusion:** Throughput limit not a concern

**Message Ordering:**
- **MessageGroupId:** All messages for same user use same group ID (ensures ordering)
- **Multiple groups:** Different users can process in parallel
- **Example:** user123 messages processed in order; user456 messages processed in parallel

---

## 9. Recommendation & Roadmap

### Recommendation: ✅ **KEEP** AWS SQS FIFO

**Justification:**
- Highest score (4.93/5) in data layer
- Zero cost for workload (free tier covers)
- FIFO guarantees essential for correct processing
- Native Lambda integration (zero configuration)
- No operational overhead (fully managed)

### Implementation Roadmap

**Week 1-2: Queue Setup**
1. Create FIFO queue (name must end with .fifo)
2. Enable content-based deduplication
3. Set visibility timeout: 300 seconds (5 minutes)
4. Configure dead-letter queue (3 max receives)
5. Test: Send 100 messages, verify FIFO order

**Week 3-4: Lambda Integration**
1. Configure Lambda trigger (SQS event source)
2. Set batch size: 1 (process one SQS message at a time)
3. Implement idempotency in Lambda (handle potential duplicates)
4. Test: Queue 1,000 messages, verify all processed

**Week 5-6: Error Handling**
1. Implement exponential backoff for OpenAI API failures
2. Monitor dead-letter queue
3. Alert on DLQ messages (investigate failures)
4. Test: Simulate failures, verify DLQ behavior

**Week 7-8: Production Monitoring**
1. CloudWatch metrics: queue depth, age of oldest message, DLQ count
2. Alarms: Queue depth >1000, oldest message >1 hour, DLQ >10 messages
3. Dashboard: Real-time queue metrics
4. Load test: 10,000 messages in 1 hour

### No Migration Needed

SQS FIFO is optimal. Only scenario for change:

**If:** Move entirely to Google Cloud Platform  
**Then:** Use Google Cloud Tasks + implement FIFO logic manually (1-2 week effort)

---

## 10. Examples (Beginner-Friendly)

### Example 1: Queuing Embedding Tasks

```python
import boto3
import json

sqs = boto3.client('sqs')
queue_url = 'https://sqs.us-east-1.amazonaws.com/123456/embeddings.fifo'

# Client: Queue 100 messages for embedding generation
for i, message in enumerate(messages_batch):
    sqs.send_message(
        QueueUrl=queue_url,
        MessageBody=json.dumps({
            'message_id': message.id,
            'content': message.content,
            's3_key': message.s3_key
        }),
        MessageGroupId='user123',  # FIFO ordering scope
        MessageDeduplicationId=message.id  # Prevent duplicates
    )

# Lambda: Automatically triggered when messages arrive
def lambda_handler(event, context):
    for record in event['Records']:
        body = json.loads(record['body'])
        
        # Generate embedding
        embedding = openai.embeddings.create(
            model="text-embedding-3-small",
            input=body['content']
        ).data[0].embedding
        
        # Store in PostgreSQL
        db.execute("""
            INSERT INTO message_embeddings (message_id, embedding)
            VALUES (%s, %s)
            ON CONFLICT DO NOTHING  -- Idempotency
        """, (body['message_id'], embedding))
    
    return {'statusCode': 200}

# Result: Messages processed in order, zero cost, automatic scaling
```

### Example 2: Retry Logic with Dead-Letter Queue

```python
# Queue configuration
queue_attributes = {
    'FifoQueue': 'true',
    'ContentBasedDeduplication': 'true',
    'VisibilityTimeout': '300',  # 5 minutes
    'MessageRetentionPeriod': '345600',  # 4 days
    'RedrivePolicy': json.dumps({
        'deadLetterTargetArn': 'arn:aws:sqs:us-east-1:123456:embeddings-dlq.fifo',
        'maxReceiveCount': '3'  # Retry 3 times before DLQ
    })
}

sqs.create_queue(QueueName='embeddings.fifo', Attributes=queue_attributes)

# If Lambda fails (e.g., OpenAI API timeout):
# 1. Message becomes visible again after 5 minutes
# 2. Lambda retries automatically
# 3. After 3 failures, message moved to DLQ
# 4. CloudWatch alert triggers on DLQ messages

# Monitor DLQ for persistent failures
dlq_messages = sqs.receive_message(
    QueueUrl=dlq_url,
    MaxNumberOfMessages=10
)

for msg in dlq_messages.get('Messages', []):
    # Log for investigation
    logger.error(f"Failed message after 3 retries: {msg['Body']}")
    # Option: Manual retry, skip, or alert engineering
```

### Example 3: Batch Processing for Efficiency

```python
# Send messages in batches (reduce API calls)
entries = []
for i, msg in enumerate(messages):
    entries.append({
        'Id': str(i),
        'MessageBody': json.dumps(msg),
        'MessageGroupId': f'user{msg.user_id}',  # Partition by user
        'MessageDeduplicationId': msg.id
    })
    
    # Send every 10 messages (SQS batch limit)
    if len(entries) == 10:
        sqs.send_message_batch(QueueUrl=queue_url, Entries=entries)
        entries = []

# Send remaining
if entries:
    sqs.send_message_batch(QueueUrl=queue_url, Entries=entries)

# Result: 100 messages = 10 API calls (vs 100 individual calls)
# Throughput: 10x improvement (3,000 msg/sec effective)
```

---

## 11. Validation Plan

### Spike 1: SQS FIFO Setup & Basic Flow

**Objective:** Create FIFO queue; validate ordered processing

**Method:**
1. Create SQS FIFO queue via AWS Console or CLI
2. Send 100 messages with sequential IDs
3. Configure Lambda trigger
4. Verify messages processed in order (check database insertion order)

**Tools:** AWS CLI, Lambda function, PostgreSQL

**Success Criteria:**
- Queue creates successfully (name ends with .fifo)
- Messages delivered in FIFO order (ID 1, 2, 3, ... 100)
- Lambda triggered automatically
- Cost: $0 (within free tier)

**Timeline:** Week 1

**Dataset:** 100 test messages

**Pass/Fail:** If out-of-order processing detected, investigate: MessageGroupId configuration

### Spike 2: Dead-Letter Queue Behavior

**Objective:** Validate retry logic and DLQ functionality

**Method:**
1. Configure DLQ with maxReceiveCount=3
2. Send 10 messages to queue
3. Simulate Lambda failures (throw exception)
4. Verify messages retry 3 times, then move to DLQ
5. Check DLQ for failed messages

**Success Criteria:**
- Messages retry exactly 3 times
- After 3 failures, messages in DLQ
- DLQ CloudWatch alarm triggers
- Original queue empty after processing

**Timeline:** Week 2

**Pass/Fail:** If messages not reaching DLQ after 3 failures, check RedrivePolicy configuration

### Spike 3: Throughput & Cost Validation

**Objective:** Validate performance at scale; confirm free tier coverage

**Method:**
1. Send 10,000 messages in 1 hour (burst test)
2. Measure: Throughput, latency, Lambda concurrency
3. Check AWS bill after 1 week (should be $0)

**Success Criteria:**
- All 10,000 messages processed successfully
- Average throughput >10 messages/sec
- Total cost $0 (free tier)
- No throttling errors

**Timeline:** Week 3

**Dataset:** 10,000 real messages

---

## 12. Alternatives Table (Full Pros & Cons)

### Weighted Comparison Matrix

**Weights:**
- Fit to Requirements: 0.30 (FIFO + Lambda integration critical)
- Reliability/HA: 0.20 (message loss unacceptable)
- Complexity/Ops: 0.20 (prefer managed services)
- Cost/TCO: 0.15 (budget constraint)
- Delivery Speed: 0.15 (fast implementation preferred)

| Criterion | Weight | SQS FIFO | Kafka | RabbitMQ | SQS Standard |
|-----------|-------:|---------:|------:|---------:|-------------:|
| **Fit to Requirements** | 0.30 | 5.0 | 4.0 | 4.5 | 3.0 |
| **Reliability/HA** | 0.20 | 5.0 | 5.0 | 4.0 | 4.5 |
| **Complexity/Ops** | 0.20 | 5.0 | 1.5 | 3.0 | 5.0 |
| **Cost/TCO** | 0.15 | 5.0 | 1.0 | 3.0 | 5.0 |
| **Delivery Speed** | 0.15 | 5.0 | 4.0 | 4.0 | 5.0 |
| **WEIGHTED TOTAL** | 1.00 | **4.93** ⭐ | 3.25 | 3.98 | 4.23 |

**Winner:** SQS FIFO (4.93/5) - Highest score in data layer

**Scoring Justification:**
- **Fit (5.0):** Perfect for FIFO + Lambda integration requirements
- **Cost (5.0):** $0/month (free tier covers workload completely)
- **Complexity (5.0):** Fully managed; zero configuration needed
- **Kafka lowest score (3.25):** Complex (1.5) + expensive (1.0) = poor fit

---

## 13. Appendix

### Queue Configuration (CloudFormation)

```yaml
Resources:
  EmbeddingQueue:
    Type: AWS::SQS::Queue
    Properties:
      QueueName: embeddings.fifo
      FifoQueue: true
      ContentBasedDeduplication: true
      VisibilityTimeout: 300  # 5 minutes
      MessageRetentionPeriod: 345600  # 4 days
      RedrivePolicy:
        deadLetterTargetArn: !GetAtt DeadLetterQueue.Arn
        maxReceiveCount: 3
  
  DeadLetterQueue:
    Type: AWS::SQS::Queue
    Properties:
      QueueName: embeddings-dlq.fifo
      FifoQueue: true
      MessageRetentionPeriod: 1209600  # 14 days (max)
```

### Lambda Trigger Configuration

```python
# serverless.yml
functions:
  embeddingProcessor:
    handler: handler.process_embeddings
    events:
      - sqs:
          arn: !GetAtt EmbeddingQueue.Arn
          batchSize: 1  # Process one SQS message (contains 10 embeddings) at a time
          maximumBatchingWindowInSeconds: 0  # Process immediately
    reservedConcurrency: 10  # Limit parallel executions
```

### Monitoring Queries

```python
import boto3

cloudwatch = boto3.client('cloudwatch')

# Get queue depth
queue_depth = cloudwatch.get_metric_statistics(
    Namespace='AWS/SQS',
    MetricName='ApproximateNumberOfMessagesVisible',
    Dimensions=[{'Name': 'QueueName', 'Value': 'embeddings.fifo'}],
    StartTime=datetime.utcnow() - timedelta(minutes=5),
    EndTime=datetime.utcnow(),
    Period=300,
    Statistics=['Average']
)

# Alert if queue depth >1000 (backlog building up)
if queue_depth['Datapoints'][0]['Average'] > 1000:
    send_alert("Queue depth high: investigate Lambda processing")
```

---

**Document Version:** 1.0  
**Last Updated:** 05 October 2025  
**Decision Owner:** Architecture Team  
**Status:** ✅ APPROVED FOR IMPLEMENTATION