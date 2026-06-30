<p align="center" draggable="false"><img src="https://github.com/AI-Maker-Space/LLM-Dev-101/assets/37101144/d1343317-fa2f-41e1-8af1-1dbb18399719"
     width="200px"
     height="auto"/>
</p>

<h1 align="center" id="heading">Session 8: Model Context Protocol (MCP)</h1>

### [Quicklinks]()

| Session Sheet | Recording | Slides | Repo | Homework | Feedback |
|:--------------|:----------|:-------|:-----|:---------|:---------|
| [Session 8: MCP](https://github.com/AI-Maker-Space/The-AI-Engineering-Certification-v1.0/tree/main/00_Docs/Modules/08_MCP) |[Recording!](https://us02web.zoom.us/rec/share/rqw5I5hwbOOHy8TrGjnu0IjDJi53ykHb0k897jYfyHqZpgRhUuFP4A18d4NrcEKS.18sNk6Do9XwyaVUy) <br> passcode: `E56&^V+8`| [Session 8 Slides](https://canva.link/k8cixqgkfeghdsn) |You are here! | [Session 8 Assignment](https://forms.gle/TcjjChq38ydMjuqn8) | [Feedback 6/25](https://forms.gle/DvcWDgBXatBWCXqi7) |

## Useful Resources

**MCP (Model Context Protocol)**
- [MCP Official Docs](https://modelcontextprotocol.io/) — Spec, tutorials, and guides
- [MCP-UI](https://mcpui.dev/) — Official standard for interactive UI in MCP
- [MCP Auth Guide (Auth0)](https://auth0.com/blog/mcp-specs-update-all-about-auth/) — Deep dive into MCP auth spec updates

## Main Assignment

In this session, you will build an MCP server with OAuth authentication — a cat
shop application that exposes tools for browsing products, managing a cart, and
checking out.

The main entry point is:

```text
server.py
```

The server implementation lives in:

```text
app/
```

Available MCP tools:

- `list_products`
- `get_product`
- `add_to_cart`
- `view_cart`
- `remove_from_cart`
- `checkout`

## Setup

From this folder:

```bash
uv sync
```

Copy the example env file and fill in your OpenAI API key:

```bash
cp .env.example .env
```

## Running the MCP Server

Run the server locally:

```bash
uv run server.py
```

The server starts on `http://localhost:8000`.

### Expose the server with ngrok

In a separate terminal, start an ngrok tunnel:

```bash
ngrok http 8000
```

Copy the ngrok forwarding URL (e.g. `https://xxxx-xx-xx-xx-xx.ngrok-free.app`) and
restart the server with it:

```bash
ISSUER_URL=https://xxxx-xx-xx-xx-xx.ngrok-free.app uv run server.py
```

> **Note:** The `ISSUER_URL` must match the public URL clients use to reach the
> server, otherwise OAuth authentication will fail.

## Outline

### Breakout Room #1

- Set up the MCP server with OAuth and the product database
- Explore the MCP tools: `list_products`, `get_product`, `add_to_cart`, `view_cart`, `remove_from_cart`, `checkout`

### Breakout Room #2

- Connect an MCP client to the server
- Build an end-to-end interaction flow using the MCP tools

## Ship

The completed MCP server and client integration!

### Deliverables

- A short Loom of either:
  - the MCP server you built and a demo of the client interacting with it; or
  - the notebook you created for the Advanced Build

## Share

Make a social media post about your final application!

### Deliverables

- Make a post on any social media platform about what you built!

Here's a template to get you started:

```
🚀 Exciting News! 🚀

I am thrilled to announce that I have just built and shipped an MCP server with OAuth authentication! 🎉🤖

🔍 Three Key Takeaways:
1️⃣
2️⃣
3️⃣

Let's continue pushing the boundaries of what's possible in the world of AI and tool integration. Here's to many more innovations! 🚀
Shout out to @AIMakerspace !

#MCP #ModelContextProtocol #OAuth #Innovation #AI #TechMilestone

Feel free to reach out if you're curious or would like to collaborate on similar projects! 🤝🔥
```

## Submitting Your Homework 

Follow these steps to prepare and submit your homework assignment:

1. Review the MCP server code in `server.py` and the `app/` directory
2. Run the MCP server locally using `uv run server.py`
3. Connect to the server using an MCP client (e.g., Claude Desktop, or a custom client)
4. Test all available tools: browsing products, adding to cart, viewing cart, removing items, and checkout
5. Record a Loom video reviewing what you have learned from this session

## Questions

### Question #1

Why is OAuth important for MCP servers, and what security considerations should you keep in mind when exposing tools to AI clients?

#### Answer

OAuth matters here because the cat shop's MCP tools take real actions on a user's behalf. add_to_cart, remove_from_cart, and especially checkout change state. Once those tools are reachable by an AI client, the server has to answer two questions on every call: who is acting, and what are they allowed to do. OAuth solves this by letting a user grant the client a scoped, expiring access token through an explicit consent step, instead of sharing a password. The client gets the token, the tools use it to identify the user, and the user never hands credentials to the agent.

Security considerations when exposing tools to AI clients:
* Short lived tokens and revocation: access tokens expire (about 1 hour) and can be revoked, so a leaked or compromised client has limited blast radius.
* Least privilege and scopes: mutating tools like checkout deserve tighter authorization than read only tools like list_products.
* Audience binding (the confused deputy problem): the server should verify the token was actually issued for this server, not blindly accept any valid looking token.
* The model is not a trust boundary: an AI client can be steered by prompt injection into calling tools, so safety has to come from authorization on the server, not from the model behaving.
* Transport hygiene: use TLS and HTTPS in transit, and never log tokens.

### Question #2

What is Streamable HTTP transport in MCP, and why might you expose a server publicly with OAuth instead of using a local stdio connection?

#### Answer

MCP separates two things: the protocol (the JSON-RPC messages such as initialize, tools/list, and tools/call) and the transport (how those bytes actually move). Streamable HTTP is a transport that exposes a single HTTP endpoint, commonly /mcp. The client POSTs JSON-RPC messages over the network, and the server can stream responses back over a long lived connection. It replaced the older two endpoint HTTP plus SSE design.
The contrast is stdio. With stdio the client launches the server as a local subprocess and talks to it over stdin and stdout. stdio assumes one user, same machine, implicitly trusted, so it usually needs no auth.
You would expose a server publicly with OAuth instead of stdio when it needs to be reachable by remote clients or multiple users, for example a hosted or shared server, or reaching it through an ngrok tunnel. stdio cannot do that because it is local only. And the moment the server is network accessible, anonymous calls to mutating tools like checkout are unacceptable, so you add OAuth for authentication and authorization.

## Activity 1: Extend the MCP Server

Add at least one new tool to the cat shop MCP server (e.g., search_products, update_cart_quantity, or get_order_history). Ensure the new tool integrates properly with the existing database and OAuth authentication. Demo the new tool through an MCP client and include it in your Loom video.

I added two new tools to app/tools.py, both registered on the shared mcp instance and backed by the existing catshop.db SQLite database.

search_products(query)

A read only tool that searches the catalog by keyword, matching product name, description, or category, case insensitive using COLLATE NOCASE. It reuses oauth_provider._get_db(), so it queries the same product table as list_products.

Demo: search for salmon returns Salmon Treats.

update_cart_quantity(product_id, quantity)

A write tool that sets the exact quantity of an item already in the cart, where a quantity of 0 removes it. Like add_to_cart, it calls _get_username(), so it is gated by the same OAuth login and scoped to the authenticated user's cart.

Demo: change product 1 to 5 in my cart updates the cart to quantity 5.

Both tools appear in the client's loaded tool list and were demoed live in the Loom video.

## Advanced Activity: Build a Custom MCP Client

Build a custom MCP client that connects to the cat shop server over Streamable HTTP, authenticates via OAuth, and orchestrates a multi-step shopping flow (browse → add to cart → checkout). Compare the developer experience of MCP-based tool integration vs. traditional REST API calls.

Include your findings and a demo in your Loom video.
