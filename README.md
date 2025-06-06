# Translatable Criteria

[![Development Stage](https://img.shields.io/badge/Development-Early%20Stage-yellow)]()

A TypeScript library for building data-source agnostic, translatable query criteria. Define complex filtering, ordering, and join logic in a structured, type-safe way, then translate it to your specific data source using custom translators.

## Installation

```bash
  npm install translatable-criteria
```

## Overview

This library simplifies the construction of complex data queries by providing a consistent and abstract way to define filtering, ordering, and relationship (joins) configurations. The core concept revolves around the `Criteria` object, which allows developers to define sophisticated query specifications in a data source-agnostic manner. These `Criteria` objects can then be processed by a `CriteriaTranslator` to generate queries for various data sources.

## Key Features

- **Type-Safe Criteria Building:** Construct queries with a fluent, strongly-typed interface.
- **Powerful Filtering:** Define intricate filtering logic with multiple operators and grouping.
- **Flexible Join System:** Support for various join types (inner, left, full outer) and pivot table configurations.
- **Data Source Agnostic:** Design criteria independently of the underlying data source.
- **Translator-Based Architecture:** Implement custom `CriteriaTranslator` instances to convert criteria into specific query languages (e.g., SQL, NoSQL queries, ORM-specific queries).
- **Full TypeScript Support:** Benefit from compile-time validation and autocompletion.

## Core Concepts

### CriteriaTranslator (Interface)

The library provides a `CriteriaTranslator` interface. You (or the community) will implement this interface to convert `Criteria` objects into queries for specific data sources (e.g., SQL, TypeORM QueryBuilder, MongoDB queries).

## Usage Example

### Schemas

Schemas define the structure, available fields, and potential join relationships of your data entities. This enables type-safe criteria construction.

## Usage Example

```typescript
import { GetTypedCriteriaSchema } from 'repository-criteria-interface'; // Adjust path if needed
export const PostSchema = GetTypedCriteriaSchema({
  source_name: 'post',
  alias: ['posts'],
  fields: ['uuid', 'title', 'body', 'user_uuid'],
  joins: [
    { alias: 'comments', join_relation_type: 'one_to_many' },
    { alias: 'publisher', join_relation_type: 'many_to_one' },
  ],
});

export const CommentSchema = GetTypedCriteriaSchema({
  source_name: 'comment',
  alias: ['comments'],
  fields: ['uuid', 'comment_text', 'post_uuid', 'user_uuid', 'comment_uuid'],
  joins: [
    { alias: 'comments', join_relation_type: 'one_to_many' },
    { alias: 'user', join_relation_type: 'many_to_one' },
  ],
});

export const UserSchema = GetTypedCriteriaSchema({
  source_name: 'user',
  alias: ['users', 'publisher', 'user'],
  fields: ['uuid', 'email', 'username', 'direction_uuid'],
  joins: [
    {
      alias: 'permissions',
      join_relation_type: 'many_to_many',
    },
    {
      alias: 'address',
      join_relation_type: 'one_to_many',
    },
    {
      alias: 'posts',
      join_relation_type: 'one_to_many',
    },
  ],
});

export const PermissionSchema = GetTypedCriteriaSchema({
  source_name: 'permission',
  alias: ['permissions'],
  fields: ['uuid', 'name'],
  joins: [
    {
      alias: 'users',
      join_relation_type: 'many_to_many',
    },
  ],
});

export const DirectionSchema = GetTypedCriteriaSchema({
  source_name: 'direction',
  alias: ['address'],
  fields: ['uuid', 'direction', 'user_uuid'],
  joins: [
    {
      alias: 'users',
      join_relation_type: 'many_to_one',
    },
  ],
});
```

### Criteria

Build type-safe query specifications using the `Criteria` class.

## Usage

### Basic Example

```typescript
import { Criteria, FilterOperator } from 'repository-criteria-interface';
const criteria = Criteria.Create(UserSchema, 'users')
  .where({
    field: 'email',
    operator: FilterOperator.EQUALS,
    value: 'contact@nelsoncabrera.dev',
  })
  .join(Criteria.CreateInnerJoin(PermissionSchema, 'permissions'), {
    parent_to_join_relation_type: 'many_to_many',
    join_source_name: 'permission_user',
    join_field: { pivot_field: 'permission_uuid', reference: 'uuid' },
    parent_join_field: { pivot_field: 'user_uuid', reference: 'uuid' },
  });
```

### Advanced Joins Example with order and offset pagination

```typescript
const criteriaWithJoins = Criteria.Create(PostSchema, 'posts')
  .where({
    field: 'title',
    operator: FilterOperator.LIKE,
    value: 'New NPM Package Released',
  })
  .join(
    Criteria.CreateInnerJoin(CommentSchema, 'comments').join(
      Criteria.CreateInnerJoin(UserSchema, 'user'),
      {
        parent_to_join_relation_type: 'many_to_one',
        parent_field: 'uuid',
        join_field: 'uuid',
      },
    ),
    {
      parent_to_join_relation_type: 'one_to_many',
      parent_field: 'uuid',
      join_field: 'uuid',
    },
  )
  .orderBy('uuid', 'ASC')
  .setSkip(10)
  .setTake(3);
```

### Many-to-Many Relationship Example

```typescript
const userCriteriaWithPermissions = Criteria.Create(UserSchema, 'users')
  .where({
    field: 'email',
    operator: FilterOperator.EQUALS,
    value: 'contact@nelsoncabrera.dev',
  })
  .join(Criteria.CreateLeftJoin(PermissionSchema, 'permissions'), {
    parent_to_join_relation_type: 'many_to_many',
    join_source_name: 'permission_user', // pivot table name
    join_field: {
      pivot_field: 'permission_uuid',
      reference: 'uuid',
    },
    parent_join_field: {
      pivot_field: 'user_uuid',
      reference: 'uuid',
    },
  });
```

## Type Safety Features

- Compile-time validation of field names within criteria based on schemas
- Type-checked join configurations ensuring compatibility between schemas.
- Autocomplete support for schema fields and defined join aliases.
- Validation of join parameters based on schema definitions (simple vs. pivot joins).

## Roadmap

- [ ] Provide example translator implementations (e.g., for a common ORM or SQL dialect).
- [ ] Implement cursor pagination ability
- [ ] Enhance documentation with more detailed examples for translator development.
- [ ] Add more complex filtering capabilities (e.g., nested logical groups directly in where clauses).
- [ ] Explore utility functions to simplify translator development.
- [ ] Add more comprehensive test coverage for criteria construction and edge cases.

## Contributing

This project is in early development. Contributions are welcome! Please feel free to submit a Pull Request on our [GitHub repository](https://github.com/Techscq/translatable-criteria).

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Author

Nelson Cabrera ([@Techscq](https://github.com/Techscq))
