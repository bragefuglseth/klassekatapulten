<script>
    let mode = "setup";

    let group = {
        name: "1A",
        members: [
            { name: "Ole", id: "1" },
            { name: "Dole", id: "2" },
            { name: "Doffen", id: "3" },
            { name: "Pluto", id: "4" }
        ],
        layout: [
            [2],
            [2],
        ],
        lastGenerated: "2007-04-05",
        seating: [
            [["Ole", "Dole"]],
            [["Doffen", "Pluto"]]
        ],
        previousSeatings: []
    };

    let files = null;

    // TODO: Fiks dette
    $: if (files) {
        let file = files[0];
        if (file) {
            let p = files[0].text();
            p.then(text => {
                group = JSON.parse(text);
                files = null;
            })
            .catch(err => {
                console.log(err);
            });
        }
    }

    function addMember() {
        group.members.push({ name: "", id: crypto.randomUUID() });
        group.members = group.members;
    }

    function removeMember(id) {
        group.members = group.members.filter(member => member.id != id);
    }

    function addLayoutColumn() {
        let len = group.layout.length
        group.layout.push([...group.layout[len - 1]]);
        group.layout = group.layout;
    }

    function removeLayoutColumn() {
        group.layout.pop();
        group.layout = group.layout;
    }

    function addLayoutRow() {
        let len = group.layout[0].length;
        for (let i = 0; i < group.layout.length; i++) {
            group.layout[i].push(group.layout[i][len - 1]);
        }
        group.layout = group.layout;
    }

    function removeLayoutRow() {
        for (let i = 0; i < group.layout.length; i++) {
            group.layout[i].pop();
        }
        group.layout = group.layout;
    }

    function benchHTML(size) {
        let html = "<div class=\"bench\">";
        for (let i = 0; i < size; i++) {
            html += "<div class=\"bench-inner setup\"></div>";
        }
        html += "</div>";
        return html;
    }

    function updatePreviousSeatings() {
        group.previousSeatings.push(group.seating.slice(0));

        if (group.previousSeatings.length > 3) {
            group.previousSeatings.splice(0, group.previousSeatings.length - 3);
        }
    }

    function generateSeating() {
        updatePreviousSeatings();

        let previousCombos = [];

        for (let previousSeating of group.previousSeatings) {
            for (let col of previousSeating) {
                for (let bench of col) {
                    previousCombos.push(bench);
                }
            }
        }

        let members = group.members.map(m => m.name);

        group.seating = [];

        for (let col of group.layout) {
            group.seating.push([]);
        }
        
        for (let i = 0; i < group.layout[0].length; i++) {
            for (let j = 0; j < group.layout.length; j++) {
                let bench = [];

                while (bench.length < group.layout[j][i]) {
                    let randomIndex = Math.floor(Math.random() * members.length);

                    let candidate = members[randomIndex];
                    let candidateBenchmates = previousCombos
                        .filter(combo => combo.includes(candidate))
                        .map(combo => combo.filter(name => name != candidate))
                        .flat();

                    if (!candidateBenchmates.some(benchmate => bench.includes(benchmate)) || members.length <= candidateBenchmates.length) {
                        bench.push(members.length > 0 ? members.splice(randomIndex, 1)[0] : "");
                    }
                }
                group.seating[j].push(bench);
            }
        }
        
        group.seating = group.seating;

        group.lastGenerated = new Date().toISOString();
        lastGeneratedString = new Date(group.lastGenerated).toLocaleDateString();
    }

    function saveFile() {
        const file = new Blob([JSON.stringify(group)], {type: "application/json"});
        const a = document.createElement("a");
        const url = URL.createObjectURL(file);
        a.href = url;
        a.download = group.name + "-klassekart.json";
        document.body.appendChild(a);
        a.click();
        setTimeout(function() {
            document.body.removeChild(a);
            window.URL.revokeObjectURL(url);  
        }, 0); 
    }

    $: studentBenchBalance = group.layout.flat().reduce((partialSum, a) => partialSum += a) - group.members.length;

    let lastGeneratedString = new Date(group.lastGenerated).toLocaleDateString();
</script>

<style>
    @import url("https://rsms.me/inter/inter.css");

    /* CSS */
    :root {
        font-family: Inter, sans-serif;
        font-feature-settings: 'liga' 1, 'calt' 1; /* fix for Chrome */
    }
    @supports (font-variation-settings: normal) {
        :root { font-family: InterVariable, sans-serif; }
    }

    button {
        font-family: inherit;
        color: inherit;
    }

    h2 {
        font-size: 1.5rem;
    }

    :global(body) {
        overflow: auto;
    }

    p {
        color: #444;
        line-height: 1.5;
        margin: 0.5em 0;
    }

    .flat {
        background: none;
        border: none;
        color: rgba(0, 0, 0, 0.7);
        font-size: 1.5rem;
        font-weight: 200;
        transition: transform 200ms;
    }

    .vertical-align {
        display: flex;
        align-items: center;
        gap: 0.2rem;
    }

    .flat:hover, button.circle:hover {
        opacity: 0.8;
        transform: scale(1.2);
    }

    .flat:disabled, button.circle:disabled {
        opacity: 0.5;
    }

    .pill {
        padding: 1em 2em;
        border: none;
        background-color: whitesmoke;
        border: 1px solid silver;
        font-weight: 700;
        border-radius: 9999px;
        transition: opacity 200ms;
        align-self: center;
    }

    .suggested {
        background-color: #b635b7;
        border: transparent;
        color: white;
    }

    .circle {
        width: 1.5em;
        height: 1.5em;
        font-size: 1.2rem;
        text-align: center;
        border: 1px solid grey;
        border-radius: 9999px;
        background-color: transparent;
        transition: transform 200ms;
    }

    .pill:hover {
        opacity: 0.7;
    }

    .setup-container {
        display: flex;
        flex-direction: column;
        max-width: 60ch;
        margin: 2rem auto;
    }

    .header {
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    .class-header-container {
        display: flex;
        justify-content: space-between;
        align-items: baseline;
    }

    .class-header {
        display: flex;
        align-items: baseline;
        gap: 0.5ch;
    }

    .class-header input {
        border: transparent;
        border-bottom: 2px dotted silver;
        width: 12ch;
        font-size: 1.5rem;
        padding: 0.1em 0.2em;
    }

    .member-container {
        margin-top: 0;
        list-style-type: none;
        padding-left: 0;
        background: whitesmoke;
        border: 1px solid silver;
        border-radius: 12px;
    }

    .member {
        height: 2rem;
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 0.5rem;
    }

    .member > input {
        background-color: transparent;
    }

    .member:not(:last-child) {
        border-bottom: 1px solid silver;
    }

    .member > input {
        flex-grow: 1;
        border: none;
    }

    .seating-container {
        display: flex;
        flex-direction: row;
        gap: 10px;
    }

    .seating-row {
        display: flex;
        flex-direction: column-reverse;
        gap: 10px;
        align-items: center;
    }
    :global(.bench) {
        background-color: whitesmoke;
        display: flex;
        border-radius: 10px;
        border: 1px solid silver;
    }
    :global(.bench-inner) {
        height: 4rem;
        width: 6rem;
        display: flex;
        justify-content: center;
        align-items: center;
        text-align: center;
    }
    :global(.bench-inner.setup) {
        height: 3rem;
        max-height: 3rem;
        width: 3rem;
        max-width: 3rem;
    }
    .board {
        border: 1px solid silver;
        padding: 0.2em 1em;
        width: 20ch;
        text-align: center;
        border-radius: 10px;
        margin-top: 0.8rem;
        align-self: center;
    }
    .bench.disabled {
        background-color: transparent;
        border: 1px solid lightgrey;
    }
    .bench.invisible {
        background-color: transparent;
        border: 1px solid transparent;
    }

    :global(.bench-inner:not(:last-child)) {
        border-right: 1px solid silver;
    }

    .setup-bench-button-group {
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .vertical-group {
        display: flex;
        flex-direction: column;
        align-self: center;
        gap: 0.5rem;
    }

    .horizontal-group {
        display: flex;
        flex-direction: row;
        align-self: center;
        gap: 0.5rem;
    }

    .seating-container-outer {
        display: flex;
        flex-direction: column;
        align-items: center;
        align-self: flex-start;
        margin-bottom: 2rem;
    }

    #file-input {
        opacity: 0;
        display: absolute;
        z-index: -1;
    }

    .bench-input {
        width: 100%;
        text-align: center;
        background: transparent;
        border: none;
        overflow-x: auto;
        max-height: 100%;
        word-wrap: break-word;
    }

    .seating-wrapper-wrapper {
        margin: 2em 2em;
        display: flex;
        flex-direction: column;
        align-items: flex-start;
    }

    .gap {
        display: flex;
        gap: 0.5rem;
    }

    .seating-wrapper {
        margin-bottom: 1em;
        display: flex;
        flex-direction: column;
    }

    .legal {
        color: #777;
        font-size: 0.8rem;
        text-align: center;
        margin-top: 2.5rem;
    }

    .onlyprint {
        display: none;
    }

    @media print {
        @page {
            size: landscape;
        }

        .noprint {
            display: none;
        }

        .onlyprint {
            display: initial;
        }

        .seating-wrapper-wrapper {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
    }
</style>

<input type="file" id="file-input" bind:files={files} />
<div id="wrapper">
    {#if mode == "setup"}
        <div class="setup-container">
            <header class="header">
                <h1><img width="270" src="/logo.svg" alt="logo"></h1>
                
                <div class="horizontal-group">
                    <label class="flat" for="file-input" title="Åpne fil"><img alt="mappe" height="24" src="/mappe.svg"></label>
                    <button class="flat" title="Last ned fil" onclick={saveFile}><img alt="lagringsikon" height="24" src="/lagre.svg"></button>
                </div>
            </header>
            <p>Det har aldri vært enklere å lage klassekart. Legg inn alle elevene i klassen din, angi bordoppstillingen i rommet, og la Klassekatapulten generere et klassekart for deg. Bruk det som det er, eller gjør endringer etter behov.</p>
            <p>Ved å trykke på nedlastingsknappen oppe til høyre, kan du laste ned en liten datafil med informasjon om klassen og bordoppstillingen, som kan åpnes igjen i Klassekatapulten ved en senere anledning. Denne filen «husker» også de siste klassekartene som har blitt generert, og Klassekatapulten bruker dette til å unngå at elever settes sammen flere ganger rett etter hverandre.</p>
            <div class="class-header-container">
                <div class="class-header">
                    <h2>Klasse</h2>
                    <input bind:value={group.name}>
                </div>
                <p>{group.members.length} elever</p>
            </div>
            <ul class="member-container">
                {#each group.members as member, i}
                    <li class="member">
                        <input bind:value={group.members[i].name} placeholder="Navn">
                        <button class="flat" onclick={() => removeMember(member.id)}>×</button>
                    </li>
                {/each}
            </ul>
            <button onclick={addMember} class="pill">+ Ny elev</button>
            <div class="class-header-container">
                <h2>Plasser</h2>
                <p>
                    {#if studentBenchBalance > 0}
                        {studentBenchBalance} til overs
                    {:else if studentBenchBalance < 0}
                        {studentBenchBalance * -1} for lite
                    {:else}
                        Akkurat nok
                    {/if}
                </p>
            </div>
            <div class="seating-container-outer">
                <div class="horizontal-group">
                    <div class="seating-container">
                    {#each group.layout as col, i}
                        <div class="seating-row">
                            {#each col as bench, j}
                                <div class="setup-bench-button-group">
                                    {#if group.layout [i][j] > 0}
                                        {@html benchHTML(group.layout[i][j])}
                                    {:else}
                                        <div class="bench disabled">
                                            <div class="bench-inner setup"></div>
                                        </div>
                                    {/if}
                                    <div>
                                        <button class="flat" onclick={() => group.layout[i][j] --} disabled={group.layout[i][j] <= 0}>-</button>
                                        <button class="flat" onclick={() => group.layout[i][j] ++}>+</button>
                                    </div>
                                </div>
                            {/each}
                        </div>
                    {/each}
                    </div>
                    <div class="vertical-group">
                        <button class="circle" onclick={addLayoutColumn}>+</button>
                        <button class="circle" onclick={removeLayoutColumn} disabled={group.layout.length <= 1}>-</button>
                    </div>
                </div>
                <div class="horizontal-group">
                    <button class="circle" onclick={addLayoutRow}>+</button>
                    <button class="circle" onclick={removeLayoutRow} disabled={group.layout[0].length <= 1}>-</button>
                </div>
                <div class="board">Tavle</div>
            </div>
            <button class="pill suggested" onclick={() => { generateSeating(); mode = "tableMap"; }}>Lag klassekart</button>

            <p class="legal">© Brage Fuglseth Olsen. Avsluttende prosjekt i IT1 på Asker VGS, våren 2025. Klassekatapulten er lisensiert under <a href="https://www.gnu.org/licenses/gpl-3.0.html">GPLv3-lisensen</a>, og <a href="https://github.com/bragefuglseth/klassekatapulten">kildekoden</a> er åpent tilgjengelig.</p>
        </div>
    {:else if mode == "tableMap"}
        <div class="seating-wrapper-wrapper">
            <button class="noprint flat vertical-align" onclick={() => mode = "setup"}><img class="noprint" src="/tilbake.svg" alt="Tilbakepil" height="24"> Tilbake</button>
            <div class="seating-wrapper">
                <div class="class-header-container">
                    <h2>Klassekart for {group.name}</h2>
                    <p>Oppdatert {lastGeneratedString}</p>
                </div>
                <div class="seating-container">
                    {#each group.seating as row, i}
                        <div class="seating-row">
                            {#each row as bench, j}
                                {#if bench.length > 0}
                                <div class="bench">
                                    {#each bench as name, k}
                                        <div class="bench-inner">
                                            <div contenteditable="true" spellcheck="false" class="bench-input" bind:textContent={group.seating[i][j][k]}></div>
                                        </div>
                                    {/each}
                                </div>
                                {:else}
                                <div class="bench invisible"><div class="bench-inner"></div></div>
                                {/if}
                            {/each}
                        </div>
                    {/each}
                </div>
                <div class="board">Tavle</div>
            </div>
            <p style="max-width: 60ch;" class="noprint">Ikke helt det du så for deg? Trykk på «Generer»-knappen for å oppdatere klassekartet, eller trykk på navnene for å endre dem og bytte om på elever.</p>
            <p style="max-width: 60ch;" class="noprint">Når du er ferdig, kan du ta skjermbilde eller skrive ut klassekartet. Ikke glem å gå tilbake til startsiden for å laste ned en fil som kan importeres til Klassekatapulten senere.</p>
            <p class="onlyprint legal">Laget med Klassekatapulten — klassekart.bragefuglseth.no</p>
            <div class="gap">
                <button type="button" class="noprint pill suggested" onclick={generateSeating}>Generer</button>
                <button type="button" class="noprint pill" onclick={() => window.print()}>Skriv ut</button>
            </div>
        </div>
    {/if}
</div>