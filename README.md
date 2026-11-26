# CSSLoader-Mappings
Internal css mappings to translate css from older Steam clients to new ones


### Format:

```json
{
    "id": [
        "1",
        "2",
        "3"
    ]
}
```

- id: A randomly generated uuid. Currently serves no use.
- 1,2,3: The Steam classnames to map. All previous entries in the list will be mapped to the latest entry in the list.

### Updating translations (Webpack)

```js
// There are three localization classes, ranging from 1000-8000 keys of UI strings
const withoutLocalizationClasses = DFL.classMap.filter((module) => Object.keys(module).length < 1000)

const allClasses = withoutLocalizationClasses.map((module) => {
    let filteredModule = {}
    Object.entries(module).forEach(([propertyName, value]) => {
      // Filter out things that start with a number (eg: Breakpoints like 800px)
      // I have confirmed the new classes don't start with numbers
      if (isNaN(value.charAt(0))) {
        filteredModule[propertyName] = value;
      }
    })
    return filteredModule;
}).filter((module) => {
    // Some modules will be empty after the filtering, remove those
    return Object.keys(module).length > 0;
});

const mappings = allClasses.reduce((acc, cur) => {
    Object.entries(cur).forEach(([property, value]) => {
        if (acc[property]) {
            acc[property].push(value)
        } else {
            acc[property] = [value]   
        }
    })
    return acc
}, {})
```

+ webpack_mappings.py

### Inlined classes in build 1795567705

The Steam beta inlines some CSS class strings in its JavaScript. Those strings
are absent from the module exports used to capture the snapshot. The snapshot
retains three previous module IDs for manually verified classes so the
generator can follow their existing translation histories.

| Previous module ID | Evidence in the Steam UI bundles |
| --- | --- |
| `79614` | The library artwork component in `chunk~5ad53110ba.js` uses the same image class arguments and wrapper conditions as the earlier `sp.js` component. These identify Container, GreyBackground, Image, Visibility and Visible. |
| `84194` | The recent-games background rule in `chunk~5ad53110ba.css` retains the BigArtMode override, brightness filter and transition timings from `sp.css`. |
| `18402` | In `chunk~1fc66b8190.css`, the absolute-positioned focus root and its high z-index precede the ring with the four focus animations, matching the earlier rules in `sp.css`. |

Regenerate the class maps from the repository root in this order.

```sh
python3 webpack_id_mappings.py
python3 classname_mappings.py
python3 backport_classname_mappings.py
```
