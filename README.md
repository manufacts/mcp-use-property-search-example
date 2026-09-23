# HomeScout SF: property search MCP App

An `mcp-use` MCP App with a Zillow-style split view: San Francisco listing
cards on one side and a live map with price pins on the other.
The model opens the view once with `search-homes`, then refines it in place
through tools the view registers itself.

All listings are fictional. Map tiles come from Esri's keyless Canvas basemaps
(light or dark gray, following the host theme). No listing API, API key, or
paid service is involved.

## Live demo

[Open the chat demo](https://inspector.manufact.com/inspector?embedded=true&autoConnect=https%3A%2F%2Fmanufact-property-search-example.run.mcp-use.com%2Fmcp&embeddedConfig=%7B%22singleTab%22%3Atrue%2C%22defaultTab%22%3A%22chat%22%2C%22visibleTabs%22%3A%5B%22chat%22%5D%7D)
and ask for homes in San Francisco.

MCP endpoint: https://manufact-property-search-example.run.mcp-use.com/mcp

## Run

Requires Node.js 22.22.2 or later.

```sh
git clone https://github.com/manufacts/mcp-use-property-search-example.git
cd mcp-use-property-search-example
npm ci
npm run dev
```

Open the Inspector URL printed by the CLI, select **Chat**, and sign in with
Manufact or configure a supported model provider. Try these prompts in order:

1. "Show me homes in San Francisco."
2. "Now search the Mission, under $2M."
3. "Remove the two most expensive homes."
4. "Open the cheapest one on the map and save it."
5. "Zoom in one step."

The first prompt calls `search-homes` and opens the view. Every later prompt
calls a view tool, so the open map updates without rendering a second view.
Click **Fullscreen** in the view to switch display modes.

## How it works

`search-homes` returns the matching listing IDs plus the whole staged catalog in
`structuredContent`, so the view can filter any neighborhood locally. After it renders, the view registers tools
with `useViewTool` that filter, select, and move the map locally.

| Tool | Called by | Purpose |
| --- | --- | --- |
| `search-homes` | Model | Open the view with an initial search |
| `get-listing-details` | View (app-only) | Load extra facts when a card or pin is selected |
| `search-in-view` | Model, via the view | Change area, price, beds, baths, home type, or sort in place |
| `remove-listings` | Model, via the view | Hide homes from the cards and map |
| `select-listing` | Model, via the view | Fly to a home and open its detail card |
| `save-listings` | Model, via the view | Save or unsave homes |
| `fit-visible-results` | Model, via the view | Fit every visible home in frame |
| `zoom-map` | Model, via the view | Zoom in or out |
| `pan-map` | Model, via the view | Pan north, south, east, or west |

The view also reports its current area, filters, and sort to the model with
`ModelContext`, and sends follow-up messages with `useSendFollowUp`.

The catalog covers Pacific Heights, Marina, Russian Hill, Nob Hill, Hayes
Valley, SoMa, Mission District, Noe Valley, Potrero Hill, and Bernal Heights.

- `src/index.ts`: the staged catalog, `search-homes`, and `get-listing-details`.
- `views/property-search/view.tsx`: the view, its view tools, and model context.
- `views/property-search/map.tsx`: the Leaflet map, Esri tiles, pins, and camera controls.
- `views/property-search/cards.tsx`: result cards and the detail panel.

## Check and build

```sh
npm run typecheck
npm run build
```

## Deploy

Build with `npm ci && npm run build` and start with `npm start`. The MCP
endpoint is `/mcp`. No environment variables are required.

Source: [mcp-use property search example](https://github.com/mcp-use/mcp-use/tree/main/libraries/typescript/packages/server/examples/views/property-search).
