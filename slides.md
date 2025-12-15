---
theme: bricks
title: Collaboboard
info: |
  ## Collaboboard
  Realtime-Collaboration Board mit React und Y.js
class: text-center
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# duration of the presentation
duration: 15min
---

# Collaboboard
Realtime-Collaboration Board mit React und Y.js

---
transition: fade-out
layout: center
---
# Was ist Collaboboard?

Collaboboard ist ein Realtime-Collaboration Board mit React und Y.js

- <span v-mark.underline.orange>**Realtime-Collaboration**</span><v-click> - Mehrere Personen, gleichzeitig</v-click>
- <span v-mark.circle.blue>**Board**</span><v-click> - Whiteboard zum gemeinsamen Arbeiten </v-click>
- <span v-mark.underline.red>**React**</span><v-click> - reaktives Javascript Framework</v-click>
- <span v-mark.circle.green>**Y.js**</span><v-click> - synchronisiert den Status von mehreren Browsern</v-click>

---
dragPos:
  square: 786,145,174,100
---

# Features 

Collaboboard ist ein Realtime-Collaboration Board mit React und Y.js



<v-clicks>

- <span v-mark.circle.orange> **Zeichnen**</span> - Freihand, Formen, Pfeile, ... 
- **Bilder** - Bilder einfügen, annotieren und vergrößern 
<img v-drag="'square'" src="/logo.png" height="15vh">
- **Text** - Text hinzufügen, farblich markieren und suchen
- **Teilen** - Einfaches Teilen per Link oder QR-Code
- **E2E Verschlüsselt speichern** - Sicheres Ende-zu-Ende-Verschlüsseltes speichern der Daten
- **Export der Daten** - Einfaches exportieren und importieren der Daten
- **Mermaid** - Mermaid Diagramme einfügen und live bearbeiten
- **Dark-Mode/Light-Mode** - Integrierter Dark-Mode

</v-clicks>

---
layout: items
cols: 3
transition: slide-left
---
# Tech-Stack

::items::
<div class="grid-collumn">Typescript
<img class="mx-auto" width="50%" src="/ts.png"/>
</div>
<div width="50%"class="grid-collumn">
React mit Next.js
<img class="mx-auto" width="30%"src="/react.png"/>
<img class="mx-auto" width="50%"src="/nextjs.png"/>
</div>
<div width="50%"class="grid-collumn">
TailwindCSS
<img  class="mx-auto" width="50%"src="/tailwindcss.png"/>
</div>
<div width="50%"class="grid-collumn">
Excalidraw
<img  class="mx-auto" width="50%"src="/excalidraw.svg"/>
</div>
<div width="50%"class="grid-collumn">
Drizzle ORM
<img  class="mx-auto" width="50%"src="/drizzle.png"/>
</div>
<div width="50%"class="grid-collumn">
Websockets
<img  class="mx-auto" width="50%" src="/websockets.png"/>
</div>

---
layout: quote
---
## "Plan to throw one away; you will, anyhow."
### Fred Brooks
---
transition: slide-left
---

# 1. Iteration: Alles selbst mit Canvas

- Mit **fabric.js** und Canvas
- Sehr viel Aufwand
- ziemlich verbuggt
- Funktional aber wenig Features 

---
transition: slide-left
---

# 2. Iteration: Excalidraw mit WebRTC und Y.js

- Excalidraw statt **fabric.js**
- Deutlich weniger Aufwand
- Verbuggt
- Je nach Browser funktional oder nicht

---
transition: slide-left
---

# 3. Iteration: Excalidraw mit Websockets und Y.js

- Behebt alle Probleme mit WebRTC und Browserkompatibilität
- Speichern mit E2E-Verschlüsselung
- Konnektivität anzeigen und Autosave
- Weitere Usability Features
- Mermaid Diagramme

---
layout: section
transition: slide-left
---
# WebRTC vs Websockets
::right::
## WebRTC
- 93.17% Unterstützung (caniuse.com)
- Technologie von Videokonferencen
- Benötigt STUN/TURN-Server wegen NAT
- Grundsätzlich große Kompatibilität mit Browsern
- In Praxis jedoch immer wieder Probleme
- Schneller als Web Sockets, da UDP
---
layout: section
transition: slide-left
---
# WebRTC vs Websockets
::right::
## Web Sockets
- 93.6% Unterstützung (caniuse.com)
- Wird verwendet von vielen Websites
- Tests mit verschiedenen Browsern hat weniger Probleme bereitet
- Langsamer als WebRTC, da TCP 
- Zuverlässiger, da TCP
---

---
layout: section
transition: slide-left
---
# Datenbank
### Implementation  

::right::
```sql {1|2|3|4-5}
CREATE TABLE `whiteboards` (
	`id` text PRIMARY KEY NOT NULL,
	`encrypted_data` text NOT NULL,
	`updated_at` integer NOT NULL,
	`created_at` integer NOT NULL
);
```

- SQLite unterstützt keinen `UUID`-Typ deshalb `text`
- Kein Index benötigt, da UUID immer bekannt

---
layout: section
---
## E2E-Verschlüsselung
### Implementation  
::right::
- Wurde gemeinsam mit befreundeter Kryptographin entwickelt
- PBKDF2 mit 100k Iterationen ist teuer für Angreifer aber ok für User
- Salt mit 16 Bytes verhindert Rainbow-Table-Angriffe
- Blake3 als Hashing-Alghorithmus ist schnell und modern
- AES-256-GCM leakt keine Datenmuster, keine Manipulation, NIST-Standard, Webcrypto Hardwarebeschleunigung
- 12-Byte IV ist optimal für GCM und verhindert Replay- & Pattern-Angriffe

---
layout: section
transition: slide-left
---
## Websocket-Verbindung mit Y.js
::right::
```ts {1-3|5|6|7|8|9-11}
const ydoc = new Y.Doc();
const yElements = ydoc.getArray<Y.Map<any>>("elements");
const yAssets = ydoc.getMap("assets");

const provider = new WebsocketProvider(
    "ws://localhost:1234", 
    uuid, 
    ydoc, {
        connect: true,
        resyncInterval: 5000,
});
```
---
layout: section
transition: slite-left
---
## Verbindung von Y.js mit Excalidraw
::right::
```ts {1|2-5|6-9|11}
const binding = new ExcalidrawBinding(
    yElementsRef.current,
    yAssetsRef.current,
    apiRef.current,
    providerRef.current.awareness,
    {
        excalidrawDom: excalidrawRef.current!,
        undoManager: new Y.UndoManager(yElementsRef.current),
    },
);
bindingRef.current = binding;
```
---
layout: section
transition: slite-left
---
## Excalidraw initialisieren
::right::
### Vor crashes absichern durch entfernen korrupter Objekte
```ts
const safeElements = (elements: any[]) =>
Array.isArray(elements)
    ? elements.filter((el) => el 
    && typeof el === "object" && "id" in el)
    : [];
```

### Whiteboard zeichnen
```tsx {1|2|3-11}
<Excalidraw
        excalidrawAPI={(api) => (apiRef.current = api)}
        initialData={{
            elements: safeElements(
                yjsToExcalidraw(
                    yElementsRef.current ?? new Y.Array()
                ),
            ),
        }}
        ...
```

---
transition: slide-up
---
# Starten des Projekts

1. Klonen des Projekts
```bash
git clone https://github.com/Y0ngg4n/collaboboard
cd collaboboard
```

<v-click>
2. Konsole
```bash
pnpm npx drizzle-kit push
pnpm run dev
```
</v-click>

<v-click>
3. weitere Konsole
```bash
HOST=localhost PORT=1234 pnpm npx y-websocket
```
</v-click>

<v-click>
4. Im Browser <a href="http://localhost:3000">http://localhost:3000</a> öffnen
</v-click>

---
transition: slide-down
layout: iframe
url: http://localhost:3000
---

---
layout: statement
transition: slide-down
---
# Danke
## Habt ihr noch Fragen?
