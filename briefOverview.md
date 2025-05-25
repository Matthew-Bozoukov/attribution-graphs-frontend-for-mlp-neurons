## index.html:
Everything in the site is rendered through [D3](https://d3js.org/), a tool to create graphics and SVGs with raw javascript. The main logic runs through index.html and the rest of the files are callled through it, so just focus on the HTML file if you want a basic understanding of whats called. Almost all of the imports in the HTML file are just for styles or example data and the only set of things you need to worry about are the library imports and the file imports. 
```html
<!-- Library Imports (with the libraries being held on their own site for some reason) -->
<script src='https://transformer-circuits.pub/2025/attribution-graphs/static_js/lib/hotserver-client-ws.js'></script>
<script src='https://transformer-circuits.pub/2025/attribution-graphs/static_js/lib/d3.js'></script>
<script src='https://transformer-circuits.pub/2025/attribution-graphs/static_js/lib/jetpack_2024-07-20.js'></script>
<script src='https://transformer-circuits.pub/2025/attribution-graphs/static_js/lib/npy_v0.js'></script>

<!-- File Imports -->
<script src='./util.js'></script>
<script src='./attribution_graph/util-cg.js'></script>
<script src='./attribution_graph/gridsnap/init-gridsnap.js'></script>
<script src='./attribution_graph/init-cg-button-container.js'></script>
<script src='./attribution_graph/init-cg-link-graph.js'></script>
<script src='./attribution_graph/init-cg-node-connections.js'></script>
<script src='./attribution_graph/init-cg-clerp-list.js'></script>
<script src='./attribution_graph/init-cg-feature-detail.js'></script>
<script src='./attribution_graph/init-cg-feature-scatter.js'></script>
<script src='./attribution_graph/init-cg-subgraph.js'></script>
<script src='./attribution_graph/init-cg.js'></script>
```
Also of special note is are the div tags. Just look at them really quick but most the logic isn't held within them anyways, just knowing they exist is good enough.

The main script within the HTML file acts only to connect this information into one function and a corresponding D3 object. The very first line is just creating that window function and the rest of it is a really deeply nested initialization. First some simple graph and site data are set to make it interactable.
```javascript
// getting the graph data for what to visualize from some file
var {graphs} = await util.getFile('/data/graph-metadata.json')

// setting an object to hold the site data for interaction
window.visState = window.visState || {
    slug: util.params.get('slug') || graphs[0].slug,
    clickedId: util.params.get('clickedId')?.replace('null', ''),
    isGridsnap: util.params.get('isGridsnap')?.replace  ('null', ''),
}
```
Then the dropdown selection menu is configured in the div tag with the `.nav` class. 
```javascript
// selects the div tag with that class, clears its HTML, applies a custom selection element to the tag, then applies a custom function onto it.
// The function checks when the dropdown is changed, then updates the site data that was initialized above
var selectSel = d3.select('.nav').html('').append('select.graph-prompt-select')
    .on('change', function() {
        visState.slug = this.value
        visState.clickedId = undefined
        util.params.set('slug', this.value)
        render()
    })

// The options for the selection dropdown are then added into it, each with a text, attr, and property property.
selectSel.appendMany('option', graphs)
    .text(d => {
        var scanName = util.nameToPrettyPrint[d.scan] || d.scan
        var prefix = d.title_prefix ? d.title_prefix + ' ' : ''
        return prefix + scanName + ' — ' + d.prompt
    })
    .attr('value', d => d.slug)
    .property('selected', d => d.slug === visState.slug)
```
The last function creates the graph itself in some `render()` function before calling the function itself and initializing the window.
```javascript
function render() {
    // clearing the HTML in the tag
    d3.select('.cg').html('')
    // custom initCg function to edit the information with the .cg tag using all the site data initialized before.
    initCg(d3.select('.cg'), visState.slug, {
        clickedId: visState.clickedId,
        clickedIdCb: id => util.params.set('clickedId', id),
        isGridsnap: visState.isGridsnap || true
    })

    // searches through the graphs from the json with ones that match the site's slug
    var m = graphs.find(g => g.slug == visState.slug)
    // if none found, don't render
    if (!m) return
    // informational updates
    selectSel.at({title: m.prompt})
    document.title = 'Attribution Graph: ' + m.prompt
}
```

## initCg:
The initCg (probably standing for init color graph or some other shit) function is also a key part of the pipeline, being held in `init-cg.js`, and is used practically everywhere that a graph is being rendered. It is mad long so I can't do the same part by part dissection, but I can make a really simple skeleton version.
```javascript
var data = // get the json of the specific graph requested
var visState = // a lot of state variables

precautionaryChecksForVariables()
util.Cg.formatData(data, visState) // another custom function
var renderAll = util.initRenderAll(params) // another custom function for customizable rendering and event calls
colorNodes() // created function to set each colors node to a default white
colorLinks() // created function to color and size the connection between nodes
renderAll.funcUpdates() // dynamically pushing any necessary data to the renderAll event functions
connectedLinks.forEach(/* set some link info based on the link's data*/)
data.nodes.forEach(/* setting node links */)
data.features.forEach(/* setting feature links */)
initGridSnap() // inits both some gridsnap data and a selection div for the option

renderAll.callAll() // calls basically every single renderAll function
```
`initRenderAll()` is contained within util.js and just creates an object that contains a bunch of event labels, with the strings given to the function defining those event labels. These each then trigger a function (which is defined within the `renderAll.funcUpdates()` section of the simplification) when that specific event tied to the visState of the site occurs.

The corresponding rendering code is defined within util.js under a bunch of sub functions but a simple example lies within ggPlot().

# Other Files:
