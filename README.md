# Managing Evolving Contracts

Managing an evolving schema is a critical challenge in software development and data engineering. It's the process of modifying the structure of your data (database tables, API requests/responses, file formats, etc.) without breaking existing functionality.

There is no single solution, but rather a set of principles, strategies, and tools. The approach varies significantly depending on whether you're dealing with a database schema, an API schema, or a data contract for streaming.

Here’s a comprehensive breakdown of how to manage evolving schema.

- Core Philosophy: The Rules of the Game
  
1. The golden rule is never to make a breaking change. A breaking change forces all consumers (other services, applications, reports) to update simultaneously. Your goal is to make changes **backward compatible** and, where possible, **forward compatible.**

i. Backward Compatibility:
Older clients/code can work with the new schema. (e.g., Adding a new optional field to an API response).

ii. Forward Compatibility: Newer clients/code can gracefully handle data from the old schema. (e.g., A new client ignores a deprecated field it doesn't need yet).

## Strategies and Techniques by Technology
1. For Relational Databases (e.g., PostgreSQL, MySQL)

Database schema changes are risky because they can lock tables and cause downtime.

2. Additive Changes Only (Initially): This is the safest strategy.

i. ADD COLUMN: Add new columns as NULL or with a sensible default value (DEFAULT ...). This doesn't break existing INSERT or SELECT * statements.

ii. Create New Tables: Instead of modifying an existing table, create a new one and link it with a foreign key. This is common for extending functionality.

iii. Avoid Destructive Changes: Never DROP a column or table immediately.

## Deprecation Process:

i. Stop writing to the column in application code.

ii. Migrate the data out or ensure no critical reads are left.

iii. Monitor logs to confirm the column is unused for a significant period.

iv. Then, and only then, DROP the column. This process can take months.

## Use Migration Tools: Never run ALTER TABLE commands manually on production.

Tools: Use dedicated tools like Liquibase, Flyway, or Django Migrations.

## How they help: 
i. They script every schema change, version these scripts, and apply them in a controlled, predictable way across all environments (Dev -> Staging -> Prod). They also often support rolling back changes if something goes wrong.

ii. Expressive Column Names: Name new columns clearly. Avoid flag_1, flag_2; use is_account_verified instead.

## For APIs (REST, GraphQL)
- APIs have public contracts that many external or internal clients depend on.

i. Versioning: The most common way to manage breaking changes.

ii. URL Path: /api/v1/users -> /api/v2/users

- Query Parameter: /api/users?version=2

- Header: Accept: application/vnd.myapi.v2+json

- Recommendation: URL path versioning is the most explicit and easiest to debug.

- Additive Changes: Just like databases, add new fields to requests and responses. Never remove or rename existing fields.

- Deprecation: Mark old fields as deprecated in your documentation and use tools like Swagger/OpenAPI to indicate this. Log warnings when deprecated fields are used to encourage clients to migrate.

## The Expand and Contract Pattern (Parallel Change):

This is a gold-standard, zero-downtime technique.

- Expand: Deploy a new version of your service that supports both the old and new field/schema simultaneously.

i. Migrate: Update all clients to use the new field. This can be done gradually.

- Contract: Once you are sure no client is using the old field, deploy a version of the service that removes the old field.

## GraphQL Advantages: 

GraphQL is inherently more flexible.

i. Clients request only the fields they need, so adding new fields doesn't affect existing queries.

ii. Deprecation is a built-in feature (@deprecated directive).

Breaking changes are still possible (e.g., removing a field without deprecating it), so discipline is still required.

## For Streaming & Big Data (e.g., Kafka, Avro, Parquet)


This is where schema management is most formalized due to the decoupled nature of producers and consumers.

- Schema Registry:

This is the cornerstone technology. A Schema Registry (e.g., Confluent Schema Registry, AWS Glue Schema Registry) is a standalone server that holds the canonical version of all schemas.

- How it works:

i. Producers and consumers don't send the raw schema with every message. Instead, they send a tiny schema ID. The consumer uses this ID to fetch the correct schema from the registry to deserialize the message.

- Compatibility Checking: The registry can be configured to enforce compatibility rules (e.g., BACKWARD, FORWARD, FULL) when a producer tries to register a new schema version. If the new schema breaks the rules, the registry rejects it, preventing a bad deployment.

- Using Avro/Protobuf:

These binary formats are preferred over JSON for streaming because they are compact, fast, and, most importantly, require a schema to serialize/deserialize. The schema is the contract.

## Data Lakes (Parquet/ORC):

For file formats like Parquet, the strategy is again additive.

i. New columns can be added to the end of the schema.

Tools like Apache Spark and Trino are very good at reading Parquet files with evolving schemas, often allowing you to project old data with new schema definitions.

## General Best Practices

- Communication: Document every change and communicate deprecations widely to developer teams.

- Testing: Automate compatibility testing in your CI/CD pipeline. Tools like Schema Registry can be integrated to check compatibility before deployment.

- Monitoring: Monitor for usage of deprecated fields or endpoints. This data informs you when it's safe to finally remove something.

- Culture: Foster a team culture that understands the cost of breaking changes and values backward compatibility.

In summary, managing an evolving schema is not a technical problem alone; it's a **process** and **communication** problem solved with a disciplined, additive approach, powerful tooling (migration tools, schema registries), and well-defined patterns like Expand and Contract.
