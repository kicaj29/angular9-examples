- [Angular elements - custom elements](#angular-elements---custom-elements)
- [GraphQL (apollo graphql) for https://graphqlzero.almansi.me/api](#graphql-apollo-graphql-for-httpsgraphqlzeroalmansimeapi)
  - [Introduction](#introduction)
  - [Project initialization](#project-initialization)
    - [Add apollo client](#add-apollo-client)
    - [Add graphql-cli](#add-graphql-cli)
    - [Generate .graphqlconfig file](#generate-graphqlconfig-file)
    - [Command for copying graphql schema locally](#command-for-copying-graphql-schema-locally)
    - [Add GraphQL Codegen](#add-graphql-codegen)
    - [Initialize graphql-codegen](#initialize-graphql-codegen)
  - [Generate TypeScript apollo service based on local schema](#generate-typescript-apollo-service-based-on-local-schema)
  - [Use generated angular apollo service](#use-generated-angular-apollo-service)
  - [Add types.graphql-gen.ts to ```.gitignore``` file.](#add-typesgraphql-gents-to-gitignore-file)
  - [Automation for generating apollo services *.graphql-gen.ts](#automation-for-generating-apollo-services-graphql-gents)
  - [Storing apollo services in dedicated file next to ```*.graphql file```.](#storing-apollo-services-in-dedicated-file-next-to-graphql-file)


# Angular elements - custom elements
https://blog.bitsrc.io/using-angular-elements-why-and-how-part-1-35f7fd4f0457   
https://www.youtube.com/watch?v=Z1gLFPLVJjY&t=4s   
https://www.techiediaries.com/angular/angular-9-web-components-custom-elements-shadow-dom   
https://www.techiediaries.com/angular/angular-9-elements-web-components   

[hello-angular-element](./src/app/hello-angular-element) contains example of angular element.

* it can be used as normal angular component like [here](./src/app/app.component.html#L2)
* it can be used as custom element in the same angular app like [here](./src/app/app.component.ts#L12)
* it can be used in another application - **not an angular application** like [here](./preview/index.html). To generate ```angularapp.js``` make sure that ```main.ts``` points to ```HelloAngularElementModule``` like this
  ```
  platformBrowserDynamic().bootstrapModule(HelloAngularElementModule)
  .catch(err => console.error(err));
  ```
  and next run ```npm run build:component```.

  >Shadow DOM: CSS defined inside shadow DOM is scoped to it. Style rules don't leak out and page styles don't bleed in. More [here](https://developers.google.com/web/fundamentals/web-components/shadowdom).

  >Shadow DOM: selectors and styles inside of a shadow DOM node don’t leak outside of the shadow root and styles from outside the shadow root don’t leak in. There are a few exceptions that inherit from the parent document, like font family and document font sizes (e.g. rem) that can be overridden internally. More [here](https://css-tricks.com/encapsulating-style-and-structure-with-shadow-dom/#what-is-the-shadow-dom).

CSS selectors in Light DOM (normal DOM) are prevented from reaching elements inside shadow DOM but there are exceptions when it can happen.
But when a CSS property has the value inherit, **the shadow DOM will still inherit those from the light DOM**. You can **prevent** the values of properties form being inherited by using the initial value.
It can be set for all elements:
```css
* {
  all:initial;
}
```
or selected properties:
```css
* {
  color: initial;
}
```

https://stackoverflow.com/questions/49709676/light-dom-style-leaking-into-shadow-dom   
https://stackoverflow.com/questions/47189985/shadow-dom-inheriting-parent-page-css-chrome   


# GraphQL (apollo graphql) for https://graphqlzero.almansi.me/api

## Introduction
In the following chapters are explained steps that have to be executed to start using Angular Apollo for GraphQL.   
Apollo services are not stored in the repo so after cloning to run the app you have to generate them using ```npm start gql:codegen```.

## Project initialization

### Add [apollo client](https://www.apollographql.com/docs/angular/basics/setup/)   
  ```
  ng add apollo-angular@1.9.1
  ```
  It will install the following packages:
  ```
  "apollo-angular": "^1.9.1",
  "apollo-angular-link-http": "^1.10.0",
  "apollo-link": "^1.2.11",
  "apollo-client": "^2.6.0",
  "apollo-cache-inmemory": "^1.6.0",
  "graphql-tag": "^2.10.0",
  "graphql": "^14.6.0"
  ```
  >NOTE: I used version **1.9.1** because the newest available version (2.0.3) had some issues with copying graphql schema.

### Add [graphql-cli](https://github.com/Urigo/graphql-cli)
  ```
  npm install graphql-cli@3.0.14 -D
  ```
### Generate [.graphqlconfig](./.graphqlconfig) file
  ```
  PS D:\GitHub\kicaj29\angular9-examples> npx graphql init
  ? Enter project name (Enter to skip):
  ? Local schema file path: schema.graphql
  ? Endpoint URL (Enter to skip): https://graphqlzero.almansi.me/api
  ? Name of this endpoint, for e.g. default, dev, prod: default
  ? Subscription URL (Enter to skip):
  ? Do you want to add other endpoints? No
  ? What format do you want to save your config in? JSON

  About to write to D:\GitHub\kicaj29\angular9-examples\.graphqlconfig:

  {
    "schemaPath": "schema.graphql",
    "extensions": {
      "endpoints": {
        "default": "https://graphqlzero.almansi.me/api"
      }
    }
  }

  ? Is this ok? Yes
  PS D:\GitHub\kicaj29\angular9-examples>
  ```
### Command for copying graphql schema locally
  ```
  "gql:update-schema": "graphql get-schema -e default"
  ```
  It will create new file with the schema [schema.graphql](schema.graphql).
  There are 2 reasons why it makes sense copy schema locally:   
  * CI pipeline does not depend on external server resource to auto generate the schema
  * if the schema changes during code review we can easily see the changes
### Add [GraphQL Codegen](https://graphql-code-generator.com)   
  Plugin [Type Script Apollo Angular](https://graphql-code-generator.com/docs/plugins/typescript-apollo-angular) allows generate code for apollo services which next can be used in DI mechanism.
  ```
  npm install @graphql-codegen/cli@1.13.1 -D
  ```
### Initialize graphql-codegen
  ```
  PS D:\GitHub\kicaj29\angular9-examples> npx graphql-codegen init

    Welcome to GraphQL Code Generator!
    Answer few questions and we will setup everything for you.

  ? What type of application are you building? Application built with Angular
  ? Where is your schema?: (path or url) schema.graphql
  ? Where are your operations and fragments?: src/**/*.graphql
  ? Pick plugins: TypeScript (required by other typescript plugins), TypeScript Operations (operations and fragments), TypeScript Apollo Angular (typed GQL services)
  ? Where to write the output: src/generated/types.graphql-gen.ts
  ? Do you want to generate an introspection file? No
  ? How to name the config file? codegen.yml
  ? What script in package.json should run the codegen? gql:codegen

    Config file generated at codegen.yml

      $ npm install

    To install the plugins.

      $ npm run gql:codegen

    To run GraphQL Code Generator.
  ```
  It will create a new command ```gql:codegen``` in ```package.json``` and will add new devDependencies
  ```
  "@graphql-codegen/typescript": "1.13.1",
  "@graphql-codegen/typescript-operations": "1.13.1",
  "@graphql-codegen/typescript-apollo-angular": "1.13.1"
  ```
  Next ```npm install``` has to be run to install these 3 new packages.

## Generate TypeScript apollo service based on local schema

* Create sample graphql with a query to later generate for this apollo service [graphql.component.graphql](./src/app/graphql/graphql.component.graphql).
  
  ```
  query Users {
    users {
      data {
        name
      }
    }
  }
  ```
  >NOTE: Plugin [Type Script Apollo Angular](https://graphql-code-generator.com/docs/plugins/typescript-apollo-angular) requires a little bit different syntax in *.graphql files then in https://graphqlzero.almansi.me/api. For example in the file we have to specify type of operation - here it is **query** but in the playground view the query looks like this:
  ```
  {
    users {
      data {
        name
      }
    }
  }
  ```

* Generate apollo service.   
  Run: 
  ```
  npm run gql:codegen
  ```
  It will create a new file ```types.graphql-gen.ts```.
  >NOTE: because this file is generated from the schema and from ```*.graphql``` files it is not stored in the repo.
  
  On the bottom of this file is generated angular apollo service:
  ```ts
  @Injectable({
    providedIn: 'root'
  })
  export class UsersGQL extends Apollo.Query<UsersQuery,         UsersQueryVariables> {
    document = UsersDocument;    
  }
  ```
## Use generated angular apollo service

In [graphql.component.ts](./src/app/graphql/graphql.component.ts) inject generated service and render its data.   

Run the app to see that it is working fine.

## Add types.graphql-gen.ts to ```.gitignore``` file.

It is good practice to keep all generated type script files ```*.graphql-gen.ts``` in ```.gitignore```. Then typical workflow for CI works like this:
* developer on demand updates [schema.graphql](./schema.graphql). 
* developer runs ```npm run gql:codegen``` to generate new angular apollo services based on the new schema. If everything on local branch works fine then updated schema can be pushed to the remote branch.
* CI once again generate all ```*.graphql-gen.ts``` (because they do not exist in the repo) and in this way can make sure that everything compiles and all possible tests still pass with the new updated schema.

## Automation for generating apollo services *.graphql-gen.ts

If needed angular apollo services can be generated automatically when new ```*.graphql``` file is created or existing file is updated or deleted.
The following commands make it possible:

```
npm install npm-run-all -D
```
Version 4.1.5 has been installed.

Next in ```package.json``` new scripts can be added:

```
"start:ng": "ng serve",
"gql:codegen:watch": "graphql-codegen --watch"
"start": "run-p start:ng gql:codegen:watch"
```

## Storing apollo services in dedicated file next to ```*.graphql file```.

```
npm install @graphql-codegen/near-operation-file-preset@1.13.1 -D
```

Next update codegen.yml:

```yaml
overwrite: true
schema: "schema.graphql"
documents: "src/**/*.graphql"
generates:
  src/generated/types.graphql-gen.ts:
    plugins:
    - "typescript"
  src/generated:
    preset: near-operation-file
    presetConfig:
      extension: .graphql-gen.ts
      baseTypesPath: types.graphql-gen.ts
    plugins:
      - "typescript"
      - "typescript-operations"
      - "typescript-apollo-angular"
```

Next run again ```npm run gql:codegen``` to see that now angular apollo services are stored in separated files.

For example now [graphql.component.graphql](./src/app/graphql/graphql.component.graphql) and [graphql.component.graphql-gen.ts](./src/app/graphql/graphql.component.graphql-gen.ts) are next to each other and the service is generated on the bottom of [graphql.component.graphql-gen.ts](./src/app/graphql/graphql.component.graphql-gen.ts).
