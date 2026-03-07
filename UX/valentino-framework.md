# Valentino Framework: Le Regole d'Oro della UI EasyWay

Questo documento formalizza i principi di design e User Experience adottati nel Portale EasyWay, denominati affettuosamente "Valentino Framework" (Il Sarto del Frontend). 

Ogni Agente (es. Kortex, Coder) o sviluppatore umano che tocca l'interfaccia deve attenersi a queste regole per garantire un aspetto Premium, "Hextech" e Sovereign.

## 1. Gerarchia Visiva delle Call-To-Action (CTAs)

Non tutti i bottoni sono uguali. L'interfaccia deve guidare l'utente senza farlo pensare.

- **`.btn-primary`**: È l'azione principale assoluta (es. "Richiedi Accesso" o "Deploy"). Deve spiccare. Ha un gradiente di sfondo e, all'hover, subisce un salto verso l'alto (`transform: translateY(-2px)`) con un forte incremento del glow (`box-shadow`).
- **`.btn-glass`**: È l'azione secondaria (es. "Scopri di più"). Usa bordi semi-trasparenti e uno sfondo quasi invisibile (backdrop filter). All'hover, subisce uno spostamento minore (`translateY(-1px)`) e si "riempie" leggermente di azzurro.
- **`.btn-github`**: Usato per i link Open Source. Stile simile al glass ma con dominante grigia/bianca trasparente per non interferire con il colore brand "Hextech Cyan" delle azioni di business.

## 2. Ritmo degli Spazi (Micro-Tipografia)

L'interfaccia deve "respirare". 
- **Raggruppamento Logico**: Titolo principale (`h1`) e sottotitolo (`p.tagline`) devono essere considerati un singolo blocco di lettura. Il `margin-bottom` del titolo deve essere contenuto (es. `0.5rem`).
- **Respiro prima dell'Azione**: Il contenitore delle azioni (es. `.actions` box dei bottoni) deve avere uno stacco netto dal testo sovrastante (es. `margin-top: 3rem`) per dare peso alla decisione che l'utente sta per prendere.

## 3. Reattività del Contesto ("The Living UI")

L'applicativo non è statico; risponde e "respira" anche quando l'utente è fermo.
- **Elementi Energetici**: Componenti come il "Core Neurale" o i dispositivi di attivazione (`#gedi-guardian`) devono avere una pulsazione infinita (`@keyframes`) che varia leggermente raggio d'ombra (`box-shadow`) e scala (`transform: scale(...)`). Questo crea l'illusione di un reattore attivo o di un'Intelligenza vigile.

## 4. Micro-Interazioni

Ogni elemento interattivo deve premiare l'intenzione dell'utente **prima** del click.
- Tutte le micro-interazioni (hover sui bottoni, card, link nel nav) devono usare transizioni morbide: `transition: all var(--transition-fast);`.
- Evitare contrasti violenti: usare giochi di ombre (glow), sfocature (`backdrop-filter`) e spostamenti su asse Y.

---
*Architettura d'Interfaccia "Sovereign Intelligence". "L'Automazione è il manovale, ma il Framework è Sovrano."*
