# Kako se pravi chatbot

> Sekcija 2 od [materijala za seminar](../README.md)

## 1. Problem: model nema memoriju

```mermaid
sequenceDiagram
    participant U as Pozivalac
    participant M as LLM
    U->>M: "Zovem se Ana."
    M-->>U: "Drago mi je, Ana!"
    U->>M: "Kako se zovem?"
    M-->>U: "Ne znam kako se zoveš."
```

- Svaki poziv je **bez stanja** (stateless)

## 2. Rešenje: ponovo pošalji istoriju

```mermaid
sequenceDiagram
    participant U as Pozivalac
    participant M as LLM
    U->>M: "Zovem se Ana."
    M-->>U: "Drago mi je, Ana!"
    U->>M: "Zovem se Ana. Drago mi je, Ana! Kako se zovem?"
    M-->>U: "Zoveš se Ana."
```

- **Pozivalac** čuva istoriju, ne model
- Svaki poziv šalje **ceo razgovor** iznova

## 3. Poruke i uloge

```mermaid
flowchart LR
    MSG["Poruke<br/>system / user / assistant"] --> TPL["Šablon"] --> SEQ["Jedna sekvenca tokena<br/>sa specijalnim tokenima za uloge"] --> M["LLM"]
```

```text
<|system|> Odgovaraj jednom rečenicom.
<|user|> Koji je glavni grad Francuske?
<|assistant|>
```

- Istorija = **lista poruka**, svaka ima ulogu
- Uloge su označene **specijalnim tokenima** u jednoj ravnoj sekvenci
- Model samo **nastavlja tekst** posle `<|assistant|>`
- `system` poruka = uputstvo koje određuje ponašanje

## 4. Petlja razgovora

```mermaid
flowchart TD
    A["Čekaj unos korisnika"] --> B["Dodaj korisnikovu poruku u istoriju"]
    B --> C["Pošalji celu istoriju LLM-u"]
    C --> D["Dodaj odgovor kao assistant poruku"]
    D --> E["Prikaži odgovor"]
    E --> A
```

```python
history = [{"role": "system", "content": "Ti si koristan asistent."}]
while True:
    history.append({"role": "user", "content": input("> ")})
    reply = llm(history)
    history.append({"role": "assistant", "content": reply})
    print(reply)
```

- Chatbot = **petlja + lista** oko LLM-a bez stanja

## 5. Istorija raste

- Svaki krug čini sledeći poziv većim → **sporije i skuplje**
- Prozor pun? Izbaci najstarije poruke ili ih sažmi

## Zaključci

- „Razgovor“ živi kod pozivaoca, a ne u modelu
- Chatbot = čuvaj listu poruka, pošalji je celu, dodaj odgovor, ponovi

---

Prethodno: [Kako rade LLM-ovi](01-how-llms-work.md) · Sledeće: [Šta radi harness](03-harness.md)
