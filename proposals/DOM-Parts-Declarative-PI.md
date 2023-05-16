# DOM Parts Declarative Processing Instruction API

This proposal covers the declarative API for creating a DOM part with processing
instructions.

## Proposal

Processing instructions provide a directive to the HTML parser to construct DOM
parts for specific nodes and ranges of nodes.

In the most basic level, this proposal consists of two processing instructions:

1. `<?node-part?>` creates a `NodePart` attached to the immediately following
   `Node`.
1. `<?child-node-part?><?/child-node-part>` which creates a `ChildNodePart`
   attached to the parent `Node` and contains any nodes in between the
   processing instructions.

### Basic example

Suppose we had the following template in some HTML extension template languages
where `{name}` and `{email}` indicated locations of dynamic data insertion:

```html
<section>
  <h1 id="name">{name}</h1>
  Email: <a id="link" href="mailto:{email}">{email}</a>
</section>
```

And the application has produced the following HTML with the static content:

```html
<section>
  <h1 id="name"><?child-node-part?><?/child-node-part?></h1>
  Email:
  <?node-part metadata?><a id="link"></a>
</section>
```

This will create a `ChildNodePart` attached to `<h1>` with no content and a
`NodePart` attached to `<a>`.

A framework could fetch these parts use either a
[`DocumentPart`](./DOM-Parts-Imperative.md#option-2-documentpartgroup) or a
[`ChildNodePart`](./DOM-Parts-Imperative.md#option-3-childnodepart-is-a-partgroup),
depending on what direction the imperative API goes to solve DOM part grouping.

## `ChildNodePart` Previous and Next Siblings

One question is how `ChildNodePart` is constructed from the processing
instruction. The [imperative API](./DOM-Parts-Imperative.md#proposal) allows
passing a `previousSibling` and `nextSibling` parameter to the constructor that
is useful in making the `ChildNodePart` smaller than the full range of children.

In Chrome, processing instructions are rendred as comments in the HTML after
parsing. These comment nodes could be used as the previous and next siblings for
the `ChildNodePart`, though this would mean there's no way to declaratively
represent a `ChildNodePart` without a previous and next sibling, for example
`new ChildNodePart(element)`.

The same questions about what to do if the siblings are mutated arise as they do
with the
[imperative API](./DOM-Parts-Imperative.md#childnodepart-after-dom-mutations).

## Permissive parsing of `ChildNodePart`

Processing instructions representing `ChildNodePart` should be permissive and
discard processing instructions that are invalid because they are imbalanced
underneath a single parent.

## Metadata

Templating systems may need to serialize data about the nodes they are marking
into the processing instructions. To support this, the rest of the processing
instruction node could be parsed as arbitrary metadata, emulating a
`HTMLElement` attribute.

```html
<?child-node-part? name="email"?><?/child-node-part>
```

This could be exposed on the imperative API to be consumed in JavaScript by
application logic.
