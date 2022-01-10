---
layout: default
title: "Comparison of code-generation tools"
date: 2022-01-10
categories: development microservices code-generation
tags: development microservices code-generation
---

## Comparison of code-generation tools

### OpenAPI Generator

#### OpenAPI Usage

Given, some [valid OpenAPI spec](https://raw.githubusercontent.com/openapitools/openapi-generator/master/modules/openapi-generator/src/test/resources/3_0/petstore.yaml) the following can be done.

generate server stub from open-api spec

```sh
docker run --rm -v %CD%\local:/local openapitools/openapi-generator-cli generate -i /local/petstore.yaml -g aspnetcore -o /local/out/aspnetcore
```

output mustache templates

```sh
docker run --rm -v %CD%\local:/local openapitools/openapi-generator-cli author template -g aspnetcore -o /local/template/aspnetcore
```

view list of mustache templates here:

1. [all templates](https://github.com/OpenAPITools/openapi-generator/tree/master/modules/openapi-generator/src/main/resources)
2. [aspnetcore](https://github.com/OpenAPITools/openapi-generator/tree/master/modules/openapi-generator/src/main/resources/aspnetcore)

create custom templates

```sh
docker run --rm -v %CD%\local:/local openapitools/openapi-generator-cli generate -i /local/petstore.yaml -g aspnetcore -o /local/out/aspnetcore -c /local/config.yaml
```

create custom generator

```sh
docker run --rm -v %CD%\local:/local openapitools/openapi-generator-cli meta -o /local/generators/my-aspnetcore -n my-aspnetcore -p com.my.aspnetcore
```

#### OpenAPI Findings

Pros:

- templating approach looks applicable to any required files, e.g. class, CI (dockerfile), config etc.

Cons:

- mustache files are hard to read, arbitrary in their layout and purpose.
- lots of "tool" specific lang to learn and that lang is not .NET although that's what it can output.

- can't seem to get arbitrary additional files generated in combination with the input data model from open-api spec. e.g., Validators file and types can be internal to "Controller" folder, or external, but then don't have API data model applied i.e. can't use enumerations of {{#operations}}

- templateTypes appear to be hard-coded to folder paths.

- even with a custom generator you appear to have very limited api to work with, e.g. you can only control the output folder for the "api" and "models", amongst other small things.

### Yeoman generator

#### Yeoman Usage

#### Yeoman Findings

### T4 templates

#### T4 Usage

#### T4 Findings

### Roslyn API

#### Roslyn Usage

#### Roslyn Findings

#### OpenAPI Docs

- [templating](https://openapi-generator.tech/docs/templating)
- [customization](https://openapi-generator.tech/docs/customization/)

#### Yeoman Docs

#### T4 Docs

#### Roslyn Docs

### Other References

[Home]( {{ site.url }})
