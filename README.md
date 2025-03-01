![Quill Mention](docs/static/quill-mention.png "Quill Mention")

# Fork status

This package is a fork of the [quill-mention](https://www.npmjs.com/package/quill-mention) npm package. The original source 
code is available on GitHub at [afry/quill-mention](https://github.com/afry/quill-mention).
Version of this package is equal to version of original package, to make it clear which version of original package is used.

# Quill Mention

[![npm version](https://badge.fury.io/js/quill-mention.svg)](https://badge.fury.io/js/quill-mention)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Quill Mention is a module to provide @mentions or #hashtag functionality for the [Quill](https://quilljs.com/) rich text editor.

## Demo

https://quill-mention.com/

![Mention Demo GIF](docs/static/mention.gif "Mention Demo GIF")

## Getting Started

### Install

Install with npm:

```bash
npm install quill-mention --save
```

Install with [Yarn](https://yarnpkg.com/en/):

```bash
yarn add quill-mention
```

### Import package

```javascript
import "quill-mention/autoregister";
```

Importing quill-mention automatically adds it to Quill modules.

Now you only need to pass quill-mention config to quill.

Or you can manually import module and register it

```javascript
import Quill from "quill";
import {Mention, MentionBlot} from "quill-mention";

Quill.register({ "blots/mention": MentionBlot, "modules/mention": Mention });
```

### Example

```javascript
import "quill-mention/autoregister";

const atValues = [
  { id: 1, value: "Fredrik Sundqvist" },
  { id: 2, value: "Patrik Sjölin" }
];
const hashValues = [
  { id: 3, value: "Fredrik Sundqvist 2" },
  { id: 4, value: "Patrik Sjölin 2" }
];
const quill = new Quill("#editor", {
  modules: {
    mention: {
      allowedChars: /^[A-Za-z\sÅÄÖåäö]*$/,
      mentionDenotationChars: ["@", "#"],
      source: function(searchTerm, renderList, mentionChar) {
        let values;

        if (mentionChar === "@") {
          values = atValues;
        } else {
          values = hashValues;
        }

        if (searchTerm.length === 0) {
          renderList(values, searchTerm);
        } else {
          const matches = [];
          for (let i = 0; i < values.length; i++)
            if (
              ~values[i].value.toLowerCase().indexOf(searchTerm.toLowerCase())
            )
              matches.push(values[i]);
          renderList(matches, searchTerm);
        }
      }
    }
  }
});
```

### Hover and Click Example
```javascript
  window.addEventListener('mention-hovered', (event) => {console.log('hovered: ', event)}, false);
  window.addEventListener('mention-clicked', (event) => {console.log('hovered: ', event)}, false);
```

### Async example

```javascript
async function suggestPeople(searchTerm) {
  const allPeople = [
    {
      id: 1,
      value: "Fredrik Sundqvist"
    },
    {
      id: 2,
      value: "Patrik Sjölin"
    }
  ];
  return allPeople.filter(person => person.value.includes(searchTerm));
}

const quill = new Quill("#editor", {
  modules: {
    mention: {
      allowedChars: /^[A-Za-z\sÅÄÖåäö]*$/,
      mentionDenotationChars: ["@", "#"],
      source: async function(searchTerm, renderList) {
        const matchedPeople = await suggestPeople(searchTerm);
        renderList(matchedPeople);
      }
    }
  }
});
```

**Note**: if you whitelist quill formats via ["formats" option](https://quilljs.com/docs/configuration/#formats),
you need to add the mention format (default: "mention") there. Another way quill-mention won't work.
Here's an example with whitelisted formats:

```javascript
const quill = new Quill("#editor", {
  formats: ["bold", "italic", "mention"],
  // note "mention" format above
  modules: {
    mention: {
      allowedChars: /^[A-Za-z\sÅÄÖåäö]*$/,
      mentionDenotationChars: ["@", "#"],
      source: function(searchTerm, renderList, mentionChar) {
        // some source implementation
      }
    }
  }
});
```

### Settings

| Property                                      | Default                                                      | Description                                                  |
| --------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `source(searchTerm, renderList, mentionChar)` | `null`                                                       | Required callback function to handle the search term and connect it to a data source for matches. The data source can be a local source or an AJAX request. The callback should call `renderList(matches, searchTerm);` with matches of JSON Objects in an array to show the result for the user. The JSON Objects should have `id` and `value` but can also have other values to be used in `renderItem` for custom display. |
| `renderItem(item, searchTerm)`                | `function`                                                   | A function that gives you control over how matches from source are displayed. You can use this function to highlight the search term or change the design with custom HTML. This function will need to return either a string possibly containing unsanitized user content, or a class implementing the [Node](https://developer.mozilla.org/en-US/docs/Web/API/Node) interface which will be treated as a sanitized DOM node. |
| `allowedChars`                                | `[a-zA-Z0-9_]` (or `function`)                               | Allowed characters in search term triggering a search request using regular expressions.  Can be a function that takes the denotationChar and returns a regex. |
| `minChars`                                    | `0`                                                          | Minimum number of characters after the @ symbol triggering a search request |
| `maxChars`                                    | `31`                                                         | Maximum number of characters after the @ symbol triggering a search request |
| `offsetTop`                                   | `2`                                                          | Additional top offset of the mention container position      |
| `offsetLeft`                                  | `0`                                                          | Additional left offset of the mention container position     |
| `mentionDenotationChars`                      | `["@"]`                                                      | Specifies which characters will cause the mention autocomplete to open |
| `isolateCharacter`                            | `false`                                                      | Whether or not the denotation character(s) should be isolated. For example, to avoid mentioning in an email. |
| `allowInlineMentionChar`                      | `false`                                                      | Only works if `isolateCharacter` is set to `true`. Whether or not the denotation character(s) can appear inline of the mention text. For example, to allow mentioning an email with the @ symbol as the denotation character. |
| `fixMentionsToQuill`                          | `false`                                                      | When set to true, the mentions menu will be rendered above or below the quill container. Otherwise, the mentions menu will track the denotation character(s); |
| `showDenotationChar`                          | `true`                                                       | Whether to show the used denotation character in the mention item or not |
| `defaultMenuOrientation`                      | `'bottom'`                                                   | Options are `'bottom'` and `'top'`. Determines what the default orientation of the menu will be. Quill-mention will attempt to render the menu either above or below the editor. If `'top'` is provided as a value, and there is not enough space above the editor, the menu will be rendered below. Vice versa, if there is not enough space below the editor, and `'bottom'` is provided as a value (or no value is provided at all), the menu will be rendered above the editor. |
| `blotName`                                    | `'mention'`                                                  | The name of the [Quill Blot](https://github.com/quilljs/parchment#blots) to be used for inserted mentions. A default implementation is provided named 'mention', which may be overridden with a custom blot. |
| `dataAttributes`                              | `['id', 'value', 'denotationChar', 'link', 'target','disabled']` | A list of data values you wish to be passed from your list data to the html node. (`id, value, denotationChar, link, target` are included by default). |
| `onOpen`                                      | `function`                                                   | Callback when mention dropdown is open.                      |
| `onBeforeClose`                               | `function`                                                   | Callback before the DOM of mention dropdown is removed.      |
| `onClose`                                     | `function`                                                   | Callback when mention dropdown is closed.                    |
| `onSelect(item, insertItem)`                  | `function`                                                   | Callback for a selected item. When overriding this method, `insertItem` should be used to insert `item` to the editor. This makes async requests possible. |
| `linkTarget`                                  | `'_blank'`                                                   | Link target for mentions with a link                         |
| `listItemClass`                               | `'ql-mention-list-item'`                                     | Style class to be used for list items (may be null)          |
| `mentionContainerClass`                       | `'ql-mention-list-container'`                                | Style class to be used for the mention list container (may be null) |
| `mentionListClass`                            | `'ql-mention-list'`                                          | Style class to be used for the mention list (may be null)    |
| `spaceAfterInsert`                            | `true`                                                       | Whether or not insert 1 space after mention block in text    |
| `positioningStrategy`                         | `'normal'`                                                 | Options are `'normal'` and `'fixed'`. When `'fixed'`, the menu will be appended to the body and use fixed positioning. Use this if the menu is clipped by a parent element that's using `overflow:hidden|scroll`. |
| `renderLoading`                               | `function`                                                   | A function that returns the HTML for a loading message during async calls from `source`. The function will need to return either a string possibly containing unsanitized user content, or a class implementing the [Node](https://developer.mozilla.org/en-US/docs/Web/API/Node) interface which will be treated as a sanitized DOM node. The default functions returns `null` to prevent a loading message. |
| `selectKeys`                                  | `[13]`                                                       | An array of keyboard key codes that will trigger the select action for the mention dropdown. Default is ENTER key. See [this reference](http://gcctech.org/csc/javascript/javascript_keycodes.htm) for a list of numbers for each keyboard key. |

### Methods

You may retrieve the module from Quill like `quill.getModule('mention')` then call one of the imperative methods below.

| Method                                 | Example                                          | Description                                                 |
| -------------------------------------- | ------------------------------------------------ | ----------------------------------------------------------- |
| `insertItem(data, programmaticInsert, overriddenOptions = {})` | `insertItem({id:'123',value:'My Mention'},true, {blotName: "new_blot"})` | Inserts the given mention into the editor.  Also you can override `blotName`, `showDenotationChar` and `spaceAfterInsert` options using `overriddenOptions` param |
| `openMenu(denotationChar)`             | `openMenu('@')`                                  | Opens the mentions menu for the given denotation character. |

### Styling

To allow styling based on the menu orientation, a class is added depending on the orientation and the `mentionContainerClass` option. By default this will be `ql-mention-list-container-bottom` or `ql-mention-list-container-top`.

#### Mention Blot Styling

Mention blots can also be dynamically styled from data attributes, but requires
implement a custom Quill blot. For example:

```javascript
const MentionBlot = Quill.import("blots/mention");

class StyledMentionBlot extends MentionBlot {
  static render(data) {
    const element = document.createElement('span');
    element.innerText = data.value;
    element.style.color = data.color;
    return element;
  }
}
StyledMentionBlot.blotName = "styled-mention";

Quill.register(StyledMentionBlot);
```

Then, when creating your Quill instance, use the following settings:

```javascript
const quill = new Quill('#editor', {
  modules: {
    mention: {
      // ...
      dataAttributes: ['id', 'value', 'denotationChar', 'link', 'target', 'disabled', 'color'],
      blotName: 'styled-mention',
    }
  }
});
```

### Headers and Informational Items

Sometimes you may want to display a menu item that should not be selectable. These items may be group headers, hint text, or even a message saying there were no matching results. To show items like these, add `disabled:true` to items passed to `renderList` from your `source` method. Disabled items are shown but not selectable with the mouse or keyboard. If you need to style the disabled items differently, you will need to override the `renderItem` method.

### Upgrade Notes

As of 4.0.0, HTML embedded in `value` will no longer be rendered, due to potential XSS issues. Instead, to adjust rendering of list items, you can return a class implementing the [Node](https://developer.mozilla.org/en-US/docs/Web/API/Node) interface which will be treated as a sanitized DOM node, and in order to customize blot rendering, you will need to create a [custom mention blot](https://github.com/quill-mention/quill-mention#mention-blot-styling)

## Authors

**Fredrik Sundqvist** ([MadSpindel](https://github.com/MadSpindel))

See also the list of [contributors](https://github.com/quill-mention/quill-mention/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details
