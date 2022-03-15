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

customize open-api template. below is an example of the `controller.mustache` template

```c#
{{>partial_header}}
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
{{#operationResultTask}}
using System.Threading.Tasks;
{{/operationResultTask}}
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Http;
{{#useSwashbuckle}}
using Swashbuckle.AspNetCore.Annotations;
using Swashbuckle.AspNetCore.SwaggerGen;
{{/useSwashbuckle}}
{{^isLibrary}}
using Newtonsoft.Json;
{{/isLibrary}}
using {{packageName}}.Attributes;
using {{modelPackage}};

namespace {{apiPackage}}
{ {{#operations}}
    /// <summary>
    /// {{description}}
    /// </summary>{{#description}}
    [Description("{{.}}")]{{/description}}
    [ApiController]
    public {{#classModifier}}{{.}} {{/classModifier}}class {{classname}}Controller : ControllerBase
    { {{#operation}}
        /// <summary>
        /// {{summary}}
        /// </summary>{{#notes}}
        /// <remarks>{{.}}</remarks>{{/notes}}{{#allParams}}
        /// <param name="{{paramName}}">{{description}}{{#isDeprecated}} (deprecated){{/isDeprecated}}</param>{{/allParams}}{{#responses}}
        /// <response code="{{code}}">{{message}}</response>{{/responses}}
        [{{httpMethod}}]
        [Route("{{{basePathWithoutHost}}}{{{path}}}")]
{{#authMethods}}
{{#isApiKey}}
        [Authorize(Policy = "{{name}}")]
{{/isApiKey}}
{{#isBasicBearer}}
        [Authorize{{#scopes}}{{#-first}}(Roles = "{{/-first}}{{scope}}{{^-last}},{{/-last}}{{#-last}}"){{/-last}}{{/scopes}}]
{{/isBasicBearer}}
{{/authMethods}}
        {{#vendorExtensions.x-aspnetcore-consumes}}
        [Consumes({{&vendorExtensions.x-aspnetcore-consumes}})]
        {{/vendorExtensions.x-aspnetcore-consumes}}
        [ValidateModelState]{{#useSwashbuckle}}
        [SwaggerOperation("{{operationId}}")]{{#responses}}{{#dataType}}
        [SwaggerResponse(statusCode: {{code}}, type: typeof({{&dataType}}), description: "{{message}}")]{{/dataType}}{{/responses}}{{/useSwashbuckle}}{{^useSwashbuckle}}{{#responses}}{{#dataType}}
        [ProducesResponseType(statusCode: {{code}}, type: typeof({{&dataType}}))]{{/dataType}}{{/responses}}{{/useSwashbuckle}}
        {{#isDeprecated}}
        [Obsolete]
        {{/isDeprecated}}
        public {{operationModifier}} {{#operationResultTask}}{{#operationIsAsync}}async {{/operationIsAsync}}Task<{{/operationResultTask}}IActionResult{{#operationResultTask}}>{{/operationResultTask}} {{operationId}}({{#allParams}}{{>pathParam}}{{>queryParam}}{{>bodyParam}}{{>formParam}}{{>headerParam}}{{^-last}}{{^isCookieParam}}, {{/isCookieParam}}{{/-last}}{{/allParams}}){{^generateBody}};{{/generateBody}}
        {{#generateBody}}
        {
            {{#cookieParams}}
            var {{paramName}} = Request.Cookies["{{paramName}}"];
            {{/cookieParams}}

{{#responses}}
{{#dataType}}
            //TODO: Uncomment the next line to return response {{code}} or use other options such as return this.NotFound(), return this.BadRequest(..), ...
            // return StatusCode({{code}}, default({{&dataType}}));
{{/dataType}}
{{^dataType}}
            //TODO: Uncomment the next line to return response {{code}} or use other options such as return this.NotFound(), return this.BadRequest(..), ...
            // return StatusCode({{code}});
{{/dataType}}
{{/responses}}
{{#returnType}}
            string exampleJson = null;
            {{#examples}}
            exampleJson = "{{{example}}}";
            {{/examples}}
            {{#isListCollection}}{{>listReturn}}{{/isListCollection}}{{^isListCollection}}{{#isMap}}{{>mapReturn}}{{/isMap}}{{^isMap}}{{>objectReturn}}{{/isMap}}{{/isListCollection}}
            {{!TODO: defaultResponse, examples, auth, consumes, produces, nickname, externalDocs, imports, security}}
            //TODO: Change the data returned
            return {{#operationResultTask}}Task.FromResult<IActionResult>({{/operationResultTask}}new ObjectResult(example){{#operationResultTask}}){{/operationResultTask}};{{/returnType}}{{^returnType}}
            throw new NotImplementedException();{{/returnType}}
        }
        {{/generateBody}}
        {{/operation}}
    }
{{/operations}}
}
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

1. install node with [chocolatey](https://chocolatey.org/install)
2. install yo:

```sh
npm install -g yo
yo --version
```

```sh
# install a generator
npm install -g <generator-name>

# e.g. 
npm install -g generator-webapp

# scaffold new project using installed generator
yo <generator-name>

# e.g. 
yo webapp

# use sub-generators, e.g. add a controller
yo <generator-name>:<sub-generator-name>

# troubleshoot & debug
yo doctor
yo --generators
yo --help
yo --version
```

```sh
# create folder to hold generator, name is NB: generator-name, e.g. generator-sample
cd generator-name

# create package.json in folder
echo '' > package.json or npm init
npm install --save yeoman-generator

# create folders
# ├───package.json
# └───generators/
#     ├───app/
#     │   └───index.js
#     └───router/
#         └───index.js

npm link

yo <generator-name> # e.g. yo sample [doesn't need 'generator' prefixing it]
```

The available priorities are (in running order):

initializing - Your initialization methods (checking current project state, getting configs, etc)
prompting - Where you prompt users for options (where you’d call this.prompt())
configuring - Saving configurations and configure the project (creating .editorconfig files and other metadata files)
default - If the method name doesn’t match a priority, it will be pushed to this group.
writing - Where you write the generator specific files (routes, controllers, etc)
conflicts - Where conflicts are handled (used internally)
install - Where installations are run (npm, bower)
end - Called last, cleanup, say good bye, etc

#### Yeoman Sample

```c#
namespace <%= namespace %>.Controllers
{
    [Route("api")]
    [ApiController]
    public class AccountController : ControllerBase
    {

        <%_ if (authenticationType === 'jwt') { _%>
        private readonly ILogger<AccountController> _log;
        <%_ if (cqrsEnabled) { _%>
        private readonly IMediator _mediator;
        <%_ } else { _%>
        private readonly IMapper _userMapper;
        private readonly IMailService _mailService;
        private readonly UserManager<User> _userManager;
        private readonly IUserService _userService;
        <%_ } _%>

        <%_ if (cqrsEnabled) { _%>
        public AccountController(ILogger<AccountController> log, IMediator mediator)
        {
            _log = log;
            _mediator = mediator;
        }
        <%_ } else { _%>
        public AccountController(ILogger<AccountController> log, UserManager<User> userManager, IUserService userService,
            IMapper userMapper, IMailService mailService)
        {
            _log = log;
            _userMapper = userMapper;
            _userManager = userManager;
            _userService = userService;
            _mailService = mailService;
        }
        <%_ } _%>

        [HttpPost("register")]
        [ValidateModel]
        public async Task<IActionResult> RegisterAccount([FromBody] ManagedUserDto managedUserDto)
        {
            if (!CheckPasswordLength(managedUserDto.Password)) throw new InvalidPasswordException();
            <%_ if (cqrsEnabled) { _%>
            var user = await _mediator.Send(new AccountCreateCommand { ManagedUserDto = managedUserDto });
            <%_ } else { _%>
            var user = await _userService.RegisterUser(_userMapper.Map<User>(managedUserDto), managedUserDto.Password);
            await _mailService.SendActivationEmail(user);
            <%_ } _%>
            return CreatedAtAction(nameof(GetAccount), user);
        }

        /// etc...
        }
    }    
}    
```

#### Yeoman Findings

Pros:

- well known tool, widely used in JS world
- API looks quite flexible and capable of generating full-on real-world APIs end-to-end e.g.:
- [JHipster-DotNetCore](https://github.com/jhipster/jhipster-dotnetcore/blob/main/generators/server/templates/dotnetcore/src/Project/Controllers/AccountController.cs.ejs)
- [WebApi sample](https://github.com/sbarski/generator-webapi/blob/master/app/templates/src/web/controllers/_valuescontroller.cs)
- The tool is capable of being run after initial generation.
- Applicable to a wide number of languages as it isn't language specific, just a templating engine.

Cons:

- It's in JS, and I'm more focused on .NET so the learning curve is greater.
- It is still "just" templating., i.e. very similar to T4 etc.
- Unlike OpenAPI generator, it's "just" a generator, it's not specifically going to generate OpenAPI spec, that would need to be written / combined.

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

- [yeoman](https://yeoman.io/authoring/)

#### T4 Docs

#### Roslyn Docs

### Other References

- ...

[Home]( {{ site.url }})
