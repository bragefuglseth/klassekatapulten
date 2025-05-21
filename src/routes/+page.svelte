<script>
    let mode = "setup";

    let group = {
        name: "",
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
        a.download = group.name + "klassekart.json";
        document.body.appendChild(a);
        a.click();
        setTimeout(function() {
            document.body.removeChild(a);
            window.URL.revokeObjectURL(url);  
        }, 0); 
    }

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

    :global(body) {
        overflow: auto;
    }

    p {
        color: #444;
        line-height: 1.25;
    }

    .flat {
        background: none;
        border: none;
        color: rgba(0, 0, 0, 0.7);
        font-size: 1.5rem;
        font-weight: 200;
        transition: transform 200ms;
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
        background-color: slateblue;
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
        align-items: baseline;
    }

    .member-container {
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
        flex-direction: column;
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
        height: 3rem;
        width: 5rem;
        display: flex;
        justify-content: center;
        align-items: center;
        text-align: center;
        padding: 0.5em;
    }
    :global(.bench-inner.setup) {
        height: 2rem;
        max-height: 2rem;
        width: 2rem;
        max-width: 2rem;
    }
    .bench.disabled {
        background-color: transparent;
        border: 1px solid lightgrey;
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
    }

    .seating-wrapper-wrapper {
        display: flex;
        align-items: center;
        flex-direction: column;
    }

    .seating-wrapper {
        margin-bottom: 1em;
    }

    .noprint {
        visibility: visible;
    }

    @media print {
        @page {
            size: landscape;
        }

        .noprint {
            visibility: hidden;
        }
    }
</style>

<div id="wrapper">
    {#if mode == "setup"}
        <div class="setup-container">
            <header class="header">
                <h1>Klassekatapulten</h1>
                <input type="file" id="file-input" bind:files={files} />
                <label class="flat" for="file-input">🗁</label>
                <button class="flat" onclick={saveFile}>🖫</button>
            </header>
            <p>Det har aldri vært enklere å lage klassekart. Legg inn elevene dine, konfigurer klasserommet, og få et fiks ferdig oppsett! Du kan laste ned klassekartfilen til senere bruk. Klassekatapulten vil da huske de siste oppsettene og forsøke å unngå at to elever blir satt sammen flere ganger.</p>
            <h2>Elever</h2>
            <ul class="member-container">
                {#each group.members as member, i}
                    <li class="member">
                        <input bind:value={group.members[i].name} placeholder="Navn">
                        <button class="flat" onclick={() => removeMember(member.id)}>×</button>
                    </li>
                {/each}
            </ul>
            <button onclick={addMember} class="pill">+ Ny elev</button>
            <h2>Oppsett</h2>
            <div class="seating-container-outer">
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
                    <div class="vertical-group">
                        <button class="circle" onclick={addLayoutColumn}>+</button>
                        <button class="circle" onclick={removeLayoutColumn} disabled={group.layout.length <= 1}>-</button>
                    </div>
                </div>
                <div class="horizontal-group">
                    <button class="circle" onclick={addLayoutRow}>+</button>
                    <button class="circle" onclick={removeLayoutRow} disabled={group.layout[0].length <= 1}>-</button>
                </div>
            </div>
            <button class="pill suggested" onclick={() => mode = "tableMap"}>Klassekart</button>
        </div>
    {:else if mode == "tableMap"}
        <button class="noprint flat" onclick={() => mode = "setup"}>⭠ Tilbake</button>
        <div class="seating-wrapper-wrapper">
            <div class="seating-wrapper">
                <h2>Klassekart</h2>
                <p>Oppdatert {lastGeneratedString}</p>
                <div class="seating-container">
                    {#each group.seating as row}
                        <div class="seating-row">
                            {#each row as bench}
                                <div class="bench">
                                    {#each bench as name}
                                        <div class="bench-inner">
                                            <input class="bench-input" value="{name}">
                                        </div>
                                    {/each}
                                </div>
                            {/each}
                        </div>
                    {/each}
                </div>
            </div>
            <div class="horizontal-container">
                <button type="button" class="noprint pill suggested" onclick={generateSeating}>Generer</button>
                <button type="button" class="noprint pill" onclick={() => window.print()}>Skriv ut</button>
            </div>
        </div>
    {/if}
</div>