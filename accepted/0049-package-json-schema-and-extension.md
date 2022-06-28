# Define an Extensible JSON Schema for `package.json`

## Summary

Define the following:

- a JSON Schema definition for the `package.json` fields that npm
  uses.
- a mechanism for dependencies to extend this schema with their
  own configuration object.
- tooling to validate the current `package.json` against the
  expectations of npm and the tools found in `node_modules`.

## Motivation

The `package.json` format has evolved over many years and has
reached a high degree of consistency and stability by this point.

Since this file can be found in nearly all JavaScript projects
(even those that do not use npm or node!), it is an attractive
place for development tools to look for configuration fields.
This is often preferrable to adding a `.blahrc` or `blahrc.yaml`
or `.blah.json` file, which clutters up the project root and
requires users to remember yet another thing.

The evolving pattern is to add something like this in
`package.json`:

```json
{
  "name": "my-project",
  "version": "1.2.3",
  "some-tool": {
    "port": 1234,
    "useTLS": true,
    "plugins": [
      "@some-tool/foobarbaz"
    ]
  }
}
```

However, apart from run-time checking by the tools themselves,
there is no straightforward way for users to validate that the
configuration that they've added to `package.json` is valid (even
to a cursory types level), and it is even possible for multiple
tools in `node_modules` to have conflicting expectations.

When the `npm/cli` project wishes to add a new field (most
recently, `"overrides"`), they have to do an extremely
time-consuming task of crawling the npm registry to ensure that
no other tools are using the field that they intend to take over.
Even this can often miss the use of dev tools, since the projects
using those fields may be private.

With a declarative structure in place for development tools to
define the schema of the package.json configuration that they
expect, it would be possible to provide edit-time assistance to
users, validation that none of the tools in `node_modules` expect
conflicting schemas in `package.json`, and npm itself could more
effectively search the registry for known fields before adding
new semantics to `package.json`.

Past attempts to define a `package.json` schema have often
stalled, in large part because they were not maintained by the
npm team themselves.  This RFC proposes a distributed means for
package authors (including the `npm/cli` maintainers) to declare
the schema that they expect in `package.json`.

## Detailed Explanation

Note: apart from the definition and ongoing maintenance of the
`package.json` schema as used by npm itself, nothing is required
of the npm project as far as implementation.  However, this could
of course set the stage for a future npm command to validate the
`package.json` schema, more comprehensive checks for a valid
`package.json` at publish or install time, and so on.

Package authors can declare the schema for the fields that they
care about in package.json by creating a **`FILENAMETBD.json`**
schema file.

The `npm/cli` project will create a schema file like this,
defining the fields that npm itself uses.  This will serve as the
"root" schema for editors and other tooling that validates
`package.json`.

The schema files of all dependencies of a project (either a root
project or workspace) are combined to form the total schema used
to validate package.json.

### Description and Example of the Package.json Schema File

### Method of Assembling Schemas

Each package would need to define its JSON Schema identifier URI
and the file where that's located. (It probably makes sense to have
a default file location, such as package.schema.json in the
distributions root.)

A single schema would then be automatically created which references
all of these schemas. It's then possible to use the spec defined
bundling process to bundle all the schemas into a single schema,
which will be supported by modern tooling.

### Proof of Concept Tooling

#### Validation
Aside from bundling, tooling will be required to run validation and
provide feedback in a digestable format. While ajv has long been
the JSON Schema tool of choice, it does not support the standardized
output format, which is advised for long term support and easing the
job for downstream tooling.

The only other JS based tool I've seen which supports current
versions of JSON Schema and the standardized output format is
Hyperjump JSON Schema Validator: https://github.com/hyperjump-io/json-schema-validator.

Hyperjump JSV was created by a member of the JSON Schema core team
who happens to be starting work on JSON Schema full time. This
helps as we can have some level of support on the implementation,
while the maintainer of ajv is looking to offload the work of
maintaining the project.

#### Bundling and JSON Schema version
While JSON Schema has been bundled by various tooling, it has
for the most part been done badly, without full schema context.
As these schemas will mostly be new, we can mandate a specific
version of JSON Schema (or minimum), which will cause the least
amount of friction.

JSON Schema 2020-12 formally defined the bundling process. The
latest two versions of JSON Schema also remove some of the
complexity that caused previous tooling to behave incorrectly
and in some cases cause problems.

It's also worth noting that VSCode recently started moving to
full support of JSON Schema 2019-09 and 2020-12. OpenAPI, a
popular API specification format, as of 3.1, fully supports
JSON Schema 2020-12.

#### JSON Schema 2020-12 extensibility
As package publishers will be producing JSON Schemas, our
expectation is that they will need to provide some additional
fields of importance, which will need to be validated.

Extending the JSON Schema meta-schema before 2019-09 was
considerably more complicated. In solving some of the
challenges around this issue, the OpenAPI (prior to 3.1)
subset superset JSON Schema parts JSON Schema was significantly
reduced.

I imagine the additions and requirements we will want to make
are likely to be minimal, and is something that warrants
further discussion later in this document, but the point here
is that using newer versions of JSON Schema is critical to
making the job of extending the meta-schema, easeir.

## Rationale and Alternatives

- **Ad-hoc field/value checking in npm and dev tools**

    This is the current status quo.  It is not ideal for the
    reasons listed in the Motivation section.  While simple, it
    presents ongoing maintenance issues which can cause problems
    for package authors and users.

- **Definition of package.json schema by an external party**

    For example: [SchmaStore](https://github.com/SchemaStore/schemastore/blob/master/src/schemas/json/package.json)

    This can work, but without active involvement and ongoing
    maintenance by the npm team, it is likely to be abandoned or
    forgotten when field semantics change in the future.

    Also, it does not provide a means for dev tool authors to
    declare the schemas that they use.

    Note: Currently, most code editors will use schemas defined
    in the SchemaStore by default.

- **Using package.json `config` field exclusively**

    The initial design of `package.json` designated the `config`
    field to be used for packages to define their own fields.

    However, the use and specification of this field was never
    adequately formalized or adopted.

    It would be better to pave the path that dev tools are using
    _now_, rather than try to convert them all to move their
    configuration into a new location.

<!--
other things here? Often this gets filled out during
discussion, so if there isn't much to say yet, that's fine.
-->

## Implementation

The package.json schema filename should be added to the list of
files that npm _always_ includes in published artifacts.

{{Give a high-level overview of implementation requirements and concerns. Be specific about areas of code that need to change, and what their potential effects are. Discuss which repositories and sub-components will be affected, and what its overall code effect might be.}}

{{THIS SECTION IS REQUIRED FOR RATIFICATION -- you can skip it if you don't know the technical details when first submitting the proposal, but it must be there before it's accepted}}

## Prior Art

{{This section is optional if there are no actual prior examples in other tools}}

{{Discuss existing examples of this change in other tools, and how they've addressed various concerns discussed above, and what the effect of those decisions has been}}

One notable use of JSON Schema is Microsofts VSCode.
VSCode uses the JSON Schema Store to automatically download schemas for validating files which match a specific fileame.
In addition,JSON Schema is used to validate some configuration (although I'm not sure which or how much).
The VSCode team have defined and make use of a [few additional keywords](https://github.com/microsoft/vscode/blob/a69f95fdf3dc27511517eef5ff62b21c7a418015/src/vs/base/common/jsonSchema.ts#L76-L90).

Considerations:
Do any of these additional keywords make senes to formalize as an extension to JSON Schema for autocomplete purposes? (It's unclear what their purpose is, although one can guess from the name.)
Can we see what other major code editors are doing in this space? It's assumed others may have made similar extensions.

VSCode provides "[Contribution Points](https://code.visualstudio.com/api/references/contribution-points)" for extension publishers to extend various functionalities within VSCode. This includes exposing the configuration of the extension.

The docs [provide details](https://code.visualstudio.com/api/references/contribution-points#contributes.configuration) on field definitions and how to use JSON Schema in this context. An example of VSCodes internal schema registry being used to register a configuration segment can be seen in thier [code on Github](https://github.com/microsoft/vscode/blob/4ae2e2ddfd008f0af026afb75b811b51a84e91c6/src/vs/platform/userDataSync/common/userDataSync.ts#L51).

Considerations: The configuration UI for VSCode is pretty good, including the extensions section.


## Unresolved Questions and Bikeshedding

{{Write about any arbitrary decisions that need to be made (syntax, colors, formatting, minor UX decisions), and any questions for the proposal that have not been answered.}}

Do we need to get / would it benefit from any code editors / IDEs involved in this process?
Should we seek more prior art? (I would guess other code editors do similar.)

{{THIS SECTION SHOULD BE REMOVED BEFORE RATIFICATION}}
