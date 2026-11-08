# V1 architecture diagrams

Diagrams for the TDD's "Architecture and components" section.

## High level

The whole V1 design in one glance. The complete diagram below opens up each box.

```mermaid
flowchart LR
    Browser(["Browser"])

    subgraph Service["Go service (one binary, one deployable)"]
        Server["Server<br/>middleware, handlers, templates"]
        Matcher{{"Matcher"}}
        Classifier{{"Classifier"}}
        Catalog{{"Catalog"}}
    end

    Data[/"Taxonomy and dataset files<br/>embedded, validated at startup"/]
    LLM[("LLM provider API")]

    Browser <-->|"pages, htmx, JSON"| Server
    Server -->|"phrase"| Matcher
    Matcher -->|"phrase"| Classifier
    Classifier <-->|"prompt and mood"| LLM
    Matcher -->|"mood"| Catalog
    Data -->|"loaded once"| Catalog
    Data -->|"loaded once"| Classifier
```

In words: the browser talks only to the server. The server hands the phrase to the `Matcher`. The matcher asks the `Classifier` for a mood, which is the only step that leaves the service, and then asks the `Catalog` for items in that mood. The data files are read once at startup.

## Complete architecture

How to read it:
- Solid arrows mean "calls" or "uses". Dotted arrows are wiring, configuration, or something that replaces a component in tests or a later version.
- Hexagons are interfaces. The implementation V1 uses is named inside the hexagon.
- Responses travel back along the same path the request took.

```mermaid
flowchart TB
    Browser(["Browser"])
    Gemini[("LLM provider API")]
    Env[/"Environment variables"/]
    Files[/"Taxonomy file and dataset files"/]
    Root["Composition root<br/>cmd/moodwise"]

    subgraph Startup["Startup (runs once, any error stops boot)"]
        Config["Config<br/>read and validated"]
        Loader["Loader<br/>parses and validates the files"]
        Taxonomy["Taxonomy<br/>immutable value"]
    end

    subgraph Service["Go service (one binary)"]
        MW["Middleware<br/>request ID, access logging, panic recovery,<br/>spend guard, security headers, timeouts"]

        subgraph Server["Server"]
            PageH["Page handler<br/>form post, htmx"]
            APIH["JSON handler<br/>POST /api/match"]
            Health["GET /healthz"]
            VM["View model<br/>exactly what the page shows"]
            Tpl["Templates and web assets<br/>embedded, parsed once"]
        end

        Matcher{{"Matcher (interface)<br/>V1: bucket matcher"}}
        Classifier{{"Classifier (interface)<br/>V1: LLM classifier, builds the prompt<br/>from the taxonomy and checks the answer"}}
        Adapter["Provider adapter (Gemini)<br/>the only code that imports an LLM SDK"]
        Catalog{{"Catalog (interface)<br/>V1: in-memory, items indexed by mood"}}
        Rand["Random source<br/>injected"]
    end

    FakeC["Fake classifier<br/>tests only"]
    Semantic["Semantic matcher<br/>V2"]

    Browser -->|"form post or JSON request"| MW
    MW --> PageH
    MW --> APIH
    MW --> Health

    PageH --> Matcher
    APIH --> Matcher
    PageH -->|"outcome becomes"| VM
    VM --> Tpl

    Matcher --> Classifier
    Matcher --> Catalog
    Classifier --> Adapter
    Adapter --> Gemini
    Catalog --> Rand

    Env --> Config
    Files --> Loader
    Loader --> Taxonomy
    Loader -->|"builds"| Catalog
    Taxonomy --> Classifier
    Taxonomy --> Tpl

    Root -.->|"reads"| Config
    Root -.->|"creates and wires"| Service
    Config -.->|"key, model, timeout"| Adapter
    Config -.->|"limits"| MW
    FakeC -.->|"replaces in tests"| Classifier
    Semantic -.->|"added in V2"| Matcher
```

What the diagram shows:
- **The handler knows one thing downstream:** the `Matcher`. It never touches the `Classifier` or the `Catalog`, which is the decision recorded in the TDD.
- **Only the provider adapter reaches the outside world,** and only it imports an LLM SDK.
- **The taxonomy is loaded once** by the loader and handed to the three consumers: the classifier, the templates, and the loader itself (which checks every dataset tag against it).
- **The composition root is the only place that knows the concrete types.** It reads the config, creates the components and wires them together.
- **Dotted nodes are not part of the V1 request path:** the fake classifier replaces the real one in tests, and the semantic matcher arrives in V2 beside the bucket matcher.

## Q2: the handler both ways

Option A, rejected: the handler orchestrates the steps itself.

```mermaid
flowchart LR
    H["Handler"] --> C{{"Classifier"}}
    H --> K{{"Catalog"}}
```

The handler knows that matching means "get a mood, then look up that mood". V2 matches by embeddings, with no mood in the middle, so the handler would have to be rewritten.

Option B, chosen: the handler calls one `Matcher`.

```mermaid
flowchart LR
    H["Handler"] --> M{{"Matcher"}}
    M --> B["Bucket matcher (V1)"]
    M -.-> S["Semantic matcher (V2)"]
    B --> C{{"Classifier"}}
    B --> K{{"Catalog"}}
```

The handler knows only "phrase in, outcome out". V2 adds a second `Matcher` implementation (dotted) and the handler is untouched.
