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

### Proof of Concept Tooling

{{Describe the expected changes in detail, }}

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

## Unresolved Questions and Bikeshedding

{{Write about any arbitrary decisions that need to be made (syntax, colors, formatting, minor UX decisions), and any questions for the proposal that have not been answered.}}

{{THIS SECTION SHOULD BE REMOVED BEFORE RATIFICATION}}
