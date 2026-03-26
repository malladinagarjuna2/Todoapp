# GSoC 2025 Proposal: Web Components for Calico

---

## Personal Information

- **Full Name:** [Your Full Name]
- **Email:** [Your Email]
- **GitHub:** [Your GitHub Username]
- **LinkedIn:** [Your LinkedIn URL]
- **Location & Timezone:** [Your Location, e.g., City, Country (UTC+X)]
- **University/Institution:** [Your University]
- **Program & Year:** [e.g., B.Tech Computer Science, 3rd Year]
- **Phone/Telegram/Discord:** [Your Preferred Contact]

---

## Title

**Web Components for Calico: Typed, Reactive Bindings for the Web Component Ecosystem**

---

## Synopsis

Calico is a pure, reactive UI library for Scala.js that leverages Cats Effect and FS2 to provide a type-safe, composable DSL for building web applications. However, Calico currently lacks first-class support for [Web Components](https://developer.mozilla.org/en-US/docs/Web/API/Web_components) — the browser-native standard for creating reusable, framework-agnostic custom elements. This project will enable Calico users to seamlessly consume the vast ecosystem of existing web component libraries (e.g., Shoelace, Material Web Components, Vaadin) by:

1. Extending Calico's HTML DSL to support custom element tags, attributes, properties, slots, and events.
2. Building a code-generation pipeline that reads web component definitions (from Custom Elements Manifest or similar sources) and produces typed Calico bindings.
3. Delivering a reference binding library for a popular web component suite to validate the approach.

The result is that Calico users gain access to hundreds of production-quality UI components with full type safety and reactive Signal integration — without leaving Calico's functional, resource-safe paradigm.

---

## Benefits to Community

### For the Calico / Typelevel Ecosystem
- **Access to a vast component library:** The web component ecosystem includes hundreds of polished, accessible, well-tested UI components. Rather than building every UI widget from scratch in Scala, Calico users can leverage existing work.
- **Framework-agnostic interop:** Web Components are a W3C standard supported by all modern browsers. Supporting them positions Calico as a framework that embraces web standards rather than fighting them.
- **Code-generation sets a precedent:** The code-gen tooling built in this project can be reused for future integrations (SVG elements, other custom element registries, etc.), extending Calico's existing `scala-dom-types`-based generation pipeline.

### For the Scala.js Community
- **Bridges a significant gap:** No Scala.js UI framework currently has a mature, code-generated web component integration story. This would be a first-mover advantage for the Typelevel.js ecosystem.
- **Educational value:** The project demonstrates how functional, purely-reactive abstractions (Cats Effect `Resource`, FS2 `Signal`) can model real-world imperative browser APIs like Custom Elements.

### For Open Source
- **Reusable tooling:** The Custom Elements Manifest parser and code-generator can be published as standalone tools, benefiting other Scala.js projects.
- **Documentation and examples** will lower the barrier for newcomers to both Calico and Scala.js.

---

## Technical Background & Related Work

### Calico's Architecture

Calico models each UI component as a `Resource[IO, HtmlElement[IO]]`. The `Resource` manages the full lifecycle: allocation creates the DOM element and starts background fibers (event listeners, WebSocket connections); finalization cancels fibers and releases resources. This is built on:

- **Cats Effect** (`Async[F]`, `Resource[F, _]`) for safe concurrency and resource management.
- **FS2** (`Stream`, `Signal`, `SignallingRef`) for reactive state propagation. A `Signal[F, A]` represents a time-varying value; when bound to a DOM attribute or child, changes propagate automatically.
- **scala-dom-types** (via `com.raquo.domtypes`) for code-generating typed HTML tag, attribute, property, and event definitions at compile time. The `CalicoGenerator` and `DomDefsGenerator` in `project/src/main/scala/calico/html/codegen/` drive this process.

Key source files:
- `HtmlTag.scala` — Creates elements via `document.createElement(name)` and applies `Modifier`s.
- `HtmlAttr.scala` — Typed HTML attributes with `:=` (static) and `<--` (reactive `Signal`-bound) operators.
- `Prop.scala` — DOM property bindings (set via JS dictionary assignment).
- `Modifier.scala` — The core `Modifier[F, E, A]` typeclass that applies a value `A` to element `E` within `Resource[F, _]`.
- `EventProp.scala` — Event stream bindings using FS2 `Pipe`.
- `CalicoGenerator.scala` — Code-gen extending `CanonicalGenerator` from scala-dom-types.

### Web Components Standard

The [Web Components](https://developer.mozilla.org/en-US/docs/Web/API/Web_components) standard comprises:

1. **Custom Elements** — `customElements.define("my-tag", MyClass)` registers a new tag backed by a class extending `HTMLElement`. Lifecycle callbacks: `connectedCallback`, `disconnectedCallback`, `attributeChangedCallback`, `adoptedCallback`.
2. **Shadow DOM** — Encapsulated DOM subtree attached via `element.attachShadow()`. Provides style isolation.
3. **Slots** — Named projection points (`<slot name="header">`) in Shadow DOM where light DOM children are distributed.
4. **HTML Templates** — `<template>` elements for declarative DOM fragments (less relevant for Calico's programmatic approach).

From a Calico consumer's perspective, web components are custom HTML elements with:
- **Custom tag names** (must contain a hyphen, e.g., `sl-button`).
- **Attributes** (reflected from the DOM, always strings).
- **Properties** (set via JS properties, can be any type — objects, arrays, etc.).
- **Events** (custom events dispatched with `CustomEvent`, e.g., `sl-change`).
- **Slots** (named child projection points).
- **CSS Custom Properties / Parts** (for styling, out of scope for initial work).

### Custom Elements Manifest

The [Custom Elements Manifest](https://github.com/webcomponents/custom-elements-manifest) is a JSON schema (`custom-elements.json`) that describes a web component library's public API. Many major libraries ship this file. It includes:
- Tag names and their corresponding JS class names.
- Attributes with names, types, defaults, and descriptions.
- Properties with names, types, and whether they reflect to attributes.
- Events with names, detail types, and descriptions.
- Slots with names and descriptions.
- CSS custom properties and parts.

This manifest is the ideal input for Calico's code-generation pipeline.

### Existing Work & Comparison

- **Laminar** (Scala.js) — Has a manual approach to web components via `CustomHtmlTag`, requiring users to define attributes/properties by hand. No code-generation.
- **Lit** (TypeScript) — Defines web components with decorators; ships Custom Elements Manifest.
- **Stencil** — Web component compiler; also generates manifests.
- **scala-dom-types issue #97** — Tracks web component library definitions, but no implementation exists.

This project will be the first to provide a **code-generated, type-safe, reactive** web component integration for any Scala.js framework.

---

## Deliverables

### Required Deliverables

| # | Deliverable | Description |
|---|-------------|-------------|
| 1 | **Custom Element Tag DSL** | Extend `HtmlTag` to support custom element tag names with correct typing. A `WebComponentTag[F, E]` that creates elements via `document.createElement(tagName)` with the appropriate DOM element type and applies Calico modifiers. |
| 2 | **Property/Attribute Binding DSL** | Support web component-specific property types (not just strings). Extend `Prop` and `HtmlAttr` so code-generated bindings can model both reflected attributes and rich JS properties with reactive `Signal` binding. |
| 3 | **Custom Event Binding DSL** | Support custom events (e.g., `sl-change`) with typed event detail access. Extend `EventProp` to handle `CustomEvent` with typed `detail` field via FS2 `Pipe`. |
| 4 | **Slot Support** | A `slot` modifier/DSL that allows placing child elements into named slots: `myComponent(slot("header") := headerElement)`. |
| 5 | **Custom Elements Manifest Parser** | A Scala library (JVM-side, usable as an sbt plugin) that parses `custom-elements.json` into a structured Scala model (`ComponentDef`, `AttrDef`, `PropDef`, `EventDef`, `SlotDef`). |
| 6 | **Code Generator** | Extend Calico's existing `CalicoGenerator` pipeline to emit typed Scala.js bindings from parsed Custom Elements Manifest data. Each web component becomes a Scala object with typed tag, attributes, properties, events, and slots. |
| 7 | **Reference Binding: Shoelace** | A published `calico-shoelace` library generated from Shoelace's manifest, demonstrating the full workflow and serving as a real-world test. |
| 8 | **Documentation** | User guide covering: consuming web components in Calico, generating custom bindings, and contributing new component library support. |
| 9 | **Tests** | Unit tests for the manifest parser, integration tests for generated bindings using Scala.js test frameworks. |

### Optional / Stretch Deliverables

| # | Deliverable | Description |
|---|-------------|-------------|
| 10 | **CSS Custom Properties DSL** | Support for setting CSS custom properties (`--sl-color-primary`) via a typed DSL. |
| 11 | **Shadow DOM Introspection** | Utilities for querying elements inside a component's shadow root. |
| 12 | **sbt Plugin** | An sbt plugin that automatically downloads a component library's manifest and generates bindings as a source generator (similar to how `DomDefsGenerator` works today). |
| 13 | **Second Reference Binding** | Generate bindings for a second library (e.g., Material Web Components or Vaadin). |

---

## Detailed Design

### 1. Custom Element Tag DSL

Currently, `HtmlTag` creates elements via:

```scala
// HtmlTag.scala (current)
private def build = F.delay(dom.document.createElement(name).asInstanceOf[E])
```

Web components work identically — `document.createElement("sl-button")` returns the registered custom element. The key difference is that the DOM element type is `HTMLElement` (or a more specific subtype if the component's JS class is typed in scala-js-dom).

**Approach:** A `WebComponentTag[F]` will be generated per component:

```scala
// Generated code for Shoelace Button
object SlButton:
  val tag: HtmlTag[F, HTMLElement] = HtmlTag[F, HTMLElement]("sl-button", void = false)

  // Attributes
  val variant: HtmlAttr[F, String] = HtmlAttr("variant", encoders.identity)
  val size: HtmlAttr[F, String] = HtmlAttr("size", encoders.identity)
  val disabled: HtmlAttr[F, Boolean] = HtmlAttr("disabled", encoders.booleanAsAttrPresence)

  // Properties (rich types, set via JS property)
  val loading: Prop[F, Boolean, Boolean] = Prop("loading", encoders.identity)

  // Events
  val onSlFocus: EventProp[F, dom.FocusEvent] = EventProp("sl-focus", ...)
  val onSlBlur: EventProp[F, dom.FocusEvent] = EventProp("sl-blur", ...)

  // Slots
  val prefix: SlotModifier = SlotModifier("prefix")
  val suffix: SlotModifier = SlotModifier("suffix")
```

Usage would look like:

```scala
import calico.html.io.{*, given}
import calico.shoelace.SlButton

SlButton.tag(
  SlButton.variant := "primary",
  SlButton.size := "large",
  SlButton.disabled <-- isDisabledSignal,
  SlButton.onSlFocus(_ => IO.println("focused!")),
  SlButton.prefix(iconElement),
  "Click me"
)
```

This is idiomatic Calico — every modifier is applied via the existing `Modifier[F, E, M]` typeclass within a `Resource`.

### 2. Slot Support

Slots require placing a `slot` attribute on the child element. We introduce a `SlotModifier`:

```scala
final class SlotModifier private[calico] (slotName: String):
  def apply[M](child: Resource[F, HtmlElement[F]]): Resource[F, HtmlElement[F]] =
    child.flatTap { el =>
      Resource.eval(F.delay(el.asInstanceOf[dom.Element].setAttribute("slot", slotName)))
    }
```

This sets the `slot` attribute on the child element before it is appended, ensuring correct Shadow DOM distribution.

### 3. Custom Event Detail Typing

Web components dispatch `CustomEvent` with a typed `detail` field. We extend `EventProp`:

```scala
// For a custom event like sl-change with detail of type { value: String }
val onSlChange: EventProp[F, String] =
  EventProp("sl-change", e => e.asInstanceOf[dom.CustomEvent].detail.asInstanceOf[String])
```

For complex detail types, code-generation will produce Scala.js facade traits:

```scala
@js.native
trait SlChangeDetail extends js.Object:
  val value: String = js.native

val onSlChange: EventProp[F, SlChangeDetail] =
  EventProp("sl-change", e => e.asInstanceOf[dom.CustomEvent].detail.
  asInstanceOf[SlChangeDetail])
```

### 4. Custom Elements Manifest Parser

The parser will be a JVM-side library using circe for JSON parsing:

```scala
// Model
case class CustomElementsManifest(
  schemaVersion: String,
  modules: List[Module]
)

case class Module(
  path: String,
  declarations: List[Declaration],
  exports: List[Export]
)

case class CustomElementDeclaration(
  tagName: String,
  name: String,               // JS class name
  attributes: List[AttrDef],
  members: List[MemberDef],   // includes properties
  events: List[EventDef],
  slots: List[SlotDef],
  cssProperties: List[CssPropertyDef],
  cssParts: List[CssPartDef],
  superclass: Option[Reference],
  description: Option[String]
)

case class AttrDef(name: String, fieldName: Option[String], tpe: Option[TypeRef],
default: Option[String], description: Option[String])
case class EventDef(name: String, tpe: Option[TypeRef], description: Option[String])
case class SlotDef(name: String, description: Option[String])
```

### 5. Code Generator

The code generator extends Calico's existing pipeline. It will:

1. Read the `custom-elements.json` manifest.
2. For each `CustomElementDeclaration`, emit a Scala object containing:
   - A `tag` val of type `HtmlTag[F, HTMLElement]`.
   - Vals for each attribute (`HtmlAttr[F, V]`), property (`Prop[F, V, DomV]`), event (`EventProp[F, E]`), and slot (`SlotModifier`).
3. Map manifest type strings to Scala types:
   - `"string"` -> `String`
   - `"boolean"` -> `Boolean`
   - `"number"` -> `Double`
   - Complex/union types -> `String` (with docs noting the accepted values)
4. Generate `given` instances for `Modifier` to ensure all generated bindings work seamlessly with Calico's existing composition model.

The generator will be structured as an extension of the existing `DomDefsGenerator` pattern:

```scala
object WebComponentGenerator:
  def generate(
    manifest: CustomElementsManifest,
    packageName: String,
    srcManaged: File
  ): IO[List[File]] =
    manifest.modules.flatMap(_.declarations).collect {
      case ce: CustomElementDeclaration => generateComponent(ce, packageName,
      srcManaged)
    }.sequence
```

### 6. Integration with sbt Build

Following Calico's existing pattern in `build.sbt`:

```scala
lazy val shoelace = project
  .in(file("shoelace"))
  .enablePlugins(ScalaJSPlugin)
  .settings(
    name := "calico-shoelace",
    Compile / generateWebComponentDefs := {
      WebComponentGenerator.generate(
        manifest = parseManifest(baseDirectory.value / "custom-elements.json"),
        packageName = "calico.shoelace",
        srcManaged = (Compile / sourceManaged).value / "webcomponents"
      ).unsafeRunSync()
    },
    Compile / sourceGenerators += (Compile / generateWebComponentDefs)
  )
  .dependsOn(calico)
```

---

## Timeline

### Community Bonding Period (May 8 - June 1)

- Deep-dive into Calico's codebase, particularly the codegen pipeline (`CalicoGenerator`, `DomDefsGenerator`, `TagDefMapper`).
- Study the Custom Elements Manifest specification in detail.
- Survey popular web component libraries (Shoelace, Material Web, Vaadin, FAST) and their manifest completeness.
- Set up development environment with Scala.js, sbt, and a test harness.
- Discuss design decisions with mentor (@armanbilge): naming conventions, module structure, which component library to target first.
- Write a design document summarizing the approach for mentor review.

### Phase 1: Core DSL Extensions (June 2 - June 29) [4 weeks]

**Week 1-2: Custom Element Tag & Attribute Support**
- Implement `WebComponentTag` that works with Calico's existing `Modifier` system.
- Verify that `HtmlTag` already works for custom elements (it should, since `document.createElement` handles registered custom elements).
- Add support for boolean attributes (presence/absence) specific to web components.
- Write manual bindings for 2-3 Shoelace components to validate the approach.
- Unit tests.

**Week 3: Property, Event, and Slot Support**
- Implement `SlotModifier` for named slot distribution.
- Verify `EventProp` works with `CustomEvent` and typed `detail` extraction.
- Handle property types beyond strings (objects, arrays) via `Prop` with appropriate encoders.
- Write integration tests with Shoelace components in a Scala.js test environment.

**Week 4: Refinement & Buffer**
- Address feedback from mentor on DSL design.
- Refactor as needed for consistency with Calico conventions.
- Document the manual binding approach.

**Milestone 1 (June 29):** Manual web component bindings work end-to-end with Calico's reactive DSL. A developer can hand-write bindings for any web component and use them idiomatically.

### Phase 2: Code Generation (June 30 - August 3) [5 weeks]

**Week 5-6: Custom Elements Manifest Parser**
- Implement the JSON parser using circe.
- Model the full manifest schema (modules, declarations, attributes, properties, events, slots, CSS properties, CSS parts, superclass references).
- Handle inheritance (components extending other components).
- Handle type resolution (manifest type strings to Scala types).
- Comprehensive tests against real manifests (Shoelace, Lit examples).

**Week 7-8: Code Generator**
- Build the Scala source code generator that emits typed component objects.
- Integrate with Calico's existing `CalicoGenerator` patterns.
- Handle edge cases: reserved Scala keywords as names, naming collisions, components with no attributes/events.
- Generate Scaladoc from manifest descriptions.
- Test generated code compiles and works correctly.

**Week 9: Shoelace Reference Binding**
- Generate full `calico-shoelace` bindings from Shoelace's manifest.
- Build example applications using generated bindings.
- Fix any issues discovered during real-world usage.
- Performance validation (ensure no overhead vs. manual bindings).

**Milestone 2 (August 3):** Code generator produces working, typed Shoelace bindings. Example applications demonstrate real-world usage.

### Phase 3: Polish, Documentation & Stretch Goals (August 4 - August 25) [3 weeks]

**Week 10: sbt Plugin & Build Integration**
- Package the generator as an sbt plugin or sbt task.
- Document how to add a new web component library.
- CI integration for the generated bindings.

**Week 11: Documentation & Examples**
- Write user guide: "Using Web Components in Calico".
- Write contributor guide: "Generating Bindings for a New Component Library".
- Build a showcase application demonstrating various Shoelace components with Calico's reactive features (signals, resources, router integration).

**Week 12: Stretch Goals & Final Submission**
- Attempt CSS Custom Properties DSL (stretch goal).
- Attempt a second component library binding (stretch goal).
- Final code cleanup, review, and submission.
- Write a blog post / project summary.

**Final Milestone (August 25):** All required deliverables complete, documented, and tested. Reference binding published. Stretch goals attempted as time permits.

---

## Biographical Information

- **Education:** [Your Degree, University, Expected Graduation]
- **Relevant Coursework:** [e.g., Functional Programming, Web Development, Compiler Design, Type Systems]
- **Programming Experience:**
  - **Scala:** [Describe your experience — projects, duration, libraries used]
  - **Scala.js:** [Any experience with Scala.js]
  - **Functional Programming:** [Experience with Cats, Cats Effect, FS2, or similar]
  - **Web Development:** [Experience with HTML/CSS/JS, Web APIs, web components]
  - **Code Generation:** [Any experience with metaprogramming, macros, codegen tools]
- **Open Source Contributions:** [List any PRs, issues, or projects — especially in Typelevel or Scala ecosystem]
- **Relevant Projects:**
  - [Project 1: Brief description and what you learned]
  - [Project 2: Brief description and what you learned]
- **Why this project:** [1-2 sentences on why you're interested in this specific project]

---

## Availability & Commitments

- **Hours per week:** [e.g., 30-35 hours/week — adjust based on your situation]
- **Other commitments:** [Classes, exams, part-time job — be transparent]
- **Known absences:** [Any planned time off during the GSoC period]
- **Timezone overlap with mentor:** [Describe your availability for sync communication]

---

## Communication Plan

- **Weekly sync:** 30-minute video/voice call with @armanbilge to review progress and discuss design decisions.
- **Async communication:** Typelevel Discord (#calico channel) for day-to-day questions.
- **Progress tracking:** Weekly status updates posted to the project's GitHub Discussion or issue tracker.
- **Code review:** Pull requests submitted at each milestone for mentor review before merging.

---

## Appendix: Example End-to-End Usage

This appendix shows what the final developer experience will look like.

### Installing

```scala
// build.sbt
libraryDependencies += "com.armanbilge" %%% "calico-shoelace" % "0.1.0"
```

### Using a Shoelace Dialog with Calico

```scala
import calico.*
import calico.html.io.{*, given}
import calico.shoelace.{SlButton, SlDialog, SlInput}
import cats.effect.*
import fs2.concurrent.*

object MyApp extends IOWebApp:
  def render: Resource[IO, HtmlElement[IO]] =
    (
      SignallingRef[IO].of(false),
      SignallingRef[IO].of("")
    ).tupled.toResource.flatMap { (isOpen, userName) =>
      div(
        // Open the dialog
        SlButton.tag(
          SlButton.variant := "primary",
          SlButton.onClick(isOpen.set(true)),
          "Open Dialog"
        ),

        // The dialog itself
        SlDialog.tag(
          SlDialog.label := "Enter your name",
          SlDialog.open <-- isOpen,
          SlDialog.onSlAfterHide(_ => isOpen.set(false)),

          // Dialog body content
          SlInput.tag(
            SlInput.placeholder := "Your name",
            SlInput.onSlInput.withSelf { self =>
              _.foreach(_ => self.value.get.flatMap(userName.set))
            }
          ),

          // Footer slot
          SlDialog.footer(
            SlButton.tag(
              SlButton.variant := "primary",
              SlButton.onClick(isOpen.set(false)),
              "Close"
            )
          )
        ),

        // Reactive output
        span(" Hello, ", userName.map(n => if n.isEmpty then "stranger" else n))
      )
    }
```

This example demonstrates:
- **Custom element tags** (`SlButton.tag`, `SlDialog.tag`, `SlInput.tag`)
- **Typed attributes** (`variant := "primary"`, `label := "Enter your name"`)
- **Reactive property binding** (`open <-- isOpen` binds a `Signal[IO, Boolean]`)
- **Custom events** (`onSlAfterHide`, `onSlInput`)
- **Named slots** (`SlDialog.footer(...)`)
- **Full integration** with Calico's `Resource`-based lifecycle and `SignallingRef` state management
