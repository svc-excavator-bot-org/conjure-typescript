<p align="right">
<a href="https://autorelease.general.dmz.palantir.tech/palantir/conjure-typescript"><img src="https://img.shields.io/badge/Perform%20an-Autorelease-success.svg" alt="Autorelease"></a>
</p>


# conjure-typescript ![NPM](https://img.shields.io/npm/v/conjure-typescript.svg?label=conjure-typescript) [![License](https://img.shields.io/badge/License-Apache%202.0-lightgrey.svg)](https://opensource.org/licenses/Apache-2.0)


_CLI to generate TypeScript Interfaces and clients from [Conjure API definitions](https://github.com/palantir/conjure)._

## Overview

The generated clients provide a simple Promise based interface for executing strongly typed remote procedure calls from
the browser or node.

## Usage
The recommended way to use conjure-typescript is via a build tool like [gradle-conjure](https://github.com/palantir/gradle-conjure).
However, if you don't want to use gradle-conjure, there is also an executable which conforms to [RFC 002](https://github.com/palantir/conjure/blob/develop/rfc/002-contract-for-conjure-generators.md),  published on [bintray](https://bintray.com/palantir/releases/conjure-typescript).

```
conjure-typescript generate <input> <output> [..Options]

Generate TypeScript bindings for a Conjure API

Positionals:
  input   The location of the API IR
  output  The output directory for the generated code

Options:
  --version                Show version number                                                         [boolean]
  --help                   Show help                                                                   [boolean]
  --packageVersion         The version of the generated package                                         [string]
  --packageName            The name of the generated package                                            [string]
  --nodeCompatibleModules  Generate node compatible javascript                        [boolean] [default: false]
  --rawSource              Generate raw source without any package metadata           [boolean] [default: false]
  --productDependencies    Path to a file containing a list of product dependencies                     [string]
```

## SemVer releases

This project is versioned according to [SemVer](https://semver.org/). We consider the generated code to be part of
the 'public API' of conjure-typescript, i.e. TypeScript generated by old conjure-typescript and new
conjure-typescript (within a major version) should be compatible, so your consumers should be able to upgrade without compilation problems.

We also consider the command line interface and feature flags to be public API.


## Example generated objects

- **Conjure object: [ManyFieldExample](./src/commands/generate/__tests__/resources/types/manyFieldExample.ts)**

  Objects can easily be instantiated:

    ```typescript
    const example: ManyFieldExample = {
      string: "foo",
      integer: 123,
      optionalItem: "bar",
      items: []
    }
    ```

- **Conjure union: [UnionTypeExample](./src/commands/generate/__tests__/resources/types/unionTypeExample.ts)**

    Union types can be one of a few variants. To interact with a union value, users should use the `.accept` method and define a Visitor that handles each of the possible variants, including the possibility of an unknown variant.

    ```typescript
    const unionExample = IUnionTypeExample.string("Hello, world");

    const output = IUnionTypeExample.visit(unionExample, {

        string: (value: string) => {
            // your logic here!
        },

        set: (value: string[]) => {},

        // ...

        unknown: (unknownType: IUnionTypeExample) => {}

    });
    ```

    Visitors may seem clunky in TypeScript, but they have the upside of compile-time assurance that you've handled all the possible variants.  If you upgrade an API dependency and the API author added a new variant, the TypeScript compiler will force you to explicitly deal with this new variant.  We intentionally avoid `switch` statements.

    We also generate [type-guards](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-guards-and-differentiating-types):

    ```typescript
    if (IUnionTypeExample.isString(unionTypeExample)) {
        const inner: string = unionTypeExample.string;
    }
    ```

- **Conjure enum: [EnumExample](./src/commands/generate/__tests__/resources/types/enumExample.ts)**

    conjure-typescript leverages TypeScript's [string Enums](https://www.typescriptlang.org/docs/handbook/enums.html#string-enums).

  ```typescript
  export enum EnumExample {
      ONE = "ONE",
      TWO = "TWO"
  }

  console.log(EnumExample.ONE); // prints "ONE"
  ```

- **Conjure alias**

  TypeScript uses structural (duck-typing) so aliases are currently elided.

## Example Client interfaces

Example service interface: [PrimitiveService](./src/commands/generate/__tests__/resources/services/primitiveService.ts)

```typescript
export interface IPrimitiveService {
    getPrimitive(): Promise<number>;
}

export class PrimitiveService {
    public getPrimitive(): Promise<number> {
        return this.bridge.callEndpoint<number>({
            endpointName: "getPrimitive",
            endpointPath: "/getPrimitive",
            method: "GET",
            requestMediaType: MediaType.APPLICATION_JSON,
            responseMediaType: MediaType.APPLICATION_JSON,
        });
    }
}
```

### Constructing clients

Use clients from [conjure-typescript-runtime](https://github.com/palantir/conjure-typescript-runtime) which configures the browser's
[Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) with sensible defaults:

```typescript
import { DefaultHttpApiBridge } from "conjure-client";
const recipes = new RecipeBookService(new DefaultHttpApiBridge({
    baseUrl: "https://some.base.url.com",
    userAgent: {
        productName: "yourProductName",
        productVersion: "1.0.0"
    }
}));

const results: Recipe[] = await recipes.getRecipes();
```

## Contributing

For instructions on how to set up your local development environment, check out the [Contributing document](./CONTRIBUTING.md).

## License
This project is made available under the [Apache 2.0 License](/LICENSE).
