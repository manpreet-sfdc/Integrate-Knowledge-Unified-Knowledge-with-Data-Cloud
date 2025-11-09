# Integrate-Knowledge-Unified-Knowledge-with-Data-Cloud

A complete guide, architecture, and step-by-step implementation checklist for bringing Salesforce Knowledge (and Unified Knowledge from external sources) into Data Cloud so you can power grounded AI (RAG), richer self-service, and better agent experiences.

1 — Overview

This repository documents how to integrate Salesforce Knowledge (your internal articles) and Unified Knowledge (external third-party sources brought into Salesforce) with Data Cloud (formerly Customer 360 / Data 360) so that knowledge content is stored as Data Cloud objects (DMOs), indexed, and available for AI grounding, searches, and analytics. 
Salesforce
+1

2 — What’s new / Why this matters (quick)

You can now ingest first-party (Salesforce Knowledge) and third-party knowledge (SharePoint, Confluence, Google Drive, websites, etc.) into Data Cloud to create a single knowledge foundation for AI and service experiences. 

Knowledge content is mapped into Data Cloud DMOs (Knowledge Article DMO + related engagement DMOs), enabling indexing, RAG grounding and analytics. 

Article size limits and indexing: Data Cloud supports much larger article sizes (up to ~100 MB), but content above certain sizes (≈25 MB) may not be indexed for search—plan accordingly. 


3 — Architecture (high level)

```
Third-party sources (SharePoint, Confluence, Websites, Google Drive, etc.)
        ↓   (Unified Knowledge connectors, crawlers, or ingestion)
Salesforce (Unified Knowledge layer + Enhanced Knowledge) 
        ↓   (Sync / Data Stream)
Data Cloud (Knowledge Article DMO, Knowledge Engagement DMO, etc.)
        ↓
AI & Service Layer (Einstein for Service, RAG pipelines, AgentAssist, Chatbots)
        ↓
Channels: Service Console, Self-Service Portal, Mobile Field Service, Bots

```

Key components:

Unified Knowledge: connector/ingestion layer to bring external knowledge into Salesforce. 

Enhanced Knowledge: Salesforce Knowledge configuration that can "Sync with Data Cloud". 

Data Cloud DMOs: Knowledge Article DMO, Engagement DMO, Knowledge_kav mapping for versions. 


4 — Prerequisites & licensing

Salesforce org with Data Cloud enabled (Data Cloud / Customer 360 entitlement). 

Enhanced Knowledge / Unified Knowledge access (Unified Knowledge initially rolled out as a product announcement and subject to GA/beta and edition limitations). Confirm availability for your edition. 

Admin access to Setup, Data Cloud Admin, and permissions to create Data Streams & DMO mappings.

If ingesting external sources, credentials and API access for connectors (SharePoint, Confluence, Google Drive, website crawler) and any partner connectors (Zoomin or third-party vendors). 


5 — Implementation — step-by-step

The following steps describe an end-to-end flow from preparing content to making it available for RAG/Einstein.

Step 0 — Plan & audit

Inventory all knowledge sources (Salesforce articles, SharePoint spaces, Confluence spaces, Google Drive folders, websites, Slack transcripts, product docs).

Decide metadata taxonomy: categories, topics, tags, locales, article statuses, product lines, audience (agent/public).

Decide on the indexing policy and which fields must be searchable. Flag very large content (≥25 MB) for special handling.


Step 1 — Prepare Data Cloud & Enable sync

Set up Data Cloud in your org (Data Cloud admin setup and enable Data Streams). Follow Data Cloud enablement docs for your org. 
Salesforce

In Setup → Enhanced Knowledge Settings (or Knowledge settings), enable Sync With Data Cloud (this will create the Knowledge DMOs and enable Data Stream mapping). Follow the "Sync Knowledge with Data Cloud" help guide. 
Salesforce
+1

Step 2 — Ingest Salesforce Knowledge content into Data Cloud

In Data Cloud, go to Data Streams → Salesforce CRM and select Knowledge as a source. Review and adjust the field mapping. The connector includes standard mapping for Knowledge fields; you may need to manually map custom fields. 
Salesforce
+1

Confirm the Knowledge_kav (Knowledge Article Version) mapping and that the Knowledge Article DMO is created and populated. 
Salesforce Developers

Sample checklist:

 Article title, url, article number mapped

 Rich text body (HTML) mapped to DMO field for full text ingestion

 Status, language, product, tags, publish date, last modified date mapped

 Attachments and asset links—decide whether to store as separate objects or external links


Step 3 — Connect & ingest external sources (Unified Knowledge)

For each external source, configure the appropriate connector (SharePoint, Confluence, Google Drive, website crawler or partner connector such as Zoomin). Unified Knowledge aims to centralize such connectors. 


Ensure the connector pushes content into Salesforce’s Knowledge (or into a staging area) and that Enhanced Knowledge / Unified Knowledge maps it to the Knowledge Article DMO in Data Cloud.

Apply metadata normalization: unify categories/tags across source systems.


Step 4 — DMO mappings, transformations & normalization

Use Data Cloud mapping tools to adjust DMO field mappings, normalizations, and enrichment (e.g., add product metadata, region, language). Example DMOs you will see:

```
ssot__KnowledgeArticle__dlm / ssot__KnowledgeArticleEngagement__dlm (engagements). 
```

Create derived fields for RAG – e.g., searchable_text that concatenates title, summary, body, metadata, and important tags for retrieval vectors.

For content longer than indexing thresholds, extract summaries and/or chunk documents for indexing.


Step 5 — Indexing & search configuration

Configure Data Cloud / Search indexing to include the Knowledge DMO fields you want searchable (title, summary, body, tags, product). Note: very large documents may not be fully indexed—extract and index summaries or segments for RAG-friendly retrieval. 


Set up periodic reindexing or incremental updates when articles are published or updated.


Step 6 — Expose for AI (RAG) and Einstein for Service

Configure your RAG pipelines (Einstein for Service or your custom generative pipeline) to ground responses using Data Cloud Knowledge DMOs as the retrieval corpus instead of (or in addition to) CRM-only sources. This reduces hallucinations and improves relevance. 

If using Einstein for Service: switch grounding to Data Cloud (follow the feature notes in the release notes to enable Data Cloud grounding). 

For custom LLM setups, build a retrieval layer that queries Data Cloud (via supported APIs or connectors) to fetch top-k relevant Knowledge articles, then pass them as context to the LLM.


Step 7 — Surface content to channels

Agents: Surface knowledge suggestions in the Service Console via Einstein recommendations.

Self-Service: Configure Knowledge components on Experience Cloud / portals to use Data Cloud search.

Field Service / Mobile: Build or update mobile components to call Data Cloud retrieval when offline/online.

Step 8 — Monitoring, governance & ongoing operations

Monitor article usage & engagement via the Knowledge Article Engagement DMO. Use Data Cloud dashboards to spot stale content or unused articles. 

Establish a cadence for content review, retention, and TTL for third-party content.

Track indexing errors and content ingestion failures.

6 — Data mappings, DMOs & examples
Example: Common DMO fields (conceptual)

```
{
  "ssot_Id_c": "KA-000123",
  "title": "How to reset device X",
  "body_html": "<p>Step 1 ...</p>",
  "summary": "Reset instructions for device X",
  "language": "en_US",
  "status": "Published",
  "publish_date": "2025-05-01T12:00:00Z",
  "tags": ["troubleshooting","device-x"],
  "source": "Salesforce_Knowledge", // or SharePoint
  "attachment_links": ["https://..."]
}

```

Knowledge Article Version mapping

Data Cloud connector provides Knowledge_kav mappings for versioned article data; ensure you ingest the latest published version and store versions if needed. 



7 — Using the unified knowledge for RAG / Einstein for Service

Retrieval step: Query Data Cloud DMOs for top-N relevant documents (use semantic retrieval or text search).

Augment step: Optionally chunk long articles and add metadata filters (locale, product).

Generation step: Pass retrieved chunks as grounding context to the LLM (Einstein or custom LLM).

Cite sources: Return answers with citations to the Knowledge Article IDs/URLs used for grounding. This increases trust and allows agents/customers to click through.

Notes: Switching grounding to Data Cloud improves coverage (first + third-party content) and the accuracy of AI replies. Confirm the exact Einstein for Service option for Data Cloud grounding in your release notes. 


8 — Best practices & governance

Taxonomy first: Invent consistent tags and categories before ingesting many sources.

Chunking strategy: Break very long articles into logical chunks for retrieval; keep chunk size balanced for LLM context windows.

Index the right things: Index summaries, key headings, and conclusions rather than huge blobs.

Monitor usage: Use the Knowledge Article Engagement DMO to track relevance and decay. 

Security & access: Ensure Data Cloud respects article visibility (internal vs public). Don’t expose internal docs to public channels.

Licensing & editions: Confirm GA/beta status and license requirements before production roll-out. 


9 — Troubleshooting checklist

Articles not appearing in Data Cloud?

Confirm Enhanced Knowledge → Sync With Data Cloud is enabled. 


Check Data Stream logs for the Salesforce CRM connector and mapping errors. 


Search results missing content or truncated:

Check document size; content > ~25 MB may not be indexed—create indexed summaries. 


Wrong versions showing:

Verify Knowledge_kav (version) mapping and ensure you're using the published version. 

External connector failures:

Validate connector credentials, API limits, and ingestion schedules (SharePoint/Confluence tokens often expire).

10 — References & docs (start here)

Salesforce unified knowledge announcement (May 2024). 

Sync Knowledge with Data Cloud (how-to, release note snippet). 

Knowledge Article DMO docs (Data Cloud DMO mapping). 

Knowledge_kav mapping & Salesforce CRM connector for Data Cloud. 

Industry & blog coverage of Unified Knowledge + Data Cloud. 


Appendix — Example snippet: create a searchable_text derived field (pseudo)

This example demonstrates how to create a derived field that concatenates title + summary + top 2 headings + tags for better retrieval:

```
-- pseudo / conceptual (Data Cloud transformation)
UPDATE DMO_KnowledgeArticle
SET searchable_text = CONCAT(title, ' ', COALESCE(summary,''), ' ', COALESCE(extract_top_headings(body_html,2), ''), ' ', COALESCE(tags, ''))
WHERE last_modified > DATE_SUB(NOW(), INTERVAL 30 DAY)

```

