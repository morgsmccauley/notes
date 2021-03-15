# GraphQL Schema Stitching

Schema stitching combines multiple GraphQL schemas in to a single schema. The schemas can either be remote or local.

[graphql-tools](https://graphql-tools.com) is a javascript library which exposes a number of different helper methods for creatings/manipulating/stitching/merging GraphQL schemas. The project is a mono-repo with each package aimed at achieving different things.

## Stitching vs. Merging
With graphql-tools these have subtle differences. Merging schemas essentially combines the type definitions and resolvers of multiple `GraphQLSchema`s in to a single resource. Stitching schemas gives you more control over combining the schemas and allows you to create extensions and define conflict resolution functions. This is helpful when there are duplicate types or links across schemas.
