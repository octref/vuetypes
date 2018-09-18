# Vue Types

Version: 0.0.1. Currently collecting feedback.

This is a proposal for https://github.com/vuejs/vue/issues/7186. It tries to standardize a "Vue Type" format that [Vetur](https://github.com/vuejs/vetur) could read from and enhance the **HTML** editing experience in Vue SFC. **This proposal is not about type-checking in Vue SFC.**

Feedback welcome! You can join the [discussion](https://github.com/vuejs/vue/issues/7186) or open new issues here. Specifically, I would want to know:

- What more features would you want from the editor, based on the Vue Types data?
- Where do you think is the right place to author those data (Vue SFC as JSDocs, Vue SFC Custom Block, another file that you can import from Vue SFC, a centralized file for the whole library)?

## Motivation

TypeScript has a great ecosystem for typings, for example, run `npm i -D @types/node` and get auto completion & type check working in the editor. We want the same for using Vue libraries. For example, if I do `npm i -D @vuetypes/element-ui`, I should get all HTML tag and attribute auto completion & attribute value checking in the editor.

For that to happen, we need 3 things:
- A way to write those "vue types" in Vue components (SFC and JS/TS), and a tool to extract those typings into a specified format.
- A repo to store high-quality "vue types" and a NPM scope to download it from. Similar to `DefinitelyTyped/DefinitelyTyped` and `@types`, we could have `vuejs/VerilyTyped` and `@vuetypes`.
- Editor integration (Vetur / VLS) that reads the specified format and provides the desired features, including:
  - Auto completion
  - Error checking
  - Hover information
  - Jump to website doc

The rest of the document describes the "specified format".

## Vue Types Spec

### Components

Components should provide:
- A short description of the component.
- Its allowed attributes.
- Its required attributes.
- (?) Whether the component can only be a child of another component. For example, `el-checkbox-group` and `el-checkbox`.

### Attributes

Attributes should provide:
- A short description of the attribute.
- Its allowed values. If the values are enums, it should also provide short descriptions for those enum values.

### Sample JSON

```json
{
  "components": [
    {
      "name": "v-button",
      "description": "A button",
      "attributes": [
        {
          "name": "width",
          "description": "The width of the button",
          "type": "number",
          "required": true
        },
        {
          "name": "height",
          "description": "The height of the button",
          "type": "number",
          "required": true
        },
        {
          "name": "corner-style",
          "description": "The style of the button's corner",
          "type": "string",
          "enum": [
            { "value": "round", "description": "Round corners" },
            { "value": "sharp", "description": "Sharp corners" }
          ],
          "required": false,
          "default": "round"
        }
      ]
    }
  ]
}
```

### Expected Editor Behavior

#### Auto completion

- `<v|` should auto complete `<v-button|>`.
- `<v-button w|>` should auto complete `<v-button width="">`.
- `<v-button corner-style="|">` should auto complete `round` and `sharp`.

#### Diagnostics

- `<v-btn>` should show error, as `v-btn` is not a defined component.
- `<v-button width="">`, `<v-button width="20px">`, `<v-button width="foo">` should show error, as the provided values do not satisfy the `valueType` requirement.
- `<v-button width="50"></-button>` should show error, as required attribute `height` is not provided.
- `<v-button />` should show error, as required attributes `width` and `height` are not provided.
- `<v-button corner-style="square">` should show error, as `square` is not one of the enum values for `corner-style`.

#### Hover Information

- "A button" should show when hovering on `<v-button>`.
- "The style of the button's corner" should show when hover on `corner-style` in `<v-button corner-style="round">`.

#### (?) Jump to Website

If the website has a URL for each component / attribute, we could provide a command that allows user to jump to the website for a more comprehensive doc. This can be part of the hover / completion information, or just a special command.

I'm not sure if we want this. Although we could provide a `link` field in the spec for each component / attribute and easily achieve this in the editor.

### Distribution of the VueTypes JSON

After building a JSON following the spec, you have two options to distribute it:

- Put it in the NPM module you are publishing, and add a key `vueTypes` that points to the path of the file. Benefit is the JSON always has the right version.
- Publish it a `VerilyTyped`, which can automatically publish it to `vuetypes`. (I don't know if this is worth the hassle)
