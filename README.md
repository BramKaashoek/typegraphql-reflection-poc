# TypeGraphQL reflection (PoC)

Proof of concept of an enhanced reflection system for TypeScript.

## How it works?

I uses a custom transform plugin and TS compiler API with a bit of help from [`ts-morph`](https://github.com/dsherret/ts-morph) to read deeply nested generic types info for GraphQL schema making purposes.

## What does it do?

It can read type definitions of this class declaration:

```ts
@ObjectType
export class Sample {
  @Field
  dateField!: Date;

  @Field
  optionalStringField?: string;

  @Field
  optionalStringArrayField?: string[];

  @Field
  nullableStringArrayField!: Array<string | null>;

  @Field
  nullableStringPromiseField!: Promise<string | null>;

  @Field
  nullableStringNullableArrayPromiseField!: Promise<Array<string | null> | null>;
}
```

And emit that metadata, which then allows to read and transform them into this GraphQL type definition (SDL):

```graphql
type Sample {
  dateField: Date!
  optionalStringField: String
  optionalStringArrayField: [String!]
  nullableStringArrayField: [String]!
  nullableStringPromiseField: String
  nullableStringNullableArrayPromiseField: [String]
}
```

## How to run it?

You can run the example by `npm start` - it will run a `ts-node` with a special `ttypescript` compiler flag (`ts-node -C ttypescript src/example/index.ts`) that allows to apply the plugins defined in `tsconfig.json` during the TS files transpilation.

The transform plugin files are located in `src/transformer` directory. The example code (class source file and reading the metadata) are in `src/example` folder and the related supporting code is in `src/helpers` dir. In `src/sdl` you can find some simple helpers for creating GraphQL type SDL string for this demo purposes.

## Further work

This proof of concept supports only basic subset of types - primitive types, array, nullability.

There is a lot of work to do next, both on the reflection plugin side and the TypeGraphQL core side.

If you like this idea of making TypeGraphQL easier to use without all the additional boilerplate, please consider supporting the project and our development efforts!

[![](https://opencollective.com/typegraphql/donate/button.png?color=blue)](https://opencollective.com/typegraphql)

- detecting wrong types (e.g. TS interfaces) at compile time
- methods supports (query/mutation)
  - detecting return types
  - detecting arg names and their types
  - allowing usage of args without `@Arg()` or `@Args()` decorator (method args are implicitly the graphql args)
- union support
  - detecting unions in field types and return types (`Promise<Author | Movie | Book>`)
  - auto generating GraphQL union name using query/mutation name or type and field name (with customization support)
  - detecting literal unions (`"YES" | "NO`) and auto generating graphql enums from them
  - detecting wrong union types (e.g. `ColorEnum | string`) at compile time
- enum support
  - detecting enums in field types and methods return types
  - support for declaring descriptions and depreciations using comments or JSDoc
- generic types support
  - collecting type param names and positions in generic class
  - collecting type params usage in generic class fields and return types
  - detecting generic types usage in fields and return types in normal properties and methods
  - detecting type param names and positions in generic types usage (both object types, input and args types)
  - detecting wrong type params usage (e.g. TS interfaces) at compile time
  - generating dedicated graphql types based on generic types usage by substituting type params on corresponding generic positions
  - figure out good naming convention for concrete GraphQL types of generic types (`Edge<User>` = `UserEdge`, `Connection<User, UserArgs>` = ???) + naming customization support
- JSDoc support
  - reading info from code comments and applying it as `description` in GraphQL schema
